# 第六章 应用构建块

上一章讨论了 Lo-Dash 的关键功能——包装值和执行链式函数调用。这种编程风格的一个好处是能够构建更大的功能单元，这些单元是通用和可移植的。我们在上一章处理包装实例时看到了通用性和可移植性的影子。本章的目标是实现这些想法。编写一个应用程序不仅仅是抛出过滤器和映射，以获得你需要的数据。如果你反复写相同的内容，你的代码很快就会变得混乱。

在本章中，我们将学习如何编写内部使用 Lo-Dash API 的通用函数。我们还将通过探索它们如何像拼图一样组合在一起，以及最终为你的应用程序提供一个坚实的基础，将链式函数调用进一步推进。当你为你的应用程序编写的函数属于 Lo-Dash 基础设施时，就到了这一点。也就是说，你需要将你自己的通用代码与 Lo-Dash API 混合。我们也将处理这一点。

在本章中，我们将涵盖以下主题：

+   通用函数

+   通用包装器和链

+   函数组合

+   创建混入

# 通用函数

创建通用函数可以在我们代码的大小和可理解性方面产生重大差异。通用函数在多个上下文中都很有用。它与应用程序松散耦合。这就是高级构建块的全部内容；无论我们是在使用函数式编程模型、更面向对象的方法，还是两者的混合，关键在于通用组件。与编程的许多其他方面一样，Lo-Dash 提供了许多构建通用组件的途径。我们将在本章的整个过程中探讨其中许多途径。让我们从查看不那么通用的函数以及它们与更灵活的表亲如何比较开始。

## 特定函数

一个函数是否适合单一目的，以及它是否只适合单一目的，并不总是那么明确。根据你的视角，存在不同程度的特定性。考虑以下函数：

```js
var collection = [
    { name: 'Ronnie', age: 43 },
    { name: 'Ben', age: 19 },
    { name: 'Sharon', age: 25 },
    { name: 'Melissa', age: 29 }
];

function collectionNames() {
    return _.map(collection, 'name');
}

function indirectionNames(coll, prop) {
    return _.map(coll, prop);
}

function genericCollNames(coll) {
    return _.map(coll, 'name');
}

function genericPropNames(prop) {
    return _.map(collection, prop);
}

collectionNames();
indirectionNames(collection, 'name');
genericCollNames(collection);

genericPropNames('name');
// → [ "Ronnie", "Ben", "Sharon", "Melissa" ]
```

这四个函数都产生相同的结果，但它们在我们的应用中的实现方式各有其独特的后果。我们可以评估每个函数的两个通用属性。首先，我们看看正在转换的集合——主要操作数。此外，还有传递的次要参数，它们会影响结果。

`collectionNames()`函数相当具体，因为它期望在其作用域中有一个`collection`变量，并且将`name`参数硬编码为传递给`map()`的参数。`indirectionNames()`函数则相反——它是完全通用的，因为它接受集合和属性参数，但它也是完全无意义的，因为它只是一个代理，我们直接调用`map()`也行。`genericCollNames()`函数很有趣；我们用这个函数映射的集合是通用的，因为它作为参数传递，而`name`参数是硬编码的。最后，`genericPropNames()`函数在硬编码集合时使用了一个通用参数。

### 注意

在定义函数时，要记住考虑每个极端——从间接到完全硬编码——。这两种极端都几乎不值得追求，而中间地带则是我们应努力达到的目标。至于你硬编码什么，保持什么通用，每个都有独特的权衡，取决于你正在构建的内容。随着应用程序的发展，你经常会发现自己需要调整这些设置。

## 通用函数参数

JavaScript 允许我们在定义函数时有一定的自由度。并非所有参数都需要在事先静态声明，就像在其他语言中那样。`arguments`对象可以帮助我们，尤其是在我们试图保持通用性时。例如，某些调用者可能不会传递所有参数。这没关系，我们的函数可以应对这种情况，我们可以利用这种能力来定义更好的函数，这些函数可以更通用地与 Lo-Dash API 交互，如下面的代码所示：

```js
function insert(coll, callback) {
    var toInsert;

    if (_.isFunction(callback)) {
        toInsert = _.slice(arguments, 2);
    } else {
        toInsert = _.slice(arguments, 1);
        callback = _.identity;
    }

    _.each(toInsert, function(item) {
        coll.splice(_.sortedIndex(coll, item, callback), 0, item);
    });

    return coll;

}

var collection = _.range(1, 11);

insert(collection, 8.4);
// → [ 1, 2, 3, 4, 5, 6, 7, 8, 8.4, 9, 10 ]

insert(collection, 1.1, 6.9);
// → [ 1, 1.1, 2, 3, 4, 5, 6, 6.9, 7, 8, 8.4, 9, 10 ]

insert(collection, 4, 100);
// → [ 1, 1.1, 2, 3, 4, 4, 5, 6, 6.9, 7, 8, 8.4, 9, 10, 100 ]
```

`insert()`函数接受`coll`和`callback`参数。集合总是必需的，但回调是可选的。如果没有提供回调，它默认为`identity()`函数。

在这里还有一些额外的技巧，因为函数中提供的任何其他参数都是要插入到集合中的目标。我们使用`slice()`函数将这些参数放入`toInsert`变量中，并且根据是否提供了回调函数，我们以不同的方式切片它们。然后，只需遍历每个要插入的参数值，并将我们的回调传递给`sortedIndex()`函数。

### 注意

将回调值设置为`identity()`在这里并不是严格必要的。这是大多数接受回调的 Lo-Dash 函数的默认行为。明确指定也不会有害，尤其是如果我们不想使用相同的默认函数。

## 使用部分应用

解决出现的一般参数问题的便捷模式是使用部分应用，即使用`partial()`函数部分应用函数参数。这让我们可以在运行时构建可以重复使用的函数，而无需始终应用相同的参数。有时甚至无法提供函数参数。以下是一个使用部分应用的示例：

```js
var flattenProp = _.compose(_.flatten, _.prop),
    skills = _.partialRight(flattenProp, 'skills'),
    names = _.partialRight(flattenProp, 'name');

var collection = [
    { name: 'Danielle', skills: [ 'CSS', 'HTML', 'HTTP' ] },
    { name: 'Candice', skills: [ 'Lo-Dash', 'jQuery' ] },
    { name: 'Larry', skills: [ 'KineticJS', 'Jasmine' ] },
    { name: 'Norman', skills: [ 'Grunt', 'Require' ] }
];

_.contains(skills(collection), 'Lo-Dash');
// → true
_.contains(names(collection), 'Candice');
// → true
```

我们的`flattenProp()`函数是`flatten()`和`prop()`的组合。返回的结果是一个扁平化的数组。因此，如果这些属性值本身是数组，它们就会被添加到单个数组中。

在我们的应用中，当使用的数据模型在实体之间共享许多属性时，没有必要总是提供需要扁平化的属性名称。这正是使用部分函数的完美案例。记住，部分函数并非完全静态——它们最终还是会返回函数。我们的代码定义了两个带有预应用`prop`参数的部分函数。稍后，我们可以使用这个函数与特定的集合一起使用。

### 注意

从通用函数创建部分函数是一种函数组合的形式，是构建高级应用组件的关键工具。

## 通用回调

设计一个在代码中手动调用的函数是一回事。然而，回调对于 Lo-Dash 来说至关重要。因此，始终考虑我们的函数可能被用作回调的事实是值得的，如下面的代码所示：

```js
var YEAR_MILLISECONDS = 31560000000;

function validItem(item) {       
    return item.age > 21 &&
        _.isString(item.first) &&
        _.isString(item.last);
}

function computed(item) {
    return _.extend({
        name: _.result(item, 'first', '') + ' ' +
            _.result(item, 'last', ''),
        yob: new Date(new Date() - (YEAR_MILLISECONDS * item.age))
            .getFullYear()
    }, item);
}

var invalidItem = _.negate(validItem);

    { first: 'Roderick', last: 'Campbell', age: 56 },
    { first: 'Monica', last: 'Salazar', age: 38 },
    { first: 'Ross', last: 'Andrews', age: 45 },
    { first: 'Martha', age: 51 }
];

_.every(collection, validItem);
// → false

_.filter(collection, validItem);
// →
// [
//   { first: "Roderick", last: "Campbell", age: 56 },
//   { first: "Monica", last: "Salazar", age: 38 },
//   { first: "Ross", last: "Andrews", age: 45 }
// ]

_.find(collection, invalidItem);
// → { first: "Martha", age: 51 }

_.map(collection, computed);
// →
// [
//   {
//     name: "Roderick Campbell",
//     yob: 1958,
//     first: "Roderick",
//     last: "Campbell",
//     age: 56
//   }, {
//     name: "Monica Salazar",
//     yob: 1976,
//     first: "Monica",
//     last: "Salazar",
//     age: 38
//   }, {
//     name: "Ross Andrews",
//     yob: 1969,
//     first: "Ross",
//     last: "Andrews",
//     age: 45
//   }, {
//     name: "Martha ",
//     yob: 1963,
//     first: "Martha",
//     age: 51 }]
```

在此代码中定义的第一个回调是`validItem()`，这是一个极其有用的函数，因为在很多情况下，你可能只对有效的项目感兴趣。这个函数接受一个通用的`item`参数，如果该参数满足某些条件，则返回`true`。这是对集合迭代应用回调的理想格式。第二个回调是`computed()`，它也接受一个通用的`item`参数。这个回调在映射场景中很有用，因为它返回一个包含计算属性的项目的扩展版本。这里还有一个第三个回调——`invalidItem()`。这是`validItem()`函数的逆函数，我们可以使用`negate()`来创建它。

### 注意

你可能已经注意到，我们很多回调函数都使用`item`作为第一个命名参数。这是一个好的实践，因为它给代码的读者一个很好的提示，即给定的函数可能被用作某个地方的回调。

# 通用包装器和链

在掌握了通用函数之后，我们现在可以将注意力转向 Lo-Dash 包装器实例，并创建通用函数调用链。链在陷入困境且需要快速摆脱复杂编程情况时很有用，但它们在通用意义上也很有用。也就是说，你可以组合足够通用的功能链，以便在多种上下文中应用。

## 通用过滤器

让我们先看看通用过滤器以及它们如何在我们的函数中发挥作用。过滤器特别适合链式函数调用，因为它们可以通过在先前的过滤器之后应用另一个过滤器来连接起来。在过滤器的末尾通常会有一些排序或其他约束，例如限制返回的结果数量，如下面的代码所示：

```js
function byName(coll, name, take) {
    return _(coll)
        .filter({ name: name })
        .take(_.isUndefined(take) ? 100 : take)
        .value();
}

var collection = [
    { name: 'Theodore', enabled: true },
    { name: 'Leslie', enabled: true },
    { name: 'Justin', enabled: false },
    { name: 'Leslie', enabled: false }
];

byName(collection, 'Leslie');
// →
// [
//   { name: "Leslie", enabled: true },
//   { name: "Leslie", enabled: false }
// ]

byName(_.filter(collection, 'enabled'), 'Leslie');
// →
// [ { name: "Leslie", enabled: true } ]

byName(_(collection).filter('enabled'), 'Leslie');
// →
// [ { name: "Leslie", enabled": true } ]
```

我们的 `byName()` 函数包装了传入的集合，并应用了一个 `filter()` 和一个 `take()` 操作。它还接受一些参数。`name` 参数是我们过滤集合的名称。`take` 参数是可选的，如果提供，则指定要返回的项目数量。如果 `take` 参数缺失，我们默认为 `100`。

在前面的代码中演示了三种不同的 `byName()` 调用。第一次调用是最直接的。我们只是传递了名称 `Leslie`，因为这是我们想要过滤集合的名称。接下来的调用在集合上执行了一个 `filter()` 操作，然后将结果传递给 `byName()`。最后的调用与第二个调用得到相同的结果。然而，你会注意到我们已经包装了集合，由于 `filter()` 函数是可链式的，包装器实例被作为 `coll` 参数传递。

### 注意

重新包装 Lo-Dash 包装器是安全的。构造函数能够识别这一点，并知道如何处理它。

这个函数构建了一个相对通用的链。我们可以在运行时传入集合以及我们想要过滤的名称值。我们甚至可以传入我们想要获取的结果数量，而函数并不关心它是否得到一个包装的值。这一点尤其有用，因为它允许我们使用我们开发的执行链式函数调用的其他函数，返回这些链，并使用它们。`byName()` 函数的限制因素在于它调用 `value()`，并返回未包装的集合。

## 返回链

总是让我们的构造包装器的函数返回相同的包装器实例几乎总是一个好主意。在上一节中，我们的函数在调用链完成之后解包了值，并返回了它。这种方法的缺点是调用者可能还有更多操作要应用在链上。为了做到这一点，值需要再次被包装。Lo-Dash 包装器实例应该有自由在你的代码中移动，并从函数传递到函数，就像它是一个普通的数组一样，如下面的示例所示：

```js
function sort(coll, prop, desc) {
    var wrapper = _(coll).sortBy(prop);
    return desc ? wrapper.reverse() : wrapper;
}

var collection = [
    { first: 'Bobby', last: 'Pope' },
    { first: 'Debbie', last: 'Reid' },
    { first: 'Julian', last: 'Garcia' },
    { first: 'Jody', last: 'Greer' }
];

sort(collection, 'first').value(),
// →
// [
//   { first: "Bobby", last: "Pope" },
//   { first: "Debbie", last: "Reid" },
//   { first: "Jody", last: "Greer" },
//   { first: "Julian", last: "Garcia" }
// ]

sort(collection, 'first', true).value(),
// →
// [
//   { first: "Julian", last: "Garcia" },
//   { first: "Jody", last: "Greer" },
//   { first: "Debbie", last: "Reid" },
//   { first: "Bobby", last: "Pope" }
// ]

sort(collection, 'last')
    .takeRight(2)
    .pluck('last')
    .value();
// → [ "Pope", "Reid" ]
```

`sort()` 函数相当简单，看起来并没有做太多。表面上，它只是接收一个集合，对其进行排序，然后返回它。是的，这是高层次的目标。首先，你会注意到 `coll` 参数被包装在 Lo-Dash 构造函数中——参数值可以是包装器实例或未包装的值。该函数还接受一个属性名称或一个用于排序集合的函数回调。`desc` 参数是可选的，如果为 `true`，则反转排序顺序。

与我们之前实现的 `byName()` 函数相比，这个函数的主要区别在于 `sort()` 总是返回一个包装实例。这意味着如果调用者需要向链中添加更多函数调用，我们不需要重新包装返回的值。你可以在前面的代码中看到这一点，即 `sort()` 的最后调用。在这里，我们向链中添加了 `takeRight()` 和 `pluck()` 调用。以这种方式设计函数使我们能够在整个代码中使用包装器具有很大的灵活性。一般规则是让你的函数对包装器友好，无论是它们接受的参数还是它们返回的内容。

看起来，这种权衡是调用者不仅需要调用你的函数，还需要调用 `value()` 函数。有时，如果你只想获取实际值以便开始工作，这可能会很麻烦，但请记住，直到调用 `value()` 函数，链本身并不会执行。这涉及到延迟求值，简单来说就是返回值不会在调用 `value()` 之前计算。所以这实际上可能是一个期望的特性——能够在不执行它们的情况下构建链。

### 注意

你应该始终以某种形式记录你的函数确实返回一个 Lo-Dash 包装器，并且调用者需要调用 `value()`。

# 函数组合

不论我们的函数是手动调用，还是作为另一个函数的回调使用，或者在涉及链的其他上下文中使用，函数组合有助于构建更大的功能块。例如，我们可能有两个较小的函数，它们各自服务于特定的目的。当我们处于可能需要这些函数的场景时，我们可以使用 Lo-Dash 中的函数工具来组合一个新的函数，利用它们，而不是自己从头开始编写。

## 组合泛型函数

在本章的早期，我们强调了函数需要是通用的，如果它们要在多个上下文中提供帮助。当组合较小的函数的较大组件时，这个想法同样适用。如果我们要用它们来组合任何更大的东西，较小的函数需要是通用的。同样，组合也应该尽可能通用，这样我们就可以将其用作应用程序更大块的一部分的成分。以下代码展示了这一点：

```js
function enabledIndex(obj) {
    return _.transform(obj, function(result, value, key) {
        result[key] = _.result(value, 'enabled', false);
    });
}

var collection = [
    { name: 'Claire', enabled: true },
    { name: 'Patricia', enabled: false },
    { name: 'Mario', enabled: true },
    { name: 'Jerome', enabled: false }
];

var indexByName = _.partialRight(_.indexBy, 'name'),
    enabled = _.partial(_.flow(indexByName, enabledIndex),
    collection);

enabled();
// →
// {
//   Claire: true,
//   Patricia: false,
//   Mario: true,
//   Jerome: false
// }

collection.push({ name: 'Gloria', enabled: true });
enabled();
// →
// {
//   Claire: true,
//   Patricia: false,
//   Mario: true,
//   Jerome: false,
//   Gloria: true
// }
```

这段代码有两个泛型实用函数。`indexByName()` 函数接受一个集合，并返回一个对象，其中键是集合中每个项目的 `name` 属性。`enabledIndex()` 函数接受一个对象，并根据其 `enabled` 属性将每个属性值转换为布尔值。也许，这些函数在其他地方的应用中单独使用，但现在，在开发新组件的过程中，我们找到了一个需要将它们一起使用的用例。

而不是必须独立地调用每个函数并将第一个函数的输出传递给第二个函数，我们决定编写一个`enabled()`函数。这样，每次我们需要将名称映射到`enabled`布尔值的对象结构时，都可以使用简单的调用。这是通过使用`flow()`函数部分应用集合参数到我们创建的函数来实现的。`flow()`函数将第一个参数传递给第一个函数，然后传递下一个，依此类推，并返回结果。

### 注意

此代码假设每个对象的`name`属性是唯一的。否则，这种索引方式就没有意义了。

## 组合回调

我们在前一节中编写的`enabled()`函数旨在直接由我们的代码在某个地方调用。另一方面，回调通常传递给 Lo-Dash 函数。传递内联匿名函数效果很好，除非你发现自己一次又一次地编写相同的回调函数，或者至少是略有差异的类似函数。没有理由我们的应用程序不能组合通用回调并在整个应用程序中使它们可用，以鼓励重用而不是复制。让我们看看这个例子：

```js
var collection = [
    { first: 'Andrea', last: 'Stewart', age: 28 },
    { first: 'Clarence', last: 'Johnston', age: 31 },
    { first: 'Derek', last: 'Lynch', age: 37 },
    { first: 'Susan', last: 'Rodgers', age: 41 }
];

var minimal = _.flow(_.identity,
    _.partialRight(_.pick, [ 'last', 'age' ]));

_.map(collection, minimal);
// →
// [
//   { last: "Stewart", age: 28 },
//   { last: "Johnston", age: 31 },
//   { last: "Lynch", age: 37 },
//   { last: "Rodgers", age: 41 }
// ]
```

回调函数面临的一个常见问题是它们无法控制自己的调用方式。一个函数可能使用单个参数调用其每个回调，而下一个则使用三个参数。这阻止了我们做某些我们本来想做的事情，例如使用已经构建并准备就绪的 Lo-Dash 函数组合回调。

在此代码中定义的`minimal()`函数用于从传入的参数中仅选择必要对象属性。假设我们想将此回调传递给`map()`。嗯，`map()`使用三个参数调用其回调，第一个参数是我们真正感兴趣的实际项。这意味着我们几乎不可能使用部分应用参数作为回调的 Lo-Dash 函数。

我们使用`minimal()`回调采用的解决方案是使用`flow()`来组合回调。你会注意到第一个函数是`identity()`。这个函数除了返回传递给它的任何值外，什么都不做。换句话说，它返回第一个参数。在流中下一个是我们的部分函数，它使用`pick()`。而且你知道吗？它只会收到一个参数，正如我们所需要的，即使在使用带有三个参数的`map()`回调时。

## 组合链

现在，我们将探讨如何组合与链一起工作的函数。正如我们在本章中看到的，对于您和任何使用您函数的人来说，使函数接受的参数和返回值灵活是有益的。例如，接受包装实例并返回包装实例的函数意味着它可以传递几乎所有内容，调用者可以自由地扩展调用链。就像我们可以组合普通函数和回调函数一样，我们也可以组合主要关注与链一起工作的函数。请看以下示例：

```js
function sorted(wrapper) {
    return _(wrapper).sortBy();
}

function rejectOdd(wrapper) {
    return _(wrapper).reject(function(item) {
        return item % 2
    });
}

var sortedEvens = _.flow(sorted, rejectOdd),
    evensSorted = _.flow(rejectOdd, sorted,
        _.partialRight(_.result, 'value')),
    collection = _.shuffle(_.range(1, 11));

sortedEvens(collection)
    .reverse()
    .value();
// → [ 10, 8, 6, 4, 2 ]

evensSorted(collection);
// → [ 2, 4, 6, 8, 10 ]
```

这里 `rejectOdd()` 函数接受一个集合或包装实例作为第一个参数，并过滤掉奇数。请注意，它返回一个包装实例而不是未展开的值。我们使用这个对包装友好的函数来组合两个新的函数。第一个是 `sortedEvens()`，它使用我们的 `sorted()` 函数对集合进行排序。这返回一个包装实例，然后将其传递给 `rejectOdd()` 函数。`evensSorted()` 函数执行类似操作，但顺序不同。它在排序之前拒绝奇数，然后使用 `partialRight()` 通过 `result()` 函数展开值。

您可以看到，当我们调用 `sortedEvens()` 函数时，它返回一个包装实例，因为我们通过 `reverse()` 扩展了函数调用链，然后我们获取值。然而，我们不会用我们的组合 `evensSorted()` 函数执行这种扩展，因为它为我们展开了值。

## 方法组合

有时，将函数附加到特定对象作用域或实现方法是有意义的。如果函数需要特定实例的值才能操作，那么将其作为原型的属性实现可能是个好主意，这样它就始终可用于实例。因此，我们可以使用本章中讨论的技术来帮助我们构建对象方法，这些方法是我们应用程序结构中的另一个构建块。请看以下对象方法的示例：

```js
function validThru(next, value) {
    return value && next;
}

function User(first, last, age) {
    this.first = first;
    this.last = last;
    this.age = age;
}

User.prototype.valid = function() {
    return _.chain(this.first)
        .isString()
        .thru(_.partial(validThru, this.last))
        .isString()
        .thru(_.partial(validThru, this.age))
        .isFinite()
        .value();
}

new User('Orlando', 'Olson', 25).valid();
// → true

new User('Timothy', 'Davis').valid();
// → false

new User('Colleen').valid();
// → false
```

我们的 `User` 构造函数接受三个参数，并且所有这些都被设置为实例值。我们还实现了一个 `valid()` 方法。在这里，我们正在使用函数调用链来验证每个实例属性。请注意，我们在这里启用了显式链式调用。这意味着链中的函数通常返回未展开的值，但不会这样做。我们这样做是因为我们需要通过链传递原始值。

`first` 属性被包装，我们使用 `isString()` 函数验证它是否为字符串。接下来，我们使用 `thru()`。在这里，我们使用我们的 `validThru()` 函数作为 `thru()` 的回调。基本上，如果 `isString()` 返回的值（上一个调用）是 `true`，则返回下一个值。在这种情况下，它部分应用为 `last` 属性。对 `age` 属性执行相同的步骤。

这种方法的优点在于，链式调用需要访问几个属性，并且所有这些属性都包含在方法内部。然后我们可以构建一个可读的链式调用，验证所有这些属性，而不需要多个控制流语句，这比链式调用中的两行代码更难维护。

# 创建 mixins

本章我们将要访问的最后一个主要构建块是 mixin。Lo-Dash 有一个 `mixin()` 函数，允许我们通过提供自己的函数来扩展 API。你想要这样做有两个原因。第一个原因是通过将你的通用工具集放在 Lo-Dash 对象中，你可以在 `_` 符号可访问的地方使用它们。第二个原因是，一旦你混合了自己的函数，它就可以作为函数调用链的一部分使用。

## 创建一个 average() 函数

一个库如 Lo-Dash 能够实际提供的实用工具是有限的。那些被认为对普通用户最有用的工具是我们默认提供的。这并不意味着你正在开发的应用程序没有高价值的使用案例，你希望 Lo-Dash 实现这个功能。例如，假设你的应用程序到处都在计算平均值。虽然库中没有提供 `average` 函数，但这并不意味着我们不能将这个函数添加到代码中：

```js
_.mixin({average: function(coll, callback) {
    return _(coll)
        .map(callback)
        .reduce(function(result, item) {
            return result + item;
        }) / _.size(coll);
}});

var collection = [
    { name: 'Frederick', age: 41, enabled: true },
    { name: 'Jasmine', age: 29, enabled: true },
    { name: 'Virgil', age: 47, enabled: true },
    { name: 'Lila', age: 22, enabled: false }
];

_.average(collection, 'age');
// → 34.75

_.average(collection, function(item) {
    return _.size(item.name);
});
// → 6.5

_(collection)
    .filter('enabled')
    .average('age');
// → 39
```

在我们的 `average()` mixin 中进行的计算非常简单——将项目除以集合的长度。我们需要考虑的是这些项目的映射。如果你查看 `average()` mixin 接受的参数，你会注意到它需要一个集合，这是必需的，还有一个回调函数。回调函数是可选的，可以是任何作为 `map()` 回调接受的函数。然后我们的链式调用在除以集合大小时，将这些项目减少到总和。

你可以看到，`average()` 函数现在是 Lo-Dash 对象的一部分，并且我们可以传递一个字符串参数或一个函数回调。你还可以看到，该函数是链式可用的，如最后调用所示。

## 创建一个 distance() 函数

让我们创建一个更复杂的 mixin 函数，称为 `distance`。它将使用 Levenshtein 距离算法来测量两个字符串之间的编辑距离。我们将创建另一个使用 `distance()` 的 mixin。这个函数将根据与目标字符串的最短距离对集合进行排序：

```js
_.mixin({distance: function(source, target) {
    var sourceSize = _.size(source),
        targetSize = _.size(target),
        matrix;

    if (sourceSize === 0) {
        return targetSize;
    }
    if (targetSize === 0) {
        return sourceSize;
    }

    matrix = _.map(_.range(targetSize + 1), function(item) {
        return [ item ];
    });

     _.each(_.range(sourceSize + 1), function(item) {
        matrix[0][item] = item;
    });

    _.each(target, function(targetItem, targetIndex) {
        _.each(source, function(sourceItem, sourceIndex) {
            if (targetItem === sourceItem) {
                matrix[targetIndex + 1][sourceIndex + 1] =
                    matrix[targetIndex][sourceIndex];
            } else {
                matrix[targetIndex + 1][sourceIndex + 1] = Math.min(
                    matrix[targetIndex][sourceIndex] + 1,
                    Math.min(matrix[targetIndex + 1][sourceIndex] + 1,
                        matrix[targetIndex][sourceIndex + 1] + 1));
            }
        });
    });

    return matrix[targetSize][sourceSize]

}});

_.mixin({closest: function(coll, value, callback) {
    return _.sortBy(coll, _.flow(_.callback(callback), function(item) {
        return _.distance(value, item);
    }));
}});

var collection = [
    'console',
    'compete',
    'competition',
    'compose',
    'composition'
];

_.distance('good', 'food');
// → 1

_.closest(collection, 'composite');
// →
// [
//   "compose",
//   "compete",
//   "composition",
//   "console",
//   "competition"
// ]

_(collection)
    .closest('consulate')
    .first();
// → "console"
```

我们不会专注于 Levenshtein 距离算法的细节；关于这一点，网上有大量的资源。我们刚刚实现的 `distance()` mixin 接受一个 `source` 字符串和一个 `target` 字符串，用于进行比较。返回值表示使目标匹配源所需的编辑次数。例如，前面代码中 `distance()` 的调用结果为 `1`。

`closest()` 混合函数使用 `distance()` 作为 `sortBy()` 回调函数。这是一个有用的函数，因为它通常是我们比较源字符串的目标字符串集合。此外，由于它是一个混合函数，我们能够将其用于链式调用。`closest()` 的最后调用执行此操作，然后使用 `first()` 获取最接近的值。

# 摘要

在本章中，我们学习了一些构建高级应用程序组件的有用方法。函数是 Lo-Dash 编程的基本单元，因此正确利用它们提供的一切非常重要。我们讨论了在设计可重用函数时遇到的一些常见问题。例如，函数的特异性和它接受的参数可以影响函数在 Lo-Dash 代码中的使用位置和方式。

通用包装器和它们实现的链式函数调用是强大的工具，并且它们附带了许多实现选项。我们探讨了链的不同方面的几个示例，以及这些包装器如何与我们的应用程序中的各种函数交互。

函数组合是函数式编程的一个基本组成部分，我们学习了如何利用 Lo-Dash 提供的函数式工具来组合更大的应用程序代码片段。这包括我们手动调用的通用函数和回调函数。本章以查看用于扩展 Lo-Dash API 的混合函数结束。下一章将向您展示如何使用这些应用程序级别的 Lo-Dash 组件，并确保它们与其他库良好地协同工作。
