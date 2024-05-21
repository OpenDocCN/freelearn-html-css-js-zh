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

+   **调用**（通过`fn()`，`fn```js or `new fn()`)
*   **Returning** (returning from a function via either implicit or explicit `return`)
*   **Yielding** (yielding from a generator via `yield`)
*   **Breaking** (breaking from a loop or switch via `break`)
*   **Continuing** (continuing an iteration via `continue`)
*   **Throwing** (throwing an exception via `throw`)

# Invocation

Invocation occurs, in its most simple form, by explicitly calling a function. We do this by attaching calling parentheses (`(...)`) to a value we know to be a function. This value on the left side of `(...)` can be a direct reference to a variable or property that holds a function or any expression that evaluates to a function:

````调用函数）

someFunction();

(function(){})();

someObject.someMethod();

[function(){}][0]();

```js

To construct instances, as we've explored, you can use the `new` operator. This is also a type of invocation although, in the case of zero arguments, it doesn't technically require calling parentheses:

```

function MyConstructor() {}

// 两者等效：

new MyConstructor();

new MyConstructor;

```js

The exact syntax of evaluation before the calling parentheses (on the left side of `(...)`) is not important as long as it evaluates to a function. If it does not, then you will receive `TypeError`:

```

1();     // ! TypeError: 1 is not a function

[]();    // ! TypeError: [] is not a function

'wat'(); // ! TypeError: "wat" is not a function

```js

When a function is called, JavaScript will create a new **Lexical Environment** (a scope) in which that function will be evaluated, and the function will become the current *execution context*, shifting control from the current area of code to the function's code. This should not be too unintuitive. It makes sense that, in the code, `foo();`, `baz();`, and `foo()` will be given control and will run to completion before `baz()` is then given control.

A function will return control to you in the following ways:

*   By *returning* (implicitly or via an explicit `return` statement)
*   By *throwing* (implicitly due to `SyntaxError`, `TypeError`, and so on or via an explicit `throw` statement)
*   By *yielding* (in the case of a generator)

Invocation can also occur indirectly, via JavaScript's internal mechanisms. For example, in the case of coercion, as explored in the last chapter, methods such as `valueOf`, `toString`, or `Symbol.toPrimitive` may be called in various scenarios. Additionally, JavaScript enables you to define *setters* and *getters* so that your custom functionality is activated whenever a given property is accessed or assigned to:

```

const person = {

set name(name) {

console.log('You are trying to set the name to', name);

}

};

person.name = 'Leo';

// 日志：“您正在尝试将名称设置为 Leo”

```js

By assigning to the `name` property here, we are effectively invoking a function, which itself may then do all manner of things, potentially invoking other functions itself. You can imagine how the control flow of a given program can become potentially incomprehensible when there are many implicit means of invocation such as this. Such implicit mechanisms do have their advantages, but if too much of our problem domain's logic is embedded within such places, then it's less plainly visible to our fellow programmers and hence more likely to cause confusion.

# Returning

*Returning* is a shift of control from a function to its caller. It is achieved either via an explicit `return` statement within the function itself or implicitly when the function runs to completion:

```

function sayHiToMe(name) {

if (name) {

return `Hi ${name}`;

}

// In the case of a truthy `name` this code is never arrived at

// because `return` exists on a previous line:

throw 'You do not have a name! :(';

}

sayHiToMe('James'); // => "Hi James"

```js

Here, you'll notice that we don't bother placing the implied `else` condition of a falsy name in its own else block (`else {...}`) as this would be unnecessary. Because we return when the name is truthy, any code following that return statement will therefore only run in the implied `else` condition. It's quite common to see such patterns in functions that carry out preemptive input checks:

```

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

```js

As we see here, returning is not only used to return control to the caller but also for its side-effect: avoiding work that exists on lines below itself in its function. This is often termed *returning early* and can significantly help to reduce the overall complexity of a function.

# Yielding

*Yielding* is a shift of control between a generator and its caller. It is achieved by the `yield` expression, which can optionally designate a value to its right side (the yielded value). It is only valid to use a `yield` statement within a generator function:

```

function* makeSomeNumbers() {

yield 645;

yield 422;

yield 789;

}

const iterable = makeSomeNumbers();

iterable.next(); // => {value: 645, done: false}

iterable.next(); // => {value: 422, done: false}

iterable.next(); // => {value: 789, done: false}

```js

If you yield without a value (`yield;`) then the result will be the same as yielding `undefined`. 

Yielding will force any subsequent calls to the generator function to continue evaluation from the point of yield (as if the yield hadn't occurred). Yielding can be thought of as *pausing* a function with the prospect of coming back to it later. We can see this in action if we log which part of our generator runs during consecutive calls:

```

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

```js

This will log the following:

*   `"Calling first time"`
*   ``"Chunk A"``
*   `"Done calling first time"`
*   `"Calling second time"`
*   `"Chunk B"`
*   `"Done calling second time"`

It is also possible to return from a generator function with a regular `return;` statement. This is the same as yielding for the final time. That is, no further code will ever be run within that generator. 

# Yielding to a yield

Yielding is not necessarily a shift of control in just one direction. You can use a generator as a *data consumer* or *observer*. In such scenarios, when a caller requests the next yielded value by calling `iterable.next()`, it can optionally pass an argument to this `next()` method. Whatever value is passed will then cause the `yield` expression within the generator to evaluate to that value.

This is more easily explained with an example. Here, we have created a generator that consumes numbers and yields the sum of all numbers previously consumed:

```

function* createAdder() {

let n = 0;

while (true) n += yield n;

}

const adder = createAdder();

adder.next(); // Initialize (kick things off!)

adder.next(100).value; // => 100

adder.next(100).value; // => 200

adder.next(150).value; // => 350

```js

Here, we are using the return value of our `yield` expression (`yield n`) and then adding it to the existing value of `n` on each run of the generator. We need to call `next()` once initially to kick things off as, before this, the `n += yield n` expression has not been run and is hence is not *waiting for* a `next()` call yet.  

Using generators as consumers does not have many use cases and can be quite an awkward pattern to employ since we must use the designated `next()` method to pass in data. It is, however, useful to know about the flexibility of the `yield` expression since you may encounter it in the wild.

# Complexity of yielding

For fellow programmers, comprehending the flow of control within generators can be complicated and counter-intuitive since it involves a lot of *back-and-forth* between the caller and the generator. Knowing what exact code is running at any specific point may be difficult to determine and so it is advisable to keep your generators short and ensure that they yield consistently—in other words, don't have too many different pathways of yielding within your generators and generally attempt to keep *cyclomatic complexity* quite low (you can read more about this if you skip ahead to the *Handling cyclomatic complexity* section).

# Breaking

*Breaking* is a shift of control from within the current `for`, `while`, `switch`, or labeled statement to the code following the statement. It effectively terminates the statement, preventing any following code from being executed.

In the context of iteration, whether or not to continue or break from iteration is usually determined by `ConditionExpression` within the construct itself (for example, `counter < array.length`), or by the length of the data structure in the case of `for..in` and `for..of`. However, it may still be necessary, at times, to *break out* of the iteration early.

For example, if you are looking for a specific item within a data structure (a *needle-in-a-haystack* situation), then it would make sense to stop looking once the item is found. We achieve that by breaking:

```

for (let i = 0; i < array.length; i++) {

if (myCriteriaIsMet(array[i]) {

happyPath();

break;

}

}

```js

Breaking from an iteration will immediately halt and exit the iteration, meaning any remaining code within the containing `IterationBody` will not be run. The code immediately following `IterationBody` will then run.

The `break` statement is also used to break out from `switch` statements, typically when you have executed the relevant `case` statement. As we will discuss later in this chapter, the `switch` statement will transfer control to the `case` statement that is considered strictly equal (`===`) to the value passed to `switch(...)`, and will then run all code following that `case` statement until an explicit `break;` (or `return;`, `yield;`, or `throw;`) occurs:

```

switch (2) {

case 1: console.log(1);

case 2: console.log(2);

case 3: console.log(3);

case 4: console.log(4); break;

case 5: console.log(5);

}

// Logs: 2, 3, 4

```js

Here, we see that a value of `2` shifts control to the matching `case 2`, and then all of the following code within the switch's body will run naturally until a `break;` statement is encountered. Hence, we only see logs for `2`, `3`, and `4`. A log for `1` is avoided as `case 1` does not match the value, `2`, and a log for `5` is avoided as `break;` occurs before it.

When `case` within `switch` does not break, it is called **fallthrough**. This common technique used in `switch` statements is useful when you want to carry out a single action or cascade of actions based on more than one matching condition (we will cover this concept more in the *The* *switch statement se*).

To the right side of the `break` keyword there may be a label that refers to the `switch`, `for`, or `while` statement. If you don't supply a label, then JavaScript will assume you are referring to the current containing iteration or `switch` construct. This is only useful when you have two or more breakable constructs within each other, for example, an iteration within an iteration. Observe here how we've labeled our outer `for` loop with the `outerLoop` label, enabling us to break out of it from within the inner `for` loop:

```

outerLoop: for (let obj in objects) {

for (let key in obj) {

if (/* some condition */) {

break outerLoop;

}

}

}

```js

You can, in fact, break out of any labeled statement (even if it is outside of an iteration or `switch` construct) but you must explicitly provide the label:

```

specificWork: {

doSomeSpecificWork();

if (weAreFinished) {

break specificWork;

// immediately exits the `specificWork: {...}` block

}

做其他工作();

}

```js

This is very rarely applicable but is nonetheless worth knowing about in case you ever run into such code.

One last thing to note on *breaking out* of iterations or `switch` statements is that, although we typically do so by using an explicit `break;` statement, it is something that can also effectively occur via other mechanisms of moving control such as *yielding*, *returning*, or *throwing*. It's quite common, for example, to see an iteration that uses `return;` to *break out* not only of itself but also of the containing function.

# Continuing

*Continuing* is a shift of control from the current statement to the potential start of the next iteration. It is achieved via a `continue` statement.

The `continue` statement is valid in all iteration constructs, including `for`, `while`, `do...while`, `for...in`, and `for...of`.

Here is an example of continuing conditionally, so that the body of the iteration does not execute for a specific item but the iteration still continues to progress:

```

const numbers = [1, 2, 3];

for (const n of numbers) {

if (n === 2) continue;

console.log(n);

}

// Logs: 1, 3

```js

*Continuing* skips all of the code following `continue` in the current iteration and then moves onto whatever would naturally occur next. 

Similar to the `break` statement, to the right side of the `continue` keyword can optionally be a label that indicates which iteration construct should be continued. If you don't supply it, then JavaScript will assume you are referring to the current iteration construct. If you have two or more iteration constructs nested within each other, then it may be necessary to use an explicit label:

```

objectsIteration: for (let obj in objects) {

for (let key in obj) {

if (/* some condition */) {

continue objectsIteration;

}

}

}

```js

The `continue` statement will only work in our native looping constructs. If we wish to continue in an abstracted looping construct such as `Array#forEach`, then we'll typically want to use a `return` statement instead (to return from the callback and hence continue the iteration).

Since *continuing* is a movement of control, we want to remain cautious about how clearly we are communicating our intent. If we have several layers of loops or several `continue` or `break` statements, it can burden the reader with an unnecessary level of complexity. 

# Throwing

*Throwing* is a shift of control from the current statement to the nearest containing `try...catch` statement on the call stack. If no such `try...catch` statement exists, then the execution of the program will terminate entirely. Throwing is conventionally used to raise exceptions when specific requirements or expectations are not met:

```

function nameToUpperCase(name) {

if (typeof name !== 'string') {

throw new TypeError('Name should be a string');

}

return name.toUpperCase();

}

```js

To catch this error, we would need to have a `try...catch` block somewhere on the call-stack, wrapping the call to the `nameToUpperCase` function or the call to the function that calls it (and so on):

```

let theUpperCaseName;

try {

theUpperCaseName = nameToUpperCase(null);

} catch(e) {

e.message; // => "Name should be a string"

}

```js

It is a best practice to throw objects that are instances of the natively provided generic `Error` constructor. There are several native sub-classed constructors of `Error`:

*   `SyntaxError`: This indicates that a parsing error has occurred
*   `TypeError`: This indicates an unsuccessful operation when none of the other `Error` objects are appropriate
*   `ReferenceError`: This indicates that an invalid reference value has been detected
*   `RangeError`: This indicates a value that is not in the set or range of allowable values
*   `URIError`: This indicates that a URI handling function was used in a way that is incompatible with its definition

JavaScript will naturally raise such exceptions to you if you misuse native APIs or produce invalid syntax, but you can also use these constructors yourself to provide more semantically meaningful errors to your fellow programmers. If none of the preceding are suitable, then you can directly use `Error` or extend from it to produce your own specialized instance, as follows:

```

class NetworkError extends Error {}

async function makeDataRequest() {

try {

const response = await fetch('/data');

} catch(e) {

throw NetworkError('Cannot fetch data');

}

// ... (process response) ...

}

```js

All `Error` instances will contain a `name` and `message` property. Depending on the JavaScript implementation, there may also be additional properties related to the stack trace of the error. In both the V8 JavaScript engine (used in Chromium and Node.js) and in SpiderMonkey (Mozilla), there is a stack property that gives us serialized call stack information:

```

try {

throw new Error;

} catch(e) {

e.stack; // => "Error\n at filename.js:2:9"

}

```js

There may be unique situations where you wish to throw a value that is not an `Error` instance, and technically, this is perfectly legal, but it is rarely useful to do so. It's best to only throw in the case of an actual error, and in that case, it is best to use an appropriate `Error` object to represent the error. 

# Statements of control flow

Now that we've cemented our understanding of how *control* is moved at a high level, we can delve further into the specific statements and mechanisms that JavaScript gives us to control flow. We'll combine an exploration of the syntax of each statement with some best practices and pitfalls to avoid.

# The if statement

The `if` statement is composed of the `if` keyword followed by a parenthesized expression and then an additional statement:

```

if (ConditionExpression) Statement

```js

`ConditionExpression` can be of limitless complexity as long as it is truly an expression:

```

if (true) {}

if (1 || 2 || 3) {}

if ([1, 2, 3].filter(n => n > 2).length > 0) {}

```js

The statement following the parenthesized expression can be a single-line statement or a block and designates the code that should be run if the `ConditionExpression` evaluates to a truthy value:

```

// These are equivalent

if (true) { doBaz(); }

if (true) doBaz();

```js

The value you pass as `ConditionExpression` is compared to a Boolean to determine its truthiness. We've already been aptly introduced to the concepts of truthiness and falsiness in Chapter 6*, Primitive and Built-In Types,* but just in case you're rusty: there are only seven falsy values in JavaScript, and as such, only seven possible values that you can pass to an `if` statement that won't satisfy it:

```

if (false) {}

if (null) {}

if (undefined) {}

if (0n) {}

if (0) {}

if ('') {}

if (NaN) {}

```js

When an `if` statement is not satisfied, it will run an optional `else` statement, which you may specify immediately following your `if` statement. Just as with `if`, you may use a block here as well:

```

if (isLegalDrinkingAge) drink(); else leave();

// Equivalent, with Blocks:

if (isLegalDrinkingAge) {

drink();

} else {

leave();

}

```js

You can effectively *chain* together `if`/`else` statements as follows:

```

if (number > 5) {

// For numbers larger than five

} else if (number < 3) {

// For numbers less than three

} else {

// For everything else

}

```js

Syntactically, it's important to understand that this isn't a construct of its own (there is no such thing as an `if`/`else`/`if`/`else` construct); it is merely a regular `if` statement, followed by an `else` statement that itself contains its own `if`/`else` duo. Therefore, it is more accurate, perhaps, to see it as follows:

```

if (number > 5) {

// For numbers larger than five

} else {

if (number < 3) {

// For numbers less than three

} else {

// For everything else

}

}

```js

An `if` statement is best suited for when there are one or two possible outcomes of a condition. If there are more possible outcomes, then you may be better off using a switch statement. Long `if`/`else` chains can get unwieldy. See the *Handling cyclomatic complexity* section later in this chapter to explore other novel ways of handling complex conditional logic.

# The for statement

The `for` statement is used to iterate through a set, typically, an array or any iterable structure. It comes in four broad varieties:

*   **Conventional for**: This includes the following:
    *   **Syntax**: `for (initializer; condition; incrementer) {...}`
    *   **Usage**: Typically used to iterate in a custom fashion through an indexed structure 
*   **For...in**: This includes the following:
    *   **Syntax**: `for (let item in object) {...}`
    *   **Usage**: Used to iterate through the keys of any object (typically used on *plain objects*)
*   **For...of**: This includes the following:
    *   **Syntax**: `for (let item of iterable) {...}`
    *   **Usage**: Used to iterate over an iterable (typically array-like) structure

The type of `for` construct you'll employ will depend on what exactly you wish to iterate over. For straightforward indexed and array-like structures, for example, the `for...of` construct will be most useful. We'll go over each of these constructs to explore use cases and potential challenges.

# Conventional for

The conventional `for` statement is used to iterate over all manner of data structures or conceptual looping scenarios. It includes three expressions, parenthesized and separated by semicolons, and a statement at the end, which is considered the body of the iteration:

```

for (

InitializerExpression;

ConditionExpression;

UpdateExpression

) IterationBody

```js

The purpose of each part is as follows:

*   The `InitializerExpression` initializes the iteration; this will be evaluated first and only once. This can be any statement (it usually includes a `let` or `var` assignment, but doesn't need to).
*   The `ConditionExpression` checks whether the iteration may continue; this will be evaluated and coerced to a Boolean (as if via `Boolean(...)`) before each iteration to determine whether the next iteration will occur. This can be any expression, though it is usually used to check whether the current index is less than some upper bound (usually the length of the data structure that you are iterating through).
*   The `UpdateExpression` finalizes each iteration, ready for the next iteration. This will be evaluated at the end of each iteration. This can be any statement though is most idiomatically used to increment or decrement the current index.
*   The `IterationBody` contains the actual iteration logic—the code that will be evaluated on every iteration. This is typically a *block* but can be a single-line statement.

Using the conventional `for` statement to loop over an array would look like this:

```

for (let i = 0; i < array.length; i++) {

array[i]; // => (Each `array` item)

}

```js

It is preferable to use `for...of` if you're just iterating over a regular array or iterable structure. However, if you need to iterate over a structure indexed unconventionally, then it may be appropriate to use the conventional `for` loop.

An example of an unconventionally indexed structure is the pixel data of a `<canvas>` element, which forms an array containing the RGBA (*Red, Green, Blue*, and *Alpha*) values of every pixel consecutively, like so:

```

[r, g, b, a, r, g, b, a, ...]

```js

Since each individual pixel occupies four elements of the array, we would need to iterate over it four indexes at a time. The conventional `for` loop is perfectly suited to this:

```

const pixelData = canvas.getContext('2d').getImageData(0, 0, 100, 100).data;

for (let i = 0; i < pixelData.length; i += 4) {

let red = pixelData[i];

let blue = pixelData[i + 1];

let green = pixelData[i + 2];

let alpha = pixelData[i + 3];

// (do something with RGBA)

}

```js

The conventional `for` statement is a well understood and idiomatic piece of syntax. It is best to ensure that you use each of its parts for its purpose. It is entirely possible (though unadvisable) to exploit its syntax by including the actual logic of your iteration in the parenthesized portion of the construct, but this and other misuses can be quite hard to parse for humans:

```

var copy = [];

for (

let i = 0;

i < array.length;

copy[i] = array[i++]

);

```js

`UpdateExpression` here includes the `copy[i] = array[i++]` expression, which will copy across the element of the array at the current index and will then increment the index. The postfix `++` operator ensures that the previous value of its operand will be returned, guaranteeing that the index accessed on `copy[i]` is always equal to `array[i++]`. This is a clever but rather obscure syntax. It would have been far clearer to use the idiomatic `for` structure, which places the iteration logic in its own statement after `for(...)`:

```

for (

let i = 0;

i < array.length;

i++

) {

copy[i] = array[i];

}

```js

This is a more familiar and comprehensible piece of code for most programmers. It is more verbose, and perhaps not as fun to write, but in the end, as explored in the initial chapters of this book, we are most interested in writing code that communicates its intent clearly.

Naturally, this fictional scenario, copying the contents of one array to another array, would be better solved by using the `Array#slice` method (`array.slice()`) but we used it here as an illustration.

# for...in

The `for...in` construct is used to iterate over an  object's set of enumerable property names. It has the following syntax:

```

for (LeftSideAssignment in Object) IterationBody

```js

The various parts have the following constraints:

*   `LeftSideAssignment` can be anything that would be valid on the left side of an assignment expression and is evaluated within the scope of `IterationBody` on every new iteration
*   `Object` can be any expression that evaluates to (or can be coerced to) an object—in other words, anything except `null` or `undefined`
*   `IterationBody` is any single-line or block statement

The `for...in` construct is usually used to iterate through a plain object's properties:

```

const city = { name: 'London', population: 8136000 };

for (const key in city) {

console.log(key);

}

// Logs: "name", "population"

```js

You can see that we're using `const key` here to initialize our `key` variable on each iteration. This is the preferred declaration to use unless you have a specific need for the mutable behavior of `let` or the different scoping behavior of `var`. Naturally, all of these declarations are perfectly valid to use, in addition to using no declaration whatsoever:

```

for (let key in obj) {}

for (var key in obj) {}

for (const key in obj) {}

for (key in obj) {}

```js

A new block scope is created for each iteration. When you use either a `let` or `const` declaration, it will be scoped to that iteration, while a variable declared via `var` will, as we know, be scoped to the nearest execution context's scope (*function scope*). Using no declaration whatsoever is fine, but you should ensure that you have already initialized that identifier beforehand:

```

let key;

for (key in obj) {}

```js

Since anything that would be valid on the left side of an assignment expression is valid on the left side of `in`, we can also place a property reference here, as in the following example:

```

let foo = {};

for (foo.key in obj) {}

```js

This would result in `foo.key` being assigned each key of `obj` as the iteration progresses. This would be quite an odd thing to do, but will nonetheless work correctly.

Now that we have the syntax out of the way, we can discuss the behavior and use cases of `for..in`. It is, as mentioned, useful in iterating through the properties of an object. By default, this will include all properties inherited from the object's `[[Prototype]]` chain as well, but only if they are *enumerable*:

```

const objectA = { isFromObjectA: true };

const objectB = { isFromObjectB: true };

Object.setPrototypeOf(objectB, objectA);

for (const prop in objectB) {

console.log(prop);

}

// 输出："isFromObjectB", "isFromObjectA"

```js

As you can see, properties on the object itself are iterated over before those from inherited objects. The order of iteration, however, should not be depended upon as this may differ between implementations. If you're looking to iterate through a set of keys in a specific order, it may be better instead to gather the keys via `Object.keys(obj)` and then iterate over that as an array.

Since `for...in` will naturally iterate over inherited properties, it's conventional to place an additional check within the iteration body to avoid these properties:

```

for (const key in obj) {

if (obj.hasOwnProperty(key)) {

// `key` is a non-inherited (direct) property of `obj`

}

}

```js

Where you have an iterable object (such as an array), it is advisable to use `for...of` instead, which is more performant and idiomatic for such situations. 

# for...of

The `for...of` construct is used to iterate over an iterable object. Natively provided iterable objects include `String`, `Array`, `TypedArray`, `Map`, and `Set`. Syntactically, `for...of` shares the characteristics of `for...in`:

```

for (LeftSideAssignment in IterableObject) IterationBody

```js

The purpose of each part is as follows: 

*   `LeftSideAssignment` can be anything that would be valid on the left side of an assignment expression and is evaluated within the scope of `IterationBody` on every new iteration
*   `IterableObject` can be any expression that evaluates to an *iterable* object—in other words, anything that implements `[Symbol.iterator]` as a method
*   `IterationBody` is any single-line or block statement

An idiomatic `for...of` usage may look like this:

```

const array = [1, 2, 3];

for (const i of array) {

console.log(i);

}

// 输出: 1, 2, 3

```js

Since its introduction into the language, `for...of` has become the most idiomatic way to loop over arrays, replacing the previously idiomatic `for (var i = 0; i < array.length; i++) {...}` pattern.

The scoping behavior of `let`, `var`, and `const` is identical to that described in the last section on `for...in`. It is advisable to use `const` as it will initialize a fresh and immutable variable for each iteration. Using `let` is not awful but, unless you have a specific reason to need to mutate the variable yourself within `IterationBody`, you'll be better off using `const`.

# The while statement

The `while` statement is used to run a piece of code until some condition stops being met. It has the following syntax:

```

while (ConditionExpression) IterationBody

```js

The purpose of each part is as follows: 

*   `ConditionExpression` is evaluated to determine whether `IterationBody` should run. If it evaluates to `true`, then the `IterationBody` portion will run. `ConditionExpression` will then be re-evaluated and so on. The cycle only stops when `ConditionExpression` evaluates to `false`.
*   `IterationBody` can be either a single-line or block statement and will be run as many times as `ConditionExpression` evaluates to `true`.

It is rare to use `while` for straightforward iteration because there are more suitable constructs for this (for example, `for...of`), but if we wanted to, it might look something like the following:

```

const array = ['a', 'b', 'c'];

let i = -1;

while (++i < array.length) {

console.log(array[i]);

}

// 输出: 'a', 'b', 'c'

```js

Since we are initializing `i` to `-1` and are using the prefix increment operator (`++i`), `ConditionExpression` will evaluate to `0 < array.length`, `1 < array.length`, `2 < array.length`, and `3 < array.length`. Naturally, the last check will fail as `3` is not less than `array.length`, meaning that the `while` statement will stop running its `IterationBody`. This means `Body` will only `3` times in total.

It's common to use `while` when the limit of iteration is, as yet, unknown or computed in a complex fashion. In such instances, it is common to see `true` directly passed as `ConditionExpression` to `while(...)` and then a manual `break;` statement within the iteration to force it to end:

```

while (true) {

if (/* some custom condition */) {

break;

}

}

```js

The `while` statement is also used in the context of generator functions if those generators are intended to produce infinite outputs. For example, you may wish to create a generator that always produces the *next* letter in an alphabet, and then loops round to the start of the alphabet when it gets to `z`:

```

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

```js

Such infinite applications of generators are rare but they do exist and are a perfect place to use `while(...)`. Most other applications of `while` have been replaced with more succinct and contained methods of iteration such as `for...in` and `for...of`. Nonetheless, it is useful to know how to cleanly wield it.

# The do...while statement

The `do...while` statement is similar to while it although guarantees an iteration before the check is carried out. Its syntax is formed of the `do` keyword followed by its body and then a typical parenthesized `while` expression:

```

do IterationBody while (ConditionExpression)

```js

The purpose of each part is as follows: 

*   `IterationBody` can be either a single-line or block statement and will be run once initially and then as many times as `ConditionExpression` evaluates to `true`.
*   `ConditionExpression` is evaluated to determine whether `IterationBody` should run more than once. If it evaluates to `true`, then the `Body` portion will run. `ConditionExpression` will then be re-evaluated and so on. The cycle only stops when `ConditionExpression` evaluates to `false`.

Although the behavior of the `do...while` statement is different from  regular `while` statement, its semantics and broad applications remain the same. It is most useful in contexts where you need to always complete at least one step of an iteration before either checking whether to continue or changing the subject of the iteration. An example of this would be upward DOM traversal. If you have a DOM element and wish to run certain code on it and each of its DOM ancestors, then you may wish to use a `do...while` statement as follows:

```

do {

// Do something with `element`

} while (element = element.parentNode);

```js

A loop like this will execute its body once for the `element` value, whatever `element` is, and then will evaluate the assignment expression, `element = element.parentNode`. This assignment expression will evaluate to its newly assigned value, meaning that, in the case of `element.parentNode` being falsy (for example, `null`) the `do...while` will halt its iteration.

Assigning values in the `ConditionExpression` portion of a `while` or `do...while` statement is relatively common although it can be obscure to fellow programmers, so it's best to only do so if it's plainly obvious what the intent of the code is. If the preceding code was wrapped in a function called `traverseDOMAncestors`, then that would provide a helpful clue.

# The switch statement

The `switch` statement is used to move control to a specific inner `case` clause that specifies a value that matches the value passed to `switch(...)`. It has the following syntax:

```

switch (SwitchExpression) SwitchBody

```js

`SwitchExpression` will be evaluated once and its value compared via strict-equality to case statements within `SwitchBody`. Within `SwitchBody` there may be one or more `case` clauses and/or a `default` clause. The `case` clauses designate `CaseExpression`, whose value will be compared to that of `SwitchExpression`, and their syntax is as follows:

```

case CaseExpression:

[other JavaScript statements or additional clauses]

```js

The `switch` statement is usually used to specify a selection of two or more mutually exclusive outcomes based on a specific value. With fewer conditions, it'd be conventional to use an `if...else` construct, but to accommodate more potential conditions, it's simpler to use `switch`:

```

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

```js

Once the `switch` mechanism finds the appropriate `case`, it will execute all code following that `case` statement until the very end of the `switch` statement or until it encounters a `break` statement. A `break` statement is used to *break out* of `SwitchBody` when the desired work is accomplished.

# Breaking and fallthrough

Given that `switch` statements are usually used to execute specific and mutually exclusive pieces of code depending on the value, it is conventional to use `break` between every `case` statement to ensure that only the appropriate code runs for any given value. Sometimes, however, it is desirable to avoid breaking between cases and let the `SwitchBody`code continue to run through multiple `case` statements and beyond. Doing this is known as **fallthrough**:

```

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

```js

Here, you can see that we are employing fallthrough so that a language of either `'German'`, `'Deutsche'`, or `'DE'` will result in the same code running `welcomeMessage = 'Willkommen!'`. And following that, we immediately break to prevent any more of `SwitchBody` from running.

It's unfortunately quite easy to accidentally forget the odd `break;` statement, resulting in accidental fallthrough and a very confused programmer. To avoid this, I'd recommend using a linter that has a rule that warns or gives an error in such cases unless given a specific directive. (We will cover linters in more detail in Chapter 15, *Tools for Cleaner Code*.)

# Returning from a switch directly

When you have a `switch` statement residing in a function, it is sometimes best to simply `return` the intended values instead of having to rely on `break` statements. For example, in `generateWelcomeMessage`, we can simply return the welcome string. There's no need to go through the rigamarole of initializing a variable, assigning it, and breaking between cases:

```

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

```js

Returning directly, in this way, is arguably clearer than breaking within each case, especially if each case's logic is fairly simple. 

# Case blocks

Usually, the code following a `case` or `default` clause will not only occupy a single line. As such, it has become conventional to wrap these statements with a block, so that there is a sense of containment:

```

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

```js

This isn't strictly necessary and doesn't change any functionality, but it does offer more clarity to the reader of our code. It also paves the way for any block-level variables, should we wish to introduce these later. As we know, within a block (delimited with `{` and `}`), we can use `const` and `let` to declare variables that will be scoped only to that block:

```

switch (month) {

case 'December':

case 'January':

case 'February': {

const message = 'In the UK, Spring is coming soon!';

// ...

}

//...

}

```js

Here, we're able to declare specific variables that are scoped to the `February` case only. This is useful if we have a large amount of logic that we'd like to isolate. At this point, however, we should consider abstracting that logic in some other way. Lengthy `switch` statements can be incredibly hard to understand.

# Multivariant conditions

Often, there's a need to express more complex conditions in each `case`, instead of just matching a singular value. If we pass `true` as `SwitchExpression`, then we are free to express custom conditional logic within each `CaseExpression`, as long as each `CaseExpression` evaluates to `true` when successful:

```

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

```js

This pattern allows us to express more multivariate and hybrid conditions. You may usually feel inclined toward multiple `if`/`else`/`if`/`else` statements, but if your logic can be expressed in a `switch` statement, then it may be best to opt for that. As always, you should consider the nature of your problem domain and its logic, and seek to make an informed decision about how you wish to implement your control flow. In some cases, a `switch` statement will only end up being more confusing.

In the next section, we will cover some other approaches you can use to handle complex and lengthy logic that doesn't suit native constructs such as `switch`.

# Handling cyclomatic complexity

**Cyclomatic complexity** is a measure of how many *linearly independent paths* there are through a program's code.

Consider a simple program that contains several conditional checks and function invocations:

```

if (a) {

alpha();

if (b) bravo();

if (c) charlie();

}

if (d) delta();

```js

Even in this misleadingly simple piece of code, nine distinct paths can be taken. So, depending on the values of `a`, `b`, `c`, and `d`, there are nine possible sequences of `alpha`, `bravo`, `charlie`, and `delta` that will run:

*   `alpha()`
*   `alpha()` and `bravo()`
*   `alpha()`, `bravo()`, and `charlie()`
*   `alpha()`, `bravo()`, `charlie()`, and `delta()`
*   `alpha()`, `bravo()`, and ``delta()``
*   `alpha()` and `charlie()`
*   `alpha()`, `charlie()`, and `delta()`
*   `alpha()` and `delta()`
*   `delta()`

A high level of cyclomatic complexity is undesirable. It can lead to the following:

*   **Cognitive burden**: Cyclomatically complex code can be difficult for programmers to understand. Code with many branches is difficult to internalize and hold in our minds and therefore harder to maintain or change.
*   **Unpredictability**: Cyclomatically complex code can be unpredictable, especially if rare situations occur where there is, for example, an unforeseen state transition or underlying change of data. 
*   **Fragility**: Cyclomatically complex code can be fragile in the face of change. Changing one line can have a disproportionate effect on the functionality of many other lines.
*   **Bugginess**: Cyclomatically complex code can cause obscure bugs. If there are a dozen or more code paths within a singular function, then it's possible for a maintainer to not see all of them, leading to regressions.

There are tools that can quantify a code base's cyclomatic complexity. We will cover these in [Chapter 15](https://cdp.packtpub.com/clean_code_in_javascript/wp-admin/post.php?post=415&action=edit#post_508), *Tools for Cleaner Code*. Knowing areas of high cyclomatic complexity can help us to focus on those areas for maintenance and testing.

It's frustratingly easy to end up in a situation where there are so many different conditions and branches within a singular module that nobody can understand what's happening. In addition to using tools to help us to identify areas of high complexity, we can use our own judgment and intuitions. The following are some examples of complexity that we can easily identify and avoid:

*   A function that has more than one `if`/`else`/`if` combination 
*   An `if` statement that has many sub-conditions (many `if` statements within `if` statements)
*   A `switch` statement that has many sub-conditions following each `case` clause
*   Many `case` clauses within a `switch` block (for example, over 20 would be alarming!)

These are not precise cautions but they should give you an idea of what you should watch out for. When we find such complexity, the first thing we should do is to sit back and re-consider our problem domain. Can we describe our logic differently? Can we form new or different abstractions?

Let's explore an example of a piece of code with high cyclomatic complexity and consider how we might simplify it with these questions in mind.

# Simplifying conditional spaghetti

To illustrate too much cyclomatic complexity and how we should approach simplifying it, we're going to be refactoring a piece of code that is responsible for deriving a set of ID numbers and types from a set of licenses:

```

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

```js

This function accepts an array of licenses and then extracts the ID numbers of those licenses (avoiding cases of `null` or `undefined` IDs). We determine the type of license based on characters found within its ID. There are four types of licenses that need to be identified and extracted:

*   `car`: These are of the `c{digits}` form, where digits form a number greater than or equal to 1,000,000
*   `car_old`: These are of the `c{digits}` form, where digits form a number less than 1,000,000
*   `hgv`: These are of the `h{digits}`
*   `motorcycle`: These are of the `m{digits}`

The following is an example of the input and the derived output of the `getIDsFromLicenses` function:

```

getIDsFromLicenses([

{ name: 'Jon Smith', id: 'c32948' },

{ name: 'Marsha Brown' },

{ name: 'Leah Oak', id: 'h109' },

{ name: 'Jim Royle', id: 'c29283928' }

]);

// 输出:

[

{type: "car_old", digits: 32948}

{type: "hgv", digits: 109}

{type: "car", digits: 29283928}

]

```js

As you may have observed, the code we've used to extract the IDs is quite cyclomatically complex. You may consider it perfectly reasonable code, and it arguably is, but it could be simpler still. Our function achieves its results imperatively, using up a lot of syntax to explain *how* it wants to accomplish its task instead of *what* it wants to accomplish. 

To simplify our code, it's first useful to take a fresh look at the problem domain. The task we want to accomplish is to take an input array and, from it, derive a set of license ID types and values. The output array will be an almost **1:1 **mapping from the input array, except for cases where licenses have a falsy `id` property (`null`, in this case). The following is an illustration of our I/O flow:

```

[INPUT LICENSES] ==> (DERIVATION LOGIC) ==> [OUTPUT ID TYPES AND DIGITS]

```js

Looked at abstractly in this way, this seems like the perfect opportunity to use `Array#map`. The `map` method allows us to run a function on every element within an array to derive a new array containing mapped values.

The first thing we'll want to map is the license to its `id`:

```

ids = licenses.map(license => license.id)

```js

We'll want to handle cases where there is no `id`. To do this, we can apply a filter on the derived IDs:

```

ids = ids.filter(id => id != null)

```js

And, in fact, because we know that all valid IDs are truthy, we can simply do a Boolean check by directly passing `Boolean` as our filter function:

```

ids = ids.filter(Boolean)

```js

From this, we'll receive an array of our licenses but only those with a truthy `id` property. Following this, we can consider the next transformation we wish to apply to the data. We'd like to split the `id` value into its constituent parts: we need the initial character of the ID (`id.charAt(0)`), and then we want to extract the remaining characters (the digits) and cast them to the `Number` type (`Number(id.slice(1))`). We can then pass these parts to another function, which will be responsible for extracting the correct ID fields (`type` and `digits`) from this information:

```

ids = ids.map(id => getIDFields(

id.charAt(0),

Number(id.slice(1))

));

```js

The `getIDFields` function will need to determine the type from the individual character and digits for the ID, returning an object of the `{ type, digits }` form:

```

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

```js

Since we've abstracted this part our logic away to an individual function, we can independently observe and test its behavior:

```

getIDFields('c', 1000); // => { type: "car_old", digits: 1000 }

getIDFields('c', 2000000); // => { type: "car", digits: 1000 }

getIDFields('h', 1000); // => { type: "hgv", digits: 1000 }

getIDFields('i', 1000); // => { type: "motorcycle", digits: 1000 }

```js

Tying everything together, we end up with a new implementation of `getIDsFromLicenses` that looks like this:

```

function getIDsFromLicenses(licenses) {

return licenses

.map(license => license.id)

.filter(Boolean)

.map(id => getIDFields(

id.charAt(0),

Number(id.slice(1))

))

}

```js

What we have achieved here is a significant reduction in the amount of cyclomatic complexity that our fellow programmers will need to contend with. We are utilizing `Array#map` and `Array#filter` to abstract away both decision-making and iteration logic. This means we end up with an implementation that is far more *declarative*.

You may notice, as well, that we have extracted repeated logic and generalized it. For example, in our initial implementation, we were implementing many calls to discover the first character of the ID (for example, `license.id.indexOf('m') === 0`). Our new implementation generalizes this by mapping to a data structure that already includes the first character as a distinct value that we can then pass through to `getIDFields` to get the relevant `type` and `digits` for that ID.

To summarize, our general refactoring approach has involved the following considerations:

*   We've considered the problem domain with a fresh perspective
*   We've considered whether there is a common functional or declarative idiom for our I/O
*   We've considered whether individual logic can be abstracted away or separated

Our code is now easier to comprehend, and hence easier to maintain and debug. It'll likely also be more reliable and stable since its individual units can be more simply tested and can hence avoid regressions in the future. There is, naturally, the potential for a slight performance decrease due to the increased usage of higher abstracted declarative idioms and functions over imperative code, but this is an incredibly marginal difference and, in the vast majority of situations, is worth implementing for the significant benefits that the refactoring produces in terms of maintainability and reliability.

# Asynchronous control flow

Most of the constructs we've looked at so far are used for synchronous code, where statements are evaluated sequentially, with each line completing before the next one begins:

```

const someValue = getSomeValue();

doSomethingWithTheValue(someValue);

```js

Code like this is straightforward. We intuitively understand that these two lines of code will run one after the other. There is also an assumption that neither of these lines will take very long to execute, probably taking no more than a few micro- or milliseconds. 

But what happens if we wish to bind to a user Event or fetch some remote data? These are things that take time and will only complete when some future Event occurs. In a less kind universe, there would be no way to deal with such scenarios other than simply waiting for them to complete and then continuing the execution of our program:

```

fetchSomeData();

processFetchedData();

```js

In this unkind universe, `fetchSomeData()` would be a *blocking* function call, so named because it would block the execution of all other code until it finally completes. This means that we wouldn't be able to carry out any other vital tasks, and our application would essentially be at a standstill state until the task is completed, negatively affecting the user experience.

Thankfully, JavaScript gives us a nicer universe than this—one in which we can initialize a task, such as fetching data, and then continue on with the rest of our program while that task is running. Such tasks are named *asynchronous* because they occur and complete non-synchronously, at a later time than *now*. When they do finally complete, JavaScript can helpfully notify us of this fact, calling whatever code depends upon the completion of that task.

# The Event Loop

To accomplish this, JavaScript maintains a single-threaded *Event Loop*. When the *Event Loop* kicks off, it'll run our program. Following the execution of a piece of code (such as that which initiates our program), the *Event Loop* will await messages (or Events) indicating that something has occurred (for example, a network request has completed or a browser UI event has occurred). When it receives a message, it will then execute any code that is depending upon or listening for that Event. The *Event Loop* will, again, run that code to completion before continuing to await other messages. This process repeats infinitely until the JavaScript program is halted (for example, by closing a tab in a browser).

The fact that the *Event Loop* will always run a given piece of code to its completion means that any long-running or *blocking* code will prevent any other code from executing until it has completed. Some older browser API methods such as `alert()` and `prompt()` are examples of blocking functions that you may encounter. Calling these will effectively block any further execution of your JavaScript program:

```

alert('Hello!');

console.log('The alert has been dismissed by the user');

```js

Here, `console.log()` will not be evaluated until the alert dialog is dismissed by the user. This could be milliseconds, minutes, or even hours. During this period, our JavaScript program is halted, unable to continue. Its *Event Loop* may be receiving Events but it will not run the code associated with those Events until `alert()` finally completes.

# Native asynchronous APIs

Nowadays, it's normal to expect APIs within a browser and server to provide non-blocking asynchronous ways to call native mechanisms. Common examples of such APIs include the following:

*   The DOM Event API, enabling code such as `window.addEventListener('click', callback)`
*   The Node.js file API, enabling code such as `fs.readFile(path, callback)`
*   The Browser Fetch API, enabling code such as `fetch().then(callback)`

All such interfaces share something in common: they all provide a way to somehow listen for their completion. Usually, this is achieved via a provided callback (a function). This callback will be called at some later point when the task has completed. Similarly, some native APIs return promises, which enable a richer mechanism of asynchronous control flow, but fundamentally still rely on passing callbacks via the Promise API. Additionally, ECMAScript 2017 introduced the concept of asynchronous functions (`async function() {}`) and the `await` keyword, which finally provided language support for promises, meaning that the completion of asynchronous work no longer requires callbacks.

Let's explore each of these asynchronous of control flow mechanisms individually.

# Callbacks

A callback is a conventional approach to providing a way to hook into asynchronous tasks. A callback is simply a function that is passed to another function and is expected to be called at some later point, possibly immediately, possibility soon, and possibly never. Consider the following `requestData` function:

```

function requestData(path, callback) {

// (Implementation of requestData)

}

```js

As you can see, it accepts a callback as its second argument. When calling `requestData`, the callback will typically be anonymously passed inline, like so:

```

requestData('/data/123', (response) => { /* ... */ });

```js

It is, of course, totally fine to have previously declared the callback, and doing so can aid comprehensibility as now the reader of your code will have an inkling as to when a callback might be invoked. Observe here how we're calling our `onResponse` callback to make clear that it is expected to be called upon the response becoming available (when it completes):

```

function onResponse(response) {

// Do something with the response...

}

requestData('/data/123', onResponse);

```js

Similarly, in complex APIs with multiple asynchronous state changes, it's common to see named callbacks registered in bulk, via an *object literal*:

```

createDropdownComponent({

onOpen() {},

onSelect() {},

onClose() {},

onHover() {} // etc.

});

```js

A callback will typically be passed arguments that indicate some important state that has been determined from the asynchronous work. For example, the Node.js `readFile` function invokes its callback with two arguments, a (possibly null) error and the (possibly null) data from the file itself:

```

fs.readFile('/path/to/file', (error, data) => {

if (error) {

// Handle the error!

} else {

// Handle the data! (No error has occurred!)

}

});

```js

The function you pass a callback to is entirely in control of when your callback is invoked, how it is invoked, and what data is passed along with that invocation. This is why sometimes callbacks are spoken about as an *inversion of control*. Normally, you are in control of what functions you call, but when using callbacks, the control is inverted so that you are relying on another function or abstraction to (at some point) call your callback in the expected manner.

*Callback hell* is the name given to the undesirable proliferation of multiple nested callbacks within a piece of code, usually done to carry out a series of asynchronous tasks that each rely on another previous asynchronous task. Here is an example of such a situation:

```

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

```js

Here, you can see we have three different callbacks, all appearing in one hierarchy of scopes. We await the response of `/data/current-user`, then we optionally make a request to `twitterFeedURL`, and then, upon the rendering of the twitter feed (`renderTwitterFeed()`), we finally log a `"twitterFeedRender"` Event. That final log depends on two previous asynchronous tasks completing and so is (seemingly unavoidably) nested quite deeply.

We can observe that this deeply nested piece of code is at the peak of a kind of *horizontal pyramid* of indentation. This is a common trait of *callback hell*, and as such, you can use the existence of these *horizontal pyramids*as something to watch out for. Not all deep indentations will be due to callbacks, of course, but it's usually high on the list of suspects:

![](img/9fabad56-8ede-4573-9fcb-0467a4188f66.png)

To avoid the *callback hell *indicated by the *horizontal pyramid*, we should consider re-thinking and potentially re-abstracting our code. In the preceding case, logging a Twitter feed render Event, we could, for example, have a generalized function for *getting and rendering Twitter feed data*. This would simplify the top-level of our program:

```

requestData('/data/current-user', (userData) => {

if (userData.preferences.twitterEnabled) {

renderTwitterForUser(userData);

}

});

```js

Observe how we have shortened our *horizontal pyramid* here. We are now free to implement `renderTwitterForUser` as we wish and import it as a dependency. Even though its implementation may involve its own callbacks, it is still a reduction in overall complexity for the programmer as it abstracts away half of the *pyramid* to a neatly separated abstraction. Most *callback hell* scenarios can be solved with a similar approach to re-designing and abstraction. This was a simple scenario, though. With more intertwined asynchronous tasks, it may be useful to use other mechanisms of asynchronous control flow.

# Event subscribing/emitting

JavaScript is a language that feels right at home when subscribing to and emitting Events. Events are incredibly common in most JavaScript programs, whether dealing with user-derived Events within the browser or server-side Events in Node.js.

There are various names used for operations relating to Events in JavaScript, so it's useful to know all of these names upfront so we're not confused when encountering them. An event is an occurrence in time that will result in the invocation of any callbacks that have been subscribed for that Event. Subscribing to an Event has many names, which all effectively mean the same thing: *subscribing*, *registering*, *listening*, *binding*, and so on. When the Event occurs, the subscribed callback is invoked. This, as well, has many names: *invoking*, *calling*, *emitting*, *firing*, or *triggering*. The actual function that is called can also have various names: *function*, *callback*, *listener*, or *handler*.

At its core, any abstraction that supports Events will usually do so by storing callbacks to be called later, keyed with specific Event names. We can imagine that a DOM element might store its Event listeners in a structure like the following:

```

{

"click": [Function, Function, Function],

"mouseover": [Function, Function],

"mouseout": [Function]

}

```js

Any Event-supporting abstraction will simply store a series of callbacks to be called later. As such, when subscribing to an Event, you will need to provide both the callback you wish it to call and the Event name that it will be tied to. In the DOM, we would do this like so:

```

document,body.addEventListener('mousemove', e => {

e; // => the Event object

});

```js

Here, we see that an `Event` object is passed to the callback. This is idiomatically named `e` or `evt` for succinctness. Most abstractions that provide an Events API will pass specific Event-related information to the callback. This may be in the form of a singular `Event` object or several arguments.

It's important to note that there truly is no single standard for Events although there are conventions that have emerged. Typically there will always be a method used to register or subscribe to an Event and then another to remove that subscription. The following is an example of using the Node.js Event-Emitter API, which is supported by the native HTTP module:

```

const server = http.createServer(...);

function onConnect(req, cltSocket, head) {

// Connect to an origin server...

}

// Subscribe

server.on('connect', onConnect);

// Unsubscribe

server.off('connect', onConnect);

```js

Here, you can see that the `on()` method is used to subscribe to Events, and the `off()` method is used to unsubscribe. Most Events APIs have similar event registration and de-registration methods although they may implement them in different ways. If you're crafting your own *Events* implementation, then it's advisable to ensure that you're providing a familiar set of methods and abstractions. To do this, take inspiration from either the native DOM Events interface or the Node.js **Event Emitter**. This will ensure that your Events implementation does not surprise or horrify other programmers too much.

Even though an Events API is essentially just a series of callbacks stored and invoked at specific times, there are still challenges in crafting it well. Amongst them are the following:

*   Ensuring the order of invocation when a singular Event fires
*   Handling cases where Events are emitted while other Events are mid-emission
*   Handling cases where Events can be entirely canceled or removed per callback
*   Handling cases where Events can be bubbled, propagated, or delegated (this is usually a DOM challenge)

*Propagation*, *bubbling*, and *delegation* are terms related to firing Events within a hierarchical structure. In the DOM, since `<div>` may exist within `<body>`, the Events API has prescribed that, if the user clicks on `<div>`, the emitted Event will propagate or *bubble* upward, first triggering any `click` listeners on `<div>` and then `<body>`. Delegation is intentional listening at a higher level of hierarchy, for example, listening at the `<body>` level and then reacting in a certain way, depending on what the Event object tells you about the Event's `target` node.

Events provide more possibilities than a simple callback. Since they allow several different Events to be listened for, and the same Event to be listened for several times, any consuming code has far more flexibility in how it constructs its asynchronous control flow. An object that has an Events interface can be passed around throughout a code base and may be subscribed to many times, potentially. The nature of distinct Events, as well, means that different asynchronous concepts or occurrences are usefully kept separated so that a fellow programmer can easily tell which action will be taken in specific circumstances:

```

const dropdown = new DropDown();

dropdown.on('select', () => { /*...*/ });

dropdown.on('deselect', () => { /*...*/ });

dropdown.on('hover', () => { /*...*/ });

```js

This type of transparent separation helps to encode expectations within the mind of the programmer. It's simple to discern which function will be called in each case. Compare this to a generalized *something happened* `event` with an internal `switch` statement:

```

// Less transparent & more burdensome:

dropdown.on('action', event => {

switch (event.action) {

case 'select': /*...*/; break;

case 'deselect': /*...*/; break;

// ...

}

});

```js

Well-implemented Events provide a good semantic separation between conceptually different Events and, therefore, provide the programmer with a predictable series of asynchronous actions that they can reason about easily.

# Promises

A *Promise* is an abstraction that surrounds the concept of an eventual value. It's easiest to think of a *Promise* as a simple object that will, at some point, contain a value. A Promise provides an interface via which you can pass callbacks to wait for either the eventually-fulfilled value or an error.

At any given time a *Promise* will have a certain state:

*   **Pending**: The *Promise* is awaiting its resolution (the asynchronous task has not yet completed).
*   **Settled**: The *Promise* is no longer pending and has either been fulfilled or rejected:
    *   **Fulfilled**: The *Promise* has been successful and now has a value
    *   **Rejected**: The *Promise* has failed with an error

Promises can be constructed via the *Promise* constructor, by passing a singular function argument (called an *executor*) that calls either a `resolve` or `reject` function to indicate either a settled value or an error, respectively:

```

const answerToEverything = new Promise((resolve, reject) => {

setTimeout(() => {

resolve(42);

}, 1000);

});

```js

The instantiated `Promise` has the following methods available so that we can access its changed state (when it moves from *pending* to either *fulfilled* or *rejected):*

*   `then(onFulfilled[, onRejected])`: This will append a fulfillment callback to the *Promise* and optionally a *rejection* callback. It will return a new *Promise* object, which will resolve to the return value of the called fulfillment or rejection handler, or will resolve as per the original *Promise* if there is no handler.
*   `catch(onRejected)`: This will append a *rejection* callback to the *Promise* and will return a new *Promise* that will resolve to either the return value of the callback or (if the original *Promise* succeeds) its fulfillment value.
*   `finally(onFinally)`: This will append a handler to the *Promise*, which will be called when the *Promise* is resolved, regardless of whether the resolution is a fulfillment or a rejection.

We can access the eventually resolved value of `answerToEverything` by passing a callback to its `then` method:

```

answerToEverything.then(answer => {

answer; // => 42

});

```js

We can illustrate the exact nature of a *Promise* by exploring the native Fetch API, supported by most modern browsers:

```

const promiseOfData = fetch('/some/data?foo=bar');

```js

The `fetch` function returns a *Promise* that we assign to our variable, `promiseOfData`. We can then hook into the request's eventual success (or failure) like so:

```

const promiseOfData = fetch('/some/data');

promiseOfData.then(

response => {

response; // The "fulfilled" Response

},

error => {

error; // The "rejected" Error

}

);

```js

It may appear as though promises are just a slightly more verbose abstraction than callbacks. Indeed, in the simplest case, you might just pass a *fulfillment* callback and a *rejection* callback. This, arguably, does not provide us with anything more useful than the original callback approach. But promises can be so much more than this.

Since a *Promise* is just a regular object, it can be passed around your program just like any other value, meaning that the eventual resolution of a task no longer needs to be tied to code at the call site of the original task. Additionally, the fact that each `then`, `catch`, or `finally` call returns a *Promise* of its own, we can chain together any number of either synchronous or asynchronous tasks that rely on some original fulfillment.

In the case of `fetch()`, for example, the fulfilled `Response` object provides a `json()` method, which itself completes asynchronously and returns a *Promise*. Hence, to get the actual JSON data from a given resource, you would have to do the following:

```

fetch('/data/users')

.then(response => response.json())

.then(jsonDataOfUsers => {

jsonDataOfUsers; // the JSON data that we got from response.json()

});

```js

Chaining together `then` calls is a popular pattern used to derive a new value from some prior value. Given the response, we wish to compute the JSON, and given the JSON, we may wish to compute something else:

```

fetch('/data/users')

.then(response => response.json())

.then(users => users.map(user => user.forename))

.then(userForenames => userForenames.sort());

```js

Here, we are using multiple `then` calls to compute the sorted forenames of our users. There are, in fact, four distinct promises being created here, as foll:

```

const promiseA = fetch('/data/users');

const promiseB = promiseA.then(response => response.json());

const promiseC = promiseB.then(users => users.map(user => user.forename))

const promiseD = promiseC.then(userForenames => userForenames.sort());

promiseA === promiseB; // => false

promiseB === promiseC; // => false

promiseC === promiseD; // => false

```js

Each *Promise* will only ever resolve to a single value. Once it's been either *fulfilled* or *rejected*, no other value can take its place. But as we see here, we can freely derive a new *Promise* from an original *Promise* by simply registering a callback via `then`, `catch`, or `finally`. The nature of only resolving once and of returning new derived promises means that we can compose promises together in a number of useful ways. In our example, we could derive two promises from our `users` data *Promise*: one that collects the forenames of users and another that collects their surnames:

```

const users = fetch('/data/users').then(r => r.json());

const forenames = users.then(users => users.map(user => user.forename));

const surnames = users.then(users => users.map(user => user.surname));

```js

We can then freely pass around these `forenames` and `surnames` promises, and any consuming code can do what it wants with them. For example, we may have a DOM element that we'd like to populate with the forenames when they are eventually available:

```

function createForenamesComponent(forenamesPromise) {

const div = document.createElement('div');

function render(forenames) {

div.textContent = forenames ? forenames.join(', ') : 'Loading...';

}

render(null); // 初始渲染

forenamesPromise.then(forenames => {

// 当我们收到名字时，我们想要渲染它们：

render(forenames);

});

return div;

}

```js

This `createForenamesComponent` function accepts the `forenames` *Promise* as an argument and then returns a `<div>` element. As you can see, we have called `render()` initially with `null`, which populates the DIV element with the `"loading..."` text. Once the *Promise* is fulfilled, we then re-render with the newly populated forenames.

The ability to pass around promises in this manner makes them far more flexible than callbacks, and similar in spirit to an object that implements an Events API. However, with all of these mechanisms, it is necessary to create and pass around functions so that you can listen for future Events and then act on them. If you have a significant amount of asynchronous logic to express, this can be a real struggle. The control flow of a program littered with callbacks, Events, and promises can be unclear, even to those well accustomed to a particular code base. Even a small number of independently asynchronous Events can create a large variety of *states* throughout your application. A programmer can become very confused, as a result; the confusion relates to *what* is happening *when*. 

The *state* of your program is determined at runtime. When a value or piece of data changes, no matter how small, it will be considered a *change of state*. *State* is typically expressed in terms of outputs from the program, such as a GUI or a CLI can be also be held internally and manifest in a later observed output.

To avoid confusion, it's best to implement any timing-related code as transparently as possible, so that there is no room for misunderstanding. The following is an example of code that may lead to misunderstanding:

```

userInfoLoader.init();

appStartup().then(() => {

const userID = userInfoLoader.data.id;

const userName = userInfoLoader.data.name;

renderApplication(userID, userName);

});

```js

This code seems to assume that the *Promise* returned by `appStartup()` will always fulfill after `userInfoLoader` has completed its work. Perhaps the author of this code happens to know that the `appStartup()` logic will always complete after `userInfoLoader`. Perhaps that is a certainty. But for us, reading this code for the first time, we have no confidence that `userInfoLoader.data` will be populated by the time `appStartup()` is fulfilled. It would be better to make the timing more transparent by, for example, returning a *Promise* from `userInfoLoader.init()` and then carrying out `appStartup()` on the explicit fulfillment of that *Promise:*

```

userInfoLoader.init()

.then(() => appStartup())

.then(() => {

const userID = userInfoLoader.data.id;

const userName = userInfoLoader.data.name;

renderApplication(userID, userName);

});

```js

Here, we are arranging our code so that it is obvious what actions are dependent on what other actions and in what order the actions will occur. Using promises by themselves, just like any other asynchronous control flow abstraction, does not guarantee that your code will be easily comprehensible. It's important to always consider the perspective of your fellow programmers and the temporal assumptions that they'll make. Next, we will explore a newer addition to JavaScript that gives us native linguistic support for asynchronous code: you'll see how these additions enable us to write asynchronous code that is clearer in terms of *what* is happening *when*.

# async and await

The ECMAScript 2017 specification introduced new concepts to the JavaScript language in the form of the `async` and `await` keywords. The `async` keyword is used to designate a function as asynchronous:

```

async function getNumber() {

return 42;

}

```js

Doing this, effectively, wraps whatever the function returns in `Promise` (if it is not already `Promise`)*.* So, if we attempt to call this function we will receive `Promise`:

```

getNumber() instanceof Promise; // => true

```js

As we've learned, we can subscribe to the fulfillment of `Promise` by using its `then` method:

```

getNumber().then(number => {

number; // => 42

});

```js

In concert with async functions that return *Promises,* we also have an `await` keyword. This enables us to wait for the fulfillment (or rejection) of the `Promise` simply by passing it to the right side of `await`. This may, for example, be a `Promise` returned from an `async` function call:

```

await someAsyncFunction();

```js

Or it may be a *Promise* designated inline, like so:

```

const n = await new Promise(fulfill => fulfill(123));

n; // => 123

```js

As you can see, the `await` keyword will wait for its *Promise* to resolve and thereby prevents any following lines from executing until that occurs.

The following is another example—a `setupFeed` async function that awaits both `fetch()` and `response.json()`:

```

async function setupFeed() {

const response = await fetch('/data');

const json = await response.json();

console.log(json);

}

```js

It's important to note that the `await` keyword does not block like `alert()` or `prompt()`. Instead, it simply pauses the execution of the asynchronous function, freeing up the *Event Loop* to continue with other work, and then, when its *Promise* resolves, it will continue execution where it left off. Remember, `await` is only syntactic *sugar* over functionality that we can already achieve. If we wanted to implement our `setupFeed` function without `async`/`await`, we could easily do that by reverting to our old pattern of passing callbacks to `Promise#then`:

```

function setupFeed() {

fetch('/data').then(response => {

return response.json()

}).then(json => {

console.log(json);

});

}

```js

Observe how the code is slightly clunkier and more congested when we don't use `await`. Using `await` in concert with asynchronous functions gives us the same satisfyingly linear and procedural appearance as regular synchronous code. This can vastly simplify an otherwise complicated asynchronous control flow, making it clearer to our fellow programmer *what* is happening *when*. 

The await keyword is also available for use within the `for...of` iteration construct. Doing so will await each value iterated over. If, during iteration, any encountered value is a *Promise,* the iteration will not continue until that *Promise* has been resolved:

```

const allData = [

fetch('/data/1').then(r => r.json()),

fetch('/data/2').then(r => r.json()),

fetch('/data/3').then(r => r.json())

];

for await (const data of allData) {

console.log(data);

}

// 记录来自/data/1、/data/2 和/data/3 的数据

```

没有*Promises*或`await`和`async`，表达这种异步过程不仅需要更多的代码，还需要更多的时间来理解。这些构造和抽象的美妙之处在于它们使我们能够忽略异步操作的实现细节，从而使我们能够纯粹地专注于表达我们的问题领域。随着我们在本书中的进展，我们将进一步探索这种抽象精神，因为我们将处理一些更大更棘手的问题领域。

# 总结

在本章中，我们已经完成了对 JavaScript 语言的探索，讨论了命令式和声明式语法之间的区别，探讨了如何清晰地控制流程，并学习了如何处理同步和异步上下文中的圈复杂度情况。这涉及对语言中所有迭代和条件构造的深入研究，对它们的使用进行指导，并警告反模式。

在下一章中，我们将把我们对 JavaScript 语言积累的所有知识与对真实世界设计模式和范式的探索相结合，这将帮助我们构建清晰的抽象和架构。
