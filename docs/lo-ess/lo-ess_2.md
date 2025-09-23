# 第二章 使用对象

在 JavaScript 中，一切都是对象。这包括函数、数组和字符串。还有一个普通对象的观念——键值对的字典。后者结构在需要通过键查找值时很有用。换句话说，这是程序员可能想要读取的东西——而不是数组中找到的数值索引。

许多 API 返回 JSON 数据——你经常会发现普通对象。虽然你可以仅使用 JavaScript 对象实现很多功能，但 Lo-Dash 使使用对象进行常见操作变得更加容易。这些函数使日常任务变得不那么无聊，正如你很快就会发现的，你通常可以找到一种更简洁的方法来处理对象。

除了平面对象的访问和操作之外，Lo-Dash 还提供了一些可以应用于代码中任何对象的实用函数。这些函数主要关注验证你正在处理的对象的类型，这是一个使用纯 JavaScript 实现的重复性任务。

在本章中，我们将涵盖以下主题：

+   确定对象类型

+   属性的赋值和访问

+   遍历对象

+   调用方法

+   对象的转换

+   创建和克隆对象

# 确定对象类型

在本节中，我们将探讨在 JavaScript 中通常如何处理类型验证以及 Lo-Dash 中的类型检查函数如何改进这种情况。

## 类型强制转换

当一个对象与另一个对象进行比较时，JavaScript 中会发生类型强制转换。也就是说，我们有一个操作数对象，一个操作符，以及第二个操作数对象。根据要执行的操作，第二个对象可能会被强制转换为与第一个操作数兼容的表示形式。以下是一个示例操作：

```js
true == 1
```

这些显然是代表不同原始类型的不同对象。在松散类型编程的精神下，这个表达式触发了类型强制转换。第一个操作数是一个布尔值，第二个操作数是一个数字。`==` 等于操作符会将 `1` 的布尔表示与 `true` 进行比较。这就是为什么这个表达式总是评估为 `true`。这些值*大致上*是相等的。

你可以使用严格等于操作符来避免类型强制转换。以下表达式将评估为 `false`，因为两个操作数是以其原始形式进行比较的：

```js
true === 1
```

那么为什么关闭类型强制转换呢？这似乎是一个有用的工具。好吧，当你不在乎某些事情时，它是有用的。例如，上一章向您介绍了真值和假值的概念——那些大致上是真或大致上是假的事物。在这里，类型强制转换是你的朋友，因为它捕捉了一组可能值，而不是必须严格检查许多可能值的相等性。简而言之，类型强制转换的存在是为了通过编写更少的代码来使你的生活更轻松。

然而，有时类型强制转换根本帮不上忙，反而可能引入难以追踪的隐蔽错误。例如，让我们用以下表达式对一些值取反：

```js
!false;
!undefined;
!0;
```

这些都评估为`true`，这个事实可能会对我们的代码造成问题。特别是，一个对象可以缺少属性，这会评估为与具有`false`或`0`值的定义属性相同。再次强调，在这些情况下，显式操作和关闭类型强制转换使用严格相等/不等运算符是有帮助的。

那么，所有这些的目的是什么，它与 Lo-Dash 有什么关系呢？前面的表达式只是大量边缘情况和问题的一个小样本，这些问题可能在我们应用程序中不同类型的对象交互时出现。Lo-Dash 通过提供一致的行为来减少这些复杂性。内部，Lo-Dash 必须执行各种丑陋的类型比较和检查，这样我们就不必执行它们。作为额外的奖励，这些实用函数在 Lo-Dash API 中公开。

## 管理函数参数

并非总是清楚我们的函数将以何种类型的参数被调用。我们的函数声明中指定的参数数量也不一定与调用者提供的数量相匹配——这些被称为**可变参数**函数。Lo-Dash 提供的类型检查功能可以更好地准备你的函数来处理可能抛给它们的任何内容。

例如，你可以显式检查每个函数参数，以确定传递给函数的内容。在可选参数的情况下，你可以使用这些函数显式检查是否传递了任何参数，如下面的代码所示：

```js
function hello(name) {
    if (_.isString(name)) {
        return 'hello, ' + name;
    }   
}   

hello('world');
// → "hello, world"

hello();
// → undefined
```

如果`name`参数不是一个字符串，它就不是其他任何东西，换句话说，函数什么都不做。这就是前面代码中在第二次调用`hello()`时的情形。我们与其什么都不做，不如在我们的函数中内置一些其他补救措施，但这取决于我们的函数具体做什么。关键是我们要意识到可能会传递给我们的内容。

在函数调用中，关于是否存在参数的一个变体是参数传递的顺序。你可以忽略最后一个参数，因为它的值是未定义的，而且它本身也是可选的。然而，如果我们的函数接受三个参数，并且第二个是可选的，那会怎样呢？我们不得不在我们的函数内部调整参数及其值。许多库都这样做，Lo-Dash 通过其类型检查函数简化了这些问题，如下面的代码所示：

```js
function hello(greeting, person) {
    if (_.isPlainObject(greeting)) {
        person = greeting;
        greeting = 'hi, ';
    }   
    return greeting + person.name;
}

hello('hello, ', { name: 'Dillan' });
// → "hello, Dillan"

hello({ name: 'Danielle' });
// → "hi, Danielle"
```

在这里，`hello()` 函数期望一个 `greeting` 字符串和一个 `person` 对象。结果证明，`greeting` 参数实际上是可选的，但它是最先的参数。因此，函数检查 `greeting` 是否实际上是一个普通对象，这表明省略了 `greeting` 字符串。然后，你只需确保 `person` 被分配了 `greeting` 的值。

### 注意

所有这些类型检查操作实际上都可以使用 Vanilla JavaScript 完成。另一方面，由于 JavaScript 的神秘类型系统，这样做有一些细微差别。Lo-Dash 只提取那些你不需要自己检查的常见操作，并将它们作为易于理解的函数名暴露出来。

## 算术

如果你曾经在 JavaScript 应用程序中做过算术运算，你知道使用错误的类型作为操作数可能会导致一些真正令人困惑的结果。例如，以下表达式可能或可能不熟悉：

```js
1/0; // Infinity
1+'one'; // NaN
```

问题的关键在于，当这些家伙抬头时，通常不是一个好兆头。这可能是由于我们自己的功能代码（显然是在开发中且未投入生产）导致的，或者仅仅是因为我们的功能被调用时使用了错误的数据。在任一情况下，我们都需要准备好去排查发生的问题。我们通过为我们的算术操作提供一个安全网来实现这一点，如下所示：

```js
var operand1 = 1/0,
    operand2 = NaN,
    results = [];

_.forEach([ operand1, operand2 ], function(op) {
    if (_.isFinite(op)) {
        results.push('operand 1 is finite');
    } else {
        if (!_.isNumber(op) || _.isNaN(op)) {
            results.push(op.toString() + ' is not a number');
        } else {
            results.push('Infinity cannot be used here');
        }
    }
});

return results;
// → 
// [
//   "Infinity cannot be used here",
//   "NaN is not a number"
// ]
```

这段代码遍历操作数，并使用 Lo-Dash 的 `isFinite()` 函数检查每个操作数是否是有限的。这个函数可以被视为一个万用工具；如果这个测试通过，那么你通常可以使用操作数进行算术运算。如果 `isFinite()` 失败，则执行 `else` 代码，这是尝试找出失败原因的一种尝试。如果它不是一个数字，那么显然它不是有限的。这包括像 `true`、`String` 或 `null` 这样的值。另一方面，如果它是一个非有限的数字，我们知道我们正在处理无穷大。

### 注意

`NaN` 实际上是一个数字——这是 JavaScript 类型系统的最佳表现。这就是为什么在前面代码中的 `if` 语句有一个对 `!_.isNumber()` 或 `_.isNaN()` 的检查。

## 可调用对象

如果你曾经尝试调用一个不是函数的东西，你可能已经看到了类似 `TypeError: undefined is not a function` 的错误消息。在这种情况下，属性或变量根本不存在。然而，如果我们尝试调用一个存在但不可调用的对象，我们也会得到类似的错误消息。

有时这种错误是期望的，因为我们的代码正在尝试调用一个不是函数的东西。解决方案：我们去修复它。记住，JavaScript 是动态类型的，根据我们的应用程序是如何设计的，可能存在需要在我们尝试调用它之前显式检查某物是否是函数的情况，如下面的示例所示：

```js
var object = {
    a: function() { return 'ret'; },
    b: []
};

_.isFunction(object.a) && object.a();
// → "ret" 

_.isFunction(object.b) && object.b();
// → false
```

第一个属性 `a` 是一个函数，因此调用 `isFunction()` 发出的检查通过，函数被调用。另一方面，`b` 属性是一个数组，不可调用。所以在那里没有发生任何事情。

# 分配和访问属性

使用值创建新的 JavaScript 对象是一个简单的任务。繁琐的部分在于当我们需要将一个对象的内容合并到另一个对象中，或者当我们需要确保新对象有默认值时。在对象中定位值并验证键是否存在实际上需要大量的代码，如果不是 Lo-Dash 工具帮助我们完成这些活动的话。 

## 扩展对象

在 JavaScript 库中，一个常见的模式是通过其他对象扩展对象以分配属性值。这可以通过逐个语句将一个属性分配给对象来实现。这种方法的麻烦在于你需要提前确切地知道哪些属性将被分配到目标对象中。考虑当新值来自函数参数时。提前知道所有可能的属性值是不切实际的。直接使用传递的源并扩展目标更简单。这就是为什么你会在 Lo-Dash 中找到这个模式，包括 Lo-Dash。这些工具被公开，以便你在你的应用程序中遵循相同的模式。以下是一个示例：

```js
var object = { 
    name: 'Jeremy',
    age: 42
};

_.assign(object, {
    occupation: 'Programmer'
});

// →
// {
//   name: "Jeremy",
//   age: 42,
//   occupation: "Programmer"
// }
```

在这里，目标对象是 `object` 变量，它被分配了 `occupation` 属性。实际上，使用 `assign()`，我们必须小心，因为它不关注现有属性。任何传入的源都将覆盖它们，如下面的代码所示：

```js
var object1 = { 
        name: 'Jenny',
        age: 27
    },  
    object2 = { 
        age: 31
    },  
    object3 = { 
        occupation: 'Neurologist'
    };

_.assign(object1, object2, object3);
// →
// {
//   name: "Jenny",
//   age": 31,
//   occupation: "Neurologist"
// }
```

两个对象被分配到目标 `object1`。`assign()` 函数接受你需要的任何参数——它们按从左到右的顺序依次连接，并覆盖前面的属性。例如，在前面提到的代码中，没有新的对象被分配以覆盖 `name` 属性。然而，第二个对象覆盖了 `age` 属性。最终对象有一个全新的属性，并简单地添加到目标中。请注意，最终对象是 `object1`，它被就地修改。

### 注意

Lo-Dash 为其一些函数使用了别名。例如，`extend()` 只是 `assign()` 的别名；它做的是完全相同的事情。这是一个个人喜好问题，哪个被使用。你更喜欢将一个对象视为被 *分配* 到另一个对象，还是将一个对象视为 *扩展* 另一个对象？

到目前为止，我们处理了相互覆盖的简单属性，但更复杂的属性，如对象和数组，怎么办？我们是否希望这些值合并在一起而不是完全被覆盖？以下是一个展示这一点的示例：

```js
var object1 = { 
        states: { running: 'poweroff' },
        names: [ 'CentOS', 'REHL' ]
    },  
    object2 = { 
        states: { off: 'poweron' },
        names: [ 'Ubuntu', 'Debian' ]
    };

_.merge(object1, object2, function(dest, src) {
    if (_.isArray(dest) && _.isArray(src)) {
        return dest.concat(src);
    }   
});
// →
// {
//   states: {
//     running: "poweroff",
//     off: "poweron"
//   },
//   names: [
//     "CentOS",
//     "REHL",
//     "Ubuntu",
//     "Debian"
//   ]
// }
```

`merge()` 函数在覆盖属性之前会递归地检查对象的属性，这与 `assign()` 函数不同。在其他方面，这两个函数是相似的，我们正在将一个或多个对象的属性复制到单个目标对象中。注意 `states` 属性——它没有被覆盖。相反，`merge()` 将检查两个对象并将它们合并在一起。其他在目标中已经存在的、具有相同属性名的类型将被简单地覆盖。这包括数组。

注意，我们能够将你自己的回调函数传递给 `merge()`。这个函数决定了如何合并属性。代码检查目标属性和源属性是否为数组。如果是，将源数组合并到目标中，否则它将被覆盖。回调将忽略任何不是数组的值。

## 新对象的默认值

在 JavaScript 编程中，一个常见的做法是通过参数自定义属性。也就是说，当我们创建一个新的对象实例时，我们可能提供一个在对象被使用的上下文中独特的参数。然而，为了有效地使用此模式，我们必须在调用者没有提供任何值时提供一些默认值。有几种方法可以做到这一点，但 Lo-Dash 提供了一个函数可以处理绝大多数情况，如下面的代码所示：

```js
var object = { 
    name: 'George'
};

_.defaults(object, {
    name: '', 
    age: 0,
    occupation: ''
});
// →
// {
//   name: "George",
//   age": 0,
//   occupation": ""
// }
```

如我们所见，`name` 属性没有被默认值覆盖。其他两个默认值，`age` 和 `occupation`，被分配给对象，因为它们是未定义的。如果属性存在任何其他值，`defaults()` 将使用该值，而不是默认值。

### 注意

`defaults()` 函数实际上使用了 `assign()` 函数。它只是传递 `assign()`，一个自定义默认值分配方式的回调函数。具体来说，是通过查找未定义的值。

## 查找键和值

在纯 JavaScript 中，对象访问属性值使用与数组相同的语法。也就是说，使用方括号表示法，但通常使用可读性强的字符串，而不是数值索引。然而，与数值索引和数组存在相同问题的还有对象和键。仅仅因为键是字符串，并不意味着我们知道哪些键是可用的。有时，我们必须搜索对象以找到我们正在寻找的键。

我们使用 `findKey()` 函数来定位第一个对象属性，该属性返回的回调函数为真值：

```js
var object = { 
    name: 'Gene',
    age: 43, 
    occupation: 'System Administrator'
};

_.findKey(object, function(value) {
    return value === 'Gene';
});
// → "name"
```

这里，结果是 `name`；因为它是我们 `findKey()` 回调返回 `true` 的第一个属性值。奇怪的是，pluck 风格的简写方式并不像你想象的那样工作。调用 `_.findKey(object, 'Gene')` 什么也没找到。那是因为它将每个属性值都视为嵌套对象。以下是一个如何使用此函数的 where 风格简写方式的示例：

```js
var object = { 
    programmers: {
        Keith: 'C',
        Marilyn: 'JavaScript'
    },  
    designers: {
        Lori: 'CSS',
        Marilyn: 'HTML'
    }   
};

_.findKey(object, { Marilyn: 'JavaScript' });
// → "programmers"
```

正如我们所见，它将每个属性值视为另一个对象；这些是 `where` 条件所测试的值。我们还可以使用以下代码找到具有数组值的对象属性的键：

```js
var object = { 
    Maria: [
        'Python',
        'Lisp',
        'Go'
    ],  
    Douglas: [
        'CSS',
        'Clojure',
        'Haskell'
    ]   
}; 

var lang = 'Lisp';

_.findKey(object, function(value) {
    if (_.isArray(value)) {
        return _.contains(value, lang);
    } else {
        return value === lang;
    }   
});
// → "Maria"
```

传递给 `findKey()` 的回调函数检查属性值是否为数组。如果是，它会检查值是否存在于其中。否则，它将只执行严格的值比较。由于 `search` 项存在于第一个属性值中，所以结果键将是 `Maria`。

### 注意

`findKey()` 函数有一个互补函数叫做 `findLastKey()`。这个函数简单地朝相反方向搜索。它有点像集合中的 `find()` 和 `findLast()`。区别在于数组中顺序是保留的。在对象中，你正在处理一个无序的键值对集合。由于顺序从未得到保证，`findLastKey()` 的实用性有限。

我们可能会发现自己正在处理一个对象，但我们不一定需要使用键。记住，对象也是集合，所以你仍然可以使用 `find()` 或 `where()`，就像在数组上使用一样，如下面的示例所示：

```js
var object = {
    8490: {
        first: 'Arthur',
        last: 'Evans',
        enabled: false
    },  
    7035: {
        first: 'Shirley',
        last: 'Rivera',
        enabled: false
    },  
    4818: {
        first: 'William',
        last: 'Howard',
        enabled: true
    }   
};

_.find(object, 'enabled');
// → 
// {
//   first: "William",
//   last: "Howard",
//   enabled: true
// }

_.where(object, { last: 'Rivera' });
// → 
// [
//   {
//     first: "Shirley",
//     last: "Rivera",
//     enabled: false
//   }
// ]
```

这些函数将每个 `object` 属性值视为数组的一个元素，忽略键。接下来，我们将探讨在简单集合简写不足以满足需求的情况下如何遍历对象。

# 遍历对象

当我们需要遍历对象的属性以实现组件的行为时，Lo-Dash 有几个函数非常有用。我们将从探索一些基本迭代开始。然后，我们将探讨如何遍历继承的对象属性，接着查看键和值以及遍历它们的简单方法。

## 基本遍历

正如我们在上一章中看到的，对象可以像数组一样迭代——它们都是集合。虽然这样做的方式略有不同，但 Lo-Dash 通过统一的函数 API 抽象掉了这些差异，如下面的代码所示：

```js
var object = { 
    name: 'Vince',
    age: 42, 
    occupation: 'Architect'
}, result = [];

_.forOwn(object, function(value, key) {
    result.push(key + ': ' + value);
});
// → 
// [
//   "name: Vince",
//   "age: 42",
//   "occupation: Architect"
// ]
```

上述代码看起来有些熟悉。它就像 `forEach()` 函数一样。传递给回调函数的第二个参数不是索引，而是属性键。

### 注意

当应用于对象时，`_.forOwn()` 和 `_.forEach()` 函数的行为相同。这两个函数共享同一个基础函数，用于遍历集合。Lo-Dash 有几个足够通用的基础函数，可以服务于许多目的。虽然这些函数不是作为公共 API 的一部分公开的，但它们使得公开的函数更小、更易于理解。

## 包含继承属性

对象迭代只包括 *自有* 属性。也就是说，直接在对象上定义的属性，而不是在 **原型链** 更高的地方定义的属性。我们可以使用 `hasOwnProperty()` 方法来测试所讨论的属性是否是自有属性。我们传递这个方法要查找的属性名称，如果该属性定义在这个属性上而不是原型链的更高处，它将返回 `true`。

### 注意

如果“原型链”这个术语听起来很陌生，你可能想了解一下它们是什么以及它们是如何工作的。JavaScript 对象是原型的，所以如果你是 JavaScript 程序员，理解这个概念很重要。这个话题远远超出了本书的范围，但网上有数百个关于原型的优秀资源。对于本书，你不需要完全理解这个主题，只需要这一节的内容。

Lo-Dash 有另一个对象迭代函数，称为 `forIn()`。这个函数能够遍历自有属性，以及通过原型链继承的属性。以下是一个示例：

```js
function Person() {
    this.full = function() {
        return this.first + ' ' + this.last;
    };  
}   

function Employee(first, last, occupation) {
    this.first = first;
    this.last = last;
    this.occupation = occupation;
}   

Employee.prototype = new Person();

var employee = new Employee('Theo', 'Cruz', 'Programmer'),
    resultOwn = [], 
    resultIn = [];

_.forOwn(employee, function(value, key) {
    resultOwn.push(key);
});
// → [ "first", "last", "occupation" ]

_.forIn(employee, function(value, key) {
    resultIn.push(key);)
});
// → [ "first", "last", "occupation", "full" ]
```

这段代码同时使用了对象迭代的形式，`forOwn()` 后跟 `forIn()`。两者之间的区别在于 `full` 键，它只出现在 `forIn()` 生成的结果中。这是因为它在 `Person` 对象中定义，而 `Person` 对象是 `Employee` 的原型。

## 键和值

之前，我们一直在使用直接遍历对象键和值的 Lo-Dash 函数。这是直接的方法。然而，如果我们已经编写了一些期望键或值数组的代码，会怎样呢？有一种间接的方法可以遍历对象，这涉及到将对象的键或值作为数组获取。然后你遍历这些数组。

例如，以下是一些遍历对象键的代码：

```js
var object = { 
    occupation: 'Optometrist',
    last: 'Lynch',
    first: 'Shari'
};

_.sortBy(_.keys(object));
;
// → [ "first", "last", "occupation" ]
```

上述结果是 `keys()` 函数构建的字符串数组。我们使用 `sortBy()` 函数作为一种快速而简单的手段来排序数组。然后按照顺序将每个属性键推入 `result` 数组。让我们构建这个示例，并使用它作为获取对象属性值的有序访问手段：

```js
var object = { 
    occupation: 'Optometrist',
    last: 'Lynch',
    first: 'Shari'
};

return _.at(object, _.sortBy(_.keys(object)));
// → [ "Shari", "Lynch", "Optometrist" ]
```

这段代码采取了一些捷径。然而，这不就是编写好代码的全部意义吗？我们不是使用 `forEach()` 函数在排序后迭代键，而是直接将它们传递给 `at()` 函数。这个函数接受一个键或索引数组，并将按顺序为我们查找值。上述结果是按键排序的属性值数组。

### 注意

`keys()` 函数在用于遍历对象的 `forOwn()` 函数中起着至关重要的作用。这个函数用于获取对象键，然后迭代键，查找对象值。再次强调，一些外部 Lo-Dash 函数在内部起着至关重要的作用。

当你真的不需要键名时，`values()`与`keys()`互补。例如，为了构建一个包含对象值的数组，你可以使用以下代码：

```js
var object = { 
    first: 'Hue',
    last: 'Burton',
    occupation: 'Horticulturalist'
};

_.values(object);
// → [ "Hue", "Burton", "Horticulturalist" ]
```

从这个点开始，我们有一个值数组可以工作。键被完全忽略。例如，如果我们想根据值的具体属性而不是键来对`object`属性值进行排序，就像我们之前看到的那样，可以使用以下代码：

```js
var object = { 
    Angular: { name: 'Patrick' },
    Ember: { name: 'Jane' },
    Backbone: { name: 'George' }
};

_.sortBy(_.values(object), 'name');
// → 
// [
//   { name: "George" },
//   { name: "Jane" },
//   { name: "Patrick" }
// ]
```

这就像我们只是通过截断键将`object`转换成一个数组。实际上，用`toArray()`替换`values()`会产生完全相同的结果。在底层，如果传递给`toArray()`的是一个对象，它实际上会调用`values()`。

# 调用方法

对象不仅仅包含静态属性值——其中一些值是可调用的函数。作为对象属性的函数通常被称为方法，因为它们通常与它们所属的对象的封装状态进行交互。有时，对象只是方便地在代码中分配和传递函数的一种方式。最终，它们只是被分配给属性值的函数，而 Lo-Dash 有一些函数可以帮助找到并调用它们。

## 获取结果

当我们不确定给定的属性名是函数还是其他类型时，我们可以使用`result()`函数。这可以极大地简化我们的代码，因为我们不需要编写检查属性是否应该像常规静态属性值那样访问或是否需要调用的代码。`result()`函数的使用在以下代码中显示：

```js
var object1 = { 
        name: 'Brian'
    },  
    object2 = { 
        name: function() {
            return 'Brian';
        }   
    },  
    object3 = {};

_.result(object1, 'name', 'Brian');
// → "Brian" 

_.result(object2, 'name', 'Brian');
// → "Brian" 

_.result(object3, 'name', 'Brian');
// → "Brian" 
```

我们可以看到结果始终是`Brian`，并且`result()`在所有三个对象上的调用都是相同的。然而，结果是`Brian`有三个不同的原因。第一个对象有一个字符串值为`Brian`的`name`属性。第二个对象有一个返回字符串`Brian`的函数值的`name`属性。第三个对象没有`name`属性，因此使用了默认的`Brian`值。通过你非常少的努力，使用`result()`函数的对象促进了对象属性访问的一致性。

适度地使用`result()`，否则我们会在代码中对其频繁使用而感到困惑。在直接属性访问或直接方法调用产生更简洁代码的情况下，走这条路。在属性访问产生一致结果呈现问题时，这些情况应该很少，`result()`是我们的朋友。

## 查找方法

在我们调用方法之前，如果方法不存在，我们可能想要执行比简单默认值更复杂的逻辑。例如，我们可能知道某些对象上有`name()`方法，但其他对象上没有。我们确定知道的是，没有带有简单值的`name`属性，所以`result()`函数在这里没有帮助。

`functions()`函数会遍历一个对象，并返回一个数组，其中包含值是函数的键，如下面的代码所示：

```js
function Person(first, last) {
    this.first = first;
    this.last = last;
}

Person.prototype.name = function() {
    return this.first + ' ' + this.last;
};

_.functions(new Person('Teresa', 'Collins'));
// → [ "name" ]
```

注意，`name()`方法被定义为人的原型的一部分，而不是直接在`Person`实例上。如果我们这样考虑，这是有道理的。如果这个方法在原型链的更高处存在，它仍然是可以调用的，使用当前实例作为其上下文。因此，我们希望那些方法名出现在结果数组中，这正是这里发生的情况。

# 转换对象

有时候，我们在实现一个功能，而我们正在处理的对象根本不适合这个用途——你需要将其转换成一个更适合我们需求的结构。Lo-Dash 附带了一些函数可以帮助我们做到这一点。我们可以从对象中创建一个数组的数组，我们可以选择我们想要与之工作的对象属性，我们还可以通过反转键和值来翻转对象。

## 使用成对

`pairs()`函数接受一个对象参数，并生成一个数组，其中每个元素本身也是一个数组，包含键和值。在某些情况下，这种结构可能会更加方便。以下是一个例子：

```js
function format(label, value) {
    return label + ': ' + value;    
}   
var object = { 
    first: 'Katherine',
    last: 'Bailey',
    age: 33
}, result = '';

_.forEach(_.pairs(object), function(pair) {
    result += format.apply(null, pair) + '\n';
});
// → "first: Katherine\nlast: Bailey\nage: 33\n" 
```

这段代码遍历`object`，但在这样做之前，它调用了`pairs()`函数。这把对象转换成了一个数组，因此`forEach()`的回调函数接收这个数组的一个元素。`pair`参数是一个数组，第一个元素是键，第二个是值。使用这个键值对，我们可以对`format()`函数调用`apply()`。

这意味着如果我们有一个通用的回调函数，它使用类似`format()`这样的函数，我们不需要传递特定的参数。正如这里所示，当传递`pair`数组时，你可以调用`apply()`。

## 选择和省略属性

有时候，并不是每个对象属性都是必要的。在扩展对象时，实际上这可能是一项有害的练习——添加不必要的属性。相反，你可以使用`pick()`函数来选择你需要的属性：

```js
var object1 = { 
        name: 'Kevin Moore',
        occupation: 'Programmer'
    },  
    object2 = { 
        specialty: 'Python',
        employer: 'Acme'
    };

_.assign(object1, _.pick(object2, 'specialty'));
// →
// {
//   name: "Kevin Moore",
//   occupation: "Programmer",
//   specialty: "Python"
// }
```

在这个例子中，第二个对象只有一个我们感兴趣的属性，即`specialty`。碰巧的是，只有一个属性被删除，即`employer`。然而，我们在这里所做的，就是只选择我们需要用来扩展现有对象的内容，从而排除了任何不想要的属性在后续造成问题的可能性。

另一方面，我们可能确切地知道我们不想从一个对象中得到什么。`pick()`的补充是`omit()`，它从对象中排除指定的属性，以下是一个例子：

```js
var object1 = { 
        name: 'Kevin Moore',
        occupation: 'Programmer'
    },  
    object2 = { 
        specialty: 'Python',
        employer: 'Acme'
    };

_.assign(object1, _.omit(object2, 'employer'));
// →
// {
//   name: "Kevin Moore",
//   occupation: "Programmer",
//   specialty: "Python"
// }
```

这段代码是 `pick()` 示例的反面。我们不是使用 `pick()` 来指定要包含的内容，而是使用 `omit()` 来指定要从赋值中排除的内容。我们使用哪一个取决于我们对对象和哪些属性有价值所拥有的必要知识。

除了提供我们想要包含或排除的属性名称之外，我们还可以提供自定义逻辑，以回调函数的形式做出决策，如下面的代码所示：

```js
var object = { 
    name: 'Lois Long',
    age: 0,
    occupation: null
};

_.omit(object, function(value) {
    return !(!_.isBoolean(value) && value);
});
// → { name: "Lois Long" }
```

这段代码与 `compact()` 在集合上的工作方式相同。我们的回调函数应用于每个 `object` 属性值，如果它返回 `true`，则该值将从结果对象中省略。在这里，我们省略了非真值，除了布尔类型。

## 反转键和值

我们的应用可能定义了一个使用 `keys()` 或 `values()` 来处理对象的函数。然而，我们可能会遇到想要该函数反向工作的情形。也就是说，如果函数使用 `keys()`，我们希望它使用 `values()`。如果它使用 `values()`，我们希望它使用 `keys()`。

而不是改变我们到处使用且已知稳定的函数，我们可以在传递之前简单地使用 Lo-Dash 的 `invert()` 函数来反转对象：

```js
function sortValues(object) {
    return _.values(object).sort();
}   

var object1 = { 
        first: 'Mathew',
        last: 'Johnson'
    },  
    object2 = { 
        first: 'Melissa',
        last: 'Willians'
    };

sortValues(object1);
// → [ "Johnson", "Mathew" ]

sortValues(_.invert(object2));
// → [ "first", "last" ]
```

`sortValues()` 函数相当简单。它接受一个 `object` 参数，使用 `values()` 函数构建属性值数组，然后返回排序后的数组。如果我们想出于某种原因在对象键上重用 `sortValues()`，我们只需在传递之前对对象使用 `invert()`。这使得对象的键成为值。因此，当 `sortValues()` 调用 `values()` 函数时，它实际上获取的是键。

# 创建和克隆对象

本章的最后一个主题是创建和克隆 JavaScript 对象。在日常工作中，我们通常不需要过多思考就完成了对象的创建或克隆。`new` 关键字或对象字面量记法已经足够满足我们的需求。很少有必要去克隆对象。然而，当需要时，Lo-Dash 提供了处理这两种情况所需的工具。

## 创建对象

`create()` 函数帮助我们缩小了函数式和面向对象范式之间的差距。它允许我们在对象中利用一些关键的函数式 JavaScript 组件。特别是，当涉及到创建新对象时指定原型。

这可能听起来不是什么大事，但它可以带来一些有趣、复杂的黑客技巧。假设我们使用字面量记法定义了一个对象集合。这些对象具有简单的字符串属性值。再假设我们有一个通过方法定义了一些行为的类。使用 `create()` 函数，我们可以直接将属性值传递给该类的新实例，以便利用其行为，如下面的代码所示：

```js
function Person() {}
Person.prototype.name = function() {
    return this.first + ' ' + this.last;
};  

var collection = [ 
        { first: 'Jean', last: 'Flores' },
        { first: 'Edward', last: 'Baker' },
        { first: 'Jennifer', last: 'Walker' }
    ],  
    people = []; 

_.forEach(collection, function(item) {
    people.push(_.create(Person.prototype, item));
}); 

_.invoke(people, 'name');
// → [ "Jean Flores", "Edward Baker", "Jennifer Walker" ]
```

在前面的例子中，将`Person`对象视为一个合约或接口是有帮助的。然后我们使用`create()`函数将这个合约绑定到集合中的每个对象。现在，集合中的每个对象都有一个`name()`方法，我们通过使用`invoke()`函数生成一个名称数组来证明这一点，该函数将调用集合中每个项目的给定方法名称。

## 克隆对象

重复创建相同的对象实例可能导致代码重复。特别是，如果我们使用对象字面量语法。一个替代方案是扩展一个新对象以包含新属性，但如果我们要复制的东西不是一个普通对象，这可能会出现问题。Lo-Dash 有一个`clone()`函数，足够灵活，可以复制几乎所有东西，包括嵌套对象。这种灵活性是以性能成本为代价的，所以请明智地使用它。以下是如何使用`clone()`函数的示例：

```js
function Person(first, last) {
    this.first = first;
    this.last = last;
}

var object1 = { 
        first: 'Laura',
        last: 'Gray'
    },  
    object2 = new Person('Bruce', 'Price'),
    clone1 = _.clone(object1),
    clone2 = _.clone(object2);

clone1.first === 'Laura';
// → true

clone2.first === 'Bruce' && clone2 instanceof Person;
// → false
```

`object1`变量包含一个普通对象，而`object2`变量包含`Person`类的一个实例，但它们本质上是一样的。它们都有`first`和`last`属性。`clone1`和`clone2`变量包含它们各自的克隆。有趣的是，我们接下来执行的断言。第一个断言通过，因为我们只是验证原始`name`属性中的字符串仍然存在于克隆属性中。第二个断言失败，并不是因为克隆的`first`属性不等于`Bruce`。它失败是因为`clone2`不是`Person`的实例。相反，它是一个`Object`的实例，因为`clone()`函数没有采取设置适当构造函数属性等必要步骤。

除了克隆的对象不是`Person`类的实例之外，它几乎与原始对象和属性访问等相同。它应该仍然像在普通对象上那样工作。`clone()`的重点实际上是复制一个普通对象，以便断开与原始对象的引用。然后它可以被操作而不会触及源。

# 摘要

本章向我们介绍了如何使用 Lo-Dash 函数来执行复杂的对象交互。我们从那些让对 JavaScript 类型系统进行推理变得不那么痛苦的工具开始。接下来，我们看到了在各个上下文中如何访问和分配对象属性。遍历对象属性是下一个话题，Lo-Dash 提供了大量的工具供你使用。特别是，如果我们只想遍历对象的键或值，而不需要编写大量的样板代码，那么 Lo-Dash 函数就帮我们处理了这一点。当我们向 Lo-Dash 函数传递一个对象时，它会产生一个新的结构，这就是所谓的转换。比如当我们寻找一组键值对时。选择或省略属性也是一个非常直接的活动。我们通过查看对象创建和克隆功能来结束本章。这些功能在我们需要稍微调整规则以满足应用程序需求时非常有帮助。

通过前两章，你可能已经注意到定义和使用了大量函数。这不是偶然的——JavaScript 将函数视为一等公民，Lo-Dash 也是如此。函数将是下一章的重点。
