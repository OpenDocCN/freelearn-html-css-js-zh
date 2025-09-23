# 第三章：与模型一起工作

JavaScript 模型是客户端数据管理的重要组成部分。在具有状态的状态 JavaScript 应用程序中，本地或远程数据存储在模型中，并且模型提供了一系列函数来处理这些数据，例如转换、验证、数据持久化等。Backbone 模型与这些模型没有区别，并提供类似的功能，例如设置/获取数据、验证、保存到或从服务器获取、删除属性以及与服务器同步。

在本章中，我们将讨论 Backbone 开发者通常遇到的一些模型基本问题，并提出一些可能的解决方案。此外，我们还将介绍一些有趣的模型插件和扩展，这些插件可以帮助减少代码中的样板代码。以下是需要涵盖的主要点：

+   **模型的基本用法**：学习 Backbone 模型的基础知识，例如重要方法、属性和数据操作。

+   **验证数据**：我们将看到如何使用 Backbone 模型执行基本数据验证。此外，我们还将分析一个重要的插件`Backbone.Validation`，它可以帮助我们减少大量的样板验证代码。

+   **序列化模型**：发送到服务器或从服务器接收的数据格式可能与模型期望的格式不同。在本节中，我们将看到如何通过重写`parse()`和`toJSON()`方法来帮助模型直接与服务器通信。

+   **理解关系型数据模型**：我们将借助 Backbone 关系插件来阅读嵌套模型和集合的分析。

# 模型的基本用法

模型是 Backbone 最重要的组件之一。从存储数据开始，它们提供了很多功能，包括数据逻辑、验证、数据交互等。一个模型可以通过扩展`Backbone.Model`类来定义，如下所示：

```js
var User = Backbone.Model.extend({});
```

模型由一个`attributes`属性组成，该属性存储其内部的数据。你可以使用`get()`方法获取模型数据，并通过使用`set()`方法在`attributes`中设置数据：

```js
var newUser  =  new User({
  name : 'Jayanti De',     
  age : 40
});

var name = newUser.get('name');  // Jayanti De
newUser.set('age', 42);

console.log(newUser.toJSON()); 
// Output => {"name": "Jayanti De", "age": 42}
```

模型的`toJSON()`方法返回一个 JSON 对象，其中包含模型属性的副本。注意，输出现在将`age`设置为新的值。每次通过`set()`方法更改任何属性时，都会在模型上触发一个`change`事件：

```js
newUser.on('change' , function(model, options){
  console.log(model.changed);  // Output => {"age" : 42}
});
```

每个更改的属性都会触发`change`事件：

```js
newUser.on('age:change' , function(model, newAge){
  console.log(newAge);  // Output => 42
});
```

当你只想部分更新视图时，这非常有用，因为在这种情况下会触发`change`和`change:age`事件。你可以只监听特定属性的变化并相应地采取行动。

## 使用默认属性

在某些情况下，你可能希望你的模型在添加新数据之前具有一组默认值。Backbone 提供了一个`defaults`属性，你可以在这里指定初始数据，如下面的代码片段所示：

```js
var User = Backbone.Model.extend({
  defaults: {
    name: 'John Doe',
    age: 20
  }
});

console.log(new User().get('name')); 
// Output => John Doe
```

当为模型的每个实例添加时，任何未指定的属性将自动设置为默认值。

### 避免在 `defaults` 属性中使用对象引用

确保您永远不要在 `defaults` 属性中直接使用任何对象或数组。这是因为 JavaScript 中的对象是通过引用共享的，如果添加到 `defaults` 中，这些对象将在模型的各个实例之间共享。以下是一个示例来解释这种情况：

```js
var User = Backbone.Model.extend({
    defaults : {
        hobbies : []
    }
});

var user1 = new User(),
user2 = new User();

user1.get('hobbies').push('photography');
user2.get('hobbies').push('biking');

console.log(user1.get('hobbies'));
// Output => ["photography", "biking"]
```

您会看到 `hobbies` 数组现在成为模型两个实例之间的共享属性。这不是一个期望的情况，您应该始终避免将对象作为默认属性。可以通过使用函数而不是对象来为 `defaults` 属性解决问题：

```js
defaults: function() {
  return {
    hobbies: []
  }
}
```

```js
console.log(user1.get('hobbies'));
// Output => ["photography"]
```

这个函数会在每次创建模型实例时执行，因此总是会为 `defaults` 发送一个新的对象。

## 与服务器进行数据交互

Backbone 模型通过提供一系列有趣的方法，如 `fetch()`、`save()`、`sync()` 和 `destroy()`，使得与服务器进行数据操作变得非常简单。让我们逐一查看这些方法。我们将使用之前相同的用户模型。

```js
var User = Backbone.Model.extend({
  url: '/users'
});
```

### 创建一个模型

通常，如果您将新值设置到模型中并对其调用 `save()` 方法，您的服务器应该在数据库中创建一个新的模型。从下一次开始，模型将携带这个 `id` 属性，再次调用 `save()` 方法应该只更新模型而不是创建一个新的模型：

```js
var user = new User({
  name : 'Ashim De',
  age : 55
});

user.save({
  success : function(){},
  error : function(){}
});  
```

由于目前还没有 `id` 属性，因此发送一个 POST 请求到 `/users` URL，服务器会发送一个包含新 ID 的响应。

### 更新一个模型

更新一个模型的过程也类似。如果存在 `id` 属性，相同的 `save()` 方法会向服务器发送一个带有新属性的 PUT 请求：

```js
var user = new User({
  id: 23,
  name: 'Shankha De',
  age: 14
});

// Send PUT request to the server
user.save();
```

### 获取一个模型

如果存在 `id` 属性，模型的 `fetch()` 方法会发送一个 GET 请求以检索并填充模型：

```js
var user = new User({
  id: 23
});

// Sends GET request to /users/23
user.fetch();
```

### 删除一个模型

使用 `destroy()` 方法删除一个模型。此方法向服务器发送一个带有模型 ID 的 DELETE 请求：

```js
var user = new User({
  id: 23
});

user.destroy({
  success: function () {}
});
```

# 验证数据

在 Backbone 中，验证由 `model.validate()` 方法处理。默认情况下，`Backbone.Model` 类本身没有 `validate()` 方法。然而，开发者被鼓励添加一个 `validate()` 方法，该方法在每次使用 `validate: true` 传递时被模型调用。所有更改的值都会发送到 `validate()` 方法的一个属性副本。让我们看看一个简单的数据验证示例：

```js
var User = Backbone.Model.extend({
  validation: {    emailRegEx: /^\s*[\w\-\+_]+(\.[\w\-\+_]+)*\@[\w\-\+_]+\.[\w\-\+_]+(\.[\w\-\+_]+)*\s*$/
},

  defaults: {
    name: '',
    email: ''
  },

  validate: function (attr) {
    if (attr.name.length === 0) {
      return 'Name is required';
    }

    if (attr.email.length === 0) {
      return 'Email is required';
    }

    if (!this.validation.emailRegEx.test(attr.email)) {
      return 'Please provide a valid email';
    }
  }
});

// Define the user view
var UserView = Backbone.View.extend({
  initialize: function () {
    this.model.on('invalid', this.handleError, this);
  },

  handleError: function (model, error, options) {
    alert(error);
  }
});

var user = new User();

var userView = new UserView({
  model: user
});

// Set new attributes 
user.set({
  name: '',
  email: 'johndoe#www.com'
}, {
  validate: true
});
```

在这里，我们创建了一个具有两个属性 `name` 和 `email` 的模型，添加了一个 `validate()` 方法来测试这些属性的值，并定义了一个视图来处理可能出现的验证错误。

由于我们是在单个 `set()` 方法中设置两个值，所以 `validate()` 方法只会被调用一次。然而，一旦它发现一个无效属性，它就会返回一个错误。如果我们想在表单上一起显示所有错误怎么办？在这种情况下，我们必须返回一个包含所有错误信息的数组或对象，如下面的代码片段所示：

```js
validate: function (attr) {
  var errors = {};

  if (attr.name.length === 0) {
    errors['name'] = 'Name is required';
  }

  if (attr.email.length === 0) {
    errors['email'] = 'Email is required';
  }

  if (!this.validation.emailRegEx.test(attr.email)) {
    // If already there is an error for email, 
    // then skip other errors for email
    errors['email'] = errors['email'] || 'Please provide a valid email';
  }

  return errors;
}

// Set both empty values
user.set({
  name: '',
  email: 'johndoe#www.com'
}, { validate: true });
```

现在，我们将接收到一个包含所有错误的对象。当你需要单独显示数据，即使它们一个接一个地设置时，这很有用。例如，当我们在 blur 事件上验证字段时，这将非常有用。

## 使用 Backbone.Validation 插件

因此，我们刚刚看到了数据验证的简单实现。然而，当有大量具有多个验证标准的表单字段时，`validate()` 方法会变得很大，包含多个嵌套的 if-else 条件。从头开始创建完整的验证逻辑可能会使其更加复杂和耗时。幸运的是，有一个叫做 `Backbone.Validation` 的出色插件（[`thedersen.com/projects/backbone-validation/`](http://thedersen.com/projects/backbone-validation/))，它通过提供多个内置验证方法和简化与视图的验证绑定，使事情变得容易得多。让我们使用这个插件重新实现之前的验证。

### 配置验证规则

有许多内置验证器，例如 `required`、`maxLength`、`minLength`、`max`、`min`、`length` 和 `pattern`。它们的使用方法如下所示：

```js
var User = Backbone.Model.extend({
  validation: {
    name: {
      required: true
    },

    email: {
      required: true,
      pattern: 'email'
    }
  },

  defaults: {
    name: '',
    email: ''
  }
});
```

存在一些现有的验证模式，如电子邮件、数字和 URL。或者，你可以使用正则表达式作为模式。同样，你可能需要为属性定义完整的验证功能，而不仅仅是正则表达式。在这种情况下，你可以向属性添加自定义方法验证器。查看以下示例：

```js
var User = Backbone.Model.extend({
  validation: {
    // Do not return anything if validation is passed
    name: function (value, attr, computedState) {
      if (!value) {
        return 'Name is required';
      }
    },

    // the method will be called on model's scope
    email: 'validateEmail'
  },

  validateEmail: function (value, attr, computedState) {
    if (!value) {
      return 'Email is required';
    }
  }
});
```

你可以直接将自定义方法添加到属性中作为一个函数，或者你可以将方法名作为字符串添加，就像我们在这里做的那样。每个属性可以为每个验证规则有一个错误消息，或者它可以为所有验证规则有一个单一的错误消息。例如，在以下代码中，我们为 `email` 的 `required` 和 `format` 验证提供了单独的消息：

```js
{
  name: {
    required: true,
    msg: 'Name is required'
  },

  email: [{
    required: true,
    msg: 'Email is required'
  }, {
    pattern: 'email',
    msg: 'Please provide a valid email'
  }]
}
```

### 使用 preValidate() 方法预先验证模型

此插件还提供了另一个重要的功能，即在不接触模型本身的情况下预先验证模型的属性，如下所示：

```js
var errorMessage = model.preValidate('attributeName', 'Value');
```

因此，该属性将与其分配的验证器集进行验证，如果验证失败，返回值将是一个错误信息。

如果你的应用程序需要多个表单验证，`Backbone.Validation` 插件非常有效。它从你的代码库中移除了许多样板代码，并提供了一个简单而健壮的验证机制。

# 序列化模型

到目前为止，我们在前几章的示例中使用的模型数据都是简单的具有属性的数据对象。然而，可能存在服务器发送不同数据格式的情况，你需要从中提取关键部分并将其应用于相关模型。例如，考虑以下数据：

```js
{
  "name": "John Doe",
  "email": "johndoe@example.com"
}
```

代替发送前面的数据，服务器返回以下数据：

```js
{
  "user": {
    "name": "John Doe",
    "email": "johndoe@example.com"
  }
}
```

这份数据不能直接应用于具有 `name` 和 `email` 属性的模型。如果我们现在对模型调用 `fetch()` 方法，它将只是向模型添加另一个名为 `user` 的属性。可以帮助我们克服这个问题的方法称为 `parse()`。默认情况下，这个方法只是传递服务器响应，模型应用从 `parse()` 方法接收到的任何内容。这是在 `Backbone.js` 中如何定义的：

```js
parse: function (resp, options) {
  return resp;
}
```

然而，我们可以覆盖 `parse()` 方法来修改原始服务器响应，并只发送属性 `hash`。对于这种情况，`parse()` 方法应返回一个具有 `name` 和 `email` 属性的对象，如下面的代码片段所示：

```js
 var User = Backbone.Model.extend({
  url: 'server.json',
  defaults: {
    name: '',
    email: ''
  },

  // Returns the attribute hash 
  parse: function (response) {
    return response.user;
  }
});

var user = new User();
user.fetch({
  success: function () {
    console.log(user.get('name')); // John Doe
  }
});
```

在这里，`server.json` 文件包含新格式化的数据。在 `parse()` 方法中，我们正在解析响应并返回 Backbone 模型可以接受的数据。

### 小贴士

记住，`fetch()` 方法不会清除你的模型，但只会扩展属性。所以，在我们的例子中，如果服务器只发送一个电子邮件作为响应，之前的电子邮件将被更新，但名称仍然保持不变。

与从服务器获取数据类似，向服务器发送数据也可能遇到相同的问题，即服务器可能期望它发送给模型的确切格式。现在，如果我们对模型调用 `save()` 方法，它将以以下格式发送数据：

```js
{ 
      name : 'Swarnendu De',
      email: 'swarnendu@email.com'
}
```

因此，如果服务器期望的数据格式与现在发送的格式相同，我们需要覆盖 `toJSON()` 方法，这相当简单。在下面的代码中，我们创建了一个具有 `user` 属性的新对象，并从 `toJSON()` 方法返回该对象：

```js
// Add this method to model
toJSON: function () {
  return {
    user: _.clone(this.attributes)
  }
}

// Let's set new data and send that to the server
user.set({
  name: 'Swarnendu',
  email: 'swarnendu@email.com'
});

user.save();
```

请求将以以下数据发送到服务器，这正是我们所寻找的：

```js
{
  "user": {
    "name": "Swarnendu",
    "email": "swarnendu@email.com"
  }
}
```

然而，这个过程有一个缺点。在大多数情况下，我们使用 `toJSON()` 方法直接获取模型属性哈希。由于我们在这里覆盖了这个方法，返回的数据将与预期数据不同。因此，你需要决定你是否将采用这种方法来序列化模型或单独实现服务器端交互。如果你选择这个过程，记得在使用 `toJSON()` 方法时相应地应用模型数据。或者，你也可以克隆 `model.attributes` 属性以获取 `hash` 属性：

```js
var jsonData = _.clone(this.attributes);
```

### 小贴士

最好不直接使用 `model.attributes` 属性。直接操作 `hash` 属性可能会引起一些意外的后果，因为对象将通过引用传递。

# 理解关系型数据模型

我们到目前为止所讨论的所有示例都使用了简单的模型来表示数据。然而，在任何非平凡的应用中，数据结构都要复杂得多，实体之间的关系是多重关系的。对于任何中等或大型应用，都存在大量的一对一、一对多和多对一关系。保持这些关系与服务器同步通常变得是一项繁琐的工作，尤其是在使用多个请求保存或检索数据时。

在研究这本书的过程中，我发现大多数 Backbone 开发者在学习阶段某个时候都遇到过嵌套模型和集合的问题。幸运的是，有一个名为 Backbone-relational 的优秀插件（[`backbonerelational.org/`](http://backbonerelational.org/)），由 Paul Uithol 开发，通过使用单个 `save()` 或 `fetch()` 方法同步模型及其所有相关模型，最小化了 Backbone 模型手动管理。它提供了一些很棒的特性，包括以下内容：

+   双向关系，通过事件通知相关模型的变化

+   控制关系序列化的方式

+   将模型属性中的嵌套对象自动转换为模型实例

+   轻松检索一组相关模型

+   确定 `HasMany` 集合的类型

我们将通过一个简单的公司-员工关系示例来解释 Backbone-relational 插件的原理：

```js
var Company = Backbone.RelationalModel.extend({
  defaults: {
    name: ''
  },
  relations: [{
    // 'type' can be HasOne or HasMany 
    // or a direct reference to a relation
    type: Backbone.HasMany,

    // 'key' refer to an attribute name of the related model
    key: 'employees',
    relatedModel: 'Employee',

    // a collection of the related models
    collectionType: 'Employees',

    // defines the reverse relation with this model
    reverseRelation: {
      key: 'worksIn',
      includeInJSON: 'id'
      // 'relatedModel' is automatically set to 'Company'; 
      // the 'relationType' to 'HasOne'.
    }
  }]
});

var Employee = Backbone.RelationalModel.extend({
  defaults: {
    name: '',
    worksIn: null
  }
});

var Employees = Backbone.Collection.extend({
  model: Employee
});
```

在这里，我们创建了一个公司-员工的一对多关系。在公司模型配置中，你需要定义用于创建 `Backbone.Relation` 实例的关系类型。`type` 关系属性可以是 `Backbone.HasMany`、`Backbone.HasOne` 或对特定关系实例的直接引用。你还需要指定公司模型中包含所有员工模型的属性。一旦完成基本配置，我们将定义 `Employee` 模型和 `Employees` 集合。现在让我们用一些虚拟数据来测试这个关系：

```js
var innofied = new Company({
  name: 'Innofied'
});

var john = new Employee({
  name: 'John Doe',
  worksIn: innofied
});

var swarnendu = new Employee({
  name: 'Swarnendu De',
  worksIn: innofied
});

// 'employees' in 'innofied' now contains 
// 'John Doe and Swarnendu De'
alert(innofied.get('employees').pluck('name'));
```

我们现在已经创建了一个完全管理的关联。当你从 `innofied.employees` 中添加或删除模型或更新 `employee.worksIn` 时，关系的另一侧会自动更新。

之前提到的代码只是 Backbone-relational 模型的基本示例。一旦你阅读了他们的完整文档，你会发现该插件提供了许多可以极大地增强应用开发过程的特性。

# 摘要

本章讨论了围绕 Backbone 模型的一些基本问题，以及我们如何在项目中解决这些问题。我们学习了基本的数据验证，以及如何从我们的 validate 方法中收集所有错误消息。此外，我们还看到了如何使用 Backbone 验证插件通过提供许多内置特性来减少数据验证时的努力。

如果服务器发送的数据格式与模型期望的不同，我们现在知道如何覆盖`parse()`方法来解决这个问题。同样，我们也覆盖了`toJSON()`方法来改变将传递给服务器的数据格式。

对于大多数非平凡应用，嵌套模型关系是一个基本要求，Backbone-relational 插件可以为此提供一个现成的解决方案。该插件被 Backbone 社区广泛接受，并且许多项目目前正在成功使用它。

在使用模型时，有一些重要的话题需要讨论，例如集合、事件和同步。我们将在接下来的章节中分别详细讨论这些点。事件和同步功能在第六章*处理事件、同步和存储*中进行了详细讨论。在下一章中，我们将讨论 Backbone 集合的不同功能，基本和多种排序，过滤机制，以及具有多种模型类型的集合。
