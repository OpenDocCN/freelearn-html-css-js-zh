# 第十章：控制流

这是我们对 JavaScript 语法的探索的最后一章。到目前为止，我们已经涵盖了它更原子的组件，包括它的许多类型、运算符、声明和语句。熟练掌握这些对于在基础级别有效地使用语言至关重要，现在允许我们退一步考虑一个更大的问题：控制程序的流程。我们将把我们学到的所有语法结合起来，编写干净和易懂的程序。

在本章中，我们将涵盖以下主题：

+   什么是控制流？

+   命令式与声明式编程

+   控制的移动

+   控制流语句

+   处理圈复杂度

+   异步控制流

# 什么是控制流？

控制流指的是表达式和语句（以及整个代码块）运行的顺序。编程在某种程度上是*控制流的艺术*。通过编写代码，我们指定了控制在任何单一时刻的位置。

在细粒度上，执行顺序由我们在表达式中使用的各个运算符决定。在上一章中，我们探讨了运算符的优先级和结合性，发现即使有一系列运算符，一个接一个，它们的执行顺序也由各个运算符的优先级和结合性定义，因此在表达式`1 + 2 * 3`中，`2 * 3`的操作将在加法之前发生。

在语句级别上，除了表达式外，我们以以下方式控制流程：

+   我们可以通过按照我们希望它们发生的顺序来排列我们的语句。

+   我们可以通过使用条件或迭代语言结构来实现，包括以下内容：

+   `switch()`语句

+   `if()`语句

+   `for()`语句

+   `while()`语句

+   `do{...} while()`语句

+   我们可以通过调用函数或生成器来实现，然后从函数或生成器中返回或产出（*产出*和*返回*都是*将控制权交还*给调用者的方式）。

最容易想象控制流程全局地作为一种*光标*或*手指*，它总是指向特定的表达式或代码语句。当程序执行时，控制流将逐行向下进行，直到遇到一段语法，将重定向控制到另一段代码。如果遇到对函数的调用，那么该函数将以相同的方式执行；控制将在函数内的每一行连续进行，直到通过`return`语句将其返回给函数的调用者。当*控制*穿过程序时，它遇到的每个语言结构都将控制执行，直到它们各自完成。考虑以下简单的代码片段：

```js
let basket = [];
for (let i = 0; i < 3; i++) {
  basket.push(
    makeEgg()
  );
}
```

在上述代码中采取的控制流程如下：

1.  我们从`let basket = [];`开始

1.  `for`循环开始：`let i = 0`

1.  检查`i < 3`（为`true`！）：

1.  运行`makeEgg()`

1.  通过`basket.push(...)`推送结果

1.  `i++`（`i`现在是`1`）

1.  检查`i < 3`（为`true`！）：

1.  运行`makeEgg()`

1.  通过`basket.push(...)`推送结果

1.  `i++`（`i`现在是`2`）

1.  检查`i < 3`（为`true`！）：

1.  运行`makeEgg()`

1.  通过`basket.push(...)`推送结果

1.  `i++`（`i`现在是`3`）

1.  检查`i < 3`（为`false`！）。

1.  程序结束

即使对于这样一个非常简单的程序，流程也可能相当复杂且冗长。为了使我们的同行程序员受益，尽可能地减少这种复杂性是有意义的。实现这一点的方法是通过抽象。抽象某事物不会消除复杂性，但它会隐藏它，以便程序员不需要关心它。因此，在深入研究 JavaScript 中控制流的具体语言结构之前，我们将探讨控制流和抽象如何通过*命令式*和*声明式*编程这两种相反的方法相互关联。

# 命令式与声明式编程

命令式编程关注于**如何**完成某事，而声明式编程关注于**我们想要**完成什么。很难看出它们之间的区别，所以最好用一个简单的程序来说明它们：

```js
function getUnpaidInvoices(invoiceProvider) {
  const unpaidInvoices = [];
  const invoices = invoiceProvider.getInvoices();
  for (var i = 0; i < invoices.length; i++) {
    if (!invoices[i].isPaid) {
      unpaidInvoices.push(invoices[i]);
    }
  }
  return unpaidInvoices;
}
```

这个函数的问题领域将是：*获取未付发票*。这是函数的任务，也是我们希望在函数内部实现的*目标*。然而，这个特定的函数非常关注*如何*实现它的任务：

+   它初始化一个空数组

+   它初始化一个计数器

+   它检查计数器（*多次*）

+   它增加了计数器（*多次*）

这个函数的这些和其他元素与*获取未付发票*的问题领域毫不相关。相反，它们是我们必须经历的相当烦人的实现细节。这样的函数被称为**命令式**，因为它们主要关注*如何*。

虽然*命令式*形式的编程忙于任务中涉及的程序低级步骤，*声明式*形式的编程使用抽象来避免直接控制流，更倾向于仅用问题领域本身来表达事物。以下是我们`getUnpaidInvoices`函数的更声明式版本：

```js
function getUnpaidInvoices(invoiceProvider) {
  return invoiceProvider.getInvoices().filter(invoice => {
    return !invoice.isPaid;
  });
}
```

在这里，我们委托给`Array#filter`来处理初始化新数组、迭代和条件检查的具体细节。通过使用抽象，我们摆脱了传统控制流的复杂性。

这样的声明式模式已经成为现代 JavaScript 的主流。它们允许您在问题领域的层面上表达所需的逻辑，而不必担心更低层次的抽象，比如*如何迭代*。重要的是要看到，声明式和命令式方法都不是完全不同的。它们处于光谱的两端。在光谱的声明式一侧，您在更高层次的抽象上操作，因此不会暴露在没有这种抽象的情况下会暴露的实现细节。在光谱的命令式一侧，您在更低层次的抽象上操作，利用更低级别的命令式构造来告诉机器您想要实现的目标：

![](img/e7797e12-6ae7-42f3-b4f0-e8e1d33b4ed3.png)

这两种方法都对我们的控制流产生影响。更命令式的方法直接说明它将一次通过数组迭代，然后有条件地推送到输出数组。更声明式的方法不会对数组如何进行迭代提出任何要求。当然，我们知道原生的`Array#filter`和`Array#map`方法将独立地迭代它们的输入数组，但这不是我们在指定的内容。我们指定的只是我们的数据应该被过滤和映射的条件。数据如何进行迭代完全是`Array#filter`和`Array#map`抽象的关注。

更声明式方法的好处在于它可以增加人类读者的清晰度，并使您能够更有效地对复杂的问题领域进行建模。由于您不必担心*如何*发生事情，您的思维可以纯粹关注*您希望实现的*目标。

想象一下，我们被要求有条件地执行特定的代码片段，但只有在某个功能启用时才能执行。在我们的想法中，这就是它应该工作的方式：

```js
if (feature.isEnabled) {
  // Do the task.
}
```

这是我们想要编写的代码，但后来我们发现事情并不那么简单。首先，我们没有`isEnabled`属性可以在功能对象上使用。但是，有一个`flags`数组属性，当完全禁用时将包括`Feature.DISABLED_FLAG`：

```js
// A feature that is disabled:
feature.flags; // => [Feature.DISABLED_FLAG]
```

这似乎很简单。 但是然后我们发现，即使该功能没有此标志，因此似乎已启用，我们还需要检查当前时间是否与存储在`feature.enabledTimeSlots`中的一组时间对齐。 如果当前时间不在启用的时间段之一，则我们必须得出结论，即使具有该标志，该功能也已禁用。

这开始变得相当复杂。 除了检查*disabled*标志之外，我们还需要通过这些时间段来发现基于当前时间功能当前是否已启用。 因此，我们简单的`if`语句很快就变成了一个难以控制的混乱，具有多层控制流：

```js
let featureIsEnabled = true;

for (let i = 0; i < feature.flags.length; i++) {
  if (feature.flags[i] === Feature.DISABLED_FLAG) {
    featureIsEnabled = false;
    break;
  }
}

if (!featureIsEnabled) {
  for (let i = 0; i < feature.enabledTimeSlots.length; i++) {
    if (feature.enabledTimeSlots[i].isNow()) {
      featureIsEnabled = true;
      break;
    }
  }
}

if (featureIsEnabled) {
  // Do the task.
}
```

这是不受欢迎的复杂代码。 它与我们最初想要编写的原始声明性代码相去甚远。 要理解此代码，其他程序员在扫描每个单独的构造时必须在脑海中维护`featureIsEnabled`的状态。 这是一段令人心烦的代码，因此更容易产生误解，错误和一般的不可靠性。

我们现在必须问自己的关键问题是：我们需要做什么才能将所有这些嵌套的控制流层次抽象出来，以便我们可以恢复我们简单的`if`语句？

我们最终决定将所有这些逻辑放在新创建的`Feature`类中的`isEnabled`方法中-但不仅如此！ 我们决定通过委托给两个内部方法`_hasDisabledFlag`和`_isEnabledTimeSlotNow`来进一步抽象逻辑。 而这些方法本身将它们的迭代逻辑委托给数组方法`includes(...)`和`filter(...)`：

```js
class Feature {
  // (Other methods of the Feature class here,..)

  _hasDisabledFlag() {
    return this.flags.includes(Feature.DISABLED_FLAG);
  }

  _isEnabledTimeSlotNow() {
    return this.enabledTimeSlots.filter(ts => ts.isNow()).length;
  }

  isEnabled() {
    return !this._isDisabledFlag() && this._isEnabledTimeSlotNow();
  }
}
```

这些对`Feature`类的非常小的声明性添加使我们能够编写最初的声明性代码：

```js
if (feature.isEnabled()) {
  // Do the task.
}
```

这不仅仅是一个简单抽象的练习。 这是一个减少控制流层次的练习。 我们避免了使用嵌套的`if`和`for`块的需要，减少了我们自己和其他程序员面临的认知负担，并以最干净的方式完成了最初设定的任务。

通过仔细重构和抽象我们最初混乱的控制流，我们最终得到了一组代码，其中包含了非常少的传统控制流语句（`if`，`for`，`switch`等）。 这并不意味着我们的代码没有控制流； 相反，它意味着控制流要么被最小化，要么被隐藏在抽象的层次下。 在使用 JavaScript 语言的本机控制流构造时，重要的是要记住它们不是您表达程序流程的唯一工具； 您可以将复杂的逻辑重定向和分割为每个处理程序程序流程的非常特定部分的抽象。

现在我们已经对控制流有了坚实的基础理解，并且知道它如何与我们对抽象的了解相融合，我们可以逐个讨论 JavaScript 的各个控制流机制，突出挑战和潜在的陷阱。

# 控制的移动

在 JavaScript 中，有几种控制可以从一段代码移动到另一段代码。 通常，代码将从*左到右*和*上到下*进行评估，直到达到以下任何一种情况：

+   **调用**（通过`fn()`，`` fn` ` ``或者`new fn()`调用函数）

+   **Returning** （通过隐式或显式的`return`从函数返回）

+   **Yielding** （通过`yield`从生成器中产出）

+   **Breaking**（通过`break`从循环或 switch 中断）

+   **Continuing** （通过`continue`继续迭代）

+   **Throwing** （通过`throw`抛出异常）

# 调用

调用以最简单的形式通过显式调用函数来发生。我们可以通过在我们知道是函数的值的左侧附上调用括号（`(...)`)来实现这一点。这个左侧的值可以是直接引用一个持有函数的变量或属性，也可以是一个求值为函数的表达式。

```js
someFunction();
(function(){})();
someObject.someMethod();
[function(){}][0]();
```

要构造实例，正如我们所探讨的，你可以使用`new`操作符，这也是一种调用方式，尽管在零参数的情况下，它在技术上不需要调用括号：

```js
function MyConstructor() {}

// Both equivalent:
new MyConstructor();
new MyConstructor;
```

在调用括号之前（在`(...)`的左侧）的评估的确切语法并不重要，只要它评估为一个函数即可。如果它不是函数，你就会收到 `TypeError`：

```js
1();     // ! TypeError: 1 is not a function
[]();    // ! TypeError: [] is not a function
'wat'(); // ! TypeError: "wat" is not a function
```

当调用一个函数时，JavaScript 将创建一个新的**词法环境**（作用域），在这个环境中，该函数将被评估，函数将成为当前的*执行上下文*，从当前的代码区域转移到函数的代码中。这不应该太让人感到困惑。在代码中，`foo();`、 `baz();` 和 `foo()` 将获得控制权，并在运行完成后才将控制权交给 `baz()`。

一个函数将以以下方式返回控制权给你：

+   通过*returning* （隐式或通过显式的`return`语句）

+   通过*throwing*（由于`SyntaxError`，`TypeError`等隐式地或通过显式的`throw`语句抛出异常）

+   通过*yielding*（在生成器的情况下）

调用也可能通过 JavaScript 的内部机制间接发生。例如，在上一章探讨的强制转换的情况下，诸如`valueOf`、`toString`或`Symbol.toPrimitive`等方法可能会在各种场景下被调用。此外，JavaScript 还使你能够定义*setters*和*getters*，以便在访问或赋值给特定属性时激活你的自定义功能：

```js
const person = {
  set name(name) {
    console.log('You are trying to set the name to', name);
  }
};

person.name = 'Leo';
// Logs: "You are trying to set the name to Leo"
```

在这里给`name`属性赋值，实际上是在调用一个函数，该函数本身可能会执行各种操作，可能会间接调用其他函数。当存在许多这样的隐式调用方式时，你可以想象给定程序的控制流可能会变得难以理解。这样的隐式机制确实有其优点，但如果我们问题领域的大部分逻辑都内嵌在这些地方，那么对同事程序员而言，这些内嵌的逻辑就不那么容易看到，因此更容易造成混淆。

# 返回

*Returning*是从函数转移控制权给其调用方。这既可以在函数内部通过显式的`return`语句实现，也可以在函数运行完毕时隐式地实现：

```js
function sayHiToMe(name) {

 if (name) {
   return `Hi ${name}`;
 }

 // In the case of a truthy `name` this code is never arrived at
 // because `return` exists on a previous line:
 throw 'You do not have a name! :(';

}

sayHiToMe('James'); // => "Hi James"
```

在这里，你会注意到我们没有将一个 falsy 名称的暗指`else`条件放在自己的 else 块（`else {...}`）中，因为这是不必要的。因为当名称为真时我们返回，所以跟在返回语句后面的任何代码都只会在暗指的`else`条件中执行。这种模式在执行输入预检查的函数中很常见：

```js
function findHighestMountain(mountains) {

  if (!mountains || !mountains.length) {
    return null;
  }

  if (mountains.length === 1) {
    return mountains[0];
  }

  // Do the actual work of finding the 
  // highest mountain here...
}
```

正如我们在这里看到的，返回不仅用于将控制返回给调用者，还用于它的副作用：避免存在于其函数中下方行中的工作。这通常被称为*提前返回*，可以显著帮助减少函数的整体复杂性。

# 产出

*产出*是生成器和其调用者之间的控制转移。这是通过`yield`表达式实现的，该表达式可以在其右侧可选地指定一个值（产出的值）。只有在生成器函数中才能使用`yield`语句：

```js
function* makeSomeNumbers() {
  yield 645;
  yield 422;
  yield 789;
}

const iterable = makeSomeNumbers();
iterable.next(); // => {value: 645, done: false}
iterable.next(); // => {value: 422, done: false}
iterable.next(); // => {value: 789, done: false}
```

如果你没有值就产出（`yield;`），那结果将和产出`undefined`一样。

产出将强制后续对生成器函数的调用从产出点继续评估（就好像产出没有发生过一样）。产出可以被视为*暂停*一个函数，有望以后回来继续执行。如果我们在连续的调用中记录生成器运行的哪一部分，我们可以看到这一点：

```js
function* myGenerator() {
  console.log('Chunk A');
  yield;
  console.log('Chunk B');
  yield;
}

const iterable = myGenerator();

console.log('Calling first time');
iterable.next();
console.log('Done calling first time');

console.log('Calling second time');
iterable.next();
console.log('Done calling second time');
```

这将记录以下内容：

+   `"第一次调用"`

+   ``"块 A"``

+   `"第一次调用完成"`

+   `"第二次调用"`

+   `"块 B"`

+   `"第二次调用完成"`

也可以使用普通的`return;`语句从生成器函数中返回。这与最终产出是一样的。也就是说，再也不会在生成器内执行任何代码了。

# 将产出交给了产出

产出不一定只是单向控制的转移。你可以将生成器用作*数据消费者*或*观察者*。在这种情况下，当调用者通过调用`iterable.next()`请求下一个产出的值时，可以选择性地向这个`next()`方法传递一个参数。传递的任何值都将导致生成器中的`yield`表达式评估为该值。

这更容易通过一个例子来解释。在这里，我们创建了一个消耗数字并产出所有先前消耗数字的总和的生成器：

```js
function* createAdder() {
  let n = 0;
  while (true) n += yield n;
}

const adder = createAdder();

adder.next(); // Initialize (kick things off!)

adder.next(100).value; // => 100
adder.next(100).value; // => 200
adder.next(150).value; // => 350
```

在这里，我们使用我们的`yield`表达式（`yield n`）的返回值，并在每次生成器运行时将其添加到`n`的现有值上。我们需要最初调用`next()`一次来启动这一切，因为在这之前，`n += yield n`表达式还没有运行，因此还没有*等待*`next()`的调用。

作为消费者使用生成器并没有很多用例，并且很可能是一种尴尬的模式，因为我们必须使用指定的`next()`方法来传递数据。但是，了解`yield`表达式的灵活性是有用的，因为你在实际应用中可能会遇到。

# 产出的复杂性

对于程序员来说，理解生成器内部控制流可能会变得复杂和难以理解，因为它涉及*来回*很多次调用者和生成器之间的交互。在任何特定点知道正在运行的确切代码可能很难确定，因此建议保持生成器的简短，并确保它们在其他方面一直保持一致——换句话说，在你的生成器内不要有太多不同的生成路径，并且通常尽量保持*圈复杂度*很低（如果您直接跳到*处理圈复杂度*部分，您可以阅读更多相关信息）。

# 中断

*中断*是从当前`for`、`while`、`switch`或带标签的语句内部转移控制到该语句后面的代码。它有效地终止了该语句，阻止后续任何代码的执行。

在迭代的上下文中，是否继续或中断迭代通常由构造本身内的`ConditionExpression`（例如，`counter < array.length`）确定，或者由数据结构的长度在`for..in`和`for..of`的情况下确定。然而，有时仍然可能需要*提前中断*迭代。

例如，如果您正在查找数据结构中的特定项（类似于在大海里找针的情况），那么一旦找到该项就停止查找是有意义的。我们通过中断来实现这一点：

```js
for (let i = 0; i < array.length; i++) {
  if (myCriteriaIsMet(array[i]) {
    happyPath();
    break;
  }
}
```

从迭代中中断将立即停止并退出迭代，这意味着包含的`IterationBody`中的任何剩余代码将不会被执行。随后将执行`IterationBody`后面紧跟的代码。

`break`语句也用于从`switch`语句中退出，通常是在执行相关的`case`语句之后。正如我们稍后将在本章讨论的那样，`switch`语句将将控制转移到与传递给`switch(...)`的值严格相等（`===`）的`case`语句，然后运行所有该`case`语句之后的代码，直到出现显式的`break;`（或者`return;`、`yield;`、`throw;`）：

```js
switch (2) {
  case 1: console.log(1);
  case 2: console.log(2);
  case 3: console.log(3);
  case 4: console.log(4); break;
  case 5: console.log(5);
}

// Logs: 2, 3, 4
```

在这里，我们看到值为`2`将控制转移到匹配的`case 2`，然后 switch 体内的所有后续代码将自然运行，直到遇到`break;`语句。因此，我们只能看到`2`、`3`和`4`的日志。`1`的日志被避免了，因为`case 1`不匹配值`2`，而`5`的日志也被避免了，因为`break;`出现在它之前。

当`switch`中的`case`不中断时，称为**贯穿**。在`switch`语句中使用的这种常见技术在你想要根据多个匹配条件执行单个操作或级联操作时是有用的（我们将在*switch 语句*部分更详细地介绍这个概念）。

在`break`关键字的右侧可能有一个标签，表示`switch`、`for`或`while`语句。如果没有提供标签，JavaScript 将默认认为你是指当前包含的迭代或`switch`结构。只有当你有两个或更多可打破的结构相互嵌套时，例如在一个迭代中嵌套另一个迭代。请注意这里我们如何用`outerLoop`标签标记我们外部的`for`循环，使我们能够从内部的`for`循环中跳出：

```js
outerLoop: for (let obj in objects) {
  for (let key in obj) {
    if (/* some condition */) {
      break outerLoop;
    }
  }
}
```

实际上，你可以跳出*任何*带标签的语句（即使它在迭代或`switch`结构之外），但你必须显式提供标签：

```js
specificWork: {
  doSomeSpecificWork();
  if (weAreFinished) {
    break specificWork;
      // immediately exits the `specificWork: {...}` block
  }
  doOtherWork();
}
```

这种情况非常少见，但是确实值得了解，以防你碰到这样的代码。

最后要注意的一点是关于*跳出*迭代或`switch`语句的是，尽管我们通常使用显式的`break;`语句来做到这一点，但也可以通过其他控制移动的机制有效地发生，例如*yielding*、*returning*或*throwing*。例如，看到使用`return;`来*跳出*不仅是它本身的迭代，也是包含函数的迭代是非常常见的。

# 继续

*Continuing*是一种控制的转移，从当前语句到可能的下一个迭代的开始。它是通过一个`continue`语句来实现的。

`continue`语句在所有迭代构造中都有效，包括`for`、`while`、`do...while`、`for...in`和`for...of`。

这是一个有条件继续的例子，所以迭代体不会对特定项目执行，但迭代仍然会继续进行：

```js
const numbers = [1, 2, 3];

for (const n of numbers) {
  if (n === 2) continue;
  console.log(n);
}

// Logs: 1, 3
```

*Continuing*会跳过当前迭代中`continue`后面的所有代码，然后继续执行接下来的自然情况。

与`break`语句类似，在`continue`关键字的右侧可以选择性地加上一个标签，表示应该继续的哪个迭代构造。如果没有提供标签，JavaScript 将默认认为你是指当前迭代构造。如果你有两个或更多嵌套在一起的迭代构造，那么可能需要使用显式标签：

```js
objectsIteration: for (let obj in objects) {
  for (let key in obj) {
    if (/* some condition */) {
      continue objectsIteration;
    }
  }
}
```

`continue`语句只会在我们原生的循环构造中起作用。如果我们希望在类似`Array#forEach`这样的抽象化循环结构中继续，那么通常我们会希望使用`return`语句（从回调返回，因此继续迭代）。

由于*continuing*是一种控制的移动，我们必须谨慎地考虑我们在传达意图时是否清晰。如果我们有多层循环或多个`continue`或`break`语句，那么会给读者带来不必要的复杂性。

# 抛出

*抛出* 是控制从当前语句转移到调用堆栈上最近的包含 `try...catch` 语句。如果不存在这样的 `try...catch` 语句，则程序的执行将完全终止。抛出通常用于在特定要求或期望不满足时引发异常：

```js
function nameToUpperCase(name) {
  if (typeof name !== 'string') {
    throw new TypeError('Name should be a string');
  }
  return name.toUpperCase();
}
```

要捕获这个错误，我们需要在调用堆栈的某个位置上有一个 `try...catch` 块，包裹住对 `nameToUpperCase` 函数的调用，或者调用这个函数的函数（以此类推）：

```js
let theUpperCaseName;
try {
  theUpperCaseName = nameToUpperCase(null);
} catch(e) {
  e.message; // => "Name should be a string"
}
```

最佳做法是抛出作为原生提供的通用 `Error` 构造函数的实例对象。其中有几个原生的子类构造函数 `Error`：

+   `SyntaxError`：这表示发生了解析错误

+   `TypeError`：这表示在没有其他 `Error` 对象适用的情况下，操作不成功

+   `ReferenceError`：这表示检测到无效的引用值

+   `RangeError`：这表示一个不在可允许值的集合或范围内的值

+   `URIError`：这表示以与其定义不兼容的方式使用了 URI 处理函数

如果您误用本机 API 或产生无效语法，JavaScript 将自然将这些异常提供给您，但您也可以自己使用这些构造器为您的其他程序员提供更语义化的错误。如果以上情况都不适用，则可以直接使用`Error`或从中扩展出自己的专门实例，如下所示：

```js
class NetworkError extends Error {}

async function makeDataRequest() {
  try {
    const response = await fetch('/data');
  } catch(e) {
    throw NetworkError('Cannot fetch data');
  }
  // ... (process response) ...
}
```

所有的 `Error` 实例都会包含 `name` 和 `message` 属性。根据 JavaScript 的实现，可能还会有与错误的堆栈追踪相关的其他属性。在 V8 JavaScript 引擎（用于 Chromium 和 Node.js）和 SpiderMonkey（Mozilla）中都有一个 stack 属性，提供了序列化的调用堆栈信息：

```js
try {
  throw new Error;
} catch(e) {
  e.stack; // => "Error\n at filename.js:2:9"
}
```

可能会出现独特的情况，您希望抛出一个不是 `Error` 实例的值，从技术上讲这是完全合法的，但很少有用。最好只在真正出现错误的情况下进行抛出，并且在这种情况下，最好使用适当的 `Error` 对象来表示该错误。

# 控制流语句

现在我们已经巩固了我们对*控制*在高层次上是如何移动的理解，我们可以进一步探索 JavaScript 给我们控制流的特定语句和机制。我们将探讨每个语句的语法，并结合一些最佳实践和需要避免的陷阱。

# 如果语句

`if` 语句由 `if` 关键词 开始，后面跟着一个括号表达式，再然后是一个额外的语句：

```js
if (ConditionExpression) Statement
```

`ConditionExpression` 可以是无限复杂的表达式，只要它真正是一个表达式：

```js
if (true) {}
if (1 || 2 || 3) {}
if ([1, 2, 3].filter(n => n > 2).length > 0) {}
```

在括号表达式后面的语句可以是一个单行语句或一个 代码块，并指定了当 `ConditionExpression` 评估为真时应运行的代码：

```js
// These are equivalent
if (true) { doBaz(); }
if (true) doBaz();
```

您传递为`ConditionExpression`的值将与布尔值进行比较，以确定其真实性。我们在第六章，*基本和内置类型*中已经恰当地介绍了真实性和虚伪性的概念，但以防万一您生疏了：在 JavaScript 中只有七个虚假值，因此，您可以传递给`if`语句的只有七个可能的值不会满足它：

```js
if (false) {}
if (null) {}
if (undefined) {}
if (0n) {}
if (0) {}
if ('') {}
if (NaN) {}
```

当`if`语句不满足时，它将运行一条可选的`else`语句，您可以在`if`语句后面立即指定。就像`if`一样，您也可以在此处使用一个*块*：

```js
if (isLegalDrinkingAge) drink(); else leave();

// Equivalent, with Blocks:
if (isLegalDrinkingAge) {
  drink();
} else {
  leave();
}
```

您可以有效地*链式*将`if`/`else`语句连接在一起，如下所示：

```js
if (number > 5) {
  // For numbers larger than five
} else if (number < 3) {
  // For numbers less than three
} else {
  // For everything else
}
```

在语法上，重要的是要理解这不是自己的结构（没有像`if`/`else`/`if`/`else`结构一样的东西）；它只是一个常规的`if`语句，然后是一个包含自己`if`/`else`对的`else`语句。因此，也许更准确地看待它如下所示：

```js
if (number > 5) {
  // For numbers larger than five
} else {
  if (number < 3) {
    // For numbers less than three
  } else {
    // For everything else
  }
}
```

当条件有一个或两个可能的结果时，最适合使用`if`语句。如果有更多可能的结果，那么您可能更适合使用 switch 语句。*长*`if`/`else`链条会变得难以操作。稍后在本章中查看*处理圈复杂度*部分，探索处理复杂条件逻辑的其他新颖方法。

# `for`语句

`for`语句用于循环遍历一组，通常是数组或任何可迭代的结构。它有四种广义的变体：

+   **传统 for**：包括以下内容：

    +   **语法**：`for (initializer; condition; incrementer) {...}`

    +   **用法**：通常用于自定义方式在索引结构中进行迭代

+   **For...in**：包括以下内容：

    +   **语法**：`for (let item in object) {...}`

    +   **用法**：用于遍历任何对象的键（通常用于*纯对象*）

+   **For...of**：包括以下内容：

    +   **语法**：`for (let item of iterable) {...}`

    +   **用法**：用于在可迭代的结构（通常是类似数组的结构）上进行迭代

您将使用的`for`结构的类型取决于您希望迭代的确切内容。例如，对于简单的索引和类似数组的结构，`for...of`结构最有用。我们将逐个讨论这些结构，以探讨其用例和潜在挑战。

# 传统的 for

传统的`for`语句用于迭代各种数据结构或概念循环场景。它包括三个表达式，用分号分隔，并且最后是一个语句，它被认为是迭代的*主体*：

```js
for (
  InitializerExpression;
  ConditionExpression;
  UpdateExpression
) IterationBody
```

每个部分的目的如下：

+   `InitializerExpression`初始化迭代；这将首先进行评估，并且仅进行一次。这可以是任何语句（通常包括`let`或`var`分配，但不必是）。

+   `ConditionExpression`检查迭代是否可以继续；在每次迭代之前，将对其进行评估和强制转换为布尔值（就像通过`Boolean(...)`一样），以确定下一次迭代是否会发生。这可能是任何表达式，尽管通常用于检查当前索引是否小于某个上限（通常是您正在迭代的数据结构的长度）。

+   `UpdateExpression`完成每次迭代，准备进行下一次迭代。这将在每次迭代结束时进行评估。这可以是任何陈述，虽然在习惯用法上最常用于增加或减少当前索引。

+   `IterationBody`包含实际的迭代逻辑——将在每次迭代时评估的代码。这通常是一个*块*，但可以是一个单行语句。

使用传统的`for`语句循环遍历数组的代码如下：

```js
for (let i = 0; i < array.length; i++) {
  array[i]; // => (Each `array` item)
}
```

如果只需要遍历常规数组或可迭代结构，则最好使用`for...of`。然而，如果需要对结构进行非常规索引的迭代，那么使用传统的`for`循环可能是合适的。

一个非常规索引结构的示例是`<canvas>`元素的像素数据，它形成一个包含每个像素的 RGBA（红色、绿色、蓝色和 Alpha 通道）值的数组，连续排列，如下所示：

```js
[r, g, b, a, r, g, b, a, ...]
```

由于每个单独的像素占据数组的四个元素，我们需要每次迭代四个索引。传统的`for`循环非常适合于这种情况：

```js
const pixelData = canvas.getContext('2d').getImageData(0, 0, 100, 100).data;

for (let i = 0; i < pixelData.length; i += 4) {
  let red = pixelData[i];
  let blue = pixelData[i + 1];
  let green = pixelData[i + 2];
  let alpha = pixelData[i + 3];
  // (do something with RGBA)
}
```

传统的`for`语句是一个被理解并习惯使用的语法结构。最好确保您使用每个部分来实现其目的。虽然可以（尽管不建议）通过将迭代的实际逻辑包含在结构的括号部分来利用其语法，但这和其他误用对人类来说可能非常难解析：

```js
var copy = [];
for (
  let i = 0;
  i < array.length;
  copy[i] = array[i++]
); 
```

这里的`UpdateExpression`包括`copy[i] = array[i++]`表达式，它将复制当前索引处的数组元素，然后递增索引。后缀`++`运算符确保其操作数的先前值将被返回，从而保证在`copy[i]`上访问的索引始终等于`array[i++]`。这是一个巧妙但相当晦涩的语法。使用习惯用法的`for`结构将会更清晰，它在`for(...)`之后将迭代逻辑放在自己的语句中：

```js
for (
  let i = 0;
  i < array.length;
  i++
) {
  copy[i] = array[i];
}
```

对于大多数程序员来说，这是一个更熟悉和易懂的代码片段。它更冗长，也许写起来不那么有趣，但最终，正如本书的初步章节中所探讨的，我们最感兴趣的是编写能清晰传达其意图的代码。

当然，这个虚构的情景，将一个数组的内容复制到另一个数组中，最好使用`Array#slice`方法（`array.slice()`）来解决，但我们在这里使用它进行说明。

# for...in

`for...in`构造用于迭代对象的一组可枚举属性名称。它具有以下语法:

```js
for (LeftSideAssignment in Object) IterationBody
```

各个部分具有以下限制:

+   `LeftSideAssignment`可以是在每次新迭代中在`IterationBody`范围内评估的任何有效赋值表达式左侧，并且

+   `Object`可以是任何求值为（或可以被强制转换为）对象的表达式——换句话说，除了`null`或`undefined`之外的任何东西。

+   `IterationBody`是任何单行或块语句

`for...in`构造通常用于遍历普通对象的属性:

```js
const city = { name: 'London', population: 8136000 };
for (const key in city) {
  console.log(key);
}
// Logs: "name", "population"
```

你可以看到我们在这里使用`const key`来初始化我们的`key`变量。除非你特别需要`let`的可变行为或`var`的不同作用域行为，否则这是首选的声明。当然，除了不声明，所有这些声明都是完全有效的：

```js
for (let key in obj) {}
for (var key in obj) {}
for (const key in obj) {}
for (key in obj) {}
```

每次迭代都会创建一个新的块作用域。当你使用`let`或`const`声明时，它将作用于该迭代，而通过`var`声明的变量将作用于最近的执行上下文范围（*函数作用域*）。完全不声明也没问题，但你应该确保之前已经初始化了该标识符：

```js
let key;
for (key in obj) {}
```

由于任何在赋值表达式左侧有效的东西在`in`的左侧也是有效的，我们也可以在这里放置一个属性引用，就像下面的例子:

```js
let foo = {};
for (foo.key in obj) {}
```

这将导致`foo.key`在迭代进行中被赋予`obj`的每个键。这将是一个非常奇怪的事情，但仍然可以正确工作。

现在我们介绍了语法，可以讨论`for..in`的行为和用例了。如前所述，它在迭代对象的属性时非常有用。默认情况下，这将包括从对象的`[[Prototype]]`链继承的所有属性，但仅当它们是*可枚举*时：

```js
const objectA = { isFromObjectA: true };
const objectB = { isFromObjectB: true };

Object.setPrototypeOf(objectB, objectA);

for (const prop in objectB) {
 console.log(prop);
}

// Logs: "isFromObjectB", "isFromObjectA"
```

正如你所看到的，对象本身的属性先于继承对象的属性进行迭代。然而，迭代的顺序不应该被依赖，因为这可能会在不同的实现之间有所不同。如果你想以特定顺序迭代一组键，最好通过`Object.keys(obj)`来收集键，然后像遍历数组一样对其进行迭代。

由于`for...in`自然会迭代继承的属性，因此在迭代体内放置附加检查以避免这些属性是传统做法:

```js
for (const key in obj) {
  if (obj.hasOwnProperty(key)) {
    // `key` is a non-inherited (direct) property of `obj`
  }
}
```

当你有一个可迭代对象（比如一个数组）时，最好使用`for...of`，它更适合这种情况，并且性能更好。

# for...of

`for...of`结构用于遍历可迭代对象。原生提供的可迭代对象包括`String`，`Array`，`TypedArray`，`Map`和`Set`。在语法上，`for...of`具有与`for...in`相似的特征：

```js
for (LeftSideAssignment in IterableObject) IterationBody
```

每个部分的目的如下：

+   `LeftSideAssignment`可以是任何在赋值表达式左侧有效的东西，并在每次新迭代中在`IterationBody`的范围内进行评估

+   `IterableObject`可以是任何评估为*可迭代*对象的表达式，换句话说，任何实现`[Symbol.iterator]`为方法的东西

+   `IterationBody`是任何单行或块语句

一个惯用的`for...of`用法可能是这样的：

```js
const array = [1, 2, 3];

for (const i of array) {
  console.log(i);
}

// Logs: 1, 2, 3
```

自从引入语言以来，`for...of`已成为循环数组的最惯用方式，取代了先前惯用的`for (var i = 0; i < array.length; i++) {...}`模式。

`let`，`var`和`const`的作用域行为与上一节关于`for...in`描述的相同。建议使用`const`，因为它将为每次迭代初始化一个新的不可变变量。使用`let`并不可怕，但除非你有特定的原因需要在`IterationBody`内自己对变量进行变化，否则最好使用`const`。

# `while`语句

`while`语句用于运行一段代码，直到某个条件不再被满足。它的语法如下：

```js
while (ConditionExpression) IterationBody
```

每个部分的目的如下：

+   `ConditionExpression`被评估以确定`IterationBody`是否应该运行。如果评估为`true`，那么`IterationBody`部分将运行。然后`ConditionExpression`将被重新评估，依此类推。只有当`ConditionExpression`评估为`false`时，循环才会停止。

+   `IterationBody`可以是单行或块语句，将根据`ConditionExpression`评估为`true`运行多次。

很少使用`while`进行直接迭代，因为有更适合此目的的结构（例如，`for...of`），但如果我们想要，可能会看起来像下面这样：

```js
const array = ['a', 'b', 'c'];

let i = -1;
while (++i < array.length) {
  console.log(array[i]);
}

// Logs: 'a', 'b', 'c'
```

由于我们将`i`初始化为`-1`并使用前缀递增运算符（`++i`），`ConditionExpression`将评估为`0 < array.length`，`1 < array.length`，`2 < array.length`，和`3 < array.length`。自然地，最后一个检查将失败，因为`3`不小于`array.length`，这意味着`while`语句将停止运行其`IterationBody`。这意味着`Body`总共只会运行`3`次。

当迭代的限制尚不明确或以复杂的方式计算时，通常会使用`while`。在这种情况下，常常会看到`true`被直接传递给`ConditionExpression`以在`while(...)`内部强制结束迭代的手动`break;`语句：

```js
while (true) {
  if (/* some custom condition */) {
    break;
  }
}
```

`while` 语句也在生成器函数的上下文中使用，如果这些生成器旨在产生无限的输出。例如，您可能希望创建一个始终产生字母表中的 *下一个* 字母的生成器，然后在到达 `z` 时循环到字母表的开头：

```js
function *loopingAlphabet() {
 let i = 0;
 while (true) {
   yield String.fromCharCode(
     97 + (i >= 26 ? i = 0 : i++)
   );
 }
}

const alphabet = loopingAlphabet();

alphabet.next(); // => { value: "a" }
alphabet.next(); // => { value: "b" }
alphabet.next(); // => { value: "c" }
// ...
alphabet.next(); // => { value: "z" }
alphabet.next(); // => { value: "a" }
alphabet.next(); // => { value: "b" }
// ...
```

这种无限应用的生成器很少见，但它们确实存在，并且是使用 `while(...)` 的理想场所。大多数其他 `while` 的应用已被更简洁且更受限制的迭代方法（如 `for...in` 和 `for...of`）取代。尽管如此，了解如何清晰地使用它还是有用的。

# do...while 语句

`do...while` 语句类似于 while 语句，尽管它保证在执行检查之前会进行一次迭代。其语法由 `do` 关键字后面跟着其主体，然后是典型的带有括号的 `while` 表达式组成：

```js
do IterationBody while (ConditionExpression)
```

每个部分的目的如下：

+   `IterationBody` 可以是单行语句或块语句，并将首先运行一次，然后根据 `ConditionExpression` 的评估结果运行多次。

+   评估 `ConditionExpression` 来确定 `IterationBody` 是否应运行多次。如果评估为 `true`，则将运行 `Body` 部分。然后将重新评估 `ConditionExpression`，依此类推。只有当 `ConditionExpression` 评估为 `false` 时，循环才会停止。

虽然 `do...while` 语句的行为与常规的 `while` 语句不同，但其语义和广泛的应用仍然相同。它在需要在检查是否继续或更改迭代主题之前始终完成至少一个步骤的上下文中最有用。其中一个例子是向上的 DOM 遍历。如果您有一个 DOM 元素并希望在它及其每个 DOM 祖先上运行某些代码，那么可以像下面这样使用 `do...while` 语句：

```js
do {
  // Do something with `element`
} while (element = element.parentNode);
```

像这样的循环将为 `element` 值执行其主体一次，无论 `element` 是什么，然后将评估赋值表达式 `element = element.parentNode`。这个赋值表达式将评估为其新分配的值，这意味着在 `element.parentNode` 为虚假值（例如 `null`）的情况下，`do...while` 将停止其迭代。

在 `while` 或 `do...while` 语句的 `ConditionExpression` 部分分配值相对常见，尽管对其他程序员来说可能不太明显，因此最好只有在代码意图明显的情况下才这样做。如果前面的代码包装在一个名为 `traverseDOMAncestors` 的函数中，那将提供一个有用的线索。

# `switch` 语句

`switch` 语句用于将控制移动到特定的内部 `case` 子句，该子句指定与传递给 `switch(...)` 的值匹配的值。它具有以下语法：

```js
switch (SwitchExpression) SwitchBody
```

`SwitchExpression`将被评估一次，其值将通过严格相等性与`SwitchBody`内的 case 语句进行比较。在`SwitchBody`中可能有一个或多个`case`子句和/或一个`default`子句。`case`子句指定`CaseExpression`，其值将与`SwitchExpression`的值进行比较，其语法如下：

```js
case CaseExpression:
  [other JavaScript statements or additional clauses]
```

`switch`语句通常用于根据特定值指定两个或多个互斥结果的选择。如果条件较少，习惯上会使用`if...else`结构，但为了适应更多的潜在条件，使用`switch`更简单：

```js
function generateWelcomeMessage(language) {

  let welcomeMessage;

  switch (language) {
    case 'DE':
      welcomeMessage = 'Willkommen!';
      break;
    case 'FR':
      welcomeMessage = 'Bienvenue!';
      break;
    default:
      welcomeMessage = 'Welcome!';
  }

  return welcomeMessage;
}

generateWelcomeMessage('DE'); // => "Willkommen!"
generateWelcomeMessage('FR'); // => "Bienvenue!"
generateWelcomeMessage('EN'); // => "Welcome!"
generateWelcomeMessage(null); // => "Welcome!"
```

一旦`switch`机制找到适当的`case`，它将执行所有跟随该`case`语句的代码，直到`switch`语句的最后，或者直到遇到`break`语句为止。使用`break`语句是为了在完成所需的工作时*跳出*`SwitchBody`。

# 中断和穿透

鉴于`switch`语句通常用于根据值执行特定且互不相同的代码块，习惯上在每个`case`语句之间使用`break`，以确保对于任何给定值只执行适当的代码。但有时，希望在情况之间避免中断，让`SwitchBody`代码继续通过多个`case`语句和更多。这样做被称为**穿透**：

```js
switch (language) {

  case 'German':
  case 'Deutsche':
  case 'DE':
    welcomeMessage = 'Willkommen!';
    break;

  case 'French':
  case: 'Francais':
  case 'FR':
    welcomeMessage = 'Bienvenue!';
    break;

  default:
    welcomeMessage = 'Welcome!';
}
```

在这里，你可以看到我们使用了穿透，以便`'German'`、`'Deutsche'`或`'DE'`的任何语言都会导致相同的代码运行`welcomeMessage = 'Willkommen!'`。随后，我们立即中断，以防止任何更多的`SwitchBody`代码运行。

遗憾的是，很容易不小心忘记奇怪的`break;`语句，导致意外的穿透和一个非常困惑的程序员。为了避免这种情况，我建议使用一个具有规则的检查器，该规则在这种情况下发出警告或错误，除非给定特定指令。（我们将在第十五章 *更清洁代码的工具*中更详细地介绍检查器。）

# 直接从开关返回

当你在一个函数中有一个`switch`语句时，有时最好的方法是简单地`return`预期的值，而不是依赖于`break`语句。例如，在`generateWelcomeMessage`中，我们可以简单地返回欢迎字符串。没有必要初始化变量，赋值，和在不同的情况下来回跳转：

```js
function generateWelcomeMessage(language) {
  switch (language) {
    case 'DE':
      return 'Willkommen!';
    case 'FR':
      return 'Bienvenue!';
    default:
      return 'Welcome!';
  }
}
```

直接返回，这种方式可以说比在每个 case 中中断要更清晰，特别是如果每个 case 的逻辑相当简单。

# case 块

通常，`case`或`default`子句之后的代码不止占据一行。因此，习惯上将这些语句包含在一个块中，以便有一种包容性：

```js
switch (speed) {
  case 'slow': {
    console.log('Initiating slow speed');
    car.changeSpeedTo(speed);
    car.enableUrbanCollisionControl();
  }
  case 'fast': {
    console.log('Initiating fast speed');
    car.changeSpeedTo(speed);
    car.enableSpeedLimitWarnings();
    car.enableCruiseControlOption();
  }
  case 'regular':
  default: {
    console.log('Initiating regular speed');
    car.changeSpeedTo(speed);
  }
}
```

这并不是严格必要的，也不会改变任何功能，但它确实为我们的代码读者提供了更多的清晰度。它还为我们引入块级变量铺平了道路，如果我们以后想引入这些变量的话。正如我们所知，在一个由`{`和`}`界定的块中，我们可以使用`const`和`let`来声明仅限于该块的作用域的变量：

```js
switch (month) {
  case 'December':
  case 'January':
  case 'February': {
    const message = 'In the UK, Spring is coming soon!';
    // ...
  }
  //...
}
```

在这里，我们能够声明仅限于`February`情况的特定变量。如果我们有大量逻辑需要隔离，这将会很有用。然而，在这个时候，我们应该考虑以其他方式对这些逻辑进行抽象。冗长的`switch`语句可能是难以理解的。

# 多变条件

经常需要在每个`case`中表达更复杂的条件，而不仅仅是匹配单个值。如果我们将`SwitchExpression`传递为`true`，那么我们可以在每个`CaseExpression`中自由表达自定义的条件逻辑，只要每个`CaseExpression`在成功时都求值为`true`：

```js
switch (true) {
  case user.role === 'admin' || user.role === 'root': {
    // ...
    break;
  }
  case user.role === 'member' && user.isActive: {
    // ...
    break;
  }
  case user.role === 'member' && user.isRecentlyInactive: {
    // ...
    break;
  }
}
```

这种模式允许我们表达更多多变和混合条件。你可能通常倾向于多个`if`/`else`/`if`/`else`语句，但如果你的逻辑可以在一个`switch`语句中表达，那么最好选择这种方式。总是应该考虑你的问题领域的特性和逻辑，并努力做出关于如何实现控制流的明智决定。在某些情况下，`switch`语句可能会变得更加混乱。

在下一节中，我们将介绍一些其他方法，这些方法可以用于处理不适合原生结构（如`switch`）的复杂和冗长逻辑。

# 处理圈复杂度

**圈复杂度**：是衡量程序代码中有多少*线性独立路径*的指标。

考虑一个包含多个条件检查和函数调用的简单程序：

```js
if (a) {
 alpha();
 if (b) bravo();
 if (c) charlie();
}
if (d) delta();
```

即使在这段看似简单的代码中，也存在九条不同的路径。因此，根据`a`、`b`、`c`和`d`的值，可能会有九种`alpha`、`bravo`、`charlie`和`delta`的运行序列：

+   `alpha()`

+   `alpha()` 和 `bravo()`

+   `alpha()`，`bravo()` 和 `charlie()`

+   `alpha()`，`bravo()`，`charlie()` 和 `delta()`

+   `alpha()`，`bravo()` 和 ``delta()``

+   `alpha()` 和 `charlie()`

+   `alpha()`，`charlie()`，和 `delta()`

+   `alpha()` 和 `delta()`

+   `delta()`

高圈复杂度是不可取的，可能会导致以下情况：

+   **认知负荷**：具有圈复杂度的代码对程序员来说可能很难理解。具有许多分支的代码不容易内化并记住，因此更难维护或更改。

+   **不可预测性**：具有圈复杂度的代码可能是不可预测的，特别是在罕见情况下，例如出现了未预料的状态转换或数据底层变化。

+   **脆弱性**：圈复杂的代码在面对变化时可能是脆弱的。改变一行可能会对许多其他行的功能产生不成比例的影响。

+   **Bugginess**：圈复杂的代码可以导致难以捉摸的错误。如果在一个单一函数中有十几个或更多的代码路径，那么维护者可能看不到所有这些，导致回归。

有工具可以量化代码库的圈复杂性。我们将在[第十五章](https://cdp.packtpub.com/clean_code_in_javascript/wp-admin/post.php?post=415&action=edit#post_508)中介绍这些，*更干净代码的工具*。了解高圈复杂性区域可以帮助我们专注于这些区域的维护和测试。

很容易陷入一种情况，在一个单一模块中有太多不同的条件和分支，以至于没有人能够理解发生了什么。除了使用工具来帮助我们识别高复杂性区域外，我们还可以使用自己的判断和直觉。以下是一些我们可以轻松识别和避免的复杂性的例子:

+   一个具有多个`if`/`else`/`if`组合的函数

+   一个有许多子条件的`if`语句（即在`if`语句内部有许多`if`语句）

+   一个`switch`语句，后面跟随着许多子条件的`case`子句

+   在一个`switch`块中有很多`case`子句（例如，超过 20 个将是令人担忧的！）

这些并不是精确的警告，但它们应该给你一个关于你应该注意的内容的想法。当我们发现这样的复杂性时，我们应该做的第一件事是坐下来重新考虑我们的问题领域。我们能否以不同的方式描述我们的逻辑? 我们是否可以创建新的或不同的抽象?

让我们探讨一个具有较高圈复杂度的代码示例，并考虑如何以这些问题为依据来简化它。

# 简化条件分支乱麻

为了说明圈复杂性过高以及我们应该如何简化它，我们将重构一段代码，该代码负责从一组许可证中产生一组 ID 号码和类型：

```js
function getIDsFromLicenses(licenses) {
  const ids = [];
  for (let i = 0; i < licenses.length; i++) {
    let license = licenses[i];
    if (license.id != null) {
      if (license.id.indexOf('c') === 0) {
        let nID = Number(license.id.slice(1));
        if (nID >= 1000000) {
          ids.push({ type: 'car', digits: nID });
        } else {
          ids.push({ type: 'car_old', digits: nID });
        }
      } else if (license.id.indexOf('h') === 0) {
        ids.push({
          type: 'hgv',
          digits: Number(license.id.slice(1))
        });
      } else if (license.id.indexOf('m') === 0) {
        ids.push({
          type: 'motorcycle',
          digits: Number(license.id.slice(1))
        });
      }
    }
  } 
  return ids;
}
```

此函数接受许可证的数组，然后提取这些许可证的 ID 号码(避免`null`或`undefined`ID 的情况)。我们根据 ID 中的字符确定许可证的类型。需要鉴定和提取四种类型的许可证:

+   `car`: 这些是`c{digits}`形式，其中 digits 形成一个大于或等于 1,000,000 的数字

+   `car_old`: 这些是`c{digits}`形式，其中 digits 形成一个小于 1,000,000 的数字

+   `hgv`: 这些是`h{digits}`形式的

+   `motorcycle`: 这些是`m{digits}`形式的

以下是`getIDsFromLicenses`函数的输入和派生输出的示例:

```js
getIDsFromLicenses([
    { name: 'Jon Smith', id: 'c32948' },
    { name: 'Marsha Brown' },
    { name: 'Leah Oak', id: 'h109' },
    { name: 'Jim Royle', id: 'c29283928' }
]);
// Outputs:
[
  {type: "car_old", digits: 32948}
  {type: "hgv", digits: 109}
  {type: "car", digits: 29283928}
]
```

正如你可能已经观察到的那样，我们用于提取 ID 的代码具有相当复杂的圈复杂度。你可能认为它是完全合理的代码，而且它确实是，但它还可以更简单。我们的函数以命令式方式实现了其结果，使用大量语法来解释它希望如何完成任务，而不是它希望完成什么任务。

为了简化我们的代码，首先需要重新审视问题域。我们想要完成的任务是从输入数组中得出一组许可证 ID 类型和值。输出数组几乎与输入数组一一对应，只有许可证的`id`属性为假值（在这种情况下为`null`）的情况除外。以下是我们的输入/输出流程的示例：

```js
[INPUT LICENSES] ==> (DERIVATION LOGIC) ==> [OUTPUT ID TYPES AND DIGITS]
```

从抽象地看，这似乎是使用`Array#map`的绝佳机会。`map`方法允许我们对数组中的每个元素运行一个函数，以得出包含映射值的新数组。

我们要映射的第一件事是将许可证映射到其`id`：

```js
ids = licenses.map(license => license.id)
```

我们需要处理没有`id`的情况。为此，我们可以对衍生的 ID 应用过滤器：

```js
ids = ids.filter(id => id != null)
```

实际上，由于我们知道所有有效的 ID 都是真值，我们可以直接用`Boolean`作为过滤函数进行布尔检查：

```js
ids = ids.filter(Boolean)
```

从中，我们将收到一个包含我们的许可证的数组，但只有那些具有真值`id`属性的许可证。在此之后，我们可以考虑对数据应用的下一个转换。我们想要将`id`值拆分为其构成部分：我们需要 ID 的初始字符（`id.charAt(0)`），然后我们想提取剩余的字符（数字），将它们转换为`Number`类型（`Number(id.slice(1))`）。然后我们可以将这些部分传递给另一个函数，负责从这些信息中提取正确的 ID 字段（`type`和`digits`）：

```js
ids = ids.map(id => getIDFields(
  id.charAt(0),
  Number(id.slice(1))
));
```

`getIDFields`函数需要根据 ID 的单个字符和数字确定类型，返回一个形如`{ type, digits }`的对象：

```js
function getIDFields(idType, digits) {
  switch (idType) {
    case 'c': return {
      type: digits >= 1000000 ? 'car' : 'car_old',
      digits
    };
    case 'h': return { type: 'hgv', digits };
    case 'm': return { type: 'motorcycle', digits };
  }
}
```

由于我们将逻辑的这部分抽象给了一个独立的函数，我们可以独立观察和测试它的行为：

```js
getIDFields('c', 1000); // => { type: "car_old", digits: 1000 }
getIDFields('c', 2000000); // => { type: "car", digits: 1000 }
getIDFields('h', 1000); // => { type: "hgv", digits: 1000 }
getIDFields('i', 1000); // => { type: "motorcycle", digits: 1000 }
```

将所有部分联系在一起，我们最终得到一个类似下面这样的对`getIDsFromLicenses`的新实现：

```js
function getIDsFromLicenses(licenses) {
  return licenses
    .map(license => license.id)
    .filter(Boolean)
    .map(id => getIDFields(
      id.charAt(0),
      Number(id.slice(1))
    ))
}
```

我们在这里取得的成就是大大减少了同行程序员需要处理的圈复杂度。我们利用了`Array#map`和`Array#filter`来抽象决策和迭代逻辑。这意味着我们最终得到了一个更加*声明式*的实现。

你可能还注意到，我们提取了重复逻辑并将其概括化。例如，在我们最初的实现中，我们实现了许多调用来发现 ID 的第一个字符（例如，`license.id.indexOf('m') === 0`）。我们的新实现通过映射到已经包括第一个字符的数据结构来概括这个问题，然后我们可以通过`getIDFields`获得该 ID 的相关`type`和`digits`。

总结来说，我们的一般重构方法包括以下考虑因素：

+   我们以新的视角考虑了问题领域

+   我们考虑了是否有常见的函数式或声明式习惯用法来处理我们的 I/O

+   我们考虑了个别逻辑是否可以抽象化或分离。

现在我们的代码更容易理解，因此更容易维护和调试。它可能也更可靠和稳定，因为其各个单元可以更简单地测试，因此可以避免未来的回归。当然，由于更高程度的抽象化声明习惯和函数的增加使用，可能会导致轻微的性能下降，但这是一个非常边缘的差异，在绝大多数情况下，为了维护性和可靠性的重大益处而实施是值得的。

# 异步控制流

到目前为止，我们看过的大部分构造都用于同步代码，其中语句按顺序评估，每一行完成后下一行开始：

```js
const someValue = getSomeValue();
doSomethingWithTheValue(someValue);
```

像这样的代码很简单。我们直观地理解这两行代码会依次运行。我们还假设这两行代码都不会花费太长时间来执行，可能只需要几个微秒或毫秒。

但是如果我们希望绑定到用户事件或获取一些远程数据会发生什么？这些事情需要时间，只有当未来事件发生时才会完成。在一个不那么友好的宇宙中，除了等待它们完成然后继续执行我们的程序之外，没有其他处理这种情况的方法：

```js
fetchSomeData();
processFetchedData();
```

在这个不友好的宇宙中，`fetchSomeData()`将是一个*阻塞*的函数调用，因为它会阻塞所有其他代码的执行，直到最终完成。这意味着我们将无法执行任何其他重要任务，我们的应用程序基本上会处于停滞状态，直到任务完成，从而对用户体验产生负面影响。

幸运的是，JavaScript 给了我们一个比这更好的世界——一个可以初始化一个任务（比如获取数据），然后在任务运行时继续进行程序的其余部分的世界。这些任务被称为 *异步*，因为它们发生和完成的时间比 *现在* 晚。当它们最终完成时，JavaScript 可以帮助我们通知这一事实，调用任何依赖于该任务完成的代码。

# 事件循环

为了实现这一点，JavaScript 保持单线程的 *事件循环*。当 *事件循环* 开始时，它将运行我们的程序。在执行完一段代码（比如启动我们的程序的代码）后，*事件循环* 会等待消息（或事件），表明发生了什么（例如，网络请求已完成或浏览器 UI 事件已发生）。当它收到消息时，它将执行依赖或监听该事件的任何代码。*事件循环* 将再次运行该代码直到完成，然后继续等待其他消息。这个过程会一直重复下去，直到 JavaScript 程序停止（例如，通过关闭浏览器选项卡）。

*事件循环* 总是运行给定的代码直到完成，这意味着任何长时间运行或 *阻塞* 的代码都会阻止其他代码执行直到它完成。一些旧的浏览器 API 方法，如 `alert()` 和 `prompt()` 就是你可能会遇到的阻塞函数的例子。调用这些函数将有效地阻止 JavaScript 程序的进一步执行：

```js
alert('Hello!');
console.log('The alert has been dismissed by the user');
```

在这里，`console.log()` 在用户关闭警告对话框之前不会被评估。这可能是毫秒、分钟，甚至小时。在此期间，我们的 JavaScript 程序被停止，无法继续执行。它的 *事件循环* 可能正在接收事件，但直到 `alert()` 最终完成才会运行与这些事件相关的代码。

# 本机异步 API

如今，在浏览器和服务器中期望提供非阻塞异步调用本机机制的 API 是很正常的。这类 API 的常见例子包括以下内容：

+   DOM 事件 API，使得能够运行这样的代码：`window.addEventListener('click', callback)`

+   Node.js 文件 API，使得能够运行这样的代码：`fs.readFile(path, callback)`

+   浏览器的 Fetch API，使得能够运行这样的代码：`fetch().then(callback)`

所有这样的接口都有共同之处：它们都提供了一种监听其完成的方式。通常，这是通过提供的回调（函数）实现的。此回调将在任务完成后的某个时刻被调用。同样，一些本机 API 返回 promises，这使得有更丰富的异步控制流机制，但基本上仍然依靠通过 Promise API 传递回调。此外，ECMAScript 2017 引入了异步函数（`async function() {}`）和`await`关键字的概念，最终为 promises 提供了语言支持，这意味着异步工作的完成不再需要回调。

让我们分别探讨这些异步控制流机制。

# 回调

回调是提供连接到异步任务的常规方法。回调只是一个传递给另一个函数的函数，并且预计将在以后的某个时刻被调用，可能是立即，可能很快，或可能永远不会。考虑以下的`requestData`函数：

```js
function requestData(path, callback) {
  // (Implementation of requestData)
}
```

如您所见，它将回调作为其第二个参数。在调用`requestData`时，回调通常会被匿名地内联传递，如下所示：

```js
requestData('/data/123', (response) => { /* ... */ });
```

当然，先前声明回调是完全可以的，这样做可以增加可理解性，因为现在你的代码读者会对何时可能调用回调有所了解。请注意这里我们是如何调用我们的`onResponse`回调的，以明确表明期望在响应可用时（当它完成时）调用它：

```js
function onResponse(response) {
  // Do something with the response...
}

requestData('/data/123', onResponse);
```

类似地，在具有多个异步状态更改的复杂 API 中，通常会看到通过*对象文字*批量注册命名回调：

```js
createDropdownComponent({
  onOpen() {},
  onSelect() {},
  onClose() {},
  onHover() {} // etc.
});
```

回调通常会传递参数，指示已从异步工作中确定的一些重要状态。例如，Node.js 的`readFile`函数会用两个参数调用它的回调函数，即（可能为 null 的）错误和文件本身的（可能为 null 的）数据：

```js
fs.readFile('/path/to/file', (error, data) => {
  if (error) {
    // Handle the error!
  } else {
    // Handle the data! (No error has occurred!)
  } 
});
```

您将回调传递给的函数完全控制何时调用您的回调，如何调用它以及在调用时传递了什么数据。这就是为什么有时会将回调称为*控制反转*。通常情况下，您控制调用哪些函数，但是当使用回调时，控制被颠倒，因此您依赖另一个函数或抽象（在某个时刻）以期望的方式调用您的回调。

*回调地狱*是指在代码片段中不希望存在多个嵌套回调的繁殖现象，通常用于执行一系列相互依赖的异步任务。以下是这种情况的一个示例：

```js
requestData('/data/current-user', (userData) => {
  if (userData.preferences.twitterEnabled) {
    requestData(userData.twitterFeedURL, (twitterFeedData) => {
      renderTwitterFeed(twitterFeedData, {
        onRendered() {
          logEvent('twitterFeedRender', { userId: userData.id });
        }
      });
    });
  }
});
```

在这里，你可以看到我们有三个不同的回调，都出现在一个范围层次的层级结构中。我们等待 `/data/current-user` 的响应，然后我们可以选择地发送请求到 `twitterFeedURL`，最后，在 Twitter feed 渲染(`renderTwitterFeed()`)完成后，我们最终记录了一个 `"twitterFeedRender"` 事件。这个最终的日志取决于前两个异步任务的完成，因此嵌套得非常深。

我们可以看到，这个嵌套深度的代码片段处在一种*水平金字塔* 式缩进的顶峰。这是*回调地狱* 的一个常见特征，因此，你可以将这些*水平金字塔* 的存在视为一个需要注意的事项。当然，并非所有的深缩进都是由回调引起的，但通常在嫌疑名单中排名很高：

![](img/9fabad56-8ede-4573-9fcb-0467a4188f66.png)

为了避免*水平金字塔* 所指示的*回调地狱*，我们应该考虑重新思考和可能重构我们的代码。在上述情况中，记录 Twitter feed 渲染事件，我们可以，例如，有一个通用的*获取和渲染 Twitter feed 数据*的函数。这将简化我们程序的顶层：

```js
requestData('/data/current-user', (userData) => {
  if (userData.preferences.twitterEnabled) {
    renderTwitterForUser(userData);
  }
});
```

请注意，我们在这里缩短了*水平金字塔*。我们现在可以自由地实现`renderTwitterForUser`，并将其作为一个依赖导入。即使其实现可能涉及自己的回调，它对于程序员来说仍然是整体复杂性的减少，因为它将一半的*金字塔*抽象为一个整洁分离的抽象。大多数*回调地狱* 的情况都可以通过重新设计和抽象的类似方法来解决。尽管这是一个简单的情况。对于更加交织的异步任务，可能有必要使用其他异步控制流机制。

# 事件订阅/发射

JavaScript 在订阅和发射事件时感觉非常自然。事件在大多数 JavaScript 程序中都非常常见，无论是处理浏览器中用户派生的事件，还是在 Node.js 中处理服务器端事件。

JavaScript 中有许多与事件相关的操作名称，因此事先了解所有这些名称是很有用的，这样我们在遇到它们时就不会感到困惑。事件是时间上的发生，将导致已订阅该事件的任何回调的调用。订阅事件有很多名称，它们都有效地意味着相同的事情：*订阅*，*注册*，*监听*，*绑定*等。当事件发生时，订阅的回调被调用。这也有许多名称：*调用*，*调用*，*发射*，*触发*等。被调用的实际函数也可以有各种名称：*函数*，*回调*，*监听器*或*处理器*。

从其核心来看，任何支持事件的抽象通常都会通过存储稍后要调用的回调，并使用特定的事件名称作为键，来实现这一点。我们可以想象，DOM 元素可能会将其事件侦听器存储在以下结构中：

```js
{
  "click": [Function, Function, Function],
  "mouseover": [Function, Function],
  "mouseout": [Function]
}
```

任何支持事件的抽象只会简单地存储一系列稍后要调用的回调。因此，当订阅事件时，你需要同时提供你希望它调用的回调和它将与之相关联的事件名称。在 DOM 中，我们会这样做：

```js
document,body.addEventListener('mousemove', e => {
  e; // => the Event object
});
```

在这里，我们看到`Event`对象被传递给回调函数。这是为了简洁起见，习惯上用`e`或`evt`来命名。大多数提供事件 API 的抽象将向回调传递特定的与事件相关的信息。这可能以一个单独的`Event`对象或几个参数的形式传递。

重要的是要注意，事件真的没有单一的标准，尽管已经出现了一些惯例。通常情况下，会始终有一种方法用于注册或订阅事件，然后另一种方法用于取消订阅。以下是一个使用 Node.js 事件发射器 API 的示例，该 API 受到原生 HTTP 模块支持：

```js
const server = http.createServer(...);

function onConnect(req, cltSocket, head) {
  // Connect to an origin server...
}

// Subscribe
server.on('connect', onConnect);

// Unsubscribe
server.off('connect', onConnect);
```

在这里，你可以看到`on()`方法用于订阅事件，而`off()`方法用于退订。大多数事件 API 都有类似的事件注册和取消注册方法，尽管它们可能以不同的方式实现它们。如果你正在设计自己的*事件*实现，那么建议确保你提供一套熟悉的方法和抽象。为此，可以从原生 DOM 事件接口或 Node.js 的**事件发射器**中汲取灵感。这将确保你的事件实现不会让其他程序员感到太惊讶或害怕。

尽管事件 API 本质上只是一系列在特定时间存储和调用的回调，但在设计良好的情况下仍然存在一些挑战。其中包括以下内容：

+   确保单一事件触发时的调用顺序

+   处理事件在其他事件正在进行中发射的情况。

+   处理事件可以完全取消或根据回调移除的情况

+   处理事件可能会被冒泡、传播或委托的情况（这通常是 DOM 的一个挑战）。

*传播*、*冒泡*和*委托*是在分层结构内触发事件相关的术语。在 DOM 中，由于`<div>`可能存在于`<body>`内，事件 API 规定，如果用户点击`<div>`，发射的事件将向上传播或*冒泡*，首先触发`<div>`上的任何`click`监听器，然后是`<body>`上的。委托是在更高层次的层次上有意地监听，例如，在`<body>`级别上进行监听，然后根据事件对象告诉你有关事件的`target`节点的信息做出相应的反应。

事件提供了比简单回调更多的可能性。因为它们允许监听多种不同的事件，并且多次监听同一个事件，任何消费代码在构建其异步控制流时都具有更大的灵活性。具有事件接口的对象可以在整个代码库中传递，并且可能被订阅多次。不同事件的性质意味着不同的异步概念或发生可以被有用地分开，以便其他程序员可以轻松地了解特定情况下会采取哪些操作：

```js
const dropdown = new DropDown();
dropdown.on('select', () => { /*...*/ });
dropdown.on('deselect', () => { /*...*/ });
dropdown.on('hover', () => { /*...*/ });
```

这种透明的分离有助于在程序员的头脑中编码期望。很容易辨别每种情况下将会调用哪个函数。将其与带有内部`switch`语句的泛化的*发生了某事*`事件`进行比较:

```js
// Less transparent & more burdensome:
dropdown.on('action', event => {
  switch (event.action) {
    case 'select': /*...*/; break;
    case 'deselect': /*...*/; break;
    // ...
  }
});
```

良好实施的事件在概念上不同的事件之间提供了很好的语义分离，因此为程序员提供了可以轻松推理的可预测的一系列异步操作。

# *Promise*

*Promise*是包围潜在值概念的抽象。最容易将*Promise*视为一个简单的对象，该对象最终会包含一个值。*Promise*提供了一个接口，通过该接口可以传递回调函数，以等待最终完成值或错误。

在任何给定时间，*Promise*都会具有某种状态:

+   **挂起**: *Promise*正在等待其解析（异步任务尚未完成）。

+   **已解决**: *Promise*不再处于挂起状态，并且已经被完成或拒绝：

    +   **已完成**: *Promise*已成功，现在有一个值

    +   **已拒绝**: *Promise*已因错误而失败

可以通过*Promise*构造函数构造*Promise*，通过传递一个名为*executor*的函数参数（调用`resolve`或`reject`函数来指示已解决值或错误）来构造*Promise*:

```js
const answerToEverything = new Promise((resolve, reject) => {
   setTimeout(() => {
     resolve(42);
   }, 1000);
});
```

实例化的*Promise*具有以下方法，以便我们可以访问其更改的状态（当它从*挂起*转移到*完成*或*拒绝*）：

+   `then(onFulfilled[, onRejected])`: 这将在*Promise*上附加一个*完成*回调，并可选地附加一个*拒绝*回调。它将返回一个新的*Promise*对象，该对象将解析为所调用的完成或拒绝处理程序的返回值，或者如果没有处理程序，则将根据原始*Promise*解析。

+   `catch(onRejected)`: 这将在*Promise*上附加一个*拒绝*回调，并将返回一个新的*Promise*，将解析为回调的返回值或（如果原始*Promise*成功）其完成值。

+   `finally(onFinally)`: 这将在*Promise*上附加一个处理程序，当*Promise*被解决时，无论解决是完成还是拒绝，该处理程序都将被调用。

通过向`then`方法传递回调函数，我们可以访问`answerToEverything`最终解决的值：

```js
answerToEverything.then(answer => {
  answer; // => 42
});
```

通过探索大多数现代浏览器支持的本机 Fetch API，我们可以说明*Promise*的确切性质：

```js
const promiseOfData = fetch('/some/data?foo=bar');
```

`fetch`函数返回一个*Promise*，我们将其赋给我们的变量`promiseOfData`。然后我们可以像这样连接到请求的最终成功（或失败）：

```js
const promiseOfData = fetch('/some/data');

promiseOfData.then(
  response => {
    response; // The "fulfilled" Response
  },
  error => {
    error; // The "rejected" Error 
  }
);
```

也许看起来 Promise 只是比回调更啰嗦的抽象。事实上，在最简单的情况下，你可能只需传递一个*完成*回调和一个*拒绝*回调。可以说，这并没有比原始回调方法提供更有用的内容。但 Promise 可以是更多。

由于*Promise*只是一个常规对象，它可以像任何其他值一样在您的程序中传递，这意味着任务的最终解决不再需要与原始任务的调用站点的代码绑定。此外，每个`then`、`catch`或`finally`调用返回自己的*Promise*，我们可以连接任意数量的依赖某些原始完成的任何同步或异步任务。

例如，在`fetch()`的情况下，完成的`Response`对象提供了一个`json()`方法，该方法本身是异步完成并返回一个*Promise*。因此，要从给定资源获取实际的 JSON 数据，您需要执行以下操作：

```js
fetch('/data/users')
  .then(response => response.json())
  .then(jsonDataOfUsers => {
    jsonDataOfUsers; // the JSON data that we got from response.json()
  });
```

链接`then`调用是一种常用的模式，用于从先前的值派生新值。给定响应，我们希望计算 JSON，而给定 JSON，我们可能希望计算其他内容：

```js
fetch('/data/users')
  .then(response => response.json())
  .then(users => users.map(user => user.forename))
  .then(userForenames => userForenames.sort());
```

在这里，我们使用多个`then`调用来计算我们用户的排序 forenames。实际上，这里创建了四个不同的 promise，如下所示：

```js
const promiseA = fetch('/data/users');
const promiseB = promiseA.then(response => response.json());
const promiseC = promiseB.then(users => users.map(user => user.forename))
const promiseD = promiseC.then(userForenames => userForenames.sort());

promiseA === promiseB; // => false
promiseB === promiseC; // => false
promiseC === promiseD; // => false
```

每个*Promise*只会解决为一个值。一旦它被*完成*或*拒绝*，没有其他值可以取而代之。但正如我们在这里所看到的，我们可以通过简单地通过`then`、`catch`或`finally`注册回调来自原始*Promise*派生一个新的*Promise*。只解决一次并返回新派生的 promise 的性质意味着我们可以以许多有用的方式组合 promise。在我们的例子中，我们可以从我们的`users`数据*Promise*派生两个 promise：一个收集用户的 forenames，另一个收集他们的 surnames:

```js
const users = fetch('/data/users').then(r => r.json());
const forenames = users.then(users => users.map(user => user.forename));
const surnames = users.then(users => users.map(user => user.surname));
```

然后我们可以自由地传递这些`forenames`和`surnames` promises，任何消费代码都可以随意处理它们。例如，当它们最终可用时，我们可能有一个 DOM 元素，我们想要用 forenames 填充它:

```js
function createForenamesComponent(forenamesPromise) {

  const div = document.createElement('div');

  function render(forenames) {
    div.textContent = forenames ? forenames.join(', ') : 'Loading...';
  }

  render(null); // Initial render

  forenamesPromise.then(forenames => {
    // When we receive the forenames we want to render them:
    render(forenames);
  });

  return div; 
}
```

这个`createForenamesComponent`函数接受`forenames`*Promise*作为参数，然后返回一个`<div>`元素。如您所看到的，我们最初用`null`调用`render()`，它用`"loading..."`文本填充 DIV 元素。一旦*Promise*实现了，我们就会重新渲染，用新填充的 forenames 重新渲染。

以这种方式传递 Promise 的能力使它们比回调更加灵活，并且与实现 Events API 的对象精神相似。然而，通过这些机制，有必要创建和传递函数，以便您能监听未来的事件，然后对其进行操作。如果要表达大量的异步逻辑，这可能是一个真正的挑战。代码中到处充斥着回调、事件和 Promise 的控制流可能不清晰，甚至对于熟悉特定代码库的人也是如此。即使少量独立的异步事件也可以在应用程序中产生大量的*状态*。程序员可能会变得非常困惑；困惑与*什么时候*发生*什么*有关。

你的程序的*状态*是在运行时确定的。当一个值或数据发生变化，无论多么小，都将被视为*状态的改变*。*状态*通常以程序输出的形式来表达，例如 GUI 或 CLI 也可以内部保存并在稍后观察的输出中体现。

为了避免混淆，最好尽可能透明地实现与时间相关的代码，以便不会产生误解。以下是一个可能导致误解的代码示例：

```js
userInfoLoader.init();

appStartup().then(() => {
  const userID = userInfoLoader.data.id;
  const userName = userInfoLoader.data.name;
  renderApplication(userID, userName);
});
```

这段代码似乎假设 `appStartup()` 返回的 *Promise* 在 `userInfoLoader` 完成工作后总是会被执行。也许这段代码的作者碰巧知道 `appStartup()` 逻辑总是在 `userInfoLoader` 完成之后执行。也许这是一个确定性。但对于我们来说，第一次阅读这段代码，我们无法确信 `appStartup()` 被执行时 `userInfoLoader.data` 是否已被填充。最好通过更加透明的方式来控制时机，比如，从 `userInfoLoader.init()` 返回一个 *Promise*，然后在该 *Promise* 明确被执行时执行 `appStartup()`。

```js
userInfoLoader.init()
  .then(() => appStartup())
  .then(() => {
    const userID = userInfoLoader.data.id;
    const userName = userInfoLoader.data.name;
    renderApplication(userID, userName);
  });
```

在这里，我们安排我们的代码，使得什么动作依赖于什么其他动作，以及动作的执行顺序显而易见。仅仅使用 Promise，就像任何其他异步控制流抽象一样，并不能保证你的代码会易于理解。重要的是要始终考虑你的同行程序员的视角和他们会做出的时间假设。接下来，我们将探讨 JavaScript 的一个新添加，它为我们提供了对异步代码的本地语言支持：你将看到这些添加如何使我们能够编写更清晰的异步代码，以便清楚地说明*什么时候*发生*什么*。

# 异步和等待

ECMAScript 2017 规范引入了一种新的概念，用 `async` 和 `await` 关键字形式添加到了 JavaScript 语言中。 `async` 关键字用于指定一个函数是异步的：

```js
async function getNumber() {
  return 42;
}
```

这样做，实际上将函数返回的内容包装在`Promise`中（如果它还不是`Promise`的话）。所以，如果我们尝试调用这个函数，我们将收到`Promise`：

```js
getNumber() instanceof Promise; // => true
```

正如我们所了解的，我们可以通过使用`then`方法来订阅`Promise`的满足：

```js
getNumber().then(number => {
  number; // => 42
});
```

与返回*Promises*的异步函数相协作，我们还有一个`await`关键字。这使我们能够等待`Promise`的满足（或拒绝），只需将其传递到`await`的右侧即可。例如，这可能是从`async`函数调用返回的`Promise`：

```js
await someAsyncFunction();
```

或者它可能是内联指定的*Promise*，像这样：

```js
const n = await new Promise(fulfill => fulfill(123));
n; // => 123
```

正如你所看到的，`await`关键字将等待它的*Promise*解决，从而阻止任何后续行动，直到这种情况发生。

以下是另一个例子——一个`setupFeed`异步函数，它等待`fetch()`和`response.json()`：

```js
async function setupFeed() {
  const response = await fetch('/data');
  const json = await response.json();
  console.log(json);
}
```

值得注意的是，`await`关键字不像`alert()`或`prompt()`一样阻塞。相反，它只是暂停异步函数的执行，释放*Event Loop*以继续其他工作，然后，当它的*Promise*解决时，它将在离开的地方继续执行。记住，`await`只是对我们已经实现的功能的语法*糖*。如果我们想要在不使用`async`/`await`的情况下实现我们的`setupFeed`函数，我们可以很容易地通过恢复到将回调传递给`Promise#then`的旧模式来做到这一点：

```js
function setupFeed() {
  fetch('/data').then(response => {
    return response.json()
  }).then(json => {
    console.log(json);
  });
}
```

注意，当我们不使用`await`时，代码略显笨拙和拥挤。与异步函数一起使用`await`可以给我们提供与常规同步代码一样令人满意的线性和程序化外观。这可以大大简化否则复杂的异步控制流程，使我们的同行程序员更清楚*何时*发生*什么*。

`await`关键字也可用于`for...of`迭代结构内部。这样做将等待每个迭代的值。如果在迭代期间遇到任何*Promise*值，那么迭代将不会继续，直到*Promise*被解决为止：

```js
const allData = [
  fetch('/data/1').then(r => r.json()),
  fetch('/data/2').then(r => r.json()),
  fetch('/data/3').then(r => r.json())
];

for await (const data of allData) {
  console.log(data);
}

// Logs data from /data/1, /data/2 and /data/3
```

没有*Promises*或`await`和`async`，表达这种异步过程不仅需要更多的代码，还需要更多的时间来理解。这些构造和抽象的美妙之处在于它们使我们能够忽略异步操作的实现细节，从而使我们能够纯粹地专注于表达我们的问题领域。随着我们在本书中的进展，我们将进一步探索这种抽象精神，因为我们将处理一些更大更棘手的问题领域。

# 总结

在本章中，我们已经完成了对 JavaScript 语言的探索，讨论了命令式和声明式语法之间的区别，探讨了如何清晰地控制流程，并学习了如何处理同步和异步上下文中的圈复杂度情况。这涉及对语言中所有迭代和条件构造的深入研究，对它们的使用进行指导，并警告反模式。

在下一章中，我们将把我们对 JavaScript 语言积累的所有知识与对真实世界设计模式和范式的探索相结合，这将帮助我们构建清晰的抽象和架构。
