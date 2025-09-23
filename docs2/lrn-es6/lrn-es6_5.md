# 第五章：实现 Reflect API

ES6 引入了一个新的 API，即 **Reflect API**，用于对象反射（即检查和操作对象的属性）。尽管 ES5 已经有了对象反射的 API，但这些 API 组织得并不好，并且在失败时，它们通常会抛出异常。ES6 的 Reflect API 组织得很好，使得代码的阅读和编写更加容易，因为它在失败时不会抛出异常。相反，它返回一个布尔值，表示操作是否成功。由于开发者正在适应 Reflect API 进行对象反射，因此深入了解这个 API 非常重要。

在本章中，我们将介绍：

+   使用给定的 `this` 值调用函数

+   使用另一个构造函数的 `prototype` 属性调用构造函数

+   定义或修改对象属性的属性

+   使用迭代器对象遍历对象的属性

+   获取和设置对象的内部 `[[prototype]]` 属性

+   以及许多与检查和操作对象的方法和属性相关的其他操作。

# Reflect 对象

ES6 全局 `Reflect` 对象公开了所有新的对象反射方法。`Reflect` 不是一个函数对象，因此你不能调用 `Reflect` 对象。此外，你不能使用 `new` 操作符来使用它。

ES6 Reflect API 的所有方法都被封装在 `Reflect` 对象中，使其看起来组织得很好。

`Reflect` 对象提供了许多方法，在功能上与全局 `Object` 对象的方法重叠。

让我们看看 `Reflect` 对象提供的各种对象反射方法。

## `Reflect.apply(function, this, args)` 方法

`Reflect.apply()` 方法用于使用给定的 `this` 值调用函数。通过 `Reflect.apply()` 调用的函数被称为目标函数。它与函数对象的 `apply()` 方法相同。

`Reflect.apply()` 方法接受三个参数：

+   第一个参数表示目标函数。

+   第二个参数表示目标函数内部的 `this` 值。此参数是可选的。

+   第三个参数是一个数组对象，指定目标函数的参数。此参数是可选的。

`Reflect.apply()` 方法返回目标函数返回的任何内容。

这里有一个代码示例，演示如何使用 `Reflect.apply()` 方法：

```js
function function_name(a, b, c)
{
  return this.value + a + b + c;
}

var returned_value = Reflect.apply(function_name, {value: 100}, [10, 20, 30]);

console.log(returned_value); //Output "160"
```

## `Reflect.construct(constructor, args, prototype)` 方法

`Reflect.construct()` 方法用于将函数作为构造函数调用。它与 `new` 操作符类似。将要作为构造函数调用的函数被称为 **目标构造函数**。

有一个特殊的原因，你可能想要使用 `Reflect.construct()` 方法而不是 `new` 操作符，那就是当你想要目标构造函数的 `prototype` 与另一个构造函数的 `prototype` 匹配时。

`Reflect.construct()` 方法接受三个参数：

+   第一个参数是目标构造函数。

+   第二个参数是一个数组，指定目标构造函数的参数。此参数是可选的。

+   第三个参数是另一个构造函数，其 `prototype` 将用作目标构造函数的 `prototype`。此参数是可选的。

`Reflect.construct()` 方法返回由目标构造函数创建的新实例。

这里是代码示例，演示如何使用 `Reflect.constructor()` 方法：

```js
function constructor1(a, b)
{
  this.a = a;
  this.b = b;

  this.f = function(){
    return this.a + this.b + this.c;
  }
}

function constructor2(){}
constructor2.prototype.c = 100;

var myObject = Reflect.construct(constructor1, [1,2], constructor2);

console.log(myObject.f()); //Output "103"
```

在前面的例子中，我们在调用 `constructor1` 时使用了 `consturctor2` 的 `prototype` 作为 `constructor1` 的 `prototype`。

## `Reflect.defineProperty(object, property, descriptor)` 方法

`Reflect.defineProperty()` 方法直接在对象上定义新属性或修改对象上的现有属性。它返回一个布尔值，指示操作是否成功。

它类似于 `Object.defineProperty()` 方法。区别在于 `Reflect.defineProperty()` 方法返回一个布尔值，而 `Object.defineProperty()` 返回修改后的对象。如果 `Object.defineProperty()` 方法无法修改或定义对象属性，则抛出异常，而 `Reflect.defineProperty()` 方法返回 `false` 结果。

`Reflect.defineProperty()` 方法接受三个参数：

+   第一个参数是用于定义或修改属性的对象

+   第二个参数是要定义或修改的属性的符号或名称

+   第三个参数是要定义或修改的属性的描述符

### 理解数据属性和访问器属性

自从 ES5 以来，每个对象属性要么是数据属性，要么是访问器属性。数据属性有一个值，这个值可能是可写的，也可能不可写，而访问器属性有一对用于设置和检索属性值的函数。

数据属性的特性包括值、可写、可枚举和可配置。另一方面，访问器属性的特性包括 `set`、`get`、`enumerable` 和 `configurable`。

**描述符**是一个描述属性特性的对象。当使用 `Reflect.defineProperty()` 方法、`Object.defineProperty()` 方法、`Object.defineProperties()` 方法或 `Object.create()` 方法创建属性时，我们需要传递属性描述符。

数据属性的描述符对象具有以下属性：

+   **值**：这是与属性关联的值。默认值是 `undefined`。

+   **可写性**：如果这是 `true`，则可以使用赋值运算符更改属性值。默认值是 `false`。

+   **可配置性**：如果这是 `true`，则可以更改属性属性，并且可以删除属性。默认值是 `false`。记住，当可配置属性为 `false` 且可写为 `true` 时，可以更改值和可写属性。

+   **可枚举**：如果这是 `true`，则该属性会在 `for…in` 循环和 `Object.keys()` 方法中出现。默认值是 `false`。

访问器属性的描述符具有以下属性：

+   **获取**：这是一个返回属性值的函数。该函数没有参数，默认值是未定义。

+   **设置**：这是一个设置属性值的函数。该函数将接收被分配给属性的新值。

+   **可配置**：如果这是 `true`，则属性描述符可以被更改，并且属性可以被删除。默认值是 `false`。

+   **可枚举**：如果这是 `true`，则该属性会在 `for…in` 循环和 `Object.keys()` 方法中出现。默认值是 `false`。

根据描述符对象的属性，JavaScript 决定属性是数据属性还是访问器属性。

如果你没有使用 `Reflect.defineProperty()` 方法、`Object.defineProperty()` 方法、`Object.defineProperties()` 方法或 `Object.create()` 方法添加属性，则该属性是一个数据属性，并且 `writable`、`enumerable` 和 `configurable` 属性都被设置为 `true`。属性添加后，你可以修改其属性。

当调用 `Reflect.defineProperty()` 方法、`Object.defineProperty()` 方法或 `Object.defineProperties()` 方法时，如果对象已经存在具有指定名称的属性，则该属性会被修改。描述符中未指定的属性保持不变。

你可以将数据属性更改为访问器属性，反之亦然。如果你这样做，描述符中未指定的 `configurable` 和 `enumerable` 属性将在属性中保留。其他未在描述符中指定的属性将被设置为它们的默认值。

这里是一个示例代码，展示了如何使用 `Reflect.defineProperty()` 方法创建一个数据属性：

```js
var obj = {}

Reflect.defineProperty(obj, "name", {
  value: "Eden",
  writable: true,
  configurable: true,
  enumerable: true
});

console.log(obj.name); //Output "Eden"
```

这里是另一个示例代码，展示了如何使用 `Reflect.defineProperty()` 方法创建一个访问器属性：

```js
var obj = {
  __name__: "Eden"
}

Reflect.defineProperty(obj, "name", {
  get: function(){
    return this.__name__;
  },
  set: function(newName){
    this.__name__ = newName;
  },
  configurable: true,
  enumerable: true
});

obj.name = "John";
console.log(obj.name);      //Output "John"
```

## `Reflect.deleteProperty(object, property)` 方法

`Reflect.deleteProperty()` 方法用于删除对象的一个属性。它与 `delete` 操作符相同。

此方法接受两个参数，即第一个参数是对象的引用，第二个参数是要删除的属性的名称。如果 `Reflect.deleteProperty()` 方法成功删除了属性，则返回 `true`。否则，它返回 `false`。

这里是一个示例代码，展示了如何使用 `Reflect.deleteProperty()` 方法删除一个属性：

```js
var obj = {
  name: "Eden"
}

console.log(obj.name);      //Output "Eden"

Reflect.deleteProperty(obj, "name");

console.log(obj.name);      //Output "undefined"
```

## `Reflect.enumerate(object)` 方法

`Reflect.enumerate()` 方法接受一个对象作为参数，并返回一个迭代器对象，该对象表示对象的可枚举属性。它还返回对象的继承的可枚举属性。

`Reflect.enumerate()` 方法类似于 `for…in` 循环。`Reflect.enumerate()` 方法返回一个迭代器，而 `for…in` 循环遍历可枚举属性。

这里有一个示例来演示如何使用 `Reflect.enumerate()` 方法：

```js
var obj = {
  a: 1,
  b: 2,
  c: 3
};

var iterator = Reflect.enumerate(obj);

console.log(iterator.next().value);
console.log(iterator.next().value);
console.log(iterator.next().value);
console.log(iterator.next().done);
```

输出如下：

```js
a
b
c
true
```

## `Reflect.get(object, property, this)` 方法

`Reflect.get()` 方法用于检索对象的属性值。第一个参数是对象，第二个参数是属性名。如果属性是访问器属性，则我们可以提供一个第三个参数，它将是 `get` 函数内部的 `this` 的值。

这里是一个代码示例，展示了如何使用 `Reflect.get()` 方法：

```js
var obj = {
  __name__: "Eden"
};

Reflect.defineProperty(obj, "name", {
  get: function(){
    return this.__name__;
  }
});

console.log(obj.name);      //Output "Eden"

var name = Reflect.get(obj, "name", {__name__: "John"});

console.log(name);      //Output "John"
```

## `Reflect.set(object, property, value, this)` 方法

`Reflect.set()` 方法用于设置对象的属性值。第一个参数是对象，第二个参数是属性名，第三个参数是属性值。如果属性是访问器属性，则我们可以提供一个第四个参数，它将是 `set` 函数内部的 `this` 的值。

`Reflect.set()` 方法如果成功设置属性值，则返回 `true`。否则，它返回 `false`。

这里是一个代码示例，展示了如何使用 `Reflect.set()` 方法：

```js
var obj1 = {
  __name__: "Eden"
};

Reflect.defineProperty(obj1, "name", {
  set: function(newName){
    this.__name__ = newName;
  },

  get: function(){
    return this.__name__;
  }
});

var obj2 = {
  __name__: "John"
};

Reflect.set(obj1, "name", "Eden", obj2);

console.log(obj1.name); //Output "Eden"
console.log(obj2.__name__); //Output "Eden"
```

## `Reflect.getOwnPropertyDescriptor(object, property)` 方法

`Reflect.getOwnPropertyDescriptor()` 方法用于检索对象的属性描述符。

`Reflect.getOwnPropertyDescriptor()` 方法与 `Object.getOwnPropertyDescriptor()` 方法相同。`Reflect.getOwnPropertyDescriptor()` 方法接受两个参数。第一个参数是对象，第二个参数是属性名。

这里有一个示例来演示 `Reflect.getOwnPropertyDescriptor()` 方法：

```js
var obj = {
  name: "Eden"
};

var descriptor = Reflect.getOwnPropertyDescriptor(obj, "name");

console.log(descriptor.value);
console.log(descriptor.writable);
console.log(descriptor.enumerable);
console.log(descriptor.configurable);
```

输出如下：

```js
Eden
true
true
true
```

## `Reflect.getPrototypeOf(object)` 方法

`Reflect.getPrototypeOf()` 方法用于检索对象的原型，即对象的内部 `[[prototype]]` 属性的值。

`Reflect.getPrototypeOf()` 方法与 `Object.getPrototypeOf()` 方法相同。

这里是展示如何使用 `Reflect.getPrototypeOf()` 方法的代码示例：

```js
var obj1 = {
  __proto__: {
    name: "Eden"
  }
};

var obj2 = Reflect.getPrototypeOf(obj1);

console.log(obj2.name); //Output "Eden"
```

## `Reflect.setPrototypeOf(object, prototype)` 方法

`Reflect.setPrototypeOf()` 用于设置对象的内部 `[[prototype]]` 属性的值。如果成功设置内部 `[[prototype]]` 属性的值，`Reflect.setPrototypeOf()` 方法将返回 `true`。否则，它将返回 `false`。

这里是一个代码示例，它展示了如何使用它：

```js
var obj = {};

Reflect.setPrototypeOf(obj, {
  name: "Eden"
});

console.log(obj.name); //Output "Eden"
```

## `Reflect.has(object, property)` 方法

`Reflect.has()` 用于检查属性是否存在于对象中。它也会检查继承属性。如果属性存在，则返回 `true`。否则，它将返回 `false`。

它与 `in` 操作符相同。

这里是一个代码示例，展示了如何使用 `Reflect.has()` 方法：

```js
var obj = {
  __proto__: {
    name: "Eden"
  },
  age: 12
};

console.log(Reflect.has(obj, "name")); //Output "true"
console.log(Reflect.has(obj, "age")); //Output "true"
```

## `Reflect.isExtensible(object)` 方法

`Reflect.isExtensible()` 方法用于检查一个对象是否可扩展，也就是说，我们是否可以向对象添加新的属性。

可以使用 `Object.preventExtensions()`、`Object.freeze()` 和 `Object.seal()` 方法将对象标记为不可扩展。

`Reflect.isExtensible()` 方法与 `Object.isExtensible()` 方法相同。

这里是代码示例，演示了如何使用 `Reflect.isExtensible()` 方法：

```js
var obj = {
  name: "Eden"
};

console.log(Reflect.isExtensible(obj)); //Output "true"

Object.preventExtensions(obj);

console.log(Reflect.isExtensible(obj)); //Output "false"
```

## `Reflect.preventExtensions(object)` 方法

`Reflect.preventExtensions()` 用于将对象标记为不可扩展。它返回一个布尔值，指示操作是否成功。

它与 `Object.preventExtensions()` 方法相同：

```js
var obj = {
  name: "Eden"
};

console.log(Reflect.isExtensible(obj)); //Output "true"

console.log(Reflect.preventExtensions(obj)); //Output "true"

console.log(Reflect.isExtensible(obj)); //Output "false"
```

## `Reflect.ownKeys(object)` 方法

`Reflect.ownKeys()` 方法返回一个数组，其值表示提供对象属性的键。它忽略了继承的属性。

这里是演示此方法的示例代码：

```js
var obj = {
  a: 1,
  b: 2,
  __proto__: {
    c: 3
  }
};

var keys = Reflect.ownKeys(obj);

console.log(keys.length); //Output "2"
console.log(keys[0]); //Output "a"
console.log(keys[1]); //Output "b"
```

# 摘要

在本章中，我们学习了什么是对象反射，以及如何使用 ES6 Reflect API 进行对象反射。我们通过示例看到了 `Reflect` 对象的各种方法。总的来说，本章介绍了 ES6 Reflect API，用于检查和操作对象的属性。

在下一章中，我们将学习关于 ES6 代理及其用途的内容。
