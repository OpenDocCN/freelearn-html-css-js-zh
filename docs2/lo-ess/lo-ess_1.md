# 第一章. 使用数组和集合

Lo-Dash 提供了各种在数组和集合上操作的功能。这通常涉及以某种形式遍历集合。Lo-Dash 帮助使迭代行为易于实现，包括搜索数据以及构建新的数据结构。

集合是应用编程的核心。这是我们将会连续对集合中的每个元素应用函数的地方。本章介绍了集合的概念，这是 Lo-Dash 代码广泛使用的。

在本章中，我们将涵盖以下主题：

+   遍历集合

+   排序数据

+   搜索数据

+   将集合切割成更小的部分

+   转换集合

# 数组和集合之间的区别

对于 Lo-Dash 新手来说，最初感到困惑的来源之一是数组和集合之间的区别。Lo-Dash API 为数组提供了一套函数，并为集合提供了一套函数。但为什么？这些函数似乎对任何集合都是可互换的。好吧，根据 Lo-Dash，对集合实际上是什么的更好定义可能会澄清一些事情。

集合是一个抽象概念。也就是说，我们可以使用 Lo-Dash 中找到的集合函数来迭代我们想要的任何 JavaScript 对象。例如，`forEach()` 函数会愉快地遍历数组、字符串或对象。这些类型之间的微妙差异以及它们在迭代中的含义对开发者来说是隐藏的。

Lo-Dash 提供的数组函数更少抽象，实际上它们确实期望一个数组。从某种意义上说，即使是这些函数也是抽象的，因为它们并没有明确检查 `Array` 类型。它们要求对象支持数值索引，并且具有数值的 `length` 属性。

吸取的教训是，在作为 Lo-Dash 程序员的大部分日子里，数组和集合之间的区别并不重要。主要是因为主要的集合类型将始终是数组。在极少数情况下，当区别很重要时，只需记住数组函数对它们认为可接受的数据有稍微严格一些的标准。

# 遍历集合

Lo-Dash 在集合上做了很多迭代，无论是开发者明确执行，还是通过高级 Lo-Dash 函数隐式执行。让我们看看基本的 `forEach()` 函数：

```js
var collection = [
    'Lois',
    'Kathryn',
    'Craig',
    'Ryan'
];

_.forEach(collection, function(name) {
    console.log(name);
});
// → 
// Lois
// Kathryn
// Craig
// Ryan
```

这段代码并没有做什么，除了记录集合中每个项目的值。然而，它确实给你一个关于 Lo-Dash 中迭代函数外观的一般感觉。正如其名所示，对于数组中的每个元素，应用函数回调。传递给回调函数的不仅仅是当前元素。我们还可以得到当前索引。

### 小贴士

**下载示例代码**

你可以从你购买的所有 Packt Publishing 书籍的账户中下载示例代码文件，网址为[`www.packtpub.com`](http://www.packtpub.com)。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

看看下面的代码：

```js
var collection = [
    'Timothy',
    'Kelly',
    'Julia',
    'Leon'
];

_.forEach(collection, function(name, index) {
    if (name === 'Kelly') {
        console.log('Kelly Index: ' + index);
        return false;
    }
});
// → Kelly Index: 1
```

返回`false`告诉`forEach()`这个迭代将是最后一个。索引，在每次迭代中，都会递增并作为第二个参数传递给回调函数。

`forEach()`函数以典型的从左到右的方式遍历集合。如果我们想要相反的行为，我们可以使用其堂兄弟函数，`forEachRight()`：

```js
var collection = [
    'Carl',
    'Lisa',
    'Raymond',
    'Rita'
];

var result = [];

_.forEachRight(collection, function(name) {
    result.push(name);
});
// →
// [
//   "Rita",
//   "Raymond",
//   "Lisa",
//   "Carl"
// ]
```

这种行为在处理已排序的集合时很有用，正如前面代码所示。但是，假设我们想要以降序在 DOM 中渲染这个数组数据。前面的代码显示我们可以渲染给定迭代中的每个项目。在这种情况下使用`forEachRight()`函数的优势在于不需要对数组进行反转排序。

然而，很多时候这个快捷方式是不够的，你必须对你的集合进行排序。接下来我们将查看 Lo-Dash 中帮助排序的函数。

# 排序数据

在纯 JavaScript 中，排序的方法涉及数组和两种方法。`sort()`方法按升序对数组进行排序，使用原始的比较操作。你可以通过传递`sort()`，一个**比较器**函数来自定义这种行为。例如，你可以使用这个回调函数按降序排序一个数组。另一种方法，`reverse()`，只是简单地反转数组的顺序。它是当前顺序的相反，无论是什么。

原生的数组`sort()`方法在原地排序数组，尽管你可能不希望这样。不可变操作减少了副作用，因为它们不会改变原始集合。具体来说，你可能已经按特定顺序请求了 API 数据。UI 的一个区域想要以不同的排序顺序渲染这个数组。好吧，你不想改变请求的顺序。在这种情况下，有一个返回包含原始项目但按预期排序顺序的新数组的函数会更好。

## 使用`sortBy()`

`sortBy()`函数是 Lo-Dash 对原生`Array.sort()`方法的回应。由于它是一个抽象的集合函数，它不仅限于数组。看看下面的代码：

```js
_.sortBy('cba').join('');
```

当函数与字符串作为输入正常工作时，输出是一个排序后的字符数组；因此，需要调用它们将它们重新组合在一起。这是因为`sortBy()`函数始终返回一个数组作为结果。

`sortBy()` 函数与原生的 `Array.sort()` 方法类似，默认情况下按升序对集合项进行排序。同样，与原生方法类似，我们可以向 `sortBy()` 传递一个回调函数来定制排序行为，如下所示：

```js
var collection = [
    { name: 'Moe' },
    { name: 'Seymour' },
    { name: 'Harold' }, 
    { name: 'Willie' }
];

_.sortBy(collection, function(item) {
    return item.name;
});
```

传递给 `sortBy()` 的前面的回调函数返回对象属性的值。通过这样做，排序行为将比较属性值——在这种情况下，`name`——而不是对象本身。实际上，有一种更短的方法可以达到相同的结果：

```js
_.sortBy(collection, 'name');
```

这在 Lo-Dash 术语中被称为 **pluck 风格** 简写。我们传入我们想要按其排序集合的属性名称。然后从集合中的每个对象中提取该属性的值。实际上，我们将在稍后更深入地查看 `pluck()` 函数。

`sortBy()` 有一个最后的技巧，它将 pluck 简写提升到了下一个层次，并允许按多个属性名进行排序，如下面的代码所示：

```js
var collection = [
    { name: 'Clancy', age: 43 },
    { name: 'Edna', age: 32 },
    { name: 'Lisa', age: 10 },
    { name: 'Philip', age: 10 }
];

_.sortBy(collection, [ 'age', 'name' ]);
// →
// [
//   { name: "Lisa", age: 10 },
//   { name: "Philip", age: 10 },
//   { name: "Edna", age: 32 },
//   { name: "Clancy", age: 43 }
// ]
```

这里的主要排序决定因素是 `age` 属性。如果我们指定第二个属性名，则用于确定具有相同主要排序顺序的元素顺序。它充当决定性因素。这里有两个对象的 `age` 等于 `10`。由于 `name` 属性是次要排序顺序，这就是这两个对象被排序的方式。在 Web 应用程序中，多个排序属性是一个典型用例，如果没有这个 Lo-Dash 工具，我们将需要编写大量 JavaScript 代码来实现。

## 维护排序顺序

使用 `sortBy()` 函数是改变现有集合排序顺序的绝佳工具，尤其是如果我们不希望永久更改该集合的默认排序顺序时。然而，在其他时候，你可能希望永久保持集合排序。有时，这实际上是由发送给我们以 JSON 数据形式集合的后端 API 完成的。

在这些情况下，排序很简单，因为你实际上不需要对任何东西进行排序。它已经完成了。挑战在于维护排序顺序。因为，有时元素会实时添加到集合中。维护排序顺序的直观方法是将新元素简单地添加到集合中然后重新排序。Lo-Dash 的替代方案是找出将保持当前集合排序顺序的插入点。这在上面的代码中显示：

```js
var collection = [ 
    'Carl',
    'Gary',
    'Luigi',
    'Otto'
];

var name = 'Luke';

collection.splice(_.sortedIndex(collection, name), 0, name);
// → 
// [
//   "Carl",
//   "Gary",
//   "Luigi",
//   "Luke",
//   "Otto"
// ]
```

新的 `name` 变量被插入到倒数第二个位置。这实际上是维护排序集合顺序所需的功能。相同的 `splice()` 数组方法用于从集合中删除项目，这不会破坏顺序。添加新项目是一个挑战，因为需要搜索以确定插入索引。`sortedIndex()` 函数在集合上执行二分搜索以确定新项目适合的位置。

# 搜索数据

应用程序不使用整个集合。相反，它们遍历集合的子集，或者它们在集合中查找特定的项目。Lo-Dash 提供了一些功能工具，帮助程序员找到他们需要的数据。

## 筛选集合

使用 Lo-Dash 在集合上执行筛选操作的最简单方法是使用`where()`函数。该函数接受一个对象参数，并将匹配其属性与集合中的每个项目，如下面的代码所示：

```js
var collection = [ 
    { name: 'Moe', age: 47, gender: 'm' },
    { name: 'Sarah', age: 32, gender: 'f' },
    { name: 'Melissa', age: 32, gender: 'f' },
    { name: 'Dave', age: 32, gender: 'm' }
];

_.where(collection, { age: 32, gender: 'f' });
// →
// [
//   { name: "Sarah", age: 32, gender: "f" },
//   { name: "Melissa", age: 32, gender: "f" }
// ]
```

之前的代码在`age`和`gender`属性上对集合进行了筛选。查询结果对应的是 32 岁的女性。`Moe`对象与这两个属性都不匹配，而`Dave`对象与`age`属性匹配，但与`gender`属性不匹配。关于`where()`筛选的一个好的思考方式是，你传递给筛选器的每个对象属性都将进行逻辑**并且**连接。例如，匹配`age`和`gender`属性。

`where()`函数因其简洁的语法和对集合的直观应用而非常出色。这种简单性带来了一些限制。首先，我们比较的属性值必须与集合中的每个项目完全匹配。有时，我们需要比严格相等更复杂的比较。其次，`where()`连接查询条件的方式的逻辑`and`并不总是令人满意。逻辑`or`条件同样常见。

对于这些高级筛选功能，你应该转向`filter()`函数。这是一个基本的筛选操作，比`where()`查询还要简单：

```js
var collection = [ 
    { name: 'Sean', enabled: false },
    { name: 'Joel', enabled: true },
    { name: 'Sue', enabled: false },
    { name: 'Jackie', enabled: true }
];

_.filter(collection, 'enabled');
// →
// [
//   { name: "Joel", enabled: true },
//   { name: "Jackie", enabled: true }
// ]
```

由于这个集合中`enabled`属性对两个对象有**真值**，它们被返回到一个新的数组中。

### 注意

Lo-Dash 在所有地方都使用真值的概念。这仅仅意味着，如果在一个`if`语句或三元运算符中使用，值将测试为正。值不需要是布尔类型和`true`才能是真值。一个对象、一个数组、一个字符串、一个数字——这些都是真值。而 null、undefined 和 0——都是假值。

如前所述，`filter()`函数填补了`where()`函数的空白。与`where()`不同，`filter()`接受一个回调函数，该函数应用于集合中的每个项目，如下面的代码所示：

```js
var collection = [ 
    { type: 'shirt', size: 'L' },
    { type: 'pants', size: 'S' },
    { type: 'shirt', size: 'XL' },
    { type: 'pants', size: 'M' }  
];

_.filter(collection, function(item) {
    return item.size === 'L' || item.size === 'M';
});
// →
// [
//   { type: "shirt", size: "L" },
//   { type: "pants", size: "M" }
// ]
```

回调函数使用`or`条件来满足这里的`size`约束——`medium`或`large`。这用`where`函数是无法实现的。

### 注意

`filter()`函数也接受一个对象参数。在 Lo-Dash 术语中，这被称为**where 风格回调**。许多函数，不仅仅是`filter()`，接受指定为对象的筛选标准，并像`where()`一样表现。

当我们知道我们要找什么时，使用 `filter()` 函数过滤集合是好的。回调函数给程序员足够的灵活性来组合复杂的标准。但有时，我们不知道从集合中需要什么。相反，我们只知道我们不需要什么，如下面的代码所示：

```js
var collection = [
    { name: 'Ryan', enabled: true },
    { name: 'Megan', enabled: false },
    { name: 'Trevor', enabled: false },
    { name: 'Patricia', enabled: true }
];

_.reject(collection, { enabled: false });
// →
// [
//   { name: "Ryan", enabled: true },
//   { name: "Patricia", enabled: true }
// ]
```

你可以在这里看到，只有 `enabled` 项目被返回，这相当于执行 `_.filter(collection, {enabled: true})`，这是对 `filter()` 的简单反转。你使用哪个函数取决于个人偏好以及它们被使用的上下文。选择在代码中读起来更清晰的那个。

### 注意

`reject()` 实际上在内部使用了 `filter()` 函数。它使用 `negate()` 函数来反转传递给 `filter()` 的回调函数的结果。

## 在集合中查找项目

有时，我们需要特定的集合项目。过滤集合只是生成一个包含较少项目的新的集合。相反，在集合中查找项目意味着找到特定的项目。

用于在集合中查找项目的函数被恰当地命名为 `find()`。此函数接受与 `filter()` 函数相同的参数。你可以传递属性名称作为字符串，传递一个包含属性名称和值的对象以执行类似 `where` 风格的搜索，或者传递一个简单的回调函数以匹配你想要的内容。以下是一个示例：

```js
var collection = [ 
    { name: 'Derek', age: 37 },
    { name: 'Caroline', age: 35 },
    { name: 'Malcolm', age: 37 },
    { name: 'Hazel', age: 62 } 
];

_.find(collection, { age:37 });
// → { name: "Derek", age: 37 }
```

实际上，有两个项目符合集合中的样式标准——`Derek` 和 `Malcolm`。如果我们运行这段代码，我们会看到只有 `Derek` 被返回。这是因为 `find()` 函数一旦找到匹配项就会立即返回。在集合中搜索项目时，集合的顺序很重要。它不考虑重复的值。

让我们换个方向看看我们会找到什么。使用相同的集合和相同的搜索标准，你可以向相反方向搜索：

```js
_.findLast(collection, { age:37 });
// → { name: "Malcolm", age: 37 }
```

虽然 `find()` 函数在集合中搜索匹配项的第一个出现，但 `findLast()` 函数则搜索最后一个出现。这在处理已排序的集合时非常有用——你可以更好地优化你的线性搜索。

### 注意

虽然 Lo-Dash 对迭代集合时使用的 `while` 循环进行了大量优化，但使用 `find()` 等函数执行搜索是线性的。重要的是要记住，使用 Lo-Dash 的程序员需要考虑他们独特应用数据带来的性能影响。Lo-Dash 函数针对通用常见情况进行优化，它们不会因为使用它们而神奇地使你的代码更快。它们是帮助程序员制作高性能代码的工具。

# 将集合切割成更小的部分

到目前为止，我们已经看到了如何过滤集合，创建一个新的较小的集合。Lo-Dash 为你提供了一些函数，这些函数接受现有的数组并生成一个或多个较小的数组。例如，你可能想要任何数组的开头部分或结尾部分的一部分。数组可以被分成较小的数组块，这对于批量处理非常有用。你还可以使用 Lo-Dash 数组工具来删除重复项，从而确保数组的唯一性。

## 首尾集合部分

使用原生的 JavaScript 数组，你可以使用 `slice()` 数组方法来截取数组的前一部分。Lo-Dash 在原生的数组 `slice()` 方法之上提供了抽象，这使得开发者编写直观的代码变得稍微容易一些——这并不总是原生数组方法的情况。此外，Lo-Dash 的 `take()` 函数作用于集合，因此它可以与数组和字符串一起使用，如下面的代码所示：

```js
var array = [ 
    'Steve',
    'Michelle',
    'Rebecca',
    'Alan'
];

_.take(array, 2);
// → [ "Steve", "Michelle" ]

_.take('lodash', 2).join('');
// → "lo"
```

使用 `take()` 对数组进行操作和字符串操作时的输出有所不同。当应用于数组时，它生成一个新的数组，这是原始数组的一个子集。然而，当应用于字符串时，它返回一个包含单个字符的新数组。前面的代码将返回 `[ 'l', 'o' ]`。这很可能不是我们想要的，所以我们将使用空字符串将这些字符重新组合在一起。

我们可以使用 `takeRight()` 函数来截取集合和字符串的最后部分。使用相同的数组和字符串，你可以运行以下代码来获取集合的最后部分：

```js
_.takeRight(array, 2);
_.takeRight(string, 4).join('');
```

结果数组看起来像 `[ 'Rebecca', 'Alan']`。结果字符串看起来像 `'dash'`。

将 `take()` 应用于不带任何参数的集合将截取第一个项目。同样，将 `takeRight()` 应用于不带任何参数将截取最后一个项目。在这两种情况下，返回的值是一个包含一个项目的数组，而不是项目本身。如果你只是想要第一个或最后一个集合项目，请分别使用 `first()` 和 `last()` Lo-Dash 函数。

## 将集合分成块

有时候，我们会遇到大型集合。真的是非常大的集合。特别是当使用 API 数据时，前端并不总是能控制返回的数据集的大小。当 API 返回大量数据时，我们的代码处理这些数据可能会导致 UI 冻结。我们无法确切地说给我们更少的数据来工作，这样 UI 就不会冻结。UI 冻结也是不可接受的。

Lo-Dash 非常高效地遍历集合。然而，它并没有控制你代码中可能执行的成本高昂的操作。这就是导致 UI 冻结的原因——不是集合本身的大小，也不是执行一次成本高昂的操作——而是这两个因素结合在一起，对 UI 的响应性构成了致命的威胁。

`chunk()` 函数是一种将处理非常大的集合分解为几个较小任务的方法。这给了 UI 更新的机会——渲染挂起的 DOM 更新并处理挂起的事件。这个函数的用法可以在以下代码中看到：

```js
function process(chunks, index) {
    var chunk = chunks[index];
    if (_.isUndefined(chunk)) {
        return;
    };  
    console.log('doing expensive work ' + _.last(chunk));
    _.defer(_.partial(process, chunks, ++index));
}

var collection = _.range(10000),
    chunks = _.chunk(collection, 50);

process(chunks, 0);
// → 
// doing expensive work 49
// doing expensive work 99
// doing expensive work 149
```

如果前面的代码看起来有点让人望而却步，别担心。这里介绍了一些可能让你感到困惑的新概念。让我们先从高层次上解释一下代码实际上在做什么。创建了一个大集合，并将其分割成更小的集合块。`process()` 函数对每个块进行一些工作，然后再次调用自身来处理下一个块，直到没有块为止。

集合本身是通过`range()`函数生成的，其中包含`10000`个整数。重要的是集合的大小，而不是内容。`chunk()`函数用于将大集合分割成更小的集合。我们指定了每个分割集合的大小，在这种情况下，我们得到了 20 个包含 50 个项目的较小集合。处理工作是通过调用`process(chunks, 0)`开始的。第二个参数是要开始的第一个块。

`process()` 函数本身根据`index`参数获取下一个要处理的块。如果块未定义，这意味着已经到达了末尾，没有更多的块可以处理。否则，我们可以开始对块进行昂贵的处理，如示例中的`console.log()`调用所示。最后，`defer()`函数将开始处理下一个块。我们使用`defer()`的原因是给调用栈一个清空的机会，也给 DOM 操作一个运行的机会。如果我们不这样做，使用`chunk()`来分割处理就没有意义了。`defer()`函数期望一个回调，我们使用`partial()`来创建一个新函数，该函数已经提供了参数。

### 注意

`defer()`和`partial()`函数在第三章*使用函数*中有更深入的介绍。

我们如何知道我们的数组块的大小？在之前的代码中，我们选择了`50`作为块大小。但这是一个任意决定，还是基于应用程序中使用的典型数据集？简短的答案是，我们必须稍微调整并针对常见情况进行优化。这可能意味着根据整体集合大小的百分比来确定块大小，如下面的代码所示：

```js
var collection = _.range(10),
    size = Math.ceil(0.25 * collection.length);
_.chunk(collection, size);
// → 
// [
//   [ 0, 1, 2 ],
//   [ 3, 4, 5 ],
//   [ 6, 7, 8 ],
//   [ 9 ]
// ]
```

这里的块大小最终是`3`。实际大小是 2.5，但由于没有 2.5 个集合元素，所以取其上限。此外，你感兴趣的不是块大小的精确度，而是接近 25%的程度。

### 注意

你可能已经注意到`3`不能整除`10`。`chunk()`函数足够智能，不会遗漏任何项。任何未填满块大小的剩余项仍然被包括在内。

## 构建唯一数组

集合中有时会有不想要的重复项。这可能是 API 数据本身包含重复项的结果，或者是在前端执行的其他计算中的副作用。无论原因如何，Lo-Dash 提供了必要的工具，可以快速生成唯一的集合。

`uniq()`函数接受一个集合作为输入，并生成一个新的集合作为输出，其中任何重复项都被移除：

```js
var collection = [ 
    'Walter',
    'Brenda',
    'Arthur',
    'Walter'
];

_.uniq(collection);
// → [ "Walter", "Brenda", "Arthur" ]
```

默认情况下，潜在的重复项使用严格相等运算符进行比较。在先前的集合中，由于`'Walter' === 'Walter'`，找到了重复项并将其移除。您可以更详细地指定您希望`uniq()`如何比较值。例如，如果我们有一个对象集合，我们只想基于`name`属性的唯一对象，我们可以编写`_.uniq(collection, 'name')`。该函数还接受一个回调，用于在比较之前计算值。这在对象的唯一性不是那么直接的情况下很有用，如下面的代码所示：

```js
var collection = [ 
    { first: 'Julie', last: 'Sanders' },
    { first: 'Craig', last: 'Scott' },
    { first: 'Catherine', last: 'Stewart' },
    { first: 'Julie', last: 'Sanders' },
    { first: 'Craig', last: 'Scott' },
    { first: 'Janet', last: 'Jenkins' }
];

_.uniq(collection, function(item) {
    return item.first + item.last;
});
// →
// [
//   { first: "Julie", last: "Sanders" },
//   { first: "Craig", last: "Scott" },
//   { first: "Catherine", last: "Stewart" },
//   { first: "Janet", last: "Jenkins" }
// ]
```

此代码确保集合中每个对象的唯一性基于全名。可能没有全名属性，也许在应用程序的其他地方并不需要。因此，`uniq()`函数可以即时构建一个，这仅用于验证此约束的唯一目的。

# 转换集合

Lo-Dash 提供了一系列工具，可以将集合转换为新的数据结构。此外，还有一些工具可以将两个或多个集合组合成一个单一的集合。这些函数专注于前端开发者面临的最常见且最繁重的编程任务。您不必专注于样板集合转换，可以回到制作出色的应用程序——用户对您那样酷炫的紧凑代码的关注程度不如您。

## 分组集合项

集合中的项有时会隐式分组。例如，假设有一个给定类对象的`size`属性，其允许的值是`'S'`、`'M'`或`'L'`。你的前端应用程序中的代码可能需要将包含这些各种组的项四舍五入以供显示。我们不会编写自己的代码，而是让`groupBy()`函数处理构建此类分组的复杂性：

```js
var collection = [ 
    { name: 'Lori', size: 'S' },
    { name: 'Johnny', size: 'M' },
    { name: 'Theresa', size: 'S' },
    { name: 'Christine', size: 'S' }
];

_.groupBy(collection, 'size');
// →
// {
//   S: [
//     { name: "Lori", size: "S" },
//     { name: "Theresa", size: "S" },
//     { name: "Christine", size: "S" }
//   ],
//   M: [
//     { name: "Johnny", size: "M" }
//   ]
// }
```

`groupBy()` 函数，正如你可能已经注意到的，它并不返回一个集合——它接受一个集合作为输入，但将其转换为一个对象。`groupBy()` 返回的对象包含输入集合的原始项，只是组织方式不同。该对象的属性是你要按其分组的值。前述代码中的大多数集合项将驻留在 `S` 属性中。

### 注意

你还会看到，像 `groupBy()` 这样的转换函数实际上并没有修改项本身——只是它们所在的集合。这就是为什么在前述代码的结果对象中，每个项仍然有它的 `size` 属性，尽管实际上并不需要。

当你以字符串形式传递属性名时，`groupBy()` 将使用 pluck 风格的回调函数从集合中的每个项中获取该属性的值。唯一的属性值形成分组对象的键。正如通常情况那样，对象属性并不明确，需要在运行时计算。在分组项的上下文中，函数回调可以用于在分组不是简单比较的情况下对集合项进行分组，如下述代码所示：

```js
var collection = [ 
    { name: 'Andrea', age: 20 },    
    { name: 'Larry', age: 50 },  
    { name: 'Beverly', age: 67 },
    { name: 'Diana', age: 39 }
];

_.groupBy(collection, function(item) {
    return item.age > 65 ? 'retired' : 'working';
});
// →
// {
//   working: [
//     { name: "Andrea", age: 20 },
//     { name: "Larry", age: 50 },
//     { name: "Diana", age: 39 }
//   ],
//   retired: [
//     { name: "Beverly", age: 67 }
//   ]
// }
```

与测试相等性不同，此回调函数测试近似值。也就是说，`age` 属性中大于 `65` 的任何内容都被认为是退休的。我们将其作为分组标签返回。请记住，如果这些回调函数返回原始类型作为键，那就最好了。对于任何其他值，返回字符串 `working`。这些回调函数的优点是，它们可以用来快速生成关于你正在处理的 API 数据的报告。前述示例通过传递给 `groupBy()` 的单行回调函数说明了这一点。

### 注意

虽然 `groupBy()` 函数可以接受一个 where 风格的对象作为第二个参数，但这可能不是你想要的。例如，如果集合中的项通过了测试，它最终会进入 `true` 组。否则，它就是 `false` 组的一部分。在深入使用 pluck 或 where 风格的回调函数之前要小心——它们可能不会按预期工作。尝试一下，快速得到结果以验证你的方法。

## 计算集合项数量

Lo-Dash 帮助我们找到集合的最小值和最大值。如果我们处理的是只包含数字的大量数组，我们可能不需要任何帮助。如果是这种情况，`Math.min()` 就是我们的朋友。在几乎所有其他情况下，`min()` 和 `max()` 函数都是最佳选择，至少因为它们支持回调。让我们看看以下示例：

```js
var collection = [ 
    { name: 'Douglas', age: 52, experience: 5 }, 
    { name: 'Karen', age: 36, experience: 22 },
    { name: 'Mark', age: 28, experience: 6 }, 
    { name: 'Richard', : age: 30, experience: 16 }
];

_.min(collection, 'age'),
// → { name: "Mark", age: 28, experience: 6 }

_.max(collection, function(item) {
    return item.age + item.experience;
});
// → { name: "Karen", age: 36, experience: 22 }
```

第一个调用是`min()`，它接收一个字符串参数——我们想要在集合中获取最小值的属性的名称。这使用了 pluck 风格的回调简写，并产生了简洁的代码，其中你知道你正在处理的属性。前面代码中的第二个调用是`max()`。这个函数支持与`min()`相同的回调简写，但在这里，没有预先存在的属性值供你使用。由于你想要的是`age`属性加上`experience`属性，提供给`max()`的回调函数为我们计算这个值并找出最大值。

注意，`min()`和`max()`函数返回的是实际的集合项，而不是最小或最大值。这很有道理，因为我们可能想要对项本身进行操作，而不仅仅是获取最小/最大值。

除了找到集合的最小和最大值之外，还需要找到集合的实际大小。如果你正在处理数组，这很容易，因为它们已经具有内置的`length`属性。字符串也是如此。然而，对象并不总是具有`length`属性。Lo-Dash 的`size()`函数会告诉你对象有多少键，这是从对象中期望的直观行为，但默认情况下并不存在。看看下面的代码：

```js
var collection = [ 
    { name: 'Gloria' },
    { name: 'Janice' },
    { name: 'Kathryn' },
    { name: 'Roger' }
];  

var first = _.first(collection);
_.size(collection); // → 4
_.size(first); // → 1
_.size(first.name); // → 6
```

`size()`函数的第一个调用返回集合的长度。它会寻找`length`属性，如果集合有一个，则返回这个值。由于它是一个数组，`length`属性存在，并且值为`4`。这就是返回的值。`first`变量是一个对象，因此它没有`length`属性。它会计算对象中的键的数量并返回这个值——在这个例子中，`1`。最后，`size()`被调用在一个字符串上。这个字符串的长度值为`6`。

从`size()`函数的三种用法中我们可以看出，其中几乎没有猜测的成分。在默认的 JavaScript 行为不一致且不直观的情况下，Lo-Dash 提供了一个单一的功能来处理常见的用例。

## 展平和压缩

数组可以嵌套到任意深度，有时会包含没有实际用途的假值。Lo-Dash 有函数来处理这两种情况。例如，我们的 UI 组件可能被传递为一个包含嵌套数组的数组。但我们的组件并不使用这种结构，实际上，它更多的是一种阻碍而不是帮助。我们可以*展平*数组以提取并丢弃组件不需要的不必要结构，如下面的代码所示：

```js
var collection = [ 
    { employer: 'Lodash', employees: [
        { name: 'Barbara' },
        { name: 'Patrick' },
        { name: 'Eugene' }
    ]}, 
    { employer: 'Backbone', employees: [
        { name: 'Patricia' },
        { name: 'Lillian' },
        { name: 'Jeremy' }
    ]}, 
    { employer: 'Underscore', employees: [
        { name: 'Timothy' },
        { name: 'Bruce' },
        { name: 'Fred' }
    ]}  
];  
var employees = _.flatten(_.pluck(collection, 'employees'));

_.filter(employees, function(employee) {
    return (/^[bp]/i).test(employee.name);
});
// → 
// [
//   { name: "Barbara" },
//   { name: "Patrick" },
//   { name: "Patricia" },
//   { name: "Bruce" }
// ]
```

当然，我们实际上并没有改变原始集合的结构，而是在实时构建一个新的集合，使其更适合当前的环境。在先前的例子中，该集合由`employer`对象组成。然而，我们的组件更关注的是`employee`对象。因此，第一步是使用`pluck()`方法将这些对象从其原始对象中提取出来。这会得到一个数组的数组。因为我们实际上提取的是每个`employer`数组中的`employee`数组。

下一步是将这个`employee`数组扁平化为一个`employee`对象的数组，`flatten()`函数可以轻松处理这一点。做所有这些事情的目的，虽然实际上并不多，现在我们有一个易于过滤的结构。特别是，这段代码使用扁平化的集合结构来过滤掉以`b`或`p`开头的员工姓名。

### 注意

另有一个名为`flattenDeep()`的扁平化函数，它可以访问任意嵌套数组的深度以创建一个扁平化的结构。当你需要超出`flatten()`函数查看的一级嵌套时，这很有用。然而，由于性能影响，不建议扁平化未知大小和深度的数组。大型数组结构有很大可能锁定用户的 UI。

`flatten()`函数的近亲是`compact()`函数，通常一起使用。我们将使用`compact()`来从扁平化数组中移除假值，或者仅用于已经存在的普通数组，或者在其过滤之前移除假值。这将在以下代码中展示：

```js
var collection = [ 
    { name: 'Sandra' },
    0,  
    { name: 'Brandon' },
    null,
    { name: 'Denise' },
    undefined,
    { name: 'Jack' }
    ];
var letters = [ 's', 'd' ],
    compact = _.compact(collection),
    result = [];

_.each(letters, function(letter) {
    result = result.concat(
        _.filter(compact, function(item) {
            return _.startsWith(item.name.toLowerCase(), letter);
        })
    );  
});
// → 
// [
//   { name: "Sandra" },
//   { name: "Denise" }
// ]
```

我们可以看到这个集合中有些值是我们显然不想处理的。但，希望不是那么令人悲伤的现实是，在动态类型语言中进行前端开发，并且使用后端数据意味着你无法控制很多合理性检查。前面代码使用`compact()`函数所做的只是从集合中移除任何假值。这些值包括`0`、`null`和`undefined`。实际上，如果没有压缩集合，这段代码甚至无法运行，因为它对集合中每个对象的`name`属性被定义有隐含的假设。

`compact()`不仅可以用于安全目的——移除违反约定的项——还可以用于性能目的。你会看到前面的代码在循环内部搜索集合。因此，在进入外部循环之前从集合中移除的任何项，性能提升就越大。

回到前面的代码，有一个问题可能会让 Lo-Dash 程序员感到意外。假设我们不想有任何没有`name`属性的东西。嗯，我们只是在移除假值——没有`name`属性的对象仍然是有效的，`compact()`函数允许它们通过。例如，`{}`没有`name`属性，`2`也没有，但它们在上一个方法中都被允许通过。一个更安全的方法可能是先提取再压缩，如下面的代码所示：

```js
var collection = [ 
    { name: 'Sandra' },
    {}, 
    { name: 'Brandon' },
    true,
    { name: 'Denise' },
    1,  
    { name: 'Jack' }
    ];
var letters = [ 's', 'd' ],
    names = _.compact(_.pluck(collection, 'name')),
    result = [];

_.each(letters, function(letter) {
    result = result.concat(
        _.filter(names, function(name) {
            return _.startsWith(name.toLowerCase(),
                                letter);
        })  
    );  
});
```

在这里，我们面临一个类似的过滤任务，但集合略有不同。它包含一些会导致我们的代码失败的对象，因为这些对象没有具有字符串值的`name`键。快速而简单的解决方案是在执行`compact()`调用之前从集合中的所有项目提取`name`属性。这将导致没有`name`属性的对象返回`undefined`值。但这正是我们想要的，因为`compact()`可以轻松排除这些值。此外，我们的代码现在实际上更简单了。需要注意的是，有时简单的方法不起作用。有时，你需要完整的对象，而不仅仅是名称。只有在可以逃避的情况下才作弊。

## 验证某些或所有项目

有时，我们的代码的某些部分取决于所有或某些集合项目的有效性。Lo-Dash 为您提供了两个互补的工具来完成这项工作。`every()`函数如果回调对集合中的每个项目都返回`true`，则返回`true`。`some()`函数是`every()`的懒惰兄弟——它一旦回调对某个项目返回`true`，就提供并返回`true`，如下面的代码所示：

```js
var collection = [ 
    { name: 'Jonathan' },
    { first: 'Janet' },
    { name: 'Kevin' },
    { name: 'Ruby' }
];

if (!_.every(collection, 'name')) {
    return 'Missing name property';
}
// → "Missing name property"
```

此代码在对其进行任何操作之前检查集合中的每个项目是否有`name`属性。由于其中一个项目使用了错误的属性名称，代码将提前返回。在`if`语句下面的代码可以假设每个项目都有一个`name`属性。

另一方面，我们可能只想知道是否有任何项目具有必要的值。您可以使用这项技术来显著提高性能。例如，假设您有一个循环，它对每个集合项目执行昂贵的操作。您可以进行一个相对便宜的**预检**，以确定是否值得运行这个昂贵的循环。以下是一个示例：

```js
var collection = [ 
    { name: 'Sean' },
    { name: 'Aaron' },
    { name: 'Jason' },
    { name: 'Lisa' }
];
if (_.some(collection, 'name')) {
    // Perform expensive processing...
}
```

如果`some()`调用遍历整个集合而没有任何`true`回调返回值，这意味着我们可以跳过更昂贵的处理。例如，如果我们有一个可能很大的集合，并且我们需要使用一些非平凡的比较运算符来过滤它，也许还需要一些函数调用，那么开销真的开始增加。如果不需要，使用`some()`是一种避免这种重处理的好方法。

## 并集、交集和差集

本章的最后部分探讨了 Lo-Dash 函数，这些函数可以比较两个或多个数组并生成一个结果数组。从某种意义上说，我们正在将多个集合合并成一个集合。`union()` 函数连接集合，同时移除重复值。`intersection()` 函数构建一个包含所有提供集合共有值的集合。最后，`xor()` 函数构建一个包含所有提供集合差异的集合。这有点像是 `intersection()` 的逆操作。

当有多个重叠的集合包含相似的项目——可能是相同的项目时，可以使用 `union()` 函数。与其逐个遍历每个集合，不如将集合合并，同时移除重复项，如下面的代码所示：

```js
var css = [ 
    'Philip',
    'Donald',
    'Mark'
];  
var sass = [ 
    'Gary',
    'Michelle',
    'Philip'
];  
var less = [ 
    'Wayne',
    'Ruth',
    'Michelle'
];

_.union(css, sass, less);
// →
// [
//   "Philip",
//   "Donald",
//   "Mark",
//   "Gary",
//   "Michelle",
//   "Wayne",
//   "Ruth"
// ]
```

这段代码将三个数组转换成一个单一的数组。你可以看到在结果数组中没有重叠。也就是说，存在于输入数组中的任何项目在结果数组中只包含一次。让我们看看使用 `intersection()` 如何查看重叠的情况：

```js
var css = [ 
    'Rachel',
    'Denise',
    'Ernest'
];  

var sass = [ 
    'Lisa',
    'Ernest',
    'Rachel'
];  

var less = [ 
    'Ernest',
    'Rachel',
    'William'
];

_.intersection(css, sass, less);
// → [ "Rachel", "Ernest" ]
```

在这里，交集是 `Ernest` 和 `Rachel`，因为这些字符串存在于传递给 `intersection()` 的所有三个集合中。现在，让我们看看如何使用 `xor()` 比较两个集合之间的差异：

```js
var sass = [ 
    'Lisa',
    'Ernest',
    'Rachel'
];
var less = [ 
    'Ernest',
    'Rachel',
    'William'
];

return _.xor(sass, less);
// → [ "Lisa", "William" ]
```

将这两个数组传递给 `xor()` 将会生成一个新的数组，该数组包含两个数组之间的差异。在这种情况下，差异是 `Lisa` 和 `William`。其余的都是交集。

### 注意

`xor()` 函数可以接受任意数量的集合进行比较。然而，在比较超过两个集合时，请谨慎行事。最常见的情况是比较两个集合以找出它们之间的差异。超出这个范围就是进入集合论领域，你可能会得到你预期的结果。

# 摘要

本章向您介绍了集合的概念以及它们如何与数组进行比较。Lo-Dash 将集合视为一个抽象概念——所有 JavaScript 数组都是集合，但并非所有集合都是数组。我们通过 Lo-Dash 提供的工具了解了遍历集合的概念——这是应用编程中的一个基本概念，本书中将会经常提及。

集合可以被过滤，并且可以从集合中检索项目。Lo-Dash 还为你提供了将集合转换为在实现前端 UI 组件时所需的其它结构的工具。

我们已经尝到了 Lo-Dash 编程中的一些常见主题——比如回调函数几乎在所有事情中都处于核心地位的想法，以及可以节省编码努力的多种简写，例如 pluck 和 where 回调。现在，让我们看看 Lo-Dash 如何与对象一起工作，以及我们可用的各种函数，这些内容将在下一章中介绍。
