# 第六章：使用代理

代理用于定义对象基本操作的自定义行为。代理已经在编程语言如 C#、C++和 Java 中可用，但 JavaScript 从未有过代理。ES6 引入了 Proxy API，它允许我们创建代理。在本章中，我们将探讨代理、它们的用法和代理陷阱。由于代理的好处，开发者越来越多地使用代理，因此深入了解代理并举例说明非常重要，我们将在本章中这样做。

在本章中，我们将涵盖：

+   使用 Proxy API 创建代理

+   理解代理是什么以及如何使用它们

+   使用陷阱拦截对象上的各种操作

+   可用的不同类型的陷阱

+   代理的一些用例

# 代理概述

代理就像对象的包装器，并定义了对象基本操作的自定义行为。对象的一些基本操作包括属性查找、属性赋值、构造函数调用、枚举等。

一旦使用代理包装了一个对象，所有应该在对象上执行的操作现在都应该在代理对象上执行，以便执行自定义行为。

## 术语

在学习代理时，这里有一些重要的术语：

+   目标: 这是被代理器包装的对象。

+   陷阱: 这些是拦截目标对象上各种操作并定义这些操作自定义行为的函数。

+   处理器: 这是一个持有陷阱的对象。处理器附加到代理对象上。

# Proxy API

ES6 Proxy API 提供了`Proxy`构造函数来创建代理。`Proxy`构造函数接受两个参数，它们是：

+   **目标**: 这是将被代理器包装的对象

+   **处理器**: 这是一个包含目标对象陷阱的对象

可以为目标对象上的每个可能的操作定义一个陷阱。如果没有定义陷阱，则默认行为将在目标对象上执行。

这里有一个代码示例，展示了如何创建代理，并在目标对象上执行各种操作。在这个例子中，我们没有定义任何陷阱：

```js
var target = {
  age: 12
};
var handler = {};
var proxy = new Proxy(target, handler);

proxy.name = "Eden";

console.log(target.name);
console.log(proxy.name);
console.log(target.age);
console.log(proxy.age);
```

输出如下：

```js
Eden
Eden
12
12
```

在这里，我们可以看到`target`对象的`age`属性可以通过`proxy`对象访问。当我们向`proxy`对象添加`name`属性时，它实际上被添加到了`target`对象上。

由于没有为属性赋值附加陷阱，因此`proxy.name`赋值导致默认行为，即简单地将值赋给属性。

因此，我们可以说`proxy`只是`target`对象的包装器，并且可以定义陷阱以更改操作的默认行为。

许多开发者没有为目标对象保留引用变量以使用代理强制访问对象。只有在你需要为多个代理重用处理器时才保留处理器的引用。以下是他们重写之前代码的方式：

```js
var proxy = new Proxy({
  age: 12
}, {});

proxy.name = "Eden";
```

## 陷阱

对于可以在对象上执行的不同操作，有不同的陷阱。其中一些陷阱需要返回值。在返回值时，它们需要遵循一些规则。返回的值被代理拦截以进行过滤，以及/或检查返回的值是否遵守规则。如果一个陷阱在返回值时违反规则，则代理抛出 `TypeError` 异常。

在陷阱内部，`this` 的值始终是处理器的引用。

让我们来看看各种陷阱。

### `get(target, property, receiver)` 方法

当我们使用点或方括号表示法检索属性值，或使用 `Reflect.get()` 方法时，会执行 `get` 陷阱。它接受三个参数，即目标对象、属性名和代理。

它必须返回表示属性值的值。

这里是一个代码示例，展示了如何使用 `get` 陷阱：

```js
var proxy = new Proxy({
    age: 12
  }, {
    get: function(target, property, receiver){
      if(property in target)
      {
        return target[property];
      }
      else
      {
        return "Not Found";
      }
    }
  }
);

console.log(Reflect.get(proxy, "age"));
console.log(Reflect.get(proxy, "name"));
```

输出如下：

```js
12
Not found
```

在这里，我们可以看到 `get` 陷阱在 `target` 对象中查找属性，如果找到，则返回属性值。否则，它返回一个字符串，表示未找到。

`receiver` 参数是我们打算访问其属性的对象的引用。考虑以下示例以更好地理解 `receiver` 参数的值：

```js
var proxy = new Proxy({age: 13}, {
    get: function(target, property, receiver){

      console.log(receiver);

      if(property in target)
      {
        console.log(receiver);
        return target[property];
      }
      else
      {
        return "Not Found";
      }
    }
  }
);

var temp = proxy.name;

var obj = {
  age: 12,
  __proto__: proxy
}

temp = obj.name;
```

输出如下：

```js
{age: 13}
{age: 12}
```

这里 `obj` 继承了 `proxy` 对象。因此，当在 `obj` 对象中找不到 `name` 属性时，它会在 `proxy` 对象中查找。由于 `proxy` 对象有一个 `get` 陷阱，它提供了一个值。

因此，当我们通过 `obj.name` 表达式访问 `name` 属性时，`receiver` 参数的值是 `obj`，当我们通过 `proxy.name` 表达式访问 `name` 属性时，是 `proxy`。

对于所有其他陷阱，`receiver` 参数的值也是以相同的方式决定的。

#### 规则

在使用 `get` 陷阱时，不应违反以下规则：

+   如果目标对象的属性是一个不可写、不可配置的数据属性，则返回该属性的值必须与目标对象属性的值相同。

+   如果目标对象的属性是一个不可配置的访问器属性，并且其 `[[Get]]` 属性为 `undefined`，则返回该属性的值必须是 `undefined`。

### `set(target, property, value, receiver)` 方法

当我们使用赋值运算符或 `Reflect.set()` 方法设置属性值时，会调用 `set` 陷阱。它接受四个参数，即目标对象、属性名、新属性值和接收者。

如果赋值成功，`set` 陷阱必须返回 `true`。否则，它将返回 `false`。

这里是一个代码示例，展示了如何使用 `set` 陷阱：

```js
var proxy = new Proxy({}, {
  set: function(target, property, value, receiver){
    target[property] = value;
    return true;
  }
});

Reflect.set(proxy, "name", "Eden");
console.log(proxy.name); //Output "Eden"
```

#### 规则

在使用 `set` 陷阱时，不应违反以下规则：

+   如果目标对象属性是一个不可写、不可配置的数据属性，那么它将返回 `false`，也就是说，你不能更改属性值

+   如果目标对象属性是一个不可配置的访问器属性，并且其 `[[Set]]` 属性为 `undefined`，那么它将返回 `false`，也就是说，你不能更改属性值

### `has(target, property)` 方法

当我们使用 `in` 操作符检查属性是否存在时，将执行 `has` 陷阱。它接受两个参数，即目标对象和属性名。它必须返回一个布尔值，表示属性是否存在。

这里有一个代码示例，演示了如何使用 `has` 陷阱：

```js
var proxy = new Proxy({age: 12}, {
  has: function(target, property){
    if(property in target)
    {
      return true;
    }
    else
    {
      return false;
    }
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

#### 规则

在使用 `has` 陷阱时，不应违反以下规则：

+   如果属性作为 `target` 对象的非配置性自身属性存在，你不能返回 `false`

+   如果属性作为 `target` 对象的自身属性存在，并且 `target` 对象不可扩展，你不能返回 `false`

### `isExtensible(target)` 方法

当我们使用 `Object.isExtensible()` 方法检查对象是否可扩展时，将执行 `isExtensible` 陷阱。它只接受一个参数，即 `target` 对象。它必须返回一个布尔值，表示对象是否可扩展。

这里有一个代码示例，演示了如何使用 `isExtensible` 陷阱：

```js
var proxy = new Proxy({age: 12}, {
  isExtensible: function(target){
    return Object.isExtensible(target);
  }
});

console.log(Reflect.isExtensible(proxy)); //Output "true"
```

#### 规则

在使用 `isExtensible` 陷阱时，不应违反以下规则：

+   如果目标可扩展，你不能返回 `false`。同样，如果目标不可扩展，你不能返回 `true`。

### `getPrototypeOf(target)` 方法

当我们使用 `Object.getPrototypeOf()` 方法或 `__proto__` 属性检索内部 `[[prototype]]` 属性的值时，将执行 `getPrototypeOf` 陷阱。它只接受一个参数，即 `target` 对象。

它必须返回一个对象或 `null` 值。`null` 值表示对象没有继承其他内容，并且是继承链的末端。

这里有一个代码示例，演示了如何使用 `getPrototypeOf` 陷阱：

```js
var proxy = new Proxy({age: 12, __proto__: {name: "Eden"}}, {
  getPrototypeOf: function(target){
    return Object.getPrototypeOf(target);
  }
});

console.log(Reflect.getPrototypeOf(proxy).name); //Output "Eden"
```

#### 规则

在使用 `getPrototypeOf` 陷阱时，不应违反以下规则：

+   它必须返回一个对象或返回 `null` 值。

+   如果目标不可扩展，那么这个陷阱必须返回实际的原型

### `setPrototypeOf(target, prototype)` 方法

当我们使用 `Object.setPrototypeOf()` 方法或 `__proto__` 属性设置内部 `[[prototype]]` 属性的值时，将执行 `setPrototypeOf` 陷阱。它接受两个参数，即目标对象和要分配的属性值。

这个陷阱将返回一个布尔值，表示是否已成功设置原型。

这里有一个代码示例，演示了如何使用 `setPrototypeOf` 陷阱：

```js
var proxy = new Proxy({}, {
  setPrototypeOf: function(target, value){
    Reflect.setPrototypeOf(target, value);
    return true;
  }
});

Reflect.setPrototypeOf(proxy, {name: "Eden"});

console.log(Reflect.getPrototypeOf(proxy).name); //Output "Eden"
```

#### 规则

在使用 `setPrototypeOf` 陷阱时，不应违反以下规则：

+   如果目标不可扩展，你必须返回`false`。

### `preventExtensions(target)`方法

当我们使用`Object.preventExtensions()`方法防止添加新属性时，会执行`preventExtensions`陷阱。它只接受一个参数，即`target`对象。

它必须返回一个布尔值，指示是否已成功防止对象的扩展。

这里是一个代码示例，演示了如何使用`preventExtensions`陷阱：

```js
var proxy = new Proxy({}, {
  preventExtensions: function(target){
    Object.preventExtensions(target);
    return true;
  }
});

Reflect.preventExtensions(proxy);

proxy.a = 12;
console.log(proxy.a); //Output "undefined"
```

#### 规则

在使用`preventExtensions`陷阱时，不应违反以下规则：

+   此陷阱只有在目标不可扩展或已使目标不可扩展的情况下才能返回`true`。

### `getOwnPropertyDescriptor(target, property)`方法

当我们使用`Object.getOwnPropertyDescriptor()`方法检索属性的描述符时，会执行`getOwnPropertyDescriptor`陷阱。它接受两个参数，即目标对象和属性名称。

此陷阱必须返回一个描述符对象或`undefined`。如果属性不存在，则返回`undefined`值。

这里是一个代码示例，演示了如何使用`getOwnPropertyDescriptor`陷阱：

```js
var proxy = new Proxy({age: 12}, {
  getOwnPropertyDescriptor: function(target, property){
    return Object.getOwnPropertyDescriptor(target, property);
  }
});

var descriptor = Reflect.getOwnPropertyDescriptor(proxy, "age");

console.log("Enumerable: " + descriptor.enumerable);
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

#### 规则

在使用`getOwnPropertyDescriptor`陷阱时，不应违反以下规则：

+   此陷阱必须返回一个对象或返回一个`undefined`属性。

+   如果属性作为`target`对象的非可配置自有属性存在，则不能返回`undefined`值。

+   如果属性作为`target`对象的非可配置自有属性存在，并且`target`对象不可扩展，则不能返回`undefined`值。

+   如果属性不是`target`对象的自有属性，并且`target`对象不可扩展，你必须返回`undefined`。

+   如果属性作为`target`对象的自有属性存在，或者作为可配置的自有属性存在，则不能将返回的描述符对象的`configurable`属性设置为`false`。

### `defineProperty(target, property, descriptor)`方法

当我们使用`Object.defineProperty()`方法定义属性时，会执行`defineProperty`陷阱。它接受三个参数，即`target`对象、属性名称和描述符对象。

此陷阱应返回一个布尔值，指示是否已成功定义属性。

这里是一个代码示例，演示了如何使用`defineProperty`陷阱：

```js
var proxy = new Proxy({}, {
  defineProperty: function(target, property, descriptor){
    Object.defineProperty(target, property, descriptor);
    return true;
  }
});

Reflect.defineProperty(proxy, "name", {value: "Eden"});

console.log(proxy.name); //Output "Eden"
```

#### 规则

在使用`defineProperty`陷阱时，不应违反以下规则：

+   如果`target`对象不可扩展，并且属性尚未存在，则必须返回`false`。

### `deleteProperty(target, property)`方法

当我们使用`delete`运算符或`Reflect.deleteProperty()`方法删除属性时，会执行`deleteProperty`陷阱。它接受两个参数，即`target`对象和`property`名称。

此陷阱必须返回一个布尔值，指示属性是否成功删除。

下面是一个代码示例，演示了如何使用 `deleteProperty` 陷阱：

```js
var proxy = new Proxy({age: 12}, {
  deleteProperty: function(target, property){
      return delete target[property];
  }
});

Reflect.deleteProperty(proxy, "age");
console.log(proxy.age); //Output "undefined"
```

#### 规则

在使用 `deleteProperty` 陷阱时，不应违反此规则：

+   如果属性作为 `target` 对象的非配置性自身属性存在，则此陷阱必须返回 `false`。

### `enumerate(target)` 方法

当我们使用 `for...in` 循环或 `Reflect.enumerate()` 方法遍历属性键时，会执行 `enumerate` 陷阱。它接受一个参数，即 `target` 对象。

这个陷阱必须返回一个迭代器对象，表示对象的可枚举键。

下面是一个代码示例，演示了如何使用 `enumerate` 陷阱：

```js
var proxy = new Proxy({age: 12, name: "Eden"}, {
  enumerate: function(target){
    var arr = [];

    for(var p in target)
    {
      arr[arr.length] = p;
    }

    return arr[Symbol.iterator]();
  }
});

var iterator = Reflect.enumerate(proxy);

console.log(iterator.next().value);
console.log(iterator.next().value);
console.log(iterator.next().done);
```

输出如下：

```js
age
name
true
```

#### 规则

在使用 `enumerate` 陷阱时，不应违反此规则：

+   此陷阱必须返回一个对象

### `ownKeys(target)` 方法

当我们使用 `Reflect.ownKeys()`、`Object.getOwnPropertyNames()`、`Object.getOwnPropertySymbols()` 和 `Object.keys()` 方法检索自身属性键时，会执行 `ownKeys` 陷阱。它只接受一个参数，即 `target` 对象。

`Reflect.ownKeys()` 方法类似于 `Object.getOwnPropertyNames()` 方法，即它们返回对象的枚举和非枚举属性键。它们忽略继承的属性。唯一的区别是 `Reflect.ownKeys()` 方法返回的是符号键和字符串键，而 `Object.getOwnPropertyNames()` 方法只返回字符串键。

`Object.getOwnPropertySymbols()` 方法返回键为符号的可枚举和非枚举属性。它忽略继承的属性。

`Object.keys()` 方法类似于 `Object.getOwnPropertyNames()` 方法，但唯一的不同是 `Objecy.keys()` 方法只返回可枚举属性。

`ownKeys` 陷阱必须返回一个数组，表示自身的属性键。

下面是一个代码示例，演示了如何使用 `ownKeys` 陷阱：

```js
var s = Symbol();

var object = {age: 12, __proto__: {name: "Eden"}, [s]: "Symbol"};

Object.defineProperty(object, "profession", {
  enumerable: false,
  configurable: false,
  writable: false,
  value: "Developer"
})

var proxy = new Proxy(object, {
  ownKeys: function(target){
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

在这里，我们可以看到 `ownKeys` 陷阱返回的数组值被代理根据调用者期望的值进行过滤。例如，`Object.getOwnPropertySymbols()` 调用者期望一个包含符号的数组。因此，代理从返回的数组中移除了字符串。

#### 规则

在使用 `ownKeys` 陷阱时，不应违反这些规则：

+   返回数组的元素必须是字符串或符号

+   返回的数组必须包含 `target` 对象所有非配置性自身属性的键。

+   如果 `target` 对象不可扩展，则返回的数组必须包含 `target` 对象所有自身属性的键，且不包含其他值。

### `apply(target, thisValue, arguments)` 方法

如果目标是函数，则调用代理将执行 `apply` 陷阱。`apply` 陷阱也会在函数的 `apply()` 和 `call()` 方法以及 `Reflect.apply()` 方法中执行。

`apply` 陷阱接受三个参数。第一个参数是 `target` 对象，第三个参数是一个数组，表示函数调用时的参数。第二个参数与目标函数的 `this` 值相同，即它等同于目标函数在没有代理的情况下被调用时的 `this` 值。

下面是一个代码示例，演示了如何使用 `apply` 陷阱：

```js
var proxy = new Proxy(function(){}, {
  apply: function(target, thisValue, arguments){
    console.log(thisValue.name);
    return arguments[0] + arguments[1] + arguments[2];
  }
});

var obj = {
  name: "Eden",
  f: proxy
}

var sum = obj.f(1,2,3);

console.log(sum);
```

输出如下：

```js
Eden
6
```

### `construct(target, arguments)` 方法

如果目标是函数，则使用 `new` 操作符或 `Reflect.construct()` 方法将目标作为构造函数调用时，将执行 `construct` 陷阱。

`construct` 陷阱接受两个参数。第一个参数是 `target` 对象，第二个参数是一个数组，表示构造函数调用时的参数。

`construct` 陷阱必须返回一个对象，表示新创建的实例。

下面是一个代码示例，演示了如何使用 `construct` 陷阱：

```js
var proxy = new Proxy(function(){}, {
  construct: function(target, arguments){
    return {name: arguments[0]};
  }
});

var obj = new proxy("Eden");
console.log(obj.name); //Output "Eden"
```

## `Proxy.revocable(target, handler)` 方法

可撤销代理是一种可以被撤销（即关闭）的代理。

要创建可撤销的代理，我们必须使用 `Proxy.revocable()` 方法。`Proxy.revocable()` 方法不是一个构造函数。此方法也接受与 `Proxy` 构造函数相同的参数，但它不会直接返回一个可撤销的代理实例，而是返回一个具有两个属性的对象，具体如下：

+   `proxy`: 这是一个可撤销的代理对象

+   `revoke`: 当此函数被调用时，它将撤销 `proxy`

一旦撤销了可撤销代理，任何尝试使用它的操作都将抛出 `TypeError` 异常。

下面是一个演示如何创建可撤销代理并撤销它的示例：

```js
var revocableProxy = Proxy.revocable({
  age: 12
 }, {
  get: function(target, property, receiver){
    if(property in target)
    {
     return target[property];
    }
    else
    {
     return "Not Found";
    }
  }
 }
);

console.log(revocableProxy.proxy.age);

revocableProxy.revoke();

console.log(revocableProxy.proxy.name);
```

输出如下：

```js
12
TypeError: proxy is revoked
```

### 用例

你可以使用可撤销代理代替常规代理。当你将代理传递给一个异步运行或并行运行的函数时，你可以使用它，这样你就可以随时撤销它，以防你不想让该函数再使用该代理。

# 代理的使用

代理有多种用途。以下是一些主要用例：

+   创建虚拟对象，例如远程对象、持久对象等

+   对象的懒加载创建

+   透明的日志记录、跟踪、分析等

+   嵌入特定领域的语言

+   通过泛化插入抽象来强制执行访问控制

# 摘要

在本章中，我们学习了什么是代理以及如何使用它们。我们通过示例看到了可用的各种陷阱。我们还看到了不同陷阱需要遵循的不同规则。本章深入解释了 ES6 Proxy API 的所有内容。最后，我们看到了一些代理的使用案例。

在下一章中，我们将介绍面向对象编程和 ES6 类。
