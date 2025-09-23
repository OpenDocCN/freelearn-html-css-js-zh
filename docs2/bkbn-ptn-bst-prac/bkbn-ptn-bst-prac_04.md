# 第四章：处理集合

Backbone 集合的目的非常直接。作为一个有序的模型集合，集合提供了一系列有用的方法来操作，包括一组 Underscore.js 工具方法。集合包括添加、删除、排序和过滤模型的功能，以及保存到或从服务器获取数据。集合监听其模型上触发的事件——如果集合的模型上触发了一个事件，它也会在集合本身上触发。当你想监听模型的属性更改事件时，这个功能相当重要。我们将通过本章的 *集合的基本用法* 部分中的示例来探讨它。

在前面的章节中，我们看到了许多简单集合的实现，用于在列表视图中显示多个项目。然而，可能存在你想要根据多个标准对列表进行排序或你想要过滤列表项以仅显示特定类型的情况。在这种情况下，你必须修改集合以重新结构化模型位置或获取符合过滤条件的数据。让我们看看在本章中我们将学习什么：

+   **集合的基本用法**：理解 Backbone 集合的基本用法以及与集合的数据操作

+   **排序集合**：集合的基本和多重排序

+   **过滤集合**：执行基本过滤，避免使用重复的集合重新过滤，以及使用完整数据指针进行过滤

+   **包含多种模型类型的集合**：当从服务器传递混合数据集且每种类型属于不同的模型时管理集合

# 集合的基本用法

我们将通过一个简单的示例开始研究 `Backbone` 集合的不同特性。假设我们有一个 `User` 模型和 `Users` 集合。

```js
// Model definition
var User = Backbone.Model.extend({
  initialize: function () {
    this.on('change', function () {
      console.log('User model changed!');
    });
  }
});

// Collection definition
var Users = Backbone.Collection.extend({
  model: User,
  url : '/users',
  initialize: function () {
    this.on('change', function () {
      console.log('Users collection changed!');
    });
  }
});

var users = new Users(),
  newUser = new User({
    name: 'Jayashi De',
    age: 21
  });

users.add([newUser]);

// Change an attribute of the model
newUser.set('age', 22);
```

在前面的代码中，已经描述了简单的模型和集合定义。在这里，我们试图证明，正如我们在本章引言中提到的，任何在模型上触发的事件也会在集合上触发。当你运行此代码时，模型更改处理程序首先运行，然后是集合更改事件处理程序。

`Backbone` 集合提供了一组丰富的方法，以及一系列 `Underscore` 工具方法来操作它。由于集合处理多个数据，你会发现 `Underscore` 工具方法在操作它时非常有用。讨论所有这些方法和它们的函数性超出了本书的范围，但我们将看到如何使用 `fetch()` 和 `save()` 方法通过 AJAX 请求从服务器检索数据并将数据保存到服务器，在下一节中。

## 使用集合执行数据操作

您可以使用 AJAX 请求将数据保存到服务器并从服务器获取数据。然后需要将结果应用到集合上。然而，Backbone 通过提供一些方法，如`fetch()`和`save()`，简化了与服务器直接交互的整个过程。我们将使用之前章节中使用的相同集合来演示如何使用集合执行所有数据操作。

### 从服务器获取数据

从服务器获取数据相当简单。您只需在集合上调用`fetch()`方法，如下面的代码行所示，然后向我们在集合配置中添加的 URL 发送一个`GET`请求。它接收一个包含对象的`JSON`数组，这些对象被添加到集合中作为模型。

```js
users.fetch();
```

在接收到数据后，集合的`set()`方法会自动被调用以更新集合。如果不存在模型，则将其添加；如果模型已存在，则最新数据与它合并；如果集合中有一个模型在新数据中不存在，则将其移除。

### 将数据保存到服务器

与从服务器获取数据不同，集合没有将数据作为一个整体存储到服务器的方法。相反，需要调用每个单独模型的`save()`方法，如下面的代码片段所示：

```js
var user = users.get(1);
user.save();
```

`save()`方法将模型的 ID 附加到服务器的 URL（`/users/1`）并向该 URL 发送一个`PUT`请求。因此，要将整个集合保存到服务器，你需要遍历集合并对每个模型单独调用`save()`方法。

# 对集合进行排序

使用 Backbone 按多个属性对集合进行排序相当简单，因为已经内置了用于此目的的方法。要对集合进行排序，向集合中添加一个`comparator`，它通常是一个可以接受单个模型或两个连续模型进行比较的函数，或者它可以是指向其模型属性的字符串。每当模型被添加到集合中时，比较器就会相应地排序集合。稍后更改模型的属性不会自动启动排序功能，您需要再次在集合上调用`sort()`方法以重新排序。让我们看看排序集合的一个简单示例：

```js
var User = Backbone.Model.extend();

var Users = Backbone.Collection.extend({
  model: User,
  comparator: 'age'
});

var users = new Users();
users.add([{
  name: 'John Doe', 
  age: 29
}, {
  name: 'Richard Smith',
  age: 35
}, {
  name: 'Swarnendu De',
  age: 29
}, {
  name: 'Emily Johnson',
  age: 25
}, {
  name: 'Sarah Castle',
  age: 40
}, {
  name: 'Ben Cooper',
  age: 29
}]);

console.log(users.pluck('name')); // ["Emily Johnson", "John Doe", // "Swarnendu De", "Ben Cooper", "Richard Smith", "Sarah Castle"]

console.log(users.pluck('age')); // 25, 29, 29, 29, 35, 40
```

如您所见，输出是一个按年龄排序的集合。同样，我们可以通过将`name`作为比较器来实现字母顺序。前述功能也可以通过以下两种选项来复制：

```js
// Underscore's sortBy() comparator
comparator: function (model) {
  return model.get('age');
}

// Underscore's sort() comparator
comparator: function (model1, model2) {
  return model1.get('age') < model2.get('age');
}
```

第一种情况很简单；它向比较器提供了一个字符串属性。第二种情况提供了两个模型（`model1`和`model2`）之间的比较。如果`model1`的`age`属性大于`model2`的`age`属性，则`model1`和`model2`将互换位置以保持升序。

## 按多个属性对集合进行排序

注意，在前一节提到的示例中，有三个 29 岁的值。如果我们想根据`name`属性对这些模型进行排序怎么办？字符串比较也可以像我们为数字做的那样进行；功能将是简单的（见以下示例）：

```js
comparator: function (model1, model2) {
  // If age is same, then sort by name
  if (model1.get('age') === model2.get('age')) {
    return model1.get('name') > model2.get('name');
  } else {
    return model1.get('age') > model2.get('age');
  }
}  

console.log(users.pluck('name'));
console.log(users.pluck('age'));
```

上述代码仅按`name`属性对集合进行排序。如果有两个模型的年龄值相同，结果将如下所示：

```js
["Emily Johnson", "Ben Cooper", "John Doe", "Swarnendu De", "Richard Smith", "Sarah Castle"]
[25, 29, 29, 29, 35, 40]
```

当集合中存在比较器并且向其中添加数据时，总会触发一个`sort`事件。当你专门在集合上调用`sort()`方法时，它也会被触发。

# 过滤集合

过滤集合是一个相当简单的概念；在这里，我们希望根据某些标准获取数据的一部分。例如，如果你有一个项目列表，并且你只想显示所有项目的一个子集，你可以过滤附加的集合。默认情况下，Backbone 提供了一些内置函数来处理基本过滤。`where()`和`findWhere()`方法产生类似的功能，尽管`findWhere()`只返回第一个符合条件的数据模型。

## 执行基本过滤

`where()`方法接受一组模型属性并返回匹配的模型数组。

```js
var users = new Backbone.Collection([
  { name: 'John', company: 'A' }, 
  { name: 'Bill', company: 'B' }, 
  { name: 'Rick', company: 'A' }
]);

users.where({
  company: 'A'
});
```

结果将是一个包含两个模型的数组，它们的公司是`A`。然而，请注意，过滤集合并不会改变原始集合数据；相反，它只是返回一个包含结果的数组。如果有一个 Backbone 视图正在将集合数据作为列表显示，过滤集合对列表没有任何影响。

那么，我们该如何解决这个问题呢？一个简单的任务可以是——让我们用过滤后的数据重置集合并重新渲染列表。以下代码将正常工作，并且集合将只包含过滤后的数据：

```js
var filteredData = users.where({
  company: 'A'
});

// Reset the collection with array with filtered data
users.reset(filteredData);

// A collection with only filtered data
console.log(users);
```

现在，如果你重新渲染列表，它将只显示过滤后的数据。这看起来不错；然而，如果你想要重新过滤集合，它将不会应用于完整的数据集合，而是应用于之前过滤的数据。这是错误的；建议避免可能导致后期严重问题的模式。当你只过滤一次集合时，不应该有任何问题，但如果相同的集合在其他地方也被使用，多次过滤集合肯定会引起问题。例如，以下代码将因为相同的原因返回零结果：

```js
users.where({
  company: 'B'
});
```

让我们尝试一些选项来找到这个问题的解决方案。我们首先尝试使用重复集合。

## 使用重复集合过滤集合

对于上一节中我们发现的问题，存在多种解决方案。例如，每当集合被过滤时，我们可以创建另一个集合实例，并且始终用过滤后的数据重置这个第二个集合。这样，主集合就不会被改变，将第二个集合传递给视图实例将产生期望的结果。

```js
var filteredData = users.where({
  company: 'A'
});

// Create a new collection that will only hold filtered data
var filteredCollection = new Backbone.Collection();

// Reset this collection every time 
// there is a new set of filtered data
filteredCollection.reset(filteredData);

console.log(filteredCollection, users);
```

这次，原始集合保持其状态，而新的过滤集合提供了必要的功能。这个过程对于显示过滤后的数据集可以非常有益。这个过程的主要缺点是，你需要创建集合的另一个新实例来过滤它。

## 使用完整数据指针的自过滤

在应用过滤器之前保留完整数据集的引用也可以消除多次过滤的缺点。如果我们能在集合本身的一个属性中保存初始数据，然后对其应用过滤器，集合数据会发生变化，但原始数据仍然可用。因此，如果我们需要重新在集合上应用另一个过滤器，我们可以首先使用全部数据重置它，然后应用新的过滤器。为了通过示例理解这个概念，我们将定义一个自定义的`FilterCollection`类：

```js
var FilterCollection = Backbone.Collection.extend({
  _totalData: [],
  _isFiltered: false,

  initialize: function (data) {
    // The initial data sent to collection will be saved
    if (data) {
      this._setTotalData(data);
    }

    // If some data is added later, 
    // that should reflect in _totalData 
    this.on('add', function () {
      this._setTotalData();
    }, this);
  },

  // Every time a new data has been added to the collection
  _setTotalData: function (data) {
    this._totalData = data || this.toJSON();
  },

  // Apply a new filter to the collection
  applyFilter: function (criteria) {
    // Clear the previous filter
    this.clearFilter();

    // Apply new filter 
    this.reset(this.where(criteria));

    // Mark this as filtered
    this._isFiltered = true;
  },

  // Clear all filters applied to this collection
  clearFilter: function () {
    // skip first reset event while the collection 
    // has the original data
    if (this._isFiltered) {
      // Reset the collection with complete data set
      this.reset(this._totalData);
      this._isFiltered = false;
    }
  }
});
```

我们创建一个自定义的`Collection`类，它有一个`_totalData`属性，这个属性应该包含该集合的全部数据。在`initialize`方法中，我们检查是否已向集合传递了任何数据；如果有，我们将该数据保存到这个变量中。我们还包括一个`add`事件监听器，以便新添加的数据能够得到反映。

现在，一旦你在集合上调用`applyFilter()`方法，它首先使用全部数据重置集合，然后在这个集合上应用过滤器。这样，每次你使用这个方法过滤集合时，你都不必担心它是否被应用到之前过滤过的集合上。让我们通过一个测试用例来分析这个功能：

```js
var filteredCollection = new FilterCollection ([
  { name: 'John', company: 'A' }, 
  { name: 'Bill', company: 'B' }, 
  { name: 'Rick', company: 'A' }
]);

// Add another data to check whether add event is working or not
filteredCollection.add({
  name: 'John',
  company: 'C'
});

// Filter with company
filteredCollection.applyFilter({
  company: 'A'
});

// Filter with name
filteredCollection.applyFilter({
  name: 'John'
});

console.log(filteredCollection); 
// Shows two data both with name : 'John'
```

之前，在第二次过滤后，你只收到一组数据，因为集合已被过滤两次，返回的模型名称为`John`和公司`A`。但现在，因为每次过滤之前集合都会被刷新，所以你会得到正确的结果。

之前的代码尚未准备好投入生产，你需要使其更加复杂，以便`_totalData`始终包含最新的数据。无论如何，这种模式在某些情况下很有用，保持一个可过滤的集合扩展或`Filterable`混入可以立即解决问题。

# 理解多模型类型集合

有时候，我们从服务器接收到的数据是混合的，我们需要将完整的数据放入单个集合中。例如，假设服务器正在发送一家公司的完整员工详细信息。现在，有不同类型的员工——开发者、经理、设计师等等，你希望为这些中的每一个都有不同的模型类型。集合应该如何将所有类型的模型放在一起？以下是一个可以让你获得所需功能的例子：

```js
var Employee = Backbone.Model.extend();
var Developer = Employee.extend();
var Manager = Employee.extend();

var Employees = Backbone.Collection.extend({
  url: 'employees.json',
  model: function (attrs, options) {
    // For each data, check the attribute type
    switch (attrs.type) {
      case "Developer":
        return new Developer(attrs, options);
        break;

      case "Manager":
        return new Manager(attrs, options);
        break;
    }
  }
});

var employees = new Employees();
employees.fetch();
console.log(employees);
```

集合请求的模型要么是模型本身，要么是每次向集合添加数据时创建的该模型的实例，或者可以包含一个函数，数据属性会被传递给它，你可以根据数据检查相关模型并传递其一个实例。在这里，在前面的例子中，我们做了同样的事情，根据数据属性的类型返回了不同的模型。

# 摘要

与集合一起工作是 Backbone 的基本要求，与集合相关的问题也与模型相关。例如，几乎每个开发者都会遇到的最常见问题是嵌套集合，而解决这个问题与模型内部数据解析的方式有关。我们在第三章*与模型一起工作*中讨论了关系数据插件，它巧妙地解决了嵌套模型和集合的问题。强烈建议在处理任何此类数据关系时使用此插件。

本章讨论了如何对集合进行排序和过滤。通过示例描述了简单的排序和多个排序过程。我们还看到了一些过滤集合的方法，这些方法在不同情况下可能很有用。

集合可以在内部持有不同类型的模型数据——这个解决方案通过一个例子进行了描述。一般来说，任何数据集都需要一些实用方法来处理。大量 Underscore.js 实用方法使得我们处理 Backbone 集合变得更加容易。

在下一章中，我们将讨论 Backbone 路由器的必要性以及为什么在简单应用中使用多个子路由器是有益的。
