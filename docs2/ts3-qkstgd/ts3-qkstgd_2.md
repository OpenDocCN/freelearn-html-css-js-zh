# 使用原始类型引入类型

原始类型是所有基本支持的价值类别。每种类型代表一个值域，其中格式的一致性得到强制执行。JavaScript 有一个有限的原始类型集合，这些类型只能通过将值分配给变量来推断。例如，一个值可以是数字、日期、布尔值、字符串等。将不同模型的后继值分配给单个变量是允许的。副作用是类型的突变，这增加了任何 JavaScript 程序的复杂性。然而，TypeScript 可以强制类型不可变性，这降低了潜在错误值误导应用程序正确执行的风险。此外，TypeScript 根据附加到特定值的显式类型提供对哪些操作可以使用支持。本章说明了变量的作用域、未定义变量和空变量之间的细微差别，以及如何使变量可选或必需。在本章结束时，读者将处于所有变量都正确声明、由 TypeScript 支持准确类型的情况。原始类型和非原始类型之间的区别将不再是一个难题。使用 `enum` 或 `symbol` 将变得自然，每当在系统中引入新的域对象时，创建新类型将变成一种习惯。

本章将涵盖以下内容：

+   `var`、`let` 和 `const` 之间的区别

+   如何在不指定类型的情况下进行强类型声明

# var、let 和 const 之间的区别

TypeScript 有多种声明变量的方式。你可以使用以下三个关键字之一在函数或全局作用域中定义变量：`var`、`let` 和 `const`。此外，你还可以在类级别上使用 `public`、`private` 或 `protected` 来定义变量。

# 使用 `var` 声明

声明变量的最基本方式是使用关键字 `var`。它是最古老的声明方式，但由于一些怪癖，它是最不受欢迎的方式。`var` 的主要问题是它在执行上下文中声明，这意味着在函数作用域或全局作用域内。如果意外地将值分配给未显式使用 `var` 声明的变量，那么该变量的作用域就是全局作用域。以下是一个例子：

```js
function f1(){
   a = 2; // No explicit "var", hence global scope instead of function scope
}
```

使用 `var` 声明可以通过 JavaScript 的严格模式变得更加严格，这样 TypeScript 可以通过编译器选项中的 `alwaysStrict` 自动在每一个文件上开启。否则，你必须记住 `var` 声明的变量是在代码执行之前创建的。没有 `var` 关键字的变量直到分配给它们的代码执行时才存在。在 JavaScript 中，可以不声明就分配变量，但在 TypeScript 中则不是这样：

```js
a = 2; // Won't create a variable in TypeScript
```

虽然 TypeScript 可以防止未声明的变量，但它不能防止 `var` 声明受到 **提升** 的影响。这个问题源于 JavaScript，其中使用 `var` 的声明会在其他代码片段之前处理，将变量声明带到作用域（函数或全局）的顶部。微妙之处在于声明被提升，但初始化没有。也就是说，TypeScript 不允许你使用在其使用下定义的变量：

```js
console.log("Test", a); // Won’t allow to use the variable in TypeScript
var a = 2;
```

最后，`var` 允许你多次定义变量，以覆盖初始声明或初始化：

```js
var a = 2;
var a = 23;
```

通常，使用 `var` 来声明变量是一种过时的方法。在 TypeScript 中，有很大动力依赖 `let` 或 `const`，因为你可以生成一个较旧的 ECMAScript 版本，该版本会生成 `var`，但格式正确且有效。

# 使用 `let` 声明

`let` 声明是作用域相关的。在同一个作用域内不能多次声明，并且它不会提升变量。它简化了代码的可读性，并避免了意外的错误。使用 `let` 声明也不会设置任何全局值。当你期望变量被多次设置时，依赖 `let` 是声明变量的方式。在以下代码中，变量 `a` 被定义了三次。代码是合法的，即使有多个声明。原因是每个声明，使用 `let`，都在不同的作用域中定义，使用花括号。第一个作用域是函数作用域。第二个作用域使用了一种不寻常的语法，但它反映了 `while`、`if` 或其他作用域功能的工作方式。第三个作用域在 `if` 语句内：

```js
function letFunction() {
   let a: number = 1;
   { // Scope start
     let a: number = 2;
   } // Scope end
   console.log(a); // 1

   if(true){ // Scope start
     let a: number = 3;
    } // Scope end
    console.log(a); // 1
}
letFunction()
```

此外，TypeScript 确保一旦完成声明，与变量关联的类型是不可变的。这意味着一个定义为数字的变量将保持为数字，直到变量的整个生命周期：

```js
let a:number = 2;
a = "two"; // Doesn’t compile
```

在 `switch` 情况中使用 `let` 可能很棘手。原因是作用域不是由 `case` 决定的，而是由包含所有情况的 `switch` 决定的。然而，通过在每个 `case` 中召唤一个花括号，可以构想一个作用域。以下代码是有效的，即使声明了两个变量 `b`：

```js
function switchFunction(num: number) {
   let b: string = "functionb";

   switch (num) {
       case 1:
         let b: string = "case 1";
       break;
   }
}
```

然而，添加一个随后也声明变量 `b` 的情况会导致编译失败：

```js
function switchFunction(num: number) {
 let b: string = "functionb";

  switch (num) {
    case 1:
     let b: string = "case 1";
    break;
    case 2:
     let b: string = "case 2";
    break;
  }
}
```

从 `switch` 的默认作用域中找到的解决方案是为每个情况创建一个人工作用域。作用域的构建可以通过添加花括号来完成，如下面的代码所示：

```js
function switchFunction(num: number) {
  let b: string = "functionb";

  switch (num) {
    case 1: {
      let b: string = "case 1";
    break;
    } // After break
    case 2: {
      let b: string = "case 2";
    } // Before break
    break;
  }

}
```

`let` 是最常用的声明之一，并且当 `const` 不是一个有效选项时，应该始终使用 `let` 而不是 `var`。

# const

如果你知道变量只设置一次且不会改变，那么使用`const`是一个更好的选择。原因是它向代码的读者表明该值不能被设置超过一次——它是声明和初始化的。TypeScript 尊重`let`和`const`，如果变量被多次定义或当变量是常量时值被两次赋值，代码将无法编译：

将变量约束为保持单一值可能看起来很限制，但在许多情况下，这是正确的事情。使用`const`声明的原始类型阻止了使用等号（`=`）进行赋值，这意味着它不允许你更改变量的引用。然而，你可以更改变量的内容。例如，原始类型的数组可以添加和从数组中删除值，但不能分配新的值列表：

```js
const arr: number[] = [1, 2, 3];
arr.push(4);
```

以下代码显示即使对象被声明为常量，成员也可以被编辑。然而，`myObj`是不可赋值的。这意味着引用将始终保持不变：

```js
const myObj: { x: number } = { x: 1 };
myObj.x = 2;
```

最后，TypeScript 通过使用`let`和`const`确保分配给变量的值与期望的变量相关联，任何错误的赋值都会导致编译器返回错误。在下面的代码中，两个变量在全局作用域和函数作用域中都被明确定义，毫无疑问，它们是两个具有任何值冲突的独立变量：

```js
const a = 2;
function z() {
   let a = 3;
}
```

# 使用 TypeScript 增强原始类型

TypeScript 具有与 JavaScript 相同的原始变量。可以声明一个变量来存储数字、字符串、布尔值和符号。此外，还有两个原始类型可用于没有值的情况：`undefined`和`null`。最后，使用这些原始类型，可以有一个包含它们的数组。

所有原始类型都必须使用之前讨论过的唯一变量名进行声明，并使用冒号后跟单词`number`。然而，当用作函数的参数时，应避免使用`declaration`关键字。没有必要指定变量的作用域，因为这个变量是为函数准备的。同样，可见性也仅限于接收参数的函数： 

```js
function noNeedConstLetvar(parameter1: number) { }
```

# Number

TypeScript 遵循 JavaScript 如何通过单一类型：number 来操作和携带数字的原始类型。一个数字可以是`整数`、`浮点数`、`双精度浮点数`、负数、正数甚至是`NaN`。

数字不能直接使用`布尔`值（无论是`true`还是`false`）。需要通过解析`布尔`值来进行转换：

```js
let boolean: number = true; // Won't compile
```

将布尔值转换为数字的方法有很多。你可以使用`Number`构造函数，它接受任何值并将其转换为`true`为`1`和`false`为`0`的数字：

```js
 let boolean1: number = Number(true);
 let boolean2: number = Number(false);
```

你可以使用三元运算符并手动选择所需的值，这些值可以超出`1`和`0`：

```js
let boolean3: number = true ? 1 : 0;
let boolean4: number = false ? 1 : 0;
```

您可以使用 `+` 符号来开始对数值的添加，这会自动将 `boolean` 值转换为数字：

```js
  let boolean5: number = +true;
  let boolean6: number = +false;
```

数字也不能直接使用字符串。许多从 JavaScript 借用的技术可用。第一种是使用 `Number`，类似于 `boolean` 的情况，会将字符串解析为数字：

```js
  let string1: number = Number("123.5");
  let string2: number = Number("-123.5");
```

第二种方法是使用 `parseInt` 函数。`parse` 函数有一个可选的第二个参数，允许指定基数。需要注意的是，这应该始终指定，以避免与八进制或十六进制发生错误：

```js
  let string3: number = parseInt("123.5", 10);
  let string4: number = parseInt("-123.5", 10);
```

您可以使用 `+` 符号来向值添加，这会自动将字符串值转换为数字：

```js
  let string5: number = +"123.5";
  let string6: number = +"-123.5";
```

如果字符串是用 **数字分隔符** 编写的，将字符串转换为数字可能会很棘手。数字分隔符允许通过在数字之间放置下划线以人类方式编写数字，从而提高可读性。例如，这里是九千一百万：

```js
let numeric_separator: number = 9_000_100;
```

使用数字分隔符解析字符串会失败，但这也适用于使用 `Number` 方法以及 `+` 符号方法的情况。结果是矛盾的，可以是 `NaN` 或仅解析第一个下划线之前的值。在这种情况下，替换所有下划线并使用之前提到的一种技术将是解决方案。

数字可以用不同的基数表示。与 JavaScript 一样，TypeScript 使用 `0x` 字面量表示十六进制，`0b` 表示二进制，`0o` 表示八进制：

```js
  let number1: number = 0x10;
  let number2: number = 0b10;
  let number3: number = 0o10;
```

# 字符串

TypeScript 对于字符串与 JavaScript 相同。您可以使用单引号、双引号或反引号定义字符串。单引号和双引号具有将引号之间的字符串分配给变量的相同功能：

```js
  let string1: string = 's1';
  let string2: string = "s2";
  let string3: string = `s3`;
```

反引号，或称为反引号，有一个特殊的名称，**字符串插值**，它允许在字符串中注入代码。这是通过使用带美元符号和大括号的特殊语法来实现的：

```js
let interpolation1: string = `This contains the variable s1: ${string1} as well as ${string2}`;
let interpolation2: string = `Can invoke variable function: ${string1.substr(0, 1)} as well as any code like this addition: ${1 + 1}`;

console.log(interpolation2);
```

最后一个例子产生以下输出：可以调用变量函数：s，以及任何类似这样的代码：2。

插值不仅限于注入其他值，还可以运行任何 TypeScript 代码。前面的例子在字符串中执行了加法。字符串插值的另一个特性是，您可以在不出现任何编译问题时添加换行符。使用单引号和双引号，字符串必须在同一行上，或者可以分成几个字符串，并用 `+` 符号连接：

```js
  let multipleLine1: string = "Line1" +
      "Line2";

  let multipleLine2: string = `Line1
      Line2`;
```

# 布尔

`boolean` 类型只允许小写的 `true` 和 `false`。不允许使用数字，也不允许值的不同的首字母大小写。可以通过将其与 `1` 进行比较来转换数值，即 `1` 或 `0`：

```js
  let bool1: boolean = true; // true
  let bool2: boolean = false; // false
  let bool3: boolean = 1 === 1; // true
```

还可以使用 JavaScript 的 `Boolean` 构造函数进行转换。TypeScript 不会移除解析过程中出现的怪癖，但返回构造函数的强类型 `bool` 值。以下是一些几乎对具有 `false` 值的字符串情况不起作用的例子：

```js
let bool4: boolean = Boolean("true"); // true
let bool5: boolean = Boolean("TRUE"); // true
let bool6: boolean = Boolean("false"); // true
let bool7: boolean = Boolean("FALSE"); // true
let bool8: boolean = Boolean(NaN); // false

let bool9: boolean = new Boolean("true").valueOf(); // true
let bool10: boolean = new Boolean("false").valueOf(); // true

let bool11: boolean = "true" as any as boolean; // true
let bool12: boolean = "false" as any as boolean; // false
```

只有最后两行是 TypeScript 特有的解决方案，其中我们将字符串转换为类型，然后再转换回`boolean`。这也是除了直接比较字符串之外，唯一可行的解决方案之一，如下面的代码所示：

```js
let bool13 = isTrue("true"); // true
let bool14 = isTrue("false"); // false

function isTrue(s: string): boolean {
    return s.toLocaleLowerCase() === "true";
}
```

最佳解决方案是避免进行任何类型的转换。在大多数情况下，转换会打开潜在意外错误的大门，即使在这个特定的例子中，转换的值是受控的，但在实际场景中可能并非如此。使用`boolean`构造函数很有吸引力，但必须谨慎使用，因为值`false`将导致`boolean`值为`true`。如果值不受控制且是字符串的一部分，最安全的方式是使用本书本节提供的`isTrue`函数进行比较。

# 空值

当主要值不可用时，可以将`Null`值分配给变量。如果没有`strictNullChecks`编译器选项，TypeScript 允许有`null`或`undefined`。作为一个最佳实践，始终设置严格的空值检查并手动指定哪些变量可以同时具有这两个值会更好。原因是你可以仔细构建每个变量和类型，围绕特定类型、`null`或`undefined`的灵活性进行适当的调整，而不留下一个敞开的大门。每次一个变量可以是`null`时，在使用对象属性之前都需要进行空值检查：

```js
  let n1: string | null = Math.random() > 0.5 ? null : "test";
  // console.log(n1.substring(0, 1)); // Won't compile since can be null

  if (n1 !== null) {
      console.log(n1.substring(0, 1));
  }
```

在 TypeScript 中，应该限制`null`的使用，而倾向于使用`undefined`。原因将在*未定义*部分解释。

当`strictNullChecks`被激活时，`null`值只能分配给允许`null`或任何类型的值。要有一个接受`null`的类型，必须使用联合类型：

```js
let primitiveWithNull: number | null = null;
```

# 未定义

当主要值不可用时，`undefined`可以被分配给变量，类似于`null`：

```js
  let n2: string | undefined = Math.random() > 0.5 ? undefined : "test";
  // console.log(n2.substring(0, 1)); // Won't compile since can be null

  if (n2 !== null) {
      console.log(n2.substring(0, 1));
  }
```

然而，还有更多的情况。例如，当一个可选参数（我们将在本书后面讨论）在 TypeScript 中没有提供时，它会被自动设置为`undefined`。原因是当 JavaScript 中的属性不存在时，它是`undefined`，而不是`null`：

```js
function f1(optional?: string): void {
    if (optional === undefined) {
        // Optional parameter was not provided OR was set to undefined
    } else {
        // The optional parameter is for sure a string (not undefined)
    }
}
```

如前所述，如果你使用方括号通过字符串访问一个不存在的对象属性，返回的值也是`undefined`：

```js
  let obj = { test: 1 };
  console.log(obj["notInObject"]);
```

当类尚未从构造函数设置初始值时，`undefined`会被分配给类的字段变量。这只有在编译器选项`strictPropertyInitialization`设置为`false`时才会发生，这是一种不好的做法。为了避免因为初始化不足而导致字段未明确提及`undefined`而默认为`undefined`，编译器选项应始终设置为`true`。

当 `strictNullChecks` 被激活时，未定义的值只能分配给允许 `undefined` 的类型、可选类型或 `any` 类型的值。要使一个类型接受 `undefined`，必须使用原始类型和 `undefined` 的联合。对于可选类型，函数可以在指定原始类型的冒号前使用问号：

```js
let primitiveWithUndefined: number | undefined = undefined;

function functOptionalArg(primitiveOptional?: number): void {
    // ...
}

functOptionalArg();
functOptionalArg(undefined);
functOptionalArg(1);
```

`undefined` 也可以在类或接口中使用可选标记，如下所示：

```js
interface InterfaceWithUndefined {
    m1?: number;
}
```

与 `undefined` 在联合类型中的类型一样，可选值可以通过与 `undefined` 进行比较来验证：

```js
let i1: InterfaceWithUndefined = {};
let i2: InterfaceWithUndefined = { m1: undefined };

console.log(i1.m1 === undefined); // True
console.log(i2.m1 === undefined); // True
```

# Symbol

`Symbol` 允许创建一个唯一的值。`Symbol` 与常量不同，因为具有相同值的两个常量是相等的，而具有相同值的两个符号则不是。常量变量像任何变量一样工作，通过比较值。与 `Symbol` 的比较则不同。每个 `Symbol` 都是唯一的，因此即使值相同，它们也不是相同的。让我们看看一些例子：

```js
const s2 = Symbol("s");
const c1 = "s";
const c2 = "s";

if(isSymbolEqualS(s2)){
    console.log("Symbols are equal"); // Won’t print
}

if(c1 === c2){
    console.log("Constants are equal");
}

function isSymbolEqualS(p1:Symbol): boolean{
    return Symbol("s") === p1;
}

```

使用 `Symbol` 可以确保提供的值确实是期望的那个。它不能传递具有相同值的另一个常量，也不能传递具有相同值的字符串。必须使用完全相同的符号：

```js
let s100 = Symbol("same");
let s101 = Symbol("same");

if (s100 === s101) {
 console.log("Same"); // Won't print
}
```

最后，`Symbol` 可以在定义对象字段时用作保险。有了这个符号，你可以确信每个字段只定义了一次。`Symbol` 本质上是不可变的：

```js
const field1 = Symbol("field");
const obj = {
    [field1]: "field1 value"
};

console.log(obj[field1]); // Print "field1 value"
```

TypeScript 需要知道在 ES2015 中引入的 `Symbol` 功能。在使用 `Symbol` 关键字之前，`tsconfig.json` 必须将 `lib` 添加到数组中：

```js
"lib": [
 "es2015",
 "es2015.symbol"
 ]
```

# 非原始类型

除了原始类型之外，还有更多更高级的变量类型。非原始类型组包括 `void`、`string literal`、`tuple`、`any`、`unknown` 和 `never`；我们现在将讨论这些变量。

# 什么是 `void`？

`void` 是一个特殊类型，主要用于返回没有值的函数。当明确返回 `void` 时，函数不能接受一个带有值的返回语句；因此，它充当了返回值的潜在错误的保护。`void` 函数仍然可以有空的返回，以便在到达闭合花括号之前离开函数。`void` 变量只能分配给 `undefined`：

```js
let a: void = undefined;
console.log(a);
```

这可能没有太大用处，但它解释了如果你向 `void` 函数返回一个没有值的函数会发生什么：

```js
function returnNothing():void{
      return;
}
console.log(returnNothing()); // undefined
```

总是标记一个函数为`void`而不是使用隐式返回值是一个好习惯。函数的隐式返回类型是一个*弱* *void*，因为函数允许返回任何东西。以下函数没有返回类型，最初什么也不返回。然而，在其生命周期中，函数发生了变化（你将在下面看到），现在返回三个不同的值，这些值与之前的值不同。*隐式返回*的值不再是 void。有一个显式返回类型定义了一个合同，并指示任何接触该函数的人预期的返回类型是什么，应该遵守。在这个示例代码中，函数返回一个布尔值、数字和字符串的联合类型：

```js
function returnWithoutType(i: number) {
    if (i === 0) {
        return false;
    } else if (i < 0) {
        return -1;
    } else {
        return "positive";
    }
}
```

# 避免使用任何类型的理由

`any`是一个通配符类型，它不仅允许`any`类型，还可以随意更改类型。`any`有很多问题。第一个问题是很难跟踪变量的类型；我们又回到了 JavaScript 的编写方式：

```js
let changeMe: any;
changeMe = 1;
changeMe = "string too";
changeMe = false;
```

应该避免使用`any`，因为它可以持有不符合预期的值，并且仍然可以编译，因为 TypeScript 不知道类型并且无法执行验证：

```js
let anyDangerous: any = false; // still not a boolean, neither a string
console.log(changeMe.subString(0, 1)); // Compile, but crash at runtime
```

使用`any`的唯一原因是在两种情况下。第一种情况是你正在将代码从 JavaScript 迁移到 TypeScript。迁移代码可能需要很长时间，TypeScript 自然地以这种方式构建，你可以保持混合模式一段时间。这意味着你不仅可以降低编译器选项的严格性，还可以通过允许`any`来创建在类型上不完全详细的函数、变量和类型。

`any`可能被接受的第二种情况是你处于一个无法在某些高级场景中确定类型的情境，你必须继续前进。后者应该是一个路标，必须有一个后续行动，因为我们不希望让它成为一种习惯。

# `never`类型的用法

`never`是一个不应该被设置的变量。一开始这可能听起来没有用，但在你想要确保没有任何东西落入特定的代码路径时，它可能是有用的。一个函数很少返回`never`，但这种情况是可能发生的。如果你有一个不允许你完成方法执行或返回任何变量的函数；因此，它永远不会完全返回。这可以通过异常来编码：

```js
function returnNever(i: number): never {

  // Logic here

  if (i === 0) {
      throw Error("i is zero");
  } else {
      throw Error("i is not zero");
  }

  // Will never reach the end of the function

}
```

当你在编写代码时，你编写了一个不可能发生的条件，TypeScript 通过你的代码使用推断出类型。这可能发生在你有几个条件，其中一个包含另一个，使得某些变量落入`never`场景。如果所有变量值都被条件覆盖，并且有一个`else`语句（或带有`switch case`的默认值），也可能发生这种情况。值不能有任何其他值，只能是*从未被分配*的，因为所有值都被检查了。以下是一个说明这种可能性的示例：

```js
type myUnion = "a" | "b";

function elseNever(value: myUnion) {
    if (value === "a") {
        value; // type is “a”
    } else if (value === "b") {
        value; // type is “b”
    } else {
        value; // type is never
    }
}
```

在实践中，`never`类型用于检查枚举或联合的所有值是否都处理了所有值。这允许在开发者在枚举或联合中添加值时忘记添加条件时创建验证。缺少条件会导致代码在详尽检查中失败。TypeScript 足够智能，可以验证所有情况，并理解代码可能会进入接受`never`参数的函数，这是不允许的，因为不能将任何内容赋值给`never`：

```js
type myUnion = "a" | "b";
let c: myUnion = Math.random() > 0.5 ? "a" : "b";

if (c == "a") {
    console.log("Union a");
} else {
    exhaustiveCheck(c); //”b” will fallthrough
}

function exhaustiveCheck(x: never): never {
    throw new Error("");
}
```

# 将`unknown`类型与更严格的`any`类型结合

TypeScript 的`unknown`类型是新增的功能，旨在减少`any`的使用。当一个变量是`unknown`类型时，可以将任何内容赋值给该变量。然而，`unknown`值只能赋值给另一个`unknown`类型或`any`类型。以下是关于`unknown`类型的几个规则：

+   `unknown`类型只能与等号运算符一起使用；其他运算符将无法编译。

+   返回`unknown`类型的函数不需要返回任何内容。

+   与`unknown`类型的交集是没有用的，因为与`unknown`类型相交的类型将占主导地位。然而，当在联合中使用时，它将始终具有优先级并覆盖联合中的任何其他类型。

+   `unknown`类型的键总是`never`。

`unknown`类型的一个用例是，当不知道类型时可以使用它。而不是依赖于可以接受任何内容并通过任何代码传递的`any`，使用`unknown`限制了变量的流动。由于未知变量不能被设置到另一个变量中，这迫使开发者正确地将值缩小到其类型，并继续进一步。如果没有`unknown`类型，`any`将是唯一的选择。它打开了广泛接受任何内容的大门，并：

```js
function f1(x: any): string {
 return x;
}

function f2(x: unknown): string {
 return x; // Does not compile
}
```

在这个例子中，第一个函数可以接受任何类型并期望返回一个字符串。然而，不需要进行类型转换或任何操作，因为`any`类型的变量可以返回一个字符串。但是，对于`unknown`类型，必须进行处理。正如之前提到的，原因是`unknown`类型不能被赋值给除了`unknown`或`any`之外的任何类型。

# 在列表中强制类型

数组可以通过两种不同的方式创建。第一种是使用方括号和`Array`泛型对象。它们可以互换使用，并且都可以进行类型注解。

方括号格式更紧凑，允许在方括号之前指定类型。`Array`泛型对象在较小/较大的符号之间指定类型：

```js
let arrayWithSquareBrackets: number[] = [1, 2, 3];
let arrayWithObject: Array<number> = [1, 2, 3];
let arrayWithObjectNew: Array<number> = new Array<number>(1, 2 ,3);
```

可以通过将数组类型与联合结合来创建包含多种类型的数组：

```js
let arrayWithSquareBrackets2: (number | string)[] = [1, 2, "one", "two"];
let arrayWithObject2: Array<number | string> = [1, 2, "one", "two"];
```

TypeScript 的行为与 JavaScript 相同，除了指定数组的类型。您可以通过使用索引位置和所有自动使用指定数组类型的方法来访问内容：

```js
const position1 = arrayWithObject2[0]; // 1
const unexisting = arrayWithObject2[100]; // undefined
```

如果一个位置不包含值，返回的类型是`undefined`。

TypeScript 允许循环数组并检索每个位置的强类型元素。类型是可选的，因为 TypeScript 可以推断类型。这意味着以下代码可以带有或不带有`number`类型：

```js
arrayWithSquareBrackets.forEach(function (element: number){
  console.log(element);
});
```

# 使用`enum`定义一组常量

TypeScript 有一个关键字`enum`，允许你指定一组可能的值，其中只能选择一个项。定义一个`enum`可以通过提供潜在的键来完成，这些键将自动从`0`开始分配给`enum`的第一个潜在选择，依此类推：

```js
  enum Weather {
       Sunny,
       Cloudy,
       Rainy,
       Snowy
   }
```

可以指定一个值给键以获得更精细的控制。任何缺失的值将是下一个序列值。在下面的代码示例中，`Sunny`被设置为`100`，而`Cloudy`自动为`101`，`Rainy`为`102`，依此类推：

```js
  enum Weather {
    Sunny = 100,
    Cloudy,
    Rainy,
    Snowy
}
```

在这种情况下，可以跳过，你只能提供一个更大的值，分配的值是顺序的。在下面的代码示例中，值是`100`、`101`、`200`和`201`：

```js
  enum Weather {
    Sunny = 100,
    Cloudy,
    Rainy = 200,
    Snowy
}
```

`enum`也可以支持字符串或字符串和数字的混合：

```js
enum Weather {
    Sunny = "Sun",
    Cloudy = "Cloud",
    Rainy = 200,
    Snowy
}
```

`enum`可以通过`enum`或通过值来访问。通过`enum`访问需要直接从`enum`使用点符号。返回的值是`enum`。这是在 TypeScript 中分配`enum`的常见方式。也可以分配值。当数据来自 JSON 时，值分配很有用。例如，值是从 Ajax 响应中返回的。它将非 TypeScript 与 TypeScript 连接起来：

```js
let today: Weather = Weather.Cloudy;
let tomorrow: Weather = 200;

console.log("Today value", today); // Today value Cloud
console.log("Today key", Weather[today]); // Today key undefined
console.log("Tommorow value", tomorrow); // Tommorow value 200
console.log("Tommorow key", Weather[tomorrow]); // Tommorow key Rainy
```

在之前的代码中，只有当方括号中的值是类型而不是值时，使用方括号访问值才有效。

除了`number`和`string`之外，`enum`在位运算符的帮助下还支持位值。它允许使用和符号（`&`）检查值是否包含单个值或值的聚合。原因是，使用管道`|`可以创建包含多个值的变量。堆叠的值也可以位于`enum`中，用于重用目的，但不是必需的：

```js
  enum Weather {
    Sunny = 0,
    Cloudy = 1 << 0,
    Rainy = 1 << 1,
    Snowy = 1 << 2,
    Stormy = Cloudy | Rainy // Can reside inside
}

let today: Weather= Weather.Snowy | Weather.Cloudy; // Can be outside as well

if (today & Weather.Rainy) { // Check
    console.log("Bring an umbrella");
}
```

一个值可以保存多个值。如果我们想完整地保留现有值，并且需要使用符号`|=`，这很有用。要删除特定的状态，你需要使用`&= ~`。使用这些运算符将在其二进制格式中交换右位置的值，而不会影响数字的其余部分：

```js
today |= Weather.Rainy;
today &= ~Weather.Snowy;
console.log(today); // 3 -> 011 = Cloudy and Rainy
```

最后，为了检查变量是否具有特定的状态，你必须使用三个等号与你想检查的值旁边的和符号。使用单个和符号进行比较是一个错误。和符号返回一个数字，而不是`boolean`。比较需要与我们要检查的值进行比较。通过创建一个组合值在比较中，可以检查多个值：

```js
if (Weather.Rainy === (today & Weather.Rainy)) { // Check
  console.log("Rainy");
}

if (Weather.Cloudy === (today & Weather.Cloudy)) { // Check
  console.log("Cloudy");
}

if ((Weather.Cloudy & Weather.Rainy) === (today & Weather.Cloudy & Weather.Rainy)) { // Check
  console.log("Cloudy and Rainy");
}
```

`enum` 是定义变量可能值的特定域集合的绝佳方式。它具有通过命名选择来清晰表达的优势，并在需要时允许你决定每个条目的值。

# 字符串字面量及其与字符串的区别

`string` 是一种允许任何类型字符的类型。`字符串字面量` 是将特定的 `string` 与类型关联。当 `string` 被设置为类型时，可以分配一个值并在以后更改它。唯一可以设置到 `字符串字面量` 的值是在声明时打印的精确字符串：

```js
let x: string = "Value1";
x = "Value2";

let y: "Literal";
y = "Literal";
y = "sdasd"; // Won't compile
```

TypeScript 将代码编译成纯 JavaScript，不包含类型。一个 `字符串字面量` 确保在 TypeScript 中编写代码时，只能将单个字符串值与变量关联，并且这个值会被编译成具有此强制值的 JavaScript 对象。这种特性的本质在于，我们在两种语言中都有确保值是唯一的保证。这在需要条件化一个在编译后不会存在的类型的情况下非常有用。例如，接口和一段必须根据接口不同而采取不同行动的代码的情况。在接口之间共享一个具有唯一字符串字面量的字段，可以在设计时和运行时进行比较。在设计时，TypeScript 将能够缩小类型范围，从而为指定的类型提供更好的支持，在运行时能够在正确的位置执行流程：

```js
interface Book {
    type: "book";
    isbn: string;
    page: number;
}

interface Movie {
    type: "movie";
    lengthMinutes: number;
}

let hobby: Movie = { type: "movie", lengthMinutes: 120 };

function showHobby(hobby: Book | Movie): void {
    if (hobby.type === "movie") {
        console.log("Long movie of " + hobby.lengthMinutes);
    } else {
        console.log("A book of few pages: " + hobby.page);
    }
}
```

代码示例显示，两个接口共享一个 `字符串字面量` 类型的类型。为了能够在函数中访问一个或另一个 `interface` 的唯一属性，需要进行一个判别器的比较。如果没有比较，接受联合作为参数的函数不知道传递的是两种类型中的哪一种。然而，TypeScript 分析这两个接口，并识别出一个共同字段，允许你在缩小类型之前使用这个字段。一旦 TypeScript 能够找到处理哪种类型，它就允许使用该类型的特定字段。在示例中，在条件内部，所有电影的 `interface` 字段都是可用的。另一方面，`else` 允许仅使用书籍的 `interface` 字段。

字面量字符串是 TypeScript 支持的三种可能的字面量中的一种。TypeScript 在字符串之上还支持 `number` 和 `boolean`。最后，当使用 `字符串字面量` 时，始终使用冒号提供类型：

```js
let myLiteral: "onlyAcceptedValue" = "onlyAcceptedValue";
```

与依赖于 `let`（它打开了多种赋值的大门）不同，使用 `const` 可以确保单个赋值；因此，它将自动为这三种类型推断出 `字面量` 类型：

```js
const myLiteral = "onlyAcceptedValue"; // Not a string
```

有可能通过省略类型来创建一个字面量，但前提是必须使用`const`声明，因为值不能改变；因此，TypeScript 将范围缩小到其最窄的表达式。然而，从`const`到`let`的未来变化会将类型恢复为`string`。我建议尽可能明确，以避免不希望的类型变化。

# 构建一个类型化函数

**函数**在 JavaScript 中是一等公民。自从 ECMAScript 的早期版本以来，函数一直是执行代码和创建作用域的主要概念。TypeScript 以相同的方式使用函数，但提供了额外的类型化功能。

一个函数有一个主要签名，包含该函数的名称、参数列表和返回类型。参数在括号中定义，就像 JavaScript 一样，但每个参数后面都会使用冒号语法跟随其类型：

```js
function funct1(param1: number): string { return ""; }
```

一个`函数`可以拥有多个不同类型的参数：

```js
function funct2(param1: boolean, param2: string): void { }
```

它还可以有一个具有多个类型的参数，使用联合：

```js
function funct3(param1: boolean | string): void { }
```

一个`函数`只有一个返回声明，但该类型可以使用联合来允许类型：

```js
function funct4(): string | number | boolean { return ""; }
```

一个函数可以有一个互补的签名来指示消费者哪些参数匹配在一起以及与返回类型。为同一主体提供多个函数签名是重载函数的概念。当使用重载函数时，所有签名必须从上到下从最具体到最一般地编写。所有定义都需要以分号结束，除了最后一个。最后一个签名总是有一个联合，覆盖了每个位置的所有可能类型。在以下代码示例中，我们指定如果参数是`boolean`，则函数返回一个字符串。如果参数是`Date`，则返回类型是`number`。最后一个签名包含一个第一个参数，即两个可能值（`boolean`和日期）的联合，以及返回类型在`string`和`number`之间的联合）：

```js
function funct5(param1: boolean): string;
function funct5(param1: Date): number;
function funct5(param1: boolean | Date): string | number {
    if (typeof param1 === "boolean") {
        return param1 ? "Yes" : "No";
    } else {
        return 0;
    }
```

```js
}

const expectedString: string = funct5(true); // Yes
const expectedNumber: number = funct5(new Date()); // 0
```

一个函数可以是匿名的。以下是一个使用*胖箭头*格式和一个返回`Function`构造函数的示例：

```js
function returnAnAnonymousFunction(): () => number {
    return () => 1;
}

function returnAnAnonymousFunction2(): Function {
    return function () { return 1 };
}
```

一个`函数`可以是`变量`函数或典型函数。以下是在`变量`中设置的三个函数。`变量`可以通过使用括号和所需的参数来调用。代码示例还展示了两种使用*胖箭头*格式返回数据的方式。如果代码直接返回而不执行任何*多个*语句，则不需要大括号和*返回*语句：

```js
const variable = (message: string) => message + " world";
const variable2 = (message: string) => { return message + " world" };
const variable3 = function (message: string) { return message + " world" };

variable("Hello");
```

一个函数可以有一个可选参数和一个默认值参数。可选参数通过在参数名称后使用问号表示。可选参数允许避免传递值。`TypeScript`自动将参数设置为`undefined`：

```js
function functWithOptional(param1?: boolean): void { }
functWithOptional();
functWithOptional(undefined);
functWithOptional(true);
```

`Optional`与将变量与`undefined`的联合不同，因为联合需要传递值或`undefined`，而可选允许传递值、`undefined`或无：

```js
function functWithUndefined(param1: boolean | undefined): void { }
functWithUndefined(true);
functWithUndefined(undefined);
```

`Optional` 只能在非可选参数之后设置。原因是其他参数是必需的，但如果在 `Optional` 之前或中间，就会很难映射 `哪个` 参数是 `哪个`。以下代码示例展示了由于该规则而导致函数无法编译的情况。然而，可以有多个可选参数：

```js
function functWithOptional2(param1?: boolean, param2: string): void { } // Doesn't compile
function functWithOptional3(param1?: boolean, param2?: string): void { }
```

函数可以位于类中（面向对象的内容将在下一章中介绍）。当这种情况发生时，语法会有所不同。它不使用 `function` 关键字。相反，提供了可见性，即 `public`、`private` 或 `protected`。`TypeScript` 允许避免使用访问修饰符，这将导致函数为 `public`。至于类变量，省略可见性默认使用 `public`：

```js
  class ClassFullOfFunctions {
      public f1() { }
      private f2(p1: number): string { return ""; }
      protected f3(): void { }
      f4(): boolean { return true; }
      f5(): void { } // Public
}
```

参数和强类型返回类型的基仍然相同。也可以创建一个变量，它包含类中的函数，如本章所示。以下是三个将 `private` 函数定义为 `variable` 的示例。第一个示例很长且非常明确。第二个示例在函数级别没有定义类型，因为它已经在声明中定义了。最后一个示例没有定义变量的类型，但由于它从初始化中推断其签名，变量仍然是强类型的：

```js
private long: (p1: number) => string = (p1: number) => { return ""; }
private short: (p1: number) => string = (p1) => "";
private tiny = (p1: number) => "";
```

变量中的 `function` 技术上称为 **函数表达式**，而更传统的 `function` 语法称为 **函数声明**。在 `TypeScript` 中，使用一个或另一个的用法与 `JavaScript` 相同。因为它遵循 JavaScript 的规则，这意味着表达式函数不会被提升。

# 如何在不指定类型的情况下进行强类型化

TypeScript 可以显式指定类型，或者你可以让 TypeScript 确定类型。后者称为 **隐式类型** 或由推断定义的类型。推断的动作由 TypeScript 根据变量在声明时的初始化或函数返回类型返回的内容来执行。

变量的推断仅在声明时赋值的情况下才可能。这意味着在使用 `var`/`let`/`const` 时，你必须设置一个值：

```js
const x = 1;
let y = 1;
let z;
// ...
z = 1;
```

在前面的代码示例中，值 `1` 在初始化时被分配给变量 `x` 和 `y`。即使没有使用冒号，这也是有效的。`TypeScript` 将为这两个变量推断类型。在未指定值的情况下，只有 `var` 或 `let` 可以编译，因为它允许在未来某个时刻进行赋值。值未指定，这意味着类型回退到 `any`。即使在变量的作用域内设置了值，这也是正确的。

在之前的代码示例中，值`1`被分配给了一个常量和变量。这两个声明符的类型不同。常量类型不是一个数字，它是一个`1`的数字字面量。这意味着类型是`1`，仅`1`，而不是任何其他数字。然而，使用`let`声明的变量的类型是`number`。原因是，对于常量，TypeScript 知道它只能初始化一次，并且其值不能改变。它会缩小到它能够找到的最简单类型，即原始值的类型。相反，使用`let`声明的变量可以在变量的生命周期内改变其值。TypeScript 会为每个数字类型进行类型缩小。这对于`number`、`string`和`boolean`都是成立的：

```js
const d1 = new Date();
let d2 = new Date();

const b1 = true;
let b2 = false;

const c1 = {
 m1: 1
};

let c2 = {
 m1: 1
};
```

然而，日期将保持为日期类型，无论声明符如何，任何类或接口也是如此，因为它是较小的分母。在之前的代码示例中，`c1`和`c2`都是必须具有名为`m1`的数字类型成员的对象类型。这个例子说明了 TypeScript 也可以在类型内部推断类型。`m1`是通过推断得到的数字类型。

推断也适用于函数。然而，它有一些限制。第一个限制是参数必须是显式的。原因是，没有潜在的错误空间，你不能通过使用来推断。在下面的代码中，参数`a`是隐式的`any`：

```js
function f1(a) {
   return a;
}
```

然而，返回可以是隐式的。通过返回一个已知类型，可以定义返回类型：

```js
function f2(a: number) {
   return a;
}
```

在返回多个值的情况下，TypeScript 会创建所有潜在类型的联合。在下面的代码示例中，有两个返回语句。TypeScript 会查找每个返回的值，并得出结论，返回了两个不同的值。生成的返回类型是`number | string`：

```js
function f3() {
   if (true) {
       return 1;
   } else {
       return "1";
   }
}
```

# 摘要

在本章中，我们看到了 TypeScript 如何声明变量，以及根据情况选择哪种声明符是最好的。我们看到了 TypeScript 通过在变量的生命周期内强制类型来改进 JavaScript 的原始类型。我们解释了如何通过联合的概念将变量转换为一个多类型容器。TypeScript 将类型引入到函数中，我们看到了如何通过重载函数来提高接受许多参数组合和返回类型的函数的可读性。TypeScript 通过从像 Java 和 C#这样的流行语言中借用的流行`enum`引入了一种新的变量类型。最后，我们简要地了解了 TypeScript 在不同情况下如何智能地推断类型，这可以有利于减少冗长的定义。

在下一章中，我们将详细探讨许多不同对象类型之间的差异，并查看我们如何通过操作类型来拥有足够灵活的强类型代码，以满足我们使用 TypeScript 定义业务模型的需求。
