# 不可变性、光学和惰性

在前面的章节中，我们学习了最基础的函数式编程技术和模式，包括一些最著名的代数数据类型。

在本章中，我们将学习许多额外的函数式编程技术和模式，包括以下内容：

+   不可变性

+   光学

+   镜头

+   Prim

+   惰性评估

再次，我们将从头开始构建一切，尽量避免使用第三方库。我们的目标是查看这些技术和模式的一些内部实现，以便我们完全理解它们是如何工作的。让我们开始吧！

# 不可变性

在本节中，我们将学习关于不可变数据结构的内容。不可变数据结构是一个不允许我们更改其值的对象。在 TypeScript 中实现不可变数据结构的最简单方法就是使用类和 `readonly` 关键字：

```js
class Person {

    public readonly name: string;
    public readonly age: number;

    public constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }

}

const person = new Person("Remo", 29);
person.age = 30; // Error
person.name = "Remo Jansen"; // Error
```

上述代码片段声明了一个名为 `Person` 的类。该类有两个公共属性，分别命名为 `name` 和 `age`。这两个属性已被标记为 `readonly`。正如代码片段所示，当我们尝试更新类的属性值时，会抛出一个编译错误。

`readonly` 属性可以使我们的代码更安全，因为它可以保护我们免受状态突变的影响。例如，如果我们将一些不可变对象作为参数传递给一个函数，该函数将无法修改原始对象。这意味着我们的函数更有可能是一个纯函数。然而，不可变对象并非全是优点。处理不可变对象有时可能会感到非常繁琐和冗长，尤其是当我们希望生成一个新的状态时。让我们来看一个例子：

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

class Company {

    public readonly name: string;
    public readonly addresses: Address[];

    public constructor(name: string, addresses: Address[]) {
        this.name = name;
        this.addresses = addresses;
    }

}
```

上述代码片段声明了三个名为 `Street`、`Address` 和 `Company` 的类。这三个类中的所有属性都是 `readonly`，这意味着这些类是不可变的。我们可以创建一个新的 `Company` 类实例，如下所示：

```js
const company1 = new Company(
   "Facebook",
   [
       new Address(
           "London",
           new Street(1, "rathbone square")
       ),
       new Address(
           "Dublin",
           new Street(5, "grand canal square")
       )
   ]
);
```

当我们说一个对象是不可变的，这意味着我们无法更改原始对象，但这并不意味着我们不希望创建其衍生版本。例如，如果我们尝试通过将街道名称转换为上标来创建一个新的 `Company` 版本，我们将得到一个错误，如下面的代码片段所示：

```js
company1.addresses = company1.addresses.map(a => R.toUpper(a.street.name)); // Error
```

然而，我们可能需要生成一个带有大写街道名称的新版本。我们可以通过创建一个新的 `Company` 实例来生成 `Company` 实例的更新版本。为了创建一个新的副本，我们需要将原始实例的所有属性复制到一个新实例中，并使用新的值来替换我们希望修改的属性：

```js
const company2 = new Company(
   company1.name,
   company1.addresses.map((a) =>
       new Address(
           a.city,
           new Street(
               a.street.num,
               R.toUpper(a.street.name)
           )
       )
   )
);
```

不可变数据结构可以帮助我们实现纯函数，并使我们的代码无副作用。修改外部变量是副作用最常见的原因之一，而使用不可变数据结构可以帮助我们防止这种修改。

请注意，您可以参考 第一章，*函数式编程基础*，以了解更多关于副作用的信息。

然而，正如我们可以在之前的代码片段中看到的那样，不可变数据结构也有其负面影响：它们可能导致冗长且繁琐的代码。好消息是，函数式编程范式的背后人士已经找到了一种解决方案，称为 **光学**。我们将在下一节中学习光学。

# 光学

光学是函数式编程的一个概念，可以帮助我们减少需要编写的代码量，并使操作更易于阅读。使用光学的好处在我们处理不可变数据结构时尤为明显。所有光学都是获取和设置对象属性的一种方式。实际上，我们可以将光学视为面向对象编程中获取器和设置器的替代品。

光学可以分为两大类——**镜头**和**棱镜**。正如我们在 第七章，*范畴论* 中学到的，代数数据类型可以用和类型和积类型来定义。镜头用于处理积类型（例如，元组和对象），棱镜用于处理和类型（例如，区分联合）。在本节的剩余部分，我们将重点关注镜头的使用。

# 镜头

镜头只是一对函数，允许我们在对象中获取和设置值。镜头的接口可以声明如下：

```js
interface Lens<T1, T2> {
    get(o: T1): T2;
    set(o: T2, v: T1): T1;
}
```

正如我们在之前的代码片段中可以看到的，镜头泛型接口声明了两个方法。`get` 方法可以用来获取类型为 `T2` 的对象中属性值。`set` 方法可以用来设置类型为 `T2` 的对象中的属性值。以下代码片段实现了 `Lens` 接口：

```js
const streetLens: Lens<Address, Street> = {
    get: (o: Address) => o.street,
    set: (v: Street, o: Address) => new Address(o.city, v)
};
```

之前实现的 `Lens` 接口被命名为 `streetLens`，它允许我们在 `Object` 类型的对象中设置 `Street` 类型的属性值。我们可以使用 `streetLens` 对象来获取 `Address` 实例中的 `Street` 实例：

```js
const address = new Address(
    "London",
    new Street(1, "rathbone square")
);

const street = streetLens.get(address);
```

我们还可以使用 `Lens` 实现来设置 `Address` 实例中的 `Street` 实例：

```js
const address2 = streetLens.set(
    new Street(
        address.street.num,
        R.toUpper(address.street.name)
    ),
    address
);
```

重要的是要注意，`set` 方法更新 `Street` 实例并返回一个新的 `Address` 实例，而不是修改原始的 `Address` 实例。现在我们知道了镜头如何工作的基础知识，我们将看看一些属性。

镜头的一个主要特征是它们可以被组合。正如我们在前面的章节中学到的，函数组合是函数式编程的主要技术之一，而镜头只是函数，因此它们可以以非常相似的方式组合。以下代码片段声明了一个高阶函数，允许我们组合两个镜头：

```js
function composeLens<A, B, C>(
    ab: Lens<A, B>,
    bc: Lens<B, C>
): Lens<A, C> {
    return {
        get: (a: A) => bc.get(ab.get(a)),
        set: (c: C, a: A) => ab.set(bc.set(c, ab.get(a)), a)
    };
}
```

现在我们已经声明了一个允许我们组合透镜的高阶函数，我们将组合两个名为 `streetLens` 和 `nameLens` 的透镜：

```js
const streetLens: Lens<Address, Street> = {
    get: (o: Address) => o.street,
    set: (v: Street, o: Address) => new Address(o.city, v)
};

const nameLens: Lens<Street, string> = {
    get: (o: Street) => o.name,
    set: (v: string, o: Street) => new Street(o.num, v)
};

const streetNameLens = composeLens(streetLens, nameLens);
```

`composeLens` 函数的返回值创建了一个名为 `streetName` 的新透镜。这个新透镜可以用来获取 `Address` 实例中 `Street` 实例的名称：

```js
const address = new Address(
    "London",
    new Street(1, "rathbone square")
);

const streetName = streetNameLens.get(address);
```

透镜还可以用来创建一个带有更新后的 `Street` 名称的新 `Address` 实例：

```js
const address2 = streetNameLens.set(R.toUpper(address.street.name), address);
```

许多函数式编程库也实现了一个函数，允许我们使用透镜将对象中的给定属性映射到新值。这个函数有时被命名为 `over`，但我们将将其命名为 `overLens` 以便更清晰：

```js
function overLens<S, A>(
    lens: Lens<S, A>,
    func: (a: A) => A,
    s: S
): S {
    return lens.set(func(lens.get(s)), s)
}
```

前面的函数将其第一个参数作为透镜，第二个参数作为映射函数，第三个参数作为对象。该函数使用透镜通过映射函数聚焦和更新对象的某个属性：

```js
const address = new Address(
    "London",
    new Street(1, "rathbone square")
);

const address2 = overLens(streetNameLens, R.toUpper, address);
```

如您所见，使用透镜生成不可变对象的新版本，比使用标准属性访问器和类构造函数要简洁得多。现在我们了解了透镜的基础知识，我们将再次实现一些透镜。之前的实现被简化以方便理解。然而，这次我们将以更接近一些流行库（如 Ramda）的实现方式来实现透镜。

这次，我们将声明两个函数，用作获取器和设置器。用作获取器的函数将实现一个名为 `Prop` 的接口。另一方面，用作设置器的函数将实现一个名为 `Assoc` 的接口。`Prop` 和 `Assoc` 接口的签名如下所示：

```js
type Prop<T, K extends keyof T> = (o: T) => T[K];
type Assoc<T, K extends keyof T> = (v: T[K], o: T) => T;
```

以下代码片段声明了实现 `Prop` 和 `Assoc` 接口的函数。这两个实现都用于访问类型为 `Address` 的对象中的 `street` 属性：

```js
const propStreet: Prop<Address, "street"> = (o: Address) => o.street;

const assocStreet: Assoc<Address, "street"> = (v: Address["street"], o: Address) => {
    return new Address(o.city, v);
};
```

新实现中的一个主要区别是我们将声明一个高阶函数，命名为 `lens`，并使用它来生成透镜实例。`lens` 函数接受两个函数，一个获取器和一个设置器，它们分别实现了 `Prop` 和 `Assoc` 接口。然后，透镜函数返回 `Lens` 接口的实现：

```js
const lens = <T1, K extends keyof T1>(
    getter: Prop<T1, K>,
    setter: Assoc<T1, K>,
): Lens<T1, T1[K]> => {
    return {
        get: (obj: T1) => getter(obj),
        set: (val: T1[K], obj: T1) => setter(val, obj),
    };
}
```

到目前为止，我们可以使用之前声明的获取器函数 `propStreet` 和设置器函数 `assocStreet` 来调用 `lens` 函数：

```js
const streetLens = lens(propStreet, assocStreet);
```

另一个显著的区别是，新的实现使用了两个额外的函数，分别命名为 `view` 和 `set`，作为获取器和设置器。`view` 和 `set` 函数都接受一个透镜实例：

```js
const view = <T1, T2>(lens: Lens<T1, T2>, obj: T1) => lens.get(obj);

const set = <T1, K extends keyof T1>(
    lens: Lens<T1, T1[K]>,
    val: T1[K],
    obj: T1
) => lens.set(val, obj);
```

前面的函数在内部使用透镜的 `get` 和 `set` 方法。然而，我们将使用 `view` 和 `set` 函数。以下代码片段演示了如何使用 `view` 函数获取 `Address` 实例中的 `Street` 实例：

```js
const address = new Address(
    "London",
    new Street(1, "rathbone square")
);

const street = view(streetLens, address);
```

以下代码片段演示了如何使用`set`函数在`Address`实例中设置`Street`实例的值：

```js
const address2 = set(
    streetLens,
    new Street(
        address.street.num,
        R.toUpper(address.street.name)
    ),
    address
);
```

在本节中，我们学习了有关透镜的基础知识。在下一节中，我们将学习另一种功能光学元件，称为**棱镜**。

# 棱镜

棱镜几乎与透镜相同。我们可以将棱镜视为一种透镜，它允许我们在对象中获取和设置可选属性。透镜和棱镜之间最显著的区别是棱镜可以与可选类型一起工作。

以下代码片段声明了`Prism`接口。正如我们所见，`Prism`接口与`Lens`接口非常相似。然而，`get`方法返回一个可选类型`Maybe<T>`：

```js
type Maybe<T> = T | null;

interface Prism<T1, T2> {
    get(o: T1): Maybe<T2>,
    set(a: T2, o: T1): T1;
}
```

就像透镜一样，棱镜也可以组合。以下代码片段声明了一个高阶函数，允许我们组合两个棱镜：

```js
function composePrism<A, B, C>(ab: Prism<A, B>, bc: Prism<B, C>): Prism<A, C> {
    return {
        get: (a: A) => {
            const b = ab.get(a)
            return b == null ? null : bc.get(b)
        },
        set: (c: C, a: A) => {
            const b = ab.get(a)
            return b == null ? a : ab.set(bc.set(c, b), a)
        }
    }
}
```

上述函数接受两个棱镜，`ab`类型为`Prism<A, B>`，和`bc`类型为`Prism<B, C>`，并返回两个棱镜的组合，类型为`Prism<A, C>`。

棱镜还允许我们实现一个函数，该函数可以将对象和棱镜给出的属性进行映射。在现实世界的库中，该函数通常命名为`over`，但就像我们在关于透镜的章节中所做的那样，我们将为了清晰起见将其命名为`overPrism`：

```js
function overPrism<S, A>(
    prism: Prism<S, A>,
    func: (a: A) => A,
    s: S
): S {
    const a = prism.get(s)
    return a ? prism.set(func(a), s) : s
}
```

在前面的代码片段中，我们声明了与棱镜一起工作的主要构建块，包括`Prism`接口和`composePrism`以及`overPrism`函数。在下一节中，我们将演示如何使用名为`firstCharacterPrism`的棱镜来关注可选字符串的第一个字符。代码片段还声明了一个棱镜来访问`Address`实例中的`street`属性和`Street`实例中的`name`属性。

然后，使用`composePrism`将三个`firstCharacterPrism`、`streetPrism`和`namePrism`棱镜组合成一个新的棱镜，命名为`streetNameFirstCharater`。最后，使用`overPrism`函数使用`R.toUpper`函数映射由`streetNameFirstCharacter`选定的值。结果是包含一个具有大写名称的新`Street`实例的新的`Address`实例。如果名称为`null`，新的`Street`实例将包含`null`作为其`name`：

```js
const firstCharacterPrism: Prism<string, string> = {
    get: s => s ? s.substring(0, 1) : null,
    set: (a, s) => s.length ? a + s.substring(1) : ""
}

const streetPrism: Prism<Address, Street> = {
    get: (o: Address) => o.street,
    set: (v: Street, o: Address) => new Address(o.city, v)
};

const namePrism: Prism<Street, string> = {
    get: (o: Street) => o.name,
    set: (v: string, o: Street) => new Street(o.num, v)
};

const address = new Address(
    "London",
    new Street(1, "rathbone square")
);

const streetNameFirstCharacterPrism = composePrism(
    composePrism(streetPrism, namePrism),
    firstCharacterPrism
);

const address2 = overPrism(streetNameFirstCharacterPrism, R.toUpper, address);
```

当我们想要处理其他类型的可选类型时，棱镜也非常有用，例如像`Either`类型这样的区分联合：

```js
type Either<T1, T2> = T1 | T2;

type Domicile = Either<
    { type: "office", address: Address },
    { type: "personal", address: string }
>;

const addressPrism: Prism<Domicile, Address> = {
    get: d => d.type === "office" ? d.address : null,
    set: (address, d) => d.type === "office" ? { type: "office", address } : d
}
```

上述代码片段声明了一个名为`Either`的可选类型和一个名为`Domicile`的类型，它使用`Either`类型来声明两种类型的联合。代码片段还声明了一个名为`addressPrism`的棱镜，允许我们在类型为`Domicile`的对象中获取或设置`address`属性。属性`address`可以是`string`或`Address`实例，`addressPrism`可以处理这两种情况，如下面的代码片段所示：

```js
const address = new Address(
    "London",
    new Street(1, "rathbone square")
);

const domicile1: Domicile = { type: "office", address: address };
const domicile2: Domicile = { type: "personal", address: "23 high street" };

const address1 = addressPrism.get(domicile1);
const address2 = addressPrism.get(domicile2);
```

到目前为止，我们应该理解透镜和棱镜的主要特性。在本章中，我们创建了透镜和棱镜的自己的实现，因为我们的主要目标是理解它们是如何工作的。然而，对于专业的软件开发项目来说，不建议使用自定义实现。在第十章，*现实世界函数式编程*中，我们将学习如何使用 Ramda 的生产就绪透镜。

在下一节中，我们将学习懒加载（lazy evaluation）。

# 懒加载

懒加载是一种技术或模式，它将表达式的评估延迟到其值需要的时候。我们将首先查看一个不使用懒加载的例子，以便我们可以在稍后将其与使用懒加载的例子进行比较。

以下代码片段声明了一个名为 `Dog` 的接口和一个包含十个 `Dog` 实例的数组。`Dog` 实例有两个属性，分别命名为 `size` 和 `name`。代码片段还声明了两个函数，分别命名为 `isLarge` 和 `isOld`。`isLarge` 函数用于查找 `size` 等于 `"L"` 的 `Dog` 实例。`isOld` 函数用于查找年龄大于 `5` 的 `Dog` 实例：

```js
interface Dog {
   size: "L" | "S";
   age: number;
   name: string;
}

const dogs: Dog[] = [
   { size: "S", age: 4, name: "Alice" },
   { size: "L", age: 2, name: "Bob", },
   { size: "S", age: 7, name: "Carol" },
   { size: "L", age: 6, name: "Dan" },
   { size: "L", age: 2, name: "Eve" },
   { size: "S", age: 2, name: "Frank" },
   { size: "S", age: 1, name: "Grant" },
   { size: "S", age: 9, name: "Hans" },
   { size: "L", age: 8, name: "Inga" },
   { size: "L", age: 4, name: "Julia" }
];

const isLarge = (dog: Dog) => dog.size === "L";
const isOld = (dog: Dog) => dog.age > 5;
dogs.filter(isLarge).find(isOld); // Dan
```

上述代码片段使用了数组方法 `filter` 和 `find`。`filter` 方法遍历 `dogs` 数组中的所有元素。使用 `isLarge` 函数过滤所有 `Dog` 实例的结果是一个包含五个元素的新数组（所有 `size` 等于 `"L"` 的元素）。然后我们使用 `find` 方法通过 `isOld` 函数在新数组中搜索 `Dog` 实例。`find` 方法在找到第一个年龄大于 `5` 的元素之前迭代了两个项目。最终结果是，我们需要迭代 12 个项目才能找到一个同时符合 `isLarge` 和 `isOld` 约束条件的项目。

懒加载是一种技术，它将某些操作的执行延迟到它们不能再延迟的时候。懒加载有时可以带来性能提升。

以下代码片段实现了一个名为 `filter` 的函数和一个名为 `find` 的函数。这两个函数分别与数组的 `filter` 和 `find` 方法等价。然而，`filter` 函数使用生成器（`function*`），而 `find` 函数使用 `for ... of` 语句，该语句用于迭代由前面的生成器创建的迭代器返回的项目：

```js
const filter = <T>(f: (item: T) => boolean) => {
   return function* (arr: T[]) {
       for (const item of arr) {
           if (f(item)) {
               yield item;
           }
       }
   };
};

const find = <T>(f: (item: T) => boolean) =>(arr: IterableIterator<T>) => {
   for (const item of arr) {
       if (f(item)) {
           return item;
       }
   }
};
```

请记住，使用迭代器需要在 `tsconfig.json` 文件中将编译设置 `downlevelIteration` 设置为 true。如果您需要关于生成器的额外帮助，请参阅第三章，*精通异步编程*。

代码片段使用了 Ramda 库中的 `compose` 函数来组合 `find(isOld)` 和 `filter(isLarge)` 的返回值。结果是一个名为 `findLargeOldDog` 的新函数。我们可以使用这个函数来查找 `dogs` 数组中同时满足 `isLarge` 和 `isOld` 约束的 `Dog` 实例：

```js
const findLargeOldDog = R.compose(find(isOld), filter(isLarge));
findLargeOldDog(dogs);
```

这个函数的结果与未使用惰性评估的示例结果相同。然而，这个示例只迭代了四个项目，而不是十二个。这是因为当我们执行过滤函数时，过滤不会立即发生。我们通过返回一个迭代器来延迟其评估。评估将延迟到迭代器的 `next` 方法被 `for ... of` 语句调用时。`find` 函数逐个迭代项目，并逐个调用 `isOld` 和 `isLarge` 函数。当迭代器返回第四个项目时，它同时满足 `isLarge` 和 `isOld` 约束，因此不需要再迭代其他项目。因此，惰性评估版本要高效得多。

在前面的示例中，我们使用了生成器和迭代器来实现惰性评估，但这并不是实现惰性评估的唯一方法。惰性评估可以使用多个 JavaScript API 实现，例如代理或承诺。

# 摘要

在本章中，我们学习了如何利用函数式编程技术，如惰性评估和不可变性，来防止一些潜在问题。我们还学习了如何使用光学方法以更简洁、更不繁琐的方式处理不可变对象。

在下一章，我们将学习关于**函数式响应式编程**（**FRP**）的内容。我们将了解响应式编程是什么，以及它与函数式编程之间的联系。
