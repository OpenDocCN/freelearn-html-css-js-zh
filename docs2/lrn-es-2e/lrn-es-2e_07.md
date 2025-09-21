# 代理

代理用于定义对象基本操作的自定义行为。代理在诸如 C#、C++和 Java 等编程语言中已经可用，但 JavaScript 从未有过代理。ES6 引入了 Proxy API，允许我们创建代理。在本章中，我们将探讨代理、它们的用法和代理陷阱。由于代理的好处，开发者越来越多地使用它们，因此深入了解代理，并通过示例进行学习变得非常重要，我们将在本章中这样做。

在本章中，我们将涵盖：

+   使用 Proxy API 创建代理

+   理解代理是什么以及如何使用它们

+   使用陷阱拦截对象上的各种操作

+   可用的不同陷阱类型

+   代理的一些用例

# 代理概述

代理就像是一个对象的包装器，并定义了对象基本操作的自定义行为。对象的一些基本操作包括属性查找、属性赋值、构造函数调用、枚举等。

想象这是一种拦截你用对象及其相关属性执行的操作的基本方式。例如，通过编写`<objectname>.propertyName`来调用属性值，从技术上讲，应该只是回显属性值，对吧？

如果你想在回声部分之前，但在调用部分之后，退一步并注入你的控制，那么代理就派上用场了。

一旦使用代理包装了一个对象，那么应该在代理对象上执行所有应该在对象上执行的操作，以便执行自定义行为。

# 代理的术语

在学习代理时，以下是一些重要的术语：

+   **目标对象**：这是被代理包装的对象。

+   **陷阱**：这些是拦截目标对象上各种操作并定义这些操作自定义行为的函数。

+   **处理器**：这是一个包含陷阱的对象。处理器附加到代理对象上。

# 与 Proxy API 一起工作

ES6 Proxy API 提供了代理构造函数来创建代理。代理构造函数接受两个参数，它们是：

+   **目标对象**：这是将被代理包装的对象

+   **处理器**：这是一个包含`target`对象陷阱的对象

可以为目标对象上的每个可能操作定义陷阱。如果没有定义陷阱，则默认在目标上执行操作。以下是一个代码示例，展示了如何创建代理，并在目标对象上执行各种操作。在这个例子中，我们没有定义任何陷阱：

```js
const target = {  age: 12 }; 
const handler = {}; 
const proxy = new Proxy(target, handler); 
proxy.name = "Eden"; 
console.log(target.name); 
console.log(proxy.name); 
console.log(target.age); 
console.log(proxy.age);
```

这会输出以下内容：

```js
Eden
Eden
12
12
```

在这里，我们可以看到可以通过`proxy`对象访问`target`对象的`age`属性。当我们向`proxy`对象添加`name`属性时，实际上它是添加到`target`对象上的。

由于没有将陷阱附加到属性赋值上，`proxy.name`赋值导致了默认行为——即简单地赋值给属性。

因此，我们可以这样说，代理只是一个`target`对象的包装器，并且可以定义陷阱来改变操作的默认行为。

许多开发者没有保留`target`对象的引用变量，因此访问对象时使用代理不是强制性的。只有在你需要为多个代理重用处理器时，才保留处理器的引用。以下是重写之前代码的方法：

```js
var proxy = new Proxy({ age: 12 }, {}); 
proxy.name = "Eden";
```

# 代理陷阱

对于可以在对象上执行的不同操作，有不同的陷阱。一些陷阱需要返回值。在返回值时，你需要遵循一些规则。返回的值被代理拦截以过滤和/或检查返回的值是否遵守规则。如果一个陷阱在返回值时违反了规则，那么代理会抛出`TypeError`异常。

陷阱内部的`this`值始终是处理器的引用。

让我们来看看各种陷阱。

# `get(target, property, receiver)`方法

当我们使用点或方括号表示法检索属性值，或使用`Reflect.get()`方法时，会执行获取陷阱。它接受三个参数——即`target`对象、属性名和代理。

它必须返回一个表示属性值的值。以下是一个代码示例，展示了如何使用获取陷阱：

```js
const proxy = new Proxy({ 
    age: 12 
}, { 
get(target, property, receiver) { 
    if(property in target) { 
        return target[property]; 
    }
    return "Property not found!"; 
} 
}); 
console.log(proxy.age); 
console.log(proxy.name);
```

输出，正如你可能已经猜到的，如下所示：

```js
12
Property not found!
```

而不是输出，没有代理时你会得到以下内容：

```js
12
undefined
```

在这里，我们可以看到获取陷阱在`target`对象中查找属性，如果找到了，则返回`property`值。否则，它返回一个字符串，表示未找到。

接收器参数是我们打算访问其属性的对象的引用。考虑以下示例，以更好地理解接收器参数的值：

```js
const proxy = new Proxy({
    age: 13
}, { 
get(target, property, receiver) { 
    if(property in target) { 
        return target[property]; 
    } else if(property == "name") {
        console.log("Receiver here is ", receiver);
        return "backup property value for name.";
    } else { 
        return console.log("Property Not Found ", receiver); 
    } 
} 
}); 
let temp = proxy.name; 
let obj = { 
    age: 12, 
    __proto__: proxy 
};
temp = obj.name;
const justARandomVariablePassingBy = obj.age;
console.log(justARandomVariablePassingBy);
```

输出：

```js
Receiver here is <ProxyObject> {age: 13}
Receiver here is {age: 12}
12
```

注意，这里的`{age: 13}`是`ProxyObject`，`{ age: 12 }`是普通对象。

在这里，`obj`继承了`proxy`对象。因此，当名称属性在`obj`对象中未找到时，它会在`proxy`对象中查找。由于`proxy`对象有一个获取陷阱，它提供了一个值。

因此，当我们通过`obj.name`表达式访问名称属性时，接收器参数的值是`obj`，当我们通过`proxy.name`访问名称属性时，表达式是`proxy`。

对于所有其他陷阱，接收器参数的值决定方式相同。

# 使用获取陷阱的规则

在使用获取陷阱时，不应违反以下规则：

+   如果`target`对象属性是一个不可写、不可配置的数据属性，则返回的属性值必须与`target`对象属性的值相同。

+   如果 `target` 对象的属性是一个不可配置的访问器属性，并且其 `[[Get]]` 属性为 `undefined`，则返回值必须为 `undefined`。

# `set(target, property, value, receiver)` 方法

当我们使用赋值运算符或 `Reflect.set()` 方法设置属性值时，将调用 `set` 陷阱。它接受四个参数--即，`target` 对象、属性名、新的属性值和接收者。

如果赋值成功，`set` 陷阱必须返回 `true`。否则，它将返回 `false`。

下面是一个代码示例，演示了如何使用 `set` 陷阱：

```js
const proxy = new Proxy({}, { 
set(target, property, value, receiver) { 
    target[property] = value; 
    return true; 
} 
}); 
proxy.name = "Eden";
console.log(proxy.name); //Output "Eden"
```

# 使用 `set` 陷阱的规则

在使用 `set` 陷阱时，不应违反这些规则：

+   如果 `target` 对象的属性是一个不可写、不可配置的数据属性，则它将返回 `false`--也就是说，您不能更改属性值

+   如果 `target` 对象的属性是一个不可配置的访问器属性，并且其 `[[Set]]` 属性为 `undefined`，则它将返回 `false`--也就是说，您不能更改属性值

# `has(target, property)` 方法

当我们使用 `in` 运算符检查属性是否存在时，将执行 `has` 陷阱。它接受两个参数--即，`target` 对象和属性名。它必须返回一个布尔值，指示属性是否存在。

下面是一个代码示例，演示了如何使用 `has` 陷阱：

```js
const proxy = new Proxy({age: 12}, { 
has(target, property) { 
    return property in target;
}
}); 
console.log(Reflect.has(proxy, "name")); 
console.log(Reflect.has(proxy, "age"));
```

输出如下：

```js
false
true
```

# 使用 `has` 陷阱的规则

在使用 `has` 陷阱时，不应违反这些规则：

+   如果属性作为非配置属性存在于 `target` 对象中，并且是其自身属性，则不能返回 `false`

+   如果属性作为 `target` 对象的自身属性存在，并且 `target` 对象是不可扩展的，则不能返回 `false`

# `isExtensible(target)` 方法

当我们使用 `Object.isExtensible()` 方法检查对象是否可扩展时，将执行 `isExtensible` 陷阱。它只接受一个参数--即，`target` 对象。它必须返回一个布尔值，指示对象是否可扩展。

下面是一个代码示例，演示了如何使用 `isExtensible` 陷阱：

```js
const proxy = new Proxy({age: 12}, { 
isExtensible(target) { 
    return Object.isExtensible(target); 
} 
}); 
console.log(Reflect.isExtensible(proxy)); //Output "true"
```

# 使用 `isExtensible` 陷阱的规则

在使用 `isExtensible` 陷阱时，不应违反此规则：

+   如果目标对象是可扩展的，则不能返回 `false`。同样，如果目标对象是不可扩展的，则不能返回 `true`

# `getPrototypeOf(target)` 方法

当我们使用 `Object.getPrototypeOf()` 方法或 `__proto__` 属性检索内部 [[prototype]] 属性的值时，将执行 `getPrototypeOf` 陷阱。它只接受一个参数--即，`target` 对象。

它必须返回一个对象或 `null` 值。`null` 值表示该对象没有继承其他任何内容，并且是继承链的末端。

下面是一个代码示例，演示了如何使用 `getPrototypeOf` 陷阱：

```js
const proxy = new Proxy({
    age: 12, 
    __proto__: {name: "Eden"}
}, 
{ 
    getPrototypeOf(target) { 
        return Object.getPrototypeOf(target); 
    } 
}); 

console.log(Reflect.getPrototypeOf(proxy).name); //Output "Eden"
```

# 使用 `getPrototypeOf` 陷阱的规则

在使用 `getPrototypeOf` 陷阱时，不应违反这些规则：

+   它必须返回一个对象或返回一个空值

+   如果目标是不可扩展的，那么这个陷阱必须返回实际的原型

# `setPrototypeOf(target, prototype)` 方法

当我们使用 `Object.setPrototypeOf()` 方法或 `__proto__` 属性设置内部 [[prototype]] 属性的值时，将执行 `setPrototypeOf` 陷阱。它接受两个参数——即 `target` 对象和要分配的属性的值。

这个陷阱将返回一个布尔值，表示是否成功设置了原型。

这里有一个代码示例，演示了如何使用 `setPrototypeOf` 陷阱：

```js
const proxy = new Proxy({}, { 
    setPrototypeOf(target, value) { 
        Reflect.setPrototypeOf(target, value); 
        return true; 
    } 
}); 

Reflect.setPrototypeOf(proxy, {name: "Eden"}); console.log(Reflect.getPrototypeOf(proxy).name); //Output "Eden"
```

# 使用 `setPrototypeOf` 陷阱的规则

在使用 `setPrototypeOf` 陷阱时，不应违反以下规则：

+   如果 `target` 不是不可扩展的，你必须返回 `false`

# `preventExtensions(target)` 方法

当我们使用 `Object.preventExtensions()` 方法防止添加新属性时，将执行 `preventExtensions` 陷阱。它只接受一个参数——即 `target` 对象。

它必须返回一个布尔值，表示是否成功阻止了对象的扩展。

这里有一个代码示例，演示了如何使用 `preventExtensions` 陷阱：

```js
const proxy = new Proxy({}, { 
preventExtensions(target) { 
    Object.preventExtensions(target); 
    return true; 
} 
}); 
Reflect.preventExtensions(proxy); 
proxy.a = 12; 
console.log(proxy.a); //Output "undefined"
```

# 使用 `preventExtensions` 陷阱的规则

在使用 `preventExtensions` 陷阱时，不应违反以下规则：

+   只有当 `target` 是不可扩展的或已使 `target` 不可扩展时，这个陷阱才能返回 `true`

# `getOwnPropertyDescriptor(target, property)` 方法

当我们使用 `Object.getOwnPropertyDescriptor()` 方法检索属性的描述符时，将执行 `getOwnPropertyDescriptor` 陷阱。它接受两个参数——即 `target` 对象和属性的名称。

这个陷阱必须返回一个 `descriptor` 对象或 `undefined`。如果属性不存在，则返回 `undefined` 值。

这里有一个代码示例，演示了如何使用 `getOwnPropertyDescriptor` 陷阱：

```js
const proxy = new Proxy({age: 12}, { 
getOwnPropertyDescriptor(target, property) { 
    return Object.getOwnPropertyDescriptor(target, property); 
} 
}); 

const descriptor = Reflect.getOwnPropertyDescriptor(proxy, "age"); console.log("Enumerable: " + descriptor.enumerable); 
console.log("Writable: " + descriptor.writable); 
console.log("Configurable: " + descriptor.configurable); 
console.log("Value: " + descriptor.value);
```

输出如下：

```js
Enumerable: true
Writable: true
Configurable: true
Value: 12
```

# 使用 `getOwnPropertyDescriptor` 陷阱的规则

在使用 `getOwnPropertyDescriptor` 陷阱时，不应违反以下规则：

+   这个陷阱必须返回一个对象或返回一个 `undefined` 属性

+   如果属性作为 `target` 对象的非配置性自有属性存在，则不能返回 `undefined` 值

+   如果属性作为 `target` 对象的自有属性存在，并且 `target` 对象是不可扩展的，则不能返回 `undefined` 值

+   如果属性不是 `target` 对象的自有属性，并且 `target` 对象是不可扩展的，那么你必须返回 `undefined`

+   如果属性作为 `target` 对象的自有属性存在，或者如果它作为 `target` 对象的可配置自有属性存在，则不能将返回的描述符对象的配置性设置为 `false`

# `defineProperty(target, property, descriptor)` 方法

当我们使用`Object.defineProperty()`方法定义属性时，会执行`defineProperty`陷阱。它接受三个参数——即`target`对象、属性名和`descriptor`对象。

这个陷阱应该返回一个布尔值，表示是否成功定义了属性。

下面是一个代码示例，演示了如何使用`defineProperty`陷阱：

```js
const proxy = new Proxy({}, { 
defineProperty(target, property, descriptor) { 
    Object.defineProperty(target, property, descriptor); 
    return true; 
} 
}); 
Reflect.defineProperty(proxy, "name", {value: "Eden"}); 
console.log(proxy.name); //Output "Eden"
```

# 使用`defineProperty`的规则

在使用`defineProperty`陷阱时，不应违反此规则：

+   如果`target`对象是不可扩展的，并且属性尚未存在，则必须返回`false`。

# `deleteProperty(target, property)`方法

当我们使用删除运算符或`Reflect.deleteProperty()`方法删除属性时，会执行`deleteProperty`陷阱。它接受两个参数——即`target`对象和属性名。

这个陷阱必须返回一个布尔值，表示属性是否成功删除。下面是一个代码示例，演示了如何使用`deleteProperty`陷阱：

```js
const proxy = new Proxy({age: 12}, { 
deleteProperty(target, property) { 
    return delete target[property]; 
} 
}); 
Reflect.deleteProperty(proxy, "age"); 
console.log(proxy.age); //Output "undefined"
```

# 使用`deleteProperty`陷阱的规则

在使用`deleteProperty`陷阱时，不应违反此规则：

+   这个陷阱必须返回`false`，如果属性作为`target`对象的非可配置自有属性存在

# `ownKeys(target)`方法

当我们使用`Reflect.ownKeys()`、`Object.getOwnPropertyNames()`、`Object.getOwnPropertySymbols()`和`Object.keys()`方法检索自有属性键时，会执行`ownKeys`陷阱。它只接受一个参数——即`target`对象。

`Reflect.ownKeys()`方法类似于`Object.getOwnPropertyNames()`方法——即它们都返回对象的可枚举和非可枚举属性键。

它们也都会忽略继承的属性。唯一的区别是`Reflect.ownKeys()`方法返回符号键和字符串键，而`Object.getOwnPropertyNames()`方法只返回字符串键。

`Object.getOwnPropertySymbols()`方法返回具有符号键的可枚举和非可枚举属性。它忽略了继承的属性。

`Object.keys()`方法类似于`Object.getOwnPropertyNames()`方法，但唯一的区别是`Object.keys()`方法只返回可枚举属性。

`ownKeys`陷阱必须返回一个数组，表示自有属性键。

下面是一个代码示例，演示了如何使用`ownKeys`陷阱：

```js
const s = Symbol(); 
const object = {age: 12, __proto__: {name: "Eden"}, [s]: "Symbol"}; Object.defineProperty(object, "profession", 
{ 
    enumerable: false, 
    configurable: false, 
    writable: false, 
    value: "Developer" 
}) 

const proxy = new Proxy(object, { 
    ownKeys(target) { 
        return Object.getOwnPropertyNames(target).concat(Object.getOwnPropertySymbols(target)); 
} 
}); 

console.log(Reflect.ownKeys(proxy)); 
console.log(Object.getOwnPropertyNames(proxy)); 
console.log(Object.keys(proxy)); 
console.log(Object.getOwnPropertySymbols(proxy));
```

输出如下：

```js
["age", "profession", Symbol()]
["age", "profession"]
["age"]
[Symbol()]
```

在这里，我们可以看到`ownKeys`陷阱返回的数组值被代理过滤，基于调用者期望的内容。例如，`Object.getOwnPropertySymbols()`调用者期望一个包含符号的数组。因此，代理从返回的数组中移除了字符串。

# 使用`ownKeys`陷阱的规则

这些规则在使用`ownKeys`陷阱时不应被违反：

+   返回数组的元素必须是字符串或符号

+   返回的数组必须包含`target`对象的所有非配置自有属性的键

+   如果`target`对象是不可扩展的，则返回的数组必须包含自有属性和`target`对象的所有键，并且不包含其他值

# `apply(target, thisValue, arguments)`方法

如果目标是函数，则调用代理将执行`apply`陷阱。`apply`陷阱也会为函数的`apply()`和`call()`方法以及`Reflect.apply()`方法执行。

`apply`陷阱接受三个参数。第一个参数是`target`对象，第三个参数是一个数组，表示函数调用的参数。

第二个参数与`target`函数的`this`值相同--即，如果目标函数没有使用代理被调用，则与目标函数的`this`值相同。

以下是一个代码示例，演示如何使用 apply 陷阱：

```js
const proxy = new Proxy(function(){}, { 
    apply(target, thisValue, arguments) { 
        console.log(thisValue.name); 
        return arguments[0] + arguments[1] + arguments[2]; 
    } 
});

const obj = { name: "Eden", f: proxy } 
const sum = obj.f(1,2,3); 
console.log(sum);
```

输出如下：

```js
Eden
6
```

# `construct(target, arguments)`方法

如果目标是函数，则使用`new`运算符或`Reflect.construct()`方法将目标作为构造函数调用将执行构造陷阱。

`construct`陷阱接受两个参数。第一个参数是`target`对象，第二个参数是一个数组，表示构造函数调用的参数。

`construct`陷阱必须返回一个对象，表示新创建的实例。以下是一个代码示例，演示如何使用`construct`陷阱：

```js
const proxy = new Proxy(function(){}, { 
    construct(target, arguments) { 
        return {name: arguments[0]}; 
    } 
}); 

const obj = new proxy("Eden"); 
console.log(obj.name); //Output "Eden"
```

# `Proxy.revocable(target, handler)`方法

可撤销代理是一种可以被撤销（即，关闭）的代理。

要创建可撤销代理，我们必须使用`Proxy.revocable()`方法。`Proxy.revocable()`方法不是一个构造函数。此方法也接受与`Proxy`构造函数相同的参数，但它不是直接返回一个可撤销代理实例，而是返回一个具有两个属性的对象，如下所示：

+   `proxy`：这是可撤销的`proxy`对象

+   `revoke`：当此函数被调用时，它将撤销代理

一旦撤销了可撤销的`proxy`，任何尝试使用它的操作都将抛出`TypeError`异常。以下是一个示例，演示如何创建可撤销的`proxy`并撤销它：

```js
const revocableProxy = Proxy.revocable({ age: 12 }, { 
get(target, property, receiver) { 
    if(property in target) {
         return target[property]; 
    }
    return "Not Found";  
} 
}); 
console.log(revocableProxy.proxy.age); 
revocableProxy.revoke(); 
console.log(revocableProxy.proxy.name);
```

输出如下：

```js
12
TypeError: proxy is revoked
```

# 可撤销代理的使用场景

您可以使用可撤销代理而不是常规代理。当您将代理传递给一个异步或并行运行的功能时，您可以撤销它，这样您就不希望该功能再使用该代理了。

# 代理的使用

代理有几种用途。以下是主要的使用场景：

+   创建虚拟化对象，例如远程对象、持久化对象等

+   对象的懒加载创建

+   透明的日志记录、跟踪、分析等

+   内嵌领域特定语言

+   通过强制访问控制来泛化抽象

# 概述

在本章中，我们学习了什么是代理以及如何使用它们。我们通过例子看到了可用的各种陷阱。我们还看到了不同陷阱需要遵循的不同规则。本章深入解释了 JavaScript 中 Proxy API 的各个方面。最后，我们看到了一些代理的使用案例。在下一章中，我们将探讨面向对象编程和 ES6 类。
