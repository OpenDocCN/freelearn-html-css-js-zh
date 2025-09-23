# 第八章：内部设计和性能

本书最后一章将探讨关键 Lo-Dash 组件的内部设计。所有前面的章节都专注于库的外部特性。现在我们已经熟悉了 Lo-Dash 的用途，是时候看看其内部结构了。这并不是对 Lo-Dash 源代码的深入剖析。不过，好奇的读者当然可以查看代码。我们将触及 Lo-Dash 实现中最重要的一些部分。这些部分使得 Lo-Dash 不仅运行速度快，而且可预测。

在考虑到这些设计的基础上，我们将在本章剩余的部分中查看一些可以改进的 Lo-Dash 代码。了解一些设计动机可能会在您做出设计决策时提供指导。

在本章中，我们将涵盖以下主题：

+   设计原则

+   提高性能

+   惰性评估

+   缓存事项

# 设计原则

Lo-Dash 最初设定了一些相当低调的目标。Underscore 因其解决的问题以及其 API 的连贯性和易用性而受到大众的喜爱。Lo-Dash 的创造者，John-David Dalton，想要证明在提供跨浏览器的连贯性和性能的同时，实现一个像 Underscore 那样的优秀 API 是可能的。此外，Lo-Dash 有自由去实现 Underscore 不欢迎的新特性。

为了证明他的观点，John-David 必须建立一些指导设计原则。一些基本原则至今仍然存在，而其他一些则已经演变成其他形式，以更好地支持使用该库并为其做出贡献的程序员。如果没有适应变化的能力，Lo-Dash 将什么都不是。

## 函数编译到基础函数

Lo-Dash 的早期版本使用了一种称为**函数编译**的技术。这意味着存在一个骨架函数的模板。然后 Lo-Dash 会填充这些模板，并即时创建函数实例。这些实例随后通过 API 暴露出来。这种方法的优点是，无需实现该函数的多个版本，就可以轻松地为单个函数实现大量的可变性。能够在保持代码体积小的同时实现这样的通用函数，意味着 Lo-Dash 能够处理各种不同的用例，从性能角度和错误修复/一致性角度来看。

函数编译的第一个问题是代码的可读性——如此动态的东西并不容易被开发者接受。开源的这一方面也就消失了——你不会因为吓跑开发者而得到他们的代码审查。第二个问题是 JavaScript 引擎在持续改进其优化运行时 JavaScript 代码的能力。这也被称为**即时**（**JIT**）优化。因此，从现在到 Lo-Dash 首次构思的时间，浏览器供应商已经取得了长足的进步。在如此短的时间内，这些改进并没有被 Lo-Dash 及其函数编译方法充分利用。

在 Lo-Dash 的最新版本（特别是 2.4 和 3.0）中，函数编译方法已被替换为**基础函数**。换句话说，基础函数是通用组件，被多个面向公众的函数所使用。库的早期版本由于担心不必要的间接引用会导致性能损失，因此回避了抽象。虽然抽象确实会产生额外的开销，但帮助浏览器执行 JIT 优化所带来的好处超过了这种成本。

这并不意味着 Lo-Dash 放弃了所有关于抽象开销的谨慎。实现方式既巧妙又易于阅读，解决了早期理解源代码的问题。一个特定的基础函数可能被用在多个地方，这减少了重复代码。更重要的是基础函数的结构方式。通过 API 公开的特定函数将进行一些初步工作，以便理解传递的参数。本质上，这是为了准备将更精确的参数传递给基础函数。这为 JavaScript 引擎提供了更好的可预测性。调用基础函数的成本通常可以忽略不计，尤其是在频繁调用相同函数时——引擎通常会将其内联到调用位置。

那么，这对 Lo-Dash 程序员有什么影响呢？实际上没有。这些内部基础函数的结构和使用方式对你的代码没有任何影响。然而，这应该能让你对 Lo-Dash 如何根据开发者反馈和不断变化的 JavaScript 技术快速演变有所了解。

## 优化常见情况

这个原则，“**优化常见情况**”，从 Lo-Dash 诞生之日起就存在。当然，细微的实现细节已经有所演变，但基本理念仍然保持不变，这可能会一直如此。把这看作是 Lo-Dash 的黄金法则（非官方规则）。正如 Linux 内核开发有一个被称为“*不要破坏用户空间*”的黄金法则一样，把“*优化常见情况*”看作是始终追求的目标。

采用现在更受欢迎的基函数方法，而不是函数编译。我们可以根据用户提供的参数选择要调用的基函数。例如，接受集合的函数可以使用仅适用于数组的基函数。它针对数组进行了优化。因此，当调用接受集合的函数时，它会检查是否正在处理数组。如果是，它将使用更快的基函数。以下是一个使用伪 JavaScript 的模式说明：

```js
function apiCollectionFunction(collection) {
    if (_.isArray(collection)) {
        return baseArray(collection);
    } else {
        return baseGeneric(collection)
    }
}
```

常见路径是第一个测试的路径。执行的 `baseArray()` 函数足够通用且使用频率足够高，以至于 JIT 会给予特殊处理。策略是假设常见情况是传递一个数组。这种假设也不是任意的；它在开发过程中与典型用例进行了基准测试。最坏的情况是我们处理字符串或当普通对象不慢时，这并不是说它没有优化。因此，这些较慢的调用，尽管它们很少发生，但会被频繁发生的优化调用所抵消。

常见情况甚至可以分级。也就是说，当函数被调用时，它会抛出一个或多个情况，所有这些可能性都有它们出现频率的顺序。例如，如果最常见的情况没有满足，下一个最常见的情况是什么？以此类推。这种技术的影响是将不常见的代码推向函数的底部。单独来看，这不会对性能产生巨大影响，但当库中的每个函数都持续遵循相同的常见情况优化技术时，影响就非常大了。

## 循环很简单

Lo-Dash 在其代码中使用了大量的循环。这是因为有很多集合的迭代。这也是因为 Lo-Dash 不使用某些会消除循环需要的原生函数。这与 Underscore.js 在这个问题上的立场相反。它更喜欢在可用时使用原生方法。逻辑是，JavaScript 库不需要担心迭代性能。相反，浏览器供应商应该提高原生方法性能。

这种方法是有意义的，尤其是在副作用是编写更少代码的情况下。然而，Lo-Dash 不依赖于浏览器供应商提供性能。我们可以从简单的 `while` 循环中获得更好的性能，这可能会在可预见的未来继续。原生方法无疑比未优化的 JavaScript 代码更快，但它们无法执行与我们使用纯 JavaScript 时相同的优化。

### 注意

Lo-Dash 是一个策略性的动物。它不喜欢依赖于某些原生 JavaScript 方法，但它非常依赖任何给定 JavaScript 引擎的 JIT 能力来实现性能和成本平衡。

Lo-Dash 也不喜欢依赖 `for` 循环——更倾向于使用 `while` 循环。`for` 循环在用于遍历集合时很有用，从而增强了代码的可读性。在这些简单情况下，尝试使用 `while` 循环只是繁琐。尽管 `while` 循环确实比 `for` 循环有轻微的性能优势，但这并不是特别明显。在频繁遍历的几个大型集合的情况下，性能差异是明显的。这是 Lo-Dash 考虑的常见情况。考虑以下代码：

```js
var collection = _.range(10000000),
    length = collection.length,
    i = 0;

console.time('for');
for (; i < length; i++) {
    collection[i];
}
console.timeEnd('for');

i = 0;

console.time('while');
while (++i < length) {
    collection[i];
}
console.timeEnd('while');
// →
// for: 13.459ms
// while: 10.670ms
```

两个循环之间的差异几乎不可察觉。可能在大约两年前，`while` 循环相对于 `for` 循环的领先优势可能更大，这也是 Lo-Dash 仍然在所有地方使用 `while` 循环的一个原因。另一个原因是一致性。由于 `while` 循环在 Lo-Dash 中的实现几乎完全相同，你可以预期其性能在整个过程中是可预测的。这一点在不是 `while` 循环、`for` 循环和原生 JavaScript 方法混合的情况下尤其正确。有时，可预测的性能比“有时它更快，但我永远无法确定”要好。

## 回调函数和函数绑定

回调函数在 Lo-Dash 中无处不在，无论是内部使用还是作为 API 函数的参数。因此，这些函数以尽可能少的开销执行是很重要的。导致这些函数调用变慢的主要问题是 `this` 上下文，即函数绑定到的上下文。如果没有上下文需要考虑，那么涉及的开销显然更少，特别是考虑到这些回调函数通常在函数对集合进行操作时每次迭代只被调用一次。

如果回调函数有特定的上下文，那么我们必须使用 `call()` 来调用该函数，因为它允许我们设置上下文。或者如果有未知数量的参数，我们使用 `apply()` 函数，将上下文和参数作为数组传递。如果迭代执行，这会特别慢。为了帮助克服这些性能障碍，Lo-Dash 使用一个基础函数来帮助构建回调函数。

此函数用于任何传递回调函数作为参数的地方。第一步是使用此函数构建一个可能被包装的回调函数。这种初步检查是值得的，因为当它需要迭代调用时，可以节省潜在的时间。以下是这个函数大致工作原理的简要说明：

```js
function baseCallback(func, thisArg, argCount) {
    if (!thisArg) {
        return func;
    }

    if (alreadyBound(func)) {
        return func;
    }

    if (argCount == 1) {
        return function(collection) {
            return func.call(thisArg, collection);
        }
    }

    return function() {
        return func.apply(thisArg, arguments);
    }
}
```

这是对`baseCallback()`实际执行内容的粗略简化，但总体模式是准确的。首先检查的是构建回调函数最常见的案例。不常见且较慢的案例被推到后面。例如，如果没有`thisArg`，我们不需要绑定函数；它可以直接返回。接下来检查的是函数是否已经被绑定。如果已经绑定，则忽略`thisArg`值并返回函数。如果这两个检查都不通过，并且提供了`argCount`参数，我们可以使用`call()`，提供确切的参数数量。前面的伪代码显示了只有一个参数的情况，但在现实中，它会检查几个确切的参数数量。

不常见的案例是当提供了`thisArg`时，这意味着我们必须绑定函数，但我们不知道有多少个参数。因此，我们使用`apply()`，这是最慢的情况。`baseCallback()`能够处理的其它情况包括将字符串或纯对象作为`func`传递而不是函数实例。对于这种情况，会返回特定的回调函数，并且这也会在早期进行检查，因为它是一个常见的案例。

`alreadyBound()`函数是为了简洁而编造的。Lo-Dash 通过查看该函数的元数据来知道一个函数是否已经被绑定。在这个上下文中，元数据是指由 Lo-Dash 附加到函数上的数据，但对开发者来说是完全透明的。例如，许多回调会跟踪它们被调用的频率数据。如果函数变得*热点*，Lo-Dash 会将其与其他不经常执行的回调区别对待。

# 提高性能

虽然 Lo-Dash 从一开始就是为了最佳性能而设计的，但这并不意味着我们无法对我们的 Lo-Dash 代码进行一些基本的修改来提高性能。实际上，我们有时可以借鉴一些 Lo-Dash 的设计原则并将其直接应用于使用 Lo-Dash 的代码中。

## 改变操作顺序

在一个值（如数组）周围使用 Lo-Dash 包装器，让我们可以链式地对该值执行许多操作。正如我们在第六章中看到的，“应用构建块”，与逐个拼接调用 Lo-Dash 函数的多个语句相比，包装器有许多优势。例如，最终结果通常是更简洁、更易读的代码。我们在链中调用这些操作的顺序可能产生相同的结果，但性能影响却不同。让我们看看三种不同的过滤集合的方法，它们都能得到相同的结果：

```js
var collection = _.map(_.range(100), function(item) {
    return {
        id: item,
        enabled: !!_.random()
    };
});

var cnt = 1000;

console.time('first');
while (--cnt) {
    _(collection)
        .filter('enabled')
        .filter(function(item) {
            return item.id > 75;
        })
        .value();
}
console.timeEnd('first');

cnt = 1000;

console.time('second');
while (--cnt) {
    _(collection)
        .filter(function(item) {
            return item.id > 75;
        })
        .filter('enabled')
        .value();
}
console.timeEnd('second');

cnt = 1000;

console.time('third');
while (--cnt) {
    _(collection)
        .filter(function(item) {
            return item.enabled && item.id > 75;
        })
        .value();
}
console.timeEnd('third');
// → 
// first: 13.368ms
// second: 6.263ms
// third: 3.198ms
```

`collection` 数组相当简单。它包含 `100` 个项，每个项都是一个具有两个属性的对象。第一个是一个数值 ID。第二个是一个随机的布尔值。目标是过滤掉任何未 `启用` 的项以及任何 `id` 值小于 `75` 的项。

第一种方法构建了一个由两个 `filter()` 调用组成的链。第一个 `filter()` 调用移除了任何禁用项。第二种方法移除了具有 `id` 属性且其值小于 `75` 的任何项。然而，这些过滤操作的顺序并不最优。你可能已经注意到，根据 `id` 值移除的项目数量很多。这是由于过滤器的性质和我们正在处理的数据集的特性所导致的。

对 `filter()` 的任何调用都意味着在集合上发生线性迭代。在第一种方法中，调用了两次 `filter()`，这意味着我们将不得不对集合进行两次线性迭代。鉴于我们现在对集合数据和过滤器所寻找的内容的了解，我们可以优化过滤调用的顺序。这是一个简单的更改。我们首先按 `id` 过滤，然后按 `enabled` 属性过滤。结果是性能有明显的提升，因为第二次调用 `filter()` 必须迭代的项目要少得多。

第三种方法更进一步，完全移除了一个迭代过程。由于两个过滤条件都在 `filter()` 回调函数中检查，因此没有必要对任何集合项进行两次迭代。

### 注意

当然，这里的权衡是在给定的回调函数中增加了更多的复杂性。如果你的应用程序进行大量的过滤，请记住这一点，因为你将想要避免定义高度专业化的、仅用于单一目的的回调函数。通常，保持函数小而通用是一个更好的主意。第二种方法达到了一个很好的平衡。这类优化通常不会一开始就发生，所以等到常见情况显现出来后再尝试对其进行优化。

## 排序和索引集合

如果你在开发的程序中，集合的顺序是一个重要因素，你可以引入一些调整，以利用其重要性。这些调整包括保持排序顺序。每次需要渲染时重新排序集合实际上并没有什么意义。相反，最好是一次性对集合进行排序，然后通过在正确的位置插入新项来维护其顺序。Lo-Dash 有一个 `sortedIndex()` 函数，它有助于找到新项的正确插入点。实际上，它执行二分搜索，比在集合中进行线性搜索要快得多。

对于更快的过滤操作，我们可以借用`sortedIndex()`函数。如果我们有一个已排序的集合，实际上根本不需要使用线性搜索来过滤项，这在最坏情况下表现相当差。让我们引入一个新的`mixin`函数，它执行与`filter()`函数相同的工作，但针对已排序的集合进行了优化：

```js
_.mixin({ sortedFilter: function(collection, value, iteratee) {
    iteratee = _.callback(iteratee);
    var index = _.sortedIndex(collection, value, iteratee),
        result = [],
        item;
    while (true) {
        item = collection[index++];
        if (_.isEqual(iteratee(item), iteratee(value))) {
            result.push(item);
        } else {
            break;
        }
    }
    return result;
}});

var collection = _.map(_.range(100), function(item) {
    return {
        id: item,
        age: _.random(50)
    };
});

var shuffled = _.shuffle(collection),
    sorted = _.sortBy(shuffled, 'age');

console.time('shuffled');
console.log(_.filter(shuffled, { age: 25 }));
console.timeEnd('shuffled');
// → 
// [
//   { id: 63, age: 25 },
//   { id: 6, age: 25 },
//   { id: 89, age: 25 }
// ]
// shuffled: 4.936ms

console.time('sorted');
console.log(_.sortedFilter(sorted, { age: 25 }, 'age'));
console.timeEnd('sorted');
// → 
// [
//   { id: 63, age: 25 },
//   { id: 6, age: 25 },
//   { id: 89, age: 25 }
// ]
// sorted: 0.831ms
```

我们引入的新功能——`sortedFilter()`——比`filter()`函数更快。再次强调，这是因为我们不需要依赖线性搜索，因为集合已经排序。相反，我们使用`sortedIndex()`函数来找到我们想要的东西。它使用二分搜索，这意味着在更大的集合中，有很多项不会被检查。最终结果是更少的 CPU 周期和更快的执行时间。

我们的`sortedFilter()`实现，很大程度上得益于`sortedIndex()`，并不复杂。二分搜索为我们提供了插入新项的位置，但我们实际上并没有插入任何东西。我们只是在寻找它。可能有几个项符合我们的标准，或者可能没有。这就是我们在集合上迭代，以插入索引作为起点的地方。现在我们必须使用`isEqual()`显式检查值并构建结果数组。由于集合已排序，我们知道当项不再符合过滤标准时停止并返回。

### 注意

在提高 Lo-Dash 函数性能时，始终要注意验证代码的正确性。最简单的方法是设置一系列自动化测试，比较 Lo-Dash 函数的输出与你的更快变体的输出。这允许你在对新的发现速度过于兴奋之前，将所有各种边缘情况都抛给你的代码。Lo-Dash 处理了很多边缘情况，所以请确保你不会为了性能而牺牲安全性。

在加快集合上的过滤操作速度的另一种技术是对其进行索引。这意味着创建一个新的数据结构，它使用键来查找集合中的常见项。这是避免在大集合中进行昂贵的线性搜索的另一种方法。以下是一个使用`groupBy()`对集合进行索引的示例，以便快速搜索使用常见过滤标准的项：

```js
var collection = _.map(_.range(100), function(item) {
    return {
        id: item,
        age: _.random(50),
        enabled: !!_.random()
    };
});

var indexed = _.groupBy(collection, function(item) {
    return +item.enabled * item.age;
});

console.time('where');
console.log(_.where(collection, { age: 25, enabled: true }));
console.timeEnd('where');
// → 
// [
//   { id: 23, age: 25, enabled: true },
//   { id: 89, age: 25, enabled: true }
// ]
// where: 5.528ms

console.time('indexed');
console.log(indexed[25] || []);
console.timeEnd('indexed');
// → 
// [
//   { id: 23, age: 25, enabled: true },
//   { id: 89, age: 25, enabled: true }
// ]
// indexed: 0.712ms
```

索引方法查找相同项所需的时间比`where()`方法少得多。当你的应用程序中有多个相同的过滤器实例时，这种方法很有用。`indexed`变量持有集合的索引版本。索引是通过`groupBy()`函数创建的。它接受一个数组作为输入并生成一个对象。索引键是对象键，传递给`groupBy()`的回调函数负责生成这些键。该函数返回键值，如果键已存在，则该项将被添加到该键。

这个想法是，我们希望根据`age`属性值和是否启用来索引项目。在这里，我们使用了一个巧妙的小技巧来实现这一点。`enabled`属性被转换成正整数，并乘以`age`值。因此，任何禁用的项目都将被索引在`0`下，那里没有人会去看。现在你可以看到，在`indexed`对象中查找项目会产生与`where()`过滤器相同的结果。在后一种方法中，我们执行的是一个简单的对象访问操作，而不是遍历一个集合并执行多个操作。

### 注意

虽然这里的速度提升相当令人印象深刻，但请务必考虑这个集合中项目的更新频率。如果你这么想，索引版本实际上只是一个常见过滤结果的缓存。所以如果集合从未更新，只要你对实际上索引集合的一次性付费没有问题，那么你就没问题。

## 绑定与未绑定回调

Lo-Dash 支持回调函数，并且在这方面做得非常好。例如，当不需要`this`上下文时，它避免了使用`call()`和`apply()`，这有一个很好的原因——这些调用比未绑定函数调用要慢得多。因此，当我们编写利用 Lo-Dash 回调函数的应用程序时，我们有选择在每个回调函数应用于集合时提供上下文的选项。在以这种方式编写函数之前，花时间权衡利弊。

当我们想在不同的上下文中使用相同的函数时，将我们的函数绑定到上下文中是方便的。这并不总是必要的，这很大程度上取决于我们代码的整体设计。如果我们有很多对象需要回调函数访问，`this`上下文就非常方便。我们甚至可能有一个用于访问其他对象的单一应用程序对象，依此类推。如果是这样，我们肯定会需要一个方法将这个对象传递给我们的回调函数。这可能意味着绑定`this`上下文，通过函数闭包访问对象，或者为我们的回调创建一个部分函数。

这些选项都不是特别适合性能。因此，如果我们发现我们的回调函数经常需要访问某些对象，那么在回调函数中定义它，而不是将其定义为变量，可能是有意义的。以下代码说明了这个想法：

```js
function callback(item) {
    return _.extend({
        version: this.version
    }, item);
}

function unbound(item) {
    return _.extend({
        version: 2.0
    }, item);
}

var cnt = 1000,
    app = { version: 2.0 },
    boundCallback = _.callback(callback, app),
    collection = _.map(_.range(1000), function(item) {
        return { id: item };
    });

console.time('bound');
while (--cnt) {
    _.map(collection, boundCallback);
}
console.timeEnd('bound');

cnt = 1000;

console.time('unbound');
while (--cnt) {
    _.map(collection, unbound);
}
console.timeEnd('unbound');
// → 
// bound: 662.418ms
// unbound: 594.799ms
```

我们可以看到，未绑定回调函数通常会比绑定回调函数表现更好。这里需要注意的是方法。`bound()`函数通过`map()`调用绑定到特定的上下文中，因为它需要从应用程序对象中获取一些东西。而`unbound()`函数，不依赖于外部实例，会自己声明变量。因此，我们可以在不需要绑定到特定回调函数的情况下，为回调函数获取所需的内容。

起初，这可能会让人觉得在回调函数内定义应用级变量是一种反直觉的方法。好吧，这归结到你的代码的其他部分。你有很多需要访问这些数据的回调函数吗？如果你将这个回调函数放在源树中容易找到的位置，那么这实际上与修改一个变量并没有太大的区别。

### 注意

当只有少数回调函数时，从绑定函数切换到非绑定函数并不会带来巨大的性能提升。即使有很多函数，只要没有影响性能，拥有几个绑定函数也是可以的。本节的想法是让你留意那些无谓地绑定到上下文中的函数。如果它们对你的设计没有明显影响，你可以尝试修复它们。

# 惰性求值

随着 Lo-Dash 3.0 的引入，一些函数使用**惰性求值**来计算它们的结果。这仅仅意味着给定集合中的项目在实际上需要之前不会迭代。确定它们何时需要才是棘手的部分。例如，仅仅调用单个 Lo-Dash 函数并不会触发任何惰性求值机制。然而，链式操作在某些情况下确实可能从这种方法中受益。例如，当我们只从结果中取 10 个项目时，就没有必要进一步迭代整个集合。

为了了解惰性求值的样子，让我们写一些代码来利用它。这里没有明确的事情要做。惰性机制在幕后透明地发生，取决于我们的链中包含哪些操作以及它们的调用顺序：

```js
var collection = _.range(10);

_(collection)
    .reject(function(item) {
        console.log('checking ' + item);
        return item % 2;
    })
    .map(function(item) {
        console.log('mapping ' + item);
        return item * item;
     })
    .value();
// → 
// checking 1
// checking 2
// mapping 2
// checking 3
```

在这里，我们的链由两个函数组成——`reject()`和`map()`。由于`reject()`首先被调用，Lo-Dash 将其作为一个惰性包装器。这意味着当调用`value()`时，事情会有些不同。而不是运行每个函数直到完成，链中的惰性函数会被要求提供一个值。例如，`reject()`只有在`map()`请求值时才会运行。当它运行时，`reject()`会一直运行直到产生一个值。我们实际上可以在输出中看到这种行为。`reject()`函数正在检查项目`1`，该项目被拒绝。然后它继续到项目`2`，该项目通过了测试。然后这个项目被传递给`map()`。然后检查项目`3`，依此类推。

这两个函数调用是交织在一起的，并且这种属性可以向上扩展到更复杂的链中的许多函数。优点是，如果这些函数在整个集合上运行成本太高，通常不需要运行。它们只会在被要求执行时执行。让我们看看这个概念的实际应用：

```js
var collection = _.range(1000000).reverse();

console.time('motivated');
_.take(_.filter(collection, function(item) {
    return !(item % 10);
}), 10);
console.timeEnd('motivated');

console.time('lazy');
_(collection)
    .filter(function(item) {
        return !(item % 10);
    })
    .take(100)
    .value();
console.timeEnd('lazy');
// → 
// motivated: 8.454ms
// lazy: 0.889ms
```

你可以看到，懒惰的方法比有动力的方法花费的时间少得多，尽管它处理了`100`个结果，而后者只处理了`10`个。原因很简单——集合很大，整个集合都是通过有动力的方法进行过滤的。懒惰的方法使用的迭代次数要少得多。

# 缓存事物

提高操作性能的最佳方式是不要执行它——至少不要执行两次，或者更糟糕的是，数百或数千次。重复昂贵的计算是 CPU 周期的无谓浪费，可以通过缓存结果来避免。`memoize()`函数在这里帮助我们，通过缓存被调用函数的结果以供以后使用。然而，缓存有其自身的开销和需要注意的陷阱。让我们首先看看幂等函数——这些函数在给定相同的输入参数时总是产生相同的输出：

```js
function primeFactors(number) {
    var factors = [],
        divisor = 2;

    while (number > 1) {
        while (number % divisor === 0) {
            factors.push(divisor);
            number /= divisor;
         }
        divisor += 1;
        if (divisor * divisor > number) {
            if (number > 1) {
                factors.push(number);
            }
            break;
        }
    }
    return factors;
}

var collection = _.map(_.range(10000), function() {
        return _.random(1000000, 1000010);
    }),
    primes = _.memoize(primeFactors);

console.time('primes');
_.each(collection, function(item) {
    primeFactors(item);
});
console.timeEnd('primes');

console.time('cached');
_.each(collection, function(item) {
    primes(item);
});
console.timeEnd('cached');
// → 
// primes: 17.564ms
// cached: 4.930ms
```

`primeFactors()`函数返回给定数字的质因数数组。它必须进行相当多的工作来计算返回的数组。没有什么是占用 CPU 大量时间的，但无论如何，它的工作——对于给定的输入产生相同结果的工作。像这样的幂等函数是进行记忆化的良好候选者。使用`memoize()`函数来做这件事很容易，我们使用这个函数来生成`primes()`函数。此外，请注意，缓存键是第一个参数，在这里很好也很简单，因为它是我们唯一感兴趣要缓存输入。

### 注意

考虑到查找缓存项所涉及的开销，这是很重要的。这并不是很多，但确实存在。通常，这种开销超过了最初缓存结果的价值。前面的代码是使用相对较大的集合进行测试的一个例子。随着集合大小的缩小，性能提升也随之减少。

虽然缓存幂等函数的结果很方便，因为你永远不必担心缓存失效，但让我们看看一个更常见的用例：

```js
function mapAges(collection) {
    return _.map(collection, 'age');
}

var collection = _.map(_.range(100), function(item) {
        return {
            id: item,
            age: _.random(50)
        };
    }),
    ages = _.memoize(mapAges, function(collection) {
        if (_.has(collection, 'mapAges')) {
            return collection.mapAges;
        } else {
            collection.mapAges = _.uniqueId();
        }
    }),
    cnt = 1000;

console.time('mapAges');
while (--cnt) {
    _.reduce(mapAges(collection), function(result, item) {
        return result + item;
    }) / collection.length;
}
console.timeEnd('mapAges');

cnt = 1000;

console.time('ages');
while (--cnt) {
    _.reduce(ages(collection), function(result, item) {
        return result + item;
    }) / collection.length;
}
console.timeEnd('ages');
// → 
// mapAges: 6.878ms
// ages: 3.535ms
```

在这里，我们正在缓存将集合映射到不同表示形式的结果。换句话说，我们正在映射`age`属性。如果这个映射操作在整个应用程序中重复进行，可能会很昂贵。因此，我们使用`memoize()`函数来缓存映射年龄值的结果，从而生成`ages()`函数。然而，仍然存在查找缓存集合的问题——我们需要一个键解析函数。我们提供的这个函数相当简单。它为集合的`mapAges`属性分配一个唯一的标识符。下次调用`ages()`时，这个标识符会被找到，然后查找缓存的副本。

我们可以看到，不必再次映射集合可以节省 CPU 周期。这是一个简单的映射；其他带有回调函数的映射可能更昂贵，比简单地提取一个值要复杂得多。

### 注意

当然，这段代码假设这个集合是恒定的，永远不会改变。如果你正在构建一个包含许多动态部分的大型应用程序，像这样的静态集合实际上是非常常见的。如果集合，或者集合中的项目，在其生命周期内频繁改变，你必须开始考虑使缓存失效。对于这些易变的集合，缓存映射或其他转换可能并不值得，因为，除了命名之外，缓存失效是编程中所有问题中最难的一个。

# 摘要

在本章中，我们介绍了一些指导 Lo-Dash 设计和实现的影響因素。库的早期版本选择了函数编译的方法，动态构建函数以最佳地处理性能和环境之间的变化。而最近的版本则选择了通用基础函数。函数编译避免了与基础函数相关的一些间接操作。然而，现代浏览器有一个 JIT 优化器，它能够更好地优化基础函数。此外，使用基础函数的代码可读性也更高。

Lo-Dash 实现的金科玉律是对常见情况的优化。你将在 Lo-Dash 的各个地方看到这条规则在发挥作用，它是其卓越性能的关键因素。在任何给定的函数中，最常见的情况首先被高度优化，将不常见的情况推到函数的末尾。回调在 Lo-Dash 中无处不在，因此它们能够可预测地执行非常重要。基础回调机制为我们处理这个问题，并作为一个优化常见情况的绝佳例子。

我们接着探讨了优化 Lo-Dash 代码的一些技术，大多数情况下遵循 Lo-Dash 的设计原则。改变 Lo-Dash 包装器中链式操作的顺序可以消除不必要的迭代。与排序集合一起工作可以对过滤性能产生重大影响。延迟评估是最近引入到 Lo-Dash 中的一个概念，它允许我们在不必要遍历整个集合的情况下与大型集合一起工作。最后，我们探讨了缓存可以帮助提高性能的一些场景，特别是在计算成本高昂的地方。

话虽如此，你们已经准备就绪了。在这本书中，我们学习并实现了许多概念，从 Lo-Dash 中开箱即用的内容开始，到如何提高速度结束。在这个过程中，我们探讨了编写稳健 Lo-Dash 代码最常用的使用模式。到目前为止，Lo-Dash 中的每一部分如何与其它部分相关，从概念到低级函数调用，应该已经很清晰了。就像任何其他库一样，在 Lo-Dash 中做某事有十几种或更多的方式。我希望你现在已经准备好以最佳方式完成它。
