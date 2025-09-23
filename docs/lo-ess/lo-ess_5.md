# 第五章。链式组装

到目前为止，本书中我们讨论的例子都是独立使用 Lo-Dash 函数的。这并不意味着它们不能一起工作；只是它们可以更简洁，或者更紧凑。我们会调用一个函数来计算一个值，存储这个值，然后调用另一个函数使用存储的值作为参数来计算一个新的值，并重复这个过程。这很累人，但可以很容易地解决。

理念是将这个功能流线化为一系列调用。这种方法遵循应用编程的概念，即我们有一个起始集合，在链的每个阶段，这个集合都会被转换。它就像一条装配线，最终产品是在特定上下文中需要的值。

Lo-Dash 通过包装器概念实现了这种编程模式——一个构造函数，用于包装原始值，从而实现链式函数调用。在本章中，我们将看到如何使用这种方法简化复杂的代码，甚至产生可重用的组件。

在本章中，我们将涵盖以下主题：

+   创建 Lo-Dash 包装器

+   构建过滤器

+   测试真值条件

+   计数项目

+   转换

+   中间结果

+   键和值

+   返回链

# 创建 Lo-Dash 包装器

在本节中，我们将介绍包装值的理念。然后我们将使用包装器来链式调用函数。我们还将探讨如何终止调用链。

## 链式调用

将函数调用链在一起是应用编程的一种模式，其中集合被转换成不同的东西。这个新转换的集合随后被传递到链中的下一个调用，依此类推。这就是“应用”这个术语的来源；你正在将函数应用于集合中的每个项目。由于这个过程反复进行，很容易将链式调用打包成一个可重用的组件。这是一个在每一步添加、删除或修改值的管道，最终产生一个结果。

另一种可能的方式是对链的更实际的观点，它只是调用函数的一种更简单的方式。jQuery 使这一概念流行起来。当阅读 jQuery 代码时，你会发现有很多链式调用，但代码仍然是可读的。通常，链可以作为一个单独的语句来构建，如下面的代码所示：

```js
$('body')
    .children()
    .first()
    .is('h1');
// → true
```

这个 jQuery 链由四个调用组成，表达为一个单独的语句。第一个调用是 jQuery 构造函数，它包装了指定的 DOM 元素。接下来，我们调用 `children()` 来获取子元素。`first()` 函数返回第一个子元素。链通过调用 `is()` 结束，它返回一个简单的布尔值，而不是 jQuery 对象。

### 注意

注意这里的代码格式。如果你要组合功能链，保持代码的可读性非常重要。我推荐遵循的主要约定是，在下一行缩进链式调用。这样，你不会有不必要的跨多列的语句，并且可以一眼看出这段代码是一个函数调用链。

## 包装值

在 Lo-Dash 中包装值的工作方式基本上与 jQuery 相同。有一个包装器调用构建 jQuery/Lo-Dash 对象。每个**链式**调用返回一个包装器对象。有一个终止调用返回一个原始类型。也有一些明显的区别，关于 jQuery 和 Lo-Dash 如何包装值。例如，你不能传递一个 CSS 选择器字符串给 Lo-Dash 包装器函数，并期望它包装 DOM 元素。你也不希望这样做——Lo-Dash 是一个低级实用程序库，而 jQuery 在根本上是更高层次的抽象。

这并不是包装值和应用函数调用链的全部故事。在每一个角落都有细微的差别和边缘情况，我们将在本章的整个过程中解决这些问题。但就目前而言，让我们进入代码：

```js
_(['a', 'b', 'c'])
    .at([1, 2])
    .value();
// → [ "b", "c" ]
```

这里是我们的 hello-world 链。我们一直在书中使用`_`对象来访问 Lo-Dash API，它也是一个构造函数。它接受一个 JavaScript 原始值作为参数。这就是被包装并传递给链中第一个函数调用的值。在这里，我们调用`at()`函数，表示我们想要索引为`1`和`2`的项。调用`value()`函数得到我们想要的结果。

显然，前面的代码不需要使用包装器——只有一个函数调用。然而，重点不是简洁性，而是调用链的基本结构。随着我们在本章中遇到更复杂的例子，我们将看到链如何显著减少代码量。这里还有两个 Lo-Dash 包装器构造函数：

```js
_({a: 'b', c: 'd'})
    .contains('b');
// → true

_('abcd')
    .contains('b');
// → true
```

第一个包装器使用一个普通对象作为其原始值。第二个包装器使用一个字符串。在两种情况下，链都会立即终止，因为我们调用了`contains()`，它本身返回一个原始布尔值。再次强调，我们不需要使用包装器和调用链来编写前面的代码。如果你只调用一个函数，最好不要这样做，否则你只会让其他阅读你代码的人感到困惑。前面代码的目的在于说明我们可以包装普通对象和字符串，并将它们视为集合。

## 显式和隐式链式调用

一旦我们有了 Lo-Dash 包装器实例，我们就可以开始进行链式函数调用了。然而，并非所有这些函数都是可链式的。**不可链式**的函数返回原始值，例如布尔值或数字。这被称为隐式链式。它是隐式的，因为那些本应返回集合的函数实际上返回了一个 Lo-Dash 包装器实例。其他函数没有集合作为返回值。对这些函数的调用将终止链式调用。

另一方面，存在显式链式调用——这将保持链式调用直到通过调用 `value()` 显式终止。例如，如果你的链式调用是显式的，调用 `contains()` 将返回一个包装器，而不是通常的布尔值。以下是一些隐式和显式链式调用的示例：

```js
_([3,2,1])
    .sort()
    .first();
// → 1

_.chain([3,2,1])
    .sort()
    .first()
    .isNumber()
    .value();
// → true
```

第一个链使用默认的 Lo-Dash 链式配置。`first()` 函数获取数组中的第一个元素并返回它。由于这个元素可以是任何东西（在这个例子中是一个数字），`first()` 函数是不可链式的。我们不需要显式调用 `value()`，因为不可链式的函数返回**未解包**的值。然而，第二个链使用显式链式调用。这是通过使用 `chain()` 函数构造 Lo-Dash 包装器实例来实现的。结果包装器在所有方面都是相同的，除了我们需要显式调用 `value()` 来解包值。在显式链式中，每个函数都是可链式的。例如，`first()` 的调用现在返回一个包装器实例而不是一个数字。这也是通过 `isNumber()` 实现的。

你想要使用这种显式链式调用的主要原因是为了避免在链式调用完成后使用临时变量和额外的语句。例如，在前面代码的显式链式中，我们只需要知道排序集合中的第一个元素是否是数字。如果我们能从链式中直接得到我们想要的，就没有必要将第一个元素存储在一个新变量中。

# 构建过滤器

链式函数调用的强大用途是构建过滤器，依次从较大的集合中过滤掉不需要的项。假设你已经有了一段使用 `filter()` 函数对集合进行操作的代码。但现在你需要修改这个过滤操作，可能需要添加额外的约束。与其去修改你已知可以正常工作的现有 `filter()` 代码，你不如构建一个过滤器链。

## 多次调用 filter()

组装过滤器链的最简单方法是将多个 `filter()` 函数调用连接起来。以下是一个这样的例子：

```js
var collection = [
    { name: 'Ellen', age: 20, enabled: true },
    { name: 'Heidi', age: 24, enabled: false },
    { name: 'Roy', age: 21, enabled: true },
    { name: 'Garry', age: 23, enabled: false }
];

_(collection)
    .filter('enabled')
    .filter(function(item) {
        return item.age >= 21;
    })
   .value();
// → [ { name: "Roy", age: 21, enabled: true } ]
```

`filter()` 的第一次调用使用 `enabled` 属性的 pluck 风格简写，过滤掉该属性为假值的项。下一次 `filter()` 调用使用一个回调函数，过滤掉 `age` 属性值小于 `21` 的项。我们最终只剩下一个项，通过调用 `value()` 来解包。

### 注意

当我们可以直接修改回调函数时，为什么还要调用两个或更多的 `filter()` 呢？这不会意味着更少的代码和更快的执行吗？真正的优势在于阅读和修改此代码。我们想看到当我们移除启用的过滤器时会发生什么吗？只需注释掉该行即可。可读性和可维护性几乎总是应该优于从复杂的回调函数中挤压性能的尝试。当然，这里有一些例外，但不要为了性能而发明性能问题。

## 将 filter() 与 where() 结合使用

`where()` 函数是一种表达性很强的过滤集合的方法，使用逻辑 `and` 条件。与其试图在一个 `filter()` 回调函数中表达所有的查询约束，为什么不利用 `where()` 语法在合适的地方呢？让我们看看它是如何工作的：

```js
var collection = [
    { name: 'Janice', age: 38, gender: 'f' },
    { name: 'Joey', age: 20, gender: 'm' },
    { name: 'Lauren', gender: 'f' },
    { name: 'Drew', gender: 'm' }
];

_(collection)
    .where({ gender: 'f' })
    .filter(_.flow(_.property('age'), _.isFinite))
    .value();
// → [ { name: "Janice", age: 38, gender: "f" } ]
```

此过滤器将包括所有女性项目，并且是 `where()` 函数的良好候选者。接下来，我们想确保所有项目都有一个 `age` 属性，其值是一个有限数字。我们通过将一个回调函数传递给 `filter()` 来做到这一点。在这里，我们使用了一些快捷方式，而不是定义自己的内联回调函数。`flow()` 函数将为我们构造一个函数，让结果流向我们给出的每个函数参数。我们使用 `property()` 函数构建一个函数，获取每个项目的 `age` 属性，并将其传递给 `isFinite()` 函数。在我们的集合中，有几项没有 `age` 属性。这些未定义的值未通过测试，并被过滤掉。

### 注意

连锁过滤函数的顺序可能很重要。例如，首先进行广泛过滤是明智的。这样，当集合通过管道流动时，它会更快地缩小规模，这意味着其他函数的工作量更少。在哪里这很重要并不立即明显，但随着你的代码成熟，你将开始注意到排序调整。代码中连锁结构的优点是，顺序更改是微不足道的。

## 删除和获取集合项目

Lo-Dash 有工具可以让我们从集合的开始或结束处过滤集合。这些工具在函数调用链的上下文中特别有用，因为使用它们通常取决于集合的先前转换。例如，考虑以下代码中的排序顺序：

```js
var collection = [
    { first: 'Dewey', last: 'Mills' },
    { first: 'Charlene', last: 'Larson' },
    { first: 'Myra', last: 'Gray' },
    { first: 'Tasha', last: 'Malone' }
];

_(collection)
    .sortBy('first')
    .dropWhile(function(item) {
        return _.first(item.first) < 'F';
    })
    .value();
// → 
// [
//   { first: "Myra", last: "Gray" },
//   { first: "Tasha", last: "Malone" }
// ]
```

在这个链中的第一个调用使用 `sortBy()` 函数根据 `first` 属性对集合进行排序。现在集合已经排序，我们可以调用 `dropWhile()`。从左侧开始，这个函数会从集合中删除项目，直到回调返回 `true`。我们的特定回调获取名字字符串的第一个字符，如果它小于 `F`，我们就删除它。这使我们只剩下一个只包含以 `F` 开头及以上的名字的集合。

除了从集合的左侧删除项目外，我们还可以从右侧删除项目。或者，我们可以使用链结合两种方法，如下面的代码所示：

```js
var name = '  Donnie Woods   ',
    emptyString = _.partial(_.isEqual, ' ');

_(name)
    .toArray()
    .dropWhile(emptyString)
    .dropRightWhile(emptyString)
    .join('');
// → "Donnie Woods"
```

在这里，我们包裹的是一个字符串值而不是一个数组，模拟了`String.trim()`的功能。因此，我们链的第一个任务就是使用`toArray()`将字符串拆分为单个字符。`drop`函数期望一个数组。接下来，我们使用`dropWhile()`函数并传递它我们的`emptyString()`回调函数。这将从字符串中删除字符，直到找到实际字符。然后使用`dropRightWhile()`从数组的另一端执行相同的任务，但方向相反。最后，我们将数组重新组合成一个字符串，去掉两端被删除的空字符。

### 注意

是的，你可以使用正则表达式和压缩代码得到相同的结果。正则表达式很棒，但它们不是为每个人而设计的，并且它们只适用于字符串。在采取任何方向之前权衡你的选项。

我们可以执行从数组的两端删除项目的逆操作。我们可以取项目，从而删除其他所有内容。例如：

```js
var collection = [
    { name: 'Jeannie', grade: 'B+' },
    { name: 'Jeffrey', grade: 'C' },
    { name: 'Carrie', grade: 'A-' },
    { name: 'James', grade: 'A' }
];

_(collection)
    .sortBy('grade')
    .takeWhile(function(item) {
        return _.first(item.grade) === 'A';
    })
    .value();
// → 
// [
//   { name: "James", grade: "A" },
//   { name: "Carrie", grade: "A-" }
// ]
```

我们只对具有`A`等级的项目感兴趣。我们与`takeWhile()`一起使用的回调函数对具有`A`的项目返回`true`。当然，这仅因为链中的第一步是按`grade`属性对数组进行排序。如果我们没有先做这一步，我们就会错过我们正在寻找的项目。

项目也可以从集合中以相反的方向取出。也就是说，我们不是从左到右移动，而是从右到左移动。这在排序很重要且你不想执行额外的步骤来从集合中取出所需项目时很理想。这种排序在下面的代码中显示：

```js
var collection = _.sample(_.range(1, 21), 10),
    total = 5,
    min = 10;

_(collection)
    .sortBy()
    .takeRightWhile(function(item, index, array) {
        return item >= min &&
            array.length - index <= total;
    })
    .value();
// → [ 13, 14, 15, 17, 20 ]
```

这里使用的集合是 10 个整数的随机抽样。我们链中的第一次调用是`sortBy()`，它只是简单地按无参数提供的数组进行排序。这是升序的，我们想要前五个项目。我们本可以反转排序顺序，但相反，我们跳过了这一步，直接进入`takeRightWhile()`函数。这里使用的回调函数将返回大于`min`的数字，并且只要我们没有超过总数。

## 拒绝项目

拒绝操作与过滤操作非常相似。在过滤的情况下，你知道你想要什么。在拒绝的情况下，你知道你不需要什么。这些拒绝操作可以链接在一起来构建复杂的查询，如下面的代码所示：

```js
var object = {
    first: 'Conrad',
    last: 'Casey',
    age: 37,
    enabled: true
};

_(object)
    .reject(_.isBoolean)
    .reject(_.isString)
    .first()
    .toFixed(2);
// → "37.00"
```

在这里，我们将两个`reject()`调用链在一起。包装的值是一个对象，我们只关心那些不是布尔值或字符串的属性值。这些函数——`isBoolean()`和`isString()`——已经作为 Lo-Dash API 的一部分存在，我们可以直接将它们传递给`reject()`。在这里没有必要编写我们自己的回调函数。

我们可以使用`result()`函数帮助我们链式拒绝集合项。`result()`函数无论指定的属性值是函数还是不可调用值，其工作方式都是相同的。以下是使用`result()`或仅使用属性名调用`reject()`时的差异说明：

```js
function User(name, disabled) {
    this.name = name;
    this.disabled = disabled;
}

User.prototype.enabled = function() {
    return !this.disabled;
};

var collection = [
        new User('Phil', true),
        new User('Wilson', false),
        new User('Kathey', true),
        new User('Nina', false)
    ],
    enabled = _.flow(_.identity,
        _.partialRight(_.result, 'enabled'));

_(collection)
    .reject('disabled')
    .value();
// →
// [
//   { name: "Wilson", disabled: false },
//   { name: "Nina", disabled: false }
// ]

_(collection)
    .reject(_.negate(enabled))
    .value();
// →
// [
//   { name: "Wilson", disabled: false },
//   { name: "Nina", disabled: false }
// ]
```

`User`实例有一个`disabled`属性，如果`disabled`为`false`，则`enabled()`方法返回`true`。`collection`变量包含这些`User`实例的数组。`enabled()`函数是我们自己构建的。我们将将其用作`reject()`的回调。这个函数使用`result()`从集合中的每个项获取`enabled()`值。在这里，`identity()`函数被用作一个技巧，以便让`partialRight()`作为`reject()`的回调工作。

## 使用 initial()和 rest()

`initial()`函数获取除了最后一个元素之外的所有元素——这可以通过链式操作以有趣的方式结合使用。例如，假设我们有一个需要清理的简单字符串：

```js
var string = 'abc\n';

_(string)
    .slice()
    .initial()
    .join('');
// → "abc"
```

如果我们知道字符串总是以我们不关心的内容结束，这是一个很容易将其删除的方法。相同的代码也适用于数组；我们并没有将范围限制在字符串上。例如，`slice()`函数是链式操作的一部分，它使链式操作能够与字符串一起工作。如果我们传递一个数组，`slice()`将没有任何影响——相同的代码仍然有效。然而，我们可能希望在稍后删除它，以及`join()`调用。鉴于我们的链式代码的格式，这很简单。

`initial()`的逆操作是`rest()`——它获取数组中的所有元素，除了第一个元素。就像我们不在乎最后一个元素的情况一样，也可能有不关心第一个元素的情况。以下是如何使用`rest()`的示例：

```js
var collection = [
    { name: 'init', task: _.noop },
    { name: 'sort', task: _.random },
    { name: 'search', task: _.random }
];

_(collection)
    .rest()
    .invoke('task')
    .value();
// → [ 1, 1 ]
```

这个集合包含具有`task()`方法的对象。集合是有序的，所以第一个任务总是`init`任务，我们对此不感兴趣，因为它是一个`noop()`函数。我们通过将`invoke()`方法链接到`rest()`函数来测试这一点，如果一切顺利，我们应该得到一个随机数字数组，并且没有未定义的值。

# 测试真值条件

除了简单地过滤集合外，你通常还需要测试集合的条件。这可能包括过滤一个集合，然后对结果进行简单的肯定/否定回答。在需要检查集合的真值条件的情况下，通常在链式操作的最后进行测试会更简单。通常没有必要在过程中编写多个语句和分配多个变量。

## 测试集合是否包含项目

可能，最直接的测试就是检查一个集合是否包含我们正在寻找的项目。在这种情况下，`contains()`函数很有用，因为它可以轻松地附加到执行其他操作之前的链的末尾。以下是一个`contains()`函数用法的示例：

```js
var string = 'abc123',
    array = [ 'a', 'b', 'c', 1, 2, 3 ];

_(string)
    .filter(_.isString)
    .contains('c');
// → true

_(array)
    .filter(_.isString)
    .contains('c');
// → true
```

代码中有两个集合——一个字符串和一个数组。接下来的两个链在除了包装不同的值之外都是相同的。然而，在这种情况下，两者都返回`true`，因为字符串中有`c`，数组中也有。

如果你只关心测试项目是否存在，那么使用`contains()`等函数总是一个好习惯。如果找到值，这些函数会提前停止循环，或者短路，从而节省宝贵的 CPU 周期。

通常，你并不拥有确切的值。相反，你只有查询约束，但你仍然只对它们是否满足感兴趣，而不是数据本身。这可以通过使用`find()`和`filter()`方法来实现：

```js
var string = 'Dana Porter',
    array = [
        { name: 'Luis', gender: 'm' },
        { name: 'Rhonda', gender: 'f' },
        { name: 'Kirk', gender: 'm' },
        { name: 'Emily', gender: 'f' }
    ];

_(string)
    .chain()
    .filter(function(item) {
        return item.toUpperCase() === 'A';
    })
    .size()
    .isEqual(2)
    .value();
// → true

!!(_(array)
    .find(function(item) {
        return _.first(item.name).toUpperCase() === 'R' &&
            item.gender === 'f';
    }));
// → true
```

代码中的第一个链是针对字符串值的。注意我们在这里使用了`chain()`来启用显式链式调用——这意味着我们最终必须显式调用`value()`来展开结果。这里的`filter()`调用返回匹配`A`的项。我们这样做是为了计算链中这些项的数量。在这种情况下，字符串通过了测试，因为有两个`A`字符。缺点是我们寻找的是一个确切的数字——`2`。`filter()`函数会在找到两个项目之后继续过滤。

第二个链使用了一个包装数组。在这里，我们将调用`find()`的结果转换为一个布尔值。在这里，我们可以使用更复杂的查询条件。

## 任何事物或任何东西都是真实的

本章最后关于检查真值条件的探讨，包括验证至少一个项目或整个集合的有效性。也就是说，如果一个或多个项目通过了我们所设定的测试，那么这个集合可能被认为是有效的。或者，也许要求更为严格，集合中的每个项目都必须通过测试才能被认为是有效的。让我们看看这些测试如何在链中使用：

```js
var collection = [
    1414728000000,
    1383192000000,
    1351656000000,
    1320033600000
];

_(collection)
    .map(function(item) {
        return new Date(item);
    })
    .every(function(item) {
        return item.getMonth() === 9 && item.getDate() === 31;
     });
// → true
```

这个集合包含时间戳数字，因此链中的第一个调用是`map()`，将每个集合项转换为`Date`实例。现在每个项都是日期，我们可以使用`every()`来验证在这个集合中，每一天都是万圣节。

现在，让我们看看如何使用`some()`函数来终止链。这将验证至少有一个项目通过了测试，并且一旦找到就会停止循环：

```js
var collection = [
    { name: 'Danielle', age: 34, skill: 'Backbone' },
    { name: 'Sammy', age: 19, skill: 'Ember' },
    { name: 'Donna', age: 41, skill: 'Angular' },
    { name: 'George', age: 17, skill: 'Marionette' }
];

_(collection)
    .reject({ skill: 'Ember' })
    .reject({ skill: 'Angular' })
    .some(function(item) {
        return item.age >= 25;
    });
// → true
```

你可以看到，在拒绝`Ember`和`Angular`爱好者之后，我们确保至少有一位`25`岁以上的`Backbone`或`Marionette`程序员。

# 计数项目

之前主题的一个变体——*测试真值条件*——是在它们的值通过处理链移动后，在集合中计数项。例如，我们可能需要知道集合中有多少项满足给定的条件。我们可以使用调用链来获取那个数字。

## 使用长度和大小()

`size()`函数很方便，因为我们可以直接在 Lo-Dash 包装器上调用它。这是我们链式操作运行后，在我们的集合中计数结果项的首选方式：

```js
var object = { first: 'Charlotte', last: 'Hall' },
    array = _.range(10);

_(object)
    .omit('first')
    .size();
// → 1

_(array)
    .drop(5)
    .size();
// → 5
```

在这里，我们有`数组`和`对象`。第一个链使用`size()`函数来计算在省略了`first`属性之后属性的数量。第二个链包裹了数组，并在删除`5`个项目之后计算剩余的项目数量。

### 注意

我们可以使用`length`属性，但我们必须先调用`value()`。使用`size()`只是一个快捷方式。

## 使用 countBy()进行分组

我们也可以计数多个项。也就是说，给定一个集合，我们可以将其分成组，然后计算每个组中的项目数量。使用链，我们可以编写一些相当复杂的代码：

```js
var collection = [
    { name: 'Pamela', gender: 'f' },
    { name: 'Vanessa', gender: 'f' },
    { name: 'Gina', gender: 'f' },
    { name: 'Dennis', gender: 'm' }
];

_(collection)
    .countBy('gender')
    .pairs()
    .sortBy(1)
    .reverse()
    .pluck(0)
    .value();
// → [ "f", "m" ]
```

这个链通过按`gender`属性对集合进行分组开始。接下来，我们使用`pairs()`函数获取一个数组的数组。我们这样做是为了能够按组中的项目数量对组进行排序。在集合排序后，我们可以提取我们感兴趣的价值。在这种情况下，`f`性别是第一个，因为那个组有更高的计数。

### 注意

之前的代码使用了两个有趣的技巧。首先，我们向`sortBy()`函数传递一个数值索引。由于键的访问方式与索引相同，所以这按预期工作。其次，我们向`pluck()`函数传递一个数值索引，这与`sortBy()`函数的原因相同。

## 减少集合

我们在链式操作中计数项的最终方法是减少集合。当你想要将整个集合减少为一个使用更复杂的函数计算得出的总和时，这很有用，这些函数应用于每个项目。可以使用以下代码减少集合：

```js
var collection = [
    { name: 'Chad', skills: [ 'backbone', 'lodash' ] },
    { name: 'Simon', skills: [ 'html', 'css', 'less' ] },
    { name: 'Katie', skills: [ 'grunt', 'underscore' ] },
    { name: 'Jennifer', skills: [ 'css', 'grunt', 'less' ] }
];

_(collection)
    .pluck('skills')
    .reduce(function(result, item) {
        return _.size(item) > 2 &&
            _.contains(item, 'grunt') &&
            result + 1;
    }, 0);
// → 1
```

在这里，我们正在从集合中的每个项目中提取`skills`属性。我们对`skills`值感兴趣的两个问题是：它是否包含字符串`grunt`？并且它是否有超过`2`个项？如果这些条件得到满足，那么我们就增加由`reduce()`调用返回的减少的总值。

# 转换

现在我们来看看数据通过我们构建的处理管道时发生的转换。Lo-Dash 和它在链中转换数据有趣的地方在于原始集合没有被修改——而是构建了一个新的集合。这减少了副作用，并且是其他函数式编程语言中不可变概念的基础。

## 构建组、并集和唯一值

在 Lo-Dash 中找到的一些最强大的转换工具可以无需太多努力即可直接使用。这包括根据它们包含的特定值对集合项进行分组、在保留唯一值的同时将数组连接起来，以及从数组中移除任何重复项。例如：

```js
var collection = [
    { name: 'Rudolph', age: 24 },
    { name: 'Charles', age: 43 },
    { name: 'Rodney', age: 37 },
    { name: 'Marie', age: 28 }
];

_(collection)
    .map(function(item) {
        var experience = 'seasoned veteran';
        if (item.age < 30) {
            experience = 'noob';
        } else if (item.age < 40) {
            experience = 'geek cred';
        }
        return _.extend({
            experience: experience
        }, item);
    })
    .groupBy('experience')
    .map(function(item, key) {
        return key +
            ' (' + _.pluck(item, 'name').join(', ') + ')';
    })
    .value();
// →
// [
//   "noob (Rudolph, Marie)",
//   "seasoned veteran (Charles)",
//   "geek cred (Rodney)"
// ]
```

这个链式操作封装了一个普通对象的集合，链式操作中的第一次调用将`item`对象映射为其扩展版本。我们正在计算它们的`experience`属性的字符串版本，并将其分配给一个新的属性。接下来，我们使用`groupBy()`函数根据这个新的`experience`属性对集合进行分组。链式操作的最后一个步骤是再次使用`map()`来生成各种经验组的字符串表示。

使用`union()`函数连接数组可能很有用，如果你已经有一个封装的数组，并且需要确保它具有某些值，同时确保这些值不重复。`union()`函数的应用在以下示例中显示：

```js
var collection = _.sample(_.range(1, 101), 10);

_(collection)
    .union([ 25, 50, 75])
    .sortBy()
    .value();
// → [ 1, 3, 21, 25, 27, 37, 40, 50, 57, 73, 75, 94 ]
```

你可以看到我们的封装数组，10 个随机数的样本，使用`union()`函数与另一个数组连接。然后我们返回排序后的结果。如果你检查输出，你会注意到它总是会包含`25`、`50`和`75`。你也会注意到这些数字永远不会重复。

最后，如果你有一个值集合，并且只需要移除重复项，`uniq()`函数允许你在链式处理中执行此操作：

```js
function name(item) {
    return item.first + ' ' + item.last;
}

var collection = [
    { first: 'Renee', last: 'Morris' },
    { first: 'Casey', last: 'Wise' },
    { first: 'Virginia', last: 'Grant' },
    { first: 'Toni', last: 'Morris' }
];

_(collection)
    .uniq('last')
    .sortBy('last')
    .value();
// →
// [
//   { first: "Virginia", last: "Grant" },
//   { first: "Renee", last: "Morris" },
//   { first: "Casey", last: "Wise" }
// ]

_(collection)
    .uniq(name)
    .sortBy(name)
    .value();
// →
// [
//   { first: "Casey", last: "Wise" },
//   { first: "Renee", last: "Morris" },
//   { first: "Toni", last: "Morris" },
//   { first: "Virginia", last: "Grant" }
// ]

_(collection)
    .map(name)
    .uniq()
    .sortBy()
    .value();
// →
// [
//   "Casey Wise",
//   "Renee Morris",
//   "Toni Morris",
//   "Virginia Grant"
// ]
```

我们看到了从封装集合中提取唯一值的三种不同方法。第一种使用提取风格的简写来过滤重复项。由于我们传递了字符串`last`，它将在这个属性中查找唯一值。第二种方法传递了一个回调函数，该函数将`first`和`last`名称属性连接起来。请注意，这个相同的函数在同一个链式操作中的`sortBy()`调用中也被使用。最后一种方法没有向`uniq()`传递任何参数，因为链式操作的第一步将我们的对象数组映射为字符串数组。

## 提取值

通常，在你的功能链中，你会意识到你不需要你的集合中每个对象的全部内容。这可以使链式操作中后续的操作变得更加简单。为了提取值，可以使用以下代码：

```js
var collection = [
    { gender: 'f', dob: new Date(1984, 3, 8) },
    { gender: 'm', dob: new Date(1983, 7, 16) },
    { gender: 'f', dob: new Date(1987, 2, 4) },
    { gender: 'm', dob: new Date(1988, 5, 2) }
];

_(collection)
    .where({ gender: 'm' })
    .pluck('dob')
    .map(function(item) {
        return item.toLocaleString();
    })
    .value();
// → [ "8/16/1983, 12:00:00 AM", "6/2/1988, 12:00:00 AM" ]
```

在这里，我们正在提取`dob`属性值，这简化了链式操作中后续的`map()`处理程序。无需查找`dob`属性，项目本身就是`dob`属性值。

## 使用`without()`函数创建数组

如果我们需要构建一个新的数组，并且我们知道某些值不应作为项目出现，我们可以使用`without()`函数。这通常是链式操作中的第一个动作，因为它创建了一个新的数组，但并不总是第一个。让我们看看这个例子：

```js
var collection = _.range(1, 11);

return _(collection)
    .without(5, _.first(collection), _.last(collection))
    .reverse()
    .value();
// → [ 9, 8, 7, 6, 4, 3, 2 ]
```

在此代码中，包装的集合包括从`1`到`10`的数字。我们链中的第一次调用将此数组中的项目复制出来，并将它们放置在一个新的数组中，除了传递给`without()`函数的参数值。这些值不包括在新数组中。

## 找到最小和最大值

每个集合都有一个最小值和最大值。使用 Lo-Dash 找到这些值很容易；你只需要使用相应的`min()`和`max()`函数。但如果你需要调整你正在寻找的最小值的范围呢？让我们使用以下代码来完成这个任务：

```js
var collection = [
    { name: 'Daisy', wins: 10 },
    { name: 'Norman', wins: 12 },
    { name: 'Kim', wins: 8 },
    { name: 'Colin', wins: 4 }
];

_(collection)
    .reject(function(item) {
        return item.wins < 5
    })
    .min('wins');
// → { name: "Kim", wins: 8 }
```

在这个例子中，我们并不关心具有小于`5`次`win`计数的项目。因此，我们知道此代码返回的绝对最小值将至少有`5`次胜利。在拒绝无效的`win`计数后，我们使用 pluck 风格简写来根据`wins`属性找到最小值。

`max()`函数可以以类似的方式用作链式操作：

```js
var collection = [
    { name: 'Kerry', balance: 500, credit: 344 },
    { name: 'Franklin', balance: 0, credit: 554 },
    { name: 'Lillie', balance: 1098, credit: 50 },
    { name: 'Clyde', balance: 473, credit: -900 }
];

_(collection)
    .filter('balance')
    .filter('credit')
    .max(function(item) {
        return item.balance + item.credit;
    });
// → { name: "Lillie", balance: 1098, credit: 50 }
```

此集合包含具有`balance`和`credit`属性的对象。前两个链式操作使用`filter()`函数删除这些字段任一为假的对象。然后`max()`函数关闭链式操作。这次，我们使用一个回调函数，允许我们映射我们想要比较的值，以便确定最大值。

## 找到索引

找到给定元素的索引有其用途，我们可以在调用链中应用`index()`函数：

```js
function rank(coll, name) {
    return _(coll)
        .sortBy('score')
        .reverse()
        .pluck('name')
        .indexOf(name) + 1;
}

var collection = [
    { name: 'Ruby', score: 43 },
    { name: 'Robert', score: 59 },
    { name: 'Lindsey', score: 38 },
    { name: 'Marty', score: 55 }
];

rank(collection, 'Ruby');
// → 3

rank(collection, 'Marty');
// → 2
```

此代码中的`rank()`函数接受一个`collection`参数和一个`name`字符串。该函数包装集合，并使用调用链根据`score`属性确定传入名称的排名。第一步是对集合进行排序，然后将其反转，使其根据`score`属性值降序排列。接下来，我们使用`pluck()`函数从集合中提取名称，该函数保持我们刚刚创建的排序顺序。现在我们可以使用`indexOf()`来确定给定用户的排名。

## 使用 difference()和 xor()

变换主题的最后一部分是使用`difference()`和`xor()`函数合并两个数组的内容。这两个函数工作方式相似，但有一些细微的差别值得注意和关注。这些函数在链式操作的开始处非常有用，确保包装的数组只包含必要的值。例如，假设你的数字数组不应该有任何奇数。然后我们可以使用以下代码来满足这个条件：

```js
var collection = _.range(1, 51),
    odds = _.filter(_.range(1, 101), function(item) {
        return item % 2;
    });

_(collection)
    .difference(odds)
    .takeRight(10)
    .reverse()
    .value();
// → [ 32, 34, 36, 38, 40, 42, 44, 46, 48, 50 ]
```

在此代码中，我们的集合由`50`个数字组成，而`odds`数组包含从`1`到`100`的奇数。我们的链首先通过调用`difference()`函数开始，将`odds`数组作为参数传入。接下来，我们从结果数组中取出前 10 个元素并对其进行排序。要注意的是，结果中没有超过`50`的值。我们已经移除了所有低于`50`的奇数，因为这是包装数组与作为参数提供的数组之间的差。换句话说，这不是对称差。为此，我们会在链中使用`xor()`函数：

```js
var collection = _.range(1, 26),
    evens = _.reject(_.range(1, 51), function(item) {
        return item % 2;
    });

_(collection)
    .xor(evens)
    .reverse()
    .value();
// → 
// [ 50, 48, 46, 44, 42, 40, 38, 36, 34, 32, 30, 28, 26,
//   25, 23, 21, 19, 17, 15, 13, 11, 9, 7, 5, 3, 1 ]
```

这次，我们的集合是一系列从`1`到`25`的数字，而`evens`数组则包含从`2`到`50`的偶数。我们在链中使用`xor()`函数将集合与`evens`数组连接起来。与`difference()`函数的区别在于，它将包括`evens`数组中超过`25`的所有值，因为`xor()`将计算对称差。

# 中间结果

有时候，我们不想等到调用链完成才访问在某个步骤计算出的值。考虑一下链中某个函数产生的中间值应该被链中稍后的另一个函数使用的情况。在其他情况下，我们需要完全覆盖链返回的值。

## 操纵链

我们可以使用`tap()`函数将我们自己的回调函数注入链中。这与提供给其他 Lo-Dash 函数的回调不同。它不会改变值在函数调用链中流动时的值。相反，将`tap()`视为拦截值流经链的方式，并可能以某种方式对其做出反应。让我们看看这个函数的例子：

```js
var collection = [
        { name: 'Stuart', age: 41 },
        { name: 'Leah', age: 26 },
        { name: 'Priscilla', age: 37 },
        { name: 'Perry', age: 31 }
    ],
    min,
    max;

_(collection)
    .filter(function(item) {
        return item.age >= 30;
    })
    .tap(function(coll) {
        min = _.min(coll, 'age'),
        max = _.max(coll, 'age')
    })
    .reject(function(item) {
        return item.age === max.age;
    })
    .value();
// min → { name: "Perry", age: 31 }
// max → { name: "Stuart", age: 41 }
// → 
// [
//   { name: "Priscilla", age: 37 },
//   { name: "Perry", age: 31 }
// ]
```

此代码将我们的集合包装起来，并过滤掉年龄低于 30 岁的项。接下来，我们使用`tap()`回调来设置我们的`min`和`max`变量。注意这些变量的作用域；它们是在链外部定义的，因此可以访问链中任何未来的回调。这正是我们在这里所做的——我们拒绝任何`age`属性等于找到的最大年龄的项。注意，如果没有在链的第一个过滤器之后计算它，`max`值可能会有所不同。

### 注意

这种方法的唯一缺点是，我们的链不再是紧密封装的单元，不能在代码中移动。然而，这种权衡是我们可以优雅地即时计算链所需的值。无论如何，要注意的是，不同的编程风格可能更倾向于一个方向而不是另一个方向。

## 注入值

操作链在运行时返回的内容的另一种方法是使用`thru()`函数。它的工作方式与`tap()`类似，但这个函数返回的任何内容都成为新的值：

```js
var collection = _.range(1, _.random(11)),
    result;

result = _(collection)
     .thru(function(coll) {
         return _.size(coll) > 5 ? coll : [];
     })
     .reverse()
     .value();

_.isEmpty(result) ? 'No Results' : result.join(',');
// → "No Results"
```

这个链式操作是通过使用`thru()`函数回调来启动的，以验证集合的最小大小。如果它小于`5`，我们甚至不需要麻烦——我们只需返回一个空数组。返回一些可以与剩余链式函数一起使用的东西很重要，而空数组非常适合。我们只是使用`thru()`来声明任何小于`5`的长度应该与空数组具有相同的意义。这个函数实际上是一个注入这些细微业务规则的理想位置，这些规则通常在代码编写后很久才会显现出来。

# 键和值

现在是时候将我们的注意力转向对象键和值，以及它们如何在函数调用链中使用。通常，这些涉及将一个普通对象包装在 Lo-Dash 实例中，然后使用`keys()`或`values()`函数来启动处理。也有时候，你有一组对象，只想处理某些属性值。为此，有`pick()`和`omit()`函数可以在链中使用。

## 过滤后的键和值

我们可以在链式操作中的后续步骤使用过滤后的对象键数组的结果。当我们不确定哪些键可用，只有最佳猜测时，这非常有用。让我们尝试通过键和值进行过滤：

```js
var object = {
    firstName: 'Jerald',
    lastName: 'Wolfe',
    age: 49
};

_(object)
    .keys()
    .filter(function(item) {
        return (/name$/i).test(item);
    })
    .thru(function(items) {
        return _.at(object, items);
    })
    .value();
// → [ "Jerald", "Wolfe" ]
```

我们正在包装的对象有两个以`name`结尾的属性名。因此，我们将`keys()`函数作为链式操作的第一步来抓取所有键，然后过滤掉不以`name`结尾的键。接下来，我们使用`thru()`函数返回与我们的键过滤结果相对应的对象属性值。对于对象属性值，也可以进行类似操作，尤其是在不需要使用键的情况下。让我们看看一个例子：

```js
var object = {
    first: 'Connie',
    last: 'Vargas',
    dob: new Date(1984, 08, 11)
};

_(object)
    .values()
    .filter(_.isDate)
    .map(function(item) {
        return item.toLocaleString();
    })
    .value();
// → [ "9/11/1984, 12:00:00 AM" ]
```

这个链式操作会抓取被包装对象的属性值，并过滤掉所有非日期的项。然后，找到的`Date`对象会被映射到一个字符串数组中。

## 忽略和选择属性

在链中挑选某些对象属性使用，以及那些要忽略的属性，有其用途，尤其是在包装的值是一个普通对象，并且基于某些标准，有一些属性我们不想使用时。例如，我们可能有一个想要转换成索引对象的集合，但在过程中，我们需要挑选或忽略某些应该或不应该存在的值，如下面的例子所示：

```js
var collection = [
    { first: 'Tracey', last: 'Doyle', age: 40 },
    { first: 'Toby', last: 'Wright', age: 49 },
    { first: 'Leonard', last: 'Hunt', age: 32 },
    { first: 'Brooke', last: 'Briggs', age: 32 }
];

_(collection)
    .indexBy('last')
    .pick(function(value) {
        return value.age >= 35;
    })
    .transform(function(result, item, key) {
        result[key] = _.omit(item, 'last');
    })
    .value();
// → 
// {
//   Doyle: { first: "Tracey", age: 40 },
//   Wright: { first: "Toby", age: 49 }
// }
```

这段代码通过`last`属性值对对象数组进行索引。链式操作的下一步是只选择`age`大于`34`的项目。最后，由于每个项目都是通过姓氏索引的，我们不再需要`last`属性，因此`transform()`函数使用`omit()`来为每个项目移除它，这是链式操作的最后一个步骤。

# 返回包装器

包装器和随后的函数调用链并不是随机存在于我们的代码中的。下一章将更深入地探讨这个主题，所以这可以被视为一个预告。到目前为止，我们只看了链在同一个语句中构建和执行的情况。然而，如果我们费尽心思设计一个具有通用目的的调用链，那么不持续组装这个链不是更好吗？让我们用以下代码来设计这个链：

```js
function best(coll, prop, count) {
    return _(coll)
        .sortBy(prop)
        .takeRight(count);
}

var collection = [
    { name: 'Mathew', score: 92 },
    { name: 'Michele', score: 89 },
    { name: 'Joe', score: 74 },
    { name: 'Laurie', score: 83 }
];

var bestScore = best(collection, 'score', 2);

bestScore.value();
// → 
// [
//   { name: "Michele", score": 89 },
//   { name: "Mathew", score: 92 }
// ]

bestScore.reverse().value();
// → 
// [
//   { name: "Michele", score: 89 },
//   { name: "Mathew", score: 92 }
// ]

bestScore.pluck('name').value();
// → [ "Michele", "Mathew" ]
```

这里定义的 `best()` 函数返回一个 Lo-Dash 包装器实例。注意，在 `best()` 内部，我们实际上正在链式调用函数，但它们实际上并没有被调用，这意味着 `best()` 的返回值是一个包装器。这可以通过 `bestScore` 变量来说明，它持有包装器实例。这个包装器可以一次又一次地被调用，而无需重新构建函数调用链。尽管如此，如果我们需要稍微调整链，我们可以在其基础上构建。我们通过调用 `reverse()` 和 `pluck()` 来对 `bestScore` 进行这样的操作。

# 摘要

本章介绍了包装值的概念以及应用于它们的函数调用链。这种多才多艺的编程模型，是 Lo-Dash 的基础，有助于使用紧凑且易于阅读的代码构建复杂的功能块。链式调用并不仅限于 Lo-Dash，它们在许多其他库中也很受欢迎，也许在 jQuery 中最为常见。

应用程序面临着过滤数据的艰巨任务——大量的数据和大量的硬约束。我们不是创建带有许多临时变量的混乱代码，而是提出了几种使用链构建复杂过滤器的方法。之后，我们探讨了使用链测试真值条件。这些就像过滤器一样，除了它们不返回集合结果，而是只返回表示为布尔值的真值陈述。我们还探讨了如何在函数调用链之后计数项目。

我们还学到了另一个基本实践，那就是将集合转换成更适合特定上下文的替代、更易用的表示形式。就像过滤一样，使用链来转换集合通常更好，因为它减少了你需要编写的代码量。

我们以探讨你的函数如何返回不一定立即执行的包装器来结束本章。这是我们在下一章中构建可重用 Lo-Dash 组件的下一步。
