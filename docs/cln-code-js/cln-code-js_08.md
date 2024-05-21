# 第六章：原始和内置类型

到目前为止，我们已经从几个不同的角度探讨了清晰代码的含义。我们探讨了我们编写的代码如何让用户通过利用抽象来处理复杂性。我们继续讨论了清晰代码的原则，如可靠性和可用性，以及在追求这些目标时需要注意的各种陷阱和挑战。

在本章中，我们将详细探讨 JavaScript 语言本身，包括更常见的语言构造和更晦涩和令人困惑的方面。我们将把我们对清晰代码的积累知识应用到语言的所有部分，并建立一个纯粹针对清晰代码创建的 JavaScript 理解。

我们将从 JavaScript 最基本的部分开始：作为任何程序的构建块的原始值。然后，我们将转向非原始值，即**对象**。在我们探索这些类型时，我们将通过示例揭示使每种类型独特的语义和在使用中需要避免的陷阱。我们在本章中获得的关键知识将应用在后面的章节中，以便我们真正完全地了解在 JavaScript 中编写清晰代码的含义。

在本章结束时，你应该对以下主题感到自如：

+   原始类型

+   对象

+   函数

+   数组和可迭代对象

+   正则表达式

# 原始类型

在 JavaScript 中，原始类型是指任何不是对象的值，因此没有任何方法或属性。JavaScript 中有七种原始类型：

+   数字

+   字符串

+   布尔

+   未定义

+   空

+   大整数

+   符号

在本节中，我们将探索这些原始值之间的共同特征，并深入研究每种类型，探讨它的工作原理以及在使用中存在的潜在危险。我们将欣赏到 JavaScript 语言本身只是一组不同抽象的集合，当巧妙地使用时，可以轻松解决任何问题领域。

# 原始值的不可变性

所有原始值都是不可变的，这意味着你不能改变它们的值。这是它们原始性的核心部分。例如，你不能将数字值 `3.14` 改变为 `42`，或者将字符串的值更改为它的大写变体。

**但我可以将字符串的值更改为它的大写变体！** 如果你记得能够这样做，你现在可能会感到困惑。但这里需要做出一个重要的区分，即变量重新赋值为新的原始值是完全可能的（也可能是你记得的），而原始值的变异是不可能的。

当我们重新分配一个变量，给它一个新的值时，我们并没有改变值本身；我们只是改变了变量所引用的值，如下所示：

```js
let name = 'simon';
let copy = name;

// Assign a new value to `name`:
name = name.toUpperCase();

// New value referred to by name:
name; // => "SIMON"

// Old value remains un-mutated:
copy; // => "simon"
```

请注意 `copy` 保持小写。原始值 `simon` 没有被改变；相反，通过 `toUpperCase` 方法派生出一个新的原始值，然后赋给之前持有小写变体的变量。

# 原始包装器

你会记得我们提到原始值没有方法，因为它们不是对象。那么，我们是如何能够在前面的字符串上调用 `toUpperCase` 的呢？那不是一个方法吗？是的，是方法。为了让我们能够访问这个方法，JavaScript 在属性访问时会将原始值包装在它们各自的包装对象中。这适用于所有原始值，除了 `null` 和 `undefined`。

在这些被包装的时刻，原始值保持不变，但是通过它们的包装实例，可以访问属性和方法。字符串值将被包装在`String`实例中，而数字值将被包装在`Number`实例中。对于所有其他非空和非未定义的原始值也是如此。您可以自由地实例化这些包装对象：您会发现它们不再像原始值那样行为了；它们是对象，因此您可以在它们上面添加和改变属性：

```js
const name = new String('James');

// We can add arbitrary properties, since it is an object:
// (Warning: this is an anti-pattern)
name.surname = 'Padolsey';
name.surname; // => "Padolsey"
```

如果您需要一个对象来添加自定义属性，最好使用一个普通对象。除了用于包装其原始值以外的任何其他内容，都是一种反模式，因为其他程序员不会预期这样做。尽管如此，观察和记住原始类型及其相应包装对象之间的差异是很有用的。

调用包装构造函数（例如`Number`，`String`等）作为常规函数具有独特的行为。它不会返回一个新的包装实例，而是会将值转换为特定类型并返回一个常规的原始值。当您需要将一种类型转换为另一种类型时，这是非常有用的：

```js
// Cast a number to a string:
String(123); // => "123"

// Cast a string to a number
Number("2"); // => 2

// Cast a number to a boolean
Boolean(0); // => false
Boolean(1); // => true
```

将包装构造函数作为函数调用，就像我们在这里所做的那样，是一种有用的转换技术，尽管这不是唯一的一种。我们将在第七章中更详细地介绍类型转换和强制转换，*动态类型*。

# 虚假原始值

在 JavaScript 中，布尔上下文中的所有值都将计算为`true`或`false`。为了描述这种行为，我们通常将值称为真实或虚假。要确定值的真实性，我们可以简单地将其传递给`Boolean`函数：

```js
Boolean('hi'); // => true
Boolean(0);    // => false
Boolean(42);   // => true
Boolean(0.1);  // => true
Boolean('');   // => false
Boolean(true); // => true
Boolean(null); // => false
```

JavaScript 中只有八个虚假值，它们都是原始类型：

+   `null`

+   未定义

+   `+0`或`-0`（零，一个数字）

+   `false`（布尔值）

+   `""`（空字符串）

+   `0n`（零，一个`BigInt`）

+   `NaN`（不是一个数字）

因此，所有不是虚假的值都是真实的。在本章和下一章中，我们将探讨这些真实和虚假值的含义。现在，只需要知道前面的虚假值在条件或逻辑上下文中使用时会表现得像假一样。例如，当虚假值在`if`语句中使用时，它会像假一样行事：

```js
if (0) {
  // This will not run. 0 is falsy.
}
if (1) {
  // This will run. 1 is truthy.
}
```

这些虚假值的存在意味着我们必须谨慎地检查某些条件。很容易陷入陷阱，只使用其真实性来确定存在的某个值状态。例如，假设我们需要能够检查一个人的年龄：

```js
if (person.age) {
  processIdentity(person);
}
```

这是一个牵强的例子，但我们可以想象一个需要以某种方式处理个体身份的系统，也许是通过医疗应用。如果年龄恰好为 0，检查`age`属性的存在将不会达到预期的效果。也许系统需要适应新生儿被输入系统的可能性，但突然间因为`age`是`0`而崩溃。在这种情况下，最好是预先明确，即使您不希望出现奇怪的虚假值。在这种情况下，我们可能希望检查`null`或`undefined`，因此我们应该明确这样做：

```js
if (person.age === null || person.age === undefined) {
  processIdentity(person);
}
```

这段代码对`age`属性的可能变化更具弹性。我们也可以更符合我们的要求，仅检查我们感兴趣的特定特征，比如`age`属性是在特定范围内的数字。关键是在布尔上下文中，比如`if`语句中最好是明确的，这样您就不会遇到意外的虚假值。

# 数字

数字原始类型用于表示数字数据。它以双精度 64 位浮点格式（IEEE 754）存储这些数据。这里的 64 位指的是有 64 个二进制数字可用于存储信息。在 IEEE 754 标准中使用的整个 64 位格式可以分解为三个部分：

+   **数字的符号需要 1 位**：表示数字是正数还是负数

+   **数字的指数需要 11 位**：这告诉我们小数点的位置

+   **用于分数或有效数字的 52 位**：这告诉我们整数值

浮点形式的一个副作用是，从技术上讲有两个零：正零（`+0`）和负零（`-0`）。幸运的是，在 JavaScript 中，您在检查这些值时不必明确指定。当使用严格相等运算符（`+0 === -0`）进行比较时，两者都将返回 true，并且都被视为假值。

从技术上讲，有 53 位可用（而不是 52）来表示整数值，因为有效数字字段的最高位位于指数字段内。这是一个重要的澄清，因为它直接影响了我们可以从 JavaScript 数字中获得多少精度。有 53 位可用于表示整数值意味着任何大于*2⁵³-1*的数字都被认为是不安全的。这些安全限制作为`Number`对象的常量可用：

+   大于`2⁵³`或`9007199254740991`（`Number.MAX_SAFE_INTEGER`）的整数

+   小于`-2⁵³`或`-9007199254740991`（`Number.MIN_SAFE_INTEGER`）的整数

如果我们尝试对上限进行加法，就会观察到超出这些范围的精度损失：

```js
const max = Number.MAX_SAFE_INTEGER;
max + 1; // => 9007199254740992 (correct)
max + 2; // => 9007199254740992 (incorrect)
max + 3; // => 9007199254740994 (correct)
max + 4; // => 9007199254740996 (incorrect)
// ... etc.
```

在这里，我们可以看到评估的加法是不正确的。超出`MAX_SAFE_INTEGER`，所有数学运算都将同样不精确。

在 JavaScript 中仍然可以表示大于`MAX_SAFE_INTEGER`的值。可以表示多达`2¹⁰²⁴`（`Number.MAX_VALUE`）的许多值，但也有许多值无法表示。因此，尝试表示超出`Number.MAX_SAFE_INTEGER`的数字被认为是非常不明智的。

总之，任何介于`Number.MIN_SAFE_INTEGER`和`Number.MAX_SAFE_INTEGER`之间的值都是安全的，并且将提供整数精度，而超出这些范围的值应被视为不安全。如果我们需要超出这些范围的整数，那么我们可以使用 JavaScript 的`BigInt`原始类型：

```js
const max = BigInt(Number.MAX_SAFE_INTEGER)
max + 1n; // => 9007199254740992n (correct)
max + 2n; // => 9007199254740993n (correct)
max + 3n; // => 9007199254740994n (correct)
max + 4n; // => 9007199254740995n (correct)
// ... etc.
```

我们将在本节的后面进一步探讨`BigInt`原始类型。现在，只需记住始终考虑您的数字的大小以及它们是否可以完全由 JavaScript 的`Number`类型容纳。同样重要的是考虑小数值的精度（例如在分数中）。在 JavaScript 中表示小数时，您可能会遇到此类问题：

```js
0.1 + 0.2; // => 0.30000000000000004
```

这是由浮点标准中表达分数的固有机制所致。您可以想象，如果我们有兴趣查询一个小数是否等于、大于或小于另一个值，那么使用以下代码将会非常简单：

```js
const someValue = 0.1 + 0.2;
if (someValue === 0.3) {
  yay();
}
```

但`yay()`永远不会运行。为了解决这个问题，有两个选择。第一个涉及到一个叫做 epsilon 的东西。Epsilon 是浮点数学固有的误差范围，JavaScript 使其可用作`Number.EPSILON`：

```js
Number.EPSILON; // => 0.0000000000000002220446049250313
```

这是一个非常小的数字，但如果我们希望对小数进行基本的数学运算，就必须考虑到它。如果我们希望比较两个数字，我们可以简单地将它们相互减去，并检查边际是否小于`EPSILON`：

```js
const someValue = 0.1 + 0.2;
if (Math.abs(someValue - 0.3) < Number.EPSILON) {
  // someValue is (effectively) equal to 0.3
}
```

我们可以采取的另一种方法是将我们处理的任何小数转换为由`Number`或`BigInt`类型表示的整数。因此，如果我们需要以八位小数的精度表示从`0`到`1`的值，那么我们可以简单地将这些值乘以`100,000,000`（或`10⁸`）：

```js
const unwieldyDecimalValue = 0.12345678;

// We can use 1e8 to express Math.pow(10, 8)
unwieldyDecimalValue * 1e8; // => 12345678
```

现在，我们可以自由地对这些值进行整数运算，并在完成后将它们分解为分数。需要注意的是，任何小数值超过 15 位数字都无法在 JavaScript 的`Number`类型中表示，因此您需要探索其他选项。JavaScript 目前没有本地的`BigDecimal`类型，但有许多第三方库可用来实现类似的目的（您可以轻松在网上找到这些）。

如果您发现自己需要在 JavaScript 中操作大型或非常精确的数字，或者如果您的代码涉及财务、医学或科学等敏感事项，那么完全理解您需要的精度级别以及 JavaScript 是否可以原生支持这些需求是非常重要的。

还有一个`Number`类型下要讨论的话题，那就是`NaN`。`NaN`是一个技术上属于`Number`类型的原始值。它表示无法将某些东西解析为数字；例如，`Number('wow')`评估为`NaN`。由于`typeof NaN`是`number`，我们应该以以下方式检查有效数字：

```js
if (typeof myNumber === 'number' && !isNaN(myNumber)) {
  // Do something with your number
}
```

当没有预见到`NaN`的存在时，它可能会带来麻烦。它通常会出现在您试图将字符串转换为数字或在这种情况下隐式发生（强制转换）的地方。

我们将在下一章中更多地涵盖强制、转换和检测的主题。这将包括一个部分，我们将深入探讨`NaN`的复杂性，并比较全局函数`isNaN()`与稍有不同的`Number.isNaN()`。目前，重要的是要欣赏`NaN`是其自己独特的值，并且在 JavaScript 中奇怪地被认为是一个数字。

`Number`类型中封装的另一个值不是普通数字：`Infinity`。当您尝试进行数学运算，如除以`0`时，您将收到`Infinity`：

```js
100/0; // => Infinity
```

`Infinity`，就像`NaN`一样，是一个全局可用的原始值，您可以引用和检查：

```js
100/0 === Infinity; // => true
```

还有`-Infinity`，这在技术上是一个不同的值：

```js
100/-0; // => -Infinity
-Infinity === Infinity; // => false
```

`Infinity`，就像`NaN`一样，属于`Number`类型，因此当传递给`typeof`运算符时，它将被评估为`"number"`：

```js
typeof Infinity; // => "number"
```

除了`Infinity`、`-Infinity`和`NaN`之外，所有属于`Number`类型的值都可以被视为普通的日常数字。总的来说，对于大多数用例，`Number`类型非常简单易用。然而，了解它的限制是非常重要的，我们在这里涵盖了许多限制，以便您可以明智地决定何时不适合使用它。

# 字符串

JavaScript 中的`String`类型允许我们表示字符序列。它通常用于封装单词、句子、列表、HTML 和许多其他形式的文本内容。

字符串通过用单引号、双引号或反引号界定字符序列来表示：

```js
// Single quotes:
const name = 'Titanic';

// Double quotes:
const type = "Ship";

// Template literals (back-ticks):
const report = `
  RMS Titanic was a British passenger liner that sank
  in the North Atlantic Ocean in 1912 after the ship
  struck an iceberg during her maiden voyage.
`;
```

只有用反引号界定的字符串，称为**模板文字**（或模板字符串），才能占据多行。单引号或双引号界定的字符串也可以在多行上分布，但这只能通过转义它们的不可见换行字符（使用`\`字符）来实现，这实际上删除了换行：

```js
const a = "example of a \
string with escaped newline \
characters";

const b = "example of a string with escaped newline characters";

a === b; // => true
```

如今，模板文字被认为是首选，因为它们保留了换行，并允许我们插入任意表达式，就像这样：

```js
const nBreadLoaves = 4;
const breadLoafCost = 2.40;

`
  I went to the market and bought ${nBreadLoaves} loaves of
  bread and it cost me ${nBreadLoaves * breadLoafCost} euros.
`
```

一旦您的使用超出了最简单的用例，字符串就会带来许多有趣的挑战。在表面下，这个普通的字符串掩盖了 Unicode 形式的复杂性奇迹。

Unicode 是一个行业标准，用于编码、表示和处理世界各地书写系统中使用的文本。Unicode 标准包含超过 130,000 个字符，包括所有您喜爱的表情符号。

稍微深入字符串抽象的表面，我们可以说 JavaScript 中的字符串实际上只是一系列有序的 16 位无符号整数。这些整数中的每一个都被解释为 UTF-16 代码单元。UTF-16 是 Unicode 字符集的一种编码类型。使用它，我们能够表示数十万个有效的 Unicode 代码点。这意味着我们可以通过我们的字符串来表示表情符号、许多语言和一大堆 Unicode 的奇特之处：

![](img/1cfee330-5c7a-4adf-9e65-53644f2acf6e.png)

Unicode 代码点是一个字符（比如字母*B*、问号或笑脸表情符号）。我们可以通过一个或多个 UTF-16 代码单元来表示一个代码点。我们日常使用的大多数代码点只需要一个代码单元。这些被称为**标量**。然而，有相当多的 Unicode 代码点需要一对代码单元（称为**代理对**）。熊猫表情符号就是这样一个代理对的例子：

![](img/932e06b3-c837-45a4-9998-fa9e94b15f83.png)

由于 UTF-16 只有 16 位可用，它必须使用一对 16 位整数来表示一些字符。自然地，如果我们使用 UTF-32 编码（有 32 位可用），那么我们将能够用一个 32 位整数来表示熊猫表情符号。

在这里，我们使用`charCodeAt()`来确定熊猫表情符号的单个 UTF-16 代码单元，并发现这些是 Unicode 中的第*55,357*和*56,380*个十进制代码单元。由于有这么多代码单元，使用十六进制数字来表示它们更简单、更方便，因此我们可以说熊猫表情符号由代码单元`U+D83D`和`U+DC3C`表示（Unicode 十六进制值通常以`U+`为前缀）。

除了代理对，还有另一种有用的组合类型需要了解。*组合代码点*可以将某些传统的*非组合*代码点增强为新的字符。其中的例子包括可以用重音或其他增强来增强的传统拉丁字符，比如组合波浪符：

![](img/74a8f34d-49af-4c20-91ec-ff6a9c3a72b3.png)

我们选择通过 Unicode 转义序列（`\u0303`）来表示这个特定的组合字符。`\uXXXX`的格式允许我们在 JavaScript 字符串中表示`U+0000`到`U+FFFF`之间的 Unicode 代码单元。

`U+0000`到`U+FFFF`之间的 Unicode 范围被称为**基本多文种平面**（**BMP**），包括最常用的日常字符。

我们的熊猫表情符号，正如我们已经看到的那样，是一个相当晦涩的符号。它在 BMP 上不存在，因此由两个 UTF-16 代码单元的代理对表示。我们可以通过两个 Unicode 转义序列在 JavaScript 字符串中分别表示它们：

![](img/027dce04-a27f-41cb-8b99-e4c742858944.png)

更晦涩和古老的符号位于`U+010000`和`U+10FFFF`之间的*补充*（或*星体*）平面。`\uXXXX`的转义格式没有足够的槽位来表示这些。星体平面内的符号需要至少五个十六进制数字来表示，因此我们必须使用更近期引入的转义序列格式`\u{X}`。这提供了最多六个十六进制槽位（`\u{XXXXXX}`），因此可以表示超过 100 万个不同的代码点。使用这种类型的转义序列，我们可以直接通过其 32 位表示（`U+1F43C`）来表示我们的熊猫表情符号：

![](img/8ac0deef-10e0-4718-a4f9-686f78db91fa.png)

新的`\u{X}`转义序列非常方便，使得 Unicode 比 JavaScript 更易于使用。但是还有更多复杂性需要探索。代理对和组合字符是 UTF-16 代码单元组合成单个符号的例子。此外，还有更长的序列称为**图形簇**。这些用于表示可以组合成一个聚合符号的代码点组合：

![](img/1eb89558-39ae-414f-a0fd-fa6322c9fe8b.png)

哇！Unicode 是一项非常了不起的工程成就，但它可能会让我们的事情变得复杂。能够以所有这些方式组合 Unicode（组合字符、代理对和图形簇）对我们来说是一个挑战。JavaScript 字符串，你可能知道，有一个`length`属性。这个属性返回给定字符串中代码单元的数量（即整个序列中的 16 位整数）。对于大多数字符串来说，这是直接的：

```js
'fox'.length;   // => 3
'12345'.length; // => 5
```

然而，正如我们所知，我们能够组合代码单元来创建代码点，也能够组合代码点来创建图形簇。这意味着`length`属性，它只关注 16 位代码单元，可能会给我们带来意想不到的结果：

![](img/db5089b2-18e2-45bf-9195-d6d02babb3a3.png)

笑脸表情符号由两个代码单元组成，因此 JavaScript 正确告诉我们这个字符串的长度为`2`。但这可能不是我们期望或希望的结果。当我们处理可能使用十几个不同代码单元来表示单个符号的图形簇时，情况会更加复杂。

在 UI 中尝试仅使用其`length`属性截断或确定文本的宽度时要小心。由于许多 Unicode 符号可能由多个代码单元表示，仅使用`length`是不可靠的。

在本节中，我们探讨了 Unicode 的棘手领域。通过对它的新理解，我们现在更有能力在 JavaScript 中清晰地处理字符串。除了 Unicode 的复杂性，JavaScript 中的字符串行为相当直观，只要我们以能清晰传达意图的方式使用它们，就不应该引起太多头痛。

# Boolean

JavaScript 中的`Boolean`原始类型用于表示`true`或`false`。这两个极端是它唯一的值：

```js
const isTrue = true;
const isFalse = false;
```

从语义上讲，布尔值用于表示现实生活或问题域的值，可以被认为是开启或关闭（`0`或`1`），例如，一个功能是否启用，或者用户是否超过一定年龄。这些都是布尔特征，因此适合通过布尔值来表达。我们可以使用这些值来控制程序中的控制流程：

```js
const age = 100;
const hasLivedTo100 = age >= 100;

if (hasLivedTo100) {
  console.log('Congratulations on living to 100!');
}
```

`Boolean`原始类型，就像`String`和`Number`一样，可以手动包装在包装实例中，如下所示：

```js
const isTrueObj = new Boolean(true);
```

请注意，一旦你这样做，`Boolean`将会像条件语句中的任何其他对象一样行为。因此，即使包装的原始值是`false`，以下条件语句也会成功：

```js
const isFalseObj = new Boolean(false);

if (isFalseObj) {
  // This will run
}
```

这里的`Boolean`实例与其原始值不等效；它只是包含其原始值。在`Boolean`上下文中，`isFalseObj`将像`Boolean`上下文中的任何其他对象一样，解析为`true`。手动包装`Boolean`不是特别有用的，应该在大多数程序中避免使用，因为它不符合布尔语义，可能会产生意外结果。

JavaScript 的逻辑运算符（如大于或等于（`>=`）或严格相等（`===`））返回`Boolean`原始值。我们将在第八章中更详细地介绍这些内容，*运算符*。

# BigInt

JavaScript 中的`BigInt`原始类型用于表示任意精度的整数。这意味着它可以用来表示 JavaScript 的`Number`类型无法精确表示的整数（大于~*2⁵³*）。通过在任何数字序列后缀加上`n`字符来声明文字 BigInt，如下所示：

```js
100007199254740991n
```

`BigInt`能够表示任意精度的整数，这意味着你可以存储任意长度的整数。这在金融应用程序或任何需要表达和操作高精度整数的情况下特别有用。

`BigInt`只能对自身进行操作，因此与 JavaScript 的许多原生`Math`方法不兼容：

```js
Math.abs(1n); // !! TypeError: Cannot convert a BigInt value to a number
```

只要两个操作数的类型相同，所有原生数学运算符都可以与`BigInt`一起使用：

```js
(1n + (2n * 3n)) + 4n; // => 11n
```

但是，如果一个操作数是`BigInt`，另一个是`Number`，那么你将收到一个`TypeError`：

```js
1n + 1; // !! TypeError: Cannot mix BigInt and other types, use explicit conversions
```

`BigInt`的语义与`Number`类似：任何直观数值且可以表示为整数的值都可以存储在`BigInt`或`Number`中，具体取决于它所需的精度。

# 符号

`Symbol`原始类型用于表示完全独特的值。通过调用`Symbol`函数创建符号，如下所示：

```js
const totallyUniqueKey = Symbol();
```

你可以选择向这个函数传递一个初始参数，以便为你自己的调试目的注释你的符号，但这并不是必要的：

```js
const totallyUniqueKey = Symbol('My Special Key');
```

符号用作属性键，需要唯一性，或者想要在对象上存储元数据。当你使用`Symbol`键向对象添加属性时，它不会被普通的对象迭代方法（如`for...in`）迭代。对象的`Symbol`键只能通过`Object.getOwnPropertySymbols`来检索：

```js
const thing = {};
thing.name = 'James';
thing.hobby = 'Kayaking';
thing[Symbol(999)] = 'Something else entirely';

for (let key in thing) console.log(key);
// => "name"
// => "hobby"

const symbols =
  Object.getOwnPropertySymbols(thing); // => [Symbol(999)]

thing[symbols[0]]; // => "Something else entirely"
```

由于`Symbol`键以显式但隐藏的方式存在，它们对于存储程序信息在语义上是有用的，这些信息与对象的核心数据无关，但在满足某些程序需求时很有用。例如，你可能有一个日志记录库，并希望用特定方式记录的自定义渲染函数注释特定对象。这样的需求可以很容易地通过符号来实现：

```js
const log = thing => {
 console.log(
   thing[log.CUSTOM_RENDER] ?
     thinglog.CUSTOM_RENDER :
     thing
 );
};
log.CUSTOM_RENDER = Symbol();

class Person {
 constructor(name) {
   this.name = name;
   this[log.CUSTOM_RENDER] = () => {
     return `Person (name = ${this.name})`;
   };
 }
}

log(123); // => Logs "123"
log(new Person('Sarah')); // => Logs: "Person (name = Sarah)"
log(new Person('Wally')); // => Logs: "Person (name = Wally)"
log(new Person('Julie')); // => Logs: "Person (name = Julie)"
```

并不是很多日常情况下需要创建和使用新符号，但有很多情况下需要通过这些符号来规定原生行为。例如，你可以使用`Symbol.iterator`属性为你的对象定义一个自定义迭代器。我们将在本章后面的*数组和可迭代对象*部分详细介绍这一点。

# null

`null`原始类型用于表示有意的值的缺失。它是一个只有一个值的类型：唯一的 null 值是`null`。

`null`的语义与`undefined`有着根本的不同。`undefined`值用于指示未声明或未定义的内容，而`null`是一个明确声明的缺失值。我们通常使用`null`值来表示一个值要么明确尚未设置，要么由于某种原因不可用。

例如，让我们考虑一个 API，我们在其中指定与餐厅评论相关的各种属性：

```js
setRestaurantFeatures({
  hasWifi: false,
  hasDisabledAccess: true,
  hasParking: null
});
```

在这种情况下，`null`值表示我们不知道`hasParking`的值。当我们有必要的信息时，我们可以将`hasParking`指定为`true`或`false`（`Boolean`），但为了表示我们对其真实值的无知，我们将其设置为`null`。我们也可以完全省略该值，这意味着它实际上是`undefined`。关键区别在于使用`null`总是主动进行的，而`undefined`是某事没有完成的结果。

如前所述，`null`值始终是假值，这意味着在`Boolean`上下文中它将始终计算为`false`。因此，如果我们尝试在条件语句中使用`null`，那么它将不会成功：

```js
function setRestaurantFeatures(features) {
  if (features.hasParking) {
    // This will not run as hasParking is null
  }
} 
```

重要的是要检查我们想要的确切值，这样我们可以避免错误并有效地向阅读我们代码的人传达信息。在这种情况下，我们可能希望明确检查`undefined`和`null`，因为我们想要针对这种情况执行不同的代码，而不是针对`false`的情况。我们可以这样做：

```js
if (features.hasParking !== null && features.hasParking !== undefined) {
  // hasParking is available...
} else {
  // hasParking is not set (undefined) or unavailable (null)
}
```

我们还可以使用抽象相等运算符（`==`）来与`null`进行比较，如果操作数是`null`或`undefined`，它将有用地评估为`true`：

```js
if (features.hasParking != null) {
  // hasParking is available...
} else {
  // hasParking is not set (undefined) or unavailable (null)
}
```

事实上，这与更明确的比较是一样的，但更加简洁。不幸的是，它并不清楚它的意图是检查`null`和`undefined`。通常我们应该更加明确，因为这样可以更有效地向其他程序员传达我们的意图。

要避免的最后一个陷阱是`null`的`typeof`运算符。由于 JavaScript 语言的一些遗留问题，`typeof null`会返回`"object"`，因此完全不可靠。

有关`typeof`和检测`null`类型的更多信息可以在第七章的*动态类型*中的*检测*部分找到。

所以，你已经知道了。`null`是一个足够简单的值，在干净的代码方面，只要记住两个关键点就不会出错：它只应该用来表示有意识地缺少一个值，并且最好明确地检查它（最好使用`value === null`）。

# undefined

`undefined`原始类型表示某物尚未被定义或仍然未定义。与`null`一样，它是一个只有一个值（`undefined`）的类型。与`null`不同，`undefined`值不应该被明确设置，但当某物没有值时语言可能会返回它：

```js
const coffee = {
  type: 'Flat White',
  shots: 2
};

coffee.name; // => undefined
coffee.type; // => "Flat White"
```

未定义最好被认为是某物的缺失。如果你发现自己希望明确地将某物设置为`undefined`，你应该考虑使用`null`代替。

重要的是要区分未定义和甚至未声明的概念。在 JavaScript 中，如果你尝试评估一个在你的范围内不存在的标识符，你会得到一个`ReferenceError`：

```js
thisDoesNotExist; // !! ReferenceError: thisDoesNotExist is not defined
```

然而，正如你已经看到的，如果你尝试评估一个对象的属性，而该属性不存在，你将不会得到任何错误。相反，它将评估为`undefined`：

```js
const obj = {};
obj.foo; // => undefined
```

然而，如果你尝试访问不存在的`foo`属性下的属性，你将收到一个`TypeError`，抱怨它无法读取一个具有`undefined`值的属性：

```js
obj.foo.baz; // !! TypeError: Cannot read property 'baz' of undefined
```

这种行为是寻求访问`undefined`或`null`值的任何属性时总是会抛出`TypeError`的扩展：

```js
(undefined).foo;  // !! TypeError: Cannot read property 'foo' of undefined
```

有趣的是，与`null`不同，`undefined`值不是一个字面量，而是语言提供的一个全局可用的值。在 ECMAScript 2015 及以后的版本中不可能覆盖这个全局值，但在本地（非全局）范围内定义自己的`undefined`标识符的值仍然是可能的：

```js
undefined; // => undefined

function weird() {
  let undefined = 1;
  undefined; // => 1
}
```

这是一种反模式，因为它可能会产生非常尴尬和意想不到的结果。在比你的范围更高的范围意外设置`undefined`可能意味着，如果你依赖于该值，你最终可能会引用一个不是`undefined`的值。对`undefined`值的不信任在历史上意味着人们已经找到其他方法来强制在他们的范围内使`undefined`可用。例如，声明一个变量但不给它赋值将始终导致它的值为`undefined`：

```js
function scopeWithReliableUndefined() {
  let undefined;
  undefined; // => undefined
}
```

你还可以对任何值使用 JavaScript 的`void`运算符，它将始终返回`real`的`undefined`值：

```js
void 0;         // => undefined
void null;      // => undefined
void undefined; // => undefined
```

在你的范围内明确设置未定义意味着你可以安全地引用你的`undefined`值，而不必担心它已被破坏。然而，幸运的是，你可以通过使用`typeof`运算符来避免担心这种风险的痛苦：

```js
if (typeof myValue === 'undefined') { ... }
```

即使`myValue`不存在，这也不会抛出`ReferenceError`。正如我们已经发现的那样，`typeof`运算符与`null`一样，有时我们不能总是依赖它，但当明确检查`undefined`时，它仍然非常有用。

避免`undefined`的另一种方法是通过使用 linting 工具在代码库中强制正确使用它。我们将在第十五章中介绍 linting 工具，*更干净的代码的工具*。

总之，如果记住以下两点，可以干净地使用`undefined`：

+   避免直接将`undefined`分配给变量；您应该使用`null`代替

+   始终明确检查`undefined`，优先使用`typeof`运算符

这结束了我们对 JavaScript 中原始类型的探索。现在，我们将转向非原始类型，也就是对象。

# 对象

在 JavaScript 中，除了原始值之外的所有内容都可以视为对象。甚至函数实际上也是专门的对象；它们唯一的区别在于它们可以被调用。然而，通常情况下，当我们使用术语`对象`时，我们指的是通常以花括号括起来的对象文字声明的普通对象，其中包含一组键值对：

```js
const animal = {
  name: 'Duck',
  hobby: 'Paddling'
};
```

您还可以通过`Object`构造函数实例化对象，然后直接添加属性：

```js
const animal = new Object();
animal.name = 'Duck';
animal.hobby = 'Paddling';
```

尽管它们是等效的，但在大多数情况下最好使用对象文字，因为它更简单声明和阅读，特别是如果有许多属性。它还具有一个额外的好处，即允许您创建并传递对象作为表达式，而无需事先准备。

# 属性名称

用于向对象添加属性（属性名称）的键内部存储为字符串。但是，当使用对象文字语法时，可以将键声明为常规标识符（即，任何您可以用作变量名的内容）、数字文字或字符串文字：

```js
const object = {
  foo: 123,   // Using an identifier as the key
  "baz": 123, // Using a String literal as the key
  123: 123    // Using a Number literal as the key
};
```

最好尽可能使用标识符，因为这有助于限制您使用可以轻松访问为属性的键名。如果您使用的是不是有效标识符的字符串文字，那么您将不得不使用方括号表示法来访问它，这可能会很麻烦：

```js
const data = {
  hobbies: ['tennis', 'kayaking'],
  'my hobbies': ['tennis', 'kayaking']
};

data.hobbies;       // Easy
data['my hobbies']; // Burdensome
```

您还可以使用计算属性名称（用方括号括起来）将动态命名的项添加到对象文字中：

```js
const data = {
  ['item' + (1 + 2)]: 'foo'
};

data; // => { item3: "foo" }
data.item3; // => "foo"
```

正如我们之前提到的，JavaScript 中的所有非原始值在技术上都是对象。但是，还有什么使某物成为对象呢？对象允许我们将任意值分配给它们作为属性，这是原始值无法做到的。除了这一特征之外，JavaScript 中对象的定义留下了令人愉快的泛化。我们可以以许多不同的方式使用对象来适应我们正在编写的代码。许多语言将为字典或哈希映射提供语言构造。在 JavaScript 中，我们可以使用对象来满足这些需求的大部分。当我们需要存储键值对，其中键不是字符串时，通常通过对象的`toString`方法提供该值的字符串表示：

```js
const me = {
  name: 'James',
  location: 'England',
  toString() {
    return [this.name, this.location].join(', ')
  }
};

me.toString(); // => "James, England"
String(me); // => "James, England"
```

当对象被放置在强制转换为字符串的上下文中时，将在内部调用此方法，例如通过方括号表示法进行访问或分配：

```js
const peopleInEurope = {};

peopleInEurope[me] = true;
Object.keys(peopleInEurope); // => ["James, England"]
peopleInEurope[me]; // => true
```

这在历史上曾被用于允许实现数据结构，其中键实际上是非原始的（尽管对象在技术上将属性名称存储为字符串）。然而，如今更倾向于使用`Map`或`WeakMap`。

# 属性描述符

以常规方式向对象添加属性，无论是通过属性访问还是通过对象文字，属性都将具有以下隐式特征：

+   `configurable`：这意味着属性可以从对象中删除（如果其属性描述符可以更改）

+   `enumerable`：这意味着属性将对`for...in`和`Object.keys()`等枚举可见

+   `writable`：这意味着可以通过赋值运算符（例如`obj.prop = ...`）更改属性的值

JavaScript 赋予你关闭这些特性的权力，但要注意，对这些特性的更改可能会使代码的行为变得模糊。例如，如果一个属性被描述为不可写，但尝试通过赋值进行写入（例如，`obj.prop = 123`），那么程序员将收到没有发生写入的警告。这可能会导致意外和难以找到的错误。因此，牢记将要使用你的接口的程序员的期望是至关重要的。因此，你要小心谨慎地保留属性描述符。

你可以通过原生提供的`Object.defineProperty()`为给定的属性定义自己的特性。在设置新属性描述符时，每个特性的默认值将为`false`，因此，如果希望给属性赋予`configurable`、`enumerable`或`writable`的特性，则需要明确指定这些特性为`true`。

```js
const myObject = {};

Object.defineProperty(myObject, 'name', {
  writeable: false,
  configurable: false,
  enumerable: true,
  value: 'The Unchangeable Name'
});

myObject.name; // => "The Unchangeable Name"
myObject.name = 'something else'; // => (Ineffective)
myObject.name; // => "The Unchangeable Name"

delete myObject.name; // => false (Ineffective)
myObject.name; // => "The Unchangeable Name"
```

你也可以使用`Object.defineProperties()`一次描述多个属性：

```js
const chocolate = Object.defineProperties({
  // Empty object where our described properties
  // will be placed
}, {
 name: { value: 'Chocolate', enumerable: false },
 tastes: { value: ['Bitter', 'Sweet'], enumerable: true }
});

chocolate.name; // => "Chocolate"
chocolate.tastes; // => ["Bitter", "Sweet"]

Object.keys(chocolate); // => ["tastes"]
```

如果尝试更改具有`configurable`设置为`false`的属性的特性，则会收到`TypeError`：

```js
const obj = {};

Object.defineProperty(
 obj,
 'timestamp',
 { configurable: false, value: Date.now() }
);

Object.defineProperty(
  obj,
  'timestamp',
  { configurable: true }
);
// ! TypeError: Cannot redefine property: timestamp
```

还可以设置自定义的 setter 和 getter。*getter 定义了在访问属性时将返回的值，而 setter 将定义在尝试对该属性进行赋值时发生的情况（即通过赋值运算符）。在希望以独特方式保存值或在赋值时对值进行过滤或处理的情况下，使用这些功能可能很有用，例如：

```js
const data = Object.defineProperties({}, {
  name: {
    set(name) { this.normalizedName = name.toLowerCase(); },
    get() { return this.normalizedName; }
  }
});

data.name = 'MoLLy BroWn';
data.name; // => "molly brown"
```

由于`name`属性是通过`defineProperties`描述的，它将禁用所有默认特性，这意味着它不可枚举，不可写，也不可配置。如果我们尝试枚举它，我们会发现我们内部使用的`normalizedName`被找到了：

```js
Object.keys(data); // => ["normalizedName"]
```

在处理属性描述符时要牢记这一点。确保你了解每个属性具有什么特性，并注意内部实现的泄漏！

值得注意的是，也可以（通常更可取）在对象文字或类定义中直接为属性定义 getter 和 setter。例如，我们可以创建一个`Array`的子类，添加一个`last`属性，该属性充当数组中最后一个元素的 getter：

```js
class SpecialArray extends Array {
  get last() { return this[this.length - 1]; }
}

const myArray = new SpecialArray('a', 'b', 'c', 'd');
myArray.last; // => "d"
myArray.push('e');
myArray.last; // => "e"
```

有许多这样创造性的 getter 和 setter 的用法。但是，与`configurable`、`enumerable`和`writable`的特性一样，重要的是要谨慎考虑你的自定义行为将如何影响你的同行程序员的期望。如果你创建的抽象或数据结构在行为上不熟悉或不可预测，那么你就为误解和错误铺平了道路。最好的方法是与语言本身的自然语义保持一致。因此，每当你要创建一个自定义 setter 或将属性描述为不可写时，请问自己程序员是否可以合理地期望它以这种方式工作。遵循一个有帮助的规则，被称为**最少惊讶原则**（**POLA**）！

POLA（或最少惊讶）适用于软件设计和 UX 设计。它广泛意味着系统的给定功能或组件应该像大多数用户期望的那样行事，并且不应该过于惊讶或使人惊讶。

# Map 和 WeakMap

`Map`和`WeakMap`抽象能够存储键值对，其中，与常规对象不同，键可以是任何东西，包括非原始值：

```js
const populationBySpecies = new Map();
const reindeer = { name: 'Reindeer', formalName: 'Rangifer tarandus' };

populationBySpecies.set(reindeer, 2000000);
populationBySpecies.get(reindeer); // => 2,000,000
```

`WeakMap`类似于`Map`，但它只保留对用作键的对象的弱引用，这意味着，如果由于在程序的其他位置进行垃圾回收而使对象不可用，那么`WeakMap`将停止保持它。

大多数情况下，普通对象就足够了。只有在需要键为非原始类型或者想要弱引用值时，才应该使用`Map`或`WeakMap`。

# 原型

JavaScript 是一种原型语言，继承是通过原型实现的。这可能是一个令人生畏的概念，但实际上非常简单。JavaScript 的原型行为可以描述如下：每当在对象上访问属性时，如果该属性在对象本身上不可用，JavaScript 将尝试在内部可用的`[[Prototype]]`属性上访问它。然后它将重复这个过程，直到找到属性或到达原型链的顶部并返回`undefined`。

了解`[[Prototype]]`属性的功能将使您对语言有很大的掌握，并且会立即使 JavaScript 变得不那么令人生畏。这可能很难理解，但最终是值得的。

`[[Prototype]]`对象本身实际上就是一个普通对象，可以合理地附加到任何其他对象上。我们可以创建一个称为`engineerPrototype`的对象，并使其包含与工程师角色相关的数据和方法，例如：

```js
const engineerPrototype = {
  type: 'Engineer',
  sayHello() {
    return `Hello, I'm ${this.name} and I'm an ${this.type}`;
  }
};
```

然后，我们可以将这个原型附加到另一个对象上，从而使其属性也在那里可用。为此，我们使用`Object.create()`，它创建一个带有硬编码`[[Prototype]]`的新对象：

```js
const pandaTheEngineer = Object.create(engineerPrototype);
```

内部的`[[Prototype]]`属性不能直接设置，因此我们必须使用`Object.create`和`Object.setPrototypeOf`等机制。请注意，您可能已经看到使用非标准的`__proto__`属性来设置`[[Prototype]]`的代码，但这是一个遗留特性，不应依赖它。

有了这个新创建的`pandaTheEngineer`对象，我们可以访问其`[[Prototype]]`上可用的任何属性，比如`engineerPrototype`：

```js
pandaTheEngineer.name = 'Panda';
pandaTheEngineer.sayHello(); // => "Hello, I'm Panda and I'm an Engineer"
```

我们可以通过向`engineerPrototype`添加新属性来说明这些对象现在是链接在一起的，并观察它如何在`pandaTheEngineer`上可用：

```js
pandaTheEngineer.sayGoodbye; // => TypeError: sayGoodbye is not a function
engineerPrototype.sayGoodbye = () => 'Goodbye!';
pandaTheEngineer.sayGoodbye(); // => 'Goodbye!'
```

正如我们之前提到的，如果对象本身上没有可用的属性，`[[Prototype]]`的属性将被用于解析属性。以下代码显示了我们如何在`pandaTheEngineer`对象上设置自己的`sayHello`方法，这样一来，我们就不再可以访问`[[Prototype]]`上定义的`sayHello`方法：

```js
pandaTheEngineer.sayHello = () => 'Yo!';
pandaTheEngineer.sayHello(); // => "Yo!"
```

然而，删除这个新添加的`sayHello`方法意味着我们再次可以访问`[[Prototype]]`上的`sayHello`方法：

```js
delete pandaTheEngineer.sayHello;
pandaTheEngineer.sayHello(); // => // => "Hello, I'm Panda and I'm an Engineer"
```

为了理解发生了什么以及哪些属性来自哪个对象，我们始终可以使用`Object.getPrototypeOf`来检查对象的`[[Prototype]]`：

```js
// We can inspect its prototype:
Object.getPrototypeOf(pandaTheEngineer) === engineerPrototype; // => true
```

现在，我们可以通过`Object.getOwnPropertyNames`检查它的属性：

```js
Object.getOwnPropertyNames(
  Object.getPrototypeOf(pandaTheEngineer)
); // => ["type", "sayHello", "sayGoodbye"]
```

在这里，我们可以看到`[[Prototype]]`对象（即`engineerPrototype`）提供了`type`、`sayHello`和`sayGoodbye`属性。如果我们检查`pandaTheEngineer`对象本身，我们会发现它只有一个`name`属性：

```js
Object.getOwnPropertyNames(pandaTheEngineer); // => ["name"]
```

正如我们之前添加`sayGoodbye`方法时观察到的，我们可以随时修改该原型，并且我们的更改将对使用该原型的任何对象可用。这里是另一个这样做的例子：

```js
// Modify the prototype object:
engineerPrototype.type = "Awesome Engineer";

// Call a method on our object (that uses the prototype):
pandaTheEngineer.sayHello(); // => "Hello, I'm Panda and I'm an Awesome Engineer"
```

在这里，您可以看到我们继承的`sayHello`方法是如何生成一个包含我们变异的类型属性（即`"Awesome Engineer"`）的字符串。

希望您开始看到我们如何使用原型构建继承层次结构。`[[Prototype]]`的非常简单的机制允许我们在对象表示的问题域之间表达复杂的层次关系。这就是 JavaScript 中实现 OOP 的方式。

我们可以合理地创建另一个原型，它本身使用`engineerPrototype`，可能是`fullStackEngineerPrototype`，并且它将按预期工作，每个原型定义另一层属性解析。

JavaScript 的新*类定义语法*在表面之下，你可能已经习惯了，依赖于原型的这种基本机制。这可以在这里观察到：

```js
class Engineer {
  type = 'Engineer'
  constructor(name) {
    this.name = name;
  }
  sayHello() {
    return `Hello, I'm ${this.name} and I'm an ${this.type}`;
  }
}

const pandaTheEngineer = new Engineer();

Object.getOwnPropertyNames(pandaTheEngineer); // => ["type", "name"]

Object.getOwnPropertyNames(
  Object.getPrototypeOf(pandaTheEngineer)
); // => ["constructor", "sayHello"]
```

你会注意到这里有一些细微的差别。最关键的一个是，在声明类时，目前没有办法在原型对象上定义非方法属性。当我们声明`type`属性时，我们正在填充实例本身，所以当我们检查实例的属性时，我们得到`"type"`和`"name"`。然而，方法（比如`sayHello`）将存在于`[[Prototype]]`上。另一个区别是，当使用类时，我们能够声明一个`constructor`，它本身是`[[Prototype]]`上的一个方法/属性。

基本上，*类定义语法*（在*ECMAScript 2015*中引入）并没有使语言中已经存在的任何东西成为可能。它只是利用了现有的原型机制。然而，新的语法确实使一些事情变得更简单，比如使用`super`关键字引用超类。

在类定义存在之前，我们通常通过将我们预期的`[[Prototype]]`对象分配给函数的`prototype`属性来编写类似类的抽象，如下所示：

```js
function Engineer(name) {
  this.name = name;
}

Engineer.prototype = {
  type: 'Engineer',
  sayHello() {
    return `Hello, I'm ${this.name} and I'm an ${this.type}`;
  }
};
```

当一个函数通过`new`运算符实例化时，如果有的话，JavaScript 将隐式地创建一个`[[Prototype]]`设置为函数的`prototype`属性的新对象。让我们尝试实例化`Engineer`函数：

```js
const pandaTheEngineer = new Engineer();
```

检查这个结果会得到我们在原始`Object.create`方法中看到的相同特征：

```js
Object.getOwnPropertyNames(pandaTheEngineer); // => ["name"]

Object.getOwnPropertyNames(
  Object.getPrototypeOf(pandaTheEngineer)
); // => ["type", "sayHello"]
```

总的来说，所有这些方法都是相同的，但在某些属性所在的位置上有一些细微的差别（即，它的属性是在实例本身上还是在它的`[[Prototype]]`上）。新的*类定义语法*很有用且简洁，因此现在更受欢迎，但了解原型工作的基本知识仍然很有用，因为它驱动着整个语言，包括所有的原生类型。我们可以像在前面的代码中一样检查这些原生类型：

```js
const array = ['wow', 'an', 'array'];

Object.getOwnPropertyNames(array); // => ["0", "1", "2", "length"]

Object.getOwnPropertyNames(
  Object.getPrototypeOf(array)
); // => ["constructor", "concat", "find", "findIndex", "lastIndexOf", "pop", "push", ...]
```

变异原生原型是一种反模式，应该尽量避免，因为它可能会在代码库中与其他代码产生意外的冲突。由于运行时只有一个集合的原生类型可用，当你修改它们时，你正在修改当前存在的该类型的每个实例的能力。因此，最好遵守一个简单的规则：**只修改你自己的原型**。

如果你发现自己试图修改一个原生原型，最好是创建该类型的自己的子类，并在那里添加你的功能：

```js
class HeartArray extends Array {
  join() {
    return super.join(' ❤ ');
  }
}

const yay = new HeartArray('this', 'is', 'lovely');

yay.join(); // => "this ❤ is ❤ lovely"
```

在这里，我们正在创建我们自己的`Array`子类，称为`HeartArray`，以便我们可以添加我们自己专门的`join`方法。

# 何时以及如何使用对象

任何类型的对象，就像我们的原始值一样，应该只与它所代表的语义概念一起使用。将`Array`子类化为`HeartArray`的前面案例是有意义的，因为我们希望通过它来表达的数据确实类似于数组，也就是说，它是一组顺序的单词。

当我们开始将对象塑造成适合我们需求的抽象时，我们应该始终考虑其他程序员对对象的期望以及这些期望的后果。我们将在第十一章中深入探讨设计良好的抽象的微妙之处，那里我们将利用对象以多种方式来构建抽象。

本节介绍了 JavaScript 中对象的概念——它们无处不在——以及它们是如何通过原型在表面之下运作的。这些基本知识将使你更容易地使用 JavaScript，并帮助你编写更干净的代码。

# 函数

在 JavaScript 中，函数就像任何其他类型一样；它们可以像对象和原始类型一样传递。然而，当我们谈论大多数其他值时，我们会发现通常只有一种方式来声明它们。对象文字使用大括号声明。数组文字使用方括号分隔。然而，函数以各种文字形式出现。

在对象文字或类定义之外，可以以三种不同的方式声明函数：作为函数声明，作为函数表达式，或作为一个箭头函数表达式：

```js
// Function Declaration
function myFunction() {}

// Function Expression
const myFunction = function () {};

// Named Function Expression
const myFunction = function myFunction() {};

// "Fat"-Arrow Function Expression
const myFunction = () => {};
```

然而，在对象文字中声明函数有一种更简洁的语法，称为**方法定义**：

```js
const things = {
  myMethod() {},
  anotherMethod() {}
};
```

我们需要用逗号分隔这些方法定义（就像我们必须对对象文字中定义的任何其他属性做的那样）。类定义也允许我们使用方法定义，尽管它们不需要分隔逗号：

```js
class Thing {
  myMethod() {}
  anotherMethod() {}
}
```

方法只是在调用时绑定到对象的函数。这包括在类定义内部定义的函数和以任何方式分配给对象属性的函数。然而，在讨论代码时，了解人们说*方法*和*函数*时的含义是有用的。然而，从根本上讲，JavaScript 的语言并不区分它们——它们在技术上都只是函数。

定义函数的各种方式都有微妙的差异，值得了解，因为典型的 JavaScript 代码库将使用大多数，如果不是所有这些风格。您将遇到的函数声明的差异类型包括以下内容：

+   定义风格是否*提升*到其作用域的顶部；例如，函数声明

+   定义风格是否创建具有自己绑定的函数（例如，`this`）；例如，函数表达式

+   定义风格是否创建具有自己`name`属性的函数；例如，函数声明

+   定义风格是否与代码的特定区域相关；例如，方法定义

现在，我们可以更详细地讨论各种定义风格的语法。

# 语法上下文

函数可以存在于三种语法上下文中：

+   作为一个声明

+   作为一个表达式

+   作为方法定义

**语句**可以被视为脚手架。例如，`const X = 123`是一个包含`const`声明和赋值的*语句*。**表达式**可以被视为您放入脚手架中的值；例如，后一个*语句*中的`123`是一个*表达式*。在第九章中，*语法和作用域的部分*，我们将更详细地讨论这个主题。

函数作为语句和函数作为表达式之间的区别体现在函数表达式和函数声明上。函数声明非常独特，因为它是声明函数的唯一方式。要被视为函数声明，`function name() {}`的语法必须独立存在，不能用在表达式的上下文中。这可能非常令人困惑，因为你不能仅仅根据函数自身的语法来判断函数是函数声明还是函数表达式；相反，你必须看它存在的上下文：

```js
// This is a statement, and a function declaration:
// And will therefore be hoisted:
function wow() {}

// This is a statement containing a function expression:
const wow = function wow() {};
```

正如我们之前提到的，函数表达式允许有一个名称，就像函数声明一样，但该名称可能与分配给函数的变量的名称不匹配。

最容易将表达式视为任何可以合法存在于赋值运算符的右侧的东西。以下所有的*右侧*都是合法的表达式：

```js
foo = 123;
foo = [1,2,3];
foo = {1:2,3:4};
foo = 1 | 2 | 3;
foo = function() {};
foo = (function(){})();
foo = [function(){}, ()=>{}, function baz(){}];
```

函数表达式在语法上与 JavaScript 中的所有其他值一样灵活。我们将发现，函数声明是有限制的。方法定义也受限于存在于对象字面量或类定义的范围内。

# 函数绑定和 this

函数的绑定指的是 JavaScript 在函数体内提供的一组额外和隐式值的引用。这些绑定包括以下内容：

+   `this`：`this`关键字指的是函数调用的执行上下文

+   `super`：方法或构造函数中的`super`关键字指的是其超类

+   `new.target`：此绑定告诉您函数是否是通过`new`运算符作为构造函数调用的

+   `arguments`：此绑定提供了对在调用函数时传递的参数的访问

这些绑定对所有函数都可用，除了使用箭头语法定义的函数（`fn = () => {}`）。以这种方式定义的函数将有效地吸收父作用域的绑定（如果有的话）。每个绑定都有独特的行为和约束。我们将在以下子节中探讨这些内容。

# 执行上下文

`this`关键字通常在函数调用时确定，并且通常会解析为函数被调用的对象。它有时被称为函数的执行上下文或`thisArg`。这可能不直观，因为这意味着`this`值在调用之间可以在技术上发生变化。例如，我们可以将一个对象的方法分配给另一个对象，然后在第二个对象上调用它，并观察到它的`this`始终是调用它的对象：

```js
const london = { name: 'London' };
const tokyo = { name: 'Tokyo' };

function sayMyName() {
  console.log(`My name is ${this.name}`);
}

sayMyName(); // => Logs: "My name is undefined"

london.sayMyName = sayMyName;
london.sayMyName(); // => Logs "My name is London"

tokyo.sayMyName = sayMyName;
tokyo.sayMyName(); // => Logs "My name is Tokyo"
```

当没有调用对象时，例如我们直接调用`sayMyName`时，它的假定执行上下文是代码所在的全局环境。在浏览器中，这个全局环境等同于 window 对象（提供对浏览器和文档对象模型的访问），而在 Node.js 中，this 指的是每个特定模块/文件独有的环境，其中包括该模块的 exports 等内容。

除了在全局调用函数的情况下，还有两种情况下`this`关键字会是除了明显的调用对象之外的东西：

+   如果被调用的函数被定义为箭头函数，那么它将吸收所在作用域的`this`值

+   如果被调用的函数是构造函数，它的`this`值将是一个新对象，其`[[Prototype]]`预设为函数的原型属性

在调用或声明函数时，也有一些方法可以强制`this`的值。您可以使用`bind(X)`来创建一个新函数，其`this`值设置为`X`：

```js
const sayHelloToTokyo = sayMyName.bind(tokyo);
sayHelloToTokyo(); // => Logs "My name is Tokyo"
```

您还可以使用函数的`call`和`apply`方法来强制任何给定调用的`this`值，但请注意，如果函数被调用为构造函数（即使用`new`关键字）或者使用箭头函数语法定义，则这将不起作用：

```js
// Forcing the value of `this` via `.call()`:
tokyo.sayMyName.call(london); // => Logs "My name is London"
```

在日常函数调用中，最好避免像这样的奇怪调用技术。这些技术可能会使您的代码的读者难以理解发生了什么。使用`call`、`apply`或`bind`进行调用有许多有效的应用，但这些通常局限于较低级别的库或实用程序代码。高级逻辑应该避免使用它们。如果您发现自己在高级抽象中不得不依赖这些方法，那么您可能正在使事情变得比必要的更加复杂。

# super

super 关键字有三种不同的用法：

+   `super()`作为直接函数调用将调用超类的构造函数（即其对象的`[[Prototype]]`构造函数），并且只能在构造函数中调用。它还必须在尝试访问`this`之前调用，因为是`super()`本身将启动执行上下文。

+   `super.property`将访问超类的属性（即`[[Prototype]]`），并且只能在使用方法定义语法定义的构造函数或方法中引用。

+   `super.method()`将调用超类的方法（即`[[Prototype]]`），并且只能在构造函数或使用方法定义语法定义的方法中调用。

`super`关键字是在语言中引入的同时，类定义和方法定义语法也一起引入的，因此它与这些结构有关。您可以在类构造函数、方法以及对象文字中的方法定义中自由使用`super`：

```js
const Utils {
  constructor() {
    super(); // <= I can use super here
  }
  method() {
    super.method(); // <= And here...
  }
}

const utils = {
  method() {
    return super.property; // <= And even here...
  }
};

```

`super`关键字，正如其名称所示，语义上适合引用超类，因此它的 99%有效用例将在类定义中，您希望引用被扩展的类时使用，如下所示：

```js
const Banana extends Fruit {
  constructor() {
    super(); // Call the Fruit constructor
  }
}
```

以这种方式使用`super`是完全直观的，特别是对于习惯于其他面向对象编程语言的程序员。然而，对于精通 JavaScript 原型机制的人来说，`super`的实现可能会令人困惑。与`this`值不同，`super`在定义时绑定，而不是在调用时。我们已经看到了如何通过以特定方式调用方法（例如使用`fn.call()`）来操纵`this`的值。您不能类似地操纵`super`。希望这不会以任何方式影响您，但是记住这一点也是有用的。

# new.target

`new.target`绑定将等于当前被调用的函数，如果函数是通过`new`运算符调用的。我们通常使用`new`运算符来实例化类，在这种情况下，我们将正确地期望`new.target`是该类：

```js
class Foo {
  constructor() {
    console.log(new.target === Foo);
  }
}
new Foo(); // => Logs: true
```

当我们希望在直接调用构造函数与通过`new`调用时执行某种行为时，这是有用的。一个常见的防御策略是使您的构造函数以相同的方式行为，无论是使用还是不使用`new`调用。这可以通过检查`new.target`来实现：

```js
function Foo() {
  if (new.target !== Foo) {
    return new Foo();
  }
}

new Foo() instanceof Foo; // => true
Foo() instanceof Foo;     // => true
```

或者，您可能希望抛出错误以检查构造函数是否被错误调用：

```js
function Foo() {
  if (new.target !== Foo) {
    throw new Error('Foo is a constructor: please instantiate via new Foo()');
  }
}

Foo() instanceof Foo; // !! Error: Foo is a constructor: please instantiate via new Foo()
```

这两个示例都被认为是`new.target`的直观用例。当然，也有可能根据调用模式提供完全不同的功能，但为了满足程序员的合理期望，最好避免这种行为。记住 POLA。

# arguments

`arguments`绑定作为一个类似数组的对象提供，并且将包含给定函数调用时使用的参数。

当我们说`arguments`类似于数组时，我们指的是它具有`length`属性和从零开始索引的属性（就像普通的`Array`一样），但它仍然只是一个普通的`Object`，因此没有任何数组的内置方法可用，例如`forEach`、`reduce`和`map`。

在这里，我们可以观察到参数是在给定函数的范围内提供的：

```js
function sum() {
  arguments; // => [1, 2, 3, 4, 5] (Array-like object)
  let total = 0;
  for (let n of arguments) total += n;
  return total;
}

sum(1, 2, 3, 4, 5);
```

`arguments`绑定曾经被广泛用于访问任意（即非固定）数量的参数，尽管在语言引入了*rest 参数*语法（`...arg`）后，其实用性迅速消失。这种更新的语法可以在定义函数时使用，指示 JavaScript 将剩余参数放入一个数组中。这意味着您可以实现所有旧的`arguments`绑定的实用性，而且您将获得一个不仅仅是类似数组而且实际上是真正数组的值。以下是一个示例：

```js
function sum(...numbers) {
  // We can call reduce() on our array:
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3, 4, 5);
```

尽管`arguments`对象已经不再流行，但它仍在语言规范中，并在旧环境中工作，因此你可能仍然会在实际中看到它。大多数情况下，可以避免使用它。

# 函数名称

令人困惑的是，函数有名称，这些名称与我们分配给函数的变量或属性不同。函数的名称在其括号之前的语法中：

```js
function nameOfTheFunction() {}
```

您可以通过其`name`属性访问函数的名称：

```js
nameOfTheFunction.name; // => "nameOfTheFunction"
```

当您通过函数声明语法定义函数时，它将将该函数分配给同名的局部变量，这意味着我们可以像预期的那样引用该函数：

```js
function nameOfTheFunction() {}
nameOfTheFunction; // => the function
nameOfTheFunction.name; // => "nameOfTheFunction"
```

方法定义也将方法分配给等于函数名称的属性名称：

```js
function nameOfTheFunction() {}
nameOfTheFunction; // => the function
nameOfTheFunction.name; // => "nameOfTheFunction"
```

你可能会认为这一切看起来非常直观。的确如此。我们给函数和方法的名称本身用来指示那些东西将被分配给什么变量或属性是完全合理的。然而，奇怪的是，也可以有命名函数表达式，而这些名称并不会导致这样的分配。以下是一个例子：

```js
const myFunction = function hullaballoo() {}
```

这里的`const`名称`myFunction`决定了我们将在随后的行中用来引用函数。然而，函数在技术上有一个名为`"hullaballoo"`的名称：

```js
myFunction; // => the function
myFunction.name; // => "hullaballoo"
```

如果我们尝试通过其正式名称引用函数，将会出错：

```js
hullaballoo; // !! ReferenceError: hullaballoo is not defined
```

这可能看起来很奇怪。如果函数的名称本身不用于引用函数，为什么可以给函数命名？这是一种遗留和便利的混合。命名函数表达式的一个隐藏特性是，名称实际上是可用于引用函数的，但只能在函数本身的范围内：

```js
const myFunction = function hullaballoo() {
  hullaballoo; // => the function
};
```

这在您想要为某个其他函数提供一个*匿名*回调，但仍然能够引用您自己的回调以进行任何重复或递归调用的情况下非常有用，如下所示：

```js
[
  ['chris', 'smith'],
  ['sarah', ['talob', 'peters']],
  ['pam', 'taylor']
].map(function capitalizeNames(item) { 
  return Array.isArray(item) ?
    item.map(capitalizeNames) :
    item.slice(0, 1).toUpperCase() + item.slice(1);
});

// => [["Chris","Smith"],["Sarah",["Talob", "Peters"]],["Pam","Taylor"]]
```

因此，即使命名函数表达式是一件奇怪的事情，它也有其优点。然而，在使用时，最好考虑到你的代码对于可能不了解这些特殊行为的人的清晰度。这并不意味着完全避免它，而只是在使用时更加注意代码的可读性。

# 函数声明

函数声明是一种*提升声明*。提升声明是在运行时有效地将其提升到其执行上下文的顶部，这意味着它将立即对前面的代码行可访问（看起来是*在*它声明之前）：

```js
hoistedDeclaration(); // => Does not throw an error...

function hoistedDeclaration() {}
```

当然，这对于分配给变量的*函数表达式*是不可能的：

```js
regularFunctionExpression();
  // => Uncaught ReferenceError:
  // => Cannot access 'regularFunctionExpression' before initialization

const regularFunctionExpression = function() {};
```

函数声明的变量提升行为可能会产生意想不到的结果，因此通常被认为是依赖提升的反模式。一般来说，使用函数声明是可以的，只要以程序员直观假设的方式使用。提升作为一种实践，对大多数人来说并不直观，因此最好避免使用它。

有关作用域和函数声明的变量提升发生的更多信息，请查看第九章，*语法和作用域的部分*，并转到*作用域和声明*部分。

# 函数表达式

函数表达式是最容易和最可预测的使用，因为它们在语法上类似于 JavaScript 中的所有其他值。您可以使用它们*字面上*在任何定义其他值的地方定义函数，因为它们是一种表达式。例如，观察这里，我们如何定义一个函数数组：

```js
const arrayOfFunctions = [
  function(){},
  function(){}
];
```

函数表达式的常见应用是将回调传递给其他函数，以便它们可以在以后的某个时间点被调用。许多原生的`Array`方法，如`forEach`，以这种方式接受函数：

```js
[1, 2, 3].forEach(function(value) { 
  // do something with each value
});
```

在这里，我们将一个函数表达式传递给`forEach`方法。我们没有通过将其分配给变量来命名此函数，因此它被视为匿名函数。匿名函数很有用，因为这意味着我们不需要预先将函数分配给变量以便使用它；我们可以简单地将我们的函数写入我们代码的确切位置。

函数表达式在表达方式上与箭头函数最相似。我们将发现，关键的区别在于箭头函数无法访问自己的绑定（例如`this`或`arguments`）。然而，函数表达式可以访问这些值，在某些情况下对你更有用。例如，通常需要绑定到`this`以便成功地与 DOM API 进行操作，例如，许多原生 DOM 方法将使用相关元素作为执行上下文调用回调和事件处理程序。此外，当定义对象或原型上需要访问当前实例的方法时，您将希望使用函数表达式。正如这里所示，使用箭头函数是不合适的：

```js
class FooBear {
  name = 'Foo Bear';
}

FooBear.prototype.sayHello = () => `Hello I am ${this.name}`;
new FooBear().sayHello(); // => "Hello I am ";

FooBear.prototype.sayHello = function() {
  return `Hello I am ${this.name}`;
};
new FooBear().sayHello(); // => "Hello I am Foo Bear";
```

正如你所看到的，使用箭头函数语法阻止我们通过`this`访问实例，而函数表达式语法允许我们这样做。因此，尽管箭头函数在某种程度上已被更简洁的箭头函数取代，但它仍然是一个非常有用的工具。

# 箭头函数

在许多方面，箭头函数只是函数表达式的略微更简洁的版本，尽管它确实有一些实际上的区别。它有两种风格：

```js
// Regular Arrow Function
const arrow = (arg1, arg2) => { return 123; };

// Concise Arrow Function
const arrow = (arg1, arg2) => 123;
```

正如你所看到的，*简洁*变体包括一个隐式返回，而*常规*变体，就像其他函数定义样式一样，需要你定义一个由大括号限定的常规函数体，在其中你必须明确使用`return`语句返回一个值。

此外，箭头函数允许您在声明只有一个参数的函数时避免使用括号。在这些情况下，您可以在箭头之前只放置参数的标识符，如下所示：

```js
const addOne = n => n + 1;
```

箭头函数的简洁性在需要频繁传递函数的情况下非常有用。例如，在通过`map`等原生方法操作数组时很常见：

```js
[1, 2, 3]
  .map(n => n*2)
  .map(n => `Number ${n}`);

// => ["Number 2", "Number 4", "Number 6"]
```

尽管箭头函数作为通常冗长的函数定义的简洁变体具有超级英雄的地位，但它也带来了自己的挑战。语言必须适应*简洁*和*常规*语法变体，这意味着在尝试从简洁的箭头函数中返回对象字面量时存在一些歧义：

```js
const giveMeAnObjectPlease = () => { name: 'Gandalf', age: 2019 };
// !! Uncaught SyntaxError: Unexpected token `:`
```

这种语法会让 JavaScript 解析器困惑，因为开放的大括号意味着一个常规函数体存在。因此，解析器会给出一个意外标记的错误，因为它不期望对象字面量的主体。如果我们想要从箭头函数的简洁形式返回一个对象字面量，那么我们必须笨拙地用括号将其包裹起来以消除歧义的语法：

```js
const giveMeAnObjectPlease = () => ({ name: 'Gandalf', age: 2019 });
```

从功能上讲，箭头函数与函数表达式有两种不同之处：

+   它不提供访问诸如`this`或`arguments`之类的绑定。

+   它没有`prototype`属性，因此不能用作构造函数

这些差异意味着，总的来说，箭头函数通常不适合用作方法或构造函数。它们最适合用于希望将回调或处理程序传递给另一个函数的上下文中，特别是在希望保留`this`绑定的情况下。例如，如果我们想要在`UIComponent`抽象的上下文中绑定事件处理程序，我们可能希望保留`this`值以执行某些特定于实例的功能：

```js
class MyUIComponent extends UIComponent {
  constructor() {
    this.bindEvents({
      onClick: () => {
        this; // <= usefully refers to the MyUIComponent instance
      }
    });
  }
}
```

箭头函数在这种情况下感觉最自然。然而，它的简洁性意味着在阅读过于密集的代码行时可能会产生混淆，比如下面的例子：

```js
process(
  n=>n.filter((nCallback, compute)=>compute(()=>nCallback())
)
```

因此，最好以与使用任何其他构造相同的考虑和实用性来使用箭头函数：确保始终将代码的可用性和可读性放在首位，而不是非常诱人的*酷*或简洁语法的巧妙性。

# 立即调用函数表达式

函数表达式和箭头函数是唯一的函数定义样式，从技术上讲，它们是表达式。正如我们所见，这种特性使它们在需要将它们作为值传递给其他函数时非常有用，而无需经历赋值的过程。

正如我们之前提到的，没有赋值的函数，因此没有对其值的引用，通常被称为*匿名函数*，看起来像这样：

```js
(function() {
  // I am an anonymous function
})
```

*匿名函数*的概念通过**立即调用函数表达式**（**IIFE**）的概念进一步扩展。IIFE 只是一个立即调用的常规*匿名函数*，如下所示：

```js
(function() {
  // I am immediately invoked
}());
```

注意在闭合大括号后的调用括号（也就是`...()`）。这将调用函数，因此使前面的语法结构成为 IIEE。

IIFE 在语言本身并不是一个独特的概念。它只是社区提出的一个有用术语，用来描述*立即*调用函数的常见模式。这是一个有用的模式，因为它允许我们创建一个临时作用域，这意味着在其中定义的任何变量都受到该作用域的限制，不会泄漏到外部，就像我们从任何函数中期望的那样。这种立即作用域对于快速进行自包含工作而不影响父作用域非常有用。

在浏览器时代，IIFE 变得流行起来，因为最好避免污染全局命名空间。然而，如今，预编译如此流行，IIFE 的用处就不那么大了。

IIFE 的确切语法可能会有所不同。例如，如果我们使用箭头函数，那么调用括号必须放在包装的函数表达式之后：

```js
(() => {
  // I am immediately invoked
})(); // <- () actually calls the function
```

无论我们使用函数表达式还是箭头函数，机制本质上都是相同的。

如果 IIFE 的概念令人困惑，那么如果我们用标识符`fn`替换实际函数，并想象我们之前已经将一个函数分配给了这个标识符，那么理解正在发生的事情就更简单了。在这里，我们可以这样调用`fn`：

```js
fn();
```

现在，我们可以选择用括号包裹`fn`引用。这对调用没有任何影响，尽管看起来可能很奇怪：

```js
(fn)();
```

值得记住的是，括号只是有时需要的语法容器，以避免语法歧义。因此，所有这些从技术上讲都是等价的：

```js
fn();
(fn)();
((fn))();
```

如果我们在这里用内联匿名函数替换`fn`引用，就不会发生什么突破性的事情。我们只是在现场表达一个内联函数，然后调用它，而不是引用现有的函数：

```js
(function() {
  // Called immediately...
})();
```

我们称内联函数表达式的模式为 IIFE，但它实际上并不特别。考虑到调用括号，也就是`...()`，实际上并不在乎它们附加到什么上，只要它是一个函数。在调用之前的表达式可以是任何东西，只要它求值为一个函数。

IIFE 很有用，因为它们提供了作用域隔离，而无需定义一个带有名称的函数，然后稍后引用和调用它，就像我们在这里做的一样：

```js
const initializeApp = () => {
  // Initializing...
};

initializeApp();
```

在浏览器中，在涉及编译和捆绑的复杂构建之前，IIFE 很有用，因为它们提供了作用域隔离，同时不会将任何名称泄漏到全局作用域。然而，如今，IIFE 很少是必要的。

有趣的是，前面代码中的`initializeApp`函数，可以说，通过显式名称更易读和理解。这就是为什么，即使必要，IIFE 有时被认为是不必要的混乱和花哨。有名字的函数有助于提供关于其目的和作者意图的线索。没有名称，我们的代码读者就必须承担阅读函数本身以发现其广泛目的的认知负担。因此，通常最好避免 IIFE 和类似的匿名结构，除非您有非常特定的需求。

# 方法定义

方法定义是在与类定义同时添加到语言中的，允许您轻松声明绑定到特定对象的方法。但它们不仅限于类定义。您也可以在对象文字中自由使用它们：

```js
const things = {
  myFunction() {
    // ...
  }
};
```

在类中，您也可以以这种方式声明方法：

```js
class Things {
  myFunction() {
    // ...
  }
}
```

您还可以使用传统的函数定义样式来声明您的方法，比如将函数表达式分配给一个标识符：

```js
class Things {
  myFunction = function() {
    // ...
  };
}
```

然而，方法定义和其他函数定义风格之间存在一个关键区别。方法定义将始终绑定到首次定义它的对象。这在内部被称为它的`[[HomeObject]]`。这个主对象将确定方法在被调用时可用的`super`绑定。只有方法定义允许引用`super`，它们引用的`super`将始终是它们的`[[HomeObject]]`的`[[Prototype]]`。这意味着，如果您尝试从其他对象*借用*方法，您可能会惊讶地发现`super`不是您想要的：

```js
class Dog {
  greet() { return 'Bark!'; }
}

class Cat {
  greet() { return 'Meow!'; }
}

class JessieTheDog extends Dog {
  greet() { return `${super.greet()} I am Jessie!`; }
}

class JessieTheCat extends Cat {
  greet() { return `${super.greet()} I am Jessie!`; }
}
```

在这里，我们可以观察到`JessieTheCat`和`JessieTheDog`都有`greet`方法：

```js
new JessieTheDog().greet(); // => "Bark! I am Jessie!"
new JessieTheCat().greet(); // => "Meow! I am Jessie!"
```

我们还可以观察到它们的 greet 方法以相同的方式实现。它们都返回插值字符串``${super.greet()} I am Jessie!``。因此，让`JessieTheCat`从`JessieTheDog`借用该方法似乎是合乎逻辑的。毕竟，它们完全相同：

```js
class JessieTheCat extends Cat {
  greet = JessieTheDog.prototype.greet
}
```

我们可能直觉地期望`greet`方法中的`super`指的是当前实例的超类，在`JessieTheCat`的情况下将是`Cat`。但奇怪的是，当我们调用这个借用的方法时，我们会经历一些不同的东西：

```js
new JessieTheCat().greet(); // => "Bark! I am Jessie!"
```

它会叫！借用的方法令人讨厌地保留了它对原始`[[HomeObject]]`的绑定。

总之，方法定义是更简洁的变体，比起它们更冗长的表亲，函数声明和函数表达式。然而，它们具有一个将它们与众不同的隐式机制，可能会引起混淆。99%的时间，方法定义不会让你失望；它们会表现如预期。另外的 1%的时间，至少知道为什么您的代码表现不佳是有用的，这样您就可以探索其他选项。就像往常一样，对 JavaScript 的特殊性的了解只能帮助我们追求更清洁和更可靠的代码库。

# 异步函数

**异步**（**async**）函数在函数关键字之前用`async`关键字指定。所有函数定义样式都可以以它为前缀：

```js
// Async Function Declaration:
async function foo() {}

// Async Function Expression:
const foo = async function() {};

// Async Arrow-Function:
const foo = async () => {};

// Async Method Definition:
const obj = {
  async foo() {}
};
```

异步函数允许您通过提供两个关键功能轻松进行异步操作：

+   您可以在异步函数中使用`await`来等待 Promise 的完成

+   您的函数将始终返回一个 Promise，它本身可以被等待

Promise 是处理异步操作的本地提供的抽象。它可能看起来很复杂，但最好将 Promise 视为一个对象，它将在比*现在*更晚的时间解析或拒绝（即异步）。

传统上，在 JavaScript 中，我们必须传递回调函数，以确保我们能够响应这种异步活动：

```js
getUserDetails('user1', function(userDetails) {
  // This callback is called asynchronously
});
```

然而，通过异步函数和`await`，我们可以更简洁地实现这一点：

```js
const userDetails = await getUserDetails('user1');
```

这里的`await`子句将暂停当前执行，直到`getUserDetails`完成并解析为一个值。请注意，我们只能在自身是异步的函数中使用 await。

异步执行是一个复杂的话题，因此有一个专门的章节，即第十章，*控制流*。现在，有必要知道异步函数是一种特殊类型的函数，它将始终返回一个 Promise。

除了允许`await`子句和返回 Promise 之外，异步函数具有与所使用的相应函数定义样式相同的特性和特征。异步箭头函数，就像常规箭头函数一样，没有自己的 this 或 arguments 绑定。异步函数声明像它的非异步表亲一样被提升。基本上，异步应该被视为覆盖您已经掌握的关于不同函数定义样式的所有知识的一层。

# 生成器函数

我们将要介绍的最后一种函数定义样式是非常强大的*生成器函数*。广义上，生成器用于提供和控制一个或多个，甚至无限个项目的迭代行为。

在 JavaScript 中，*生成器函数*是在函数关键字后面加上一个星号来指定的：

```js
function* myGenerator() {...}
```

当调用时，它们将返回一个生成器对象，该对象唯一地符合可迭代协议和迭代器协议，这意味着它们可以被自身迭代，或者可以作为对象的迭代逻辑。

可以直接跳到关于可迭代协议的部分。*当您将生成器函数视为创建迭代器或可迭代对象的便捷方式时，它会更加有意义。

生成器函数将在`yield`语句的位置暂停并返回一个值，这可以发生多次。在`yield`之后，函数在等待消费者需要其下一个值时实际上被暂停了。这最好通过一个例子来说明：

```js
function* threeLittlePiggies() {
  yield 'This little piggy went to market.';
  yield 'This little piggy stayed home.';
  yield 'This little piggy had roast beef.';
}

const piggies = threeLittlePiggies();

piggies.next().value; // => 'This little piggy went to market.'
piggies.next().value; // => 'This little piggy stayed home.'
piggies.next().value; // => 'This little piggy had roast beef.'

piggies.next(); // => {value: undefined, done: true}
```

正如你所看到的，从函数返回的生成器对象具有`next`方法，当调用时，将返回一个带有`value`（指示迭代的当前值）和`done`属性（指示迭代/生成是否完成）的对象。这是*迭代器协议*，也是您可以期望所有生成器满足的约定。

生成器不仅满足迭代器协议，还满足可迭代协议，这意味着它们可以被语言结构迭代（例如`for...of`或`...spread`运算符）接受：

```js
for (let piggy of threeLittlePiggies()) console.log(piggy); 
// => Logs: "This little piggy went to market."
// => Logs: This little piggy stayed home."
// => Logs: This little piggy had roast beef."

[...threeLittlePiggies()];
// => ["This little piggy went to market", "This little piggy stayed...", "..."]
```

*异步生成器函数*也可以被指定。它们有用地将异步和生成器格式结合成一种混合形式，允许自定义异步生成逻辑，就像这样：

```js
async function* pages(n) {
  for (let i = 1; i <= n; i++) {
    yield fetch(`/page/${i}`);
  }
};

// Fetch five pages (/page/1, /page/2, /page/3)
for await (let page of pages(3)) {
  page; // => Each of the 3 pages
};
```

您会注意到我们正在使用`for await`迭代结构来迭代我们的异步生成器。这将确保每次迭代都会在继续之前等待其结果。

生成器函数非常强大，但重要的是要了解其中的基本机制。它们不是常规函数，也不能保证完全运行。它们的实现应该考虑它们将被运行的上下文。如果您的生成器旨在用作迭代器，则它应该尊重迭代的暗示期望：它是对底层数据或生成逻辑的只读操作。虽然可能在生成器内部改变底层数据，但应该避免这样做。

# 数组和可迭代对象

在 JavaScript 中，数组是一种特殊的对象类型，它包含一组有序的元素。

您可以使用数组的文字语法来表示一个数组，这是一个由方括号分隔的表达式的逗号分隔列表：

```js
const friends = ['Rachel', 'Monica', 'Ross', 'Joe', 'Phoebe', 'Chandler'];
```

这些逗号分隔的表达式可以是复杂或简单的，取决于我们的需要。

```js
[
  [1, 2, 3],
  function() {},
  Symbol(),
  {
    title: 'wow',
    foo: function() {}
  }
]
```

数组能够包含各种值。我们对如何使用数组几乎没有什么限制。从技术上讲，数组的`length`由于存储为 32 位整数，因此受到约 40 亿的限制。当然，对于大多数目的来说，这应该完全没问题。

数组具有用于描述其中每个索引元素的数值属性和一个`length`属性来描述其中有多少元素。它们还有一组有用的方法来读取和操作其中的数据：

```js
friends[0]; // => "Rachel"
friends[5]; // => "Chandler"
friends.length; // => 6

friends.map(name => name.toUpperCase());
// => ["RACHEL", "MONICA", "ROSS", "JOE", "PHOEBE", "CHANDLER"]

friends.join(' and ');
// => "Rachel and Monica and Ross and Joe and Phoebe and Chandler"
```

在历史上，数组是通过传统的`for(...)`和`while(...)`循环进行迭代的，这些循环会向`length`递增一个计数器，以便在每次迭代时可以通过`array[counter]`访问当前元素，如下所示：

```js
for (let i = 0; i < friends.length; i++) {
  // Do something with `friends[i]`
}
```

然而，如今更倾向于使用其他迭代方法，比如`forEach`或`for...of`：

```js
for (let friend of friends) {
  // Do something with `friend`
}

friends.forEach((friend, index) => {
  // Do something with `friend`
});
```

`for...of`的好处是可以中断，这意味着你可以在其中使用`break`和`continue`语句，并轻松地跳出迭代。它还可以用于任何可迭代的对象，而`forEach`只是一个`Array`方法。然而，`forEach`风格的迭代对于通过回调的第二个参数提供当前迭代的索引是有用的。

你使用哪种迭代方式应该由你要迭代的值和你希望在每次迭代中执行的操作决定。如今，几乎不需要使用传统的数组迭代方式，比如`for(...)`和`while(...)`。

# 类似数组的对象

大多数原生数组方法都是通用的，这意味着它们可以用于任何*看起来像*数组的对象。我们只需要实现一个`length`属性和每个索引的单独属性（从零开始索引）来实现数组的外观：

```js
const arrayLikeThing = {
  length: 3,
  0: 'Suspiciously',
  1: 'similar to',
  2: 'an array...'
};

// We can "borrow" an array's join method by assigning 
// it to our object:
arrayLikeThing.join = [].join;

arrayLikeThing.join(' ');
// => "Suspiciously similar to an array..."
```

在这里，我们构建了一个类似数组的对象，然后通过借用数组的`join`方法（即从`Array.prototype`中）为其提供了自己的`join`方法。原生数组的`join`方法实现得非常通用，它不介意在对象上操作，只要该对象满足数组的约定，即提供一个`length`属性和相应的索引（`0`，`1`，`2`等）。大多数原生数组方法都是通用的。

语言本身中类似数组的对象的一个例子是我们在本章前面探讨过的`arguments`绑定。另一个例子是`NodeList`，它是从各种 DOM 选择方法返回的对象类型。如果需要，我们可以通过借用和调用数组的`slice`方法从这些对象中派生出真正的数组，如下所示：

```js
const arrayLikeObject = { length: 2, 0: 'foo', 1: 'bar' };

// "Borrowing" a method from an array and forcing its
// execution context via call():
[].slice.call(arrayLikeObject);

// "Borrowing" a method explicitly from the Array.prototype
// and forcing its execution context via call():
Array.prototype.slice.call(arrayLikeObject);
```

然而，在`arguments`或`NodeList`对象的情况下，我们也可以依赖它们是可迭代的，这意味着我们可以使用扩展语法来派生出一个真正的数组：

```js
// "spread" a NodeList into an Array:
[...document.querySelectorAll('div span a')];

// "spread" an arguments object into an Array:
[...arguments];
```

如果你发现自己需要创建一个类似数组的对象，考虑让它实现可迭代协议（我们即将探讨），以便以这种方式使用扩展语法。

# Set 和 WeakSet

`Set`和`WeakSet`是允许我们存储唯一对象序列的原生抽象。这与数组形成对比，数组无法保证值的唯一性。下面是一个例子：

```js
const foundNumbersArray = [1, 2, 3, 4, 3, 2, 1];
const foundNumbersSet = new Set([1, 2, 3, 4, 3, 2, 1]);

foundNumbersArray; // => [1, 2, 3, 4, 3, 2, 1]
foundNumbersSet;   // => Set{ 1, 2, 3, 4 }
```

如你所见，给定给`Set`的值如果已经存在于`Set`中，将始终被忽略。

通过将可迭代的值传递给构造函数，可以初始化集合；例如，一个字符串：

```js
new Set('wooooow'); // => Set{ 'w', 'o' }
```

如果你需要将`Set`转换为数组，你可以使用扩展语法最简单地实现这一点（因为集合本身是可迭代的）：

```js
[...foundNumbersSet]; // => [1, 2, 3, 4]
```

WeakSet 与之前介绍的 WeakMap 类似。它们用于以一种允许值在程序的其他部分被垃圾回收的方式*弱引用*值。使用集合的语义和最佳实践与使用数组的类似。建议只在需要存储唯一值序列时使用集合；否则，只需使用简单的数组。

# 可迭代协议

可迭代协议允许包含序列的值共享一组共同的特征，使它们可以被迭代或以类似的方式处理。

我们可以说，实现可迭代协议的对象是可迭代的。JavaScript 中的可迭代对象包括`Array`、`Map`、`Set`和`String`。

任何对象都可以通过简单地在属性名`Symbol.iterator`下提供迭代器函数来定义自己的可迭代协议（它映射到内部的`@@iterator`属性）。

这个迭代器函数必须通过返回一个带有`next`函数的对象来满足迭代器协议。当调用这个`next`函数时，它必须返回一个带有`done`和`value`键的对象，指示迭代的当前值和迭代是否完成：

```js
const validIteratorFunction = () => {
  return {
    next: () => {
      return {
        value: null, // Current value of the iteration
        done: true // Whether the iteration is completed
      };
    }
  }
};
```

因此，为了对此有绝对的清晰认识，有两个不同的协议：

+   **可迭代协议**：通过`[Symbol.iterator]`实现`@@iterator`的任何对象都满足这个协议。原生示例包括`Array`、`String`、`Set`和`Map`。

+   **迭代器协议**：任何返回形式为`{... next: Function}`的对象的函数，其`next`方法在调用时返回以下形式的对象：`{value: Boolean, done: ...}`。

为了满足可迭代协议，对象必须实现`[Symbol.iterator]`，如下所示：

```js
const zeroToTen = {};
zeroToTen[Symbol.iterator] = function() {
  let current = 0;
  return {
    next: function() {
      if (current > 10) return { done: true };
      return {
        done: false,
        value: current++
      };
    }
  }
};

// We can see the effect of the iterable via the spread operator:
[...zeroToTen]; // => [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

通过可迭代协议提供自定义迭代方法在您想要控制迭代顺序或在迭代过程中进行某种处理、过滤或生成值时非常有用。例如，在这里，我们将迭代器函数指定为生成器函数，正如您可能记得的那样，它返回一个满足*迭代器*和可迭代协议的生成器。这个生成器函数将为每个存储的单词产生两个变体，一个大写和一个小写：

```js
const words = {
  values: ['CoFfee', 'ApPLE', 'Tea'],
  [Symbol.iterator]: function*() {
    for (let word of this.values) {
      yield word.toUpperCase();
      yield word.toLowerCase();
    }
  }
};

[...words]
// => ["COFFEE", "coffee", "APPLE", "apple", "TEA", "tea"]
```

将迭代器函数指定为生成器函数比手动实现迭代器协议要简单得多。生成器自然地满足这一约定，因此它们可以更加无缝地使用。生成器通常也更易读和简洁，并且具有实现迭代器和可迭代协议的双重好处，这意味着它们可以用于为对象添加迭代功能：

```js
const someObject = {
  [Symbol.iterator]: function*() { yield 123; }
};

[...someObject]; // => [123]
```

它们本身也可以提供迭代功能：

```js
function* someGenerator() {
  yield 123;
}

[...someGenerator()]; // => [123]
```

重要的是要记住，在自定义可迭代中进行的任何工作都应符合消费者的期望。迭代通常被认为是只读操作，因此在迭代过程中应避免对基础值集的突变。实现自己的可迭代可以非常强大，但也可能导致消费者对您的自定义迭代逻辑不熟悉的意外行为。

对于那些了解情况的人和那些可能是第一次体验您的接口或抽象的人来说，平衡自定义迭代的便利性是非常重要的。

# RegExp

JavaScript 通过对象类型`RegExp`原生支持正则表达式，允许通过文字语法`/foo/`或直接通过构造函数(`RegExp('foo')`)来表达。正则表达式用于定义可以与字符串匹配或执行的字符模式。

以下是一个示例，我们从文本语料库中提取只有长单词（`>=10`个字符）的单词：

```js
const string = 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam sit amet odio ultrices nunc efficitur venenatis laoreet nec leo.';

string.match(/\w{10,}/g); // => ["consectetur", "adipiscing"]
```

正则表达式的语法和语法可能很复杂。从技术上讲，它本质上是一种完整的语言，需要多天的学习。我们无法在这里探索它的所有复杂性。然而，我们将涵盖我们通常在 JavaScript 中操作正则表达式的方式，并探讨其中的一些挑战。建议您自行进一步研究正则表达式。

# 正则表达式 101

正则表达式允许我们描述字符的模式。它们用于匹配和提取字符串中的值。例如，如果我们有一个包含数字`(1,2,3)`的字符串，正则表达式将允许我们轻松地检索它们：

```js
const string = 'some 1 content 2 with 3 digits';
string.match(/1|2|3/g); // => ["1", "2", "3"]
```

正则表达式是以斜杠分隔的模式编写的，最终斜杠后面可以跟随可选标志：

```js
/[PATTERN]/[FLAGS]
```

您编写的模式可以包含文字和特殊字符，这些字符共同告诉正则表达式引擎要查找什么。我们在示例中使用的正则表达式包含文字字符（即`1`、`2`、`3`）和管道（即`|`）特殊字符：

```js
/1|2|3/g
```

管道特殊字符告诉正则表达式引擎，管道左侧或右侧的字符可能匹配。在最终斜杠后面的`g`是一个*全局*标志，指示引擎在字符串中全局搜索，并在找到第一个匹配项后不放弃。对我们来说，这意味着我们的正则表达式将匹配主题字符串中出现的`"1"`、`"2"`或`"3"`。

在正则表达式中，我们可以使用特定的特殊字符作为快捷方式。`[0-9]`的表示法就是一个例子。它是一个*字符类*，将匹配从`0`到`9`的所有数字，这样我们就不必逐个列出所有这些数字。还有一个*简写字符类*`\d`，可以更简洁地表示这一点。因此，我们可以将我们的正则表达式缩短为以下形式：

```js
/\d/g
```

对于更现实的应用，我们可以想象一种情况，我们希望匹配数字序列，比如电话号码。也许我们只希望匹配以`0800`开头并包含进一步`4`到`6`位数字的电话号码。我们可以使用以下正则表达式来实现：

```js
/0800\d{4,6}/g
```

在这里，我们使用`{n, n}`语法，允许我们为前面的特殊字符`\d`表示数量。我们可以通过将其传递给测试字符串的`match`方法来确认我们的模式是否有效：

```js
`
  This is a test in which exist some phone
  numbers like 0800182372 and 08009991.
`.match(
  /0800\d{4,6}/g
);
// => ["0800182372", "08009991"]
```

这个简短的介绍只是触及了正则表达式的表面。正则表达式的语法允许我们表达重要的复杂性，使我们能够验证字符串中是否存在特定文本，或者提取特定文本以在我们的程序中使用。

# RegExp 标志

正则表达式的文字语法允许在最终的斜杠之后指定特定的*标志*，例如`i`（*忽略大小写*）。这些标志将影响正则表达式的执行方式：

```js
/hello/.test('hELlO');  // => false
/hello/i.test('hELlO'); // => true
```

在使用`RegExp`构造函数时，可以将标志作为第二个参数传递：

```js
RegExp('hello').test('hELlO');      // => false
RegExp('hello', 'i').test('hELlO'); // => true
```

JavaScript 的正则表达式有六个可用的标志：

+   `i`：*忽略大小写*标志将在匹配字母时忽略字符串的大小写（即`/a/i`将匹配字符串中的`'a'`和`'A'`）。

+   `g`：*全局匹配*标志将使正则表达式找到*所有*匹配项，而不是在找到第一个匹配项后停止。

+   `m`：*多行*标志将使开始和结束锚点（即`^`和`$`）标记单独行的开头和结尾，而不是整个字符串的开头和结尾。

+   `s`：*dotAll*标志将使正则表达式中的点字符（通常仅匹配非换行符字符）匹配换行符字符。

+   `u`：*Unicode*标志将把正则表达式中的字符序列视为单独的 Unicode 代码点，而不是代码单元。这基本上意味着您可以轻松地匹配和测试罕见或特殊符号，例如表情符号（请参阅本章中关于`String`类型的部分，以更全面地了解 Unicode）。

+   `y`：*粘性*标志将导致所有`RegExp`操作尝试在`lastIndex`属性指定的确切索引处进行匹配，然后在匹配时改变`lastIndex`。

正如我们所见，正则表达式也可以通过`RegExp`构造函数构造。这可以有用地作为构造函数或常规函数调用：无论哪种方式，你都会收到一个等同于从文字语法中派生的`RegExp`对象：

```js
new RegExp('[a-z]', 'i'); // => /[a-z]/i
RegExp('[a-z]', 'i');     // => /[a-z]/i
```

这是一种非常独特的行为。事实上，`RegExp`构造函数是唯一可以作为构造函数和常规函数调用的本地提供的构造函数，在这两种情况下都返回一个新实例。你会记得，原始构造函数（如`String`和`Number`）可以作为常规函数调用，但在作为构造函数调用时会有不同的行为。

# 接受 RegExp 的方法

JavaScript 提供了七种方法，可以利用正则表达式：

+   `RegExp.prototype.test(String)`: 对传递的字符串运行正则表达式，如果找到至少一个匹配，则返回 true。如果没有找到匹配，则返回 false。

+   `RegExp.prototype.exec(String)`: 如果正则表达式有全局（`g`）标志，那么`exec()`将从当前`lastIndex`返回下一个匹配（并在这样做后更新正则表达式的`lastIndex`）；否则，它将返回正则表达式的第一个匹配（类似于`String.prototype.match`）。

+   `String.prototype.match(RegExp)`: 这个`String`方法将返回针对字符串进行的匹配（或者如果设置了全局标志，则返回所有匹配）。

+   `String.prototype.replace(RegExp, Function)`: 这个`String`方法将在每次匹配上执行传递的函数，并且对于每次匹配，用函数返回的内容替换匹配的文本。

+   `String.prototype.matchAll(RegExp)`: 这个`String`方法将返回所有结果及其各自的组的迭代器。当你有一个具有各自匹配组的全局正则表达式时，这是很有用的。

+   `String.prototype.search(RegExp)`: 这个`String`方法将返回第一个匹配的索引，如果没有找到匹配，则返回-1。

+   `String.prototype.split(RegExp)`: 这个`String`方法将返回一个包含由提供的分隔符（可以是正则表达式）分割的字符串部分的数组。

有许多方法可供选择，但在大多数情况下，你可能会发现`RegExp`方法`test()`和`String`方法`match()`和`replace()`最有用。

以下是一些这些方法的示例。这应该让你了解每种方法可能使用的情况：

```js
// RegExp.prototype.test
/@/.test('a@b.com'); // => true
/@/.test('aaa.com'); // => false

// RegExp.prototype.exec
const regexp = /\d+/g;
const string = '123 456 789';
regex.exec(string); // => ["123"]
regex.exec(string); // => ["456"]
regex.exec(string); // => ["789"]
regex.exec(string); // => null

// String.prototype.match
'Orders: #92838 #02812 #92833'.match(/\d+/);  // => ["92838"]
'Orders: #92838 #02812 #92833'.match(/wo+w/g); // => ["92838", "02812", "92833"]

// String.prototype.matchAll
const string = 'Orders: #92333 <fulfilled> #92835 <pending>';
const matches = [
  ...string.matchAll(/#(\d+) <(\w+)>/g)
];
matches[0][1]; // => 92333
matches[0][2]; // => fulfilled

// String.prototype.replace
'1 2 3 4'.replace(/\d/, n => `<${n}>`); // => "<1> 2 3 4'
'1 2 3 4'.replace(/\d/g, n => `<${n}>`); // => "<1> <2> <3> <4>'

// String.prototype.search
'abcdefghhijklmnop'.search(/k/); // => 11

// String.prototype.split
'time_in____a__tree'.split(/_+/); // ["time", "in", "a", "tree"]

```

正如你所看到的，大多数这些方法的行为都很直观。然而，围绕*粘性*和`lastIndex`属性存在一些复杂性，我们现在将对此进行讨论。

# RegExp 方法和 lastIndex

默认情况下，如果你的`RegExp`是全局的（即使用了`g`标志），`RegExp`方法（即`test()`和`exec()`）将在每次执行时改变`RegExp`对象的`lastIndex`属性。这些方法将尝试从当前`lastIndex`属性指定的索引匹配主题字符串，该属性默认为 0，然后在每次后续调用时更新`lastIndex`。

如果你期望`exec()`或`test()`对于给定的全局正则表达式和字符串始终返回相同的结果，这可能会导致意外行为：

```js
const alphaRegex = /[a-z]+/g;

alphaRegex.exec('aaa bbb ccc'); // => ["aaa"]
alphaRegex.exec('aaa bbb ccc'); // => ["bbb"]
alphaRegex.exec('aaa bbb ccc'); // => ["ccc"]
alphaRegex.exec('aaa bbb ccc'); // => null
```

如果你尝试在多个字符串上执行全局正则表达式而不自己重置`lastIndex`，这也会导致混乱：

```js
const alphaRegex = /[a-z]+/g;

alphaRegex.exec('monkeys laughing'); // => ["monkeys"]
alphaRegex.lastIndex; // => 7
alphaRegex.exec('birds flying'); // => ["lying"]
```

如你所见，在匹配了`"monkeys"`子字符串之后，`lastIndex`被更新为下一个索引（`7`），这意味着，在不同的字符串上执行时，正则表达式将继续之前的位置，并尝试匹配该索引之后的所有内容，在第二个字符串`"birds flying"`的情况下，是子字符串`"lying"`。

通常，为了避免这些混淆，始终拥有对正则表达式的所有权是非常重要的。如果你在程序中使用`RegExp`方法，不要接受来自其他地方的正则表达式。此外，在每次执行之前不要尝试在不同的字符串上使用`exec()`或`test()`而不重置`lastIndex`：

```js
const petRegex = /\b(?:dog|cat|hamster)\b/g;

// Testing multiple strings without resetting lastIndex:
petRegex.exec('lion tiger cat'); // => ["cat"]
petRegex.exec('lion tiger dog'); // => null

// Testing multiple strings with resetting lastIndex:
petRegex.exec('lion tiger cat'); // => ["cat"]
petRegex.lastIndex = 0;
petRegex.exec('lion tiger dog'); // => ["dog"]
```

在这里，你可以看到，如果我们不重置`lastIndex`，我们的正则表达式在传递给`exec()`方法的后续字符串上无法匹配。然而，如果我们在每次后续的`exec()`调用之前重置`lastIndex`，我们将观察到匹配。

# 粘性

粘性意味着正则表达式将尝试在确切的`lastIndex`处进行匹配，如果在该确切索引处找不到匹配，它将失败（即根据使用的方法返回`null`或`false`）。粘性标志（`y`）将强制`RegExp`在每次匹配时读取和改变`lastIndex`。传统的粘性方法，如`exec()`和`test()`，如我们之前提到的，将始终这样做，但`y`标志将*强制*粘性，即使在使用非粘性方法时，如`match()`：

```js
const regexp = /cat|hat/y; // match 'cat' or 'hat'
const string = 'cat in a hat';

// lastIndex is always zero by default, so will
// match from the start of the string:
regexp.lastIndex; // => 0
regexp.test(string); // => ["cat"]

// lastIndex has been modified following the last
// match but will not match anything as there is
// no cat or hat at index 3:
regexp.lastIndex; // => 3
string.match(regexp); // => null

// Set lastIndex to 9 (index of "hat"):
regexp.lastIndex = 9;
string.match(regexp); // => ["hat"]
```

如果你正在寻找字符串或一系列字符串中特定索引处的匹配，粘性可能是有用的。然而，如果你无法完全控制`lastIndex`，它的行为可能会出乎意料。正如我们之前提到的，一个很好的一般规则是始终拥有对自己的`RegExp`对象的所有权，以便对`lastIndex`的任何变化只能由你的代码进行。

# 总结

在本章中，我们已经开始通过查看语言提供的内置类型来深入研究 JavaScript。我们探索的重点是通过清晰的代码视角来看待这些语言构造。通过这样做，我们强调了在处理语言的一些更晦涩的领域时要小心的重要性。我们发现了许多涉及使用 JavaScript 类型的恶劣边缘情况和挑战，比如浮点`Number`类型的精度不足以及`String`类型中 Unicode 的复杂性。探索语言中这些更困难的部分不仅可以避免特定的陷阱，而且可以在我们内心培养一种流利，这将极大地提高我们运用 JavaScript 服务于清晰代码的能力。

在下一章中，我们将继续增强这种流畅性。我们将更多地了解 JavaScript 的类型系统，并开始对这些类型进行操作和操纵以满足我们的需求。
