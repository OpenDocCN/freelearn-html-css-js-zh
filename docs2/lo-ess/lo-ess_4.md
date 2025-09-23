# 第四章。使用 Map/Reduce 进行转换

前三章都提到了 Lo-Dash 的转换可能性。无论你是在处理集合、对象还是函数，Lo-Dash 函数的一个常见模式是通过生成一个新的（尽管略有不同）版本来转换输入。转换值的思想是应用编程的核心，其中你编写的大部分代码都是一系列的转换。从本章开始，我们将转换方向，更详细地研究转换。

尤其是我们将探讨所有我们可以使用 Lo-Dash 和 map/reduce 编程模型完成的有趣事情。我们将从基础知识开始，通过一些基本的映射和基本的减少来湿脚。随着我们通过本章的进展，我们将开始介绍更多高级技术，这些技术可以从 map/reduce 的角度来考虑。

一旦你读完本章，目标是牢固地理解可帮助映射和减少集合的 Lo-Dash 函数。此外，你将开始注意到不同的 Lo-Dash 函数如何在 map/reduce 领域中协同工作。准备好了吗？

在本章中，我们将涵盖以下主题：

+   提取值

+   映射集合

+   映射对象

+   减少集合

+   减少对象

+   绑定上下文

+   Map/reduce 模式

# 提取值

我们已经在第一章 *处理数组和集合* 中看到了如何使用 `pluck()` 函数从集合中提取值。考虑这是你对映射的非正式介绍，因为这正是它在做的事情。它接受一个输入集合，将其映射到一个新的集合，只提取我们感兴趣的属性。以下是一个示例：

```js
var collection = [ 
    { name: 'Virginia', age: 45 },
    { name: 'Debra', age: 34 },
    { name: 'Jerry', age: 55 },
    { name: 'Earl', age: 29 }
];  

_.pluck(collection, 'age');
// → [ 45, 34, 55, 29 ]
```

这是一种非常简单的映射操作。实际上，你可以用 `map()` 做同样的事情：

```js
var collection = [ 
    { name: 'Michele', age: 58 },
    { name: 'Lynda', age: 23 },
    { name: 'William', age: 35 },
    { name: 'Thomas', age: 41 }
];

_.map(collection, 'name');
// → 
// [
//   "Michele",
//   "Lynda",
//   "William",
//   "Thomas"
// ] 
```

如你所料，这里的输出与使用 `pluck()` 的输出完全相同。事实上，`pluck()` 实际上是在底层使用 `map()` 函数。传递给 `map()` 的回调是通过 `property()` 构造的，它只返回指定的属性值。当传递一个字符串而不是一个函数给 `map()` 函数时，`map()` 函数会回退到这种提取行为。

在对映射的本质进行简要介绍之后，让我们进一步深入探讨映射集合中可能实现的内容。

# 映射集合

在本节中，我们将探讨映射集合。将一个集合映射到另一个集合的范围从简单的组合——正如我们在上一节中看到的——到复杂的回调。映射集合中每个项目的回调可以包括或排除属性，并可以计算新值。我们还将讨论过滤集合的问题以及如何与映射结合进行。

## 包括和排除属性

当应用于一个对象时，`pick()` 函数会生成一个新的对象，其中只包含指定的属性。与之相反的函数 `omit()` 会生成一个对象，除了指定的属性外，包含所有属性。由于这些函数对单个对象实例工作良好，为什么不使用它们与集合一起使用呢？你可以使用这两个函数通过映射到新的属性来从集合中去除属性，如下面的代码所示：

```js
var collection = [ 
    { first: 'Ryan', last: 'Coleman', age: 23 },
    { first: 'Ann', last: 'Sutton', age: 31 },
    { first: 'Van', last: 'Holloway', age: 44 },
    { first: 'Francis', last: 'Higgins', age: 38 }
];  

_.map(collection, function(item) {
    return _.pick(item, [ 'first', 'last' ]); 
});
// → 
// [
//   { first: "Ryan", last: "Coleman" },
//   { first: "Ann", last: "Sutton" },
//   { first: "Van", last: "Holloway" },
//   { first: "Francis", last: "Higgins" }
// ]
```

在这里，我们正在使用 `map()` 函数创建一个新的集合。提供给 `map()` 的回调函数应用于集合中的每个项目。`item` 参数是集合中的原始项目。回调函数预期返回该项目的映射版本，而这个版本可以是任何东西，包括原始项目本身。

### 注意

在 `map()` 回调中操作原始项目时要小心。如果项目是一个对象并且它在应用程序的其他地方被引用，它可能会产生意外的后果。

在前面的代码中，我们返回一个新的对象作为映射的项目。这是使用 `pick()` 函数完成的。我们只关心 `first` 和 `last` 属性。我们新映射的集合看起来与原始集合相同，只是没有任何项目有 `age` 属性。这个新映射的集合在以下代码中可以看到：

```js
var collection = [ 
    { first: 'Clinton', last: 'Park', age: 19 },
    { first: 'Dana', last: 'Hines', age: 36 },
    { first: 'Pete', last: 'Ross', age: 31 },
    { first: 'Annie', last: 'Cross', age: 48 }
];  

_.map(collection, function(item) {
    return _.omit(item, 'first');
});
// → 
// [
//   { last: "Park", age: 19 },
//   { last: "Hines", age: 36 },
//   { last: "Ross", age: 31 },
//   { last: "Cross", age: 48 }
// ]
```

这段代码与之前的 `pick()` 代码采用完全相同的方法。唯一的区别是我们排除了新创建的集合中的 `first` 属性。你也会注意到我们传递了一个包含单个属性名的字符串，而不是属性名数组。

除了将字符串或数组作为 `pick()` 或 `omit()` 的参数传递之外，我们还可以传递一个函数回调。这在不清楚集合中哪些对象应该具有哪些属性时很合适。在 `map()` 回调中使用这样的回调允许我们对具有非常少的代码的集合执行详细的比较和转换：

```js
function invalidAge(value, key) {
    return key === 'age' && value < 40; 
}   

var collection = [ 
    { first: 'Kim', last: 'Lawson', age: 40 },
    { first: 'Marcia', last: 'Butler', age: 31 },
    { first: 'Shawna', last: 'Hamilton', age: 39 },
    { first: 'Leon', last: 'Johnston', age: 67 }
];

_.map(collection, function(item) {
    return _.omit(item, invalidAge);
});
// → 
// [
//   { first: "Kim", last: "Lawson", age: 40 },
//   { first: "Marcia", last: "Butler" },
//   { first: "Shawna", last: "Hamilton" },
//   { first: "Leon", last: "Johnston", age: 67 }
// ]
```

由这段代码生成的新集合排除了 `age` 属性，对于 `age` 值小于 `40` 的项目。提供给 `omit()` 的回调函数应用于对象中的每个键值对。这段代码是 Lo-Dash 实现简洁性的良好示例。这里运行着大量的迭代代码，而且没有看到任何 `for` 或 `while` 语句。

## 执行计算

现在是时候将我们的注意力转向在 `map()` 回调中执行计算了。这包括查看项目并根据其当前状态计算一个新值，该值最终将被映射到新集合中。这可能意味着扩展原始项目的属性或用新计算出的值替换一个。无论哪种情况，映射这些计算都比编写应用于集合中每个项目的逻辑要容易得多。以下示例解释了这一点：

```js
var collection = [ 
    { name: 'Valerie', jqueryYears: 4, cssYears: 3 },
    { name: 'Alonzo', jqueryYears: 1, cssYears: 5 },
    { name: 'Claire', jqueryYears: 3, cssYears: 1 },
    { name: 'Duane', jqueryYears: 2, cssYears: 0 } 
];  

_.map(collection, function(item) {
    return _.extend({
        experience: item.jqueryYears + item.cssYears,
        specialty: item.jqueryYears >= item.cssYears ?
            'jQuery' : 'CSS'
    }, item);
});
// → 
// [
//   {
//     experience": 7,
//     specialty": "jQuery",
//     name": "Valerie",
//     jqueryYears": 4,
//     cssYears: 3
//   },
//   {
//     experience: 6,
//     specialty: "CSS",
//     name: "Alonzo",
//     jqueryYears: 1,
//     cssYears: 5
//   },
//   {
//     experience: 4,
//     specialty: "jQuery",
//     name: "Claire",
//     jqueryYears: 3,
//     cssYears: 1
//   },
//   {
//     experience: 2,
//     specialty: "jQuery",
//     name: "Duane",
//     jqueryYears: 2,
//     cssYears: 0
//   }
// ]
```

在这里，我们将原始集合中的每个项目映射到其扩展版本。特别是，我们为每个项目计算两个新值——`经验`和`专长`。`经验`属性仅仅是`jqueryYears`和`cssYears`属性的简单总和。`专长`属性是基于`jqueryYears`和`cssYears`属性中较大值的计算结果。

之前，我提到了在`map()`回调中修改项目时需要小心。一般来说，这是一个坏主意。记住`map()`是用来生成新集合的，而不是修改现有集合，这会有所帮助。以下是不小心操作带来的可怕后果的说明：

```js
var app = {}, 
    collection = [ 
        { name: 'Cameron', supervisor: false },
        { name: 'Lindsey', supervisor: true },
        { name: 'Kenneth', supervisor: false },
        { name: 'Caroline', supervisor: true }
    ];  

app.supervisor = _.find(collection, { supervisor: true }); 

_.map(collection, function(item) {
    return _.extend(item, { supervisor: false }); 
}); 

console.log(app.supervisor);
// → { name: "Lindsey", supervisor: false }
```

这个回调的破坏性性质并不明显，对程序员来说几乎不可能追踪和诊断。它实际上是为每个项目重置`supervisor`属性。如果这些项目在应用程序的其他地方被使用，每当这个映射任务执行时，`supervisor`属性值就会被覆盖。如果你需要重置这样的值，确保更改被映射到新值，而不是对原始值进行更改。

映射也可以使用原始值作为项目。通常，我们会有一组原始值数组，我们希望将其转换成另一种表示形式。例如，假设你有一个表示字节的尺寸数组。你可以使用以下代码将这些数组映射到一个新集合中，其中尺寸以可读的值表示：

```js
function bytes(b) {
    var units = [ 'B', 'K', 'M', 'G', 'T', 'P' ],
        target = 0;
    while (b >= 1024) { 
        b = b / 1024;
        target++;
    }   
    return (b % 1 === 0 ? b : b.toFixed(1)) +
        units[target] + (target === 0 ? '' : 'B');
}   

var collection = [ 
    1024,
    1048576,
    345198,
    120120120
];  

_.map(collection, bytes);
// → [ "1KB", "1MB", "337.1KB", "114.6MB" ]
```

`bytes()`函数接受一个数值参数，即要格式化的字节数。这是起始单位。我们只是不断递增`target`单位，直到我们得到一个小于`1024`的值。例如，我们集合中的最后一个项目映射到`'114.6MB'`。`bytes()`函数可以直接传递给`map()`，因为它期望我们的集合中的值是它们当前的值。

## 调用函数

我们不总是需要为`map()`编写自己的回调函数。在合理的地方，我们可以自由地利用 Lo-Dash 函数来映射我们的集合项目。例如，假设我们有一个集合，我们想知道每个项目的大小。我们可以使用`size()` Lo-Dash 函数作为我们的`map()`回调，如下所示：

```js
var collection = [ 
    [ 1, 2 ],
    [ 1, 2, 3 ],
    { first: 1, second: 2 },
    { first: 1, second: 2, third: 3 } 
];  

_.map(collection, _.size);
// → [ 2, 3, 2, 3 ]
```

这段代码的额外好处是`size()`函数返回一致的结果，无论传递给它的参数是什么。实际上，任何接受单个参数并根据该参数返回新值的函数都是`map()`回调的有效候选者。例如，我们也可以映射每个项目的最小值和最大值：

```js
var source = _.range(1000),
    collection = [ 
        _.sample(source, 50),
        _.sample(source, 100),
        _.sample(source, 150)
    ];  

_.map(collection, _.min);
// → [ 20, 21, 1 ]

_.map(collection, _.max);
// → [ 931, 985, 991 ]
```

如果我们想要将集合中的每个项目映射到一个排序版本呢？由于我们不排序集合本身，所以我们不关心项目在集合中的位置，而是关心项目本身，例如，如果它们是数组。让我们看看以下代码会发生什么：

```js
var collection = [ 
    [ 'Evan', 'Veronica', 'Dana' ],
    [ 'Lila', 'Ronald', 'Dwayne' ],
    [ 'Ivan', 'Alfred', 'Doug' ],
    [ 'Penny', 'Lynne', 'Andy' ]
];  

_.map(collection, _.compose(_.first, function(item) {
    return _.sortBy(item);
}));

// → [ "Dana", "Dwayne", "Alfred", "Andy" ]
```

这段代码使用 `compose()` 函数构建一个 `map()` 回调。第一个函数通过传递给 `sortBy()` 来返回项目的排序版本。然后，这个排序列表的第一个项目作为映射项目返回。最终结果是包含我们集合中每个数组按字母顺序排列的第一个项目的新集合，仅用三行代码。不错。

## 过滤和映射

过滤和映射是两个紧密相关的集合操作。过滤只提取那些特别感兴趣的集合项目。映射将集合转换以产生新的集合。但如果我们只想映射集合的某个子集呢？那么将过滤和映射操作链在一起是有意义的，对吧？以下是一个可能看起来像这样的例子：

```js
var collection = [ 
    { name: 'Karl', enabled: true },
    { name: 'Sophie', enabled: true },
    { name: 'Jerald', enabled: false },
    { name: 'Angie', enabled: false }
];  

_.compose(
    _.partialRight(_.map, 'name'),
    _.partialRight(_.filter, 'enabled')
)(collection);
// → [ "Karl", "Sophie" ]
```

这个映射是通过使用 `compose()` 来构建一个立即调用的函数来执行的，我们的 `collection` 作为参数。这个函数由两个部分组成。我们在两个参数上都使用了 `partialRight()`，因为我们希望在这两种情况下提供的集合作为最左边的参数。第一个部分函数是 `filter()`。我们部分应用了 `enabled` 参数。因此，这个函数将在传递给 `map()` 之前过滤我们的集合。过滤集合的结果传递给 `map()`，它部分应用了 `name` 参数。最终结果是包含启用 `name` 字符串的集合。

### 注意

关于前面代码的重要一点是，过滤操作发生在 `map()` 函数运行之前。我们本可以将过滤后的集合存储在一个中间变量中，而不是通过 `compose()` 流程化。无论哪种风格，确保你映射集合中的项目与源集合中的项目相对应是很重要的。通过不返回任何内容来过滤 `map()` 回调中的项目是可行的，但这并不明智，因为这在字面意义上和比喻意义上都不太合适。

# 映射对象

前一节主要关注集合以及如何映射它们。但是等等，对象也是集合，对吧？这确实是正确的，但区分数组和普通对象是值得的。主要原因在于在执行映射/归约操作时，顺序和键有影响。最终，数组和对象在映射/归约方面有不同的用途，本章试图承认这些差异。

现在我们将开始探讨 Lo-Dash 程序员在处理对象并将它们映射到集合时所采用的一些技术。有许多因素需要考虑，例如对象内的键，以及在对象上调用方法。我们将研究键值对之间的关系以及它们如何在映射上下文中使用。

## 处理键

我们可以用有趣的方式来使用给定对象的键，将对象映射到一个新的集合中。例如，我们可以使用`keys()`函数来提取对象的键，并将它们映射到除了属性值之外的其他值，如下面的示例所示：

```js
var object = { 
    first: 'Ronald',
    last: 'Walters',
    employer: 'Packt'
};  

_.map(_.sortBy(_.keys(object)), function(item) {
    return object[item];
});
// → [ "Packt", "Ronald", "Walters" ]
```

上述代码从`object`中构建了一个属性值数组。它是通过使用`map()`来实现的，实际上是将`object`的`keys()`数组进行映射。这些键是通过`sortBy()`进行排序的。因此，`Packt`是结果数组的第一个元素，因为在`object`键中，`employer`是按字母顺序排在第一位的。

有时，执行其他对象的查找并将这些值映射到目标对象是有用的。例如，并非所有 API 都返回给定页面所需的所有内容，打包在一个整洁的小对象中。你必须进行连接并构建所需的数据。以下代码展示了这一点：

```js
var users = {}, 
    preferences = {}; 

_.each(_.range(100), function() {
    var id = _.uniqueId('user-');
    users[id] = { type: 'user' };
    preferences[id] = { emailme: !!(_.random()) };
}); 

_.map(users, function(value, key) {
    return _.extend({ id: key }, preferences[key]);
});
// →
// [
//   { id: "user-1", emailme: true },
//   { id: "user-2", emailme: false },
//   ...
// ]
```

此示例构建了两个对象，`users`和`preferences`。在每个对象的情况下，键是我们使用`uniqueId()`生成的用户标识符。`user`对象中只有一些虚拟属性，而`preferences`对象有一个`emailme`属性，设置为随机的布尔值。

现在假设我们需要快速访问`users`对象中所有用户的这个偏好。如您所见，使用`map()`在`users`对象上实现这一点非常直接。回调函数返回一个包含用户 ID 的新对象。我们通过`key`查找来扩展这个对象，以获取特定用户的偏好。

## 调用方法

对象属性不仅限于存储原始字符串和数字。属性可以存储作为其值的函数，或者称为方法。然而，根据你使用对象的环境，方法并不总是可调用的，特别是如果你对对象使用的环境几乎没有或没有控制权。在这些情况下，一种有用的技术是将调用这些方法的结果进行映射，并在相关环境中使用这个结果。让我们看看以下代码是如何做到这一点的：

```js
var object = { 
    first: 'Roxanne',
    last: 'Elliot',
    name: function() {
        return this.first + ' ' + this.last;
    },  
    age: 38, 
    retirement: 65, 
    working: function() {
        return this.retirement - this.age;
    }   
};  

_.map(object, function(value, key) {
    var item = {}; 
    item[key] = _.isFunction(value) ? object[key]() : value
    return item;
});
// →
// [
//   { first: "Roxanne" },
//   { last: "Elliot" },
//   { name: "Roxanne Elliot" },
//   { age: 38 },
//   { retirement: 65 },
//   { working: 27 }
// ]

_.map(object, function(value, key) {
    var item = {}; 
    item[key] = _.result(object, key);
    return item;
});
// →
// [
//   { first: "Roxanne" },
//   { last: "Elliot" },
//   { name: "Roxanne Elliot" },
//   { age: 38 },
//   { retirement: 65 },
//   { working: 27 }
// ]
```

在这里，我们有一个包含原始属性值和这些属性值使用的方法的对象。现在我们希望映射调用这些方法的结果，我们将尝试两种不同的方法。第一种方法使用`isFunction()`函数来确定属性值是否可调用。如果是，我们调用它并返回该值。第二种方法更容易实现，并达到相同的效果。`result()`函数被应用于对象和当前键。这测试了我们是否正在处理一个函数，因此我们的代码不必这样做。

### 注意

在映射方法调用的第一种方法中，你可能已经注意到我们使用`object[key]()`而不是`value()`来调用方法。前者保留了对象变量的上下文，但后者失去了上下文，因为它作为一个没有任何对象的普通函数被调用。所以当你编写调用方法的映射回调但没有得到预期结果时，确保方法的上下文是完整的。

也许你有一个对象，但你不确定哪些属性是方法。你可以使用`functions()`来找出这一点，然后映射调用每个方法的结果到一个数组，如下面的代码所示：

```js
var object = { 
    firstName: 'Fredrick',
    lastName: 'Townsend',
    first: function() {
        return this.firstName;
    },  
    last: function() {
        return this.lastName;
    }   
};  

var methods = _.map(_.functions(object), function(item) {
    return [ _.bindKey(object, item) ];
}); 

_.invoke(methods, 0);
// → [ "Fredrick", "Townsend" ]
```

`object`变量有两个方法，`first()`和`last()`。假设我们不知道这些方法，我们可以使用`functions()`来找到它们。在这里，我们使用`map()`构建一个`methods`数组。输入是一个包含给定对象所有方法名称的数组。我们返回的值很有趣。它是一个单值数组；你很快就会明白原因。这个数组的值是通过将对象和方法名称传递给`bindKey()`构建的函数。这个函数在调用时，将始终使用`object`作为其上下文。

最后，我们使用`invoke()`来调用`methods`数组中的每个方法，构建一个新的结果数组。回想一下，我们的`map()`回调返回了一个数组。这是一个简单的技巧，让`invoke()`工作，因为它是一种方便调用方法的方式。它通常期望一个键作为第二个参数，但一个数字索引同样有效，因为它们都是通过相同的方式查找的。

## 映射键值对

仅因为你正在处理一个对象，并不意味着它是理想的，甚至可能是必要的。这正是`map()`的作用——将你拥有的映射到你需要的。例如，属性值有时对你所做的事情来说可能就是全部，你可以完全不用键。为此，我们有`values()`函数，并将值传递给`map()`：

```js
var object = { 
    first: 'Lindsay',
    last: 'Castillo',
    age: 51
};  

_.map(_.filter(_.values(object), _.isString), function(item) {
    return '<strong>' + item + '</strong>';
});
// → [ "<strong>Lindsay</strong>", "<strong>Castillo</strong>" ]
```

我们在这里从`object`变量中想要的只是一个属性值的列表，这些值都是字符串，这样我们就可以对它们进行格式化。换句话说，键是`first`、`last`和`age`的事实是不相关的。因此，首先我们调用`values()`来构建一个值数组。接下来，我们将该数组传递给`filter()`，移除任何不是字符串的内容。然后我们将这个输出传递给`map()`，在那里我们可以使用`<strong/>`标签映射字符串。

反过来也可能成立——没有键，值完全没有意义。如果是这样，可能将键值对映射到一个新的集合中是合适的，如下面的示例所示：

```js
function capitalize(s) {
    return s.charAt(0).toUpperCase() + s.slice(1);
}   

function format(label, value) {
    return '<label>' + capitalize(label) + ':</label>' +
        '<strong>' + value + '</strong>';
}   

var object = { 
    first: 'Julian',
    last: 'Ramos',
    age: 43
};  

_.map(_.pairs(object), function(pair) {
    return format.apply(undefined, pair);
});
// →
// [
//   "<label>First:</label><strong>Julian</strong>",
//   "<label>Last:</label><strong>Ramos</strong>",
//   "<label>Age:</label><strong>43</strong>"
// ]
```

我们将运行对象通过 `pairs()` 函数的结果传递给 `map()`。传递给我们的 `map` 回调函数的参数是一个数组，第一个元素是键，第二个是值。碰巧的是，`format()` 函数期望一个键和一个值来格式化给定的字符串，因此我们可以使用 `format.apply()` 来调用该函数，传递给它 `pair` 数组。这种方法纯粹是个人喜好问题。没有必要在 `map()` 之前调用 `pairs()`。我们同样可以直接调用 `format`。但有时，这种方法更受欢迎，原因很多，其中之一就是程序员的风格，这些原因广泛而多样。

# 简化集合

现在是时候看看如何简化集合了。Lo-Dash 在这里也提供了很多帮助，提供了帮助我们简化数组、对象以及任何向我们抛来的东西的函数。除了原始数据类型之外，所有数据结构都可以简化为更简单的东西。

我们将从查看常见的简化情况开始，例如求和值等。接下来，我们将讨论过滤集合的主题以及它与简化的关系。然后，我们将探讨一些更高级的计算技术。

## 求和值

与其他编程语言不同，JavaScript 没有内置机制来对值数组进行求和。我们所能达到的求和程度是原生的 `Array.reduce()` 方法，它实际上是通用的，并不专门用于求和值。Lo-Dash 版本的 `reduce` 更加通用，以下是一个如何在集合中求和值的示例：

```js
var collection = [ 
    { ram: 1024, storage: 2048 },
    { ram: 2048, storage: 4096 },
    { ram: 1024, storage: 2048 },
    { ram: 2048, storage: 4096 }
];  

_.reduce(collection, function(result, item) {
    return result + item.ram;
}, 0);
// → 6144

_.reduce(collection, function(result, item) {
    return result + item.storage;
}, 0);
// → 12288
```

在这里，我们有一个简单的集合，我们将其简化为两个值。第一次调用 `reduce()` 有一个回调函数，它将累加器和当前项的 `ram` 属性相加。第二次 `reduce()` 调用执行相同的事情，但它作用于 `storage` 属性。我们实际上是将集合简化为一个数字，因此得名。你还会注意到，在回调函数之后，我们向 `reduce()` 传递了一个 `0` 值。这就是累加器。正如其名所示，它的任务是随着每个项目通过回调函数而累积数据。这也被称为结果，并且总是作为第一个参数传递给 `reduce` 回调。现在让我们看看不同类型的累加器：

```js
var collection = [ 
    { hits: 2, misses: 4 },
    { hits: 5, misses: 1 },
    { hits: 3, misses: 8 },
    { hits: 7, misses: 3 } 
];  

_.reduce(collection, function(result, item) {
    return {
        hits: result.hits + item.hits,
        misses: result.misses + item.misses
    }; 
}, { hits: 0, misses: 0 });
// → { hits: 17, misses: 16 }
```

这个累加器是一个对象，它初始化两个属性为 `0`。回调函数只是不断返回一个新的累加器对象，其中包含计算出的 `hits` 和 `misses` 的总和。这种方法的副作用之一是，我们只需要一个 `reduce()` 调用而不是两个。然而，累加器并不是必需的。在简单的求和项的情况下，实际上使用它们并没有意义。以下代码展示了这一点：

```js
function add(a, b) {
    return a + b;
}   

var collection = [ 
    { wins: 34, loses: 21 },
    { wins: 58, loses: 12 },
    { wins: 34, loses: 23 },
    { wins: 40, loses: 15 }
];  

_.reduce(_.range(1, 6), add);
// → 15

_.reduce(_.pluck(collection, 'wins'), add);
// → 166

_.reduce(_.pluck(collection, 'loses'), add);
// → 71
```

此示例使用一个通用的 `reduce` 回调函数，该函数返回其两个参数的总和。然后我们有一个基本的对象集合，每个对象有两个属性。`reduce()` 的第一次调用将一个数字数组传递给 `add()` 回调。接下来的两次调用首先使用 `pluck()` 构建一个数字数组，使用它们各自的键名字符串。这些调用使用相同的回调。要注意的是，此代码中没有在 `reduce()` 调用中显式指定累加器。当调用者未指定时，默认值是集合的第一个元素。对于具有这些原始值的数组，这是可以的，实际上可以简化回调函数。

## 过滤和缩减

您并不总是需要或想要将整个集合缩减到单个值。相反，需要一个过滤后的子集。有时，您的代码接收到的集合是应用过滤后的较大集合的结果。或者，您需要应用过滤本身。考虑以下代码：

```js
var collection = [ 
    { name: 'Gina', age: 34, enabled: true },
    { name: 'Trevor', age: 45, enabled: false },
    { name: 'Judy', age: 71, enabled: true },
    { name: 'Preston', age: 19, enabled: false }
];  

_.reduce(_.filter(collection, 'enabled'), function(result, item) {
    result.names.push(item.name);
    result.years += item.age;
    return result;
}, { names: [], years: 0 });
// →
// {
//   names: [
//     "Gina",
//     "Judy"
//   ],
//   years: 105
// }
```

`filter()` 函数仅用于将启用的对象传递给 `reduce()` 调用。这被称为过滤然后缩减。然而，这里可以应用另一种方法。如下面的代码所示：

```js
var collection = [ 
    { name: 'Melissa', age: 28, enabled: true },
    { name: 'Kristy', age: 22, enabled: true },
    { name: 'Kerry', age: 31, enabled: false },
    { name: 'Damon', age: 36, enabled: false }
];  

_.reduce(collection, function(result, item) {
    if (item.enabled) {
        result.names.push(item.name);
        result.years += item.age;
    }   
    return result;
}, { names: [], years: 0 });
// →
// {
//   names: [
//     "Melissa",
//     "Kristy"
//   ],
//   years: 50
// }
```

此方法在回调函数内部执行必要的过滤。这被称为过滤和缩减。如果项目未启用，我们只需返回最后一个结果。如果已启用，我们执行常规的缩减工作。所以这就像我们只是在跳过那些无论如何都会被过滤的项目。这个优点是，对于大型集合，您不需要两次通过集合进行线性操作，而只需一次。缺点是 `reduce` 回调函数中增加了复杂性。但是，无论在哪里可以最小化这种复杂性，例如在前面的例子中，将过滤工作转移到 `reduce` 回调函数以优化您的代码。

## 最小值、最大值和平均值操作

Lo-Dash 提供了帮助进行更复杂操作的功能，同时让您编写干净、简洁的代码。例如，`min()` 和 `max()` 函数接受回调函数，使它们能够在各种情况下使用，如下面的示例所示：

```js
function score(item) {
    return _.reduce(item.scores, function(result, score) {
        return result + score;
    }); 
}   

var collection = [ 
    { name: 'Madeline', scores: [ 88, 45, 83 ] },
    { name: 'Susan', scores: [ 79, 82, 78 ] },
    { name: 'Hugo', scores: [ 90, 84, 85 ] },
    { name: 'Thomas', scores: [ 74, 69, 78 ] } 
];  

_.min(collection, score);
// →
// {
//   name: "Madeline",
//   scores: [
//     88,
//     45,
//     83
//   ]
// }

_.max(collection, score);
// →
// {
//   name: "Hugo",
//   scores: [
//     90,
//     84,
//     85
//   ]
// }
```

在此代码中定义的 `score()` 函数将传入的项目缩减为其 `scores` 属性的总和，假设它是一个数组。这旨在用作 `min()` 和 `max()` 函数的回调。想法是 `score()` 被应用于我们集合中的每个对象，并返回最小值或最大值。所以实际上我们正在进行两个缩减工作，一个用于 `scores` 属性，另一个用于集合。

将集合缩减到平均值稍微复杂一些，因为没有名为 `avg()` 的 Lo-Dash 函数可以将集合缩减到平均值。让我们看看我们是否可以实施一些不需要比前面示例更多代码的方法：

```js
function average(items) {
    return _.reduce(items, function(result, item) {
        return result + item;
    }) / items.length;
}   

var collection = [ 
    { name: 'Anthony', scores: [ 89, 59, 78 ] },
    { name: 'Wendy', scores: [ 84, 80, 81 ] },
    { name: 'Marie', scores: [ 58, 67, 63 ] },
    { name: 'Joshua', scores: [ 76, 68, 74 ] } 
];  

_.reduce(collection, function(result, item, index, coll) {
    var ave = average(item.scores);
    result.push(ave);
    if (index === (coll.length - 1)) {
        return average(result);
    }   
    return result;
}, []).toFixed(2);
// → "73.08"
```

就像本例之前的`scores()`回调函数一样，我们还有一个`average()`函数。这个函数将传入的项目减少到它们的平均值。我们的集合由对象组成，每个对象都有一个`scores`数组。我们感兴趣的是找到整个集合的平均值。因此，我们将对集合调用`reduce()`。回调函数使用`average()`函数计算每个项目的平均分数。然后将这个结果添加到`reduce()`累加器中。如果我们已经到达最后一个项目，通过检查集合长度来完成平均。然后是计算最终平均值的时刻——平均的平均值。由于累加器是一个数字数组，我们可以简单地通过将其传递给`average()`函数来返回生成的值。

# 减少对象

在本节中，我们将关注减少对象和与对象累加器一起工作。减少对象与减少数组非常相似，区别在于你有键而不是索引。哦，对了，还有顺序，这也很重要——数组是有序的，对象则不是。

在本章前面，我们瞥见了累加器是什么。在这里，我们将更深入地研究对象累加器，包括一些利用这个概念的内置 Lo-Dash 函数。

## 减少键

你可以根据对象的键将其减少为不同的内容。例如，如果你只需要某些属性，你可以使用以下代码将对象减少到只包含这些属性：

```js
var object = { 
        first: 'Kerry',
        last: 'Singleton',
        age: 41
    },  
    allowed = [ 'first', 'last' ];

_.reduce(object, function(result, value, key) {
    if (_.contains(allowed, key)) {
        result[key] = value;
    }   
    return result;
}, {});
// → { first: "Kerry", last: "Singleton" }

_.pick(object, allowed);
// → { first: "Kerry", last: "Singleton" }
```

`allowed`数组包含允许的属性键名，我们使用`reduce()`函数来检查给定的键是否允许。如果是允许的，它将被添加到对象累加器中。否则，它将被跳过。你会注意到，我们可以通过将`allowed`数组传递给`pick()`函数来实现相同的效果。所以，在编写自己的回调函数之前，先检查 Lo-Dash 默认做了什么。另一方面，你自己的代码更容易改变。

## 对象累加器

作为`reduce()`函数的替代方案，`transform()`函数用于将源对象转换为目标对象。主要区别在于使用`transform`时，存在一个隐含的累加器。这个累加器对象是在第一次调用`transform()`函数时创建的。然后，它作为引用传递给每个属性的`callback`函数，如下例所示：

```js
var object = { 
    first: '&lt;strong&gt;Nicole&lt;/strong&gt;',
    last: '&lt;strong&gt;Russel&lt;/strong&gt;',
    age: 26
};   

_.transform(object, function(result, value, key) {
    if (_.isString(value)) {
        result[key] = _.unescape(value);
    }   
});
// →
// {
//   first: "<strong>Nicole</strong>",
//   last: "<strong>Russel</strong>"
// }
```

这里有一个具有两个字符串属性的对象。我们传递给`transform()`的`callback`函数寻找字符串属性，并使用`unescape()`替换任何 HTML 字符代码。`result`参数在这里，就像在`reduce()`回调中一样，但我们不需要返回它。我们也不需要提供累加器对象，因为它为我们创建了。让我们更仔细地看看累加器是如何创建的。

### 注意

使用 `transform()` 的缺点是它看起来像是在转换并返回传入的对象，但实际上并非如此。`transform()` 函数不会修改源对象。

假设我们正在转换一个类的实例，而不仅仅是普通对象。这可以通过以下代码完成：

```js
function Person(first, last) {
    this.first = first;
    this.last = last;
}   

Person.prototype.name = function name() {
    return this.first + ' ' + this.last;
};  

var object = new Person('Alex', 'Rivera');

_.transform(object, function(result, value, key) {
    if (_.isString(value)) {
        result[key] = value.toUpperCase();
    }   
}).name();
// → "ALEX RIVERA"
```

`object` 变量持有 `Person` 的一个实例。我们的 `transform()` 回调函数简单地查找字符串并将它们转换为它们的大写等效形式。当我们对转换后的对象调用 `name()` 函数时，我们得到预期的结果。请注意，`name()` 方法是在 `Person` 原型上。`transform()` 函数使用适当的构造函数正确地构建了转换后的实例。这确保了原型方法和属性位于它们应该的位置。

Lo-Dash 有其他一些函数在对象累加器方面与上述函数类似，区别在于源是一个集合而不是一个对象。例如，你可以对集合进行分组或索引项，如下面的代码所示：

```js
var collection = [ 
    { id: _.uniqueId('id-'), position: 'absolute', top: 12 },
    { id: _.uniqueId('id-'), position: 'relative', top: 20 },
    { id: _.uniqueId('id-'), position: 'absolute', top: 12 },
    { id: _.uniqueId('id-'), position: 'relative', top: 20 }
];  

_.groupBy(collection, 'position');
// →
// {
//   absolute: [
//     { id: "id-1", position: "absolute", top: 12 },
//     { id: "id-3", position: "absolute", top: 12 }
//   ],
//   relative: [
//     { id: "id-2", position: "relative", top: 20 },
//     { id: "id-4", position: "relative", top: 20 }
//   ]
// }

_.indexBy(collection, 'id');
// →
// {
//   "id-1": {
//     id: "id-1",
//     position: "absolute",
//     top: 12
//   },
//   "id-2": {
//     id: "id-2",
//     position: "relative",
//     top: 20
//   },
//   "id-3": {
//     id: "id-3",
//     position: "absolute",
//     top: 12
//   },
//   "id-4": {
//     id: "id-4",
//     position: "relative",
//     top: 20
//   }
// }
```

`groupBy()` 函数根据指定属性的值对集合中的项进行分组。也就是说，如果两个相同的项具有相同的 `position` 属性值，它们将一起被分组在同一个对象键下。另一方面，`indexBy()` 将仅将一个项放入给定的键中。因此，这个函数更适合唯一属性，如标识符。如果我们愿意，我们可以传递一个函数来生成值，而不是传递一个 `property` 字符串。`indexBy()` 调用的结果是具有唯一键的对象，我们可以使用这些键来查找项。

# 绑定上下文

你可能并不总是想使用匿名函数或 Lo-Dash 函数作为你的 `map()` 回调。`reduce()` 也是如此。幸运的是，你可以轻松地将回调函数的上下文绑定在这两种情况下。例如，假设你有一个不是全局的应用程序对象。你仍然可以将其作为回调函数的上下文，如下面的代码所示：

```js
var app = { 
    states: [
        'running',
        'off',
        'paused'
    ],  
    machines: [
        { id: _.uniqueId(), state: 1 },
        { id: _.uniqueId(), state: 0 },
        { id: _.uniqueId(), state: 0 },
        { id: _.uniqueId(), state: 2 } 
    ]   
};  

var mapStates = _.partialRight(_.map, function(item) {
    return _.extend({
        state: this.states[item.state]
    }, _.pick(item, 'id'));
}, app);

mapStates(app.machines);
// →
// [
//   { state: "off", id: "1" },
//   { state: "running", id: " " },
//   { state: "running", id: " " },
//   { state: "paused", id: " " }
// ]
```

上述示例使用 `partialRight()` 函数来组合回调函数。我们正在部分应用 `map()` 函数的参数。第一个是回调函数，第二个是函数的上下文，在这个例子中是 `app` 实例。这基本上使得回调函数能够将 `this` 关键字作为应用程序来引用，尽管它不在全局作用域中。

同样的上下文绑定原则也可以应用于 `reduce()` 函数：

```js
var collection = [ 12, 34, 53, 43 ],
    settings = { tax: 1.15 },
    applyTax = _.partialRight(_.reduce, function(result, item) {
        return result + item * this.tax;
    }, 0, settings);

applyTax(collection).toFixed(2);
// → "163.30"
```

在这里，`reduce` 回调是一个部分函数，其上下文是 `settings` 对象。该对象有一个 `tax` 属性，用于通过将每个集合中的项的值乘以累加器来减少集合。然后将此结果添加到累加器中。

# Map/reduce 模式

我们将用一些基本的 map/reduce 模式来结束这一章，这些模式适用于你到目前为止在本章中学到的所有内容。首先，我们将看看通用回调函数是什么样的，以及为什么它们是有用的。然后我们将介绍 map/reduce 链的概念。

## 通用回调函数

随着你前端应用程序的发展，你将开始注意到你的所有 map/reduce 回调函数之间有一些共同之处。换句话说，你可能会将回调的通用方面提取到一个单一的通用回调中。正如你在本章和上一章中看到的，使用 Lo-Dash 部分应用和组合新函数很容易。当你有一系列通用函数想要用作回调时，这特别有帮助。

例如，让我们创建一些通用的 `map()` 回调函数，看看它们如何被使用：

```js
function add(item) {
    var result = _.clone(item);
    result[this.prop] += this.value;
    return result;
}   

function upper(item) {
    var result = _.clone(item);
    result[this.prop] = result[this.prop].toUpperCase();
    return result;
}   

var collection = [ 
    { name: 'Gerard', balance: 100 },
    { name: 'Jean', balance: 150 },
    { name: 'Suzanne', balance: 200 },
    { name: 'Darrell', balance: 250 }
];  

var mapAdd = _.partial(_.map, collection, add),
    mapUpper = _.partial(_.map, collection, upper);

mapAdd({ prop: 'balance', value: 50 }); 
// →
// [
//   { name: "Gerard", balance: 150 },
//   { name: "Jean", balance: 200 },
//   { name: "Suzanne", balance: 250 },
//   { name: "Darrell", balance: 300 }
// ]

mapAdd({ prop: 'balance', value: 100 });
// →
// [
//   { name: "Gerard", balance: 200 },
//   { name: "Jean", balance: 250 },
//   { name: "Suzanne", balance: 300 },
//   { name: "Darrell", balance: 350 }
// ]

mapUpper({ prop: 'name'});
// →
// [
//   { name: "GERARD", balance: 100 },
//   { name: "JEAN", balance: 150 },
//   { name: "SUZANNE", balance: 200 },
//   { name: "DARRELL", balance: 250 }
// ]
```

在这里，我们有两个通用函数，`add()` 和 `upper()`。它们都遵循相似的模式。例如，它们都引用 `this.prop` 属性。因此，它们都是上下文相关的。然而，这是一个优点，而不是缺点。`add()` 回调使用 `this.prop` 来确定要操作哪个属性。`this.value` 属性确定要添加的值。正如我们所见，为这些函数提供上下文很容易，这就是我们如何将这些特定信息传递给回调的方式。`upper()` 回调做同样的事情，但它将现有的属性转换为大写。

`mapAdd()` 和 `mapUpper()` 函数被创建为部分函数，预先提供了集合和通用的回调函数。唯一缺少的是上下文，这将在函数被调用时提供。这意味着这些函数在整个应用程序中都有可能是有用的，当被调用时可以获取新的上下文。

### 注意

就像任何其他编程任务一样，尝试提前创建通用的 map/reduce 回调函数很诱人，也就是说，试图预见你需要类似但略有不同的功能的地方。但事实是，**事后诸葛亮**是一个强大的工具。在你开始重复自己之后，你会发现通用函数在哪里变得有用要容易得多。另一方面，**先见之明**往往会导致概念上有用的回调函数，但实际上并不需要。

适用于 `map()` 回调函数的通用函数的所有想法也适用于 `reduce()` 回调函数。以下是一个例子：

```js
function sum(a, b) {
    return a + b[this.prop];
}   

var collection = [ 
    { low: 40, high: 70 },
    { low: 43, high: 83 },
    { low: 39, high: 79 },
    { low: 45, high: 74 }
];  

var reduceSum = _.partial(_.reduce, collection, sum, 0); 

reduceSum({ prop: 'low' });
// → 167

reduceSum({ prop: 'high' });
// → 306
```

通用 `sum()` 函数返回两个参数的和。然而，它使用 `this.prop` 来确定应该添加哪个属性。然后我们使用 `partial()` 创建 `reduceSum()` 函数。现在我们可以用我们想要的任何上下文调用 `reduceSum()`。

## Map/reduce 链

本章我们将探讨的最后一个模式是 map/reduce 链的概念。这与 Google 介绍的 map/reduce 编程模型密切相关。想法是，对于大型数据集，当您可以将问题分解为一系列映射操作时，解决计算密集型问题更容易。然后这些操作被馈送到一系列减少操作。从这个角度来看，跨节点分布计算要容易得多。

然而，我们感兴趣的是在映射作业和减少作业之间发生的手续转移。映射作业负责将源数据映射为减少作业可以消费的东西。这种模式实际上可能重复多次。例如，数据集被映射然后减少。减少的结果随后进一步映射，再进一步减少，依此类推。让我们看看在 Lo-Dash 中这样的操作看起来是什么样子：

```js
var collection = [ 
    { name: 'Wade', balance: 100 },
    { name: 'Donna', balance: 125 },
    { name: 'Glenn', balance: 90 },
    { name: 'Floyd', balance: 110 }
], bonus = 25; 

var mapped = _.map(collection, function(item) {
    return _.extend({
        bonus: item.balance + bonus
    }, item);
}); 

_.reduce(mapped, function(result, item, index, coll) {
    result += (item.bonus - item.balance) / item.bonus;
    if (index === (coll.length - 1)) {
        result = result / coll.length * 100;
    }   
    return result;
}, 0).toFixed(2) + '%';
// → "19.23%"
```

在这里，我们通过为每个项目添加一个 `bonus` 属性到 `balance` 项来计算金额，从而映射集合。这个新映射的集合存储在 `mapped` 变量中。然后我们将集合减少到平均增长率。请注意，减少回调期望一个映射集合，因为它使用了 `bonus` 属性，而这个属性不在原始集合中。

# 摘要

本章向您介绍了 map/reduce 编程模型以及 Lo-Dash 工具如何帮助您在应用程序中实现它。首先，我们检查了映射集合，包括如何选择包含哪些属性以及如何执行计算。然后我们转向映射对象。键在对象映射到新对象和集合的方式中起着重要作用。在映射时还需要考虑方法和函数。

本章的第二部分涵盖了减少的内容，包括如何求和项，如何转换对象，以及如何制定通用的回调函数，这些函数可以在各种环境中使用。章节以简要介绍将 map/reduce 操作链式连接起来的样子结束。

Map/reduce 是一个重要的话题，因为 Lo-Dash 支持许多编程模型的变体。现在是时候扩展链式概念了，结果发现，不仅仅是 map/reduce 函数可以被粘合在一起。
