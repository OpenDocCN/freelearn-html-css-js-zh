# 第三章：与函数一起工作

你会在足够大的 JavaScript 代码块中到处找到函数。这是因为它们被当作任何其他原始类型一样对待。在 JavaScript 中，一切都是对象，包括函数。函数有一个上下文和原型，并且可以被分配到新的上下文和变量中。

Lo-Dash 帮助最佳利用函数。在缺少部分的情况下，Lo-Dash 提供的实用工具让我们能够编写真正优雅的函数式代码。本章深入探讨这些实用工具。无论我们是改变`this`的含义还是装饰现有的函数，我们都会通过示例说明如何开始。

在本章中，我们将涵盖以下主题：

+   绑定函数上下文

+   装饰函数

+   函数约束

+   定时执行

+   组合和柯里化函数

# 绑定函数上下文

每个 JavaScript 函数都有一个上下文。如果你来自面向对象的语言，函数上下文与一个方法所属的对象非常相似。当然，区别在于 JavaScript 并不在面向对象的概念意义上对对象进行分类。相反，函数绑定到一个默认上下文，并且这个上下文可以很容易地在运行时改变。甚至有内置的语言机制来实现这一点。

Lo-Dash 使改变函数上下文变得容易。在用 Lo-Dash 编程时，我们经常需要处理函数上下文。现在我们将探讨几种处理和改变函数上下文的方法。

## 改变`this`关键字

在函数内部，执行上下文通过`this`关键字来引用。这是一个我们不需要声明的特殊绑定。它始终在给定的函数作用域内可用。重要的是要记住，如果调用者决定覆盖`this`的含义，那么这完全取决于调用者。

`bind()`函数是一种强大的方式，可以构建一个永久绑定到指定上下文的新函数。以下是如何`bind()`工作的初步了解：

```js
function sayWhat() {
    return 'Say, ' + this.what;
}   

var sayHello = _.bind(sayWhat, {
    what: 'hello'
});

var sayGoodbye = _.bind(sayWhat, {
    what: 'goodbye'
}); 

sayHello();
// → "Say, hello"

sayGoodbye();
// → "Say, goodbye"
```

之前的代码定义了一个通用的`sayWhat()`函数，它根据函数的上下文格式化字符串消息。特别是，它寻找上下文的`what`属性。接下来我们使用`bind()`根据`sayWhat()`定义两个新的函数。`sayHello()`函数绑定到一个新的上下文，而`sayGoodbye()`函数绑定到另一个上下文。`bind()`的第二个参数是函数绑定时成为`this`的对象。我们可以看到，每个上下文都定义了一个独特的`what`属性值，并且这在调用这两个函数的输出中得到了反映。

### 注意

没有 Lo-Dash，我们将依赖于函数的`call()`、`apply()`或`bind()`方法来改变其上下文。Lo-Dash `bind()`实现的优点是它性能更好，因为它能够比原生方法进行更好的优化。

`sayWhat()` 函数没有使用任何参数。但仅仅因为我们正在玩弄上下文并不意味着我们绑定的函数不能接受参数。许多函数同时使用调用者传递的参数和上下文对象。具有自定义上下文的函数确实可以接受参数。它们也可以在绑定到新的上下文后使用额外的参数调用，如下面的代码所示：

```js
function sayWhat(what) {
    if (_.isUndefined(what)) {
        what = this.what;
    }   
    return 'Say, ' + what;
}   

var sayHello = _.bind(sayWhat, {
    what: 'hello'
});

var sayGoodbye = _.bind(sayWhat, {}, 'goodbye'),
    saySomething = _.bind(sayWhat, {});

sayHello();
// → "Say, hello"

sayGoodbye();
// → "Say, goodbye"

saySomething('what?');
// → "Say, what?"
```

`sayWhat()` 函数接受一个 `what` 参数，用于构建字符串消息。如果未提供此参数，则回退到上下文的 `what` 属性。现在我们定义了三个新的函数，它们都具有独特的上下文和参数约束。`sayHello()` 函数与前面的例子没有区别；`what` 值在上下文中。`sayGoodbye()` 函数定义传递了 `bind()` 的第三个参数。在上下文对象之后，`bind()` 将接受任何数量的参数，这些参数也被绑定到函数上，但以不同的方式。这被称为**部分应用**，我们将在本章后面讨论这一点。函数始终绑定到上下文，同时也绑定到参数值。最后，`saySomething()` 函数绑定到一个缺少 `what` 属性的上下文。此外，它也没有绑定到任何 `what` 参数。然而，当函数被调用时，仍然可以提供 `what` 参数，就像这里的情况一样。

## 绑定方法

JavaScript 中本身没有方法——只有函数和上下文。然而，这并不妨碍程序员遵循更传统的面向对象模型。

如果我们将函数分配给对象属性，那么该对象就成为函数的上下文。这只是默认行为，正如前节所示，上下文可以改变。然而，函数所属的对象，作为默认上下文，很好地映射到方法和封装。`bindAll()` 函数可以帮助强制这种映射：

```js
function bindName(name) {
    return _.bind(name, {
        first: 'Becky',
        last: 'Rice'
    }); 
}   

var object = { 
    first: 'Ralph',
    last: 'Crawford',
    name: function() {
        return this.first + ' ' + this.last;
    }   
};

var name = bindName(object.name);

object.name();
// → "Ralph Crawford"

name();
// → "Becky Rice"

_.bindAll(object);

name = bindName(object.name)

name();
// → "Ralph Crawford"
```

让我们逐步分析这个实验。目标是说明一旦将 `bindAll()` 函数应用于对象，该对象的所有方法都将绑定到它，并且上下文不能改变。首先，`bindName()` 函数只是接受另一个函数并将其绑定到 `Becky` 上下文。我们将在稍后用它来证明一个观点。

`object` 变量包含一个具有一些简单属性和简单方法的普通对象。`name` 变量是使用 `bindName()` 函数定义的函数。请注意，我们正在将 `object.name()` 方法赋予一个新的上下文。我们放入 `result` 对象中的值证实了这一点。接下来是调用 `object` 上的 `bindAll()`。从这一点开始，`name()` 方法的上下文不能再改变——它被固定在 `object` 上。然后我们通过再次尝试将其绑定到新的上下文来证明这一事实，但 `bindAll()` 强制了上下文。

### 注意

当使用 `bindAll()` 时，你可能会无意中破坏应用程序中的其他功能。改变函数上下文的能力是一种优势，而不是弱点。只有在你绝对确定方法上下文永远不会改变时才使用 `bindAll()`。如果方法上下文不应该改变时改变的可能性很小，那么没有必要使用 `bindAll()`。

名称 `bindAll()` 表明这是一个全有或全无的操作，但实际上并非如此。我们不必强制每个附加到对象上的方法的上下文。我们实际上可以指定方法名称作为第二个参数，并且只有这些方法被绑定到对象上下文中。以下示例说明了这一点：

```js
function getName() {
    return this.name;
}   

var object = { 
    name: 'My Bound Object',
    method1: getName,
    method2: getName,
    method3: getName
};

_.bindAll(object, [ 'method1', 'method2' ]); 

var method3 = _.bind(object.method3, {
    name: 'New Context'
}); 

object.method1();
// → "My Bound Object"

object.method2();
// → "My Bound Object"

method3();
// → "New Context"
```

我们可以看到，`bindAll()` 的调用指定了只有 `method1` 和 `method2` 被绑定到 `object` 上。稍后，我们实际上尝试将 `method3` 绑定到一个全新的上下文，并且它按预期工作。如果我们没有将 `bindAll()` 调用限制为特定方法，我们就无法改变 `method3` 的上下文。

## 动态方法

方法也可以懒绑定到对象上。我们可以使用 `bindKey()` 函数构建一个新的函数，该函数将在给定的对象上调用给定的方法名称。该方法实际上在调用 `bindKey()` 之前不必存在。这就是懒绑定部分。如果你需要将方法作为回调分配，但不确定该方法是否已经存在，这会很有用。以下是一个示例：

```js
function workLeft() {
    return 65 - this.age + ' years';
}   

var object = { 
    age: 38
};  

var work = _.bindKey(object, 'work');

object.work = workLeft;

work();
// → "27 years"
```

这里有一个具有 `age` 属性的对象。我们还有一个 `workLeft()` 函数，它根据上下文的 `age` 属性计算一个数字。我们可以直接将这个函数分配给 `work` 属性，但我们已经使用 `bindKey()` 函数构建了一个新的函数，当调用时将引用 `work()` 方法。需要注意的是，我们能够在 `object` 中 `work()` 方法存在之前构建这个回调函数。它稍后会被添加。它也可以被替换为不同的实现，并且仍然会调用适当的方法。

### 注意

当 `bindKey()` 创建的函数最终被调用时，绑定的键必须存在。否则，你会得到一个 `TypeError`。

就像使用 `bind()` 函数将函数绑定到上下文一样，我们在管理参数方面仍然拥有自由度。也就是说，当调用绑定的函数时，我们可以绑定参数值或提供参数值，如下面的代码所示：

```js
function workLeft(retirement, period) {
    return retirement - this.age + ' ' + period;
}   

var collection = [ 
    { age: 34, retirement: 60 },
    { age: 47 },
    { age: 28, retirement: 55 },
    { age: 41 }
];

var functions = [], 
    result = []; 

 _.forEach(collection, function(item) {
    functions.push(_.bindKey(item, 'work', item.retirement ?
        item.retirement : 65));
}); 

_.forEach(collection, function(item) {
    _.extend(item, { work: workLeft }); 
}); 

_.forEach(functions, function(item) {
    result.push(item('years'));
});
// → 
// [
//   "26 years",
//   "18 years",
//   "27 years",
//   "24 years"
// ]
```

`workLeft()` 函数依赖于几个参数和上下文的 `age` 属性。接下来，我们定义了一组对象和几个空数组来执行我们的实验。现在我们有三个 `forEach()` 迭代，展示了参数如何与 `bindKey()` 一起工作。

第一次迭代是遍历集合，并在其中应用 `bindKey()` 函数以生成 `work()` 方法。我们可以看到，集合中的不是每个对象都有 `retirement` 属性值。如果没有，我们将其绑定为 `65` 作为参数值。此时，我们有一个函数数组，每个函数都绑定到其对象的 `work()` 方法。第二次迭代填充集合中每个对象的 `work` 属性，因此现在 `work()` 成为一个可调用的函数。

最后一次迭代使用另一个参数调用每个这些绑定方法函数。

# 装饰函数

装饰器的作用正如其名，它为函数添加额外的功能。它就像是对功能的一种装饰。例如，假设我们已经实现了一个在某种结构中查找数据的函数。这个函数已经在我们的应用程序中得到了使用，但现在我们正在实现一个新的组件，它需要这个相同的功能以及一些额外的功能。我们可以使用 Lo-Dash 提供的函数装饰工具来扩展现有的函数。

Lo-Dash 函数装饰有两种类型：**部分函数**，它构建具有原始函数部分参数的新函数，以及**包装器**，它构建一个新函数，该函数将原始函数包裹在一个全新的函数中。

## 部分函数

要使用 Lo-Dash 创建部分函数，您使用 `partial()` 函数。生成的函数将预先提供一些参数——在调用时我们不需要再次提供。这个概念在我们需要动态向函数提供参数，尤其是在将其传递到那些参数不可用的新上下文之前时非常有用。以下示例也展示了回调函数的情况：

```js
function sayWhat(what) {
    return 'Say, ' + what;
}

var hello = _.partial(sayWhat, 'hello'),
    goodbye = _.partial(sayWhat, 'goodbye');

hello();
// → "Say, hello"

goodbye();
// → "Say, goodbye"
```

`sayWhat()` 函数根据提供的字符串参数构建一个简单的字符串。接下来的两个 `partial()` 调用提供了这个参数。当 `hello()` 和 `goodbye()` 函数被调用时，它们将使用各自的局部参数调用 `sayWhat()`。

如本章中迄今为止所看到的，许多处理函数的 Lo-Dash 函数返回新的函数。它们也支持调用者传递的参数。这很有价值，因为向我们的函数添加新参数不需要更改我们的函数绑定，如下所示：

```js
function greet(greeting, name) {
    return greeting + ', ' + name;
}   

var hello = _.partial(greet, 'hello'),
    goodbye = _.partial(greet, 'goodbye');

hello('Fran');
// → "hello, Fran"

goodbye('Jacob');
// → "goodbye, Jacob"
```

上一段代码中的 `greet()` 函数接受两个参数，`greeting` 和 `name`。`hello()` 和 `goodbye()` 函数被构建为部分函数，它们使用已提供的第一个参数调用 `greet()`。稍后，当这些函数被调用时，我们可以提供更具体的参数——`name`。

如果上下文特定的参数是第一个函数参数，我们还能否让部分函数的调用者提供名称？为了回答这个问题，我们转向 `partialRight()` 函数：

```js
function greet(name, greeting) {
    return greeting + ', ' + name;
}   

var hello = _.partialRight(greet, 'hello'),
    goodbye = _.partialRight(greet, 'goodbye');

hello('Brent');
// → "hello, Brent"

goodbye('Alison');
// → "goodbye, Alison"
```

这段代码看起来与上一个示例相似，但有一个重要的区别。`greet()` 函数期望 `name` 参数作为第一个参数。我们希望调用者能够指定这个值，但我们还想指定 `greeting` 作为部分参数。`partialRight()` 函数与 `partial()` 函数的工作方式相同，除了它以不同的顺序传递参数给函数。

部分函数不仅限于我们自己的函数。我们可以利用这种简写来对抗 Lo-Dash 功能本身。如果你需要在回调中运行 Lo-Dash 函数，例如，你可以构造一个新的部分函数，该函数重新定义了 Lo-Dash 函数，并预先提供了参数。这在上面的代码中有所展示：

```js
var collection = [ 
    'Sheila',
    'Kurt',
    'Wade',
    'Kyle'
];

var random = _.partial(_.random, 1, collection.length),
    sample = _.partial(_.sample, collection);

random();
// → 4

sample();
// → "Wade"
```

在这里，我们有一个简单的集合和两个操作该集合的部分函数。首先，我们利用 `random()` Lo-Dash 函数，提供部分参数作为范围。然后我们利用 `sample()` 函数，提供要采样的集合作为部分参数。

## 函数装饰器

我们可以利用 `wrap()` 函数来装饰值或另一个函数，使其具有特定的行为。与所有其他 Lo-Dash 函数辅助函数一样，使用 `wrap()` 的一个优点是，生成函数的调用者可以通过参数提供更多数据，如下面的代码所示：

```js
function strong(value) {
    return '<strong>' + value + '</strong>';
}   

function regex(exp, val) {
    exp = _.isRegExp(exp) ?
        exp : new RegExp(exp);
    return _.isUndefined(val) ?
        exp : exp.exec(val);
}   

var boldName = _.wrap('Marianne', strong),
    getNumber = _.wrap('(\\d+)', regex);

boldName();
// → "<strong>Marianne</strong>"

getNumber('abc123')[1];
// → "123"
```

第一个函数 `strong()` 将值包裹在 `<strong/>` 标签中。第二个函数 `regex()` 稍微复杂一些。它将值包裹在一个 `RegExp` 实例中。但它足够智能，只有在值是字符串的情况下才这样做——如果它已经是一个正则表达式，就没有必要创建另一个。此外，如果提供了第二个参数，它将对该值执行正则表达式，并返回结果。

调用 `boldName()` 的结果不言自明。值 `'Marianne'` 被包裹在 `strong()` 函数中。`getNumber()` 函数是包裹了一个看起来像数字的正则表达式字符串的结果。然而，对 `getNumber()` 的调用提供了一个额外的参数，即调用提供了要对其执行正则表达式的字符串。这就是为什么我们使用调用后的数字索引来访问结果。

让我们把注意力转向使用 `wrap()` 装饰现有函数以添加新功能：

```js
var user = _.sample([
    'Scott',
    'Breanne'
]);

var allowed = [
    'Scott',
    'Estelle'
];  

function permission(func) {
    if (_.contains(allowed, user)) {
        return func.apply(null, _.slice(arguments, 1));
    }   
    throw new Error('DENIED');
}   

function echo(value) {
    return value;
}   

var welcome = _.wrap(echo, permission);

welcome('Yo there!');
```

这里的基本思想是使用权限检查能力来装饰 `echo()` 函数。如果 `user` 变量存在于 `allowed` 数组中，`permission()` 函数将只调用传递给它的函数。如果不满足这种情况，将引发异常。反复运行此代码将随机生成拒绝错误。这完全取决于 `'Breanne'`（她不在 `allowed` 数组中）是否被采样为当前用户。

# 函数约束

类似于用新行为装饰函数的是对函数施加的约束。这影响了函数何时以及多久可以调用。函数约束还控制了通过调用函数返回的值的缓存方式。Lo-Dash 有处理这些场景的函数。

## 限制调用次数

有两个 Lo-Dash 函数处理给定函数被调用的次数计数。`after()` 函数将在组合函数被调用指定次数后执行回调。`once()` 函数将给定的函数限制为只调用一次。让我们看看 `after()` 并了解它是如何工作的：

```js
function work(value) {
    progress();
}   

function reportProgress() {
    console.log(++complete + '%');
    progress = complete < 100 ?
        _.after(0.01 * collection.length, reportProgress) :
        _.noop;
}   

var complete = 0,
    collection = _.range(9999999),
    progress = _.noop;

reportProgress();

_.forEach(collection, work);
// →
// 1%
// 2%
// 3%
// …
```

`work()` 函数是一个虚构的函数，实际上它除了调用 `progress()` 以外什么都不做，这通知世界进度已经有所进展。一个真正做了工作的函数会在完成工作后调用 `progress()`。接下来，我们有一个 `reportProgress()` 函数。它负责记录进度。它还使用 `after()` 创建了 `progress()` 函数。直到 `complete` 变量达到 100%，它将再次调用 `reportProgress()`，这会重新定义 `progress()` 函数。`after()` 函数将在 `progress()` 被调用 `x` 次之后调用它所提供的回调函数。在这种情况下，`x` 是集合长度的 1%。

总结来说，`reportProgress()` 定义了 `progress()` 函数。这个函数被需要通知世界其进度的工作函数调用。在 `progress()` 被调用多次之后，会调用 `reportProgress()`。这是记录进度和重新定义 `progress()` 的地方。

所有这些操作都是通过创建一个相当大的集合并遍历它来实现的，在遍历过程中调用 `work()`。但在迭代开始之前，我们通过调用 `reportProgress()` 来启动进度跟踪器。这个代码的一个优点是，在跟踪进度和执行工作之间有一个关注点的分离。工作函数只需要担心调用 `progress()`。`reportProgress()` 只关心定期记录进度，而不关心实际的工作内容。

异步操作也可以使用 `after()`。前面的例子明确调用了由 `after()` 创建的函数。但是，如果我们想在几个异步回调函数触发后同步发生的事情，该怎么办？让我们使用以下代码来找出答案：

```js
function process(coll, callback) {
    var sync = _.after(coll.length, callback);
    _.forEach(coll, function() {
        setTimeout(sync, _.random(2000));
    }); 
    console.log('timeouts all set');
}   

process(_.range(5), function() {
    console.log('callbacks completed');
});
// →
// timeouts all set
// callbacks completed  
```

首先，我们有一个 `process()` 函数，它的目的是象征一个长时间运行的后台异步过程——换句话说，就是运行在后台的过程。这个函数接收两个参数：一个集合和一个回调函数。`callback` 是一个在集合上的每个异步操作完成后被调用的函数。我们通过使用 `after()` 创建一个新的 `sync()` 函数来实现这一点。集合长度被传递给 `after()`。这意味着在 `sync()` 被调用五次之后，即我们的集合长度，回调函数将被调用。

接下来，我们随机选择一个超时时间并调用 `sync()`——这是异步部分。在所有超时都设置完毕后，我们记录 `sync()` 的调用已被安排。当这些操作完成后，执行的回调记录一个基本消息。

有时候，只调用一次函数是有用的。除此之外，它只是无用的重复——无害，但不是必要的。因此，一个函数的一个有用的约束可能是只允许它被调用一次。但我们如何强制执行这样的规则呢？这可以通过以下代码来完成：

```js
function getLeader(coll) {
    return _.first(_.sortBy(coll, 'score').reverse());
}   

var collection = [ 
    { name: 'Dana', score: 84.4 },
    { name: 'Elsa', score: 44.3 },
    { name: 'Terrance', score: 55.9 },
    { name: 'Derrick', score: 86.1 }
];

var leader = _.once(getLeader);

leader(collection);
// → { name: "Derrick", score: 86.1 }
```

在此代码中，`getLeader()` 函数接收一个集合并返回领导者的名字，根据的是 `score` 属性。我们使用这个函数来构建 `leader()` 函数。使用 `once()`，我们告诉 `leader()` 函数只调用一次 `getLeader()`，并且只调用一次。你无法阻止调用者对这些函数进行 50 次调用。`once()` 函数的职责是封装传递给它的函数，存储第一次调用的返回值。如果这个值被设置，它将被缓存以供后续调用。因此，前面的代码假设集合是不变的，领导者将始终相同。

## 缓存值

之前的例子展示了使用 Lo-Dash 缓存值的第一个例子。如果函数被限制只能调用一次，那么存储第一次调用的值可能就足够了。这几乎是一种副作用式的缓存——有一个更明确的办法，使用 `memoize()` 函数。显式缓存对于数学函数特别有用，因为给定相同的输入，总是产生相同的输出。这也被称为 **引用透明性**。以下是一个例子：

```js
function toCelsius(degrees) {
    return (degrees - 32) * 5 / 9;
}   

function toFahrenheit(degrees) {
    return degrees * 9 / 5 + 32; 
}   

var celsius = _.memoize(toCelsius),
    fahrenheit = _.memoize(toFahrenheit);

toCelsius(89).toFixed(2) + ' C';
// → "31.67 C"

celsius(89).toFixed(2) + ' C'; 
// → "31.67 C"

toFahrenheit(23).toFixed(2) + ' F'; 
// → "73.40 F"

fahrenheit(23).toFixed(2) + ' F';
// → "73.40 F"
```

在这里，我们有两个简单的数学函数，它们是 **memoization** 的良好候选。`toCelsius()` 函数接收给定的华氏度数并返回相应的摄氏度值。`toFahrenheit()` 函数是它的逆函数——它接收摄氏度参数并返回华氏值。然后我们用 `memoize()` 包装这两个函数，得到两个新的函数，`celsius()` 和 `fahrenheit()`。

之后，我们连续对同一个函数进行两次调用。第一次调用计算值并存储它。第二次调用返回缓存的值而不进行计算，但这个缓存查找是如何工作的？缓存的函数是如何知道使用缓存中的值而不是计算一个值的？让我们通过以下代码来找出答案：

```js
function toCelsius(degrees) {
    return (degrees - 32) * 5 / 9;
}   

function toFahrenheit(degrees) {
    return degrees * 9 / 5 + 32; 
}   

function convertTemp(degrees, system) {
    return system.toUpperCase() === 'C' ?
        toFahrenheit(degrees).toFixed(2) + ' F' :
        toCelsius(degrees).toFixed(2) + ' C'; 
}   

var convert = _.memoize(convertTemp, function(degrees, system) {
     return degrees + system;
}); 

convert(89, 'F');
convert(89, 'F');
convert(23, 'C');
convert(23, 'C');
```

默认情况下，由 `memoize()` 生成的函数将使用提供的第一个参数作为缓存键。缓存是一个简单的对象，通过属性键来查找值。在先前的例子中，缓存函数只接受一个参数。这是可以的，但在接受多个参数的更复杂函数中，你需要一种方法来解析查找键，正如前面例子所示。

这基本上是前面例子的重写，因为它生成了相同的结果。我们仍然有 `toCelsius()` 和 `toFahrenheit()` 函数，但我们引入了一个新的 `convertTemp()` 函数。这个函数接受两个参数：`degrees` 和代表这些度数的温度 `system`。基于这些参数值，我们可以适当地调用 `toCelsius()` 或 `toFahrenheit()`。

我们接着构建 `convert()` 函数，它是 `convertTemp()` 的缓存版本。你会注意到传递给 `memoize()` 的第二个函数构建并返回一个缓存键。如果没有它，缓存键仍然只会根据第一个参数的值进行咨询，这会导致返回错误的数据。请小心。

### 注意

你可能已经注意到，我们可以继续使用之前缓存的函数，`celsius()` 和 `fahrenheit()`。这意味着多层缓存，这听起来实际上相当酷。抵制这种做法的诱惑。如果你执行得足够糟糕，以至于需要多层缓存，那么是时候在更高层次上重新考虑设计。

# 定时执行

从本质上讲，JavaScript 代码是同步执行的，也就是说，你没有多个控制线程，每个线程运行你代码的一部分，并争夺 CPU 的注意力。现代浏览器中有 Web Workers，但这些还远未普及，并且与你在其他语言中找到的线程 API 相似度不高。这一切的好处是，作为程序员，你不需要担心同步原语和与多线程编程相关的所有其他讨厌的细节。

相反，你面临的是不同类型的困难，因为你必须处理事件、DOM 和其他形式的回调；这就是同步代码的局限性。有时，这实际上是期望的。例如，你需要等待预定的时间后才能发生某些事情。或者，也许你想要更新 DOM，然后从上次离开的地方继续。Lo-Dash 提供了一些工具，帮助你处理在调用定时函数和应对副作用时的棘手细节。

## 延迟函数调用

`delay()` 函数用于在给定数量的毫秒数过去后执行一个指定的回调函数。这实际上与内置的 `setTimeout()` 函数的工作方式相同。以下代码展示了这一点：

```js
function poll() {
    if (++cnt < max) {
        console.log('polling round ' + (cnt + 1));
        timer = _.delay(poll, interval);
    } else {
        clearTimeout(timer);
    }   
}   

var cnt = -1, 
    max = 5,
    interval = 3000,
    timer;

poll();
// →
// polling round 1
// polling round 2
// polling round 3
// polling round 4
// polling round 5 
```

这段代码定义了一个 `poll()` 函数，用于定期记录我们正在进行的轮询哪一轮。轮询是前端中常用的模式，用于同步来自 API 的数据与用户正在查看的内容。我们已将控制轮询迭代次数的 `max` 变量设置为 `5`。`interval` 变量设置为 `3000` 毫秒。它控制轮询调用的频率。你可以看到，`poll()` 函数首先检查我们是否已经达到最大迭代次数。如果没有，通过调用 `delay()`，`timer` 变量获得一个超时值——只是一个整数。`delay()` 回调是 `poll()`。如果我们已经达到阈值，超时将被清除，并且不再进行进一步的轮询调度。

如果你仔细观察，你会发现使用 `delay()` 和内置的 `setTimeout()` 函数之间没有区别。两者都接受回调函数和持续时间作为参数，并且两者都返回一个可以清除的超时编号 `clearTimeout()`。与 `setTimeout()` 相比，`delay()` 的有趣之处在于它们如何处理参数。让我们看看参数是如何处理的：

```js
function sayHi(name, delay) {
    function sayHiImp(name) {
        console.log('Hi, ' + name);
    }   
    if (_.isUndefined(delay)) {
        _.delay(sayHiImp, 1, name);
    } else {
        _.delay(sayHiImp, delay, name);
    }   
}   

sayHi('Jan');
sayHi('Jim', 3000);
// →
// Hi, Jan
// Hi, Jim
```

在这里，我们创建了一个 `sayHi()` 函数。这个函数在调用的 `sayHiImp()` 函数内部有一个嵌套函数，这是实际的实现。`sayHi()` 函数只是 `sayHiImp()` 的包装器。它记录给定的 `name` 参数并检查是否提供了 `delay` 参数；如果没有，它提供一个默认的 `delay` 值。重要的是我们的函数要么始终同步运行，要么异步运行，但绝不能两者兼而有之。然而，如果有 `delay` 值，我们使用 `delay()` 函数来推迟对 `sayHiImp()` 的调用。注意，我们还把 `name` 参数传递给 `delay()` 调用。我们不需要构建自己的部分函数，而是让 `delay()` 为我们构建一个。

## 延迟函数调用

当 JavaScript 代码在浏览器中运行时，它会启动一个被称为 **调用栈** 的过程。大多数编程语言都共享调用栈的概念。它可以被看作是一个可追踪的函数调用图，从根调用开始。有趣的是，JavaScript 调用栈和 DOM 是两个完全独立的实体，它们共享相同的控制线程。这意味着在存在活动的 JavaScript 调用栈时，DOM 不会运行。这就是为什么长时间运行的 JavaScript 代码会锁定 UI 交互性的原因。

使用 `defer()` 函数是解决那些可能需要一段时间（在这里，“一段时间”是一个相对术语——2 秒就是一段时间）的函数的场景的解决方案。你可以将对该函数的调用推迟到调用栈清除之后，以下代码展示了这一点：

```js
function expensive() {
    _.forEach(_.range(Math.pow(2, 25)), _.noop);
    console.log('done');
}   

_.defer(expensive);
console.log('computing...');
// →
// computing...
// done  
```

`expensive()`函数什么都不做，只是占用 CPU 一段时间，防止`console.log()`调用执行。因此，我们使用`defer()`来调用`expensive()`，它等待当前调用栈完成。`'computing...'`字符串作为调用栈中的最后一个语句被记录。不久之后，`'done'`字符串出现在控制台日志中。这个技巧是我们给 DOM 一个更新机会，在执行昂贵的代码之前。

另一种在调用栈清除后每次想要调用某个函数时调用`defer()`的方法是创建一个包装函数。然后，你可以像调用任何其他函数一样调用这个包装函数，它会为你处理延迟。这是通过以下代码实现的：

```js
function deferred(func) {
    return _.defer.apply(_, ([ func ])
        .concat(_.slice(arguments, 1)));
}   

function setTitle(title) {
    console.log('Title: "' + title + '"');
}   

function setState(app) {
    console.log('State: "' + app.state + '"');
}   

var title = _.wrap(setTitle, deferred),
    state = _.wrap(setState, deferred),
    app = { state: 'stopped' };

title('Home');
state(app);
app.state = 'started';
// →
// Title: "Home"
// State: "started"
```

这里有两个函数，`setTitle()`和`setState()`，我们希望它们都是可延迟的。第一个函数接受一个`title`参数并记录它。第二个函数接受一个`app`对象并记录该对象的`state`属性。`deferred()`函数是一个包装器。我们将使用它和`wrap()`一起使任何函数可延迟。`deferred()`所做的只是将`defer()`应用于传递给它的函数和一些参数。

接下来，你可以看到`title()`函数是`setTitle()`的延迟版本，而`state()`函数是`setState()`的延迟版本。我们还有一个初始状态为`'stopped'`的`app`对象。调用`title()`和`state()`将始终在调用栈清除后延迟执行。这一点在前面的代码中通过在调用`state()`之后将状态设置为`started`进一步说明。你可以猜测哪个字符串会被记录。

## 节流函数调用

通常，DOM 中的事件触发频率可能会比你能够处理的频率要高得多。简单地移动鼠标指针就有可能在每秒生成数百个事件。如果每个事件都有一个处理程序，并且该处理程序执行任何有意义的操作，UI 将会滞后。无论处理器有多快，都无法跟上。唯一跟上节奏的方法是忽略大多数这些事件，并且只在一定频率下做出响应。这个想法在以下代码中得到了说明：

```js
var el = document.querySelector('#container'),
    onMouseMove = _.throttle(function(e) {
        console.log('X: ' + e.clientX + ' Y: ' + e.clientY);
    }, 750);

el.addEventListener('mousemove', onMouseMove);
window.addEventListener('hashchange', function cleanup() {
    el.removeEventListener('mousemove', onMouseMove);
    window.removeEventListener('mousemove', cleanup);
});
```

`el`变量是我们想要监听`mousemove`事件的 DOM 元素。`onMouseMove`函数是通过将一个函数传递给`throttle()`创建的。这个回调函数简单地记录鼠标坐标。我们还向`throttle()`传递`750`，因为这是这个回调函数允许运行的最大频率。接下来，我们绑定事件并设置清理操作，在完成时移除监听器。如果我们没有节流`onMouseMove()`，你会在`console.log()`的详细程度中看到明显的差异。

## 防抖函数调用

**去抖动**函数与节流函数类似。区别在于等待时间结束后会发生什么。使用`throttle()`，函数总是会被调用。例如，如果将节流函数的`wait`值设置为`10`毫秒，并且在这 10 毫秒内调用了函数，它将在下一次等待之前被调用。使用`debounce()`，在 10 毫秒的等待期间，如果调用了函数，它将额外等待 10 毫秒。让我们看看一些去抖动代码：

```js
function log(msg, item) {
    console.log(msg + ' ' + item);
}

var debounced = _.debounce(_.partial(log, 'debounced'), 1),
    throttled = _.throttle(_.partial(log, 'throttled'), 1),
    size = 1500;

_.forEach(_.range(size), debounced);
_.forEach(_.range(size), throttled);
```

我们有一个简单的`log()`函数，它记录一条消息和一个项目编号。然后我们继续构建`debounced()`和`throttled()`函数的版本。然后我们通过相同大小的循环运行这两个版本。有什么区别？输出看起来像这样：

```js
throttled 0 
throttled 1
throttled 744 
debounced 1499
throttled 1499
```

这里发生了什么？我们将`debounced()`和`throttled()`的`wait`时间都设置为`1`毫秒。在处理`1500`个项目所需的时间内，`throttled()`函数的等待期过去了两次。一旦发生这种情况，就调用了`log()`函数，因此产生了输出。注意，`debounce()`的输出只有在处理完成后才发生。这是因为`debounce()`在 1 毫秒的等待期间被多次调用，并在下一次等待期间再次调用。

### 注意

`throttle()`函数实际上在底层使用了`debounce()`。所有的复杂性都在`debounce()`中，它接受几个配置选项。其中之一是执行的**前导**和**尾随**边缘。这意味着什么？你会在前面的输出中注意到，`throttled()`函数是在`debounce()`之后被调用的。这是等待期的尾随边缘。等待期的前导边缘是在等待期开始之前。这两个边缘对于`throttle()`默认都是`true`。这意味着你在一个紧张的循环中，你的节流函数正在被猛烈地打击，函数立即被调用，在等待下一次调用之前。然后，如果循环突然结束，当等待期结束时，函数再次被调用。

# 函数组合和柯里化

本章的最后部分是关于组装函数，这些函数通过较小的函数实现更大的行为。组装此类函数有两种方法。第一种是使用名为`compose()`的适当命名的函数，它执行提供的函数的嵌套调用，或者在顺序很重要的情况下，我们可以使用`flow()`函数一起返回值。柯里化允许你根据不同的上下文连续调用函数。这些 Lo-Dash 工具中的每一个都让你能够以有趣的方式在你的应用程序中构建现有的功能。

## 函数组合

`compose()` 函数从提供的函数中构建一个新的函数。当我们调用这个新函数时，会开始一个嵌套的函数调用，即最后提供的函数会使用任何额外的参数被调用。然后返回的值会被传递给下一个函数，依此类推，最终为调用者产生一个值。以下示例更好地解释了这一点：

```js
function dough(pizza) {
    if (_.isUndefined(pizza)) {
        pizza = {};
    }   
    return _.extend({
        dough: true
    }, pizza);
}   

function sauce(pizza) {
    if (!pizza.dough) {
        throw new Error('Dough not ready');
    }   
    return _.extend({
        sauce: true
    }, pizza);
}   

function cheese(pizza) {
    if (!pizza.sauce) {
        throw new Error('Sauce not ready');
    }   
    return _.extend({
        cheese: true
    }, pizza);
}   

var pizza = _.compose(cheese, sauce, dough);

pizza();
// → { cheese: true, sauce: true, dough: true }
```

有三个函数负责组装披萨——`dough()`、`sauce()` 和 `cheese()`。每个函数的职责是在提供的 `pizza` 对象上设置相应的属性为 `true`。`pizza()` 函数是通过这些函数组合而成的，这些函数又使用 `compose()` 函数。因此调用 `pizza()` 将会调用 `cheese(sauce(dough()))`。注意这些函数中发生的一些检查。例如，`dough()` 可以接受一个对象或构造一个新的对象。然而，如果没有 `dough` 属性，`sauce()` 函数将无法工作。同样，如果没有 `sauce`，`cheese()` 函数会抱怨。

### 注意

虽然能够组合函数很方便，但进行先决条件检查是个好主意。这样它们会快速失败，因此其他尝试从你的函数中组合出东西的开发者会有一个明显的指示，如果某件事不可能的话。

如果函数调用的反向顺序让你感到困惑，不要担心。我们可以使用 `flow()` 函数来反转顺序。使用相同的 `pizza` 函数，我们可以对 `pizza()` 组合函数进行轻微的修改：

```js
var pizza = _.flow(dough, sauce, cheese);

return pizza();
```

### 注意

`compose()` 函数实际上是 `flowRight()` 函数的别名。`flow()` 和 `flowRight()` 函数较新。在 Lo-Dash 的早期版本中，`compose()` 函数是独立的。

## 柯里化函数

你是否曾经发现自己不得不创建一大堆变量，这些变量除了最终传递给函数之外没有任何作用？与变量创建不同，柯里化技术允许你部分应用函数。也就是说，你只需调用函数，提供你当时拥有的数据即可。柯里化函数会持续返回函数，直到它拥有所有必要的参数。这个技术将通过以下示例进行解释：

```js
function makePizza(dough, sauce, cheese) {
    return {
        dough: dough,
        sauce: sauce,
        cheese: cheese
    };  
}   

function dough(pizza) {
    return pizza(true);
}   

function sauceAndCheese(pizza) {
    return pizza(true, true);
}   

var pizza = _.curry(makePizza);

sauceAndCheese(dough(pizza));
// → { cheese: true, sauce: true, dough: true }
```

`makePizza()` 函数具有任意数量的三个参数——函数期望的参数数量。这意味着通过在 `makePizza()` 上调用 `curry()` 创建的 `pizza()` 函数会持续返回函数，直到它被三个参数调用。我们可以以任何我们想要的方式传递这些参数。这可以是所有三个参数同时传递，也可以是一个接一个地传递。这意味着不同的上下文可以将数据传递给函数，而无需在其他地方存储它们。

# 摘要

希望在阅读本章之后，你对 JavaScript 中函数的欣赏有所提升。Lo-Dash 让前端中的函数式编程变得更好。JavaScript 中的函数默认是灵活的，例如改变执行上下文。本章向你展示了 Lo-Dash 的一些函数如何通过移除否则可能需要的许多样板代码，使处理函数上下文变得更加容易。部分函数是函数式编程的基础，但在 JavaScript 中，这是一项相当不容易的任务。Lo-Dash 使得创建部分函数和通过包装额外的逻辑来装饰函数变得简单。

我们研究了帮助限制函数何时应该运行的函数。例如，函数是否应该只允许运行一次？返回值是否应该被缓存？函数执行的计时是一个复杂的话题，尤其是当你考虑到 DOM 以及它是如何与 JavaScript 调用栈集成的时候。Lo-Dash 有一系列处理函数定时执行的函数。我们详细地研究了这些函数。

本章以如何将较小的函数组合成较大的功能模块结束。柯里化函数允许你定义足够灵活的函数，可以在多个上下文中调用，从而减少了在传递之前临时存储参数的需求。在此基础上，我们介绍了 Lo-Dash 的基本概念。你到目前为止学到的关于集合、对象和函数的概念，在整个书籍的剩余部分都是适用的。我们现在可以继续学习映射和归约值，这是一种强大的技术，你将在使用 Lo-Dash 编程时反复使用。
