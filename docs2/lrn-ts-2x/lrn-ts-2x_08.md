# 使用装饰器

在本章中，我们将学习关于注解和装饰器的内容——这两个基于未来 ECMAScript 7 规范的新特性，但我们可以使用 TypeScript 1.5 或更高版本来使用它们。你将学习以下主题：

+   注解与装饰器

+   反射元数据 API

+   装饰器工厂

# 前置条件

本章中的 TypeScript 特性需要 TypeScript 1.5 或更高版本，并在`tsconfig.json`文件中启用以下选项：

```js
"experimentalDecorators": true, 
"emitDecoratorMetadata": true 
```

如实验装饰器编译标志所示，装饰器的 API 被认为是实验性的。这并不意味着它不适合生产使用。这意味着装饰器 API 可能会在未来面临潜在的破坏性更改。

我们还需要一个`reflect–metadata` API 的 polyfill。我们需要 polyfill 是因为大多数 JavaScript 引擎目前还不支持这个 API。我们可以预期，从长远来看，这个 polyfill 将不再需要，但当前，我们可以使用`reflect–metadata` npm 模块：

```js
npm install reflect-metadata
```

`reflect-metadata`的版本在撰写时为 0.1.12。请注意，示例包含在配套源代码中。这些示例可以使用 ts node 执行。例如，配套源代码中的第一个示例可以按以下方式执行：

`ts-node chapters/chapter_08/01_class_decorator.ts`

# 注解与装饰器的比较

注解是一种向类声明添加元数据的方式。然后，库和其他开发工具，如控制反转容器，可以使用这些元数据。注解 API 最初由 Google AtScript 团队提出，但注解不是标准。然而，装饰器是 ECMAScript 规范的一个提议标准，用于在设计时注解和修改类和属性。注解和装饰器基本上是相同的：

“注解和装饰器几乎是同一件事。从消费者角度来看，我们有完全相同的语法。唯一不同的是，我们无法控制如何将注解作为元数据添加到我们的代码中。装饰器更像是一个接口，用于构建最终成为注解的东西。然而，从长远来看，我们只需关注装饰器，因为那些是真正的提议标准。AtScript 是 TypeScript，TypeScript 实现了装饰器”。

– <q>帕斯卡尔·普雷希特</q>，《注解与装饰器的区别》

我们将使用以下类来展示如何使用装饰器：

```js
 class Person {

    public name: string; 
    public surname: string; 

    public constructor(name: string, surname: string) { 
        this.name = name; 
        this.surname = surname; 
    } 

    public saySomething(something: string): string { 
        return `${this.name} ${this.surname} says: ${something}`; 
    } 

} 
```

可以使用四种装饰器来注解：类、属性、方法和参数。

# 类装饰器

TypeScript 官方装饰器提案定义类装饰器如下：

类装饰器函数是一个接受构造函数作为其参数的函数，并返回 undefined、提供的构造函数或一个新的构造函数。返回 undefined 等同于返回提供的构造函数。  – 朗·巴克顿，TypeScript 装饰器提案

类装饰器用于以某种方式修改类的构造函数。如果类装饰器返回`undefined`，则原始构造函数保持不变。如果装饰器返回值，则该返回值将用于覆盖原始类构造函数。以下类型声明了类装饰器的签名：

```js
declare type ClassDecorator = <TFunction extends Function>(target: TFunction) => TFunction | void; 
```

请注意，此签名可能在 TypeScript 的将来版本中发生变化。请参考 TypeScript 源代码中的`lib.d.ts`文件，以找到当前签名：[`github.com/Microsoft/TypeScript/blob/master/lib/lib.d.ts`](https://github.com/Microsoft/TypeScript/blob/master/lib/lib.d.ts)

我们将要创建一个名为`logClass`的类装饰器。我们可以从定义装饰器开始，如下所示：

```js
function logClass(target: any) { 
  // ... 
} 
```

前面的类装饰器还没有任何逻辑，但我们已经可以将其应用于一个类。要应用装饰器，我们需要使用 at（`@`）符号：

```js
@logClass 
class Person { 
  public name: string; 
  public surname: string; 
  //... 
```

如果我们将前面的代码片段编译成 JavaScript，TypeScript 编译器将生成一个名为`__decorate`的函数。我们不会检查`__decorate`函数的内部实现，但我们需要了解它用于在运行时应用装饰器，因为 JavaScript 本身不支持装饰器语法。我们可以通过检查编译前面提到的装饰过的`Person`类时生成的 JavaScript 代码来看到它的实际效果：

```js
var Person = /** @class */ (function () { 
    function Person(name, surname) { 
        this.name = name; 
        this.surname = surname; 
    } 
    Person.prototype.saySomething = function (something) { 
        return this.name + " " + this.surname + " says: " + something; 
    }; 
    Person = __decorate([ 
        logClass 
    ], Person); 
    return Person; 
}()); 
```

如前述代码片段所示，`Person`类被声明，但它随后被传递给`__decorate`函数。`__decorate`函数返回的值被重新分配给`Person`类。现在我们知道了类装饰器是如何被调用的，让我们来实现它：

```js
function logClass<TFunction extends Function>(target: TFunction) { 

    // save a reference to the original constructor 
    const originalConstructor = target; 

    function logClassName(func: TFunction) { 
        console.log("New: " + func.name); 
    } 

    // a utility function to generate instances of a class 
    function instanciate(constructor: any, ...args: any[]) { 
        return new constructor(...args); 
    } 

    // the new constructor behaviour 
    const newConstructor = function(...args: any[]) { 
        logClassName(originalConstructor); 
        return instanciate(originalConstructor, ...args); 
    }; 

    // copy prototype so instanceof operator still works 
    newConstructor.prototype = originalConstructor.prototype; 

    // return new constructor (will override original) 
    return newConstructor as any; 
} 
```

类装饰器接受被装饰的类的构造函数作为其唯一参数。这意味着参数（命名为 `target`）是 `Person` 类的构造函数。装饰器首先创建类构造函数的副本，然后定义一个名为 `instanciate` 的实用函数，该函数可以用于生成类的实例。装饰器用于向装饰元素添加一些额外的逻辑或元数据。当我们尝试扩展函数的功能（方法或构造函数）时，我们需要用一个新的函数包装原始函数，该函数包含额外的逻辑并调用原始函数。在前面的装饰器中，我们添加了额外的逻辑来在控制台中记录创建新实例时的类名。为此，声明了一个新的类构造函数（命名为 `newConstructor`）。新的构造函数调用一个名为 `logClassName` 的函数，该函数实现了额外的逻辑并使用 `instanciate` 函数调用原始类构造函数。在装饰器的末尾，将原始构造函数的原型复制到新的构造函数中，以确保当 `instanceof` 操作符应用于装饰类的实例时，它仍然可以正常工作。最后，返回新的构造函数，并使用它来覆盖原始类构造函数。在装饰类构造函数后，创建了一个新的实例：

```js
const me = new Person("Remo", "Jansen"); 
```

执行此操作后，以下文本将在控制台中显示：

```js
"New: Person"  
```

# 方法装饰器

TypeScript 官方装饰器提案将方法装饰器定义为如下：

"方法装饰器函数是一个接受三个参数的函数：拥有属性的实体、属性的键（一个字符串或符号），以及可选的属性描述符。该函数必须返回 undefined、提供的属性描述符，或者一个新的属性描述符。返回 undefined 等同于返回提供的属性描述符"。

— Ron Buckton，装饰器提案 - TypeScript

方法装饰器类似于类装饰器，但它用于覆盖方法，而不是用于覆盖类的构造函数。以下类型声明了方法装饰器的签名：

```js
declare type MethodDecorator = <T>(target: Object, propertyKey: string | symbol, descriptor: TypedPropertyDescriptor<T>) => TypedPropertyDescriptor<T> | void; 
```

请注意，此签名可能在 TypeScript 的未来版本中发生变化。请参阅 TypeScript 源代码中的 `lib.d.ts` 文件，以找到当前的签名：[`github.com/Microsoft/TypeScript/blob/master/lib/lib.d.ts`](https://github.com/Microsoft/TypeScript/blob/master/lib/lib.d.ts)

方法装饰器接受三个参数：被装饰的类（目标）、被装饰的方法的名称，以及被装饰属性的 `TypePropertyDescriptor`。属性描述符是一个用于描述类属性的对象。属性描述符包含以下属性：

```js
interface TypedPropertyDescriptor<T> { 
    enumerable?: boolean; 
    configurable?: boolean; 
    writable?: boolean; 
    value?: T; 
    get?: () => T; 
    set?: (value: T) => void; 
} 
```

注意，属性描述符是一个可以通过调用 `Object.getOwnPropertyDescriptor()` 方法获得的对象。您可以在 [`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor) 上了解更多信息。

如果方法装饰器返回一个属性描述符，则返回的值将用于覆盖方法的属性描述符。现在让我们声明一个名为 `logMethod` 的方法装饰器，暂时不添加任何行为：

```js
function logMethod(target: any, key: string, descriptor: any) { 
  // ... 
} 
```

我们可以将装饰器应用于 `Person` 类中的某个方法：

```js
class Person { 

    public name: string; 
    public surname: string; 

    public constructor(name: string, surname: string) { 
        this.name = name; 
        this.surname = surname; 
    } 

    @logMethod 
    public saySomething(something: string): string { 
        return `${this.name} ${this.surname} says: ${something}`; 
    } 

} 
```

如果我们将前面的代码片段编译成 JavaScript，我们将能够观察到方法装饰器使用以下参数被调用：

+   包含被装饰方法的类的原型（`Person.prototype`）

+   被装饰的方法的名称（`saySomething`）

+   被装饰方法的属性描述符是 `Object.getOwnPropertyDescriptor(Person.prototype, saySomething)`

现在我们知道了装饰器参数的值，我们可以继续实现它：

```js
function logMethod(
    target: any, 
    key: string, 
    descriptor: TypedPropertyDescriptor<any> 
) { 

    // save a reference to the original method 
    const originalMethod = descriptor.value; 

    function logFunctionCall(method: string, args: string, result: string) { 
        console.log(`Call: ${method}(${args}) => ${result}`); 
    } 

    // editing the descriptor/value parameter 
    descriptor.value = function(this: any, ...args: any[]) { 

        // convert method arguments to string 
        const argsStr = args.map((a: any) => { 
            return JSON.stringify(a); 
        }).join(); 

        // invoke method and get its return value 
        const result = originalMethod.apply(this, args); 

        // convert result to string 
        const resultStr = JSON.stringify(result); 

        // display in console the function call details 
        console.log(); 
        console.log(`Call: ${key}(${argsStr}) => ${resultStr}`); 

        // return the result of invoking the method 
        return result; 
    }; 

    // return edited descriptor 
    return descriptor; 
}
```

就像我们在实现类装饰器时做的那样，我们首先创建被装饰元素的副本。而不是通过类原型（`target[key]`）访问方法，我们将通过属性描述符（`descriptor.value`）来访问它。然后我们创建一个新的函数来替换被装饰的方法。新函数调用原始方法，同时包含一些额外的逻辑，用于在每次调用时在控制台记录方法名称和其参数的值。在将装饰器应用于方法后，当方法被调用时，方法名称和参数将在控制台中被记录：

```js
const person = new Person("Michael", "Jackson"); 
person.saySomething("Annie, are you ok?"); 
```

这样做后，以下文本将出现在控制台：

```js
Call: saySomething("Annie, are you ok?") => "Michael Jackson says: Annie, are you ok?"
```

# 属性装饰器

TypeScript 官方装饰器提案定义方法属性如下：

属性装饰器函数是一个接受两个参数的函数：拥有属性的对象和属性的键（一个字符串或一个符号）。属性装饰器不返回。

—Ron Buckton, TypeScript 装饰器提案

以下类型声明了属性装饰器的签名：

```js
declare type PropertyDecorator = (target: Object, propertyKey: string | symbol) => void; 
```

请注意，此签名可能在 TypeScript 的未来版本中发生变化。请参阅 TypeScript 源代码中的 `lib.d.ts` 文件以获取当前签名：[`github.com/Microsoft/TypeScript/blob/master/lib/lib.d.ts`](https://github.com/Microsoft/TypeScript/blob/master/lib/lib.d.ts)

属性装饰器实际上就像方法装饰器。主要区别是属性装饰器不返回值，并且第三个参数（属性描述符缺失）不会传递给属性装饰器。让我们创建一个名为 `logProperty` 的属性装饰器来查看它是如何工作的：

```js
function logProperty(target: any, key: string) {
  // ... 
} 
```

我们可以在 `Person` 类的属性中使用它，如下所示：

```js
class Person { 

    @logProperty 
    public name: string; 

    @logProperty 
    public surname: string; 

    public constructor(name: string, surname: string) { 
        this.name = name; 
        this.surname = surname; 
    } 

    public saySomething(something: string): string { 
        return `${this.name} ${this.surname} says: ${something}`; 
    } 

} 
```

正如我们迄今为止所做的那样，我们将实现一个装饰器，该装饰器将用一个新的属性覆盖被装饰的属性，这个新属性将表现得与原始属性完全一样，但会执行一个额外的任务——每当属性值发生变化时，在控制台中记录属性值：

```js
function logProperty(target: any, key: string) { 

    // property value 
    let _val = target[key]; 

    function logPropertyAccess(acces: "Set" | "Get", k: string, v: any) { 
        console.log(`${acces}: ${k} => ${v}`); 
    } 

    // property getter 
    const getter = function() { 
        logPropertyAccess("Get", key, _val); 
        return _val; 
    }; 

    // property setter 
    const setter = function(newVal: any) { 
        logPropertyAccess("Set", key, newVal); 
        _val = newVal; 
    }; 

    // Delete property. The delete operator throws 
    // in strict mode if the property is an own 
    // non-configurable property and returns 
    // false in non-strict mode. 
    if (delete target[key]) { 
        Object.defineProperty(target, key, { 
            get: getter, 
            set: setter, 
            enumerable: true, 
            configurable: true 
        }); 
}
}
```

在前面的装饰器中，我们创建了一个原始属性值的副本，并声明了两个函数：`getter`（当改变属性的值时被调用）和`setter`（当读取属性的值时被调用）分别。方法装饰器返回一个值，用于覆盖被装饰的元素。因为属性装饰器不返回值，所以我们不能覆盖被装饰的属性，但我们可以替换它。我们手动删除了原始属性（使用`delete`关键字），并使用`Object.defineProperty`函数以及之前声明的 getter 和 setter 函数创建了一个新的属性。在将装饰器应用于`name`属性后，我们将在控制台中观察到其值的变化：

```js
const person = new Person("Michael", "Jackson"); 
// Set: name => Michael 
// Set: surname => Jackson 

person.saySomething("Annie, are you ok?"); 
// Get: name => Michael 
// Get: surname => Jackson 
```

# 参数装饰器

官方的装饰器提案定义参数装饰器如下：

"参数装饰器函数是一个接受三个参数的函数：拥有包含装饰参数的方法的对象、属性键（对于构造函数的参数为`undefined`）、以及参数的序号索引。这个装饰器的返回值被忽略"。

- Ron Buckton，装饰器提案 - TypeScript

以下类型声明了参数装饰器的签名：

```js
declare type ParameterDecorator = (target: Object, propertyKey: string | symbol, parameterIndex: number) => void; 
```

请注意，这个签名在 TypeScript 的未来版本中可能会发生变化。请参考 TypeScript 源代码中的`lib.d.ts`文件以找到当前的签名：[`github.com/Microsoft/TypeScript/blob/master/lib/lib.d.ts`](https://github.com/Microsoft/TypeScript/blob/master/lib/lib.d.ts)。

前面的装饰器和参数装饰器之间的主要区别是我们不能使用参数装饰器来扩展给定类的功能。让我们创建一个名为`addMetadata`的参数装饰器来查看它是如何工作的：

```js
function addMetadata(target: any, key: string, index: number) {
  // ... 
} 
```

我们可以将参数装饰器应用于一个参数，如下所示：

```js
@logMethod 
public saySomething(@addMetadata something: string): string { 
    return `${this.name} ${this.surname} says: ${something}`; 
} 
```

参数装饰器不返回值，这意味着我们无法覆盖作为参数传递给装饰器的原始方法。我们可以使用参数装饰器将一些元数据链接到被装饰的类。在下面的实现中，我们将添加一个名为`log_${key}_parameters`的数组作为类属性，其中`key`是包含被装饰参数的方法的名称：

```js
function addMetadata(target: any, key: string, index: number) {
    const metadataKey = `_log_${key}_parameters`; 
    if (Array.isArray(target[metadataKey])) { 
        target[metadataKey].push(index); 
    } else { 
        target[metadataKey] = [index]; 
    } 
} 
```

为了允许装饰多个参数，我们检查新字段是否为数组。如果新字段不是数组，我们创建并初始化新字段为一个包含被装饰参数索引的新数组。如果新字段是数组，则将被装饰参数的索引添加到数组中。参数装饰器本身没有用处；它需要与方法装饰器一起使用，因此参数装饰器添加元数据，而方法装饰器读取它：

```js
@readMetadata 
public saySomething(@addMetadata something: string): string { 
    return `${this.name} ${this.surname} says: ${something}`; 
} 
```

以下方法装饰器的工作方式与本章先前实现的方法装饰器类似，但它将读取参数装饰器添加的元数据，并且在方法被调用时，它不会在控制台显示传递给方法的全部参数，而只会记录那些被装饰的参数：

```js
function readMetadata(target: any, key: string, descriptor: any) {

    const originalMethod = descriptor.value; 

    descriptor.value = function(...args: any[]) { 

        const metadataKey = `_log_${key}_parameters`; 
        const indices = target[metadataKey]; 

        if (Array.isArray(indices)) { 

            for (let i = 0; i < args.length; i++) { 

                if (indices.indexOf(i) !== -1) { 
                    const arg = args[i]; 
                    const argStr = JSON.stringify(arg); 
                    console.log(`${key} arg[${i}]: ${argStr}`); 
                } 
            } 

            return originalMethod.apply(this, args); 

        } 

    }; 

    return descriptor; 
}
```

如果我们应用`saySomething`方法：

```js
const person = new Person("Remo", "Jansen"); 
person.saySomething("hello!"); 
```

`readMetadata`装饰器将通过`addMetadata`装饰器在控制台显示参数的值以及哪些索引被添加到元数据（类属性名为`_log_saySomething_parameters`）中：

```js
saySomething arg[0]: "hello!" 
```

注意，在先前的例子中，我们使用类属性来存储一些元数据。然而，这不是推荐的做法。在本章的后面部分，你将学习如何使用`reflection-metadata` API；这个 API 专门设计用来生成和读取元数据，因此，当我们需要与装饰器和元数据一起工作时，推荐使用它。

# 带参数的装饰器

我们可以使用一种特殊的装饰器工厂来允许开发者配置装饰器的行为。例如，我们可以将一个字符串传递给类装饰器，如下所示：

```js
@logClass("option") 
class Person { 
// ... 
```

要能够向装饰器传递一些参数，我们需要用函数包装装饰器。包装函数接受我们选择的选项，并返回一个装饰器：

```js
function logClass(option: string) { 
    return function(target: any) { 
        // class decorator logic goes here 
        // we have access to the decorator parameters 
        console.log(target, option); 
    }; 
} 
```

这可以应用于本章中学到的所有类型的装饰器。

非常重要的是**避免在内部函数中使用箭头函数**，以防止运行时`this`操作符可能出现的潜在问题。

# 反射元数据 API

我们已经了解到装饰器可以用来修改和扩展类的方法或属性的行为。虽然这是一种深入了解装饰器的好方法，**但不推荐使用装饰器来修改和扩展类的行为**。相反，我们应该尝试使用装饰器向被装饰的类添加元数据。然后，其他工具可以消费这些元数据。

如果 TypeScript 团队实现了名为“装饰器突变”的未来功能，那么避免使用装饰器修改和扩展类行为的建议可能会在未来被推翻。你可以在[`github.com/Microsoft/TypeScript/issues/4881`](https://github.com/Microsoft/TypeScript/issues/4881)了解更多关于装饰器突变提案的状态。

在类中添加元数据的可能性可能看起来并不有用或令人兴奋，但在我看来，这是过去几年 JavaScript 发生的最伟大的事情之一。正如我们已知的那样，TypeScript 仅在设计时使用类型。然而，当运行时没有类型信息时，一些功能，如依赖注入、运行时类型断言、反射和测试期间的自动模拟都是不可能的。运行时缺少类型信息不再是问题，因为我们可以使用装饰器来生成元数据，而这些元数据可以包含所需类型信息。然后可以在运行时处理这些元数据。当 TypeScript 团队开始思考允许开发者生成类型信息元数据的最佳方式时，他们为这个目的保留了几个特殊的装饰器名称。想法是，当使用这些保留的装饰器装饰元素时，编译器会自动将类型信息添加到被装饰的元素中。保留的装饰器如下：

"TypeScript 编译器将尊重特殊的装饰器名称，并将额外的信息流入由这些装饰器注解的装饰器工厂参数中。

@type - 装饰器目标的类型的序列化形式

@returnType - 如果装饰器目标是函数类型，则为其返回类型的序列化形式；否则为 undefined

@parameterTypes - 如果装饰器目标是函数类型，则为其参数的序列化类型列表；否则为 undefined

@name - 装饰器目标的名称  "

– Jonathan Turner 的装饰器头脑风暴

稍后，TypeScript 团队决定使用反射元数据 API（ES7 提议的功能之一）而不是保留装饰器。想法几乎相同，但我们将使用一些保留的元数据键，而不是使用保留的装饰器名称，通过反射元数据 API 来检索元数据。TypeScript 文档定义了三个保留的元数据键：

+   **类型元数据使用元数据键设计：type.**

+   *参数类型元数据使用元数据键设计：paramtypes.*

+   *返回类型元数据使用元数据键设计：returntype.*

- Issue #2577 - TypeScript 官方仓库位于 GitHub.com

我们现在将学习如何使用反射元数据 API。我们需要首先安装`reflect-metadata`npm 模块：

```js
npm install reflect-metadata 
```

我们不需要为`reflect-metadata`npm 模块安装类型定义，因为它包含了类型定义。

我们可以按照以下方式导入`reflect-metadata`npm 模块：

```js
import "reflect-metadata"; 
```

`reflect-metadata`模块**应该在整个应用程序中只导入一次**，因为`Reflect`对象旨在成为全局单例。

如果你尝试在未导入`reflect-metadata`模块的 TypeScript 中使用`reflect-metadata` API，你需要在你的`tsconfig.json`文件中添加以下选项：

```js
"types": [ 
  "reflect-metadata" 
] 
```

然后，我们可以创建一个用于测试的类。我们将获取类属性之一在运行时的类型。我们将使用名为`logType`的`property`装饰器来装饰这个类：

```js
class Demo1 { 
    @logType 
    public attr1: string; 
    public constructor(attr1: string) { 
        this.attr1 = attr1; 
    } 
} 
```

我们需要使用`design:type`作为元数据键来调用`Reflect.getMetadata()`方法。元数据值将返回为一个函数。例如，对于字符串类型，返回`function String(){}`函数。我们可以使用`function.name`属性来获取类型作为字符串：

```js
function logType(target: any, key: string) { 
    const type = Reflect.getMetadata("design:type", target, key); 
    console.log(`${key} type: ${type.name}`); 
} 
```

如果我们编译前面的代码，并在浏览器中运行生成的 JavaScript 代码，我们将在控制台看到`attr1`属性的类型：

```js
    'attr1 type: String'  
```

记住，要运行此示例，必须按照以下方式导入`reflect-metadata`库：

`import "reflect-metadata";`

我们可以类似地应用其他保留的元数据键。让我们创建一个具有许多参数的方法，并使用`design:paramtypes`保留的元数据键来检索参数的类型：

```js
class Foo {} 
interface FooInterface {} 

class Demo2 { 
    @logParamTypes 
    public doSomething( 
        param1: string, 
        param2: number, 
        param3: Foo, 
        param4: { test: string }, 
        param5: FooInterface, 
        param6: Function, 
        param7: (a: number) => void 
    ): number { 
        return 1; 
    } 
}
```

这次，我们将使用`design:paramtypes`保留的元数据键。我们正在查询多个参数的类型，因此`Reflect.getMetadata()`函数将返回一个数组：

```js
function logParamTypes(target: any, key: string) {
    const types = Reflect.getMetadata( 
        "design:paramtypes", 
        target, 
        key 
    ); 
    const s = types.map((a: any) => a.name).join(); 
    console.log(`${key} param types: ${s}`); 
} 
```

如果我们在浏览器中编译并运行前面的代码，我们将在控制台看到参数的类型：

```js
    'doSomething param types: String, Number, Foo, Object, Object, 
 Function, Function'

```

类型序列化遵循一些规则。我们可以看到函数被序列化为`function`，对象字面量（`{test : string}`）和接口被序列化为`object`。下表展示了不同类型的序列化方式：

| **Type** | **Serialized** |
| --- | --- |
| void | Undefined |
| string | String |
| number | Number |
| boolean | Boolean |
| symbol | Symbol |
| any | Object |
| enum | Number |
| Class C{} | C |
| Object literal {} | Object |
| interface | Object |

注意，一些开发者要求通过元数据访问接口的类型和类的继承树。这个特性被称为**复杂类型序列化**，在撰写本文时不可用。

为了总结，我们将创建一个具有返回类型的方法，并使用`design:returntype`保留的元数据键来检索返回类型的类型：

```js
class Demo3 { 
    @logReturntype 
    public doSomething2(): string { 
        return "test"; 
    } 
} 
```

就像在前两个装饰器中一样，我们需要调用`Reflect.getMetadata()`函数，并传递`design:returntype`保留的元数据键：

```js
function logReturntype(target: any, key: string) { 
    const returnType = Reflect.getMetadata( 
        "design:returntype", 
        target, 
        key 
    ); 
    console.log(`${key} return type: ${returnType.name}`); 
} 
```

如果我们在浏览器中编译并运行前面的代码，我们将在控制台看到返回类型的类型：

```js
    'doSomething2 return type: String'

```

# 装饰器工厂

官方的装饰器提案定义装饰器工厂如下：

装饰器工厂是一个可以接受任意数量参数的函数，并且必须返回装饰器函数的类型之一。

- Ron Buckton, TypeScript 装饰器提案

我们已经学会了实现类、属性、方法和参数装饰器。然而，在大多数情况下，我们将消费装饰器，而不是实现它们。例如，在`InversifyJS`中，我们使用`@injectable`装饰器来声明一个类将被注入到其他类中，但我们不需要实现`@injectable`装饰器。我们可以使用装饰器工厂使装饰器更容易消费。让我们考虑以下代码片段：

```js
@logClass 
class Person { 

    @logProperty 
    public name: string; 

    @logProperty 
    public surname: string; 

    public constructor(name: string, surname: string) { 
        this.name = name; 
        this.surname = surname; 
    } 

    @readMetadata 
    public saySomething(@addMetadata something: string): string { 
        return `${this.name} ${this.surname} says: ${something}`; 
    } 
} 
```

前述代码的问题是我们作为开发者需要知道`logMethod`装饰器只能应用于方法。这可能会显得微不足道，因为使用的装饰器名称（`logMethod`）使得它更容易识别。更好的解决方案是允许开发者使用名为`@log`的装饰器，而无需担心使用正确的装饰器类型：

```js
@log 
class Person { 

    @log 
    public name: string; 

    @log 
    public surname: string; 

    public constructor(name: string, surname: string) { 
        this.name = name; 
        this.surname = surname; 
    } 

    @log 
    public saySomething(@log something: string): string { 
        return `${this.name} ${this.surname} says: ${something}`; 
    } 
} 
```

我们可以通过创建一个装饰器工厂来实现这一点。装饰器工厂是一个函数，它可以识别所需的装饰器类型并返回它：

```js
function decoratorFactory( 
    classDecorator: Function, 
    propertyDecorator: Function, 
    methodDecorator: Function, 
    parameterDecorator: Function 
) { 
    return function (this: any, ...args: any[]) { 
        const nonNullableArgs = args.filter(a => a !== undefined); 
        switch (nonNullableArgs.length) { 
            case 1: 
                return classDecorator.apply(this, args); 
            case 2: 
                // break instead of return as property 
                // decorators don't have a return 
                propertyDecorator.apply(this, args); 
                break; 
            case 3: 
                if (typeof args[2] === "number") { 
                    parameterDecorator.apply(this, args); 
                } else { 
                    return methodDecorator.apply(this, args); 
                } 
                break; 
            default: 
                throw new Error("Decorators are not valid here!"); 
        } 
    }; 
} 
```

如前述代码片段所示，装饰器工厂是一个装饰器的工厂。生成的装饰器使用传递给装饰器的参数的数量和类型来识别适合每种情况的所需装饰器类型。装饰器工厂可以用来创建通用装饰器如下：

```js
const log = decoratorFactory( 
    logClass, 
    logProperty, 
    readMetadata, 
    addMetadata 
); 
```

# 摘要

在本章中，我们学习了如何消费和实现四种可用的装饰器类型（类、方法、属性和参数）。我们使用装饰器来修改原始类以理解它们的工作原理，但也了解到我们应该**避免使用装饰器来修改类的原型**。我们还学习了如何创建装饰器工厂，以便在消费时抽象开发者对装饰器类型的关注，如何向装饰器传递配置，以及如何使用反射元数据 API 在运行时访问类型信息。正如我们已经提到的，TypeScript 中的装饰器仍然是一个实验性特性，这并不意味着它们不适合在生产系统中使用，但它们的公共 API 可能在将来可能会发生破坏性变化。请注意，未来的 TypeScript 版本将记录如何应对这些潜在的破坏性变化，如果它们最终发生的话。在下一章中，我们将学习如何配置高级 TypeScript 开发工作流程。
