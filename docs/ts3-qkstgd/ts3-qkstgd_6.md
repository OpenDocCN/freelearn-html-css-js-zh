# 第六章：通过泛型复用代码

本章是基于前几章引入的概念构建的。本章通过增强类型使其泛型化来构建在概念之上。本章涵盖了基本主题，如定义泛型类和接口。通过本章，我们进入更高级的主题，如泛型约束是内容的一部分。本章的目标是使你的代码更加泛型，以增加类、函数和结构的可复用性，减少代码复制的负担。

本章内容涵盖以下内容：

+   泛型如何使你的代码可复用

+   泛型类型可接受的数据结构类型

+   如何约束泛型类型

+   泛型和交集

+   默认泛型

+   泛型可选类型

+   使用联合类型进行泛型约束

+   使用 `keyof` 限制字符串选择

+   限制对泛型类型成员的访问

+   使用映射类型减少类型创建

+   TSX 文件中的泛型类型

# 使用泛型代码提高复用性

泛型几乎在所有类型语言中都是可用的。它允许以可复用的方式转换你的代码，而无需依赖于不安全的类型转换来检索对象中存储的值。没有泛型，有不同方式来实现复用。例如，你可以有一个具有 `any` 类型的接口。

这样会使字段能够接收任何类型的对象，因此可以在许多场景中复用：

```js
interface ReusableInterface1 {
    entity: any;
}
```

一种稍微好一点的方法是指定我们是否想要接受原始类型或仅接受对象：

```js
interface ReusableInterface2 {
   entity: object;
}

const ri2a: ReusableInterface2 = { entity: 1 }; // Does not compile
const ri2b: ReusableInterface2 = { entity: { test: "" } };
```

在这两种情况下，问题出现在我们想要使用可复用字段的时候。使用 `any` 或 `object` 也会出现相同的结果，即我们无法访问原始变量的成员，因为我们没有方法知道原始类型是什么：

```js
const value = ri2b.entity; // value -> "object"
```

在这段代码中，如果不将实体转换回原始类型，就无法使用实体的 `.test` 方法。在这个特定的类型中，它是一个匿名类型，但仍然可能：

```js
const valueCasted = value as { test: string };
console.log(valueCasted.test);
```

然而，泛型可以通过将对象的类型引入类型的定义中，而不干扰要隔离的单个类型，来消除访问原始类型的障碍。要创建泛型函数、接口或类，你需要使用较小的或较大的符号 `< >`：

```js
interface MyCustomTypeA {
   test: string;
}

interface MyCustomTypeB {
   anything: boolean;
}

interface ReusableInterface3<T> {
   entity: T;
}
```

中括号之间的名称并不重要。在下面的代码中，我们使用两个自定义接口来使用实体，并将它们用作类型 `T`。我们还在直接使用一个数字。我们可以使用所有可能的类型，因为我们还没有设置泛型约束：

```js
const ri3a: ReusableInterface3<MyCustomTypeA> = { entity: { test: "yes" } };
const ri3b: ReusableInterface3<MyCustomTypeB> = { entity: { anything: true } };
const ri3c: ReusableInterface3<number> = { entity: 1 };
```

最大的优势是，如果我们访问对象，字段实体是 `T` 类型，这取决于对象是如何创建的：

```js
console.log(ri3a.entity.test); // "yes" -> string
console.log(ri3b.entity.anything); // true -> boolean
console.log(ri3c.entity); // 1 -> number
```

# 泛型类型可接受的数据结构类型

通用类型的概念不仅限于接口。通用类型不仅适用于类型，还适用于类，并且可以转换函数。定义通用类型的括号的位置紧接在接口名称、类型或类名称之后。我们稍后将会看到它也必须紧跟在函数名称之后。通用类型可以用于具有通用字段、通用参数、通用返回类型和通用变量：

```js
type MyTypeA<T> = T | string; // Type

interface MyInterface<TField, YField> { // Interface wiht two types
  entity1: TField;
  myFunction(): YField;
}

class MyClass<T>{ // Class
 list: T[] = [];
 public displayFirst(): void {
   const first: T = this.list[0]; // Variable
   console.log(first);
 }
}
```

通用类型可以同时具有多个通用类型，允许多个字段或函数参数具有不同类型的类型：

```js
function extractFirstElement<T, R>(list: T[], param2: R): T {
  console.log(param2);
  return list[0];
}
```

# 限制通用类型

在本章之前的代码示例中，我们使用了类型对象来确保在接口中没有传递原始类型。使用对象的问题在于，当你从实体返回时，你无法获得初始类型。以下代码说明了这个问题：

```js
interface ReusableInterface2 {
  entity: object;
}

const a = {
  what: "ever"
};

const c: ReusableInterface2 = { entity: a };
console.log(c.entity.what); // Does not compile because "what" is not of object
```

有可能保持原始类型，并通过`extends`关键字约束不允许使用原始类型：

```js
interface AnyKindOfObject {
  what: string;
}

interface ReusableInterface3<T extends object> {
  entity: T;
}

const d: ReusableInterface3<AnyKindOfObject> = { entity: a };
console.log(d.entity.what); // Compile
```

`extends`关键字允许指定传递给通用类型的对象必须存在的最小结构。在这种情况下，我们传递了一个对象。然而，我们可以扩展任何最小结构、接口或类型：

```js
interface ObjectWithId {
 id: number;
 what: string;
}

interface ReusableInterface4<T extends { id: number }> {
 entity: T;
}

const e: ReusableInterface4<AnyKindOfObject> = { entity: a }; // Doesn't compile
const f: ReusableInterface4<ObjectWithId> = { entity: { id: 1, what: "1" } }; // Compile
const g: ReusableInterface4<string> = { entity: "test" }; // Doesn't compile
```

之前的代码示例中有两个变量无法编译。第一个变量被设置为一个错误的对象。第三个变量被设置为一个字符串，但由于字符串没有`id:number`字段，因此无法满足通用约束。第二个变量可以编译，因为实体遵守了约束。最后，这里有一个具有约束的通用类型的示例：

```js
interface ReusableInterface5<T extends ObjectWithId> {
   entity: T;
}
```

除了可以访问完整的原始类型外，具有约束的通用类型还允许从实现通用类型的类或函数中访问约束字段。第一个代码示例，使用`function`，在函数签名中直接具有约束。它允许仅访问约束的字段：

```js
function funct1<T extends ObjectWithId>(p: T): void {
   console.log(`Access to ${p.what} and ${p.id}`);
}
```

类似地，一个类允许在其任何函数中使用通用约束的字段。在下面的代码中，函数遍历`T`的通用列表。由于`T`扩展了具有`what`属性和`id`的`ObjectWithId`，因此两者都是可访问的：

```js
class ReusableClass<T extends ObjectWithId>{
 public list: T[] = [];
 public funct1(): void {
   this.list.forEach((p) => {
    console.log(`Access to ${p.what} and ${p.id}`);
    });
 }
}
```

# 通用类型和交集

通用类型和交集工作得很好。它允许使用未确定类型，并通过已知类型或另一个通用类型的组合创建第二个类型。在下面的代码中，有一个通用函数，它接受一个必须至少符合`User`对象结构的类型`T`。函数的返回类型是参数传递的通用类型与新的`WithId`结构交集后的通用类型。这意味着传递的任何类型都将通过新的结构得到增强。在代码中，将`Developer`类型传递给函数，函数返回`Developer+WithId`。这是一个新类型，没有在任何地方定义，但仍然是强类型的：

```js
interface WithId {
 id: number;
}

interface User {
 name: string;
}

interface Developer extends User {
 favoriteLanguage: string;
}

function identifyUser<T extends User>(user: T): T & WithId {
 const newUser = (<any>Object).assign({}, user, { id: 1 });
 return newUser;
}

const user: Developer = { name: "Patrick", favoriteLanguage: "TypeScript" };
const userWithId = identifyUser(user);
console.log(`${userWithId.name} (${userWithId.id}) favorite language
 is ${userWithId.favoriteLanguage}`);
```

代码显示我们可以使用返回类型，并且所有三个成员都可用。

可以将许多通用类型组合在一起，例如：

```js
function merge<T, U>(obj1: T, obj2: U): T & U {
   return Object.assign({}, obj1, obj2);
}
```

`merge`函数接受两种不同的类型，并使用 JavaScript 的`assign`函数将它们合并。该函数返回两种类型的交集。如果我们深入研究`Object.assign`函数的定义，我们会意识到它也在利用泛型的交集。以下是 ES2015 的`Object.assign`定义文件：

```js
assign<T, U, V>(target: T, source1: U, source2: V): T & U & V;
```

# 默认泛型

越来越多地使用泛型，你可能会发现，对于系统中的特定情况，你总是使用相同的类型。它几乎可以不是泛型，而是一个特定类型。在这种情况下，为你的泛型使用默认类型是有趣的。默认泛型类型允许避免必须指定类型。默认泛型也被称为可选类型。

TypeScript 使用等于号之后泛型签名中指定的类型：

```js
interface BaseType<T = string> {
 id: T;
}
let entity1: BaseType;
let entity2: BaseType<string>;
let entity3: BaseType<number>;
```

声明了三个变量。第一个和第二个完全相同：它们期望一个具有`string`类型`id`的对象。最后一个是一个数字。第一个和第二个完全相同的原因是第一个声明依赖于默认类型。默认类型在接口的泛型定义中指定在类型名称`T`之后，使用等号允许赋值。

在多个默认值的情况下，如果没有后续的可选类型，则只能使用可选类型。以下代码显示了相同的接口，第一个无法编译，因为它在所需类型之前有一个可选泛型类型：

```js
interface User<T = string, U> { // Does not compile
  id: T;
  name: U;
}

interface User<U, T = string> {
  id: T;
  name: U;
}
```

默认泛型可以有约束，并且它似乎尊重其默认类型的约束。以下代码无法工作，因为默认类型被设置为数字。然而，约束指出结构必须有一个类型为`number`的`id`：

```js
interface WithId {
 id: number;
}

interface UserWithDefault<T extends WithId = number> { } // Does not compile
```

然而，如果我们将默认类型更改为`User<number>`，则可以编译。原因是用户界面有一个类型为`T`的`id`字段。默认类型与扩展约束不兼容，该约束要求`id`为数字类型。这意味着如果没有在默认签名中明确提及`User`的泛型类型，代码将无法编译：

```js
interface User<T = string> {
 id: T;
}

interface WithId {
 id: number;
}

interface UserWithDefault<T extends WithId = User<number>> { }
// Does not compile because User<string>
interface UserWithDefault<T extends WithId = User { }
```

当类型不是显式指定或 TypeScript 无法推断类型时，使用默认类型。

# 泛型可选类型

泛型类型可以在函数或类中是可选的。当可选且泛型时，类型变为一个空对象或未定义：

```js
function shows<T>(p1?: T): void {
 console.log(p1);
}

shows(); // p1 is {} | undefined
shows("123");
shows(123);
```

为可选类型提供一个默认值将参数从空对象更改为默认类型：

```js
function shows<T = number>(p1?: T): void {
 console.log(p1);
}
shows(); // p1 is number | undefined
```

# 联合类型的泛型约束

在泛型定义的`extends`子句中，使用联合类型有一些空间。虽然不能使用`discriminator`，但可以与数组进行比较。以下对象允许一个类型和相同类型的数组。您可以使用`instanceOf`缩小到任何两种类型之一，并操作参数值：

```js
interface ObjectWithAge {
 kind: "ObjectWithAge";
 age: number;
}

function funct2<T extends ObjectWithAge | ObjectWithAge[]>(p: T): T {
 if (p instanceof Array) {
   return p[0];
 }
 return p;
}
```

尝试使用区分符扩展两个不同的对象目前不起作用。

# 使用 `keyof` 限制字符串选择

在 JavaScript 中，字符串的使用无处不在。一种模式是使用方括号动态访问对象的成员：

```js
interface Human {
 name: string;
 birthdate: Date;
 isHappy: boolean;
}

const me: Human = {
 name: "Patrick",
 birthdate: new Date(1912, 0, 1),
 isHappy: true
}

console.log(me["name"]);
```

代码的问题在于 `name` 是一个字符串，可以是任何内容。我们可以在括号中设置 `firstname`，代码就会编译。在运行时，控制台会显示 `undefined`。为了避免陷入选择不存在成员的陷阱，我们可以使用 `keyof`，它将返回一个对象所有成员的联合。联合是所有成员名称的 `字符串字面量`。

在使用 `keyof` 之前，创建一个尝试通过字符串访问属性失败的 `function`，而不定义索引签名（参见本书中的 *索引签名*）：

```js
function showMe1(obj: Human, field: string): void {
  console.log(obj[field]); // Does not compile
}
```

然而，使用 `keyof` 的相同函数在没有索引签名的情况下也能工作。原因是 TypeScript 知道你并没有尝试访问可能不存在的字段。使用方括号访问的目标是访问存在的成员：

```js
function showMe2(obj: Human, field: keyof Human): void {
  console.log(obj[field]);
}
```

`keyof` 允许在 `keyof` 关键字之后以字符串格式指定类型中的唯一字段。在之前的代码示例中，只有字符串 `name`、`birthdate` 和 `isHappy` 可以输入，而不会让编译器显示错误：

```js
showMe2(me, "name"); // Compile
showMe2(me, "NOname"); // Does not compile
```

# 限制泛型类型的成员访问

可以在约束中使用泛型与 `keyof` 来仅指定泛型对象中的字符串格式成员名称。

在以下代码中，我们在第一个参数中传递一个对象，在第二个参数中只接受第一个参数对象的成员名称。约束语法与使用 `extends` 后跟 `keyof` 和第一个泛型类型相同。返回类型是所选成员的返回类型，可以通过使用第一个泛型与第二个泛型的索引签名来访问：

```js
function prop<TObject, TMember extends keyof TObject>(
 obj: TObject,
 key: TMember
): TObject[TMember] {
 return obj[key];
}

interface ObjectWithName {
 name: string;
 age: number;
}

const obj1: ObjectWithName = { name: "Patrick", age: 212 };
const result1: string = prop(obj1, "name");
const result2: number = prop(obj1, "age");
```

语法提供了良好的类型安全性，在指定对象成员方面，这些成员可以是各种潜在类型，同时也提供了关于返回类型的安全性。如果 `name` 从 `string` 更改为一个具有许多成员的丰富对象，那么消耗此函数返回值的代码将在编译时中断。如果名称更改，重构工具也会更改它。然而，如果更改没有使用任何重构工具，编译器将捕获名称不是有效的情况。

以下代码展示了如何使用 `keyof` 确保函数返回所需成员的名称。函数第一次被调用时返回 `name`；然而，第二次调用不会编译，因为成员的名称不在泛型类型中：

```js
function nameof<T>(instance: T, key: keyof T): keyof T {
   return key;
}

const name1 = nameof(obj1, "name");
const name2 = nameof(obj1, "nasme"); // Does not compile
console.log(name1); // "name"
```

# 使用映射类型减少类型创建

当你开始编写所有对象时，你可能会遇到需要几乎相同类型但有一些细微差异的情况。你可能想要完全相同的属性但所有可读的，当主要类型有几个只读类型时。你可能想要所有字段都是可选的，以便允许部分对象更新，或者你可能想要通过将所有属性设置为只读来密封对象。你可能甚至想要所有属性都是字符串，因为你的表单只处理字符串值，但后来有实际的接口或类型，具有良好的类型。TypeScript 允许从现有类型创建动态类型。这种类型的转换被称为*映射类型*。映射类型允许减少复制对象以更改类型属性的负担，同时保持定义的结构相同。

TypeScript 自带了许多可以使用的映射类型，无需自己构建映射类型。以下有两个常见的例子：

```js
type Readonly<T> = {
 readonly [P in keyof T]: T[P];
}

type Partial<T> = {
 [P in keyof T]?: T[P];
}
```

第一个，`Readonly`，接受一个泛型类型并遍历其所有成员，添加`readonly`。它还返回带有`T[P]`的相同类型。第二个，`Partial`，在名称后添加`*?*`字符，这意味着每个字段都变为可选：

```js
interface MyEntity {
 readonly id: number;
 name: string;
}

const e1: MyEntity = { id: 1, name: "Name1" };
```

如果我们想让变量被密封且完全不可编辑，我们可以使用`Readonly`：

```js
const e1: MyEntity = { id: 1, name: "Name1" };
const e2: Readonly<MyEntity> = e1;
e1.name = "I can change";
e2.name = "I cannot change"; // Does not compile
```

如果你想让某人只能修改你的实体的一部分，然后合并结果，你可以使用`Partial`：

```js
function edit<T>(original: T, obj: Partial<T>): T {
 const returnObject: T = Object.assign({}, original, obj);
 return returnObject;
}

edit(e1, { name: "Super" }); // The returned object is: {id: 1, name: "Super"}
edit(e1, { memberNoExist: "Super" }); // Does not compile
```

你可以通过使用类型关键字并创建一个带有*in*操作符的名称来创建自己的映射类型，以遍历成员并定义转换。重要的是要注意，我们不是操作数据，而是操作类型。这意味着如果你正在更改类型，你仍然需要操作对象以获得预期的形状，以满足映射类型。以下有两个自定义类型的例子。第一个为所有成员返回一个字符串。第二个移除了`Readonly`。你可以看到属性前的减号，这表示 TypeScript 知道成员的修饰符被移除了：

```js
type Stringify<T> = { [P in keyof T]: string; };
type UnReadonly<T> = { -readonly [P in keyof T]: T[P]; };
```

映射类型可以堆叠以创建一个最终类型，该类型结合了所有映射类型。在以下示例中，我们堆叠了两个映射类型：

```js
const e3: UnReadonly<Stringify<MyEntity>> = e1;
```

代码是合法的，但不起作用。原因是 TypeScript 推断出`e1.id`是数字类型，而有人试图将其自动转换为字符串，这是不会发生的。正如提到的，映射类型仅作为转换使用，并要求你有适当的代码。

这里有一个快速且简单的函数可以完成这个任务。不要在生产环境中使用此代码，因为它不涵盖转换为字符串属性（特别是对于`object`和`array`），但它说明了所需的转换：

```js
function castAllFieldToString<T>(obj: T): Stringify<T> {
 let returnValue: any = {};
 for (var property in obj) {
   if (obj.hasOwnProperty(property)) {
     returnValue[property] = obj[property].toString();
   }
 }
 return returnValue as Stringify<T>;
}

const e3: UnReadonly<Stringify<MyEntity>> = castAllFieldToString(e1);
e3.id = "123";
```

# TSX 文件中的泛型类型

TSX 是 JSX（JavaScript XML 扩展语言）的等价物。在很长一段时间里，TypeScript 支持 TSX，但对泛型并不友好。主要原因在于 TSX 和泛型语法共享了尖括号，这导致在 TSX 文件中编译器会错误地解释泛型类型。然而，情况已经改变，TypeScript 区分了何时使用方括号来显式定义泛型组件的类型。在下面的代码片段中，你可以看到 `CallGenericComponent` 尝试渲染一个泛型组件。返回值使用一个初始的左尖括号来初始化 TSX 组件。接下来的第二个左尖括号是用来定义类型的：

```js
interface MyTsxProps<T> {
 item: T;
}

class CallGenericComponent extends React.Component<{}>{
  public render(): JSX.Element {
    return <MyTsxComponent<string> item={"123"} />
  }
}

class MyTsxComponent<T> extends React.Component<MyTsxProps<T>>{
  // ...
}
```

使用尖括号可以避免将组件定义为变量并实例化它，这不仅需要多行代码，还需要使用 `any` 类型进行类型转换。此外，可读性受损，外部人员难以理解其意图。

# 摘要

在本章中，我们了解了如何通过使用泛型来转换代码。TypeScript 提供了约束来限制可以传递给泛型的内容，我们也看到了如何利用约束来指导用户传递什么内容。我们还看到了使用 `keyof` 的强大之处，它允许我们动态地从类型中获取成员。我们还看到了如何以泛型方式操作类型，使用映射类型。
