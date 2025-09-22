# 将您的代码转换为面向对象

面向对象有其自己的术语集，TypeScript 依赖于其中很多。在本章中，我们将使用示例讨论 TypeScript 支持的所有面向对象的概念。我们将看到类是什么以及如何将类实例化为对象。我们还将看到如何使用 TypeScript 强类型化构造函数，以及如何使用简写语法直接从构造函数分配类的字段。我们将涵盖封装的可见性原则，如何实现接口，以及如何将抽象引入类。

在本章中，我们将涵盖以下主题：

+   类是什么以及我们如何定义一个？

+   类型如何与类的构造函数相互作用

+   使用 `public`、`private` 和 `protected` 的封装是什么

+   在构造时设置值的字段定义的简化

+   什么是静态？

+   非公共构造函数的使用场景

+   从类或对象字面量中使用对象

+   接口如何在面向对象中发挥作用

+   使用抽象类引入抽象

+   如何拥有只读属性

+   强制从接口实现特定构造函数

# 类是什么以及我们如何定义一个？

面向对象的核心是类。类是对一个对象一旦实例化后可用的定义。类包含开发者认为具有凝聚力的变量和函数。类可以共享同一类的所有实例的信息，或者拥有其自己的数据，这些数据从对象生命周期的开始到结束都是独一无二的。

类的创建从关键字 `class` 开始，后面跟着类的名称。它与创建接口类似：

```js
export class MyClass {
}
```

类可以定义变量和函数。默认情况下，它们都是 `public` 的，这意味着可以通过实例的名称从类外部访问：

```js
export class Variables {
 public a: number = 1;
  private b: number = 2;
  protected c: number = 3;
  d: number = 4; // Also public
}
```

一旦定义了类，就可以实例化它。实例化意味着类变得具体，其内容的生命周期开始。要创建类的实例，必须使用 `new` 关键字。在 `new` 之后是类的名称，后面跟着括号：

```js
const d = new Variables();
```

# 类型如何与类的构造函数相互作用

类的实例化调用类的构造函数。如果没有定义，则不调用任何内容。当没有定义构造函数时，括号没有任何参数。构造函数的目的是为类提供数据初始化：

```js
const d = new Variables();
```

在参数已定义的情况下，初始化必须提供所有非可选参数。在以下代码中，实例化调用具有两个参数的构造函数：

```js
class MyClass {
   constructor(param1: number, param2: string) {
   }
}
const myClassInstance = new MyClass(1, "Must be present");
```

构造函数类似于函数但不能有重写。只能定义一个构造函数。如果没有提供可见性，默认情况下它是 `public` 的：

```js
class MyClass {
   private m1: number;
   private m2: string;
   private m3: number;
   constructor(param1: number, param2: string) {
     this.m1 = param1;
     this.m2 = param2;
     this.m3 = 123;
 }
}
```

然而，构造函数可以通过多个签名进行重载。类似于函数，可以使用分号使用多个定义。在下面的例子中，你可以看到可以使用单个数字或数字和字符串来实例化类：

```js

class ClassWithConstructorOverloaded {
 private p1: number;
 private p2: string;

 constructor(p1: number);
 constructor(p1: number, p2?: string) {
   this.p1 = p1;
   if (p2 === undefined) {
     p2 = "default";
   }
   this.p2 = p2;
 }
}
```

如果一个类扩展了另一个类，它必须使用正确的类型参数调用`super`。继承第二个类的类不需要有相同数量的构造函数参数，也不需要相同的类型。重要的是`super`调用要尊重扩展类的类型。在下面的例子中，`MyClass`有一个接受数字和字符串的构造函数。扩展`MyClass`的类`MyClass2`必须使用数字和字符串调用`super`。这个例子说明了值可以来自类的构造函数，也可以直接计算：

```js
class MyClass2 extends MyClass {
   constructor(p1: number) {
     super(p1, "Here");
   }
}
```

# 使用`public`、`private`和`protected`进行封装是什么意思

在类级别上不可使用`var`、`let`和`const`。类通过使用`public`、`private`和`protected`封装可见性关键字来声明。作用域被限制在类中，三者之间有一些细微的差别。

使用`public`关键字声明的类变量允许变量在类内外都可用。`public`修饰符将类的实例化，允许它在外部读取和写入值：

```js
export class Variables {
   public a: number = 1;
}
const d = new Variables();
d.a = 100;
console.log(d.a);
```

另一方面，`private`声明限制了对其本身的访问。这在读取和写入时都适用：

```js
export class Variables {
   private b: number = 2;
}
const d = new Variables();
d.b = 100; // Not allowed, won’t compile
console.log(d.b); // Not allowed, won’t compile
```

最后，`protected`封装类似于`private`，但它允许我们在类外部本身读取和写入`protected`变量的值。然而，访问被限制在声明`protected`的类以及扩展这个类的类。`protected`变量或函数在类层次结构中共享访问权限。这意味着所有继承具有`protected`成员的类的类都可以访问`protected`成员：

```js
class BaseClass {
  public a: number = 1;
  private b: number = 2;
  protected c: number = 3;
}

class ChildClass extends BaseClass {
  public d: number = 1;
  private e: number = 2;
  protected f: number = 3;
  public f1():void{
   super.a;
   super.c;
   this.a;
   this.c;
 }
}

const child = new ChildClass();
console.log(child.a);
console.log(child.d);
```

在示例中，子类可以访问基类的`public`和`protected`成员。一旦子类被实例化，只有`public`变量可用。私有成员仅在其定义的类中可用。这对于变量和函数都适用。

# 在构造时将字段定义的值集减小的操作

总是从构造函数设置值到字段可能会很麻烦。一种可能性是直接从构造函数签名将字段赋值到类中：

```js
class MyClass3{
 public constructor(private p1:number, public p2:string){}
}
const myClass3 = new MyClass3(1, "2");
console.log(myClass3.p2);
```

通过设置构造函数中每个参数的封装可见性，创建了一个具有相同类型的成员。在先前的例子中，为类创建了两个字段，分别对应参数的名称和类型。下面的代码与先前的例子完全相同，但在定义和赋值方面更为复杂：

```js
class MyClass3Same {
  private p1: number;
  public p2: string;
  public constructor(p1: number, p2: string) {
   this.p1 = p1;
   this.p2 = p2;
 }
}

const myClass3Same = new MyClass3Same (1, "2");
console.log(myClass3.p2);
```

# 什么是静态的？

静态成员是可以不实例化类即可访问的成员。所有静态成员在整个系统生命周期内都是共享的。静态变量和函数与类相关联，而不是与类的实例或特定对象相关联。如果你来自 JavaScript，你可以将静态视为与实例的原型链实例关联的成员。

与许多其他语言相反，TypeScript 不允许我们拥有静态类。它并没有减少多少，因为如果你需要一个静态类，你只需要将函数直接放在模块内部，而不是放在类中。如果你想将所有静态类放在一个类中，并防止你的库的用户实例化该类，你可以将类标记为抽象。抽象类不能在不被扩展的情况下实例化：

```js
abstract class FakeStaticClass {
 public static m1: number;
 public static f1(): void { }
}

console.log(FakeStaticClass.m1);
FakeStaticClass.f1();

const instance1 = new FakeStaticClass(); // Doesn't compile
```

静态成员可以是所有封装可见性：`public`、`protected` 或 `private`。然而，只有 `public` 可见性可以从类外部访问。`private` 和 `protected` 在定义状态成员的类内部可用。受保护的成员可以从扩展了具有静态成员的另一个类的类中访问：

```js
class StaticClass {
  public static ps: number;
  private static privateStatic: number;
  protected static protecStatic: number;
}

StaticClass.ps = 1;
```

只有 `public static` 成员可以从类外部访问。`private` 和 `protected` 在类内部可访问。要使用任何静态成员，必须在成员使用之前指定类的名称。这个规则在类内部也是必要的。`this` 指针仅在实例内部可用，由于静态不是任何实例的一部分，而是 `class` 的一部分，因此不能通过实例的 `this` 指针访问：

```js
class StaticClass {
  public static ps: number;
  private static privateStatic: number;
  protected static protecStatic: number;

  public nonStaticFunction():void{
    StaticClass.ps;
    StaticClass.privateStatic;
    StaticClass.protecStatic;
  }
}
```

# 非公共构造函数的使用场景

`private` 构造函数取消了从外部实例化类的可能性。以下代码无法编译，因为构造函数是 `private` 的。如果构造函数是 `protected` 的，情况也会相同：

```js
class PrivateConstructor{
 private constructor(){
 }
}

const pc = new PrivateConstructor(); // Does not compile
```

在这种情况下，实例化类的唯一方法是通过使用创建类型为 `class` 的对象的 `public static` 函数，并返回它。在以下代码中，`private` 构造函数创建了一个实例；要访问此实例，使用 `GetInstance`，它是一个静态方法，不需要实例即可调用：

```js
class SingletonClass {
 private static instance: SingletonClass;
 private constructor() {
 SingletonClass.instance = new SingletonClass();
 }

 public static GetInstance(): SingletonClass {
   return SingletonClass.instance;
 }
}
const singletonClass = SingletonClass.GetInstance();
```

一个已知的模式是拥有一个 `SingletonClass`。该类只有一个实例存在，并且可以通过使用一次 `new` 并始终返回相同实例的单个函数来管理这种控制。另一个用例是拥有一个创建所有实例的工厂。

# 使用类对象与对象字面量

对象字面量构建快速，不需要通过构造函数或 `public` 成员传递数据来填充对象。对象字面量是移动数据的高效方式。你可以读取 JSON 文件，从 Ajax 调用或许多其他来源接收数据，并将其转换为所需类型：

```js
ajax.then((response:any)=>{
   return response as DataTyped;
};
```

然而，如果您有许多函数或更复杂的逻辑需要封装，类更实用。原因是使用对象字面量需要手动在每个实例上分配函数。此外，类可以包含您可能不想公开的`private`函数，提供`private`/`protected`和接口的封装。对象字面量的字段是`public`且可访问的。在以下示例中，我们看到从 Ajax 调用返回的数据。预期的类型是`ObjectDefinition`，它有一个函数。这个函数不像类在初始化期间那样免费提供。因此，它必须附加到对象上。在这种情况下，我们需要引用一个具有函数的变量。对于复杂的定义，这可能会很繁琐：

```js
interface ObjectDefinition {
  m1: string;
  funct1: () => void;
}

let ajax: any;
const funct1 = () => { };

ajax.then((response: any) => {
  const r = response as ObjectDefinition;
  r.funct1 = funct1;
  return r;
});
```

可以通过有一个`function`来构建每个对象字面量，通过附加函数来减轻前面示例的负担。在这种情况下，`function`返回对象字面量的类型。这个`function`充当构造函数：

```js
function createObj(m1: string): ObjectDefinitionClass {
  return {
   m1: m1,
   funct1: () => { }
  }
}

ajax.then((response: any) => {
 const r = response as Model;
 return createObj(r.m1);
});
```

使用类编写的相同代码如下所示：

```js
class ObjectDefinitionClass implements ObjectDefinition {
 public m1: string;
 public funct1(): void { }

 constructor(param1: string) {
  this.m1 = param1;
 }
}

ajax.then((response: any) => {
 const r = response as ObjectDefinition;
 return new ObjectDefinitionClass(r.m1);
});
```

在那个特定的情况下，有趣的是将字段从函数中分离出来，并通过构造函数使用接口传递所有字段。以下是使用接口变量和函数的第三个版本：

```js
interface Model {
 m1: string;
}

interface Funct {
 funct1: () => void;
}

class ObjectDefinitionClass2 implements Model, Funct {
 public m1: string;
 public funct1(): void { }
 constructor(param1: Model) {
   this.m1 = param1.m1;
 }
}

ajax.then((response: any) => {
 const r = response as Model;
 return new ObjectDefinitionClass2(r);
});
```

在可测试性的方面，类具有允许任何成员容易被模拟的优势。以下是一个使用 Jest 库的简单示例：

```js
const forTesting = new ObjectDefinitionClass("1");
forTesting.funct1 = jest.fn();
```

# 接口在面向对象中的用途

接口扮演着许多角色。我们看到了您可以使用接口为特定对象定义一个合同。然而，接口可以做更多的事情。

接口定义了您希望库的消费者看到和使用哪些成员。`public`、`private`和`protected`可见性关键字服务于相同的目的。然而，在某些情况下，您可能需要具有`public`成员，同时又不允许每个人使用它们。一个原因可能是您希望对单元测试有深入访问，因此将大多数成员设置为`public`允许您进行黑盒函数测试。然而，这可能会暴露太多。因此，接口可以定义所有可访问的成员，并由类实现。您可以直接传递类的引用，而接口则在外部分布，您可以在内部使用类：

```js
class ClassA {
 public mainFunction(): void 
{
  this.subFunction1();
  this.subFunction2();
 }

 private subFunction1(): void { }
 private subFunction2(): void { }
}
```

类有两个由主函数`mainFunction`调用的`private`函数。然而，封装不允许我们不使用一些黑客手段来访问这些函数而进行单元测试。我们希望避免将主对象强制转换为任何类型来访问函数，因为这些函数如果发生变化，测试可能会失败，不是因为逻辑错误，而是因为 TypeScript 无法重构函数，因为类型被强制转换为任何类型。更好的做法是使用接口来保持类型始终存在：

```js
interface IClassA {
 mainFunction(): void;
}

class ClassA2 implements IClassA {
 public mainFunction(): void {
   this.subFunction1();
   this.subFunction2();
 }

 public subFunction1(): void { }
 public subFunction2(): void { }
}
```

虽然一切都是`公共`的，但整个系统都在使用`IClassA`而不是直接提供所需封装的类。单元测试可以使用该类并访问原始的`私有`函数。

第二个案例中，接口的亮点在于它允许我们有许多特定类型的具体实现。你可以定义一个接口，该接口将被函数消费，并且有多个此类实现。在以下示例中，我们有一个`consume`函数，它接受`IElement`作为输入。`IElement`有两个具体实现，这使代码具有许多元素实现的灵活性。这有助于通过具有表示执行任务所需的最小成员集的类型来减少消费函数中的自定义代码：

```js
interface IElement {
 m1: string;
}

class E1 implements IElement { m1: string = "E1->m1"; a: number = 1; }
class E2 implements IElement { m1: string = "E2->m1"; b: boolean = true; }

class ClassB {
 public consume(element: IElement): void { }
}
```

# 使用抽象类引入抽象

抽象是一种面向对象的概念，它允许我们有一个基类，该类将函数的实现委托给扩展`抽象`类的类。

以下示例通过实例化自定义逻辑类来创建主类。它调用主函数，该函数将执行`抽象`函数。对于`MainClass`类，`抽象`函数是一个黑盒。它只知道其名称、参数类型和返回类型。示例按照特定的顺序执行注释块代码 A-C-B：

```js
abstract class MainClass {
 public mainCoreLogic(): void {
   // Code here [A]
   this.delegatedLogic();
   // Code here [B]
 }

 public abstract delegatedLogic(): void;

}

class CustomLogic extends MainClass {

 public delegatedLogic(): void {
   // Do some custom logic here [C]
 }

}

const c: MainClass = new CustomLogic();
c.mainCoreLogic();
```

当将计算值传递给`抽象`类并且该类也返回自定义计算的结果时，`抽象`功能非常强大。这里有一个第二版本，展示了在主类保持不变的情况下，如何出现两种不同的实现。主类现在命名为`Calculus`，有一个`公共`函数，该函数接受两个数字并返回一个`布尔值`。它对参数进行一些操作并调用委托逻辑。对值的处理对主类来说是未知的。操作的重要部分是随后使用的那个结果。在通过扩展类消费`抽象`类的类的一侧。它必须提供所有抽象函数或字段。每个`抽象`成员在扩展级别都成为`公共`字段。在示例中，逻辑将两个值相乘并返回指定的类型：

```js
abstract class Calculus {
 public isAboveZero(a: number, b: number): boolean {
   const positiveA = Math.abs(a);
   const positiveB = Math.abs(b);
   const result = this.delegatedLogic(positiveA, positiveB);
   return result > 0;
 }

 public abstract delegatedLogic(a: number, b: number): number;
 }

 class AddLogic extends Calculus {
   public delegatedLogic(a: number, b: number): number {
     return a * b;
   }
 }

 const multi: Calculus = new AddLogic();
 multi.isAboveZero(1, 2);
```

代码可以通过提供执行逻辑的参数以非面向对象的方式编写。以下是相同代码的版本，没有使用`抽象`：

```js
 class CalculusWithoutAbstract {
   public constructor(private delegatedLogic: (a: number, b: number) => number) {
   }

   public isAboveZero(a: number, b: number): boolean {
     const positiveA = Math.abs(a);
     const positiveB = Math.abs(b);
     const result = this.delegatedLogic(positiveA, positiveB);
     return result > 0;
   }
}

const multi2: CalculusWithoutAbstract = new CalculusWithoutAbstract((a, b) => a * b);
multi2.isAboveZero(1, 2);
```

没有使用`抽象`的版本将函数调用传递给类的构造函数，而不是传递抽象函数。这两种方法之间的替换是个人偏好的问题。两者之间的主要区别在于，使用抽象强制`抽象`实现为公共的，而委托函数可以保持`私有`。然而，解决可见性问题的一种方法是用基类而不是子类进行初始化：

```js
const multi: Calculus = new AddLogic(); // Expose only the main function
const multi: AddLogic = new AddLogic(); // Expose the delegate function
```

# 如何拥有只读属性

只读字段只能初始化一次，在实例的生命周期内不需要更改。可以使用`readonly`关键字在接口中指定一旦字段被设置，值就不会改变：

```js
interface I1 {
 readonly id: string;
 name: string;
}

let i1: I1 = {
 id: "1",
 name: "test"
}

i1.id = "123"; // Does not compile
```

它可以在类级别，此时值只能直接在声明或构造函数中设置。当值在字段声明旁边初始化时，这个值仍然可以被构造函数重新定义。以下示例展示了这种边缘情况。然而，它只可以在声明或仅在`constructor`级别设置，这通常是情况：

```js
class C1 {
 public readonly id: string = "C1";

 constructor() {
   this.id = "Still can change";
 }

 public f1(): void {
   this.id = 1; // Doesn't compile
 }
}
```

使用`static`的只读可以用来为特定类创建一个常量。在类级别不允许使用`const`。如果你想在特定类的上下文中集中一个值，使用`public`、`static`和`readonly`是一个可接受的模式：

```js
class C2 {
 public static readonly MY_CONST: string = "TEST";
 public codeHere(): void {
   C2.MY_CONST;
 }
}
```

# 从接口强制执行特定的构造函数

这很棘手，因为你不能通过接口强制执行构造函数的形状。然而，你可以使用接口来确保通过参数传递的类具有特定的构造函数。这个过程需要两个接口。一个是构造的返回类型，另一个是用于参数的接口：

```js
interface ConstructorReturnType {    
  member1: number;
  funct(): void;
}
interface EntityConstructor {
  new(value: number): ConstructorReturnType;
}
```

在这个示例中，第一个接口有两个成员：一个字段和一个函数。接口的定义无关紧要，它可以是你想要获取函数实例的任何内容。第二个接口有一个构造函数，称为*newable*。它使用`new`关键字和输入参数以及它需要创建的内容。类型应该是创建的第一个接口：

```js
function entityFactory(ctor: EntityConstructor, value: number): ConstructorReturnType {
   return new ctor(value);
}
```

下一步是创建一个函数，该函数接受`newable`函数并将其设置为参数类型。可选地，你可以有更多参数。在示例中，传递了一个构造函数的值。函数的返回类型必须是`newable`函数返回的类型。在这个函数中，你可以调用`new`后跟具有`newable`函数定义的接口参数。代码实例化了通过参数传递的类的实例。只有遵守`newable`函数类型合同的类才被接受：

```js
class Implementation1 implements ConstructorReturnType {

 public member1: number;

 constructor(value: number) {
   this.member1 = value;
 }

 public funct(): void {
 }

}
let impl1 = entityFactory(Implementation1, 1);
```

在前面的代码中，`Implementation1`类实现了返回的实现，因此将成为此函数的候选者。它还有一个接受单个数字参数的构造函数，该参数将由函数调用。

另一方面，以下代码无法编译，因为该类没有继承由`newable`函数定义的返回类型：

```js
class Implementation2 {
   constructor(value: number) { }
}

let impl2 = entityFactory(Implementation2, 1);
```

以下是一个可能看起来不合法但可以编译的示例代码。它扩展了返回的类，但不尊重需要它具有值的`newable`函数参数。这是有效的，因为定义仅关于返回的对象，而不是构造函数。构造函数使用参数调用，但类不必处理它。在下面的代码中，打印`arguments`变量显示它具有作为第一个参数传递的`1`值，即使类没有明确要求：

```js
class Implementation3 implements ConstructorReturnType {
  public member1: number = 1;
  constructor() {
    console.log(arguments);
  }

  public funct(): void {
  }
}

let impl3 = entityFactory(Implementation3, 1);
```

# 摘要

在本章中，我们探讨了众多面向对象特性。TypeScript 正在缩小与已知面向对象编程语言（如 C#或 Java）的差距。作为 JavaScript 的超集，TypeScript 必须弥补 JavaScript 在这方面的一些弱点，但最终凭借其众多特性，以面向对象的方式编写专业应用程序成为可能。

我们学习了如何使用封装，这使我们能够控制字段和函数的可见性。我们讨论了如何强类型化构造函数，以及如何使用接口让类实现。在一个类内部，我们看到了如何拥有静态函数和抽象函数。下一章将介绍我们如何精确地识别我们正在操作的类型、对象或变量，从而利用它们各自独特的特定成员。我们将看到如何使用 JavaScript 类型检查器，如`typeof`和`instanceof`，以及如何对结构化类型使用判别器和定义守卫。
