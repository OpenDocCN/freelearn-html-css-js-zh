# 第三章：构建 RESTful API

在本章中，我们将涵盖以下配方：

+   使用 ExpressJS 的路由方法进行 CRUD 操作

+   使用 Mongoose 进行 CRUD 操作

+   使用 Mongoose 查询构建器

+   定义文档实例方法

+   定义静态模型方法

+   为 Mongoose 编写中间件函数

+   为 Mongoose 的模式编写自定义验证器

+   使用 ExpressJS 和 Mongoose 构建 RESTful API 来管理用户

# 技术要求

您需要一个 IDE、Visual Studio Code、Node.js 和 MongoDB。您还需要安装 Git，以便使用本书的 Git 存储库。

本章的代码文件可以在 GitHub 上找到：

[`github.com/PacktPublishing/MERN-Quick-Start-Guide/tree/master/Chapter03`](https://github.com/PacktPublishing/MERN-Quick-Start-Guide/tree/master/Chapter03)

查看以下视频以查看代码的运行情况：

[`goo.gl/73dE6u`](https://goo.gl/73dE6u)

# 介绍

**表示状态转移**（**REST**）是 Web 建立的一种架构风格。更具体地说，HTTP 1.1 协议标准是使用 REST 原则构建的。REST 提供了资源的表示。**URL**（**统一资源定位符**）用于定义资源的位置并告诉浏览器它的位置。

RESTful API 是遵循这种架构风格的 Web 服务 API。

最常用的 HTTP 动词或方法是：`POST, GET, PUT,`和`DELETE`。这些方法是持久存储的基础，并被称为**CRUD**操作（**创建、读取、更新和删除**）。

在本章中，我们将重点放在使用 ExpressJS 和 Mongoose 构建 RESTful API 上的配方上。

# 使用 ExpressJS 的路由方法进行 CRUD 操作

ExpressJS 的路由器具有等效的方法来处理 HTTP 方法。换句话说，可以通过此代码处理`POST`、`GET`、`PUT`和`DELETE`这些 HTTP 方法：

```js
      /* Add a new user */ 
      app.post('/users', (request, response, next) => { }) 
      /* Get user */ 
      app.get('/users/:id', (request, response, next) => { }) 
      /* Update a user */ 
      app.put('/users/:id', (request, response, next) => { }) 
      /* Delete a user */ 
      app.delete('/users/:id', (request, response, next) => { })  
```

将每个 URL 视为名词是很好的，因此动词可以对其进行操作。事实上，HTTP 方法也被称为 HTTP 动词。如果我们将它们视为动词，当对我们的 RESTful API 进行请求时，它们可以被理解为：

+   发布用户

+   获取用户

+   更新用户

+   删除用户。

在**MVC**（**模型-视图-控制器**）架构模式中，控制器负责将输入转换为模型或视图可以理解的内容。换句话说，它们将输入转换为操作或命令，并将其发送到模型或视图以相应地进行更新。

ExpressJS 的路由方法通常充当控制器。它们只是从浏览器等客户端获取输入，然后将输入转换为操作。然后将这些操作发送到模型，这是应用程序的业务逻辑，例如 mongoose 模型，或者发送到视图（ReactJS 客户端应用程序）进行更新。

# 准备工作

请记住，我们可以使用 HTTP 方法对资源进行操作，我们将看到如何基于这些概念构建 RESTful API Web 服务。在开始之前，请使用以下代码创建一个新的`package.json`文件：

```js
      { 
        "dependencies": { 
          "express": "4.16.3", 
          "node-fetch": "2.1.1", 
          "uuid": "3.2.1" 
        } 
      } 
```

然后，通过打开终端并运行以下代码来安装依赖项：

```js
 npm install
```

# 如何做...

使用内存数据库或包含用户列表的对象数组构建 RESTful API。我们将允许使用 HTTP 方法进行 CRUD 操作，包括添加新用户、获取用户或用户列表、更新用户数据和删除用户：

1.  创建一个名为`restfulapi.js`的新文件

1.  导入我们需要的包并创建一个 ExpressJS 应用程序：

```js
     const express = require('express') 
      const uuid = require('uuid') 
      const app = express() 
```

1.  定义内存数据库：

```js
      let data = [ 
          { id: uuid(), name: 'Bob' }, 
          { id: uuid(), name: 'Alice' }, 
      ] 
```

1.  创建一个包含用于进行 CRUD 操作的函数的模型：

```js
      const usr = { 
          create(name) { 
              const user = { id: uuid(), name } 
              data.push(user) 
              return user 
          }, 
          read(id) { 
              if (id === 'all') return data 
              return data.find(user => user.id === id) 
          }, 
          update(id, name) { 
              const user = data.find(usr => usr.id === id) 
              if (!user) return { status: 'User not found' } 
              user.name = name 
              return user 
          }, 
          delete(id) { 
              data = data.filter(user => user.id !== id) 
              return { status: 'deleted', id } 
          } 
      } 
```

1.  为`post`方法添加一个请求处理程序，该处理程序将用作`Create`操作。将新用户添加到`data`数组中：

```js
      app.post('/users/:name', (req, res) => { 
          res.status(201).json(usr.create(req.params.name)) 
      }) 
```

1.  为`get`方法添加一个请求处理程序，该处理程序将用作`Read`或`Retrieve`操作。如果给定`id`，则在`data`数组中查找用户。但是，如果给定的`id`是`"all"`，它将返回整个用户列表：

```js
      app.get('/users/:id', (req, res) => { 
          res.status(200).json(usr.read(req.params.id)) 
      }) 
```

1.  为`put`方法添加一个请求处理程序，该处理程序将用作`Update`操作。需要提供一个`id`来更新`data`数组中的特定用户：

```js
      app.put('/users/:id=:name', (req, res) => { 
          res.status(200).json(usr.update( 
              req.params.id, 
              req.params.name, 
          )) 
      }) 
```

1.  为`delete`方法添加一个请求处理程序，该处理程序将用作`Delete`操作。它将在`data`数组中查找用户并将其删除：

```js
      app.delete('/users/:id', (req, res) => { 
          res.status(200).json(usr.delete(req.params.id)) 
      }) 
```

1.  启动应用程序监听端口`1337`以进行新连接：

```js
      app.listen( 
          1337, 
          () => console.log('Web Server running on port 1337'), 
      ) 
```

1.  保存文件。

1.  打开终端并运行此代码：

```js
 node restfulapi.js
```

# 让我们来测试一下...

为了简单起见，创建一个脚本，将在我们的 RESTful API 服务器上请求并执行 CRUD 操作：

1.  创建一个名为`test-restfulapi.js`的新文件。

1.  添加以下代码：

```js
      const fetch = require('node-fetch') 
      const r = async (url, method) => ( 
          await fetch(`http://localhost:1337${url}`, { method }) 
              .then(r => r.json()) 
      ) 
      const log = (...obj) => ( 
          obj.forEach(o => console.dir(o, { colors: true })) 
      ) 
      async function test() { 
          const users = await r('/users/all', 'get') 
          const { id } = users[0] 
          const getById = await r(`/users/${id}`, 'get') 
          const updateById = await r(`/users/${id}=John`, 'put') 
          const deleteById = await r(`/users/${id}`, 'delete') 
          const addUsr = await r(`/users/Smith`, 'post') 
          const getAll = await r('/users/all', 'get') 
          log('[GET] users:', users) 
          log(`[GET] a user with id="${id}":`, getById) 
          log(`[PUT] a user with id="${id}":`, updateById) 
          log(`[POST] a new user:`, addUsr) 
          log(`[DELETE] a user with id="${id}":`, deleteById) 
          log(`[GET] users:`, getAll) 
      } 
      test() 
```

1.  保存文件。

1.  打开一个新的终端并运行此代码：

```js
    node test-restfulapi.js

```

# 它是如何工作的...

我们的 RESTful API 应用程序将在本地端口`1337`上运行。运行测试代码时，它将连接到它并使用不同的 HTTP 方法进行多个请求，以创建用户，检索用户，更新用户和删除用户。所有操作将在终端中记录。

如果您喜欢自己测试，可以替换`test`函数内的所有代码，并使用`r`函数进行自定义请求。例如，要创建一个名为`Smith`的新用户：

```js
r(`/users/Smith`, 'post') 
```

# 使用 Mongoose 进行 CRUD 操作

开发人员选择使用 Mongoose 而不是 Node.js 的官方 MongoDB 驱动程序的许多原因之一是，它允许您通过使用模式轻松创建数据结构，并且还因为内置验证。MongoDB 是一种面向文档的数据库，这意味着文档的结构是多样化的。

在 MVC 架构模式中，Mongoose 通常用于创建塑造或定义数据结构的模型。

这是典型的 Mongoose 模式的定义方式，然后编译成模型：

```js
      const PersonSchema = new Schema({ 
          firstName: String, 
          lastName: String, 
      }) 
      const Person = connection.model('Person', PersonSchema) 
```

模型名称应该是单数，因为 Mongoose 在将集合保存到数据库时会将它们变成复数并将它们转换为小写。例如，如果模型名为“User”，它将以“users”的名称保存在 MongoDB 中。Mongoose 包含一个内部字典来将常见名称变为复数形式。这意味着如果您的模型名称是一个常见名称，比如“Person”，它将在 MongoDB 中保存为一个名为“people”的集合。

Mongoose 允许使用以下类型来定义模式的路径或文档结构：

+   字符串

+   数字

+   布尔值

+   数组

+   日期

+   缓冲区

+   混合

+   Objectid

+   Decimal128

可以通过直接使用`String`、`Number`、`Boolean`、`Buffer`和`Date`的全局构造函数来声明模式类型：

```js
      const { Schema} = require('mongoose') 
      const PersonSchema = new Schema({ 
          name: String, 
          age: Number, 
          isSingle: Boolean, 
          birthday: Date, 
          description: Buffer, 
      }) 
```

这些模式类型也可以在导出的`mongoose`对象中的名为`SchemaTypes`的对象下使用：

```js
      const { Schema, SchemaTypes } = require('mongoose') 
      const PersonSchema = new Schema({ 
          name: SchemaTypes.String, 
          age: SchemaTypes.Number, 
          isSingle: SchemaTypes.Boolean, 
          birthday: SchemaTypes.Date, 
          description: SchemaTypes.Buffer, 
      }) 
```

可以使用对象作为属性来声明模式类型，从而更好地控制特定的模式类型。例如，看下面的代码：

```js
      const { Schema } = require('mongoose') 
      const PersonSchema = new Schema({ 
          name: { type: String, required: true, default: 'Unknown' }, 
          age: { type: Number, min: 18, max: 80, required: true }, 
          isSingle: { type: Boolean }, 
          birthday: { type: Date, required: true }, 
          description: { type: Buffer }, 
      }) 
```

模式类型也可以是数组。例如，如果我们想要一个字段来定义用户喜欢的事物的字符串数组，可以使用以下代码：

```js
      const PersonSchema = new Schema({ 
          name: String, 
          age: Number, 
          likes: [String], 
      }) 
```

要了解有关模式类型的更多信息，请访问官方 Mongoose 文档网站：[`mongoosejs.com/docs/schematypes.html`](http://mongoosejs.com/docs/schematypes.html)。

# 准备工作

在这个示例中，您将看到如何定义模式并对数据库集合执行 CRUD 操作。首先确保已安装 MongoDB 并且正在运行。作为替代方案，如果您愿意，在云中也可以使用 MongoDB **DBaaS**（**数据库即服务**）实例。在开始之前，创建一个新的`package.json`文件，其中包含以下代码：

```js
      { 
        "dependencies": { 
          "mongoose": "5.0.11" 
       } 
     } 
```

然后，通过打开终端并运行此代码来安装依赖项：

```js
 npm install
```

# 如何做...

定义一个用户模式，其中包含用户的名字、姓氏和定义用户喜欢的事物的字符串数组：

1.  创建一个名为`mongoose-models.js`的新文件

1.  包含 Mongoose NPM 模块。然后，创建与 MongoDB 的连接：

```js
      const mongoose = require('mongoose') 
      const { connection, Schema } = mongoose 
      mongoose.connect( 
          'mongodb://localhost:27017/test' 
      ).catch(console.error) 
```

1.  定义一个模式：

```js
      const UserSchema = new Schema({ 
          firstName: String, 
          lastName: String, 
          likes: [String], 
      }) 
```

1.  将模式编译成模型：

```js
      const User = mongoose.model('User', UserSchema) 
```

1.  定义一个用于添加新用户的函数：

```js
      const addUser = (firstName, lastName) => new User({ 
          firstName, 
          lastName, 
      }).save() 
```

1.  定义一个用于通过其`id`从用户集合中检索用户的函数：

```js
      const getUser = (id) => User.findById(id) 
```

1.  定义一个函数，将通过其`id`从用户集合中删除用户：

```js
      const removeUser = (id) => User.remove({ id }) 
```

1.  定义一个事件监听器，一旦连接到数据库，就会执行 CRUD 操作。首先，添加一个新用户并保存它。然后，使用其`id`检索相同的用户。接下来，修改用户的属性并保存。最后，通过其`id`从集合中删除用户：

```js
      connection.once('connected', async () => { 
          try { 
              // Create 
              const newUser = await addUser('John', 'Smith') 
              // Read 
              const user = await getUser(newUser.id) 
              // Update 
              user.firstName = 'Jonny' 
              user.lastName = 'Smithy' 
              user.likes = [ 
                  'cooking', 
                  'watching movies', 
                  'ice cream', 
              ] 
              await user.save() 
              console.log(JSON.stringify(user, null, 4)) 
              // Delete 
              await removeUser(user.id) 
          } catch (error) { 
              console.dir(error.message, { colors: true }) 
          } finally { 
              await connection.close() 
          } 
      }) 
```

1.  保存文件。

1.  打开终端并运行此代码：

```js
    node mongoose-models.js
```

在终端中执行前一个命令，如果成功，将显示类似于以下内容，例如，类似于这样的代码：

```js
      { 
          "likes": [ 
        "cooking", 
              "watching movies", 
              "ice cream" 
                ], 
          "_id": "[some id]", 
          "firstName": "Jonny", 
          "lastName": "Smithy", 
          "__v": 1 
      } 
```

# 另请参阅

+   第一章，*MERN Stack 简介*，*安装 NPM 包*部分

+   第一章，*MERN Stack 简介*，*安装 MongoDB*部分

# 使用 Mongoose 查询构建器

每个 Mongoose 模型都有静态辅助方法来执行多种操作，比如检索文档。当这些辅助方法传递一个回调时，操作会立即执行：

```js
      const user = await User.findOne({ 
          firstName: 'Jonh', 
          age: { $lte: 30 }, 
      }, (error, document) => { 
          if (error) return console.log(error) 
          console.log(document) 
      }) 
```

否则，如果没有定义的回调函数，将返回一个*查询构建器接口*，稍后可以执行：

```js
      const user = User.findOne({ 
          firstName: 'Jonh', 
          age: { $lte: 30 }, 
      }) 
      user.exec((error, document) => { 
          if (error) return console.log(error) 
          console.log(document) 
      }) 
```

查询还有一个`.then`函数，可以作为`Promise`使用。当调用`.then`时，它首先使用`.exec`内部执行查询，然后返回一个`Promise`。这使我们也可以使用`async/await`。例如，在`async`函数内部：

```js
      try { 
          const user = await User.findOne({ 
              firstName: 'Jonh', 
              age: { $lte: 30 }, 
          }) 
          console.log(user) 
      } catch (error) { 
          console.log(error) 
      }  
```

我们可以进行查询的两种方式。一种是提供一个作为条件使用的 JSON 对象，另一种方式允许您使用链接语法创建查询。链接语法将更适合熟悉 SQL 数据库的开发人员。例如：

```js
      try { 
          const user = await User.findOne() 
        .where('firstName', 'John') 
              .where('age').lte(30) 
          console.log(user) 
      }       catch (error) { 
          console.log(error) 
      }  
```

# 准备工作

在这个示例中，您将使用链接语法和`async/await`函数构建查询。首先确保已安装 MongoDB 并且正在运行。作为替代方案，如果您愿意，云中的 MongoDB DBaaS 实例也可以。在开始之前，创建一个新的`package.json`文件，其中包含以下代码：

```js
      { 
        "dependencies": { 
          "mongoose": "5.0.11" 
        } 
      } 
```

然后，通过打开终端并运行来安装依赖项：

```js
 npm install  
```

# 如何做...

1.  创建一个名为`chaining-queries.js`的新文件

1.  包含 Mongoose NPM 模块。然后，创建一个新连接：

```js
      const mongoose = require('mongoose') 
      const { connection, Schema } = mongoose 
      mongoose.connect( 
          'mongodb://localhost:27017/test' 
      ).catch(console.error) 
```

1.  定义一个模式：

```js
      const UserSchema = new Schema({ 
          firstName: String, 
          lastName: String, 
          age: Number, 
      }) 
```

1.  编译模式为模型：

```js
      const User = mongoose.model('User', UserSchema) 
```

1.  连接到数据库后，向用户集合添加一个新文档。然后，使用链接语法查询最近创建的用户。此外，使用`select`方法限制从文档中检索哪些字段：

```js
      connection.once('connected', async () => { 
          try { 
              const user = await new User({ 
                  firstName: 'John', 
                  lastName: 'Snow', 
                  age: 30, 
              }).save() 
              const findUser = await User.findOne() 
                  .where('firstName').equals('John') 
                  .where('age').lte(30) 
                  .select('lastName age') 
              console.log(JSON.stringify(findUser, null, 4)) 
              await user.remove() 
          } catch (error) { 
              console.dir(error.message, { colors: true }) 
          } finally { 
              await connection.close() 
          } 
      }) 
```

1.  保存文件

1.  打开终端并运行：

```js
    node chaining-queries.js

```

# 另请参阅

+   第一章，*MERN Stack 简介*，*安装 NPM 包*部分

+   第一章，*MERN Stack 简介*，*安装 MongoDB*部分

# 定义文档实例方法

文档有自己的内置实例方法，如`save`和`remove`。但是，我们也可以编写自己的实例方法。

文档是模型的实例。它们可以被显式创建：

```js
      const instance = new Model() 
```

或者它们可以是查询的结果：

```js
      Model.findOne([conditions]).then((instance) => {}) 
```

文档实例方法在模式中定义。所有模式都有一个名为`method`的方法，允许您定义自定义实例方法。

# 准备工作

在这个示例中，您将为修改和读取文档属性定义模式和自定义文档实例方法。首先确保已安装 MongoDB 并且正在运行。作为替代方案，如果您愿意，云中的 MongoDB DBaaS 实例也可以。在开始之前，创建一个新的`package.json`文件，其中包含以下代码：

```js
{ 
  "dependencies": { 
    "mongoose": "5.0.11" 
  } 
} 
```

然后，通过打开终端并运行此代码来安装依赖项：

```js
    npm install
```

# 如何做...

1.  创建一个名为`document-methods.js`的新文件

1.  包含 Mongoose NPM 模块。然后，创建一个到 MongoDB 的新连接：

```js
      const mongooconst mongoose = require('mongoose') 
      const { connection, Schema } = mongoose 
      mongoose.connect( 
          'mongodb://localhost:27017/test' 
      ).catch(console.error) 
```

1.  定义一个模式：

```js
      const UserSchema = new Schema({ 
          firstName: String, 
          lastName: String, 
          likes: [String], 
      }) 
```

1.  定义一个文档实例方法，用于从包含其全名的字符串中设置用户的名和姓：

```js
      UserSchema.method('setFullName', function setFullName(v) { 
          const fullName = String(v).split(' ') 
          this.lastName = fullName[0] || '' 
          this.firstName = fullName[1] || '' 
      }) 
```

1.  定义一个用于获取用户全名的文档实例方法，将`firstName`和`lastName`属性连接起来：

```js
      UserSchema.method('getFullName', function getFullName() { 
          return `${this.lastName} ${this.firstName}` 
      }) 
```

1.  定义一个名为`loves`的文档实例方法，该方法将期望一个参数，该参数将添加到字符串数组`likes`中：

```js
      UserSchema.method('loves', function loves(stuff) { 
          this.likes.push(stuff) 
      }) 
```

1.  定义一个名为`dislikes`的文档实例方法，该方法将从`likes`数组中移除用户之前喜欢的一件事：

```js
      UserSchema.method('dislikes', function dislikes(stuff) { 
          this.likes = this.likes.filter(str => str !== stuff) 
      }) 
```

1.  将模式编译成模型：

```js
      const User = mongoose.model('User', UserSchema) 
```

1.  一旦 Mongoose 连接到数据库，创建一个新用户并使用`setFullName`方法填充`firstName`和`lastName`字段，然后使用`loves`方法填充`likes`数组。接下来，使用链式语法在集合中查询用户，并使用`dislikes`方法从`likes`数组中移除`"snakes"`：

```js
      connection.once('connected', async () => { 
          try { 
              // Create 
              const user = new User() 
              user.setFullName('Huang Jingxuan') 
              user.loves('kitties') 
              user.loves('strawberries') 
              user.loves('snakes') 
              await user.save() 
              // Update 
              const person = await User.findOne() 
                  .where('firstName', 'Jingxuan') 
                  .where('likes').in(['snakes', 'kitties']) 
              person.dislikes('snakes') 
              await person.save() 
              // Display 
              console.log(person.getFullName()) 
              console.log(JSON.stringify(person, null, 4)) 
              // Remove 
              await user.remove() 
          } catch (error) { 
              console.dir(error.message, { colors: true }) 
          } finally { 
              await connection.close() 
          } 
      }) 
```

1.  保存文件。

1.  打开终端并运行此代码：

```js
       node document-methods.js
```

# 还有更多...

文档实例方法也可以使用`methods`模式属性进行定义。例如：

```js
UserSchema.methods.setFullName = function setFullName(v) { 
    const fullName = String(v).split(' ') 
    this.lastName = fullName[0] || '' 
    this.firstName = fullName[1] || '' 
} 
```

# 另请参阅

+   第一章，*MERN Stack 简介*，*安装 NPM 包*部分

+   第一章，*MERN Stack 简介*，*安装 MongoDB*部分

# 定义静态模型方法

模型具有内置的静态方法，如`find`、`findOne`和`findOneAndRemove`。Mongoose 还允许我们定义自定义静态模型方法。静态模型方法与文档实例方法的定义方式相同，都是在模式中定义的。

模式有一个名为`statics`的属性，它是一个对象。`statics`对象内定义的所有方法都会传递给模型。静态模型方法也可以通过调用`static`模式方法进行定义。

# 准备工作

在本示例中，您将定义一个模式和自定义静态模型方法，以扩展模型的功能。首先确保已安装 MongoDB 并且正在运行。或者，如果您愿意，云中的 MongoDB DBaaS 实例也可以。在开始之前，请使用以下代码创建一个新的`package.json`文件：

```js
{ 
  "dependencies": { 
    "mongoose": "5.0.11" 
  } 
} 
```

然后，通过打开终端并运行以下命令来安装依赖项：

```js
npm install
```

# 操作步骤

定义一个名为`getByFullName`的静态模型方法，允许您使用其全名搜索特定用户：

1.  创建一个名为`static-methods.js`的新文件

1.  包括 Mongoose NPM 模块并创建到 MongoDB 的新连接：

```js
      const mongoose = require('mongoose') 
      const { connection, Schema } = mongoose 
      mongoose.connect( 
          'mongodb://localhost:27017/test' 
      ).catch(console.error) 
```

1.  定义一个模式：

```js
      const UsrSchm = new Schema({ 
          firstName: String, 
          lastName: String, 
          likes: [String], 
      }) 
```

1.  定义`getByFullName`静态模型方法：

```js
      UsrSchm.static('getByFullName', function getByFullName(v) { 
          const fullName = String(v).split(' ') 
          const lastName = fullName[0] || '' 
          const firstName = fullName[1] || '' 
          return this.findOne() 
              .where('firstName').equals(firstName) 
              .where('lastName').equals(lastName) 
      }) 
```

1.  将模式编译成模型：

```js
      const User = mongoose.model('User', UsrSchm) 
```

1.  一旦连接，创建一个新用户并保存。然后，使用`getByFullName`静态模型方法通过其全名在用户集合中查找用户：

```js
      connection.once('connected', async () => { 
          try { 
              // Create 
              const user = new User({ 
                  firstName: 'Jingxuan', 
                  lastName: 'Huang', 
                  likes: ['kitties', 'strawberries'], 
              }) 
              await user.save() 
              // Read 
              const person = await User.getByFullName( 
                  'Huang Jingxuan' 
              ) 
              console.log(JSON.stringify(person, null, 4)) 
              await person.remove() 
              await connection.close() 
          } catch (error) { 
              console.log(error.message) 
          } 
      }) 
```

1.  保存文件

1.  打开终端并运行此代码：

```js
    node static-methods.js

```

# 还有更多...

静态模型方法也可以使用`statics`模式属性进行定义。例如：

```js
UsrSchm.statics.getByFullName = function getByFullName(v) { 
    const fullName = String(v).split(' ') 
    const lastName = fullName[0] || '' 
    const firstName = fullName[1] || '' 
    return this.findOne() 
        .where('firstName').equals(firstName) 
        .where('lastName').equals(lastName) 
} 
```

# 另请参阅

+   第一章，*MERN Stack 简介*，*安装 NPM 包*部分

+   第一章，*MERN Stack 简介*，*安装 MongoDB*部分

# 为 Mongoose 编写中间件函数

Mongoose 中的中间件函数也称为`hooks`。有两种类型的钩子`pre hooks`和`post hooks`。

`pre hooks`和`post hooks`之间的区别非常简单。`pre hooks`在方法调用之前调用，而`post hooks`在方法调用之后调用。例如：

```js
      const UserSchema = new Schema({ 
          firstName: String, 
          lastName: String, 
          fullName: String, 
      }) 
      UserSchema.pre('save', async function preSave() { 
          this.fullName = `${this.lastName} ${this.firstName}` 
      }) 
      UserSchema.post('save', async function postSave(doc) { 
          console.log(`New user created: ${doc.fullName}`) 
      }) 
      const User = mongoose.model('User', UserSchema) 
```

稍后，一旦与数据库建立连接，在`async`函数内：

```js
      const user = new User({ 
          firstName: 'John', 
          lastName: 'Smith', 
      }) 
      await user.save() 
```

一旦调用`save`方法，将首先执行`pre hook`。文档保存后，将执行`post hook`。在上一个示例中，终端输出将显示以下文本：

```js
    New user created: Smith John
```

Mongoose 中有四种不同类型的中间件函数：文档中间件、模型中间件、聚合中间件和查询中间件。它们都在模式级别上定义。区别在于，当钩子被执行时，`this`的上下文是文档、模型、聚合对象或查询对象。

所有类型的中间件都支持预先和后续钩子

# 准备工作

在本教程中，我们将看到 Mongoose 中这三种类型的中间件函数是如何工作的：

+   文档中间件

+   模型中间件

+   查询中间件

首先确保您已安装 MongoDB 并且正在运行。或者，如果您愿意，云中的 MongoDB DBaaS 实例也可以。在开始之前，创建一个新的`package.json`文件，其中包含以下代码：

```js
      { 
        "dependencies": { 
          "mongoose": "5.0.11" 
        } 
      } 
```

然后，通过打开终端并运行来安装依赖项：

```js
 npm install
```

# 如何做...

在文档中间件函数中，`this`的上下文指的是文档。文档具有以下内置方法，您可以为它们定义`hooks`：

+   `init`：这是在从 MongoDB 返回文档后立即内部调用的。Mongoose 使用 setter 标记文档是否已修改或文档的哪些字段已修改。`init`初始化文档而不使用 setter。

+   `validate`：执行文档的内置和自定义验证规则。

+   `save`：将文档保存到数据库中。

+   `remove`：从数据库中删除文档。

# 文档中间件函数

为文档内置方法创建预先和后续钩子：

1.  创建一个名为`1-document-middleware.js`的新文件

1.  包含 Mongoose NPM 模块并创建到您的 MongoDB 的新连接：

```js
      const mongoose = require('mongoose') 
      const { connection, Schema } = mongoose 
      mongoose.connect( 
          'mongodb://localhost:27017/test' 
      ).catch(console.error) 
```

1.  定义一个模式：

```js
      const UserSchema = new Schema({ 
          firstName: { type: String, required: true }, 
          lastName: { type: String, required: true }, 
      }) 
```

1.  为`init`文档方法添加`pre`和`post`钩子：

```js
      UserSchema.pre('init', async function preInit() { 
          console.log('A document is going to be initialized.') 
      }) 
      UserSchema.post('init', async function postInit() { 
          console.log('A document was initialized.') 
      }) 
```

1.  为`validate`文档方法添加`pre`和`post`钩子：

```js
      UserSchema.pre('validate', async function preValidate() { 
          console.log('A document is going to be validated.') 
      }) 
      UserSchema.post('validate', async function postValidate() { 
          console.log('All validation rules were executed.') 
      }) 
```

1.  为`save`文档方法添加`pre`和`post`钩子：

```js
      UserSchema.pre('save', async function preSave() { 
          console.log('Preparing to save the document') 
      }) 
      UserSchema.post('save', async function postSave() { 
          console.log(`A doc was saved id=${this.id}`) 
      }) 
```

1.  为`remove`文档方法添加`pre`和`post`钩子：

```js
      UserSchema.pre('remove', async function preRemove() { 
          console.log(`Doc with id=${this.id} will be removed`) 
      }) 
      UserSchema.post('remove', async function postRemove() { 
          console.log(`Doc with id=${this.id} was removed`) 
      }) 
```

1.  将模式编译成模型：

```js
      const User = mongoose.model('User', UserSchema) 
```

1.  一旦建立新连接，创建一个文档并执行一些基本操作，如保存、检索和删除文档：

```js
      connection.once('connected', async () => { 
          try { 
              const user = new User({ 
                  firstName: 'John', 
                  lastName: 'Smith', 
              }) 
              await user.save() 
              await User.findById(user.id) 
              await user.remove() 
              await connection.close() 
          } catch (error) { 
              await connection.close() 
              console.dir(error.message, { colors: true }) 
          } 
      }) 
```

1.  保存文件

1.  打开终端并运行：

```js
 node document-middleware.js
```

1.  在终端上，输出应该显示：

```js
      A document is going to be validated. 
      All validation rules were executed. 
      Preparing to save the document 
      A doc was saved id=[ID] 
      A document is going to be initialized. 
      A document was initialized. 
      Doc with id=[ID] will be removed 
      Doc with id=[ID] was removed 
```

当您保存一个文档时，首先触发`validation`钩子，以确保字段通过内置验证规则或自定义规则设置的规则。在您的代码中，字段被标记为必需的。然后它将触发`save`钩子。之后，使用模型方法从数据库中检索最近创建的用户时，一旦检索到文档，它将触发`init`钩子。最后，从数据库中删除文档将触发`remove`钩子。

在钩子中，您可以与文档交互。例如，以下`save`预钩子将修改`firstName`和`lastName`字段，使它们成为大写字符串：

```js
UserSchema.pre('save', async function preSave() { 
    this.firstName = this.firstName.toUpperCase() 
    this.lastName = this.lastName.toUpperCase() 
}) 
```

同样，我们可以在钩子中抛出错误，以防止下一个钩子被执行。例如：

```js
UserSchema.pre('save', async function preSave() { 
    throw new Error('Doc was prevented from being saved.') 
}) 
```

查询中间件函数的定义方式与文档中间件函数完全相同。但是，`this`的上下文不是指文档，而是指查询对象。查询中间件函数仅受支持于以下模型和查询函数：

+   `count`：计算符合特定查询条件的文档数量

+   `find`：返回符合特定查询条件的文档数组

+   `findOne`：返回符合特定查询条件的文档

+   `findOneAndRemove`：类似于`findOne`。但是，在找到文档后，它会被删除

+   `findOneAndUpdate`：类似于`findOne`，但是一旦找到符合特定查询条件的文档，还可以更新文档

+   `update`：更新符合特定查询条件的一个或多个文档

# 查询中间件函数

为查询内置方法创建预先和后续钩子：

1.  创建一个名为`2-query-middleware.js`的新文件

1.  包含 Mongoose NPM 模块并创建到您的 MongoDB 的新连接：

```js
      const mongoose = require('mongoose') 
      const { connection, Schema } = mongoose 
      mongoose.connect( 
          'mongodb://localhost:27017/test' 
      ).catch(console.error) 
```

1.  定义一个模式：

```js
      const UserSchema = new Schema({ 
          firstName: { type: String, required: true }, 
          lastName: { type: String, required: true }, 
      }) 
```

1.  为`count`、`find`、`findOne`和`update`方法定义预先和后续钩子：

```js
      UserSchema.pre('count', async function preCount() { 
          console.log( 
              `Preparing to count document with this criteria: 
              ${JSON.stringify(this._conditions)}` 
          ) 
      }) 
      UserSchema.post('count', async function postCount(count) { 
          console.log(`Counted ${count} documents that coincide`) 
      }) 
      UserSchema.pre('find', async function preFind() { 
          console.log( 
              `Preparing to find all documents with criteria: 
              ${JSON.stringify(this._conditions)}` 
          ) 
      }) 
      UserSchema.post('find', async function postFind(docs) { 
          console.log(`Found ${docs.length} documents`) 
      }) 
      UserSchema.pre('findOne', async function prefOne() { 
          console.log( 
              `Preparing to find one document with criteria: 
              ${JSON.stringify(this._conditions)}` 
          ) 
      }) 
      UserSchema.post('findOne', async function postfOne(doc) { 
          console.log(`Found 1 document:`, JSON.stringify(doc)) 
      }) 
      UserSchema.pre('update', async function preUpdate() { 
          console.log( 
              `Preparing to update all documents with criteria: 
              ${JSON.stringify(this._conditions)}` 
          ) 
      }) 
      UserSchema.post('update', async function postUpdate(r) { 
          console.log(`${r.result.ok} document(s) were updated`) 
      }) 
```

1.  将模式编译成模型：

```js
      const User = mongoose.model('User', UserSchema) 
```

1.  一旦成功连接到数据库，创建一个文档，保存它，并使用我们为其定义了钩子的方法：

```js
      connection.once('connected', async () => { 
          try { 
              const user = new User({ 
                  firstName: 'John', 
                  lastName: 'Smith', 
              }) 
              await user.save() 
              await User 
                  .where('firstName').equals('John') 
                  .update({ lastName: 'Anderson' }) 
              await User 
                  .findOne() 
                  .select(['lastName']) 
                  .where('firstName').equals('John') 
              await User 
                  .find() 
                  .where('firstName').equals('John') 
              await User 
                  .where('firstName').equals('Neo') 
                  .count() 
              await user.remove() 
          } catch (error) { 
              console.dir(error, { colors: true }) 
          } finally { 
              await connection.close() 
          } 
      }) 
```

1.  保存文件

1.  打开终端并运行：

```js
 node query-middleware.js
```

1.  在终端上，输出应该显示类似于：

```js
      Preparing to update all documents with criteria: 
                {"firstName":"John"} 
      1 document(s) were updated 
      Preparing to find one document with criteria: 
                {"firstName":"John"} 
      Found 1 document: {"_id":"[ID]","lastName":"Anderson"} 
      Preparing to find all documents with criteria: 
                {"firstName":"John"} 
      Found 1 documents 
      Preparing to count document with this criteria: 
                {"firstName":"Neo"} 
      Counted 0 documents that coincide 
```

最后，只有一个模型实例方法支持钩子：

+   `insertMany`：这会验证一个文档数组，并且只有当数组中的所有文档都通过验证时才会保存到数据库中

您可能已经猜到，模型中间件函数也是以与查询中间件方法和文档中间件方法相同的方式定义的。

# 模型中间件函数

为`insertMany`模型实例方法定义`pre`和`post`钩子：

1.  创建一个名为`3-model-middleware.js`的新文件

1.  包括 Mongoose NPM 模块并创建到您的 MongoDB 的新连接：

```js
      const mongoose = require('mongoose') 
      const { connection, Schema } = mongoose 
      mongoose.connect( 
          'mongodb://localhost:27017/test' 
      ).catch(console.error) 
```

1.  定义模式：

```js
      const UserSchema = new Schema({ 
          firstName: { type: String, required: true }, 
          lastName: { type: String, required: true }, 
      }) 
```

1.  为`insertMany`模型方法定义`pre`和`post`钩子：

```js
      UserSchema.pre('insertMany', async function prMany() { 
          console.log('Preparing docs...') 
      }) 
      UserSchema.post('insertMany', async function psMany(docs) { 
          console.log('The following docs were created:n', docs) 
      }) 
```

1.  将模式编译成模型：

```js
      const User = mongoose.model('User', UserSchema) 
```

1.  一旦建立了与数据库的连接，就可以使用`insertMany`方法一次插入两个文档：

```js
      connection.once('connected', async () => { 
          try { 
              await User.insertMany([ 
                  { firstName: 'Leo', lastName: 'Smith' }, 
                  { firstName: 'Neo', lastName: 'Jackson' }, 
              ]) 
          } catch (error) { 
              console.dir(error, { colors: true }) 
          } finally { 
              await connection.close() 
          } 
      }) 
```

1.  保存文件

1.  打开终端并运行：

```js
 node query-middleware.js
```

1.  在终端上，输出应该显示：

```js
      Preparing docs... 
      The following documents were created: 
      [ { firstName: 'Leo', lastName: 'Smith', _id: [id] }, 
        { firstName: 'Neo', lastName: 'Jackson', _id: [id] } ] 
```

# 还有更多...

将字段标记为必填项很有用，以避免在数据库中保存“null”值。另一种选择是为在文档创建时未明确定义的字段设置默认值。例如：

```js
      const UserSchema = new Schema({ 
          name: { 
              type: string, 
              required: true, 
              default: 'unknown', 
          } 
      }) 
```

创建新文档时，如果未分配路径或属性`name`，则它将分配模式类型选项`default`中定义的默认值。

模式类型`default`选项也可以是一个函数。调用此函数返回的值将被分配为默认值。

子文档或数组也可以通过在定义模式类型时添加括号来创建。例如：

```js
      const WishBoxSchema = new Schema({ 
          wishes: { 
              type: [String], 
              required: true, 
              default: [ 
                  'To be a snowman', 
                  'To be a string', 
                  'To be an example', 
              ], 
          }, 
      }) 
```

创建新文档时，它将期望`wishes`属性或路径中的字符串数组。如果未提供数组，则将使用默认值来创建文档。

# 另请参阅

+   第一章，*MERN Stack 简介*，*安装 NPM 包*部分

+   第一章，*MERN Stack 简介*，*安装 MongoDB*部分

# 为 Mongoose 的模式编写自定义验证器

Mongoose 有几个内置验证规则。例如，如果您使用模式类型为`string`并将其设置为`required`来定义属性，则将执行两个验证规则，一个用于检查属性是否为有效的`string`，另一个用于检查属性是否不为`null`或`undefined`。

Mongoose 还可以在 Mongoose 中定义自定义验证规则和自定义错误验证消息，以便更好地控制何时以及何时接受某些属性，然后才能将其保存到数据库中。

验证规则在模式中定义。所有模式类型都有一个内置验证器`required`，这意味着它不能包含`undefined`或`null`值。`required`验证器可以是`boolean`、`function`或`array`类型。例如：

```js
      path: { type: String, required: true } 
      path: { type: String, required: [true, 'Custom error message'] } 
      path: { type: String, required: () => true } 
```

字符串模式类型具有以下内置验证器：

+   `enum`：这表示字符串只能具有`enum`数组中指定的值。例如：

```js
      gender: { 
      type: SchemaTypes.String, 
      enum: ['male', 'female', 'other'], 
      } 
```

+   `match`：这使用`RegExp`来测试值。例如，允许以`www`开头的值：

```js
      website: { 
      type: SchemaTypes.String, 
      match: /^www/, 
      } 
```

+   `maxlength`：这定义了字符串的最大长度。

+   `minlength`：这定义了字符串的最小长度。例如，只允许`5`和`20`个字符之间的字符串：

```js
      name: { 
      type: SchemaTypes.String, 
      minlength: 5, 
      maxlength: 20, 
      } 
```

数字模式类型有两个内置验证器：

+   `min`：这定义了数字的最小值。

+   `max`：这定义了数字的最大值。例如，只允许`18`和`100`之间的数字：

```js
      age: { 
      type: String, 
      min: 18, 
      max: 100, 
      } 
```

未定义的值可以通过所有验证器而不出错。如果要在值为`undefined`时抛出错误，请不要忘记将`required`验证器设置为`true`

当内置验证器有时无法满足您的要求，或者您希望执行复杂的验证规则时，您有一个名为`validate`的选项或属性。这个选项接受一个具有两个属性`validator`和`message`的对象，允许我们编写自定义验证器：

```js
      nickname: { 
      type: String, 
      validate: { 
      validator: function validator(value) { 
      return /^[a-zA-Z-]$/.test(value) 
      }, 
      message: '{VALUE} is not a valid nickname.', 
      }, 
      } 
```

# 准备工作

在本教程中，您将看到如何使用自定义验证规则来确保某个字段匹配或满足定义的规则。首先确保您已安装 MongoDB 并且它正在运行。或者，如果您愿意，云中的 MongoDB DBaaS 实例也可以。在开始之前，创建一个新的`package.json`文件，其中包含以下代码：

```js
      { 
        "dependencies": { 
          "mongoose": "5.0.11" 
        } 
      } 
```

然后，通过打开终端并运行以下命令来安装依赖项：

```js
 npm install  
```

# 操作步骤...

创建一个用户模式，并确保所有用户名都是字符串类型，最小长度为六个字符，最大长度为 20 个字符，匹配正则表达式，并且是必需的：

1.  创建一个名为`custom-validation.js`的新文件

1.  包括 Mongoose NPM 模块，并创建与数据库的新连接：

```js
      const mongoose = require('mongoose') 
      const { connection, Schema } = mongoose 
      mongoose.connect( 
          'mongodb://localhost:27017/test' 
      ).catch(console.error) 
```

1.  定义包括`username`字段的验证规则的模式：

```js
      const UserSchema = new Schema({ 
          username: { 
              type: String, 
              minlength: 6, 
              maxlength: 20, 
              required: [true, 'user is required'], 
              validate: { 
                  message: '{VALUE} is not a valid username', 
                  validator: (val) => /^[a-zA-Z]+$/.test(val), 
              }, 
          }, 
      }) 
```

1.  将模式编译成模型：

```js
      const User = mongoose.model('User', UserSchema) 
```

1.  一旦与数据库建立连接，创建一个带有无效字段的新文档，并使用`validateSync`文档方法来触发内置和自定义方法的验证：

```js
      connection.once('connected', async () => { 
          try { 
              const user = new User() 
              let errors = null 
              // username field is not defined 
              errors = user.validateSync() 
              console.dir(errors.errors['username'].message) 
              // username contains less than 6 characters 
              user.username = 'Smith' 
              errors = user.validateSync() 
              console.dir(errors.errors['username'].message) 
              // RegExp matching 
              user.username = 'Smith_9876' 
              errors = user.validateSync() 
              console.dir(errors.errors['username'].message) 
          } catch (error) { 
              console.dir(error, { colors: true }) 
          } finally { 
              await connection.close() 
          } 
      }) 
```

1.  保存文件

1.  打开终端并运行：

```js
 node custom-validation.js  
```

1.  在终端上，输出应显示：

```js
      'user is required' 
      'Path `username` (`Smith`) is shorter than the minimum allowed             
       length (6).' 
      'Smith_9876 is not a valid username' 
```

# 另请参阅

+   第一章，*MERN Stack 简介*，*安装 NPM 包*部分

+   第一章，*MERN Stack 简介*，**安装 MongoDB**部分

# 使用 ExpressJS 和 Mongoose 构建 RESTful API 来管理用户

在本教程中，您将构建一个 RESTful API，允许创建新用户、登录、显示用户信息和删除用户配置文件。此外，您还将学习如何构建一个带有客户端 API 的 NodeJS REPL，您可以使用它与服务器的 RESTful API 进行交互。

**REPL**（**Read-Eval-Print Loop**）类似于一个交互式 shell，您可以依次执行命令。例如，可以通过在终端中运行此命令来打开 Node.js REPL：

```js
node -i 
```

这里，`-i`标志代表交互式。现在，您可以执行 JavaScript 代码，该代码将逐段在新上下文中进行评估。

# 准备工作

本教程将重点介绍 Mongoose 与 ExpressJS 的集成，使用了之前教程中所见的内容。首先确保您已安装 MongoDB 并且它正在运行。或者，如果您愿意，云中的 MongoDB DBaaS 实例也可以。在开始之前，创建一个新的`package.json`文件，其中包含以下代码：

```js
      { 
        "dependencies": { 
          "body-parser": "1.18.2", 
          "connect-mongo": "2.0.1", 
          "express": "4.16.3", 
          "express-session": "1.15.6", 
          "mongoose": "5.0.11", 
          "node-fetch": "2.1.2" 
        } 
      } 
```

然后，通过打开终端并运行此代码来安装依赖项：

```js
npm install
```

# 操作步骤...

首先，创建一个名为`server.js`的文件，其中包含两个中间件函数。一个配置会话，另一个确保在允许调用任何路由之前有一个与 MongoDB 的连接。然后，我们将我们的 API 路由挂载到特定路径：

1.  创建一个名为`server.js`的新文件

1.  包括所需的库。然后，初始化一个新的 ExpressJS 应用程序，并创建与 MongoDB 的连接：

```js
      const mongoose = require('mongoose') 
      const express = require('express') 
      const session = require('express-session') 
      const bodyParser = require('body-parser') 
      const MongoStore = require('connect-mongo')(session) 
      const api = require('./api/controller') 
      const app = express() 
      const db = mongoose.connect( 
          'mongodb://localhost:27017/test' 
      ).then(conn => conn).catch(console.error) 
```

1.  使用`body-parser`中间件将请求体解析为 JSON：

```js
      app.use(bodyParser.json()) 
```

1.  定义一个 ExpressJS 中间件函数，确保您的 Web 应用在允许执行下一个路由处理程序之前首先连接到 MongoDB：

```js
      app.use((request, response, next) => {
        Promise.resolve(db).then(
        (connection, err) => (
            typeof connection !== 'undefined'
            ? next()
            : next(new Error('MongoError'))
            )
          )
      })
```

1.  配置`express-session`中间件以将会话存储在 Mongo 数据库中，而不是存储在内存中：

```js
      app.use(session({ 
          secret: 'MERN Cookbook Secrets', 
          resave: false, 
          saveUninitialized: true, 
          store: new MongoStore({ 
              collection: 'sessions', 
              mongooseConnection: mongoose.connection, 
          }), 
      })) 
```

1.  将 API 控制器挂载到`"/api"`路由：

```js
      app.use('/users', api) 
```

1.  监听端口 1773 以进行新连接：

```js
      app.listen( 
          1337, 
          () => console.log('Web Server running on port 1337'), 
      ) 
```

1.  保存文件

然后，创建一个名为`api`的新目录。接下来，创建应用程序的模型或业务逻辑。为用户定义一个包含静态和实例方法的模式，这些方法将允许用户注册、登录、注销、获取配置文件数据、更改密码和删除其配置文件：

1.  在`api`目录中创建一个名为`model.js`的新文件

1.  包括 Mongoose NPM 模块，还包括将用于生成用户密码哈希的`crypto` NodeJS 模块：

```js
      const { connection, Schema } = require('mongoose') 
      const crypto = require('crypto') 
```

1.  定义模式：

```js
      const UserSchema = new Schema({ 
          username: { 
              type: String, 
              minlength: 4, 
              maxlength: 20, 
              required: [true, 'username field is required.'], 
              validate: { 
                  validator: function (value) { 
                      return /^[a-zA-Z]+$/.test(value) 
                  }, 
                  message: '{VALUE} is not a valid username.', 
              }, 
          }, 
          password: String, 
      }) 
```

1.  为`login`定义一个静态模型方法：

```js
      UserSchema.static('login', async function(usr, pwd) { 
          const hash = crypto.createHash('sha256') 
              .update(String(pwd)) 
          const user = await this.findOne() 
              .where('username').equals(usr) 
              .where('password').equals(hash.digest('hex')) 
          if (!user) throw new Error('Incorrect credentials.') 
          delete user.password 
          return user 
      }) 
```

1.  为`signup`定义一个静态模型方法：

```js
      UserSchema.static('signup', async function(usr, pwd) { 
          if (pwd.length < 6) { 
              throw new Error('Pwd must have more than 6 chars') 
          } 
          const hash = crypto.createHash('sha256').update(pwd) 
          const exist = await this.findOne() 
              .where('username') 
              .equals(usr) 
          if (exist) throw new Error('Username already exists.') 
          const user = this.create({ 
              username: usr, 
              password: hash.digest('hex'), 
          }) 
          return user 
      }) 
```

1.  为`changePass`定义一个文档实例方法：

```js
      UserSchema.method('changePass', async function(pwd) { 
          if (pwd.length < 6) { 
              throw new Error('Pwd must have more than 6 chars') 
          } 
          const hash = crypto.createHash('sha256').update(pwd) 
          this.password = hash.digest('hex') 
          return this.save() 
      }) 
```

1.  将 Mongoose 模式编译为模型并导出它：

```js
      module.exports = connection.model('User', UserSchema) 
```

1.  保存文件

最后，定义一个控制器，将请求体转换为模型可以理解的操作。然后将其导出为包含所有 API 路径的 ExpressJS 路由：

1.  在`api`文件夹中创建一个名为`controller.js`的新文件

1.  导入`model.js`并初始化一个新的 ExpressJS 路由：

```js
      const express = require('express') 
      const User = require('./model') 
      const api = express.Router() 
```

1.  定义一个请求处理程序，检查用户是否已登录，另一个请求处理程序检查用户是否未登录：

```js
      const isLogged = ({ session }, res, next) => { 
          if (!session.user) res.status(403).json({ 
              status: 'You are not logged in!', 
          }) 
          else next() 
      } 
      const isNotLogged = ({ session }, res, next) => { 
          if (session.user) res.status(403).json({ 
              status: 'You are logged in already!', 
          }) 
          else next() 
      } 
```

1.  定义一个`post`请求方法来处理对`"/login"`端点的请求：

```js
      api.post('/login', isNotLogged, async (req, res) => { 
          try { 
              const { session, body } = req 
        const { username, password } = body 
              const user = await User.login(username, password) 
              session.user = { 
                  _id: user._id, 
                  username: user.username, 
              } 
              session.save(() => { 
                  res.status(200).json({ status: 'Welcome!'}) 
              }) 
          } catch (error) { 
              res.status(403).json({ error: error.message }) 
          } 
      }) 
```

1.  定义一个`post`请求方法来处理对`"/logout"`端点的请求：

```js
      api.post('/logout', isLogged, (req, res) => { 
          req.session.destroy() 
          res.status(200).send({ status: 'Bye bye!' }) 
      }) 
```

1.  定义一个`post`请求方法来处理对`"/signup"`端点的请求：

```js
      api.post('/signup', async (req, res) => { 
          try { 
              const { session, body } = req 
              const { username, password } = body 
              const user = await User.signup(username, password) 
              res.status(201).json({ status: 'Created!'}) 
          } catch (error) { 
              res.status(403).json({ error: error.message }) 
          } 
      }) 
```

1.  定义一个`get`请求方法来处理对`"/profile"`端点的请求：

```js
      api.get('/profile', isLogged, (req, res) => { 
          const { user } = req.session 
          res.status(200).json({ user }) 
      }) 
```

1.  定义一个`put`请求方法来处理对`"/changepass"`端点的请求：

```js
      api.put('/changepass', isLogged, async (req, res) => { 
          try { 
              const { session, body } = req 
              const { password } = body 
              const { _id } = session.user 
              const user = await User.findOne({ _id }) 
              if (user) { 
                  await user.changePass(password) 
                  res.status(200).json({ status: 'Pwd changed' }) 
              } else { 
                  res.status(403).json({ status: user }) 
              } 
          } catch (error) { 
              res.status(403).json({ error: error.message }) 
          } 
      }) 
```

1.  定义一个`delete`请求方法来处理对`"/delete"`端点的请求：

```js
      api.delete('/delete', isLogged, async (req, res) => { 
          try { 
              const { _id } = req.session.user 
              const user = await User.findOne({ _id }) 
              await user.remove() 
              req.session.destroy((err) => { 
                  if (err) throw new Error(err) 
                  res.status(200).json({ status: 'Deleted!'}) 
              }) 
          } catch (error) { 
              res.status(403).json({ error: error.message }) 
          } 
      }) 
```

1.  导出路由：

```js
      module.exports = api 
```

1.  保存文件

# 让我们来测试一下...

您已经构建了一个 RESTful API，允许用户订阅或注册、登录、注销、获取其个人资料和删除其个人资料。这些操作可以通过向服务器发出 HTTP 请求来执行。现在我们将构建一个小型的 NodeJS REPL 和客户端 API，可以让您使用纯 JavaScript 函数与您的 RESTful API 服务器进行交互：

1.  转到项目目录的根目录，并创建一个名为`client-repl.js`的新文件。

1.  包括`node-fetch` NPM 模块，允许向服务器发出 HTTP 请求。还包括`repl`和`vm` Node.js 模块，允许您创建交互式 Node.js REPL：

```js
      const repl = require('repl') 
      const util = require('util') 
      const vm = require('vm') 
      const fetch = require('node-fetch') 
      const { Headers } = fetch 
```

1.  定义一个变量，稍后将包含来自 cookie 的会话 ID，一旦用户登录。该 cookie 将用于让服务器识别已登录用户，以执行诸如获取有关您的个人资料或更改密码的操作：

```js
      let cookie = null 
```

1.  定义一个名为`query`的辅助函数，允许向服务器发出 HTTP 请求。`credentials`选项允许从服务器发送和接收 cookie。我们定义了`headers`，告诉服务器请求体的内容类型将以 JSON 内容发送：

```js
      const query = (path, ops) => { 
          return fetch(`http://localhost:1337/users/${path}`, { 
              method: ops.method, 
              body: ops.body, 
              credentials: 'include', 
              body: JSON.stringify(ops.body), 
              headers: new Headers({ 
                  ...(ops.headers || {}), 
                  cookie, 
                  Accept: 'application/json', 
                  'Content-Type': 'application/json', 
              }), 
          }).then(async (r) => { 
              cookie = r.headers.get('set-cookie') || cookie 
              return { 
                  data: await r.json(), 
                  status: r.status, 
              } 
          }).catch(error => error) 
      } 
```

1.  定义一个允许用户注册的方法：

```js
      const signup = (username, password) => query('/signup', { 
          method: 'POST', 
          body: { username, password }, 
      }) 
```

1.  定义一个允许用户登录的方法：

```js
      const login = (username, password) => query('/login', { 
          method: 'POST', 
          body: { username, password }, 
      }) 
```

1.  定义一个允许用户注销的方法：

```js
      const logout = () => query('/logout', { 
          method: 'POST', 
      }) 
```

1.  定义一个允许用户获取其个人资料的方法：

```js
      const getProfile = () => query('/profile', { 
          method: 'GET', 
      }) 
```

1.  定义一个允许用户更改密码的方法：

```js
      const changePassword = (password) => query('/changepass', { 
          method: 'PUT', 
          body: { password }, 
      }) 
```

1.  定义一个允许用户删除其个人资料的方法：

```js
      const deleteProfile = () => query('/delete', { 
          method: 'DELETE', 
      }) 
```

1.  使用 REPL 导出对象的 start 方法启动新的 REPL 服务器。我们将指定 eval 方法使用 VM 模块执行 JavaScript 代码，然后，如果返回 Promise，它将等待 Promise 解析后才允许用户输入更多命令或在 REPL 中输入更多 JavaScript 代码。我们还将指定 writer 方法，用于漂亮打印调用先前定义方法的结果：

```js
      const replServer = repl.start({ 
          prompt: '> ', 
          ignoreUndefined: true, 
          async eval(cmd, context, filename, callback) { 
              const script = new vm.Script(cmd) 
              const is_raw = process.stdin.isRaw 
              process.stdin.setRawMode(false) 
              try { 
                  const res = await Promise.resolve( 
                      script.runInContext(context, { 
                          displayErrors: false, 
                          breakOnSigint: true, 
                      }) 
                  ) 
                  callback(null, res) 
              } catch (error) { 
                  callback(error) 
              } finally { 
                  process.stdin.setRawMode(is_raw) 
              } 
          }, 
          writer(output) { 
              return util.inspect(output, { 
                  breakLength: process.stdout.columns, 
                  colors: true, 
                  compact: false, 
              }) 
          } 
      }) 
```

1.  将先前定义的方法添加到 REPL 服务器的上下文中，JavaScript 代码将在其中执行：

```js
      replServer.context.api = { 
          signup, 
          login, 
          logout, 
          getProfile, 
          changePassword, 
          deleteProfile, 
      } 
```

1.  保存文件

现在您可以在终端上运行您的 RESTful API 服务器：

```js
node server.js 
```

在另一个终端中，运行您刚刚创建的 NodeJS REPL 应用程序：

```js
node client-repl.js
```

在 REPL 中，您可以执行 JavaScript 代码，并且还可以访问导出的方法。例如，您可以在 REPL 中逐行执行以下 JavaScript 代码：

```js
      api.signup('John', 'zxcvbnm') 
      api.login('John', 'zxcvbnm') 
      api.getProfile() 
      api.changePassword('newPwd') 
      api.logout() 
      api.login('John', 'incorrectPwd') 
```

# 工作原理...

您的 RESTful API 服务器将接受以下路径的请求：

+   `POST/users/login`：如果在 MongoDB 的`users`集合中不存在用户名，则向客户端发送错误消息。否则，它会返回欢迎消息。

+   `POST/users/logout`：这将销毁会话 ID。

+   `POST/users/signup`：这将使用定义的密码创建一个新的用户名。但是，如果用户名或密码未通过验证，则会向客户端发送错误消息。当用户名已存在时，它还会向客户端发送错误消息。

+   `GET/users/profile`：如果用户已登录，则会向客户端发送用户信息。否则，会向客户端发送错误消息。

+   `PUT/users/changepass/`：这将更改当前已登录用户的密码。但是，如果用户未登录，则会向客户端发送错误消息。

+   `DELETE/users/delete`：这将从 MongoDB 的`users`集合中删除已登录用户的个人资料。会话将被销毁，并向客户端发送确认消息。如果用户未登录，则向客户端发送错误消息。

# 另请参阅

+   第一章，MERN Stack 简介，安装 NPM 包部分

+   第一章，MERN Stack 简介，安装 MongoDB 部分
