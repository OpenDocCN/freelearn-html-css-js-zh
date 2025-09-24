# 构建 RESTful API

在本章中，我们将涵盖以下菜谱：

+   使用 ExpressJS 的路由方法进行 CRUD 操作

+   使用 Mongoose 进行 CRUD 操作

+   使用 Mongoose 查询构建器

+   定义文档实例方法

+   定义静态模型方法

+   为 Mongoose 编写中间件函数

+   为 Mongoose 的模式编写自定义验证器

+   使用 ExpressJS 和 Mongoose 构建用于管理用户的 RESTful API

# 技术要求

你需要有一个 IDE，Visual Studio Code，Node.js 和 MongoDB。你还需要安装 Git，以便使用本书的 Git 仓库。

本章的代码文件可以在 GitHub 上找到：

[`github.com/PacktPublishing/MERN-Quick-Start-Guide/tree/master/Chapter03`](https://github.com/PacktPublishing/MERN-Quick-Start-Guide/tree/master/Chapter03)

查看以下视频以查看代码的实际效果：

[`goo.gl/73dE6u`](https://goo.gl/73dE6u)

# 简介

**表示状态传输**（**REST**）是构建在 Web 之上的架构风格。更具体地说，HTTP 1.1 协议标准是使用 REST 原则构建的。REST 提供了资源的表示。**URL**（**统一资源定位符**）用于定义资源的位置，并告诉浏览器它在何处。

RESTful API 是一种遵循此架构风格的网络服务 API。

最常用的 HTTP 动词或方法是：`POST`、`GET`、`PUT`和`DELETE`。这些方法是持久存储的基础，被称为**CRUD**操作（**创建、读取、更新和删除**）。

在本章中，我们将专注于使用 ExpressJS 和 Mongoose 的 REST 架构风格构建 RESTful API。

# 使用 ExpressJS 的路由方法进行 CRUD 操作

ExpressJS 的路由器有处理 HTTP 方法的等效方法。换句话说，HTTP 方法`POST`、`GET`、`PUT`和`DELETE`可以通过以下代码处理：

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

好好想想，每个 URL 都可以看作一个名词，因此一个动词可以作用于它。实际上，HTTP 方法也被称为 HTTP 动词。如果我们把它们看作动词，当对 RESTful API 发起请求时，它们可以被理解为：

+   发布一个用户

+   获取一个用户

+   更新一个用户

+   删除一个用户。

在**MVC**（**模型-视图-控制器**）架构模式中，控制器负责将输入转换为模型或视图可以理解的内容。换句话说，它们将输入转换为动作或命令，并将它们发送到模型或视图以进行相应的更新。

ExpressJS 的路由方法通常充当控制器。它们只是从客户端获取输入，例如来自浏览器的请求，然后将输入转换为动作。这些动作随后被发送到模型，即应用程序的业务逻辑，例如 mongoose 模型，或者发送到视图（一个 ReactJS 客户端应用程序）以进行更新。

# 准备工作

考虑到我们可以使用 HTTP 方法在资源上调用操作，我们将了解如何根据这些概念构建 RESTful API Web 服务。在开始之前，创建一个新的`package.json`文件，包含以下代码：

```js
      { 
        "dependencies": { 
          "express": "4.16.3", 
          "node-fetch": "2.1.1", 
          "uuid": "3.2.1" 
        } 
      } 
```

然后，通过打开终端并运行以下行代码来安装依赖项：

```js
 npm install
```

# 如何实现...

使用内存数据库或包含用户列表的对象数组构建 RESTful API。我们将允许使用 HTTP 方法进行 CRUD 操作，以添加新用户、获取用户或用户列表、更新用户数据以及删除用户：

1.  创建一个名为`restfulapi.js`的新文件。

1.  导入所需的包并创建一个 ExpressJS 应用程序：

```js
     const express = require('express') 
      const uuid = require('uuid') 
      const app = express() 
```

1.  定义一个内存数据库：

```js
      let data = [ 
          { id: uuid(), name: 'Bob' }, 
          { id: uuid(), name: 'Alice' }, 
      ] 
```

1.  创建一个包含执行 CRUD 操作函数的模型：

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

1.  为`post`方法添加一个请求处理器，该处理器将用作`Create`操作。新用户将被添加到`data`数组中：

```js
      app.post('/users/:name', (req, res) => { 
          res.status(201).json(usr.create(req.params.name)) 
      }) 
```

1.  为`get`方法添加一个请求处理器，该处理器将用作`Read`或`Retrieve`操作。如果提供了`id`，则在`data`数组中查找用户。然而，如果提供的`id`是`"all"`，它将返回整个用户列表：

```js
      app.get('/users/:id', (req, res) => { 
          res.status(200).json(usr.read(req.params.id)) 
      }) 
```

1.  为`put`方法添加一个请求处理器，该处理器将用作`Update`操作。需要提供一个`id`以更新`data`数组中的特定用户：

```js
      app.put('/users/:id=:name', (req, res) => { 
          res.status(200).json(usr.update( 
              req.params.id, 
              req.params.name, 
          )) 
      }) 
```

1.  为`delete`方法添加一个请求处理器，该处理器将用作`Delete`操作。它将在`data`数组中查找用户并将其删除：

```js
      app.delete('/users/:id', (req, res) => { 
          res.status(200).json(usr.delete(req.params.id)) 
      }) 
```

1.  启动应用程序，监听端口`1337`以接收新连接：

```js
      app.listen( 
          1337, 
          () => console.log('Web Server running on port 1337'), 
      ) 
```

1.  保存文件。

1.  打开终端并运行以下代码：

```js
 node restfulapi.js
```

# 让我们测试它...

为了简化，创建一个脚本，该脚本将请求并对我们的 RESTful API 服务器执行 CRUD 操作：

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

1.  打开一个新的终端并运行以下代码：

```js
    node test-restfulapi.js

```

# 它是如何工作的...

我们的 RESTful API 应用程序将本地运行在端口`1337`上。在运行测试代码时，它将连接到它并使用不同的 HTTP 方法创建用户、检索用户、更新用户和删除用户。所有操作都将记录在终端中。

如果您想亲自测试，可以替换`test`函数内的所有代码，并使用`r`函数进行自定义请求。例如，要创建一个名为`Smith`的新用户：

```js
r(`/users/Smith`, 'post') 
```

# 使用 Mongoose 进行 CRUD 操作

开发者选择使用 Mongoose 而不是 Node.js 的官方 MongoDB 驱动程序的原因之一是，它允许您通过使用模式轻松创建数据结构，并且还因为内置的验证。MongoDB 是一个面向文档的数据库，这意味着文档的结构是可变的。

在 MVC 架构模式中，Mongoose 通常用于创建定义数据结构的模型。

这就是典型的 Mongoose 模式定义及其编译成模型的方式：

```js
      const PersonSchema = new Schema({ 
          firstName: String, 
          lastName: String, 
      }) 
      const Person = connection.model('Person', PersonSchema) 
```

模型名称应该是单数形式，因为 Mongoose 会在保存集合到数据库时将它们转换为复数并转换为小写。例如，如果模型名为 "User"，它将在 MongoDB 中保存为名为 "users" 的集合。Mongoose 包含一个内部字典来复数化常见名称。这意味着如果你的模型名称是常见名称，例如 "Person"，它将在 MongoDB 中保存为名为 "people" 的集合。

Mongoose 允许以下类型来定义模式路径或文档结构：

+   字符串

+   数字

+   布尔值

+   数组

+   日期

+   缓冲区

+   混合

+   ObjectId

+   十进制 128

可以直接使用 `String`、`Number`、`Boolean`、`Buffer` 和 `Date` 的全局构造函数来声明模式类型：

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

这些模式类型也在导出的 `mongoose` 对象下的 `SchemaTypes` 对象中可用：

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

使用对象作为属性来声明模式类型可以让你对特定的模式类型有更多的控制。以下是一个示例代码：

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

模式类型也可以是数组。例如，如果我们想在一个字符串数组中定义用户喜欢的项目，可以使用以下代码：

```js
      const PersonSchema = new Schema({ 
          name: String, 
          age: Number, 
          likes: [String], 
      }) 
```

要了解更多关于模式类型的信息，请访问官方 Mongoose 文档网站：[`mongoosejs.com/docs/schematypes.html`](http://mongoosejs.com/docs/schematypes.html)。

# 准备工作

在本教程中，你将了解如何定义模式和在数据库集合上执行 CRUD 操作。首先，确保你已经安装了 MongoDB 并且它在运行。作为替代，如果你更喜欢，也可以使用云中的 MongoDB **DBaaS**（数据库即服务）实例。在你开始之前，创建一个包含以下代码的新 `package.json` 文件：

```js
      { 
        "dependencies": { 
          "mongoose": "5.0.11" 
       } 
     } 
```

然后，通过打开终端并运行以下代码来安装依赖项：

```js
 npm install
```

# 如何做到这一点...

定义一个包含用户名、姓氏和用户喜欢的字符串数组的内容的用户模式：

1.  创建一个名为 `mongoose-models.js` 的新文件

1.  包含 Mongoose NPM 模块。然后，创建一个连接到 MongoDB：

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

1.  定义一个用于通过用户的 `id` 从用户集合中检索用户的函数：

```js
      const getUser = (id) => User.findById(id) 
```

1.  定义一个函数，通过用户的 `id` 从用户集合中删除用户：

```js
      const removeUser = (id) => User.remove({ id }) 
```

1.  定义一个事件监听器，在数据库连接建立后执行 CRUD 操作。首先，添加一个新用户并保存它。然后，使用其 `id` 检索相同的用户。接下来，修改用户的属性并保存它。最后，通过其 `id` 从集合中删除用户：

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

1.  打开终端并运行以下代码：

```js
    node mongoose-models.js
```

在终端中执行前面的命令，如果成功，将显示类似以下内容，例如，如下代码：

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

# 参见

+   第一章，*MERN Stack 简介*，*安装 NPM 包*部分

+   第一章，*MERN Stack 简介*，部分 *安装 MongoDB*

# 使用 Mongoose 查询构建器

每个 Mongoose 模型都有静态辅助方法来执行多种操作，例如检索文档。当将这些辅助方法传递给回调函数时，操作会立即执行：

```js
      const user = await User.findOne({ 
          firstName: 'Jonh', 
          age: { $lte: 30 }, 
      }, (error, document) => { 
          if (error) return console.log(error) 
          console.log(document) 
      }) 
```

否则，如果没有定义回调，则返回一个 *查询构建器接口*，该接口可以稍后执行：

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

查询也有一个 `.then` 函数，它可以作为 `Promise` 使用。当调用 `.then` 时，它首先使用 `.exec` 内部执行查询，然后返回一个 `Promise`。这允许我们使用 `async/await`。在一个 `async` 函数内部，例如：

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

我们有两种方式可以执行查询。一种是通过提供一个 JSON 对象作为条件，另一种方式允许你使用链式语法创建查询。链式语法对于更熟悉 SQL 数据库的开发者来说会感觉更舒适。例如：

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

在这个菜谱中，你将使用链式语法和 `async/await` 函数构建查询。首先，确保你已经安装了 MongoDB 并且它在运行。作为替代，如果你愿意，也可以使用云中的 MongoDB DBaaS 实例。在你开始之前，创建一个新的 `package.json` 文件，包含以下代码：

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

# 如何做到这一点...

1.  创建一个名为 `chaining-queries.js` 的新文件

1.  包含 Mongoose NPM 模块。然后，创建一个新的连接：

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

1.  将模式编译成模型：

```js
      const User = mongoose.model('User', UserSchema) 
```

1.  一旦连接到数据库，向用户的集合中添加一个新的文档。然后，使用链式语法查询最近创建的用户。此外，使用 `select` 方法限制从文档中检索的字段：

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

# 参见

+   第一章，*MERN Stack 简介*，部分 *安装 NPM 包*

+   第一章，*MERN Stack 简介*，部分 *安装 MongoDB*

# 定义文档实例方法

文档有其内置的实例方法，例如 `save` 和 `remove`。然而，我们也可以编写自己的实例方法。

文档是模型的实例。它们可以被显式创建：

```js
      const instance = new Model() 
```

或者它们可以是查询的结果：

```js
      Model.findOne([conditions]).then((instance) => {}) 
```

文档实例方法在模式中定义。所有模式都有一个名为 `method` 的方法，它允许你定义自定义实例方法。

# 准备工作

在这个菜谱中，你将定义一个模式和自定义文档实例方法来修改和读取文档属性。首先，确保你已经安装了 MongoDB 并且它在运行。作为替代，如果你愿意，也可以使用云中的 MongoDB DBaaS 实例。在你开始之前，创建一个新的 `package.json` 文件，包含以下代码：

```js
{ 
  "dependencies": { 
    "mongoose": "5.0.11" 
  } 
} 
```

然后，通过打开终端并运行以下代码来安装依赖项：

```js
    npm install
```

# 如何做到这一点...

1.  创建一个名为`document-methods.js`的新文件

1.  包含 Mongoose NPM 模块。然后，创建一个新的 MongoDB 连接：

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

1.  定义一个用于从包含用户全名的字符串中设置用户名和姓氏的文档实例方法：

```js
      UserSchema.method('setFullName', function setFullName(v) { 
          const fullName = String(v).split(' ') 
          this.lastName = fullName[0] || '' 
          this.firstName = fullName[1] || '' 
      }) 
```

1.  定义一个用于获取用户全名的文档实例方法，通过连接`firstName`和`lastName`属性：

```js
      UserSchema.method('getFullName', function getFullName() { 
          return `${this.lastName} ${this.firstName}` 
      }) 
```

1.  定义一个名为`loves`的文档实例方法，它将期望一个参数，该参数将添加到字符串数组`likes`中：

```js
      UserSchema.method('loves', function loves(stuff) { 
          this.likes.push(stuff) 
      }) 
```

1.  定义一个名为`dislikes`的文档实例方法，该方法将从用户的`likes`数组中删除之前喜欢的一项：

```js
      UserSchema.method('dislikes', function dislikes(stuff) { 
          this.likes = this.likes.filter(str => str !== stuff) 
      }) 
```

1.  将模式编译成模型：

```js
      const User = mongoose.model('User', UserSchema) 
```

1.  一旦 Mongoose 连接到数据库，创建一个新的用户，并使用`setFullName`方法填充`firstName`和`lastName`字段，然后使用`loves`方法填充`likes`数组。接下来，使用链式语法查询用户集合中的用户，并使用`dislikes`方法从`likes`数组中删除`"snakes"`：

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

1.  打开终端并运行以下代码：

```js
       node document-methods.js
```

# 更多内容...

也可以使用`methods`和`schemas`属性定义文档实例方法。例如：

```js
UserSchema.methods.setFullName = function setFullName(v) { 
    const fullName = String(v).split(' ') 
    this.lastName = fullName[0] || '' 
    this.firstName = fullName[1] || '' 
} 
```

# 参见

+   第一章，*MERN 栈简介*，部分*安装 NPM 包*

+   第一章，*MERN 栈简介*，部分*安装 MongoDB*

# 定义静态模型方法

模型有内置的静态方法，如`find`、`findOne`和`findOneAndRemove`。Mongoose 允许我们定义自定义静态模型方法。静态模型方法与文档实例方法的定义方式相同。

模式有一个名为`statics`的属性，它是一个对象。在`statics`对象内部定义的所有方法都会传递给模型。也可以通过调用`static`模式方法来定义静态模型方法。

# 准备工作

在这个菜谱中，你将定义一个模式和自定义静态模型方法来扩展你的模型功能。首先，确保你已经安装了 MongoDB 并且它正在运行。作为替代，如果你愿意，也可以使用云中的 MongoDB DBaaS 实例。在开始之前，创建一个新的`package.json`文件，包含以下代码：

```js
{ 
  "dependencies": { 
    "mongoose": "5.0.11" 
  } 
} 
```

然后，通过打开终端并运行以下代码来安装依赖项：

```js
npm install
```

# 如何做到这一点...

定义一个名为`getByFullName`的静态模型方法，它将允许你使用全名搜索特定用户：

1.  创建一个名为`static-methods.js`的新文件

1.  包含 Mongoose NPM 模块并创建一个新的 MongoDB 连接：

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

1.  连接后，创建一个新的用户并保存它。然后，使用`getByFullName`静态模型方法在用户集合中通过全名查找用户：

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

静态模型方法也可以使用`statics`模式属性定义。例如：

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

# 参见

+   第一章，*MERN 栈简介*，部分*安装 NPM 包*

+   第一章，*MERN 栈简介*，部分*安装 MongoDB*

# 为 Mongoose 编写中间件函数

Mongoose 中的中间件也称为`hooks`。有两种类型的钩子：`pre hooks`和`post hooks`。

`pre hooks`和`post hooks`之间的区别非常简单。`pre hooks`在调用方法之前调用，而`post hooks`在调用之后调用。例如：

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

一旦与数据库建立连接，在`async`函数中：

```js
      const user = new User({ 
          firstName: 'John', 
          lastName: 'Smith', 
      }) 
      await user.save() 
```

一旦调用`save`方法，首先执行`pre hook`。文档保存后，然后执行`post hook`。在上一个示例中，它将在终端输出中显示以下文本：

```js
    New user created: Smith John
```

Mongoose 中有四种不同的中间件函数：文档中间件、模型中间件、聚合中间件和查询中间件。所有这些都在模式级别上定义。区别在于，当钩子执行时，`this`的上下文分别指向文档、模型、聚合对象或查询对象。

所有类型的中间件都支持`pre`和`post`钩子

# 准备工作

在这个菜谱中，我们将看到 Mongoose 中这三种类型的中间件函数是如何工作的：

+   文档中间件

+   模型中间件

+   查询中间件

首先，确保你已经安装了 MongoDB 并且正在运行。作为替代方案，如果你愿意，也可以使用云中的 MongoDB DBaaS 实例。在开始之前，创建一个包含以下代码的新`package.json`文件：

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

# 如何做到这一点...

在文档中间件函数中，`this`的上下文指的是文档。文档有以下内置方法，并且可以为它们定义`hooks`：

+   `init`：这是在文档从 MongoDB 返回后立即内部调用的。Mongoose 使用 setter 来标记文档已修改或哪些字段已修改。`init`初始化文档而不使用 setter。

+   `validate`：这将为文档执行内置和自定义的验证规则。

+   `save`：这将在数据库中保存文档。

+   `remove`：这将从数据库中删除文档。

# 文档中间件函数

为文档内置方法创建`pre`和`post`钩子：

1.  创建一个名为`1-document-middleware.js`的新文件

1.  包含 Mongoose NPM 模块并创建到 MongoDB 的新连接：

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

1.  将模式编译成一个模型：

```js
      const User = mongoose.model('User', UserSchema) 
```

1.  一旦建立新的连接，创建一个文档并执行一些基本操作，例如保存、检索和删除文档：

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

1.  打开一个终端并运行：

```js
 node document-middleware.js
```

1.  在终端上，输出应显示：

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

当你保存一个文档时，它首先触发 `validation` 钩子，以确保字段通过内置验证规则或自定义规则的规则。在你的代码中，字段被标记为必填。然后它将触发 `save` 钩子。之后，使用模型方法从数据库检索最近创建的用户，一旦文档被检索，它将触发 `init` 钩子。最后，从数据库中删除文档将触发 `remove` 钩子。

在钩子内部，你可以与文档交互。例如，以下 `save` 预钩子将修改 `firstName` 和 `lastName` 字段，使它们成为大写字符串：

```js
UserSchema.pre('save', async function preSave() { 
    this.firstName = this.firstName.toUpperCase() 
    this.lastName = this.lastName.toUpperCase() 
}) 
```

同样，我们可以在钩子内抛出错误以防止后续的钩子执行。例如：

```js
UserSchema.pre('save', async function preSave() { 
    throw new Error('Doc was prevented from being saved.') 
}) 
```

查询中间件函数的定义与文档中间件函数完全相同。然而，`this` 的上下文不指向文档，而是指向查询对象。查询中间件函数仅支持以下模型和查询函数：

+   `count`: 计算匹配特定查询条件的文档数量

+   `find`: 返回匹配特定查询条件的文档数组

+   `findOne`: 返回匹配特定查询条件的文档

+   `findOneAndRemove`: 与 `findOne` 类似。然而，在找到文档后，它会将其删除

+   `findOneAndUpdate`: 与 `findOne` 类似，但一旦找到匹配特定查询条件的文档，还可以更新该文档

+   `update`: 更新匹配特定查询条件的单个或多个文档

# 查询中间件函数

为查询内置方法创建预和后钩子：

1.  创建一个名为 `2-query-middleware.js` 的新文件

1.  包含 Mongoose NPM 模块并创建一个新的 MongoDB 连接：

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

1.  为 `count`、`find`、`findOne` 和 `update` 方法定义预和后钩子：

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

1.  将模式编译成一个模型：

```js
      const User = mongoose.model('User', UserSchema) 
```

1.  一旦成功连接到数据库，创建一个文档，保存它，并使用我们为以下方法定义的钩子：

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

1.  打开一个终端并运行：

```js
 node query-middleware.js
```

1.  在终端上，输出应显示类似以下内容：

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

+   `insertMany`: 这会验证一个文档数组，并且只有当数组中的所有文档都通过验证时，才会将它们保存到数据库中

如你所猜，模型中间件函数也是以与查询中间件方法和文档中间件方法相同的方式定义的。

# 模型中间件函数

为 `insertMany` 模型实例方法创建一个 `pre` 和 `post` 钩子：

1.  创建一个名为 `3-model-middleware.js` 的新文件

1.  包含 Mongoose NPM 模块并创建一个新的 MongoDB 连接：

```js
      const mongoose = require('mongoose') 
      const { connection, Schema } = mongoose 
      mongoose.connect( 
          'mongodb://localhost:27017/test' 
      ).catch(console.error) 
```

1.  定义一个架构：

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

1.  将架构编译成模型：

```js
      const User = mongoose.model('User', UserSchema) 
```

1.  一旦建立了数据库连接，就可以使用`insertMany`方法一次性插入两个文档：

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

# 更多内容...

标记字段为必填项很有用，这样可以避免在数据库中保存“null”值。另一种方法是，为在文档创建时未明确定义的字段设置默认值。例如：

```js
      const UserSchema = new Schema({ 
          name: { 
              type: string, 
              required: true, 
              default: 'unknown', 
          } 
      }) 
```

当创建新文档时，如果没有分配路径或属性`name`，则它将分配在架构类型选项`default`中定义的默认值。

架构类型的`default`选项也可以是一个函数。调用此函数返回的值被分配为默认值。

通过在定义架构类型时添加括号，也可以创建子文档或数组。例如：

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

当创建新文档时，它将期望在`wishes`属性或路径中有一个字符串数组。如果没有提供数组，则将使用默认值来创建文档。

# 参见

+   第一章，*MERN 栈简介*，部分*安装 NPM 包*

+   第一章，*MERN 栈简介*，部分*安装 MongoDB*

# 为 Mongoose 的架构编写自定义验证器

Mongoose 有几个内置验证规则。例如，如果你定义了一个具有`string`架构类型的属性并将其设置为`required`，将执行两个验证规则，一个检查属性是否为有效的`string`，另一个检查属性是否不是`null`或`undefined`。

可以在 Mongoose 中定义自定义验证规则和自定义错误验证消息，以便在将某些属性保存到数据库之前有更多的控制权。

验证规则在架构中定义。所有架构类型都有一个内置验证器`required`，这意味着它不能包含`undefined`或`null`值。`required`验证器可以是`boolean`类型、一个`function`或一个`array`。例如：

```js
      path: { type: String, required: true } 
      path: { type: String, required: [true, 'Custom error message'] } 
      path: { type: String, required: () => true } 
```

字符串架构类型有以下内置验证器：

+   `enum`：这表示字符串只能具有`enum`数组中指定的值。例如：

```js
      gender: { 
      type: SchemaTypes.String, 
      enum: ['male', 'female', 'other'], 
      } 
```

+   `match`：这使用`RegExp`来测试值。例如，只允许以`www`开头的值：

```js
      website: { 
      type: SchemaTypes.String, 
      match: /^www/, 
      } 
```

+   `maxlength`：这定义了一个字符串可以拥有的最大长度。

+   `minlength`：这定义了一个字符串可以拥有的最小长度。例如，只允许`5`到`20`个字符的字符串：

```js
      name: { 
      type: SchemaTypes.String, 
      minlength: 5, 
      maxlength: 20, 
      } 
```

数字架构类型有两个内置验证器：

+   `min`：这定义了一个数字可以拥有的最小值。

+   `max`：这定义了一个数字可以拥有的最大值。例如，只允许`18`到`100`之间的数字：

```js
      age: { 
      type: String, 
      min: 18, 
      max: 100, 
      } 
```

未定义的值会通过所有验证器而不会出错。如果你想在值是 `undefined` 时抛出错误，请务必使用 `required` 验证器将其设置为 `true`

当内置验证器无法满足你的要求或你希望执行复杂的验证规则时，你可以选择或属性名为 `validate`。它接受一个对象，该对象有两个属性，`validator` 和 `message`，允许我们编写自定义验证器：

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

在这个菜谱中，你将看到如何使用自定义验证规则来确保某个字段符合或满足定义的规则。首先，确保你已经安装了 MongoDB 并且它在运行。作为替代，如果你愿意，也可以使用云中的 MongoDB DBaaS 实例。在开始之前，创建一个新的 `package.json` 文件，并使用以下代码：

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

# 如何操作...

创建一个用户模式，并确保所有用户名都是字符串类型，最小长度为六个字符，最大长度为 20 个字符，匹配正则表达式，并且是必需的：

1.  创建一个名为 `custom-validation.js` 的新文件

1.  包含 Mongoose NPM 模块并创建一个新的数据库连接：

```js
      const mongoose = require('mongoose') 
      const { connection, Schema } = mongoose 
      mongoose.connect( 
          'mongodb://localhost:27017/test' 
      ).catch(console.error) 
```

1.  定义一个包含 `username` 字段验证规则的模式：

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

1.  一旦与数据库建立了连接，创建一个新的包含无效字段的文档，并使用 `validateSync` 文档方法来触发内置和自定义的验证方法：

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

# 参见

+   第一章，*MERN 栈简介*，部分 *安装 NPM 包*

+   第一章，*MERN 栈简介*，部分 **安装 MongoDB**

# 使用 ExpressJS 和 Mongoose 构建管理用户的 RESTful API

在这个菜谱中，你将构建一个 RESTful API，允许创建新用户、登录、显示用户信息和删除用户的个人资料。此外，你还将学习如何使用客户端 API 构建一个 NodeJS REPL，你可以使用它来与你的服务器 RESTful API 交互。

**REPL**（**读取-评估-打印循环**）就像一个交互式外壳，你可以依次执行命令。例如，Node.js REPL 可以通过在终端中运行以下命令来打开：

```js
node -i 
```

这里，`-i` 标志代表交互式。现在，你可以执行 JavaScript 代码，这些代码将在新的上下文中逐个评估。

# 准备工作

这个菜谱将专注于展示如何使用之前菜谱中看到的内容来集成 Mongoose 与 ExpressJS。首先，确保你已经安装了 MongoDB 并且它在运行。作为替代，如果你愿意，也可以使用云中的 MongoDB DBaaS 实例。在开始之前，创建一个新的 `package.json` 文件，并使用以下代码：

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

然后，通过打开终端并运行以下代码来安装依赖项：

```js
npm install
```

# 如何实现...

首先，创建一个名为`server.js`的文件，该文件将包含两个中间件函数。一个用于配置会话，另一个确保在调用任何路由之前先建立到 MongoDB 的连接。然后，我们将 API 路由挂载到特定路径：

1.  创建一个名为`server.js`的新文件

1.  包含所需的库。然后，初始化一个新的 ExpressJS 应用程序并创建到 MongoDB 的连接：

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

1.  定义一个 ExpressJS 中间件函数，确保在允许执行下一个路由处理器之前，首先确保您的 Web 应用程序连接到 MongoDB：

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

1.  配置`express-session`中间件，将会话存储在 Mongo 数据库中而不是内存中：

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

1.  监听端口 1773 以接收新连接：

```js
      app.listen( 
          1337, 
          () => console.log('Web Server running on port 1337'), 
      ) 
```

1.  保存文件

然后，创建一个名为`api`的新目录。接下来，创建应用程序的模型或业务逻辑。定义一个用户模式，其中包含静态和实例方法，允许用户注册、登录、登出、获取个人资料数据、更改密码和删除个人资料：

1.  在`api`目录中创建一个名为`model.js`的新文件

1.  包含 Mongoose NPM 模块以及将用于生成用户密码散列的`crypto` NodeJS 模块：

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

1.  定义一个静态模型方法`login`：

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

1.  定义一个静态模型方法`signup`：

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

1.  定义一个文档实例方法`changePass`：

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

1.  将 Mongoose 模式编译成模型并导出：

```js
      module.exports = connection.model('User', UserSchema) 
```

1.  保存文件

最后，定义一个控制器，将请求体转换为模型可以理解的操作。然后将其作为包含所有 API 路径的 ExpressJS 路由器导出：

1.  在`api`文件夹中创建一个名为`controller.js`的新文件

1.  导入`model.js`并初始化一个新的 ExpressJS 路由：

```js
      const express = require('express') 
      const User = require('./model') 
      const api = express.Router() 
```

1.  定义一个请求处理器来检查用户是否已登录，另一个请求处理器来检查用户是否未登录：

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

# 让我们测试一下...

您已经构建了一个 RESTful API，允许用户订阅或注册、登录、登出、获取个人资料和删除个人资料。这些操作可以通过向服务器发送 HTTP 请求来执行。现在，我们将构建一个小型的 NodeJS REPL 和客户端 API，允许您使用纯 JavaScript 函数与 RESTful API 服务器交互：

1.  移动到项目目录的根目录并创建一个名为`client-repl.js`的新文件。

1.  包含`node-fetch` NPM 模块，该模块允许向服务器发送 HTTP 请求。同时，包含`repl`和`vm` Node.js 模块，这些模块允许您创建一个交互式的 Node.js REPL：

```js
      const repl = require('repl') 
      const util = require('util') 
      const vm = require('vm') 
      const fetch = require('node-fetch') 
      const { Headers } = fetch 
```

1.  定义一个变量，该变量将在用户登录后包含来自 cookie 的会话 ID。cookie 将用于让服务器识别已登录用户，以便执行诸如获取个人资料信息或更改密码等操作：

```js
      let cookie = null 
```

1.  定义一个名为`query`的辅助函数，该函数将允许向服务器发送 HTTP 请求。`credentials`选项允许从服务器发送和接收 cookie。我们定义`headers`，这将告诉服务器要发送的请求体的内容类型为 JSON 内容：

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

1.  定义一个方法，允许用户注册：

```js
      const signup = (username, password) => query('/signup', { 
          method: 'POST', 
          body: { username, password }, 
      }) 
```

1.  定义一个方法，允许用户登录：

```js
      const login = (username, password) => query('/login', { 
          method: 'POST', 
          body: { username, password }, 
      }) 
```

1.  定义一个方法，允许用户注销：

```js
      const logout = () => query('/logout', { 
          method: 'POST', 
      }) 
```

1.  定义一个方法，允许用户获取其个人资料：

```js
      const getProfile = () => query('/profile', { 
          method: 'GET', 
      }) 
```

1.  定义一个方法，允许用户更改他们的密码：

```js
      const changePassword = (password) => query('/changepass', { 
          method: 'PUT', 
          body: { password }, 
      }) 
```

1.  定义一个方法，允许用户删除其个人资料：

```js
      const deleteProfile = () => query('/delete', { 
          method: 'DELETE', 
      }) 
```

1.  使用 REPL 导出的对象的 start 方法启动一个新的 REPL 服务器。我们将指定 eval 方法使用 VM 模块执行 JavaScript 代码，然后，如果返回 Promise，它将等待 Promise 解决后再允许用户输入更多命令或更多 JavaScript 代码到 REPL 中。我们还将指定 writer 方法，该方法将格式化打印之前定义的方法的调用结果：

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

1.  将之前定义的方法添加到执行 JavaScript 代码的 REPL 服务器上下文中：

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

现在，您可以在终端上运行您的 RESTful API 服务器：

```js
node server.js 
```

在不同的终端中，运行您刚刚创建的 NodeJS REPL 应用程序：

```js
node client-repl.js
```

在 REPL 中，您可以执行 JavaScript 代码，并且您还可以访问导出的方法。例如，您可以在您的 REPL 中逐行执行以下 JavaScript 代码：

```js
      api.signup('John', 'zxcvbnm') 
      api.login('John', 'zxcvbnm') 
      api.getProfile() 
      api.changePassword('newPwd') 
      api.logout() 
      api.login('John', 'incorrectPwd') 
```

# 它是如何工作的...

您的 RESTful API 服务器将接受以下路径的请求：

+   `POST/users/login`: 如果 MongoDB 中的`users`集合中不存在该用户名，将向客户端发送错误消息。否则，它返回一个欢迎消息。

+   `POST/users/logout`: 这将销毁会话 ID。

+   `POST/users/signup`: 这将使用定义的密码创建一个新的用户名。然而，如果用户名或密码未通过验证，将向客户端发送错误。如果用户名已存在，也会向客户端发送错误消息。

+   `GET/users/profile`: 如果用户已登录，将用户信息发送到客户端。否则，向客户端发送错误消息。

+   `PUT/users/changepass/`: 这将更改当前登录用户的密码。然而，如果用户未登录，将向客户端发送错误消息。

+   `DELETE/users/delete`：这将从 MongoDB 的 `users` 集合中删除已登录用户的个人资料。会话将被销毁，并向客户端发送确认消息。如果用户未登录，将向客户端发送错误消息

# 参见

+   第一章，*MERN Stack 简介*，部分 *安装 NPM 包*

+   第一章，*MERN Stack 简介*，部分 *安装 MongoDB*
