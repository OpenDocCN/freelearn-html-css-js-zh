# 第七章：与其他库一起使用 Lo-Dash

前一章向我们展示了我们的 Lo-Dash 代码在更大的应用中定位后的样子。事物被分解成更通用的、可重用的形式，并且命名和结构保持一致。模式开始显现，随着你的代码开始实现这些模式，它将呈现出一个生产就绪系统的形态。

在将 Lo-Dash 代码部署到生产环境的精神下，本章讨论了当我们的代码被认为稳定时被投入其中的生态系统。Lo-Dash 做了很多事情，但有一些任务这个库并不适合。换句话说，Lo-Dash 很可能不是你应用程序唯一使用的库。

保持我们的代码组织和模块化的最佳方式是使用模块。模块技术在过去的几年中在 JavaScript 社区中受到了很多关注，Lo-Dash 也不例外。我们可以使用应用程序使用的相同模块加载器来实现它。还有适用于 Node.js 的 Lo-Dash 包。

在本章中，我们将涵盖以下主题：

+   模块

+   jQuery

+   Backbone

+   Node.js

# 模块

如果你过去几年中做过任何前端开发，你可能已经听说过 AMD 模块，即使你还没有尝试过它们。AMD 发展迅速，全球范围内有大量的生产部署。前端开发中的这种模块化运动源于缺乏组织具有许多依赖的大型项目的合理方式。在 Web 模块出现之前，我们用来组织依赖的唯一工具是 `<script>` 元素。这仍然是一种被接受的方式来引入 JavaScript 代码——除非有数百个模块。

模块化，尤其是前端网络开发模块化，是一个很大的话题——对于这本书（更不用说本章）来说，太大而不可能充分讨论。所以让我们将这个话题简化为我们 Lo-Dash 程序员相关的内容。将我们的代码分解成只服务于单一目的的模块是良好的编程实践。这提供了良好的关注点分离，并允许我们的组件更容易地独立发展。

在本节中，我们将探讨 **RequireJS**，这是可用的主要 AMD 模块加载技术之一。Lo-Dash 有构建来帮助我们利用这项技术并构建我们自己的模块。话虽如此，让我们深入了解一些实际细节。

### 注意

**AMD** 代表 **异步模块定义**，这是许多 JavaScript 组件遵循的一个简单模式。虽然它不是一个公认的规范，但在即将到来的 ES6 规范中有所酝酿。还有一个相关的模式称为 UMD，它旨在比 AMD 更通用，并有一些值得怀疑的回退模式。我的建议是坚持使用易于使用的东西，比如 RequireJS，直到有一个被广泛支持的采用标准。

## 使用模块组织你的代码

让我们看看一个基本模块的样子。想法是定义函数返回模块定义的组件。组件可以是函数、对象、字符串或任何值。如果你正在开发 Lo-Dash 应用程序，你的模块很可能会返回函数。

### 注意

由于 RequireJS 会发起 XHR 请求，因此使用简单的静态 Web 服务器来提供 JavaScript 模块要容易得多。这本书附带的一些代码有一个`Gruntfile`，它允许你运行一个简单的 Web 服务器。然而，你需要安装 Node.js。互联网上有许多资源可以帮助你在任何平台上安装 Node。一旦 Node 可用，你可以使用以下命令安装 Grunt：

```js
npm install -g grunt-cli grunt-contrib-connect
```

这将在你的系统上使`grunt`命令可用。在根代码目录内，`Gruntfile.js`文件所在的位置，运行以下命令：

```js
grunt connect
```

你会看到一些关于服务器无限运行的信息。按下*Ctrl* + *C*将停止它。就这样！你可以导航到`http://0.0.0.0:8000/chapter7.html`来运行本章的示例。

考虑以下代码：

```js
define([], function() {
    return function(coll, filter) {
        return _(coll)
            .filter(filter)
            .reduce(function(result, item) {
                return result + item.age;
            }, 0) / _.size(coll);
    };
});
```

你可以看到，`define()`函数接受两个参数。第一个是我们依赖的模块数组，第二个是返回该模块定义的组件的函数。在这种情况下，我们的模块没有外部依赖，它返回一个匿名函数。这个函数接受一个`coll`和一个`filter`参数。然后我们使用 Lo-Dash 构造函数来包装集合，并将其缩减到平均值。接下来，让我们看看这个模块是如何被使用的：

```js
var collection = [
    { name: 'Frederick', age: 37 },
    { name: 'Tasha', age: 45 },
    { name: 'Lisa', age: 33 },
    { name: 'Michael', age: 41 }
];

require([ 'modules/average-age' ], function(averageAge) {
    averageAge(collection);
    // → 39
});
```

你可以在这里看到，我们对`require()`函数的调用传递了一个模块依赖数组。在这种情况下，我们依赖于`average-age`模块。当这个模块被加载、评估并准备好使用时，函数回调会被触发。`averageAge`参数是模块返回的值。在这种情况下，它是我们之前定义的函数，我们展示了如何将其应用于一个集合。

## 引入 Lo-Dash

我们`average-age`模块的缺点是它没有定义任何明确的依赖。然而，它显然依赖于 Lo-Dash 可用。那么这段代码是如何工作的呢？`_`变量在哪里定义的？好吧，前面的例子之所以能运行，仅仅是因为我们使用标准的`<script>`标签包含了 Lo-Dash。这会将`_`符号添加到全局命名空间中。

这违反了模块的一个基本原理——不应该需要全局变量。我们最终得到的是隐式依赖，就像前面的代码那样。这意味着使用这个隐式依赖的模块不如它们可能的那样可移植。如果那个`<script>`标签消失了，我们的模块就停止工作了。幸运的是，我们可以将依赖于 Lo-Dash 的模块定义为具有这个依赖的显式依赖，如下面的代码所示：

```js
define([ 'lodash' ], function(_) {
    return function(coll) {
        return _(coll)
            .sortBy(function(item) {
                return item.first + '' + item.last;
            });
    };
});
```

在这里，你可以看到，我们不是将空数组作为 `define()` 的第一个参数传递，而是有一个指向 Lo-Dash 模块的字符串。现在 `_` 符号是 `define()` 函数内的一个参数，而不是全局引用。现在让我们看看这个模块是如何被使用的：

```js
var collection = [
    { first: 'Georgia', last: 'Todd' },
    { first: 'Andrea', last: 'Gretchen' },
    { first: 'Ruben', last: 'Green' },
    { first: 'Johnny', last: 'Tucker' }
];

require([ 'modules/sort-name' ], function(sortName) {
    sortName(collection).value();
    // →
    // [
    //   { first: "Andrea", last: "Gretchen" },
    //   { first: "Georgia", last: "Todd" },
    //   { first: "Johnny", last: "Tucker" },
    //   { first: "Ruben", last: "Green" }
    // ]
});
```

在这里，我们将 `sort-name` 模块作为依赖项，`sortName()` 函数是 `require()` 回调函数的参数。该函数按名称对输入集合进行排序，并返回一个包装器实例。这在这里通过调用 `sortBy()` 后跟 `value()` 来说明。这实际上是一件好事，因为它意味着返回的包装器实例可以在评估和展开之前进行扩展。

你还会注意到，我们在这里间接依赖于 Lo-Dash，因为我们依赖于 `sort-name`。我们可以调用 `value()` 函数并扩展返回的包装器，而不需要显式引用 `_` 符号。这意味着，如果将来某个时刻 `sort-name` 模块不再依赖于 Lo-Dash，我们的函数仍然可以工作，尽管我们可能需要移除 `value()` 调用。

要使 Lo-Dash 与 RequireJS 一起工作，还需要进行另一个步骤。让我们看看 `main.js` 配置文件，它帮助 RequireJS 确定模块的位置以及它们暴露的内容：

```js
require.config({
    paths: {
        lodash: 'lib/lodash'
    },
    shim: {
        lodash: { exports: '_' }
    }
});
```

我们在代码中将 `lodash` 作为依赖项。这个路径配置告诉 RequireJS 在哪里找到该模块。`shim` 配置部分用于未定义为 AMD 模块的模块。由于 Lo-Dash 就是这样，我们必须添加一个适配器，告诉 RequireJS 当某个模块被要求时实际返回什么。

## 使用 Lo-Dash AMD 模块

结果表明，使用 Lo-Dash 的更好方式是采用 AMD 模块的形式。Lo-Dash 提供了特定的 AMD 构建版本可供下载，这些版本不需要使用适配器。通过这种方式获取 Lo-Dash 组件的另一个好处是，如果我们只依赖少数几个函数，我们就不必下载整个库。例如，让我们看看我们如何依赖函数类别：

```js
function Person(first, last) {
    this.first = first;
    this.last = last;
}

Person.prototype.name = function() {
    return this.first + '' + this.last;
}

var collection = [
    new Person('Douglas', 'Wright'),
    new Person('Tracy', 'Wilson'),
    new Person('Ken', 'Phelps'),
    new Person('Meredith', 'Simmons')
];

require([ 'lib/lodash-amd/collection' ], function(_) {
    _.invoke(collection, 'name');
    // →
    // [
    //   "Douglas Wright",
    //   "Tracy Wilson",
    //   "Ken Phelps",
    //   "Meredith Simmons"
    // ]
});
```

在这个例子中，我们使用的是 Lo-Dash 的 AMD 构建。这是通过我们要求的模块路径来体现的。`collection` 模块被定义为 AMD 模块，并包含所有与集合相关的函数。你可以看到我们使用 `_` 符号作为函数参数。这意味着使用集合函数的代码可以像使用任何 Lo-Dash 模块一样编写。例如，如果我们要求完整的 Lo-Dash API 而不是仅集合函数，则不需要更改任何代码。相反，我们只要求 Lo-Dash 的一个子集，从而节省了网络传输成本。

这段代码在集合上使用`invoke()`函数调用每个项目的`name()`方法，同时收集结果。然而，这只是一个函数。集合类别中还有很多我们需要的但我们根本不使用的功能。让我们看看我们如何使用更细粒度的 Lo-Dash 函数依赖项：

```js
var collection = [
    { name: 'Susan', age: 57, enabled: false },
    { name: 'Marcus', age: 45, enabled: true },
    { name: 'Ray', age: 25, enabled: false },
    { name: 'Dora', age: 19, enabled: true }
];

require([
    'lib/lodash-amd/collection/filter',
    'lib/lodash-amd/function/partial'
], function(filter, partial) {
    function valid(age, item) {
        return item.enabled && item.age >= age;
    }

    filter(collection, partial(valid, 25));
    // → [ { age: 45, enabled: true, name: "Marcus" } ]

});
```

正如您在深入类别时可以看到的，对于您想要包含的任何特定功能，都有一个特定的模块。前面的代码使用了两个 Lo-Dash 函数。`filter()`函数来自集合类别，而`partial()`函数来自函数类别。这两个函数都直接作为回调参数传递。由于这两个函数模块本身定义为 AMD 模块，它们只需要工作所需的内部依赖项。这意味着我们只要求我们需要的，这可能在某些上下文中只是一件或两件函数，例如我们前面的例子。

这种细粒度级别的一个缺点是，如果您不确定需要什么，您将不得不不断修改您的模型依赖项列表。Lo-Dash 有很多东西可以提供，并且最好尽可能使用 Lo-Dash 函数。例如，如果您正在包装值并将函数调用链接在一起，那么事先知道您将需要哪些函数是很困难的。因此，获取整个 API 可能是一个好主意，这样就没有可能您想要使用的某些东西不在那里。考虑以下代码：

```js
var collection = [
    { name: 'Allan', age: 29, enabled: false },
    { name: 'Edward', age: 43, enabled: false },
    { name: 'Evelyn', age: 39, enabled: true },
    { name: 'Denise', age: 34, enabled: true }
];

require([ 'lib/lodash-amd/main' ], function(_) {
    _(collection)
        .filter('enabled')
        .sortBy('age')
        .reverse()
        .map('name')
        .value();
    // → [ "Evelyn", "Denise" ]
});
```

这与我们的第一个例子相似，其中我们要求整个 Lo-Dash API。这里的区别在于这是一个 AMD 构建，因此我们要求`main` Lo-Dash 模块，它包括我们需要的所有内容。另一个区别是，采用这种方法，在主 RequireJS 配置文件中不需要设置路径或 shim。

# jQuery

jQuery 无疑是历史上最成功和最广泛使用的技术之一。它的出现是因为浏览器的不一致性；约翰·雷斯（John Resig）决定对此采取行动。与其让应用程序开发者维护自己的代码来处理所有这些平凡的跨浏览器问题，为什么不让他们让 jQuery 来处理这些事情呢？

jQuery 经过多年的发展，得益于成千上万的用户和贡献开发者希望使前端开发不那么令人畏惧。随着时间的推移，它改变了自己的一些方面，并添加了新功能，以跟上其所在环境的变化。

可以肯定地说，jQuery 已经改变了前端开发的方式，并且由于其广泛的采用，将继续这样做。今天存在的几个网络标准都根植于 jQuery。Lo-Dash 在许多方面与 jQuery 相似。虽然它远不如 jQuery 成熟，但它正在迅速被采用。像 jQuery 一样，Lo-Dash 起源于解决 Underscore.js 中存在的跨浏览器问题和性能问题的努力。Lo-Dash 的开发工作已经远远超出了 Underscore.js 的简单替换，并将肯定影响未来的 JavaScript 规范。

因此，jQuery 和 Lo-Dash 在有效性方面相似。它们之间的区别在于它们为程序员解决的具体问题。让我们更仔细地看看这些问题，看看这两个库是否可以相互补充。

## Lo-Dash 面临的挑战

Lo-Dash 是一个低级框架，它在语言级别上增强 JavaScript。低级是一个相对术语。并不是说 Lo-Dash 没有任何抽象；它有很多。只是前端开发不仅仅是 JavaScript。Lo-Dash 不擅长那些其他事情，也不是为了这个目的而设计的。

虽然你可以使用 Lo-Dash 编写更好的代码，但这只是战斗的一半。独立的 JavaScript 代码并不能让你走得很远。如果你正在开发一个应用程序，在某个时候，你最终会接触到 DOM。你将进行 API 调用以加载你的应用程序数据并改变服务器端资源的状态。你还将处理这些调用的异步性质和用户事件，同时注意性能并防止泄漏。

前端开发是一项复杂的任务，但 Lo-Dash 在 JavaScript 的所有方面都表现得非常出色。编写简洁、易读且性能良好的代码正是 Lo-Dash 的强项。这通常意味着你的前端代码的核心。对于其他所有事情，还有其他库，例如 jQuery。

## jQuery 面临的挑战

jQuery 对程序员如此有吸引力的一个原因是它的入门门槛低。任何构建网站的人都可以立即学习和从 jQuery 中受益，通常只需要一两天。同时，它足够强大，可以从一个基本网站扩展到强大的网络应用程序。DOM 遍历和操作是 jQuery 擅长的，但它也能处理复杂的 Ajax 调用、DOM 事件和异步回调。

这些都是 Lo-Dash 缺乏支持的领域。再次强调，这是故意的。这两个库服务于不同的目的。然而，它们也是互补的，通常在同一个应用程序中并肩作战，执行各自的职责。jQuery 没有提供一组工具来帮助程序员在响应 Ajax 请求、用户事件等所有这些回调函数中。这不是它的用途。你可以自由地使用你喜欢的任何库来增强核心应用程序的业务逻辑，而 Lo-Dash 就是这样的选择之一。

Lo-Dash 和 jQuery 的专注特性使我们能够清晰地分离关注点。jQuery 允许 Lo-Dash 程序员专注于创建高质量的函数式代码。我们已经看到了如何使用 RequireJS 与 Lo-Dash 结合来生成模块化组件。现在让我们看看如何将 Lo-Dash 与 jQuery 结合使用。

## 将 jQuery 实例用作集合

也许，jQuery 最常见的用例是查询 DOM 元素。结果是 jQuery 对象，它非常类似于数组。我们可以运用 Lo-Dash 的知识将这些实例视为集合。例如，让我们比较 jQuery 的 `map()` 函数与 Lo-Dash 的 `map()` 函数：

```js
var i = 1000;
console.time('$');
while (i--) {
    $('li').map(function() {
        return $(this).html();
    });
}
console.timeEnd('$');
i = 1000;
console.time('_');
while (i--) {
    _.map($('li'), function(item) {
        return $(item).html();
    });
}
console.timeEnd('_');
// → 
// $: 64.127ms
// _: 27.434ms
```

这两种方法的映射输出完全相同。即使是代码差异也是微不足道的。差异仅在于循环性能——由于实现上的差异，Lo-Dash 的 `map()` 函数将始终优于 jQuery 的 `map()` 函数。

### 注意

下一章将更详细地介绍为什么迭代式 Lo-Dash 函数以这种方式执行。

性能提升并不算很大。这里那里多几毫秒又如何呢？前面的例子只找到了几个元素，测试重复了 1,000 次。在生产环境中，你可能会处理更大的查询结果，迭代超过 1,000 次，随着时间的推移，毫秒开始累积。

jQuery 的 `map()` 函数的性能是否存在根本问题？绝对没有。如果它有效，就使用它。这种变化本身不会让用户感到惊喜。另一方面，如果你是 Lo-Dash 程序员，你会用它来发挥它的长处。Lo-Dash 非常擅长遍历集合。jQuery 非常擅长查询 DOM，并且它仍然承担这个责任。那么实现这个改进的代码成本是多少？基本上为零。

## 函数绑定

在上一节中，我们看到了 Lo-Dash 和 jQuery 重叠的领域。我们选择 Lo-Dash 方法，因为它在责任角度（遍历集合）和实现成本角度（代码几乎相同）上都是合理的。另一个重叠的领域是函数绑定。jQuery 提供了将函数绑定到特定上下文中的工具，但 Lo-Dash 拥有更好的函数式工具。让我们再次比较这两种方法：

```js
function boundFunction(result, item) {
    return result + this.multiplier * item;
}

var scope = { multiplier: 10 },
    collection = _.range(1, 1000),
    jQueryBound = $.proxy(boundFunction, scope),
    lodashBound = _.bind(boundFunction, scope);

console.time('$');
console.log(_.reduce(collection, jQueryBound));
console.timeEnd('$');

console.time('_');
console.log(_.reduce(collection, lodashBound));
console.timeEnd('_');
// → 
// 4994991
// $: 3.214ms
// 4994991
// _: 0.567ms
```

这两种方法都将收集过程简化为相同的结果。代码基本上是相同的；唯一的区别是传递给 `reduce()` 的回调函数的绑定方式。我们将函数绑定到的上下文是一个具有 `multiplier` 属性的普通对象，当回调函数运行时，会查找这个属性。它是通过引用 `this` 来查找的，这就是为什么我们必须在将回调函数传递给 `reduce()` 函数之前绑定上下文的原因。

第一种方法使用 `proxy()` jQuery 函数，而第二种方法使用 Lo-Dash 的 `bind()` 函数。与前面 `map()` 的例子一样，性能优势属于 Lo-Dash，实现它没有成本，而且这是 Lo-Dash 设计得很好的功能之一。所以如果你正在将回调传递给 jQuery 事件函数，`bind()` 和 `proxy()` 一样可行，而且是在 Lo-Dash 擅长的范围内。

## 与 jQuery 延迟实例一起工作

我们已经看到 Lo-Dash 如何在 jQuery 查询 DOM 元素后帮助迭代这些元素。我们也看到了 Lo-Dash 如何在我们的 jQuery 代码中改进函数绑定。让我们反过来看看 jQuery 如何帮助我们编写 Lo-Dash 代码：

```js
function query(coll, filter, sort) {
    var deferred = $.Deferred(),
        _coll = _(coll).filter(filter);

    if (sort) {
        _coll.sortBy(_.isBoolean(sort) ? undefined : sort);
    }

    if (_.size(coll) > 5000) {
        _.defer(function() {
            deferred.resolve(_coll.value());
        });
    } else {
       deferred.resolve(_coll.value());
    }

    return deferred.promise();
}

var collection = _.map(_.range(_.random(10000)), function(item) {
    return {
        id: item,
        enabled: !!_.random()
    };
}), resultSize;

console.log('Collection size: ' + _.size(collection));
query(collection, 'enabled', true).done(function(result) {
    resultSize = _.size(result);
    console.log('Result size: ' + resultSize);
});

if (!resultSize) {
    console.log('Awaiting results...');
}
// → 
// Collection size: 9071
// Awaiting results...
// Result size: 4635
```

在这里，我们正在使用 `Deferred` jQuery 对象。这是由执行异步操作的功能返回的。一旦调用者拥有延迟实例，它就充当调用者和函数之间的通道。当函数完成其异步工作后，它会通知调用者并运行回调函数。我们可以对延迟实例执行许多技巧，但在这里我们将保持简单。

我们实现的 `query()` 函数的职责是将集合包装在 Lo-Dash 包装器中，并使用 `filter` 参数进行过滤。如果提供了 `sort` 参数，我们也会对集合进行排序。异步工作发生在我们检查集合大小时。请注意，我们实际上还没有执行任何链式函数调用，因为我们还没有调用 `value()`。如果集合包含超过 5,000 个项目，我们使用 Lo-Dash 的 `defer()` 函数在执行 `value()` 之前清除 JavaScript 调用栈。如果集合包含较少的项目，我们立即执行过滤器。

调用 `value()` 的结果通过 jQuery 延迟实例传递给调用者，使用 `resolve()` 函数。这个函数的优点是它总是对调用者异步。即使我们处理的是较小的集合，它仍然被视为异步。输出说明了当我们的随机集合有超过 5,000 个项目时，过滤器是延迟的。当我们看到“等待结果”的消息时，这意味着在查询执行之前，控制权已经返回给调用者。这是这个想法；由于集合很大，我们让其他事情先发生，以防过滤器需要一段时间才能完成。

# 背骨

与 jQuery 不同，Backbone 是一个关注于为应用程序创建高级抽象的库。例如模型、集合和视图等概念，Backbone 程序员会扩展这些概念以提供与 API 数据的无缝集成。

Backbone 认识到自己的优势，并利用其他库如 jQuery 和 Underscore 来实现某些功能，例如获取和保存数据。这对于 jQuery 来说是非常适合的，因为它在 DOM 中渲染视图。对于更底层的任务，Backbone 利用 Underscore 的能力。由于 Backbone 利用这些库，它能够保持较小的代码体积。此外，由于它遵循简单的模式，它基本上不会干扰开发者，让他们能够根据特定用例调整库。

围绕 Backbone 有一个完整的生态系统，Lo-Dash 是这个生态系统的一部分。由于它最初被构想为 Underscore 的直接替代品，Lo-Dash 与 Backbone 集成得非常紧密。在本节中，我们将探讨在 Backbone 应用程序中替换 Underscore 以及通过 Underscore 中找不到的功能扩展 Backbone 的能力。

## 替换 Underscore.js

Backbone 需要 jQuery 和 Underscore。由于它被封装为 UMD 函数，如果我们把 Lo-Dash 定义为 AMD 模块，那么替换它相当简单。让我们看看一个用 RequireJS 配置替换 Underscore 为 Lo-Dash 的例子：

```js
require.config({
    paths: {
        jquery: 'lib/jquery.min',
        underscore: 'lib/lodash.backbone.min'
    },
    shim: {
        underscore: { exports: '_' }
    }
});
```

当 Backbone 模块加载时，它会查找 `jquery` 和 `underscore` 模块，这两个模块我们在这里都提供了。你会注意到 `underscore` 模块指向 `lodash.backbone.min`。这是一个特殊的 Lo-Dash 构建，它只包含 Backbone 所需的函数。换句话说，它不包含 Backbone 内部不使用的任何额外内容。现在让我们定义一个简单的模型：

```js
define([
    'underscore',
    'lib/backbone'
], function(_, Backbone) {
    return Backbone.Model.extend({
        parse: function(data) {
            return _.extend({
                name: data.first + '' + data.last
            }, data);
        }
    });
});
```

你可以看到我们要求 `underscore`，实际上就是 Lo-Dash，因此我们可以使用 `extend()` 函数。Backbone 模型也将内部使用 Lo-Dash，因为它有相同的 Underscore 需求。现在让我们使用这个模型：

```js
require([ 'modules/backbone-model' ], function(Model) {
    new Model({
        first: 'Lance',
        last: 'Newman'
    }, { parse: true }).toJSON();
    // → {name: "Lance Newman", first: "Lance", last: "Newman"}
});
```

你之所以会将 Lo-Dash 以这种方式集成到 Backbone 中，主要原因是 Lo-Dash 相比 Underscore 在速度和一致性方面的提升。

## 全功能的 Lo-Dash 和 Backbone

随着我们的应用程序变得更加复杂，我们可能会需要更多 Lo-Dash 功能，尽管这并不是 Backbone 的严格要求。另一种选择是用 Lo-Dash 的完整版本替换 Underscore。为此，我们可以使用 Lo-Dash 的 AMD 构建。以下是一个修改后的 RequireJS 配置示例：

```js
require.config({
    paths: {
        jquery: 'lib/jquery.min',
        underscore: 'lib/lodash-amd/main'
    }
});
```

这个配置与之前的配置类似，只不过它不需要一个垫片来导出 `_` 符号，并且它指向 `main` Lo-Dash 模块。让我们使用这种方法重新定义我们的模型：

```js
define([
    'lib/lodash-amd/object/assign',
    'lib/backbone'
], function(assign, Backbone) {
    return Backbone.Model.extend({
        parse: function(data) {
            return assign({
                name: data.first + '' + data.last
            }, data);
        }
    });
});
```

在这个版本的模型定义中，我们不需要 Underscore。实际上，我们只需要除了 Backbone 之外的一个特定的 Lo-Dash 函数——`assign()`。请注意，Backbone 仍然会加载整个 Lo-Dash API。

## 增强集合和模型

如果我们要求 Lo-Dash 的完整版本，我们可以相当容易地扩展 Backbone 集合的功能。让我们定义一个扩展模块，该模块将不在 Underscore 中找到的方法扩展到 Backbone 集合中：

```js
define([
    'lib/backbone',
    'lib/lodash-amd/array/slice',
    'lib/lodash-amd/array/takeRight',
    'lib/lodash-amd/array/dropWhile'
], function(Backbone, slice, takeRight, dropWhile) {

    function extendCollection(func, name) {
        Backbone.Collection.prototype[name] = function() {
            var args = slice(arguments);
            args.unshift(this.models);
            return func.apply(null, args);
        }
    }

    extendCollection(takeRight, 'takeRight');
    extendCollection(dropWhile, 'dropWhile');

    return Backbone;
});
```

此模块在需要时，将两个新方法 `takeRight()` 和 `dropWhile()` 扩展到 `Backbone.Collection` 原型上。请注意，它返回 Backbone，所以每次我们要求 Backbone 时，我们都可以使用此模块并获取扩展版本作为结果。让我们看看这个扩展集合的使用情况：

```js
require([
    'lib/lodash-amd/collection',
    'modules/backbone-extensions'
], function(_, Backbone) {

    function name(model) {
        return model.get('name');
    }

    var collection = new Backbone.Collection([
        { name: 'Frank' },
        { name: 'Darryl' },
        { name: 'Stacey' },
        { name: 'Robin' }
    ], { comparator: name });

    _.map(collection.takeRight(2), name );
    // → [ "Robin", "Stacey" ]

    _.map(collection.dropWhile(function(model, index, coll) {
        return index < (coll.length - 2);
    }), name);
    // → [ "Robin", "Stacey" ]

});
```

如你所见，集合现在有了 `takeRight()` 和 `dropWhile()` 方法——由于这些函数已经由 Lo-Dash 实现，所以很容易添加。我们只需要将这些部分粘合在一起，就像 Backbone 对 Underscore 函数所做的那样。

# Node.js

在本章的结尾部分，我们将把注意力转向编写用于后端的 Lo-Dash 代码。这当然意味着将 Lo-Dash 作为 Node.js 包安装。

## 安装 Lo-Dash 包

假设你已经安装了 Node，因为你必须这样做才能在 RequireJS 示例中运行 `grunt` 命令，你应该在你的系统上有一个 `npm` 命令。如果是这样，安装将非常简单：

```js
npm install -g lodash

```

这将在全局范围内安装 Lo-Dash，这意味着它对任何希望使用它的其他 Node 项目都是可访问的。这可能是好主意，因为毕竟 Lo-Dash 是一个库。为了验证安装成功，你可以运行以下命令：

```js
node -e "require('lodash');"

```

如果你看到一个长的错误消息，这意味着在安装时出了问题。如果它静默存在，那么你就设置好了。

## 创建一个简单的命令

为了与 Lo-Dash 开发一起深入了解 Node.js，让我们创建一个简单的命令，该命令对逗号分隔的输入进行排序：

```js
var _ = require('lodash'),
    args = _(process.argv),
    input;

if (args.size() < 3) {
    console.error('Missing input');
    process.exit(1);
} else if (args.contains('-h')) {
    console.info('Sorts the comma-separated input');
    console.info('Use "-d" for descending order');
    process.exit(0);
}

input = _(process.argv[2].replace(/\s?(,)\s?/g, '$1').split(','))
    .sortBy();

if (args.contains('-d')) {
    input.reverse();
}

console.log(input.join(', '));
```

`args` 变量是一个 Lo-Dash 包装器，其中包含命令参数作为值。我们将在这个包装器上调用 `size()` 和 `contains()` 来验证输入。然后创建第二个包装器并存储在 `input` 变量中。这是一个以逗号分隔的列表，我们将字符串分割并删除任何多余的空白。然后我们调用 `sortBy()` 来排序列表，如果设置了 `-d` 标志，则可选地反转顺序。然后将字符串重新连接起来。调用 `join()` 实际上会执行函数调用链，这是命令的输出。

## 自定义 Lo-Dash 构建

另一个安装 Node.js 的好理由是你可以安装 `lodash-cli` 包，这是 Lo-Dash 的构建系统。使用这个工具，你可以实时地以细粒度创建自定义构建。甚至可以指定到函数级别，使用以下命令来指定包含或排除的内容：

```js
lodash modularize include=function

```

这将为我们运行 Lo-Dash 的 AMD 构建，只包括 `function` 类别中的函数所必需的内容。

# 摘要

本章重点介绍了在更广泛的前端开发背景下使用 Lo-Dash。借助 RequireJS 等技术，实现模块化变得更加容易。我们探讨了多种实现方式，Lo-Dash 在这些环境类型上提供了内置支持。我们了解到 Lo-Dash 是一个非常专注的库，帮助开发者编写干净且高效的代码，同时忽略其他事物。Lo-Dash 不擅长的地方则被其他稳定的库如 jQuery 和 Backbone 优雅地覆盖。我们还编写了一些 Lo-Dash 代码，这些代码直接帮助这些库，无论是在性能方面还是在功能方面。

我们以对 Node.js 的探讨结束本章，以及如何为运行在浏览器之外的程序编写 Lo-Dash 代码。还有一个用于构建 Lo-Dash 的 Node 包，你可以自定义这些构建以包含你喜欢的任何内容。现在我们已经从外部了解了作为一个 Lo-Dash 程序员你能做什么，让我们来看看 Lo-Dash 的内部。了解某些事物是如何和为什么被设计出来的，将更好地帮助你做出决策，以实现更好的性能。
