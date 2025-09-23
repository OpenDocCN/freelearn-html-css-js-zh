# 第五章：使用不同模式的作用域变量

在本章中，我们看到最基本的概念是变量。知道确切类型，从原始类型到对象，对于访问特定成员至关重要。在运行时和设计时缩小确切类型至关重要，以确保两个环境之间的一致性，并了解可能性和不可能性。不同类型变量之间的配置多样性需要许多不同的模式，这些模式在本章中都有介绍。

本章将涵盖以下内容：

+   如何使用 `typeof` 在运行时和设计时进行比较

+   如何确保 `undefined` 和 `null` 的检查

+   我是否需要检查联合的所有可能性以获得正确的类型？

+   `instanceof` 的局限性是什么？

+   为什么区分器对于类型识别是必不可少的

+   为什么使用用户定义的 `guard`

+   如何以及为什么进行类型转换

+   什么是类型断言？

+   如何比较类

+   如何在签名中具有联合类型的函数中缩小类型

# 与 typeof 在运行时和设计时的比较

TypeScript 将类型引入 JavaScript，但这主要是在设计类型时才成立。TypeScript 在编译过程中移除所有类型。这就是生成的代码是纯 JavaScript 并且不包含任何接口或类型痕迹的原因。对 JavaScript 的纯粹尊重使得类型比较更加困难，因为它不能依赖于类型的名称来执行类型检查。然而，我们可以使用所有 JavaScript 的技巧来知道一个值是否来自不同的类型。第一个特性回答了本节关于如何比较运行时和设计类型的主要问题。JavaScript 中存在的 `typeof` 操作符在 TypeScript 中也是以相同的方式工作的。`typeof` 操作符返回原始类型的类型，或者返回 `object`。

使用很简单：调用 `typeof` 后跟您想要比较的变量。大多数时候，您会将其与需要以字符串形式编写的类型名称进行比较：

```js
const a = "test";
let b: number = 2;
const c: boolean = true;
let d: number | string = "test";
console.log(typeof a); // string
console.log(typeof b); // number
console.log(typeof c); // boolean
console.log(typeof d); // string
```

`typeof` 操作符在具有联合类型时特别有用，其中对象可以来自多个原始类型。实际上，它甚至可以与具有复杂对象（接口或类型）的联合一起使用，因为 `typeof` 返回 `object`：

```js
const e: number | string | { complex: string, obj: number } = { complex: "c", obj: 1 };
console.log(typeof e); // object
```

要知道对象是哪种类型，将需要使用我们在本章中将要介绍的其他机制。在继续之前，即使 `typeOf` 可以与字符串比较，操作的结果可以设置一个类型：

```js
let f: number = 2;
if (typeof f === "number") {
console.log("This is for sure a number");
}
type MyNewType = typeof f;
```

注意，`typeOf` 在原始类型上工作，但与 `undefined` 或 `null` 交互时表现得很奇怪。然而，`undefined` 将返回 `undefined`，而 `null` 将返回 `object`。检查 `undefined` 或 `null` 的最佳方法是不使用 `typeof`：

```js
let g: number | undefined = undefined;
let h: number | undefined | null = null;
console.log(typeof g);
console.log(typeof h);
```

# 区分 undefined 和 null

当 `typeof` 对未定义类型执行时，它返回 `undefined` 字符串，而当它对 `null` 执行时，它返回 `object`。这种不一致性在你忘记哪个情况可以使用 `typeof` 通过对错误的 `no type` 类型执行错误操作时成为一个问题。然而，`undefined` 和 `null` 不需要使用 `typeof` 来进行类型检查。可以直接将变量与 `undefined` 或 `null` 进行比较：

```js
let g: number | undefined = undefined;
let h: number | undefined | null = null;
console.log(typeof g); // undefined
console.log(typeof h); // object
console.log(g === undefined); // true
console.log(g === null); // false
console.log(h === undefined); // false
console.log(h === null); // true
```

在一个变量可以是未定义、null 或任何其他原始类型的情况下，最好的方法是检查类型的可空性，并继续进行进一步的类型比较。

# 获取联合类型中的元素类型

TypeScript 的推理系统随着每个版本的更新而变得更好。在最新版本中，TypeScript 使用控制流以智能的方式根据代码的编写方式找出类型。如果一个检查在一个代码路径中执行，TypeScript 就知道对于类型验证的闭包，类型就是检查过的。如果一个 `else` 代码路径存在到类型检查，它知道它是类型比较的反面。

以下代码示例显示，根据执行的位置，类型会发生变化。它最初是数字或未定义。对 `undefined` 的值进行检查使得值缩小到 `if` 范围内的未定义值。`else` 只能在联合类型中是除了未定义之外的所有值。在特定情况下，它只能是一个数字。在 `if` 和 `else` 之后，TypeScript 无法知道类型是什么；因此，值又回到了两种潜在类型：

```js
function myFunction(value: number | undefined): void {
 console.log("Value is number or undefined");
 if (value === undefined) {
 console.log("Value is undefined");
 } else {
 console.log("Value is NOT undefined, hence a number");
 }
 console.log("Value is number or undefined");
}
```

TypeScript 理解代码流。它很聪明，可以从特定的类型检查中冻结类型。在以下代码示例中，一个等于 `undefined` 的值迫使函数返回。这意味着通过那个点，就不可能有一个未定义的值。从潜在值的集合中减去 `undefined` 减少了只有数字的可能性：

```js
function myFunction2(value: number | undefined): void {
 console.log("Value is number or undefined");
 if (value === undefined) {
 return;
 }
 console.log("Value is NOT undefined, hence a number");
}

```

TypeScript 会根据你的条件检查缩小联合类型，不仅仅是原始类型。你还可以使用这个行为与一个区分器和用户定义的类型守卫一起使用，这两种模式我们将在本章中看到。

# `instanceof` 的限制

与 `typeof` 类似，JavaScript 中有 `instanceof` 操作符。`instanceof` 的限制是它只能用于具有原型链的类型：一个类。像 `typeof` 一样，`instanceof` 在设计和运行时工作，并且是 JavaScript 的原生功能：

```js
class MyClass1 {
 member1: string = "default";
 member2: number = 123;
}
class MyClass2 {
 member1: string = "default";
 member2: number = 123;
}
const a = new MyClass1();
const b = new MyClass2();
if (a instanceof MyClass1) {
 console.log("a === MyClass1");
}
if (b instanceof MyClass2) {
 console.log("b === MyClass2");
}
```

与 `typeof` 不同，`instanceof` 的结果不是一个字符串，不能用于 `console.log` 函数；它只能用于设置值在类型或变量中。它只能用于比较目的。下一个示例无法编译：

```js
type MyType = instanceOf MyClass1;
```

`instanceOf` 限制不仅限于关注类。`instanceOf` 操作符也不区分在继承情况下确切使用了哪个类。在下一个代码示例中，变量 ` c` 的类型是 `MyClass3`，它继承自 `MyClass2`。  `InstanceOf` 识别变量为两种类型。在以下代码中，两个 `if` 都被进入：

```js
class MyClass3 extends MyClass2 {
 member3: boolean = true;
}
const c = new MyClass3();
if (c instanceof MyClass2) {
 console.log("c === MyClass2");
}
if (c instanceof MyClass3) {
 console.log("c === MyClass3");
}
```

# 使用判别器进行类型识别

TypeScript 是一种结构化语言，这意味着它不像命名语言那样依赖于类型的名称。JavaScript 没有类型；因此，它是一种结构化语言。C# 或 Java 都是命名语言。这种差异很重要，因为它意味着 TypeScript 不会检查接口或类型的名称来做出任何决定。当我们思考 TypeScript 如何编译代码时，这一点是有意义的。在编译过程中，所有类型都会从代码中剥离，以生成干净的 JavaScript。这种共生关系是针对 JavaScript 的；因此，赋予 TypeScript 作为 JavaScript 超集的荣誉。然而，在 TypeScript 的运行时和 JavaScript 的设计时，我们需要知道我们正在操作哪种类型。在结构化代码中，方法是通过分析、比较和通过观察结构来推断类型。如果存在特定的成员，它将给出我们正在处理的内容的提示。以下代码示例显示了两个具有相同主体的相同接口，一个与相同结构相同的类型，以及一个具有匿名类型的第一个变量。该对象可以是每种类型，因为它尊重每种类型的契约：

```js
interface Type1 {
 m1: string;
}
interface Type2 {
 m1: string;
}
type Type3 = { m1: string };
const v0 = { m1: "AllTheSame" };
const v1: Type1 = v0;
const v2: Type2 = v0;
const v3: Type3 = v0;
```

在前面的示例中，使每种类型不同的方法是使用判别器的概念。**判别器**是在需要区分的一组共同类型之间具有共享名称的成员。这个组通常是一个联合。想法是每个类型都有一个具有相同名称的唯一 `string literal`。将 `string literal` 作为类型成员需要实现实现相同的 `string`。这意味着特定类型的每个实例都将具有相同的 `string`。TypeScript 可以通过查看 `string literal` 来推断类型。以下代码示例应用了这个原则。共同的成员被命名为 `kind`*，*每个接口和类型都有一个独特的 `kind`。匿名类型试图模仿 `Type1`，但失败了，因为推断的类型是 `string` 而不是 `string literal`：

```js
interface Type1 {
 kind: "Type1";
 m1: string;
}

interface Type2 {
 kind: "Type2";
 m1: string;
}

type Type3 = { kind: "Type3"; m1: string };
const v0 = { kind: "Type1", m1: "AllTheSame" };
const v1: Type1 = v0; // Does not compile
const v2: Type2 = v0; // Does not compile
const v3: Type3 = v0; // Does not compile
```

判别器证明不仅对于避免交叉类型有用，而且对于缩小类型范围也很有用。在许多类型的联合中，当与判别器进行比较时，TypeScript 将确切知道类型以及比较的范围。以下代码允许减少到确切类型。在特定情况下，m1 成员是所有三种类型中都有的成员，因此不需要缩小到单个类型来使用：

```js
function threeLogic(param: Type1 | Type2 | Type3): void {
 switch (param.kind) {
  case "Type1":
   console.log(param.m1); // param is type Type1
  break;
  case "Type2":
   console.log(param.m1); // param is type Type2
  break;
  case "Type3":
   console.log(param.m1); // param is type Type3
  break;
 }
}
```

如果我们有一个具有完全不同成员的接口，区分它们对于访问一个仅属于一个接口或另一个接口的成员是至关重要的。以下代码缩小了接口的范围，使其能够根据比较使用适当类型的成员：

```js
interface Alpha { kind: "Alpha", alpha: string }
interface Beta { kind: "Beta", beta: string }

function AlphaBeta(param: Alpha | Beta): void {
 switch (param.kind) {
  case "Alpha":
   console.log(param.alpha);
  break;
  case "Beta":
   console.log(param.beta);
  break;
 }
}
```

将字符串字面量用作判别器的用法通常被称为**字面量类型守卫**或**标记** **联合体**。它在函数式编程中非常强大，提供了一种快速识别类型的方法，而无需像其他技术（如用户定义守卫）那样根据需要开发特定的条件。

# 用户定义守卫模式

确定接口或类型的类型可能具有挑战性。在本章中，我们看到了判别器的使用。然而，对于通常称为`字符串字面量`的方法，存在一个缺点，即与继承和交集相关。以下代码无法编译：

```js
interface Type1 extends Type2 {
  kind: "Type1"; // Does not compile, expect “Type2”
  m1: number;
}

interface Type2 {
  kind: "Type2";
  m2: string;
}
```

对于交集来说，情况也是如此：

```js
interface Type2 {
 kind: "Type2";
 m2: string;
}

interface Type3 {
 kind: "Type3";
 m3: string;
}

type Type4 = Type2 & Type3;
const type4: Type4 = { kind: ???, m2: "1", m3: "2" }; // Does not compile
```

最后一个代码示例为成员类型创建了一个类型，这个类型要求同时是`字符串字面量`，这在实际操作中既不可能实现，也不实用。有了这些信息在手，我们可以看到，当避免继承和交集时，判别器模式工作得很好。这个想法是使用每个类型的自定义用户定义守卫。虽然创建起来可能有些繁琐，但可以确保你在设计和运行时都能得到该类型。这个想法是检查字段，看它们是否被定义。这种技术对于没有可选字段的类型工作得很好，因为你需要检查字段是否存在。作为函数和类型的作者，你不需要检查每个字段。你应该知道哪个字段足以识别类型。在下面的代码中，存在两种类型，其中一种类型扩展了另一种类型。创建了两个类型用户定义守卫——一个用于每个接口：

```js

interface Type1 extends Type2 {
 m1: number;
}

interface Type2 {
 m2: string;
 m3: number;
}

function checkInterfaceICheck1(obj: any): obj is Type1 {
 const type1WithMaybeManyUndefinedMembers = (obj as Type1);
 return type1WithMaybeManyUndefinedMembers.m1 !== undefined
 && type1WithMaybeManyUndefinedMembers.m2 !== undefined
 && type1WithMaybeManyUndefinedMembers.m3 !== undefined
}

function checkInterfaceICheck2(obj: any): obj is Type2 {
 const type1WithMaybeManyUndefinedMembers = (obj as Type2);
 return type1WithMaybeManyUndefinedMembers.m2 !== undefined
 && type1WithMaybeManyUndefinedMembers.m3 !== undefined;
}

function codeWithUnionParameter(obj: Type1 | Type2): void {
 if (checkInterfaceICheck1(obj)) {
 console.log("Type1", obj.m1);
 }

 if (checkInterfaceICheck2(obj)) {
 console.log("Type2", obj.m2);
 }
}
```

函数必须知道传递了两种类型中的哪一种，并且通过使用用户定义守卫来检查。定义守卫的返回类型是唯一的。它使用参数名称后跟*is*和如果我们期望的值是`true`，则期望的类型。它允许通过比较结构自动缩小到期望的类型。如果一切都在场并且已定义，它将返回`true`，但函数不会返回实际的`boolean`值。它返回将对象转换为该类型的对象。

# 类型转换的原因

**类型转换**是将一个类型转换到另一个类型上的行为。这是危险的，应该很少使用。类型转换可能产生副作用的原因是你正在手动控制将变量强制转换为另一种类型。类型可能创建不兼容和意外的结果。类型转换对于任何类型的变量都是可能的，从原始类型到更复杂的对象。

最基本的类型转换场景是从 `any` 获取一个值并将其类型化。下面的代码显示了一个在 `any` 中设置的数字，然后将其转换为数字类型的变量。你可以注意到两种不同的转换方式。一种使用较小的和较大的符号 `<>`，另一种使用 `as`。后者是推荐的方式，因为它不会混淆使用 TSX 语法（它使用组件的符号）的代码：

```js
let something: any = 1;
let variable1: number;
variable1 = <number>something;
variable1 = something as number;
```

之前的代码之所以有效，是因为转换是从 `any` 到 `number`。将数字转换为字符串是不行的。原因是转换仅在您与子类型一起工作时才有效。此外，`any` 是所有类型的子类型，这允许将类型转换为任何类型。然而，下面的代码无法编译，因为 `variable1` 是一个被转换为字符串的数字：

```js
let variable1: number = 1;
let variable2: string = variable1 as string;
```

TypeScript 也存在以避免在缺少字段的对象之间进行类型转换。在下面的代码中，这两种类型不能相互转换。TypeScript 在 `Type1` 中找不到 `m2`，而在下面的代码中第二次转换找不到 `m1`：

```js
interface Type1 {
 m1: number;
}

interface Type2 {
 m2: string;
 m3: number;
}

let t1: Type1 = { m1: 123 };
let t2: Type2 = t1 as Type2; // Property 'm2' is missing in type 'Type1'
let t3: Type2 = { m2: "2", m3: 3 };
let t4: Type1 = t2 as Type1;// Property 'm1' is missing in type 'Type2'
```

然而，将 `m1` 添加到 `Type2` 中会改变整个情况，并允许在两侧进行类型转换而不产生任何编译错误。原因是 `Type1` 通过其结构是 `Type2` 的子类型，这是 TypeScript 中最重要的：

```js
interface Type1 {
 m1: number;
}

interface Type2 {
 m1: number;
 m2: string;
 m3: number;
}

let t1: Type1 = { m1: 123 };
let t2: Type2 = t1 as Type2;
let t3: Type2 = { m1: 1, m2: "2", m3: 3 };
let t4: Type1 = t2 as Type1;
```

最后一段代码有趣的地方在于最后的转换是无用的。原因是 Type2 拥有 `Type1` 的所有结构，而 `Type1` 是 `Type2` 的子类型。这意味着它们在结构上的最小点是结构上等效的：

```js
let t3: Type2 = { m1: 1, m2: "2", m3: 3 };
let t4: Type1 = t2;
```

类型转换对于 `t1` 到 `Type2` 是必需的，因为 `t1` 不满足合同（它缺少 `m2` 和 `m3`）。转换产生了一个 `false` 的 `Type2`，因为 `m2` 和 `m3` 不存在，这意味着它们是未定义的。`Type2` 对于这些成员没有未定义的类型，这使得它在未来的使用中变得有问题，因为 TypeScript 将允许 `m2` 使用任何字符串的函数，而此处的 `m2` 是未定义的。类型转换伴随着巨大的责任，而篡改类型将使 TypeScript 无法执行安全验证。

当类型转换影响一个 `any` 类型的对象时，滑坡效应会更陡峭。很难避免所有的 `any`。例如，当数据在系统之间传递时。一个 Ajax 请求返回一个 JSON 对象，这是不可避免的，因为它是一个 `any`。响应未进行类型化，要将值引入 TypeScript，必须执行一个关键的类型转换。

将类型转换为 `any` 然后转换为所需类型是一个坏的模式。这是绕过 TypeScript 的方法，发现转换不是有效的。任何东西都可以转换为 `any`，并且可以从任何类型转换到其他任何类型：

```js
let a: number = 1;
let b: string = "2";
a = b as number; // Doesn't compile
a = b as any as number; //Shortcircuit with any
```

# 类型断言是什么？

有一些场景，你知道一个类型不是未定义或 null，但 TypeScript 会暗示它可能是。当这种情况发生时，你可以对 `undefined` 或 `null` 进行检查，并在条件的作用域内将保证类型不是可空的。然而，有三种场景可以从更短的语法中受益。 

第一个场景是深层嵌套的对象。在这种情况下，您可能有多个级别的可空字段，如果您确信它们不是 undefined 或 null，这将有助于避免嵌套的`if`结构：

```js
interface T1 {
 myNumber: number | undefined;
}

interface T2 {
 t1: T1 | undefined;
}

interface T3 {
 t2: T2 | undefined;
}

const myObject: T3 | undefined = {
 t2: {
 t1: {
  myNumber: 1
 }
 }
}

if (myObject !== undefined) {
 if (myObject.t2 !== undefined) {
  if (myObject.t2.t1 !== undefined) {
   if (myObject.t2.t1.myNumber !== undefined) {
    console.log("My number is :", myObject.t2.t1.myNumber);
   }
  }
 }
}
```

条件检查是确保没有未定义内容的最安全方式。然而，在某些情况下，检查可能是在访问数据之前进行的，但需要在检查范围之外访问值，这使得 TypeScript 对`if`语句感到不安，同时状态已发生变化。如果我们试图在之前的代码之后立即访问`myNumber`，就会发生这种情况。这就是`断言`类型发挥作用的地方。`断言`类型是成员后面的感叹号（或称为叹号），它指定该成员不是 null 或 undefined。您正在断言这是事实，并承担起将字段*未定义*或*可空*的责任。

这意味着您可以使用单行来访问成员：

```js
console.log("My number is :", myObject!.t2!.t1!.myNumber);
```

理解这一点至关重要，因为如果在不恰当的时间使用，可能会导致潜在的运行时错误。由于某种原因，任何可空字段如果在错误的时间或位置应用，都可能变为可空。无法保证执行将成功，但会缓解 TypeScript 错误，指出您在未将其缩小到特定类型的情况下访问了可空字段。

使用类型`断言`的第二种情况是在类中定义字段时。如果 TypeScript 设置为具有编译严格性以避免未初始化的字段，则在定义字段且未在声明或构造函数中指定值时，您将遇到错误。这是一个很好的验证，但在某些罕见情况下，值可能在`initialize`函数中稍后出现。在这种情况下，您可以断言类的字段，表明您正在处理该值：

```js
class LateInitialization {
 m1!: number; // Not initialized (use type assertion)
 constructor() {
   // No initializing here
 }
 public init(): void {
   this.m1 = 1;
 }
}
```

再次强调，这应该谨慎使用，因为它可能会带来一些问题。例如，现在您可以访问成员并使用它，而无需 TypeScript 验证在访问值之前值已被分配：

```js
constructor() {
   this.m1 + 1; // This will fail
}
```

这可能看起来微不足道，因为您知道您不会这样做。但有时可能不那么明显。一个错误的情况是，您从另一个可能在`init`函数之前被调用的公共函数中访问成员，这可能导致对变量的任何使用都未定义。类型断言迫使 TypeScript 对未初始化的值视而不见。

最后一种情况也是危险的，应该非常小心地编码。你可以在任何时候使用感叹号来消除可空性。这意味着它也适用于简单变量。以下代码声明了一个类型为字符串或 `undefined` 的变量。它使用一个立即调用的函数来设置其值。该函数的返回类型也是 `string | undefined`。TypeScript 推断这个函数可能返回一个或两个类型，因此可以合法地返回 `undefined`。然而，我们知道情况并非如此，因此可以使用感叹号来消除 `undefined` 的可能性，并使用字符串的函数：

```js
let var1: string | undefined;
var1 = ((): string | undefined => "Not undefined but type is still both")();
console.log(var1!.substr(0, 5));
```

再次强调，这是危险的，可以采用更好的方法来避免。首先，要避免使用包含 `undefined` 或 `null` 的联合。如果这超出了你的控制范围，应避免使用像最后一个代码示例中那样也返回 `undefined` 的函数。如果将返回类型改为字符串，问题就可以优雅地解决。

# 类的比较

类与接口、类型或原始类型不同。它们有一个原型链并遵循不同的规则。例如，如果两个不同的类具有相同的结构，它们可以互换。以下类 `C1` 和 `C2` 在结构上是相同的，可以在需要 `C1` 的函数中互换使用。你甚至可以在 `C1` 变量中实例化 `C2`：

```js
class C1 {
 public a: number = 1;
 public funct(): void { }
}

class C2 {
 public a: number = 1;
 public funct(): void { }
}

const c1: C1 = new C1();
const c2: C2 = new C2();
const c12: C1 = new C2();

function executeClass1(c1: C1): void {
 c1.funct();
}

executeClass1(c1);
executeClass1(c2);
executeClass1(c12);
```

如果我们在 `C1` 或 `C2` 中添加 `private` 字段，那么它们就不会相同：

```js
class C1 {
 public a: number = 1;
 public funct(): void { }
 private p: string = "p";
}

class C2 {
 public a: number = 1;
 public funct(): void { }
 private p: string = "p";
}

const c1: C1 = new C1();
const c2: C2 = new C2();
const c12: C1 = new C2(); // Does not compile

function executeClass1(c1: C1): void {
 c1.funct();
}

executeClass1(c1);
executeClass1(c2); // Does not compile
executeClass1(c12);
```

`private` 和 `protected` 字段使每个类独一无二。TypeScript 继续比较结构，但在这些两个可见性修饰符方面会做出例外。

原因是当使用继承并将子类赋值给基类时，它必须来自相同的层次结构，而不是来自不同层次结构但形状相似的东西。以下代码展示了如果没有 `private` 或 `protected` 字段，基类可以被具有子类和基类结构的单个类所替代：

```js
class B1 {
  public baseFunct(): void { }
}

class C1 extends B1 {
  public a: number = 1;
  public funct(): void { }
}

class C2 {
  public a: number = 1;
  public funct(): void { }
  public baseFunct(): void { }
}

const c1: B1 = new C1();
const c2: B1 = new C2();
```

然而，在基类 `B1` 中添加一个 `private` 字段，并在 `C2` 中添加相同的字段，这使得它们变得不同，阻止了 `C2` 被赋值给类型为 `B1` 的变量 `C2`：

```js
class B1 {
 private name: string = "b1";
 public baseFunct(): void { }
}

class C1 extends B1 {
 public a: number = 1;
 public funct(): void { }
}

class C2 {
 private name: string = "c2";
 public a: number = 1;
 public funct(): void { }
 public baseFunct(): void { }
}

const c1: B1 = new C1();
const c2: B1 = new C2(); // Does not compile
```

# 在函数签名中使用联合类型的类型收缩

复杂的函数可能难以处理。这种情况通常发生在函数有一个或多个不同类型的参数，并且可以返回一个或多个类型时。TypeScript 允许将任何类型拼接在一起：

```js
function f(p: number | string): boolean | Date {
 if (typeof p === "number") {
  return true;
 }
 return new Date();
}

const r1: boolean = f(1); // Does not compile
const r2: Date = f("string"); // Does not compile
```

在前面的代码中，代码无法编译。原因是函数返回一个必须缩小的联合类型。然而，如果我们在上面的函数中添加重载，我们可以将联合类型与单一返回类型的特定参数集匹配。前面的代码无法编译，因为它将联合类型返回到一个单一类型变量中。通过指定当参数是数字时函数返回`boolean`，而当它是字符串时返回日期，就不需要任何类型转换或其它操作：

```js
function f(p: number): boolean;
function f(p: string): Date;
function f(p: number | string): boolean | Date {
 if (typeof p === "number") {
  return true;
 }
 return new Date();
}

const r1: boolean = f(1);
const r2: Date = f("string");
```

这不仅仅是将单个参数关联到返回类型。例如，在下面的代码中，我们确保只能一起发送所有数字参数或所有字符串参数：

```js
function g(p: number, q: number): boolean;
function g(p: string, q: string): Date;
function g(p: number | string, q: number | string): void {
}
g(1, "123"); // Doesn't compile
g(1, 2);
g("1", "2");
```

# 摘要

在本章中，我们介绍了如何更好地理解变量的类型。这不仅有助于做出决策，而且可以将变量缩小到单一类型，从而有机会访问特定类型特有的特定成员。

在下一章中，我们将看到如何通过使用泛型变量来泛化类型。泛型变量增加了你代码中对象和变量的可重用性，从而减少了创建平凡类型的必要性。
