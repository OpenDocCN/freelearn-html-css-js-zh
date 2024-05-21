# 第七章：动态类型化

在上一章中，我们探讨了 JavaScript 的内置值和类型，并涉及了在使用它们时涉及的一些挑战。接下来自然的步骤是探索 JavaScript 的动态系统在现实世界中是如何发挥作用的。由于 JavaScript 是一种动态类型的语言，代码中的变量在所引用的值的类型方面没有限制。这给清洁的编码者带来了巨大的挑战。由于我们的类型不确定，我们的代码可能以意想不到的方式中断，并且可能变得非常脆弱。这种脆弱性可以很简单地解释为想象一个嵌入在字符串中的数值：

```js
const possiblyNumeric = '203.45';
```

在这里，我们可以看到该值是数值，但它已被包装在一个字符串文字中，因此在 JavaScript 看来，它只是一个普通的字符串。但由于 JavaScript 是动态的，我们可以自由地将这个值传递给任何函数，甚至是一个期望一个数字的函数：

```js
setWidth('203.45');

function setWidth(width) {
  width += 20;       // Add margins
  applyWidth(width); // Apply the width
}
```

该函数通过`+=`运算符向数字添加了一个边距值。正如我们将在本章后面学到的那样，这个运算符是操作`a = a + b`的别名，而这里的`+`运算符，在任一操作数为`String`类型的情况下，将简单地将这两个字符串连接在一起。有趣的是，这个简单而无辜的实现细节是世界各地在不同时间发生的数百万次令人筋疲力尽的调试会话的关键。幸运的是，了解这个运算符及其确切的行为将为你节省无数个小时的痛苦和筋疲力尽，并且会牢固地铭记在你的脑海中，即避免我们已经陷入的`possiblyNumeric`值的陷阱的代码的重要性。

在本章中，我们将涵盖以下主题：

+   检测

+   转换、强制转换和转型

能够更轻松地处理我们的类型的第一个关键步骤是学习检测，即能够以最简单的方式辨别你正在处理的类型或类型的技能。

# 检测

检测是指确定值的类型的实践。通常，这将是为了使用确定的类型来执行特定的行为，比如回退到默认值或在误用的情况下抛出错误。

由于 JavaScript 的动态特性，检测类型是一种重要的实践，通常可以帮助其他程序员。如果你可以在某人错误地使用接口时有用地抛出错误或警告，那么对于他们来说，这意味着开发流程更加流畅和迅速。如果你可以有用地用智能默认值填充`undefined`、`null`或空值，那么它将允许你提供一个更无缝和直观的接口。

不幸的是，由于 JavaScript 中的遗留问题和设计中的一些选择，检测类型可能是具有挑战性的。使用了许多不被认为是最佳实践的不同方法。我们将在本节中讨论所有这些实践。然而，首先值得讨论一个关于检测的基本问题：**你究竟想要检测什么**？

我们经常认为我们需要特定的类型才能执行某些操作，但由于 JavaScript 的动态特性，我们可能并不需要这样做。事实上，这样做可能导致我们创建不必要的限制性或僵化的代码。

考虑一个接受`people`对象数组的函数，如下所示：

```js
registerPeopleForMarathon([
  new Person({ id: 1, name: 'Marcus Wu' }),
  new Person({ id: 2, name: 'Susan Smith' }),
  new Person({ id: 3, name: 'Sofia Polat' })
]);
```

在我们的`registerPeopleForMarathon`中，我们可能会想要实现某种检查，以确保传递的参数是预期的类型和结构：

```js
function registerPeopleForMarathon(people) {
  if (Array.isArray(people)) {
    throw new Error('People is not an array');
  }
  for (let person in people) {
    if (!(person instanceof Person)) {
      throw new Error('Each person should be an instance of Person');
    }
    registerForMarathon(person.id, person.name);
  }
}
```

这些检查有必要吗？你可能倾向于说有，因为它们确保我们的代码对潜在的错误情况具有弹性（或防御性），因此更可靠。但是如果我们仔细考虑一下，我们的这些检查都不是必要的，以确保我们寻求的可靠性。我们检查的意图，大概是为了防止错误类型或结构传递给我们的函数时产生下游错误，但是如果我们仔细观察前面的代码，我们担心的类型并没有下游错误的风险。

我们进行的第一个检查是`Array.isArray(people)`，以确定`people`值是否确实是一个数组。我们这样做，表面上是为了安全地遍历数组。但是，正如我们在前一章中发现的那样，`for...of`迭代风格并不依赖于`of {...}`值是一个数组。它只关心值是否可迭代。一个例子如下：

```js
function* marathonPeopleGenerator() {
  yield new Person({ id: 1, name: 'Marcus Wu' });
  yield new Person({ id: 2, name: 'Susan Smith' });
  yield new Person({ id: 3, name: 'Sofia Polat' });
}

for (let person of marathonPeopleGenerator()) {
 console.log(person.name);
}

// Logged => "Marcus Wu"
// Logged => "Susan Smith"
// Logged => "Sofia Polat"
```

在这里，我们使用生成器作为我们的可迭代对象。这将像数组一样在`for...of`中被迭代，因此，从技术上讲，我们可以说我们的`registerPeopleForMarathon`函数应该接受这样的值：

```js
// Should we allow this?
registerPeopleForMarathon(
  marathonPeopleGenerator()
);
```

到目前为止，我们进行的检查会拒绝这个值，因为它不是一个数组。这有意义吗？你还记得抽象原则以及我们应该关注接口而不是实现吗？从这个角度来看，可以说我们的`registerPeopleForMarathon`函数不需要知道传递值的类型的实现细节。它只关心值是否按照它的需求执行。在这种情况下，它需要通过`for...of`循环遍历值，因此任何可迭代对象都是合适的。为了检查可迭代性，我们可以使用这样的辅助函数：

```js
function isIterable(obj) {
  return obj != null &&
 typeof obj[Symbol.iterator] === 'function';
}

isIterable([1, 2, 3]); // => true
isIterable(marathonPeopleGenerator()); // => true
```

另外，要考虑的是，我们目前正在检查所有`person`值是否是`Person`构造函数的实例：

```js
// ...
if (!(person instanceof Person)) {
  throw new Error('Each person should be an instance of Person');
}
```

我们是否有必要以这种方式明确检查实例？相反，我们是否可以简单地检查我们希望访问的属性？也许我们需要断言的是属性不是假值（空字符串、null、undefined、零等）：

```js
// ...
if (!person || !person.name || !person.id) {
  throw new Error('Each person should have a name and id');
}
```

这个检查可能更符合我们真正的需求。这样的检查通常被称为**鸭子类型**，即*如果它走起来像鸭子，叫起来像鸭子，那么它一定是鸭子*。我们并不总是需要检查特定类型；我们可以检查我们真正依赖的属性、方法和特征。通过这样做，我们创建的代码更加灵活。

我们的新检查，当集成到我们的函数中时，会看起来像这样：

```js
function registerPeopleForMarathon(people) {
  if (isIterable(people)) {
    throw new Error('People is not iterable');
  }
  for (let person in people) {
    if (!person || !person.name || !person.id) {
      throw new Error('Each person should have a name and id');
    }
    registerForMarathon(person.id, person.name);
  }
}
```

通过使用更灵活的`isIterable`检查，并在我们的`person`对象上使用*鸭子类型*，我们的`registerPeopleForMarathon`函数现在可以被传递；例如，在这里，我们有一个生成器产生普通对象：

```js
function* marathonPeopleGenerator() {
  yield { id: 1, name: 'Marcus Wu' };
  yield { id: 2, name: 'Susan Smith' };
  yield { id: 3, name: 'Sofia Polat' };
}

registerPeopleForMarathon(
  marathonPeopleGenerator()
);
```

如果我们一直坚持严格的类型检查，这种灵活性是不可能的。更严格的检查通常会创建更严格的代码，并且不必要地限制灵活性。然而，这里需要取得平衡。我们不能无限制地灵活。甚至可能严格的类型检查提供的严谨性和确定性能够确保长期更清晰的代码。但相反的情况也可能成立。灵活性与严谨性的平衡是你应该不断考虑的。

一般来说，接口的期望应该尽可能接近实现的需求。也就是说，除非检查确实能够防止我们的实现中出现错误，否则我们不应该执行检测或其他检查。过度检查可能看起来更安全，但可能只意味着未来的需求和用例更难以适应。

现在我们已经解决了为什么我们要检测事物并且暴露了一些用例的问题，我们可以开始学习 JavaScript 提供给我们的检测技术。我们将从`typeof`运算符开始。

# typeof 运算符

当你第一次尝试在 JavaScript 中检测类型时，你通常会接触到的第一件事是`typeof`运算符：

```js
typeof 1; // => number
```

`typeof`运算符接受一个操作数，位于其右侧，并将根据传递的值之一求值为八种可能的字符串值之一：

```js
typeof 1; // => "number"
typeof ''; // => "string"
typeof {}; // => "object"
typeof function(){}; // => "function"
typeof undefined; // => "undefined"
typeof Symbol(); // => "symbol"
typeof 0n; // => "bigint"
typeof true; // => boolean
```

如果你的操作数是一个没有绑定的标识符，也就是一个未声明的变量，那么`typeof`将有用地返回`"undefined"`，而不是像对该变量的任何其他引用一样抛出`ReferenceError`：

```js
typeof somethingNotYetDeclared; // => "undefined"
```

`typeof`是 JavaScript 语言中唯一执行此操作的运算符。如果尚未声明该值，那么任何其他运算符和引用值的方式都会抛出错误。

除了检测未声明的变量外，`typeof`在确定原始类型时真的只有用处——即使这太宽泛了，因为并非所有原始类型都是可检测的。例如，当传递给`typeof`时，`null`值将求值为一个相当无用的`"object"`：

```js
typeof null; // => "object"
```

这是 JavaScript 语言的一个不幸且无法修复的遗留问题。它可能永远不会被修复。要检查`null`，最好明确检查值本身：

```js
let someValue = null;
someValue === null; // => true
```

`typeof`运算符在不是函数的不同类型的对象之间没有区别，除了函数。JavaScript 中的所有非函数对象都会返回简单的`"object"`：

```js
typeof [];         // => "object"
typeof RegExp(''); // => "object"
typeof {};         // => "object"
```

所有函数，无论是通过类定义、方法定义还是普通函数表达式声明的，都将求值为`"function"`：

```js
typeof () => {};          // => "function"
typeof function() {};     // => "function"
typeof class {};          // => "function"
typeof ({ foo(){} }).foo; // => "function"
```

如果`typeof class {}`求值为`"function"`让你感到困惑，那么请考虑我们所学到的，所有类都只是具有准备好的原型的构造函数（这将稍后确定任何生成实例的`[[Prototype]]`）。它们没有什么特别之处。类不是 JavaScript 中的独特类型或实体。

在比较`typeof`的结果与给定字符串时，我们可以使用严格相等(`===`)或抽象相等(`==`)运算符。由于`typeof`始终返回一个字符串，我们不必担心任何差异，所以你可以选择使用严格相等还是抽象相等检查。从技术上讲，这两种方法都可以：

```js
if (typeof 123 == 'number') {...}
if (typeof 123 === 'number') {...}
```

严格相等和抽象相等运算符（双等号和三等号）的行为略有不同，尽管当运算符两侧的值是相同类型时，它们的行为是相同的。请跳转到*运算符*部分，了解它们的区别。一般来说，最好优先使用`===`而不是`==`。

总之，`typeof`运算符只是一个晴天朋友。我们不能在所有情况下依赖它。有时，我们需要使用其他类型检测技术。

# 类型检测技术

考虑到`typeof`运算符对于检测多种类型的不适用性，特别是对象，我们必须依赖于许多不同的方法，具体取决于我们想要检查的确切内容。有时，我们可能想要检测特征而不是类型，例如，一个对象是否是构造函数的实例，或者它只是一个普通对象。在本节中，我们将探讨一些常见的检测需求及其解决方案。

# 检测布尔值

布尔值检测起来非常简单。`typeof`运算符对`true`和`false`的值正确地求值为`"boolean"`：

```js
typeof true;  // => "boolean"
typeof false; // => "boolean"
```

不过，我们很少会想要这样做。通常，当你接收到一个`Boolean`值时，你最感兴趣的是检查它的真实性而不是它的类型。

当将布尔值放置在布尔上下文中时，比如条件语句，我们隐含地依赖于它的真实性或虚假性。例如，看下面的检查：

```js
function process(isEnabled) {
  if (isEnabled) {
    // ... do things
  }
}
```

这个检查并不能确定`isEnabled`值是否真正是布尔值。它只是检查它是否评估为真值。`isEnabled`可能的所有可能值是什么？是否有所有这些真值的列表？这些值几乎是无限的，所以没有列表。我们只能说关于真值的是它们不是假值。而且我们知道，只有七个假值。如果我们希望观察特定值的真假，我们总是可以通过将`Boolean`构造函数作为函数调用来转换为`Boolean`：

```js
Boolean(true); // => true
Boolean(1); // => true
Boolean(42); // => true
Boolean([]); // => true
Boolean('False'); // => true
Boolean(0.0001); // => true
```

在大多数情况下，对`Boolean`的隐式强制转换是足够的，不会对我们造成影响，但是如果我们希望绝对确定一个值既是`Boolean`又是特定的`true`或`false`，我们可以使用严格相等运算符进行比较，如下所示：

```js
if (isEnabled === true) {...}
if (isEnabled === false) {...}
```

由于 JavaScript 的动态特性，一些人更喜欢这种确定性，但通常并不是必要的。如果我们要检查的值显然是一个`Boolean`值，那么我们可以直接使用它。通常情况下，通过`typeof`或严格相等来检查它的类型是不必要的，除非有可能该值不是`Boolean`。

# 检测数字

在`Number`的情况下，我们可以依赖`typeof`运算符正确地评估为`"number"`：

```js
typeof 555; // => "number"
```

然而，在`NaN`、`Infinity`和`-Infinity`的情况下，它也会评估为`"number"`：

```js
typeof Infinity;  // => "number"
typeof -Infinity; // => "number"
typeof NaN;       // => "number"
```

因此，我们可能希望进行额外的检查，以确定一个数字不是这些值中的任何一个。幸运的是，JavaScript 为这种情况提供了本地辅助工具：

+   `isFinite(n)`: 如果`Number(n)`不是`Infinity`、`-Infinity`或`NaN`，则返回`true`

+   `isNaN(n)`: 如果`Number(n)`不是`NaN`，则返回`true`

+   `Number.isNaN(n)`: 如果`n`不是`NaN`，则返回`true`

+   `Number.isFinite(n)`: 如果`n`不是`Infinity`、`-Infinity`或`NaN`，则返回`true`

全局变量的两个变体是语言的较早部分，正如您所看到的，它们与它们的`Number.*`等效部分略有不同。全局的`isFinite`和`isNaN`通过`Number(n)`将它们的值转换为数字，而等效的`Number.*`方法则不这样做。这种差异的原因主要是遗留问题。

最近添加的`Number.isNaN`和`Number.isFinite`是为了实现更明确的检查而引入的，而不依赖于转换：

```js
isNaN(NaN)   // => true
isNaN('foo') // => true

Number.isNaN(NaN);   // => true
Number.isNaN('foo'); // => false
```

如您所见，`Number.isNaN`更为严格，因为它在检查`NaN`之前不会将值转换为`Number`。对于字符串`'foo'`，我们需要将其转换为`Number`（因此评估为`NaN`）才能通过：

```js
const string = 'foo';
const nan = Number(string);
Number.isNaN(nan); // => true
```

全局的`isFinite`函数的工作方式也是一样的，即在检查有限性之前将其值转换为数字，而`Number.isFinite`方法则不进行任何转换：

```js
isFinite(42)   // => true
isFinite('42') // => true

Number.isFinite(42);   // => true
Number.isFinite('42'); // => false
```

如果您确信您的值已经是一个数字，那么您可以使用更简洁的`isNaN`和`isFinite`，因为它们的隐式转换对您没有影响。如果您希望 JavaScript 尝试将您的非`Number`值转换为`Number`，那么您应该再次使用`isNaN`和`isFinite`。然而，如果出于某种原因您需要明确检查，那么您应该使用`Number.isNaN`和`Number.isFinite`。

结合所有这些讨论过的检查，我们能够通过使用`typeof`结合全局的`isFinite`来自信地检测一个既不是`NaN`也不是`Infinity`的数字。正如我们之前提到的，`isFinite`将检查`NaN`本身，所以我们不需要额外的`isNaN`检查：

```js
function isNormalNumber(n) {
  return typeof n === 'number' && isFinite(n);
}
```

在检测方面，你的需求应该由你的代码上下文驱动。例如，如果你嵌入在一个可以安全假定数字是有限的代码片段中，那么可能不需要检查有限数字。但如果你正在构建一个更公共的 API，那么在将这些值发送到你的内部接口之前，你可能希望进行这样的检查，以减少错误的可能性，并为你的用户提供有用和明智的错误或警告。

# 检测字符串

检测字符串是愉快的简单。我们只需要`typeof`运算符：

```js
typeof 'hello'; // => "string"
```

为了检查给定`String`的长度，我们可以简单地使用`length`属性：

```js
'hello'.length; // => 5
```

如果我们需要检查一个`String`的长度是否大于 0，我们可以通过`length`显式地这样做，或者依赖于长度为 0 的假值，甚至依赖于空`string`本身的假值：

```js
const string = '';

Boolean(string);            // => false
Boolean(string.length);     // => false
Boolean(string.length > 0); // => false

// Since an empty String is falsy we can just check `string` directly:
if (string) { }

// Or we can be more explicit:
if (string.length) { }

// Or we can be maximally explicit:
if (string.length > 0) { }
```

如果我们只是检查一个值的真实性，那么我们也可能检测到所有潜在的真值，包括非零数字和对象。要完全确定你有一个`String`并且它不是空的，最简洁的技术如下：

```js
if (typeof myString === 'string' && myString) {
  // ...
}
```

然而，仅仅空白可能并不是我们感兴趣的全部。我们可能希望检测一个字符串是否包含实际内容。在大多数情况下，*实际内容*从`String`的开头开始，直到`String`的结尾结束，但在某些情况下，它可能嵌入在两侧的空白中。为了解决这个问题，我们可以修剪`String`，然后确认它是否为空：

```js
function isNonEmptyString(string) {
  return typeof string === 'string' && string.trim().length > 0;
}

isNonEmptyString('hi');  // => true
isNonEmptyString('');    // => false
isNonEmptyString(' ');   // => false
isNonEmptyString(' \n'); // => false
```

请注意，我们的函数`isNonEmptyString`是在修剪后的字符串上使用`length > 0`检查，而不仅仅依赖于它作为空字符串的假值。这样我们就可以安全而自信地知道我们的`isNonEmptyString`函数将始终返回一个布尔值。即使在 99%的情况下，它将被用在布尔上下文中，比如`if (isNonEmptyString(...))`，我们仍然应该确保我们的函数具有直观和一致的约定。

逻辑`AND`运算符（`a && b`）将在其左侧为真时返回其右侧。因此，诸如`typeof str === "string" && str`的表达式并不总是保证返回一个布尔值。有关更多信息，请参阅第八章的*运算符-逻辑运算符-逻辑 AND 运算符*部分。

检测字符串是简单的，但正如我们在上一章中提到的，由于 Unicode，与它们一起工作可能是一个挑战。因此，要记住，虽然检测字符串可能会给我们一些确定性，但它并不告诉我们字符串内部的内容以及它是否是我们期望的值。如果你的检测意图是为那些使用你的接口的人提供指南或警告，你可能最好通过明确检查值的内容来服务。

# 检测 undefined

`undefined`类型可以通过引用其全局可用值直接使用严格相等运算符进行检查：

```js
if (value === undefined) {
  // ...
}
```

然而，不幸的是，由于`undefined`可以在非全局范围内被覆盖（取决于你的精确设置和环境），这种方法可能会有问题。从历史上看，`undefined`可以在全局范围内被覆盖。这意味着这样的事情是可能的：

```js
let value = void 0;  // <- actually undefined
let undefined = 123; // <- cheeky override

if (value === undefined) {
  // Does not occur
}
```

`void`运算符，正如我们将在后面探讨的那样，将一个操作数取到其右侧（`void foo`），并且将始终计算为`undefined`。因此，`void 0`已经成为`undefined`的同义词，并且作为替代是有用的。因此，如果你对`undefined`值没有信心，那么你可以简单地检查`void 0`，就像这样：

```js
if (value === void 0) {
  // value is undefined
}
```

出现了各种其他方法来确保可靠的`undefined`值。例如，一个方法是简单地声明一个未赋值的变量（它将始终默认为`undefined`），然后在范围内使用它：

```js
function myModule() {
  // My local `undefined`:
  const undef;

  void 0 === undef; // => true

  if (someValue === undef) {
    // Instead of `VALUE === undefined` I can
    // use `VALUE === undef` within this scope
  }
}
```

随着时间的推移，`undefined`值的可变性已经被锁定。*ECMAScript 2015*禁止了全局修改，但奇怪的是仍然允许本地修改。

值得庆幸的是，始终可以通过简单的`typeof`运算符来检查`undefined`：

```js
typeof undefined; // => "undefined"
```

使用`typeof`这种方式比依赖`undefined`作为字面值要少风险得多，尽管随着 linting 工具的出现，直接检查`undefined`通常是安全的。

我们将在第十五章中探讨 ESLint，这是一个流行的 JavaScript linting 工具，*更干净代码的工具*。在本地范围覆盖`undefined`的情况下，这绝对是一件坏事，它会友好地给我们一个警告。这样的警告可以让我们更有信心，可以安全地使用语言中以前风险较高的方面。

# 检测 null

正如我们所见，`typeof null`评估为`"object"`。这是语言的一个奇怪的遗留。不幸的是，这意味着我们不能依赖`typeof`来检测`null`。相反，我们必须直接比较字面值`null`，使用严格的相等运算符，如下所示：

```js
if (someValue === null) {
  // someValue is null...
}
```

与`undefined`不同，`null`在语言的任何版本和任何环境中都不能被覆盖，因此在使用上不会带来任何麻烦。

# 检测 null 或 undefined

到目前为止，我们已经介绍了如何独立检查`undefined`和`null`，但我们可能希望同时检查两者。例如，一个函数签名通常有一个可选参数。如果未传递该参数或明确设置为`null`，通常会返回到一些默认值。可以通过明确检查`null`和`undefined`来实现这一点，如下所示：

```js
function printHello(name, message) {
  if (message === null || message === undefined) {
    // Default to a hello message:
    message = 'Hello!';
  }
  console.log(`${name} says: ${message}`);
}
```

通常，由于`null`和`undefined`都是假值，通过检查给定值的假值来暗示它们的存在是非常正常的：

```js
if (!value) {
  // Value is definitely not null and definitely not undefined
}
```

然而，这也将检查值是否为其他假值之一（包括`false`，`NaN`，0 等）。因此，如果我们想确认一个值是否特别是`null`或`undefined`，而不是其他假值，那么我们应该坚持使用明确的变体：

```js
if (value === null || value === undefined) //...
```

更简洁的是，我们可以采用抽象（非严格）相等运算符来检查`null`或`undefined`，因为它认为这些值是相等的：

```js
if (value == null) {
  // value is either null or undefined
}
```

尽管这利用了通常被指责的抽象相等运算符（我们将在本章后面探讨），但这仍然是检查`undefined`和`null`的一种流行方式。这是因为它的简洁性。然而，采用这种更简洁的检查会使代码不太明显。甚至可能给人留下作者只是想检查`null`的印象。这种意图的模糊性应该让我们对其干净度产生怀疑。因此，在大多数情况下，我们应该选择更明确和严格的检查。

# 检测数组

在 JavaScript 中检测数组非常简单，因为有`Array.isArray`方法：

```js
if (Array.isArray(value)) {
 // ...
}
```

这种方法告诉我们，传递的值是通过数组构造函数或数组文字构造的。但它不检查值的`[[Prototype]]`，因此完全有可能（尽管不太可能）该值，尽管看起来像一个数组，但可能没有您所期望的特征。

当我们认为需要检查一个值是否是数组时，重要的是问问自己我们真正想要检测什么。也许我们可以检查我们所期望的特征，而不是类型本身。考虑我们将如何处理这个值是至关重要的。如果我们打算通过`for...of`循环遍历它，那么检查其可迭代性可能更适合我们，而不是检查其数组性。正如我们之前提到的，我们可以使用这样的辅助程序来做到这一点：

```js
function isIterable(obj) {
  return obj != null &&
    typeof obj[Symbol.iterator] === 'function';
}

const foo = [1,2,3];
if (isIterable(foo)) {
  for (let f in foo) {
    console.log(f);
  }
}

// Logs: 1, 2, 3
```

另外，如果我们想使用特定的数组方法，比如`forEach`或`map`，那么最好通过`isArray`进行检查，因为这将给我们一个合理的信心，这些方法存在：

```js
if (Array.isArray(someValue)) {
  // Using Array methods
  someValue.forEach(v => {/*...*/});
  someValue.sort((a, b) => {/*...*/});
}
```

如果我们倾向于非常彻底，我们还可以逐个检查特定方法，或者甚至强制将值转换为我们自己的数组，以便我们可以自由地对其进行操作，同时知道该值确实是一个数组：

```js
const myArrayCopy = [...myArray];
```

请注意，通过扩展语法（`[...value]`）复制类似数组的值只有在该值可迭代时才有效。使用`[...value]`的一个适当的例子是在操作从 DOM API 返回的`NodeLists`时：

```js
const arrayOfParagraphElements = [...document.querySelectorAll('p')];
```

`NodeList` 不是真正的`Array`，因此它不提供对原生数组方法的访问。因此，创建并使用一个真正的`Array`的副本是有用的。

总的来说，采用和依赖`Array.isArray`是安全的，但重要的是要考虑是否需要检查`Array`，是否更适合检查值是否可迭代，甚至是否具有特定的方法或属性。与所有其他检查一样，我们应该努力使我们的意图明显。如果我们使用的检查比`Array.isArray`更隐晦，那么最好添加注释或使用一个描述性命名的函数来抽象操作。

# 检测实例

要检测一个对象是否是构造函数的实例，我们可以简单地使用`instanceof`运算符：

```js
const component = new Component();
component instanceof Component; 
```

`instanceof` 运算符将在第八章*，运算符*中更详细地介绍。

# 检测普通对象

当我们说“普通”对象时，我们通常指的是通过`Object`字面量或通过`Object`构造函数构造的对象：

```js
const plainObject = {
  name: 'Pikachu',
  species: 'Pokémon'
};

const anotherPlainObject = new Object();
anotherPlainObject.name = 'Pikachu';
anotherPlainObject.species = 'Pokémon';
```

这与其他对象形成对比，比如语言本身提供的对象（例如数组）和我们通过实例化构造函数自己构造的对象（例如`new Pokemon()`）：

```js
function Pokemon() {}
new Pokemon(); // => A non-plain object
```

检测普通对象的最简单方法是询问它的`[[Prototype]]`。如果它的`[[Prototype]]`等于`Object.prototype`，那么我们可以说它是普通的：

```js
function isPlainObject(object) {
  return Object.getPrototypeOf(object) === Object.prototype;
}

isPlainObject([]);            // => false
isPlainObject(123);           // => false
isPlainObject(new String);    // => false
isPlainObject(new Pokemon()); // => false

isPlainObject(new Object());  // => true
isPlainObject({});            // => true
```

我们为什么需要知道一个值是否是一个普通对象？例如，当创建一个接受配置对象以及更复杂的对象类型的接口或函数时，区分普通对象和非普通对象可能是有用的。

在大多数情况下，我们需要明确检测普通对象。相反，我们应该只依赖它提供给我们的接口或数据。如果我们的抽象的用户希望向我们传递一个非普通对象，但它仍然具有我们需要的属性，那么我们又有什么好抱怨的呢？

# 转换、强制转换和类型转换

到目前为止，我们已经学会了如何使用检测来区分 JavaScript 中的各种类型和特征。正如我们所见，当需要在出现意外或不兼容的值时提供替代值或警告时，检测是有用的。然而，处理这些值的另一个机制是：我们可以将它们从我们不希望的值转换为我们希望的值。

为了转换一个值，我们使用一种称为**类型转换**的机制。类型转换是有意和明确地从一种类型派生另一种类型。与类型转换相反，还有**强制转换**。强制转换是 JavaScript 在使用需要特定类型的运算符或语言结构时隐式和内部进行的转换过程。一个例子是将`String`值传递给乘法运算符。运算符将自然地将其`String`操作数强制转换为数字，以便它可以尝试将它们相乘：

```js
'5' * '2'; // => 10 (Number)
```

*强制转换*和*隐式转换*的基本机制是相同的。它们都是转换的机制。但是我们如何访问这些底层行为是关键的。如果我们明确地这样做，清晰地传达我们的意图，那么我们的代码读者将会有更好的体验。

考虑以下代码，其中包含将`String`转换为`Number`的两种不同机制：

```js
Number('123'); // => 123
+'123'; // => 123
```

在这里，我们使用了两种不同的技术来强制将值从`String`转换为`Number`。当作为函数调用时，`Number()`构造函数将内部将传递的值转换为`Number`原始值。一元`+`运算符也会做同样的事情，尽管它可能不够清晰。强制转换甚至不够清晰，因为它经常似乎是作为某些其他操作的副作用而发生的。以下是一些此类的例子：

```js
1 + '123'; // => "1234"
[2] * [3]; // => 6
'22' / 2;  // => 11
```

当操作数中的一个是字符串时，`+`运算符将强制转换另一个操作数为字符串，然后将它们连接在一起。当给定数组时，`*`运算符将在它们上调用`toString()`，然后将结果的`String`强制转换为`Number`，这意味着`[2] * [3]`等于`2 * 3`。此外，除法运算符在对它们进行操作之前会将它们强制转换为数字。所有这些强制行为都是隐式发生的。

*强制转换*和*显式转换*之间的界限并不是一成不变的。例如，可以通过强制性的副作用明确和有意地转换类型。考虑表达式`someString * 1`，它可以用来将字符串*强制转换*为数字，使用强制转换来实现。在我们的转换中，至关重要的是我们**清楚地传达我们的意图**。

由于强制转换是隐式发生的，它可能是许多错误和意外行为的原因。为了避免这种陷阱，我们应该始终对操作数的类型有很强的信心。然而，强制转换是完全有意的，可以帮助创建更可靠的代码库。在接口的更公共或暴露的一侧，通常会预先将类型转换为所需的类型，以防接收到的类型不正确。

在这里观察一下，我们如何明确地将`haystack`和`needle`的值都转换为`String`类型：

```js
function countOccurrences(haystack, needle) {

  haystack = String(haystack);
  needle = String(needle);

  let count = 0;

  for (let i = 0; i < haystack.length; count++, i += needle.length) {
    i = haystack.indexOf(needle, i);
    if (i === -1) break;
  }

  return count;
}

countOccurrences('What apple is the best type of apple?', 'apple'); // => 2
countOccurrences('ABC ABC ABC', 'A'); // => 3
```

由于我们依赖于`haystack`字符串上的`indexOf()`方法，根据我们所期望的防御级别，将`haystack`转换为字符串是有意义的，这样我们就可以确保它具有可用的方法。将`needle`转换为字符串也会编码更高级别的确定性，这样我们和其他程序员就可以感到放心。

当我们正在创建可重用的实用程序、面向公众的 API 或以降低对接收到的类型的信心的方式消耗的任何接口时，预先将值转换为布尔值以防止不良类型的防御性方法是最佳的。

像 JavaScript 这样的动态类型语言被许多人视为混乱的邀请。这些人可能习惯于严格类型的语言提供的舒适和确定性。事实上，如果充分并谨慎地使用，动态语言可以使我们的代码更加深思熟虑，并且更能适应用户不断变化的需求。在本节的其余部分，我们将讨论转换为各种类型，包括我们可以利用的显式转换机制以及语言内部采用的各种强制行为。我们将首先看一下布尔转换。

# 转换为布尔值

JavaScript 中的所有值在转换为布尔值时，除非它们是七个假值原始值（`false`、`null`、`undefined`、`0n`、`0`、`""`和`NaN`），否则都将返回`true`。

要将值转换为布尔值，我们可以简单地将该值传递给布尔构造函数，将其作为函数调用：

```js
Boolean(0); // => false
Boolean(1); // => true
```

当值存在于布尔上下文中时，语言会将值强制转换为布尔值。以下是一些此类上下文的示例（每个都标有`HERE`）：

+   `if ( HERE ) {...}`

+   `do {...} while (HERE)`

+   `while (HERE) {...}`

+   `for (...; HERE; ...) {...}`

+   `[...].filter(function() { return HERE })`

+   `[...].some(function() { return HERE })`

这个列表并不详尽。我们的值将被强制转换为布尔值的情况还有很多。通常很容易判断。如果一个语言结构或本地提供的函数或方法允许您指定两种可能的路径（也就是*如果 X 那么做这个，否则做那个*），那么它将在内部强制转换您表达的任何值为布尔值。

将值转换为布尔值的常见习语，除了更明确地调用`Boolean()`之外，还有*双感叹号*，即一元逻辑`NOT`运算符（`!`）重复两次：

```js
!!1;  // => true
!![]; // => true
!!0;  // => false
!!""; // => false
```

两次重复逻辑`NOT`运算符将两次反转值的布尔表示。通过将其括起来，更容易理解*双感叹号*的语义：

```js
!( !( value ) )
```

这实际上做了四件事：

+   将值转换为布尔值（`Boolean(value)`）。

+   如果值为`true`，则将其变为`false`。如果值为`false`，则返回`true`。

+   将结果值转换为布尔值（`Boolean(value)`）。

+   如果值为`true`，则将其变为`false`。如果值为`false`，则返回`true`。

换句话说：这做了一个逻辑非，然后又做了一个，结果是原始值本身的布尔表示。

当您创建一个必须返回布尔值的函数或方法，但处理的值不是布尔值时，显式地将值转换为布尔值是特别有用的。例如，我可能希望创建一个`isNamePopulated`函数，如果名称变量不是一个填充的字符串或是`null`或`undefined`，则返回`false`：

```js
function isNamePopulated(name) {
  return !!name;
}
```

如果`name`是一个空的`String`、`null`或`undefined`，这将有助于返回`false`：

```js
isNamePopulated('');        // => false
isNamePopulated(null);      // => false
isNamePopulated(undefined); // => false

isNamePopulated('Sandra');  // => true
```

如果`name`是任何其他假值（例如 0），它也会偶然返回`false`，如果`name`是任何真值，它会返回`true`：

```js
isNamePopulated(0); // => false
isNamePopulated(1); // => true
```

这可能看起来完全不可取，但在这种情况下，这可能是可以接受的，因为我们已经假设`name`是一个`String`、`null`或`undefined`，所以我们只关心函数在这些值方面是否履行了它的合同。您对此的舒适程度完全取决于您具体的实现和它提供的接口。

# 转换为字符串

通过调用`String`构造函数作为常规函数（即不作为构造函数）来实现将值转换为`String`：

```js
String(456); // => "456"
String(true); // => "true"
String(null); // => "null"
String(NaN); // => NaN
String([1, 2, 3]); // => "1,2,3"
String({ foo: 1 }); // => "[object Object]"
String(function(){ return 'wow' }); // => "function(){ return 'wow' }"
```

使用您的值调用`String()`是将值转换为`String`的最明确和清晰的方法，尽管有时会使用更简洁的模式：

```js
'' + 1234; // => "1234"
`${1234}`; // => "1234"
```

这两个表达式可能看起来是等价的，对于许多值来说确实如此。但是，在内部，它们的工作方式是不同的。正如我们将在后面看到的，`+`运算符将通过调用其内部的`ToPrimitive`机制来区分给定操作数是否为`String`，这样操作数的`valueOf`（如果有）将在其`toString`实现之前被查询。然而，当使用模板文字（例如``${value}``）时，任何插入的值都将直接转换为字符串（而不经过`ToPrimitive`）。值的`valueOf`和`toString`方法提供不同的值的可能性总是存在的。看看下面的例子，它展示了如何通过定义我们自己的`toString`和`valueOf`实现来操纵两个看似等价表达式的返回值：

```js
const myFavoriteNumber = {
  name: 'Forty Two',
  number: 42,
  valueOf() { return number; },
  toString() { return name; }
};

`${myFavoriteNumber}`; // => "Forty Two"
'' + myFavoriteNumber; // => 42
```

这可能是一个罕见的情况，但仍然值得考虑。通常，我们假设我们可以轻松地将*任何*值可靠地转换为字符串，但情况并非总是如此。

传统上，很常见依赖于值的`toString()`方法并直接调用它：

```js
(123).toString(); // => 123
```

但是，如果值为`null`或`undefined`，那么您将收到一个`TypeError`：

```js
null.toString();      // ! TypeError: Cannot read property 'toString' of null
undefined.toString(); // ! TypeError: Cannot read property 'toString' of undefined
```

此外，`toString`方法不能保证返回`string`。请注意，我们可以实现自己的`toString`方法，返回`Array`：

```js
({
  toString() { return ['not', 'a', 'string'] }
}).toString(); // => ["not", "a", "string"]
```

因此，最好总是通过非常明确和清晰的`String(...)`进行`string`转换。使用间接的强制形式、副作用或盲目依赖`toString`可能会产生意想不到的结果。请记住，即使您对这些机制有很好的了解并且感到舒适使用它们，也不意味着其他程序员会这样做。

# 转换为数字

通过调用`Number`构造函数作为常规函数，可以将值转换为`Number`：

```js
Number('10e3');     // => 10000
Number(' 4.6');     // => 4.6
Number('Infinity'); // => Infinity
Number('wat');      // => NaN
Number(false);      // => 0
Number('');         // => 0
```

此外，还有一元加号`+`运算符，它基本上做了相同的事情：

```js
+'Infinity'; // => Infinity
+'55.66';    // => 55.66
+'foo';      // => NaN
```

这是将非`Number`转换为`Number`类型的唯一两种方法，但 JavaScript 还提供了其他从字符串中提取数值的技术。其中一种技术是`parseInt`，它是一个全局可用的原生函数，接受一个`String`和一个可选的`radix`参数（默认为*base 10*，即十进制）。如果第一个参数不是`String`，它将自然地将其转换为`String`，然后尝试从`String`中提取指定`radix`的第一个整数。通过这样做，您可以实现以下结果：

```js
parseInt('1000');   // => 1000
parseInt('100', 8); // => 64 (i.e. octal to decimal)
parseInt('AA', 12); // => 130 (i.e. hexadecimal to decimal)
```

如果字符串以`0x`或`0X`为前缀，则`parseInt`将假定`radix`为`16`（*十六进制*）：

```js
parseInt('0x10'); // => 16
```

一些浏览器和其他环境也可能将`0`的前缀视为八进制`radix`的指示符。

```js
// (In **some** environments)
parseInt('023'); // => 19 (assumed octal -> decimal)
```

`parseInt()`还将有效地修剪`String`，忽略任何初始空格，并忽略`String`中第一个找到的整数之后的所有内容：

```js
parseInt(' 111 222 333'); // => 111
parseInt('\t\n0xFF');     // => 255
```

`parseInt`通常不受欢迎，因为它从`String`中提取整数的机制是晦涩的，并且如果没有提供`radix`，它可能会动态选择自己的`radix`。如果必须使用`parseInt`，请谨慎使用，并充分了解其操作方式。并始终提供`radix`参数。

与`parseInt`类似，还有一个原生的`parseFloat`函数，它将尝试从给定的`String`中提取*float*（即*浮点数*）：

```js
parseFloat('42.01');  // => 42.01
parseFloat('\n1e-3'); // => 0.001
```

`parseFloat`将修剪字符串，然后查找从*0^(th)*字符开始的可以被语言自然解析的最长字符集，就像可以解析数字文字一样。因此，它可以很好地处理包含可解析数字序列之外的非数字字符的字符串：

```js
parseFloat('   123 ... rubbish here...'); // => 123
```

如果我们将这样的字符串传递给`Number(...)`，将导致`NaN`被评估。因此，在一些罕见的情况下，`parseFloat`可能对您更有用。

`parseFloat`和`parseInt`都会在尝试提取之前将其初始参数转换为`String`。因此，如果您的第一个参数是对象，则应该注意它可能如何自然地强制转换为字符串。如果您的对象实现了不同的`toString`和`valueOf`方法，则应该期望`parseInt`和`parseFloat`只使用`toString`（除非还实现了`[Symbol.toPrimitive]()`）。这与`Number(...)`相反，后者将尝试直接将其参数转换为`Number`（而不是首先将其转换为`String`），因此将优先考虑`valueOf`而不是`toString`：

```js
const rareSituation = {
  valueOf() { return "111"; },
  toString() { return "999"; }
};

Number(rareSituation); // => 111
parseFloat(rareSituation); // => 999
parseFloat(rareSituation); // => 999
```

在大多数情况下，将任何值转换为`Number`应该通过`Number`或一元加号`+`运算符尝试。只有在需要使用它们的数值提取算法时，才应该使用`parseFloat`或`parseInt`。

# 转换为原始类型

将值转换为其原始表示形式并不是我们可以直接做的事情，但是在许多不同的情况下，语言会隐式地（即*强制性地）进行转换，比如当您尝试使用抽象相等运算符`==`来比较`String`，`Number`或`Symbol`与`Object`的值时。在这种情况下，`Object`将通过一个名为`ToPrimitive`的内部过程转换为其原始表示形式，该过程总结如下：

1.  如果`object[Symbol.toPrimitive]`存在，并且在调用时返回一个原始值，则使用它

1.  如果`object.valueOf`存在，并且返回一个原始值（非`Object`），则使用它的返回值

1.  如果`object.toString`存在，则使用它的返回值

如果我们尝试使用`==`进行比较，我们可以看到`ToPrimitive`的作用：

```js
function toPrimitive() { return 1; }
function valueOf() { return 2; }
function toString() { return 3; }

const one = { [Symbol.toPrimitive]: toPrimitive, valueOf, toString };
const two = { valueOf, toString };
const three = { toString };

1 == one; // => true
2 == two; // => true
3 == three; // => true
```

如您所见，如果一个对象有所有三种方法（`[Symbol.toPrimitive]`，`valueOf`和`toString`），那么将使用`[Symbol.toPrimitive]`。如果只有`valueOf`和`toString`，那么将使用`valueOf`。当然，如果只有`toString`，那么将使用它。

如果使用`String`的提示调用`ToPrimitive`（这意味着它已被指示尝试强制转换为`String`而不是任何原始类型），则该过程中的`*2*`和`*3*`可能会交换。这种情况的一个例子是当您使用计算成员访问运算符（`object[something]`）时，如果`something`是一个对象，则它将通过`ToPrimitive`使用`String`的提示转换为`String`，这意味着在`valueOf()`之前将尝试`toString()`。我们可以在这里看到这一点：

```js
const object = { foo: 123 };
const something = {
  valueOf() { return 'baz'; },
  toString() { return 'foo'; }
};

object[something]; // => 123
```

我们在`something`上定义了`toString`和`valueOf`，但只使用`toString`来确定在`object`上访问哪个属性。

如果我们没有定义自己的方法，比如`valueOf`和`toString`，那么将使用我们使用的任何对象的`[[Prototype]]`上可用的默认方法。例如，数组的原始表示形式是由`Array.prototype.toString`定义的，它将简单地使用逗号作为分隔符将其元素连接在一起：

```js
[1, 2, 3].toString(); // => "1,2,3"
```

所有类型都有自己本地提供的`valueOf`和`toString`方法，因此，如果我们希望强制`ToPrimitive`内部过程使用我们自己的方法，那么我们将需要通过直接提供我们的对象的方法或从`[[Prototype]]`继承来覆盖本地方法。例如，如果您希望提供一个具有自己的原始转换行为的自定义数组抽象，那么您可以通过扩展`Array`构造函数来实现：

```js
class CustomArray extends Array {
  toString() {
    return this.join('|');
  }
}
```

然后，您可以依赖于`CustomArray`实例以其自己独特的方式被`ToPrimitive`过程处理：

```js
String(new CustomArray(1, 2, 3));    // => 1|2|3
new CustomArray(1, 2, 3) == '1|2|3'; // => true
```

所有运算符和本地语言结构的强制行为都会有所不同。每当您将一个值传递给期望原始类型（通常是字符串或数字）的语言结构或运算符时，它可能会通过`ToPrimitive`。因此，了解这个内部过程是很有用的。当我们开始详细探索 JavaScript 的所有运算符时，我们也会参考这一部分。

# 总结

在本章中，我们继续探索 JavaScript 的内部机制，涵盖了语言的动态特性。我们已经看到了如何检测各种类型以及强制和转换的微妙复杂性。这些主题很难掌握，但它们将是有用的。JavaScript 代码中出现的许多反模式都归结于对语言结构和机制的基本误解，因此对这些主题有深入的了解将极大地帮助我们写出干净的代码。

在下一章中，我们将继续探讨类型，通过探索 JavaScript 的运算符。你可能已经对其中许多内容有很好的了解，但由于 JavaScript 的动态特性，它们的使用有时会产生意想不到的结果。因此，下一章将全力以赴地仔细探索语言的运算符。
