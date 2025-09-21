# 第十章：实际应用中的函数式编程

在本书的前一章中，我们学习了函数式编程和函数式响应式编程。我们试图避免使用外部库，因为我们的主要目标是理解函数式编程和函数式响应式编程范式的技术、模式和原则。

在本章中，我们将学习以下主题：

+   使用 Ramda 进行组合

+   使用 Ramda 进行柯里化和部分应用

+   使用 Ramda 的透镜

+   与 Immutable.js 一起工作

+   与 Immer 一起工作

+   与 Funfix 一起工作

我们将再次回顾本书中探索的一些主要概念。然而，这一次，我们的重点不是理解这些概念（假设我们已经做到了）。相反，我们将专注于一些生产就绪的函数式编程库的使用，例如 Ramda 或 Immutable.js。

# 与 Ramda 一起工作

Ramda 是一个开源的函数式编程库，它包含许多实用函数，可以帮助我们将一些主要的函数式编程技术付诸实践。Ramda 可以与其他库，如 Lodash 或 Underscore 相比较。然而，Ramda 的 API 受函数式编程原则的影响比这些其他库更大。例如，Ramda 的设计使得可组合性和不可变性成为其组件的两个主要特性。

我们可以使用以下 **npm** 命令安装 Ramda：

```js
npm install ramda @types/ramda
```

在以下章节中，我们将学习如何使用 Ramda 实现函数组合和透镜。

# 组合

在前面的章节中，我们声明了一个名为 `compose` 的高阶函数，它允许我们组合两个函数：

```js
const compose = <T>(f: (x: T) => T, g: (x: T) => T) => (x: T) => f(g(x));
```

`compose` 函数使我们能够展示函数组合是如何工作的，但它有一些限制。例如，`compose` 函数只接受一个通用类型参数 `T`，这意味着我们只能组合两个接受类型 `T` 参数的一元函数 `f` 和 `g`：

```js
const trim = (s: string) => s.trim();
const capitalize = (s: string) => s.toUpperCase();
const trimAndCapitalize = compose(trim, capitalize);
const result = trimAndCapitalize(" hello world ");
console.log(result); // "HELLO WORLD"
```

在实际应用中，我们可能需要组合两个接受不同类型 `T1` 和 `T2` 参数的函数 `f` 和 `g`。以下代码片段使用 Ramda 的 `compose` 函数而不是我们之前声明的函数：

```js
import R from "ramda";

const trim = (s: string) => s.trim();
const capitalize = (s: string) => s.toUpperCase();
const trimAndCapitalize = R.compose(trim, capitalize);
const result = trimAndCapitalize(" hello world ");
console.log(result); // "HELLO WORLD"
```

Ramda 的 `compose` 函数是实际应用中更好的替代方案，因为它已经在数百个项目中使用，并且与数千个函数进行了测试。

# 部分应用和柯里化

我们还了解到，函数组合仅适用于一元函数，如果我们希望组合非一元函数，例如二元函数，我们可以使用函数部分应用来调用其中一个函数，只传递部分参数，并生成新的函数，这些函数接受剩余的参数。最后，我们还了解到我们可以使用柯里化将给定的函数转换成可以部分应用的函数：

```js
function curry3<T1, T2, T3, T4>(fn: (a: T1, b: T2, c: T3) => T4) {
    return (a: T1) => (b: T2) => (c: T3) => fn(a, b, c);
}

const trim = (s: string) => s.trim();
const capitalize = (s: string) => s.toUpperCase();
const trimAndCapitalize = R.compose(trim, capitalize);
const replace = (s: string, f: string, r: string) => s.split(f).join(r);

const curriedReplace = curry3(replace);

const trimCapitalizeAndReplace = compose(
    trimAndCapitalize,
    curriedReplace("/")("-")
);

const result = trimCapitalizeAndReplace(" 13/feb/1989 ");
console.log(result); // "13-FEB-1989"
```

前面的代码片段使用`curry3`函数将三元函数转换为一个可以部分应用的函数。在实际应用中，不建议创建自定义实现，而应尝试使用经过实战检验的库。幸运的是，我们不需要搜索很长时间，因为 Ramda 包括一个名为`curry`的函数，可以用来实现我们想要的功能：

```js
import R from "ramda";

const trim = (s: string) => s.trim();
const capitalize = (s: string) => s.toUpperCase();
const replace = (s: string, f: string, r: string) => s.split(f).join(r);

const trimCapitalizeAndReplace = R.compose(
 R.compose(trim, capitalize),
 R.curry(replace)("/")("-")
);

const result = trimCapitalizeAndReplace(" 13/feb/1989 ");
console.log(result); // "13-FEB-1989"
```

前面的代码片段以完全与 Ramda 无关的方式声明了三个函数。然后我们使用 Ramda 的实用函数`compose`和`curry`来生成一个名为`trimCapitalizeAndReplace`的新函数：

# 透镜

在一些前面的章节中，我们学习了关于透镜的内容。在最后一个例子中，我们以一种非常接近 Ramda 提供的实现方式实现了透镜。

我们实现了一个名为`lens`的高阶函数，可以用来创建`Lens`实现。`lens`函数接受两个必须实现`Prop`和`Assoc`接口的函数：

```js
interface Lens<T1, T2> {
    get(o: T1): T2;
    set(o: T2, v: T1): T1;
}

type Prop<T, K extends keyof T> = (o: T) => T[K];
type Assoc<T, K extends keyof T> = (v: T[K], o: T) => T;

const lens = <T1, K extends keyof T1>(
    getter: Prop<T1, K>,
    setter: Assoc<T1, K>,
): Lens<T1, T1[K]> => {
    return {
        get: (obj: T1) => getter(obj),
        set: (val: T1[K], obj: T1) => setter(val, obj),
    };
}

const view = <T1, T2>(lens: Lens<T1, T2>, obj: T1) => lens.get(obj);

const set = <T1, K extends keyof T1>(
    lens: Lens<T1, T1[K]>,
    val: T1[K],
    obj: T1
) => lens.set(val, obj);
```

我们可以创建一个`Lens`实现，将`Assoc`和`Prop`实现传递给`Lens`函数：

```js
class Street {

    public readonly num: number;
    public readonly name: string;

    public constructor(num: number, name: string) {
        this.num = num;
        this.name = name;
    }

}

class Address {

    public readonly city: string;
    public readonly street: Street;

    public constructor(city: string, street: Street) {
        this.city = city;
        this.street = street;
    }

}

const propStreet: Prop<Address, "street"> = (o: Address) => o.street;

const assocStreet: Assoc<Address, "street"> = (v: Address["street"], o: Address) => {
    return new Address(o.city, v);
};

const streetLens = lens(propStreet, assocStreet);
```

一旦我们有一个`Lens`实例，我们可以使用`set`和`view`函数来`读取`和`设置`不可变对象中属性的值：

```js

const address = new Address(
    "London",
    new Street(1, "rathbone square")
);

const street = view(streetLens, address);

const address2 = set(
    streetLens,
    new Street(
        address.street.num,
        R.toUpper(address.street.name)
    ),
    address
);
```

现在我们已经了解了关于 Ramda 的基础知识，我们可以再次实现前面的代码片段，使用其中的一些实用函数。Ramda 包括以下实用函数，以及其他一些：

+   `prop`函数允许我们声明一个属性获取器。它期望一个属性名称作为参数。

+   `assoc`函数允许我们声明一个属性设置器。它期望一个属性名称作为参数。

+   `lens`函数允许我们声明一个透镜实例。它期望一个属性获取器（`prop`）和一个设置器（`assoc`）作为参数。

+   `lensProp`函数允许我们声明一个透镜实例。它期望一个属性名称作为参数。

+   `view`函数允许我们获取对象中属性的值。它期望一个透镜实例和一个对象作为参数。

+   `set`函数允许我们在对象中设置属性的值。它期望一个透镜实例、一个新值和一个对象作为参数。

在下面的代码片段中，我们使用了`lensProp`、`view`和`set`函数：

```js
import R from "ramda";

class Street {

    public readonly num: number;
    public readonly name: string;

    public constructor(num: number, name: string) {
        this.num = num;
        this.name = name;
    }

}

class Address {

    public readonly city: string;
    public readonly street: Street;

    public constructor(city: string, street: Street) {
        this.city = city;
        this.street = street;
    }

}

const streetLens = R.lensProp("street");

const address = new Address(
    "London",
    new Street(1, "rathbone square")
);

const street = R.view<Address, Street>(streetLens, address);

const address2 = R.set<Address, Street>(
    streetLens,
    new Street(
        address.street.num,
        R.toUpper(address.street.name)
    ),
    address
);
```

由于有`lensProp`而不是`lens`函数，大多数情况下不需要使用`prop`和`assoc`函数。

# 使用 Immutable.js

在前面的章节中，我们还学习了不可变性和使用不可变数据对象的优点。我们了解到我们可以使用`readonly`关键字来声明不可变对象：

```js
class Street {

    public readonly num: number;
    public readonly name: string;

    public constructor(num: number, name: string) {
        this.num = num;
        this.name = name;
    }

}

class Address {

    public readonly city: string;
    public readonly street: Street;

    public constructor(city: string, street: Street) {
        this.city = city;
        this.street = street;
    }

}
```

我们还了解到，与不可变对象一起工作有时可能非常冗长且繁琐，而 lenses 可以帮助我们克服这些困难。现在我们将学习一个可以帮助我们声明不可变对象的库。这个库被称为 **Immutable.js**，它还包括一个类似于 lenses API 的 API。我们可以使用以下命令安装 Immutable.js：

```js
npm install immutable
```

我们可以使用 `Record` 类型定义类型安全的不可变类，如下所示：

```js
import { Record } from "immutable";

interface StreetInterface {
    num: number;
    name: string;
}

const StreetRecord = Record({
    num: 0,
    name: ""
});

class Street extends StreetRecord implements StreetInterface {
    constructor(props: StreetInterface) {
        super(props);
    }
}
```

我们将定义一个名为 `Address` 的更多不可变类。`Address` 类包含 `Street` 类的一个实例：

```js
interface AddressInterface {
    city: string;
    street: Street;
}

const AddressRecord = Record({
    city: "",
    street: new Street({
        num: 0,
        name: ""
    })
});

class Address extends AddressRecord implements AddressInterface {
    constructor(props: AddressInterface) {
        super(props);
    }
}
```

要创建一个不可变类的实例，我们需要将所有必需的属性作为普通对象传递：

```js
const address = new Address({
    city: "Lonson",
    street: new Street({
        num: 1,
        name: "rathbone square"
    })
});
```

当我们使用 Immutable.js 声明一个不可变类时，该类继承了一些类似 lenses 的方法。我们可以使用 `get` 方法获取属性的值，使用 `set` 方法通过更新值创建一个新的不可变实例：

```js
const street = address.get("street");
const street2 = street.set("name", "Rathbone square");
const address2 = address.set("street", street2);

console.log(
    address.toJS(),
    address2.toJS()
);
```

使用 Immutable.js 声明不可变类比使用 `readonly` 访问修饰符声明它们更为繁琐，但作为交换，我们得到了内置的 lenses 功能。

# 使用 Immer

我们将查看另一个流行的不可变性库。这个库被称为 **Immer**，可以使用以下 `npm` 命令安装：

```js
npm install immer
```

Immer 允许我们使用 `readonly` 访问修饰符定义不可变类。这意味着我们也可以使用标准的类构造函数创建我们类的实例：

```js
import produce from "immer";

class Street {

    public readonly num: number;
    public readonly name: string;

    public constructor(num: number, name: string) {
        this.num = num;
        this.name = name;
    }

}

class Address {

    public readonly city: string;
    public readonly street: Street;

    public constructor(city: string, street: Street) {
        this.city = city;
        this.street = street;
    }

}

const address = new Address(
    "London",
    new Street(1, "rathbone square")
);
```

Immer 可以使用名为 `produce` 的方法生成不可变对象的新版本。`produce` 函数将当前不可变对象的版本作为其第一个参数。第二个参数是一个回调函数，它接受一个称为草稿状态的参数。草稿状态是初始版本的可变版本，在回调函数内部，我们可以尽可能多地修改它：

```js
const address2 = produce(address, draftAddress => {
    draftAddress.street.name = "Rathbone square";
});
```

`produce` 函数返回一个新不可变对象，而不修改原始对象。Immer API 可以被认为是优于 Immutable.js API 的，因为它在我们的类声明和构造函数中施加了更少的约束。Immer 不基于 lenses，并允许我们使用一种创新的内部使用代理的方法来与不可变对象一起工作。

# 使用 Funfix

**Funfix** 是一组函数式编程实用函数集合。Funfix 可以与 Ramda 相比。就像 Ramda 一样，Funfix 可以用来组合函数或部分应用函数。然而，在本节中，我们将关注一些与我们在第七章 *范畴论* 中之前探索的一些数据类型相关的 Funfix 功能的使用。

我们将首先安装 Funfix：

```js
npm install funfix @types/funfix
```

我们将在本节中实现的示例将需要一些额外的`npm`模块。我们将使用`node-fetch`从 Node.js 应用程序发送 HTTP 请求。我们还将使用一些 Node.js 核心模块，这意味着我们还需要 Node.js 的类型定义：

```js
npm install node-fetch @type/node-fetch @types/node
```

在我们的第一个 Funfix 示例中，我们将使用`IO.of`工厂方法定义一个名为`argsIO`的单子。正如我们在前面的章节中学到的，单子是一种函子，而函子是一种容器。在这种情况下，容器包含一个执行 I/O 操作（如读取命令行参数`process.argv`）的函数。`IO`类型用于存储描述某些带有副作用的计算（如从文件读取数据或修改**文档对象模型**（**DOM**）中的元素）的函数。以这种方式描述动作允许`IO`实例被组合并传递，同时保持函数的纯度和维护引用透明性。

我们还将声明两个名为`readFile`和`stdoutWrite`的函数。这两个函数都返回一个单子实例，并且这两个单子都包含 I/O 操作。第一个从文件系统中读取文件，第二个在标准输出中打印一些信息：

```js
import * as R from "ramda";
import * as fs from "fs";
import { IO } from "funfix";

const argsIO = IO.of(() => R.tail(R.tail(process.argv))[0]);
const readFile = (filename: string) => IO.of(() => fs.readFileSync(filename, "utf8"));
const stdoutWrite = (data: string) => IO.of(() => process.stdout.write(data));

const loudCat = argsIO.chain(readFile)
     .map(R.toUpper)
     .chain(stdoutWrite);

try {
   loudCat.run();
} catch(e) {
   console.log(e);
}
```

前面的代码片段还使用`chain`方法声明了一个名为`loudCat`的单子，将命令行参数传递给文件读取操作，并使用`map`方法将文件内容转换为大写。最后，它再次使用`chain`方法将大写文本传递到标准输出。

Funfix 中单子的一个主要特征是它们是惰性求值的，并且所有前面的操作都不会发生，直到我们调用`run`方法。如果一切顺利，我们可以通过命令行界面传递文件名：

```js
node example.js test.txt 
```

文件的大写内容应显示在标准输出上。以下示例使用`node-fetch`模块发送 HTTP 请求。执行 HTTP 请求的函数包含在一个单子中。这次，单子不是由`IO.of`工厂函数创建的，而是由`IO.async`工厂函数创建的。我们使用`IO.async`工厂函数是因为 Funfix 在将异步操作包装在单子中时需要它。示例还使用了`Either`类型，它也是一种函子和单子。它可以用来包装可能有两个可能值的值：

```js
import { IO, Success, Failure, Either, Left, Right } from "funfix";
import fetch from "node-fetch";

interface Todo {
    userId: number;
    id: number;
    title: string;
    completed: boolean;
}

const getTodos = IO.async<Either<Error, Todo[]>>((ec, cb) => {
   fetch(
       "https://jsonplaceholder.typicode.com/todos"
   ).then(response => {
       return response.json().then(
           (json: Todo[]) => cb(Success(Right(json)))
       )
   })
   .catch(err => cb(Failure(Left(err))));
});

const logTodos = getTodos.map((either) => {
   return either.map(todos => todos.map(t => console.log(t.title)));
});

logTodos.run();
```

错误处理逻辑非常简单，因为`Either`类型的`map`方法仅在它的值的类型不是错误时映射值。就像之前一样，整个逻辑是惰性求值的，直到我们在`logTodos`单子中调用`run`方法，实际上什么都不会发生。

# 摘要

在本章中，我们学习了如何使用一些实际的函数式编程库，包括 Ramda、Fundix、Immer 和 Immutable.js。贯穿整本书，我们了解了函数式编程和函数式响应式编程范式的核心特性、原则、模式和原则。这些概念为您提供了一套强大的工具，将帮助您使用更容易推理、更易测试和更易维护的应用程序。

我希望您喜欢这本书，并且渴望继续您的函数式编程学习之旅。在附录中，您将找到一份指南，可用于发现新的函数式库以及您可以自行探索的额外函数式编程概念，如果您想了解更多的话。
