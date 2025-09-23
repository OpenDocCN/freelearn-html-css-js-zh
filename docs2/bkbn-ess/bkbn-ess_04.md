# 第四章：使用 Collections 组织 Models

在本章中，我们将学习以下内容：

+   创建新的 `Collection` 子类

+   向 Collection 中添加和移除 Models

+   在 Collection 的变化时触发其他代码

+   将 `Collections` 的 `Models` 存储和检索到远程服务器

+   对 Collection 中的 Models 进行沙箱索引

+   利用从 Underscore 借来的便利方法

# 与 Collections 一起工作

在 Backbone 中，Models 可能是所有数据交互的核心，但很快你就会发现你需要与多个 Models 一起工作才能完成任何实际的工作，这就是 `Collections` 类发挥作用的地方。就像 Model 包装一个对象并提供额外的功能一样，`Collections` 包装一个数组，并且在使用数组时提供了几个优势：

+   `Collections` 使用 Backbone 的类系统，这使得定义方法、创建子类等变得容易

+   `Collections` 允许其他代码在 `Models` 被添加或从该 `Collection` 中移除，或者当 `Collection` 中的 `Models` 被修改时进行监听和响应

+   `Collections` 简化和封装了与服务器通信的逻辑

我们可以创建一个新的 `Collection` 子类，如下所示：

```js
var Cats = Backbone.Collection.extend({
     // The properties and methods of Cats would go here
});
```

一旦我们创建了一个子类，我们可以使用 JavaScript 的 `new` 关键字实例化它的新实例，就像我们创建新的 `Model` 实例一样。与 Models 类似，Collections 有两个可选参数。第一个参数是一个初始的 `Models` 或 `Model` 属性数组，第二个参数是 Collection 的选项。如果你传递一个 `Model` 属性数组，Backbone 会为你将它们转换为 Models，如下所示：

```js
var cartoonCats = new Cats([{id: 'cat1', name: 'Garfield'}]);
var garfield = cartoonCats.models[0]; // garfield is a  Model
```

一旦创建了一个 `Collection` 子类，它将它的 Model 存储在一个名为 `models` 的隐藏属性中。与属性类似，模型不应该直接使用，而应该依赖 `Collection` 的方法来与其 Models 一起工作。Backbone 还为每个 `Collection` 提供了一个 `length` 属性，该属性始终设置为 `Collection` 子类中 Models 的数量：

```js
var cartoonCats = new Cats([{name: 'Garfield'}, {name: 'Heathcliff'}]);
cartoonCats.models.length; // 2, but instead you can do ...
cartoonCats.length; // also 2
```

# Collections 和 Models

每个 `Collection` 类都有一个 `model` 属性，默认为 `Backbone.Model`。您可以通过在创建 `Collection` 子类时将其作为选项提供来设置此属性：

```js
var Cats = Backbone.Collection.extend({model: Cat});
var cats = new Cats(); // cats.model == Cat
```

然而，这个 `model` 属性实际上并不限制 `Collection` 可以持有的 Model 类型，实际上，任何 `Collection` 都可以持有任何类型的 `Model`：

```js
var Cat = Backbone.Model.extend();
var Cats = Backbone.Collection.extend({model: Cat});
var Dog = Backbone.Model.extend();
var snoopy = new Dog({name: 'Snoopy'});
var cartoonCats = new Cats([snoopy]);
cartoonCats.models[0] instanceof Dog; // true
```

这是因为 `Collection` 的 `model` 属性仅在通过 `Collection` 创建新的 Models 时使用。这种情况可能发生的一种方式是，当 `Collection` 使用一个属性数组初始化时，如下面的示例所示：

```js
var snoopyAttributes = {id: 'dog1', name: 'Snoopy'};
var cartoonCats = new Cats([snoopyAttributes]);
cartoonCats.models[0] instanceof Cat; // true
```

另一种使用 `Collection` 的 `model` 属性的方法是通过 `Collection` 的 `create` 方法。在 `Collection` 上调用 `create` 并传递一个 `attributes` 对象，本质上等同于调用 `new Collection.model(attributes)`，只不过在提供的属性被转换为 `Model` 之后，Backbone 会将这个 `Model` 添加到 `Collection` 中并保存它：

```js
var cartoonCats = new Cats();
var garfield = cats.create({name: 'Garfield'}); // equivalent to:
// var garfield = new Cat({name: 'Garfield'});// cats.models.push(garfield);
// garfield.save();
```

# 添加和重置集合

除了在我们首次创建 `Collection` 时传入 `Models` 或 `attributes`，我们还可以通过 `add` 方法将单个 `Models` 或 `attributes`，或 `Models` 或 `attributes` 的数组添加到 `Collection` 中：

```js
var cats = new Backbone.Collection();
cats.add({name: 'Garfield'});
cats.models[0] instanceof Cat; // true
```

与 `create` 类似，`add` 方法也会使用 `Collection` 的 `model` 属性来创建结果 `Models`。如果您想替换 `Collection` 中现有的所有 `Models` 而不是添加更多，可以使用 `reset` 方法，如下所示：

```js
var cats = new Backbone.Collection([{name: 'Garfield'}]);
cats.reset([{name: 'Heathcliff'}]);
cats.models[0].get('name'); // "Heathcliff"
cats.length; // 1, not 2, because we replaced Garfield with Heathcliff
```

# 索引

为了从 `Collection` 中获取或删除特定的 `Models`，Backbone 需要知道如何索引这些 `Models`。Backbone 以两种方式完成此操作：

+   使用模型的 `id` 属性（如果有的话）（正如我们在上一章中讨论的，可以直接或通过设置模型的 `idAttribute` 属性间接设置）

+   使用模型的 `cid` 属性（所有模型都有）

当您将 `Model` 或 `attributes` 添加到 `Collection` 中时，Backbone 使用前面提到的两种识别形式在 `Collection` 的 `_byId` 属性中注册 `Model`。`_byId` 是 Backbone 的另一个隐藏属性，但也是一个 `private` 属性（因为其名称以 `_` 为前缀）。这意味着，与其他隐藏属性相比，您应该避免直接使用 `_byId`，而应通过 `get` 等方法间接使用它。`get` 方法通过 `_byId` 返回具有提供 ID 的 `Model`（如果有的话）：

```js
var garfield = new Cat({id: 'cat1', name: 'Garfield'});
var cats = new Backbone.Collection([garfield]);
cats.get('cat1'); // garfield
cats.get(garfield.cid); // also garfield
cats.get('cat2'); // returns undefined
```

`_byId` 属性还可以在另一个 `Collection` 方法中间接使用，该方法为 `remove`。正如其名称所暗示的，`remove` 会从 `Collection` 中移除具有提供 ID 的 `Model`：

```js
var cats = new Backbone.Collection([{
    id: 'cat1',    name: 'Garfield'
}]);
cats.remove('cat1');
cats.length; // 0
```

您还可以使用您已经熟悉的集合方法 `get` 和 `remove` 从 `Collection` 中获取和删除 `Models`，因为它们也存在于 JavaScript 的数组中（并且以相同的方式工作）。以下是您可以使用的方法：

+   `push`

+   `pop`

+   `unshift`

+   `shift`

+   `slice`

最后，如果您想使用索引而不是 `id` 来从 `Collection` 中检索 `Model`，可以使用 `at`，这是 `Collection` 的最后一个模型访问方法。`at` 方法接受一个零基索引并返回该索引处的 `Model`（如果有的话）：

```js
var cats = new Backbone.Collection([
    {name: 'Garfield'},    {name: 'Heathcliff'}
]);
cats.at(1); // returns heathcliff
```

# 排序

如果您通过指定比较器来告诉 Backbone，它将自动对所有添加到 `Collection` 中的 `Models` 进行排序。与模型一样，当创建 `Collection` 类时，可以将其比较器作为选项提供：

```js
var Cats = Backbone.Collection.extend({comparator: 'name'});
var cartoonCats = new Cats();
cartoonCats.comparator; // 'name'
```

虽然提供的比较器控制着集合如何内部排序其模型，但你也可以使用 Underscore 的排序方法之一来对集合进行排序，我们将在本章末尾详细说明。

比较器本身可以采用三种形式。第一种也是最简单的一种形式是集合中模型的属性名称。如果使用这种形式，Backbone 将根据指定属性的值对集合进行排序：

```js
var cartoonCats = new Backbone.Collection([
    {name: 'Heathcliff'},
    {name: 'Garfield'}
], {
    comparator: 'name'
});
cartoonCats.at(0);// garfield, because his name sorts first alphabetically
```

比较器的第二种形式是一个接受单个参数的函数。如果使用这种形式，Backbone 将逐个将任何要排序的模型传递给该函数，然后使用返回的任何值来排序模型。例如，如果你想按字母顺序对模型进行排序，除非该`Model`是`Heathcliff`，你可以使用这种形式的比较器：

```js
var Cats = Backbone.Collection.extend({
    comparator: function(cat) {
        if (cat.get('name') == 'Heathcliff') return '0';
        return cat.get('name');
    }
});
var cartoonCats = new Cats([ 
    {name: 'Heathcliff'},    {name: 'Garfield'}
]);
cartoonCats.at(0); // heathcliff, because "0" comes before "garfield" alphabetically
```

`comparator`的最终形式是一个接受两个`Model`参数并返回一个数字的函数，表示第一个参数应该在第二个参数之前（`-1`）还是之后（`1`）。如果没有对哪个模型应该排在第一的偏好，函数将返回`0`。我们可以使用这种风格实现相同的第一个`Heathcliff`比较器：

```js
var Cats = Backbone.Collection.extend({
    comparator: function(leftCat, rightCat) {
        // Special case sorting for Heathcliff
        if (leftCat.get('name') == 'Heathcliff') return -1;
        // Sorting rules for all other cats
        if (leftCat.get('name') > rightCat.get('name')) return 1;
        if (leftCat.get('name') < rightCat.get('name')) return -1;
        if (leftCat.get('name') == rightCat.get('name')) return 0;
    }
});
var cartoonCats = new Cats([ 
    {name: 'Heathcliff'},    {name: 'Garfield'}
]);
cartoonCats.at(0);// heathcliff, because any comparison of his name will return -1
```

# 事件

就像模型一样，集合也有`on`和`off`方法，可以在集合中发生某些事件时触发逻辑。下表显示了可能发生的事件：

| 事件名称 | 触发 |
| --- | --- |
| `add` | 当模型或模型被添加到集合中时 |
| `remove` | 当模型或模型被从集合中移除时 |
| `reset` | 当集合被重置时 |
| `sort` | 当集合被排序时（通常在添加/移除之后） |
| `sync` | 当集合的 AJAX 方法完成时 |
| `error` | 当集合的 AJAX 方法返回错误时 |
| `invalid` | 当由模型的`save`/`isValid`调用触发的验证失败时 |

集合，如模型一样，也有一个特殊的`all`事件，可以用来监听集合上发生的任何事件。此外，集合的`on`方法也可以用来监听由集合中的模型触发的任何模型事件，如下所示：

```js
var cartoonCats = new Cats([{name: 'Garfield'}]);
cartoonCats.on('change', function(changedModel) {
    alert( changedModel.get('name') + ' just changed!');
});
cartoonCats.at(0).set('weight', 'a whole lot'); // alerts "Garfield just changed!"
```

# 服务器端操作

就像模型一样，集合有一个`fetch`方法，用于从服务器检索数据，以及一个`save`方法用于将数据发送到服务器。然而，有一个小差异，即默认情况下，集合的`fetch`将合并来自服务器的新数据与它已经拥有的任何数据。如果你希望完全用服务器数据替换本地数据，可以在获取数据时传递一个`{reset: true}`选项。集合还有`url`、`parse`和`toJSON`方法，这些方法控制着`fetch`/`save`的工作方式。

所有这些方法的工作方式与在模型上一样。然而，集合没有 `urlRoot`；虽然集合内部的模型可能有 ID，但集合本身没有，因此集合不需要使用 `.urlRoot` 生成它们的 URL：

```js
var Cats = Backbone.Collection.extend({
     url: '/cats'
});
var cats = new Cats({name: 'Garfield'});
cats.save(); // saves Garfield to the server
cats.fetch(); // retrieves cats from the server and adds them
```

# 下划线方法

`Collections`，就像 `Models` 一样，从 Underscore 继承了一些方法，但集合实际上比模型有更多方法...确切地说有 28 个方法。我们不可能在这里详细说明所有这些方法，所以我将简要概述每个方法的作用，然后深入探讨一些更重要的一些方法。

注意，就像在模型上的下划线方法一样，其中一些方法是在 `Collection` 内部的 `Models` 的属性上操作，而不是在 `Collection` 或 `Models` 本身上操作。然而，同时，所有返回多个结果的方法都返回普通的 JavaScript 数组，而不是新的集合。这样做是为了性能原因，并且不应该成为问题，因为如果您需要将这些结果作为集合，您可以简单地创建一个新的集合并将结果数组传递进去。

| 名称 | 它的作用 |
| --- | --- |
| `each` | 这遍历集合中的每个模型 |
| `map` | 这通过转换集合中的每个模型返回一个值数组 |
| `reduce` | 这返回由集合中的所有模型生成的一个单一值 |
| `reduceRight` | 与 `reduce` 相同，但它按反向迭代 |
| `find` | 这返回第一个与测试函数匹配的模型 |
| `filter` | 这返回所有与测试函数匹配的模型 |
| `reject` | 这返回所有不匹配测试函数的模型 |
| `every` | 如果集合中的每个模型都匹配测试函数，则返回 `true` |
| `some` | 如果集合中的某些（任何）模型与测试函数匹配，则返回 `true` |
| `contains` | 如果集合包含提供的模型，则返回 `true` |
| `invoke` | 这将在集合中的每个模型上调用指定的函数，并返回结果 |
| `max` | 这会将转换函数应用于集合中的每个模型，并返回返回的最大值 |
| `min` | 这会将转换函数应用于集合中的每个模型，并返回返回的最小值 |
| `sortBy` | 这根据指示的属性返回排序后的集合模型 |
| `groupBy` | 这根据指示的属性返回集合模型的分组 |
| `shuffle` | 这返回从集合中随机选择的一个或多个模型 |
| `toArray` | 这返回集合模型的数组 |
| `size` | 这返回集合中模型的计数，例如 `Collection.length` |
| `first` | 这返回集合中的第一个（或前 N 个）模型 |
| `initial` | 这返回集合中的所有模型，除了最后一个 |
| `rest` | 这返回集合中第一个 N 个模型之后的所有模型 |
| `last` | 返回 `Collection` 中最后一个（或最后 N 个）`Model` |
| `without` | 返回 `Collection` 中除了提供的 `Model` 之外的所有 `Model` |
| `indexOf` | 返回 `Collection` 中第一个提供的 `Model` 的索引 |
| `lastIndexOf` | 返回 `Collection` 中最后一个提供的 `Model` 的索引 |
| `isEmpty` | 如果 `Collection` 不包含任何 `Model`，则返回 `true` |
| `chain` | 返回一个可以连续调用多个 `Underscore` 方法的 `Collection` 版本（链式调用）；在链的末尾调用 `value` 方法以获取调用结果 |
| `pluck` | 返回 `Collection` 中每个 `Model` 的提供的属性 |
| `where` | 返回 `Collection` 中所有与提供的属性模板匹配的 `Model` |
| `findWhere` | 返回 `Collection` 中第一个与提供的属性模板匹配的 `Model` |

# 之前提到的 `methods`

前面的列表中的一些方法你应该已经很熟悉了，因为我们已经在 第二章 *使用 Backbone 类的面向对象 JavaScript* 中解释了 `each`、`map`、`invoke`、`pluck` 和 `reduce` 方法。如果你调用它们的 `Underscore` 版本，并将 `Collection.models` 传递给它们，这些方法将按相同的方式工作，如下所示：

```js
var cats = new Backbone.Collection([ 
    {name: 'Garfield'},    {name: 'Heathcliff'}
]);
cats.each(function(cat) {
    alert(cat.get('name'));
}); // will alert "Garfield", then "Heathcliff"
```

# 测试方法

剩余的一些方法主要关注测试 `Collection` 是否通过某种类型的测试。`contains` 和 `isEmpty` 方法允许你检查 `Collection` 是否包含指定的 `Model` 或 `Models`，或者是否包含任何模型：

```js
var warAndPeace = new Backbone.Model();
var books = new Backbone.Collection([warAndPeace]);
books.contains(warAndPeace); // true
books.isEmpty(); // false
```

对于更高级的测试，你可以使用 `every` 和 `some` 方法，这些方法允许你指定自己的测试逻辑。例如，如果你想知道 `Collection` 中的任何书籍是否超过了一百页，你可以使用 `some` 方法，如下所示：

```js
var books = new Backbone.Collection([
    {pages: 120, title: "Backbone Essentials 4: The Reckoning"},
    {pages: 20, title: "Even More Ideas For Fake Book Titles"}
]);
books.some(function(book)  {
    return book.get('pages') > 100;
}); // true
books.every(function(book)  {
    return book.get('pages') > 100;
}); // false
```

# 提取方法

另一种使用 `Underscore` 方法的方式是从 `Collection` 中提取特定的 `Model` 或 `Models`。最简单的方法是使用 `where` 和 `findWhere` 方法，它们返回所有（或 `findWhere` 的情况下，第一个）与提供的属性对象匹配的 `Model`。例如，如果你想从 `Collection` 中提取所有恰好有一百页的书籍，你可以使用 `where` 方法，如下所示：

```js
var books = new Backbone.Collection([
    { 
        pages: 100,        title: "Backbone Essentials 5: The Essentials Return"
    }, {
        pages: 100,        title: "Backbone Essentials 6: We're Not Done Yet?"
    },{
        pages: 25,        title: "Completely Fake Title"
    } 
]);
var hundredPageBooks = books.where({pages: 100});
//  hundredPageBooks array of all of the books except Completely Fake Title
var firstHundredPageBook = books.findWhere({pages: 100});
firstHundredPageBook; // Backbone Essentials 5: The Essentials Return
```

如果我们需要更复杂的筛选怎么办？例如，如果我们不想提取恰好有一百页的所有书籍，而是想提取任何有一百页或更多页的书籍，我们应该怎么办？对于这种提取，我们可以使用更强大的 `filter` 方法，或者它的逆方法 `reject` 方法，如下所示：

```js
var books = new Backbone.Collection([
    { 
        pages: 100,        title: "Backbone Essentials 5: The Essentials Return"
    }, {
        pages: 200,        title: "Backbone Essentials 7: How Are We Not Done Yet?"
    }, {
        pages: 25,        title: "Completely Fake Title"
    }
]);
var hundredPageOrMoreBooks = books.filter(function(book)  {
    return book.get('pages') >= 100;
});
hundredPageOrMoreBooks; // again, this will be an array of all books but the last
var hundredPageOrMoreBooks = books.reject(function(book)  {
    return book.get('pages') < 100;
});
hundredPageOrMoreBooks; // this will still be an array of all books but the last
```

# 排序方法

最后，我们有 `toArray`、`sortBy` 和 `groupBy`，所有这些都可以让您获取存储在 `Collection` 中的所有 `Models` 的数组。然而，虽然 `toArray` 只简单地返回集合中的所有模型，`sortBy` 则根据提供的标准返回排序后的模型，而 `groupBy` 则将模型分组到更高级别的数组中。例如，如果您想获取一个按字母顺序排序的集合中的所有书籍，可以使用 `sortBy`：

```js
var books = new Backbone.Collection([
    {title: "Zebras Are Cool"},
    {title: "Alligators Are Also Cool"},
    {title: "Aardvarks!!"}
]);
var notAlphabeticalBooks = books.toArray();
notAlphabeticalBooks;// will contain Zebras, then Alligators, then  Aardvarks
var alphabeticalBooks = books.sortBy('title');
alphabeticalBooks;// will contain Alligators, then Aardvarks, then Zebras
```

如果您想根据标题的首字母将它们组织成组，可以使用 `groupBy`，如下所示：

```js
var firstLetterGroupedBooks = books.groupBy(function(book) {
    return book.get('title')[0];
});
// will be an array of [Alligators,  Aardvarks], [Zebras]
```

# 摘要

在本章中，我们探讨了 Backbone 的 `Collection` 类。您学习了如何添加和删除模型和 `Model` 属性，如何使用 `on` 和 `off` 方法来监听事件，如何控制集合的排序和索引，以及如何使用 `fetch` 和 `save` 与远程服务器交换数据。我们还考察了集合的许多 `Underscore` 方法以及它们如何被用来实现集合的全部功能。

在下一章中，我们将探讨 Backbone 的 `View` 类。视图允许您使用我们已介绍过的模型和集合的数据来渲染一个 HTML 页面，或者其子集。
