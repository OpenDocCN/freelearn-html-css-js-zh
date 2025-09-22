# 第三章：利用对象释放类型的力量

TypeScript 中有众多不同的对象，可能会让人感到不知所措。在本章中，我们将阐述 `object`、`Object`、`object literal` 以及通过构造函数构建的对象之间的区别。本章还讨论了类型联合的概念，这将允许单个值有无限种类型的组合。此外，交集的概念也浮现出来，它为不同地操作类型提供了可能性。在本章结束时，读者将能够创建包含复杂结构的对象组合。我们将剖析如何创建具有强类型索引签名的字典，理解类型如何通过映射带来益处，以及如何使用正确的对象在定义具有广泛适用性的对象时尽可能准确。

本章涵盖了以下主题：

+   如何使用索引签名将集合/字典强类型化

+   TypeScript 与映射

+   索引签名与映射的区别

+   `object` 与 `Object` 的区别

+   何时使用 `object`、`Object` 或任何

+   什么是 `object literal`

+   如何创建一个构造对象

+   显式类型或转换的区别

+   多类型变量

+   将类型与交集结合

+   与非类型相交

+   与可选类型相交

+   将类型与继承合并

+   类型与接口的区别

+   解构类型和数组

+   元组

+   `declare` 与 `let`/`const`/`var` 的区别

# 如何使用索引签名将集合/字典强类型化

除了数组之外，*集合* 或 *字典* 是一种常见的结构，用于存储无结构数据以便快速访问。在 JavaScript 中，使用能够将成员分配给对象的动态概念创建字典。每个对象的属性成为字典的键。TypeScript 在此模式之上引入了 *索引签名*。它允许你指定键的类型（在数字和字符串之间）以及任何类型的值：

```js
interface Person {
 [id: string]: string;
}
```

向字典写入就像使用方括号并分配必须遵守定义右侧的值一样简单。在下面的代码示例中，键和值都是字符串：

```js
const p: Person = {};

p["id-1"] = "Name1";
p["string-2"] = "Name12";

console.log(p["string-2"]); // Name12 
```

索引签名可能因为历史原因而变得复杂。例如，如果索引被定义为接受字符串作为键，你将能够传递字符串和数字。反之则不成立：数字类型的键不接受字符串：

```js
const c: Person = {
  "test": "compile", 
  123: "compile too" // Key is number even if Person requires string: it compiles
};

interface NotAPerson {
  [id: number]: string;
}

// DOES NOT COMPILE:
const c2: NotAPerson = {
  "test": "compile", // THIS LINE DOES NOT COMPILE
  123: "compile too"
};
```

最后一个示例展示了关于键类型的多个问题。代码使用一种语法，通过使用所有成员都是键且其值是索引签名值的 `object literal` 直接定义索引签名的值。这是初始化默认值的语法，而另一种方式，即使用方括号，是动态添加和快速访问值的方式。

此外，TypeScript 允许你通过提供成员名称作为字符串来使用方括号访问对象的成员。与索引签名的区别在于，TypeScript 不允许你在定义中没有提供索引签名的情况下读取或添加成员：

```js
interface NotIndexSignatureObject {
    name: string;
    count: number;
}

const obj: NotIndexSignatureObject = {
    name: "My Name",
    count: 123
};

console.log(obj["doesNotExist"]); // Does not compile
console.log(obj["name"]); // Compile
```

索引签名的一个奇怪之处在于，当它与具有其他成员的对象结合时。索引签名的键只能是一个字符串，其成员返回一个字符串。这意味着在大多数情况下，你将不得不退回到使用数字键。以下代码无法编译：

```js
 interface ObjWithMembersAndIndexSignature {
    name: string;
    count: number;
    when: Date;
    [id: string]: string; // DOES NOT COMPILE
}
```

相比之下，以下代码可以编译，但很脆弱。它之所以可以编译，是因为在某些非常罕见的情况下，TypeScript 会根据其用法自动将类型转换回字符串。在这种情况下，成员 `count` 和 `when` 的 `number` 和 `Date` 被接受为字符串。然而，添加一个具有对象的成员的微小变化将破坏这一规则。以下两个代码块说明了这种变化。这个后续块包含了一个原始值：

```js
interface ObjWithMembersAndIndexSignature {
 name: string;
 count: number;
 when: Date;
 [id: number]: string; // COMPILE
}
```

这个后续块包含了一个在定义索引签名时不允许的额外对象：

```js
interface ObjWithMembersAndIndexSignature2 {
 name: string;
 count: number;
 when: Date;
 obj: { s: string }; // DOES NOT COMPILE
 [id: number]: string | number | Date;
}
```

你可能遇到的另一个编译问题是向具有数字键的索引签名的对象添加一个字符串键：

```js
const obj2: ObjWithMembersAndIndexSignature = {
    name: "My Name",
    count: 123,
    when: new Date(),
    "more": "nooo" // DOES NOT COMPILE
};
```

你可以通过提供一个具有字符串值的数字类型的成员来转换对象定义：

```js
  const obj3: ObjWithMembersAndIndexSignature = {
    name: "My Name",
    count: 123,
    when: new Date(),
    12: "string only" // Good if number->string
};
```

然而，如果你想有一个字符串作为键，你将需要将你的索引签名中允许的值类型更改为每个成员的联合：

```js
  interface ObjWithMembersAndIndexSignature2 {
    name: string;
    count: number;
    when: Date;
    [id: string]: string | number | Date;
}
```

要总结索引签名，明智的做法是使你的映射对象小且成员不多，以便能够访问一个无需缩小类型的索引签名。例如，最后一个代码示例返回一个字符串、一个数字或一个日期。这意味着每次访问对象时，都需要在消费其属性之前检查其类型。然而，具有仅索引签名的接口可以用作对象的属性，并具有所有快速访问，而无需缩小。以下代码演示了这种模式：

```js
interface MyMap<T> {
    [index: string]: T;
}

interface YourBusinessLogicObject {
    oneProps: string;
    secondProps: number;
    thirdProps: Date;
    yourDictionary: MyMap<string>;
}
```

# TypeScript 和映射

我们讨论了使用索引签名创建字典/集合，该索引签名利用了对象的灵活性。另一种选择是使用 `map` 类。`map` 是一个可以带或不带值实例化的类，它是一种在 TypeScript 中不独特的对象类型。ECMAScript 定义了映射的结构和行为；TypeScript 在类之上提供了类型。

一个映射具有与索引签名相同的快速访问能力。以下代码使用两个键值条目实例化了 `Map`。键和值可以是任何东西。在构造函数中重要的是，当提供值时，这个值必须是可迭代的：

```js
let map = new Map([["key1", "value1"], ["key2", "value2"]]);

let value1: string | undefined = map.get("key1");
```

之前的代码不仅创建了一个映射，还通过使用与构造函数中定义的相同类型的键来访问值。如果映射中不存在该键，则返回一个未定义的值，类似于索引签名。接下来的代码示例创建了两个映射，没有提供初始值。第一个没有使用泛型定义；因此，回退到键和值的类型为`any`。然而，第二行显示了一个初始化，指定泛型类型具有字符串键和数值值。即使映射在初始化时没有指定值，后者仍然为未来由函数`set`设置的值提供了强类型约束：

```js
let map2 = new Map(); // Key any, value any
let map3 = new Map<string, number>(); // Key string, value number
```

以下代码无法编译，因为键类型必须相同。在代码示例中，它有一个数字和一个字符串：

```js
let map4 = new Map([[1, "value1"], ["key2", "value2"]]); // Doesn't compile
```

映射除了`get`函数之外还有许多其他功能。它可以*设置*值，这在没有所有映射创建的值时非常有用。映射还可以通过返回`true`或`false`来查找映射中是否存在键。最后，可以通过函数而不是依赖于索引签名的`delete`关键字来删除条目：

```js
map.set("key3", "value3");
map.has("key1");
map.delete("key1"); // Similar to delete obj.key1 (index signature)
```

# 索引签名和映射之间的区别

使用映射或对象索引签名模式与对象之间的区别很小。以下是每个结构的优点列表：

**对象**：

+   可以有不仅仅是键值对。它还可以有函数和其他成员。

+   来自 JSON 的对象自动与索引签名兼容，而映射则需要手动映射。

+   对象模式在访问数据方面比映射更快，对于小数据集和中等数据集，它使用的内存更少。这在 Chrome 中是正确的，但不同浏览器之间的基准测试并不一致，以及映射/对象的整体大小。

**映射**：

+   当进行大量添加和删除操作时，映射表现更好。它使用底层的哈希函数。

+   当添加元素时，它保留了顺序。这可能是一个优势，因为映射自然可迭代。

+   映射在大数据集上表现更好。

+   映射的键不仅限于数字或字符串键。

# 对象和 Object 之间的区别

TypeScript 中有许多对象类型。有`Object`、`object`、`class object`和`object literal`。在本节中，我们将介绍大写`Object`（大写字母）和小写`object`（小写字母）之间的区别。

以大写字母开头的`Object`类型，或大写字母，或大写字母*O*代表一种普遍存在的东西，是一个与每个类型和对象都可用交叉类型。大写字母`Object`携带一组常用函数。以下是其可用函数列表：

```js
toString(): string;
toLocaleString(): string;
valueOf(): Object;
hasOwnProperty(v: string): boolean;
isPrototypeOf(v: Object): boolean;
propertyIsEnumerable(v: string): boolean;
```

许多类型都包含在`Object`类型的范畴之下。将几个不同的值分配给`Object`类型对象显示了类型的灵活性以及潜在类型范围的广泛性：

```js
let bigObject: Object;
bigObject = 1;
bigObject = "1";
bigObject = true;
bigObject = Symbol("123");
bigObject = { a: "test" };
bigObject = () => { };
bigObject = [1, 2, 3];
bigObject = new Date();
bigObject = new MyClass();
bigObject = Object.create({});
```

小写的`object`将所有不是数字、字符串、`boolean`、`null`、`undefined`或`Symbol`的值转换为对象。小写的`object`是大写`Object`的子集。它包含`object literals`、日期、函数、数组和用`new`和`create`创建的对象实例：

```js
let littleObject: object;

littleObject = { a: "test" };
littleObject = new Date();
littleObject = () => { };
littleObject = [1, 2, 3];
littleObject = new MyClass();
littleObject = Object.create({});
```

在`null`和`undefined`的情况下，它们既不是`object`也不是`Object`。它们属于一个特殊类别，并且是所有其他类型的子类型。TypeScript 的编译器必须配置为具有严格的选项`"strictNullCheck"`，这是默认配置值，这意味着即使`null`和`undefined`是所有类型的子集，只有主类型和`null`或`undefined`的联合才能允许将赋值给这两个特殊值之一：

```js
let acceptNull: number | null = null;
acceptNull = 1;

let acceptUndefined: number | undefined = 1;
acceptUndefined = null;
```

# 何时使用`object`、`Object`或`any`

使用哪种类型的对象是之前讨论`object`、`Object`和`any`之间差异的问题的一个子问题。一般规则是始终使用更约束的类型。这意味着尽可能避免使用`object`和`Object`。然而，在需要覆盖更广泛的类型且无法使用联合定义的情况下，如果你不需要原始类型，使用`object`更好，因为它具有更少的潜在值。

`object`和`Object`都比`any`好，因为`any`允许访问任何类型的任何成员，而`object`将限制你只能访问以下内容：

```js
let obj1: Object = {};
let obj2: object = {};
let obj3: {} = {};
let obj4: any = {};

obj1.test = "2"; // Does not compile
obj2.test = "2"; // Does not compile
obj3.test = "2"; // Does not compile
obj4.test = "2";

obj1.toString();
obj2.toString();
obj3.toString();
obj4.toString();
```

如果你不知道类型并且需要取一个对象，如果你不允许原始类型，你应该使用小写的`object`。如果你支持原始类型，你应该回退到大写的`Object`，最后不得已使用`any`。然而，一个更好的潜在方法是，如果可能的话，使用一个泛型类型，这样可以避免进行类型检查和转换，这通常是使用诸如`object`和`Object`之类的类型时的一个陷阱。

# 什么是`object literal`？

一个`object literal`是用花括号创建的对象。一个`object literal`既是`object`也是`Object`。你可以使用`type`或`interface`为`object literal`定义类型。这是一种快速地将数据放入具有类型结构的结构中的方法。`object literal`继承自大写的`Object`。

```js
type ObjLiteralType = { x: number, y: number };
interface ObjLiteralType2 {
  x: number;
  y: number;
}
```

我们可以用四个函数进行快速测试，看看`object literal`是否被所有函数接受，即使参数的类型在所有函数的签名中不同：

```js
function uppercaseObject(obj: Object): void { }
function lowercaseObject(obj: object): void { }
function anyObject(obj: any): void { }
function objectLiteral(obj: {}): void { }

uppercaseObject({ x: 1 });
lowercaseObject({ x: 1 });
anyObject({ x: 1 });
objectLiteral({ x: 1 });
```

两个（或更多）具有相同结构的对象可以互换。你可以定义一个`object literal`并将其设置在变量中，该变量定义了接口中相同的结构。如果你匿名或推断类型，也可以这样做。这里有四种创建类型化的`object literal`的方法。它们都可以相互赋值，因为它们具有相同的结构。这是 TypeScript 的一个优点，因为它是一种结构化语言，而不是名义语言：

```js
let literalObject: ObjLiteralType = { x: 1, y: 2 };
let literalObject2: ObjLiteralType2 = { x: 1, y: 2 };
let literalObject3: { x: number, y: number } = { x: 1, y: 2 };
let literalObject4 = { x: 1, y: 2 };

literalObject = literalObject2 = literalObject3 = literalObject4;
```

# 如何创建构造对象

最后，类对象是通过关键字`class`定义并通过关键字`new`实例化的对象。每个类都可以被实例化多次。每次实例化都始于`new`的使用，并且对象中设置的每个值都保留在该对象中，除了静态字段，这些字段在相同类的每个实例之间共享。我们将在后面的章节中看到面向对象和对象的一些特性：

```js
class ClassWithConstructor {
    constructor(){
        console.log("Object instantiated");
    }
}
const cwc = new ClassWithConstructor();
```

创建类的调用会调用构造函数。在之前的代码示例中，`console.log`将在类实例化为对象时被调用。

# 显式类型和类型转换之间的区别

如果你想要构建一个接口，你可以使用冒号设置变量类型并指定字段。如果一个字段缺失，TypeScript 将不会编译；如果定义的内容过多，TypeScript 也不会编译：

```js
interface MyObject {
    a: number;
    b: string;
}
const newObject1: MyObject = { a: 1, b: "2" };
```

另一种方式是不在冒号后指定类型，而是使用`as`进行类型转换：

```js
  const newObject2 = { a: 1, b: "2" } as MyObject;
```

问题在于类型转换会强制 TypeScript 相信对象是指定的类型，即使它不遵守契约。类型转换永远不应该用来定义变量。原因是即使最初遵守契约，如果对象发生变化，转换仍然会强制类型分配，但对象的结构不会正确。以下两行代码可以编译，但在逻辑上是不合法的。第一行有一个不在接口中的额外成员，而第二行缺失了一个字段：

```js
const newObject3 = { a: 1, b: "2", c: "OH NO" } as MyObject;
const newObject4 = { a: 1 } as MyObject;
```

# 多类型变量

在许多情况下，一个变量需要拥有多个字段。例如，你可以有一个只接受几个字符串值的字段：

```js
type MyType = "a" | "b" | "c";
const m1: MyType = "a";
const m2: MyType = "no"; // Doesn’t compile
```

使用关键字`type`创建类型不是必需的，但允许类型的重用。第一个例子是创建一个允许多个字符串的类型。使用新类型声明的变量将只接受列表中的字符串。然而，大多数情况下，联合使用更复杂类型，例如接口、类型和原始类型之间的联合：

```js
type AcceptedOption = number | string | { option1: string, option2: number };
const myOption1: AcceptedOption = "run";
const myOption2: AcceptedOption = 123;
const myOption3: AcceptedOption = { option1: "run", option2: 123 };
```

函数可以接受联合类型作为参数，也可以返回返回类型。联合通常用于接受类型以及`undefined`：

```js
function functWithUnion(p: boolean | undefined): undefined | null{
      return undefined;
}
```

当使用联合时，只有共同的字段是可访问的。在以下代码中，`TypeA`有两个字段，`a`和`b`，而`TypeB`有`b`和`c`。唯一的共同字段是`b`，这意味着它是函数中唯一可用和可访问的字段。这直到我们将类型缩小到联合中的一个类型才会成立。我们将在后面看到类型缩小是如何工作的：

```js
interface TypeA {
    a: string;
    b: string;
}

interface TypeB {
    b: string;
    c: string;
}

function functionWithUnion(param1: TypeA | TypeB): void {
    console.log(param1.b);
}
```

# 将类型与交集结合

你构建的类型越多，你就越需要一个功能来处理它们。交集功能是 TypeScript 工具箱中的一个工具，它允许你合并类型。交集符号是`&`。以下代码示例显示我们正在通过两个接口的组合创建第三个类型。共同的字段合并为一个，差异累加：

```js
interface TypeA {
    a: string;
    b: string;
}
interface TypeB {
    b: string;
    c: string;
}

type TypeC = TypeA & TypeB;
const m: TypeC = {
    a: "A",
    b: "B",
    c: "C",
};
```

也可以相交泛型和原始类型。后者较少使用，因为它几乎不实用。然而，前者（泛型）有助于将自定义类型合并到定义的契约中：

```js
interface TypeA {
    a: string;
    b: string;
}
function intersectGeneric<TT1>(t1: TT1): TT1 & TypeA {
    const x: TypeA = { a: "a", b: "b" };
    return (<any>Object).assign({}, x, t1);
}

const result = intersectGeneric({ c: "c" });
console.log(result); // { a: 'a', b: 'b', c: 'c' }
```

相交中类型的顺序并不重要。这里创建的两个类型完全相同：

```js
type TypeD1 = TypeA & TypeB;
type TypeD2 = TypeB & TypeA;
```

然而，即使它们相同，具有相同的值，每次初始化都会创建一个独特的对象，这意味着比较它们将会是错误的。关于比较具有不同名称的相同类型，它们都是`Object`类型，原因是`typeOf`是 JavaScript 运算符，类型在运行时被移除；因此，它在设计时表现相同。要比较类型，我们需要一个将在后面讨论的判别器：

```js
 let d1: TypeD1 = { a: "a", b: "b", c: "c" };
 let d2: TypeD2 = { a: "a", b: "b", c: "c" };

 console.log(typeof d1); // Object
 console.log(typeof d2); // Object
 console.log(d1 === d2); // False

 d2 = d1;
 console.log(d1 === d2); // True
```

括号的使用不会影响类型的声明。以下代码是多余的，因为联合是无用的。这里有四种不同的类型，它们具有相同的值：

```js
type TypeD3 = (TypeA & TypeB) | (TypeB & TypeA);
type TypeD4 = TypeA & TypeB | TypeB & TypeA;
type TypeD5 = (TypeA & TypeB);

type TypeD6 = TypeA & TypeB;

let d3: TypeD3 = { a: "a", b: "b", c: "c" };
let d4: TypeD4 = { a: "a", b: "b", c: "c" };
let d5: TypeD5 = { a: "a", b: "b", c: "c" };
let d6: TypeD6 = { a: "a", b: "b", c: "c" };
```

# 与非类型相交

可以与类型、原始类型和接口相交。与原始类型相交是没有用的，因为一个数字不能同时是布尔值。然而，与接口相交与与类型相交一样有效。对于类型或接口，关于相交的规则是相同的：

```js
interface InterfaceA {
 m1: string;
}

interface InterfaceB {
 m2: string;
}

type TypeAB = InterfaceA & InterfaceB;
```

虽然相交最常见的案例是类型和接口，但也可以相交类。与类的相交很少见，它创建了一个不能实例化的类型。只从每个类中提取公共字段来创建字段：

```js
type ClassAb = ClassA & ClassB;
const classAb: ClassAb = { m1: "test", m2: "test2" };
```

# 与可选类型相交

可以相交两个具有不同规则的交叉属性的类型。一个类型中的一个字段提到了可选成员，可以与一个非可选的类型合并。结果是该字段变为非可选：

```js
 interface InterfaceSameField1 {
    m1: string;
 }

 interface InterfaceSameField2 {
    m1?: string;
 }

 type Same = InterfaceSameField1 & InterfaceSameField2;
 let same: Same = { m1: "This is required" };
```

之前的代码示例显示了相交和字段`m1`是必需的。如果省略或设置为`undefined`，则代码无法编译。

# 使用继承合并类型

如果这些类型是接口或类，则可以使用`extends`合并两个类型。使用另一个接口扩展一个接口是使用连字符的替代方案。在下面的代码示例中，合并的接口包含其自身的成员，以及`InterfaceA`和`InterfaceB`的成员：

```js
interface InterfaceA {
  m1: string;
}

interface InterfaceB {
  m2: string;
}

interface InterfaceMergeAB extends InterfaceA, InterfaceB {
  m3: string;
}
```

# 类型与接口之间的差异

类型与接口并不完全相同。例如，你可以合并两个接口，但不能将接口与原始类型合并，这可以通过类型完成。你可以在许多允许未来扩展外部主模块的定义中定义一个接口。在许多领域增强接口的可能性允许许多插件或合同版本化模式发生。这个功能的术语是**开放的**：

```js
interface IA {
  m1: string;
}

interface IA {

  m2: string;
}

const ia: IA = { m1: "m1", m2: "m2" };
```

一个类可以扩展一个类型或一个接口。后者更常见，因为类型有一些注意事项。例如，包含原始值的类型不会是一个好的类选择，因为实现将不会工作。TypeScript 足够智能，可以分析类型的内 容并确定实现无法发生：

```js
type TPrimitive1 = string;
type TPrimitive2 = { m1: string };

class ExtendPrimitiv1 implements TPrimitive1 { // Does not compile
}

class ExtendPrimitiv2 implements TPrimitive2 { // Compile
 public m1: string = "Compile";
}
```

类型和接口可以有索引签名：

```js
type TypeWithIndex = {
  [key: string]: string;
  m1: string;
}

const c: TypeWithIndex = {
  m1: "m1"
};

c["m2"] = "m2";
```

常规做法是尽可能依赖接口，因为其开放性特征，减少了关于类型是否可以具有原始值的混淆，并且因为它们可以被扩展或交叉。`type`关键字用于创建原始值的联合，或动态交叉对象字面量。

# 解构类型和数组

解构是 JavaScript 的一个特性，TypeScript 通过类型保护支持它。解构将对象分解成不同的部分。TypeScript 将这些类型带到这些部分。

一种情况是你需要将对象成员提取到变量中。这可以不使用解构来完成，但需要几行代码：

```js
interface MyObject {
  id: number;
  time: number;
  name: string;
  startTime: Date;
  stopTime: Date;
  category: string;
}

const obj: MyObject = {
  id: 1,
  time: 100,
  name: "MyName",
  startTime: new Date(),
  stopTime: new Date(),
  category: "Cat5"
};

const id = obj.id;
const name = obj.name;
const category = obj.category;
```

使用解构，可以在一行内完成。所有变量都是对象的类型。这意味着`id`是类型为 number 的新变量，`name`是类型为 string，同样`category`：

```js
const { id, name, category } = obj;
```

解构可以使用剩余操作符来获取未指定的属性剩余部分。剩余的语法是变量名之前的三点，该变量将持有剩余成员：

```js
const { id, name, category, ...remainingMembers } = obj;

remainingMembers.startTime;
remainingMembers.stopTime;
remainingMembers.time;
```

如您所见，变量`remainingMember`有三个成员，这是在其余部分之前没有明确指出的三个成员。这意味着`remainingMember`的类型是一个具有`startTime`、`stopTime`和`time`成员的对象字面量，它们的类型分别是`Date`、`Date`和`number`。

解构和剩余参数也可以与数组一起使用。你可以指定一个变量名，它的类型将是数组的类型。剩余参数允许创建一个新的数组，该数组包含具有初始数组类型的值的剩余部分。在下面的代码示例中，`value1`包含数字`1`（不是字符串，而是作为数字），`value2`包含`2`，而`value3To9`是一个包含值`3`、`4`、`5`、`6`、`7`、`8`和`9`的数组：

```js
 const values = [1, 2, 3, 4, 5, 6, 7, 8, 9];
 const [value1, value2, ...value3To9] = values;
```

也可以通过使用逗号而不指定变量来跳过位置。在下面的代码示例中，`value_1`和`value_2`之间有一个空格，这意味着第二个位置的值`2`既不在任何单个变量（`value1`或`value2`）中，也不在变量`value4To9`中：

```js
const values = [1, 2, 3, 4, 5, 6, 7, 8, 9];
const [value1, value2, ...value3To9] = values;
const [value_1, , value_3, ...value4To9] = values;
```

# 元组

**元组**是对象的一种替代方案，可以在单个变量中存储多个值。通常用于通过函数传递信息，它利用数组来携带不同类型。使用元组进行赋值是通过在数组的特定索引处设置所需值来完成的，消费者必须知道以检索相关信息。在 JavaScript 中，使用数组是足够的。然而，在 TypeScript 中这样做会导致弱类型。以下代码显示 TypeScript 将推断类型为数字或字符串的数组，这在某些情况下是有意义的，但不符合代码想要强类型的要求。原因是数组可以在数组的任何位置是数字或类型，而在元组的情况下，我们希望数组中的每个位置都有特定的类型：

```js
let tuple1 = ["First name", "Last Name", 1980]; // Infered as (string | number)[]
tuple1[2] = "Not a number";
let tuple2: [string, string, number] = ["First name", "Last Name", 1980]; // Tuple
tuple2[2] = "Not a number"; // Does not compile
```

为了克服数组推断，我们需要为每个位置指定类型。然而，即使元组由于通过索引指定类型而比具有多个类型的数组更具体，但这仍然不如使用对象安全：

```js
let tuple3: [string, string, number]
tuple3 = ["Only One"]; // Does not compile
tuple3 = ["One", "Two", 3, "four"]; // Does not compile
tuple3[5] = "12"; // Compile
```

之前的代码说明了在实例化过程中，既有类型的验证也有大小的验证。这意味着在赋值时，你必须尊重预期的类型和值的数量。然而，代码的最后一行显示了数组表面的本质，并且无论元组声明指定了三个位置，都可以在定义的位置之后添加任何类型的值。在代码示例中，前三个位置（索引`0`、`1`和`2`）是强类型的，但位置四及以上可以是任何类型。尽管如此，使用方括号更改值时，将验证类型：

```js
tuple3[5] = "12"; // Compile, do not mind the type
tuple3[1]= 2; // Does not compile, require to be a string
```

元组支持扩展操作符，可以将函数参数解构为多个变量。以下代码示例显示单个元组参数可以被展开。函数`restFunction`与`resultFunction`等价。代码示例显示可以传递一个元组但不能传递一个数组：

```js
function restFunction(...args: [number, string, boolean]): void{
 const p1:number = args[0];
 const p2:string = args[1];
 const p3:boolean = args[2];
 // Code
}

function resultFunction(p1: number, p2: string, p3: boolean): void{
 // Code
}

let tuple4: [number, string, boolean] = [0, "one", false];
let array4: (number | string | boolean )[] = [0, "one", false];

restFunction(...tuple4);
restFunction(...array4); // Doesn't compile

restFunction(tuple4[0],tuple4[1],tuple4[2]);
restFunction(array4[0],array4[1],array4[2]); // Does not compile
```

元组支持可选参数。语法类似于具有可选参数的函数或具有可选成员的类型。没有值的位自动设置为`undefined`：

```js
let tuple5: [number, string?, boolean?];
tuple5 = [1];
tuple5 = [1, "two"];
tuple5 = [1, "two", false];
```

之前的声明类似于以下声明，其中可选位置也可以设置为`undefined`：

```js
let tuple5: [number, (string | undefined)?, (boolean | undefined)?]
```

在多个地方重用元组时，在类型中设置元组定义可能是有利的。语法与使用关键字`type`定义类型时相同：

```js
type MyTuple = [number, string];
let tuple6:MyTuple = [1, "Test"];
```

总结来说，元组是传递函数信息以及快速返回多个值的便捷方式。然而，一个更好的选择是定义一个带有所需成员的快速接口。这不仅不依赖于位置，而且可以通过允许扩展和交集在许多情况下轻松重用。此外，对象更容易阅读，因为赋值和可读性依赖于名称而不是数字。

# `declare`与`let`/`const`/`var`之间的区别

有可能使用`declare`代替三个声明符之一`let`、`const`或`var`。然而，`declare`在编译期间不会生成任何 JavaScript 代码，并且必须与`let`、`const`或`var`一起使用。`declare`的作用是向 TypeScript 编译器指示变量存在，但定义在别处：

```js
declare let myVariable: string;
```

`declare`的主要作用是在需要定义签名时。在定义文件中使用`declare`是有意义的，因为它只定义类型，实际上并没有声明变量。

`declare`可以用来声明一个模块。声明一个模块用于在 JavaScript 或 TypeScript 的实际代码实现之外编写定义文件：

```js
declare module "messageformat" {
}
```

# 摘要

在本章中，我们讨论了许多使用多个对象的概念来存储泛型信息的方法。我们通过对象字面量和实例化对象来消除了大写和小写对象之间的差异。我们澄清了使用索引签名和映射来存储快速访问数据的两种不同结构。章节继续讨论了如何使用联合和交集来操作几种类型。最后，我们看到了如何解构，以及`declare`与第二章中提到的三种先前声明（*使用原始类型进行类型注册*）的不同之处。

在下一章中，我们将看到如何与`面向对象`编程一起工作。下一章涵盖了如何使用继承、封装和静态函数。接口的概念以及如何在接口中定义构造函数签名将会被解释。下一章将深入 TypeScript 中强大的`面向对象`世界。
