# 第七章：Mongoose

Mongoose 是本书中将要介绍的第三个也是最后一个数据库映射工具。它是 JavaScript 世界中最知名的 MongoDB 映射工具。

# 关于 MongoDB 的一点说明

当 MongoDB 最初发布时，即 2009 年，它震惊了数据库世界。那时使用的绝大多数数据库都是关系型的，而 MongoDB 迅速成长为最受欢迎的非关系型数据库（也称为“NoSQL”）。

NoSQL 数据库与关系型数据库（如 MySQL、PostgreSQL 等）不同，它们以其他方式对存储的数据进行建模，而不是相互关联的表。

具体来说，MongoDB 是一种“面向文档的数据库”。它以 BSON 格式（“二进制 JSON”，一种包含特定于 MongoDB 的各种数据类型的 JSON 扩展）保存数据的“文档”。MongoDB 文档被分组在“集合”中。

传统的关系型数据库将数据分隔在表和列中，类似于电子表格。另一方面，面向文档的数据库将完整的数据对象存储在数据库的单个实例中，类似于文本文件。

虽然关系型数据库结构严格，但面向文档的数据库要灵活得多，因为开发人员可以自由使用非预定义的结构在我们的文档中，甚至可以完全改变我们的数据结构从一个文档实例到另一个文档实例。

这种灵活性和缺乏定义的结构意味着通常更容易更快地“映射”（转换）我们的对象以便将它们存储在数据库中。这为我们的项目带来了减少编码开销和更快迭代的好处。

# 关于 Mongoose 的一点说明

Mongoose 在技术上并不是 ORM（对象关系映射），尽管通常被称为是。相反，它是 ODM（对象文档映射），因为 MongoDB 本身是基于文档而不是关系表的。不过，ODM 和 ORM 的理念是相同的：提供一个易于使用的数据建模解决方案。

Mongoose 使用“模式”的概念。模式只是一个定义集合（一组文档）以及文档实例将具有的属性和允许的值类型的对象（即我们将称之为“它们的形状”）。

## Mongoose 和 Nest.js

就像我们在 TypeORM 和 Sequelize 章节中看到的一样，Nest.js 为我们提供了一个可以与 Mongoose 一起使用的模块。

# 入门

首先，我们需要安装 Mongoose npm 包，以及 Nest.js/Mongoose npm 包。

在控制台中运行`npm install --save mongoose @nestjs/mongoose`，然后立即运行`npm install --save-dev @types/mongoose`。

## 设置数据库

Docker Compose 是使用 MongoDB 最简单的方法。Docker 注册表中有一个官方的 MongoDB 镜像，我们建议您使用。目前写作本文时的最新稳定版本是`3.6.4`。

让我们创建一个 Docker Compose 文件来构建和启动我们将使用的数据库，以及我们的 Nest.js 应用，并将它们链接在一起，以便我们可以稍后从我们的代码中访问数据库。

```js
version: '3'

volumes:
  mongo_data:

services:
  mongo:
    image: mongo:latest
    ports:
    - "27017:27017"
    volumes:
    - mongo_data:/data/db
  api:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - NODE_ENV=development
    depends_on:
      - mongo
    links:
      - mongo
    environment:
      PORT: 3000
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    command: >
      npm run start:dev

```

我们指向 MongoDB 镜像的`latest`标签，这是一个解析为最新稳定版本的别名。如果您感到冒险，可以随意将标签更改为`unstable`...不过要注意可能会出现问题！

## 启动容器

现在您的 Docker Compose 文件已经准备好了，启动容器并开始工作吧！

在控制台中运行`docker-compose up`来执行。

## 连接到数据库

我们的本地 MongoDB 实例现在正在运行并准备好接受连接。

我们需要将几步前安装的 Nest.js/Mongoose 模块导入到我们的主应用模块中。

```js
import { MongooseModule } from '@nestjs/mongoose';

@Module({
  imports: [
    MongooseModule.forRoot(),
    ...
  ],
})
export class AppModule {}

```

我们将`MongooseModule`添加到`AppModule`中，并且我们依赖`forRoot()`方法来正确注入依赖项。如果您阅读了关于 TypeORM 的章节，或者熟悉 Angular 及其官方路由模块，您可能会发现`forRoot()`方法很熟悉。

上面的代码有一个*验证码*：它不起作用，因为 Mongoose 或`MongooseModule`仍然无法找出*如何*连接到我们的 MongoDB 实例。

### 连接字符串

如果你查看 Mongoose 文档或在 Google 上快速搜索，你会发现连接到 MongoDB 实例的通常方法是使用`'mongodb://localhost/test'`字符串作为 Mongoose 的`.connect()`方法的参数（甚至在 Node MongoDB 原生客户端中）。

这个字符串就是所谓的“连接字符串”。连接字符串告诉任何 MongoDB 客户端如何连接到相应的 MongoDB 实例。

坏消息是，在我们的情况下，“默认”示例连接字符串将无法工作，因为我们正在运行我们的数据库实例，它在另一个容器中链接，一个 Node.js 容器，这是我们的代码运行的容器。

然而，好消息是，我们可以使用 Docker Compose 链接来连接到我们的数据库，因为 Docker Compose 在 MongoDB 容器和 Node.js 容器之间建立了虚拟网络连接。

所以，我们唯一需要做的就是将示例连接字符串更改为

`'mongodb://mongo:27017/nest'`

其中`mongo`是我们 MongoDB 容器的名称（我们在 Docker Compose 文件中指定了这一点），`27017`是 MongoDB 容器正在暴露的端口（27017 是 MongoDB 的默认端口），`nest`是我们将在其上存储我们的文档的集合（你可以自由地将其更改为你的喜好）。

### `forRoot()`方法的正确参数

现在我们已经调整了我们的连接字符串，让我们修改我们原来的`AppModule`导入。

```js
import { MongooseModule } from '@nestjs/mongoose';

@Module({
  imports: [
    MongooseModule.forRoot('mongodb://mongo:27017/nest'),
    ...
  ],
})
export class AppModule {}

```

现在连接字符串作为参数添加到`forRoot()`方法中，因此 Mongoose 知道*如何*连接到数据库实例并且将成功启动。

# 对我们的数据进行建模

我们之前已经提到 Mongoose 使用“模式”的概念。

Mongoose 模式扮演着与 TypeORM 实体类似的角色。然而，与后者不同，前者不是类，而是从 Mongoose 定义（和导出）的`Schema`原型继承的普通对象。

无论如何，当你准备使用它们时，模式需要被实例化为“模型”。我们喜欢把模式看作对象的“蓝图”，把“模型”看作对象工厂。

## 我们的第一个模式

说到这一点，让我们创建我们的第一个实体，我们将其命名为`Entry`。我们将使用这个实体来存储我们博客的条目（帖子）。我们将在`src/entries/entry.entity.ts`创建一个新文件；这样 TypeORM 将能够找到这个实体文件，因为在我们的配置中我们指定实体文件将遵循`src/**/*.entity.ts`文件命名约定。

让我们创建我们的第一个模式。我们将把它用作存储我们博客条目的蓝图。我们还将把模式放在其他博客条目相关文件旁边，通过“域”（即功能）对我们的文件进行分组。

**注意：**你可以根据自己的喜好组织模式。我们（以及官方的 Nest.js 文档）建议将它们存储在你使用每一个模式的模块附近。无论如何，只要在需要时正确导入模式文件，你应该可以使用任何其他结构方法。

**`src/entries/entry.schema.ts`**

```js
import { Schema } from 'mongoose';

export const EntrySchema = new mongoose.Schema({
  _id: Schema.Types.ObjectId,
  title: String,
  body: String,
  image: String,
  created_at: Date,
});

```

我们刚刚编写的模式是：

1.  创建一个具有我们博客条目所需属性的对象。

1.  实例化一个新的`mongoose.Schema`类型对象。

1.  将我们的对象传递给`mongoose.Schema`类型对象的构造函数。

1.  导出实例化的`mongoose.Schema`，以便可以在其他地方使用。

**注意：**在一个名为`_id`的属性中存储我们对象的 ID，以下划线开头，这是在使用 Mongoose 时一个有用的约定；它将使得后来能够依赖于 Mongoose 的`.findById()`模型方法。

### 将模式包含到模块中

下一步是“通知”Nest.js`MongooseModule`，你打算使用我们创建的新模式。为此，我们需要创建一个“Entry”模块（如果我们还没有的话），如下所示：

**`src/entries/entries.module.ts`**

```js
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';

import { EntrySchema } from './entry.schema';

@Module({
  imports: [
    MongooseModule.forFeature([{ name: 'Entry', schema: EntrySchema }]),
  ],
})
export class EntriesModule {}

```

与我们在 TypeORM 章节中所做的非常相似，现在我们需要使用`MongooseModule`的`forFeature()`方法来定义它需要注册的模式，以便在模块范围内使用模型。

再次强调，这种方法受到 Angular 模块的影响，比如路由器，所以这可能对你来说很熟悉！

如果不是，请注意，这种处理依赖关系的方式极大地增加了应用程序中功能模块之间的解耦，使我们能够通过将模块添加或删除到主`AppModule`的导入中轻松地包含、删除和重用功能和功能。

### 将新模块包含到主模块中

另外，在谈到`AppModule`时，不要忘记将新的`EntriesModule`导入到根`AppModule`中，这样我们就可以成功地使用我们为博客编写的新功能。现在让我们来做吧！

```js
import { MongooseModule } from '@nestjs/mongoose';

import { EntriesModule } from './entries/entries.module';

@Module({
  imports: [
    MongooseModule.forRoot('mongodb://mongo:27017/nest'),
    EntriesModule,
    ...
  ],
})
export class AppModule {}

```

# 使用模式

如前所述，我们将使用我们刚刚定义的模式来实例化一个新的数据模型，我们将能够在我们的代码中使用它。 Mongoose 模型是将对象映射到数据库文档的重要工具，并且还抽象了操作数据的常见方法，比如`.find()`和`.save()`。

如果你来自 TypeORM 章节，Mongoose 中的模型与 TypeORM 中的存储库非常相似。

在必须将请求连接到数据模型时，Nest.js 中的典型方法是构建专用服务，这些服务作为与每个模型的“触点”，以及控制器。这将服务与到达 API 的请求联系起来。我们将在以下步骤中遵循`数据模型->服务->控制器`的方法。

## 接口

在创建服务和控制器之前，我们需要为我们的博客条目编写一个小接口。这是因为，如前所述，Mongoose 模式不是 TypeScript 类，因此为了正确地对对象进行类型定义以便以后使用，我们需要首先为其定义一个类型。

**`src/entries/entry.interface.ts`**

```js
import { Document } from 'mongoose';

export interface Entry extends Document {
  readonly _id: string;
  readonly title: string;
  readonly body: string;
  readonly image: string;
  readonly created_at: Date;
}

```

记住要保持接口与模式同步，这样你就不会在以后的对象形状问题上遇到问题。

## 服务

让我们为我们的博客条目创建一个服务，与`Entry`模型交互。

**`src/entries/entries.service.ts`**

```js
import { Component } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model, Types } from 'mongoose';

import { EntrySchema } from './entry.schema';
import { Entry } from './entry.interface';

@Injectable()
export class EntriesService {
  constructor(
    @InjectModel(EntrySchema) private readonly entryModel: Model<Entry>
  ) {}

  // this method retrieves all entries
  findAll() {
    return this.entryModel.find().exec();
  }

  // this method retrieves only one entry, by entry ID
  findById(id: string) {
    return this.entryModel.findById(id).exec();
  }

  // this method saves an entry in the database
  create(entry) {
    entry._id = new Types.ObjectId();
    const createdEntry = new this.entryModel(entry);
    return createdEntry.save();
  }
}

```

在上面的代码中，最重要的部分发生在构造函数内部：我们使用`@InjectModel()`装饰器来实例化我们的模型，通过将期望的模式（在本例中为`EntrySchema`）作为装饰器参数传递。

然后，在同一行代码中，我们将模型作为服务中的依赖项注入，将其命名为`entryModel`并为其分配一个`Model`类型；从这一点开始，我们可以利用 Mongoose 模型为文档进行抽象、简化的操作提供的所有好处。

另一方面，值得一提的是，在`create()`方法中，我们通过使用`_id`属性（正如我们之前在模式中定义的）向接收到的条目对象添加一个 ID，并使用 Mongoose 内置的`Types.ObjectId()`方法生成一个值。

## 控制器

我们需要覆盖模型->服务->控制器链中的最后一步。控制器将使得可以向 Nest.js 应用程序发出 API 请求，并且可以从数据库中写入或读取数据。

这就是我们的控制器应该看起来的样子：

**`src/entries/entries.controller.ts`**

```js
import { Controller, Get, Post, Body, Param } from '@nestjs/common';

import { EntriesService } from './entry.service';

@Controller('entries')
export class EntriesController {
  constructor(private readonly entriesSrv: EntriesService) {}

  @Get()
  findAll() {
    return this.entriesSrv.findAll();
  }

  @Get(':entryId')
  findById(@Param('entryId') entryId) {
    return this.entriesSrv.findById(entryId);
  }

  @Post()
  create(@Body() entry) {
    return this.entriesSrv.create(entry);
  }
}

```

像往常一样，我们正在使用 Nest.js 依赖注入，使`EntryService`在我们的`EntryController`中可用。然后，我们将我们期望监听的三个基本请求（`GET`所有条目，`GET`按 ID 获取一个条目和`POST`一个新条目）路由到我们服务中的相应方法。

# 第一个请求

此时，我们的 Nest.js API 已经准备好接收请求（包括`GET`和`POST`），并根据这些请求在我们的 MongoDB 实例中操作数据。换句话说，我们已经准备好从 API 中读取并向数据库写入数据。

让我们试一试。

我们将从对`/entries`端点的 GET 请求开始。显然，由于我们还没有创建任何条目，所以我们应该收到一个空数组作为响应。

```js
> GET /entries HTTP/1.1
> Host: localhost:3000
< HTTP/1.1 200 OK

[]

```

让我们通过向`entries`端点发送`POST`请求并在请求体中包含一个与我们之前定义的`EntrySchema`形状匹配的 JSON 对象来创建一个新条目。

```js
> GET /entries HTTP/1.1
> Host: localhost:3000
| {
|   "title": "This is our first post",
|   "body": "Bla bla bla bla bla",
|   "image": "http://lorempixel.com/400",
|   "created_at": "2018-04-15T17:42:13.911Z"
| }

< HTTP/1.1 201 Created

```

是的！我们之前的`POST`请求触发了数据库中的写入。让我们再次尝试检索所有条目。

```js
> GET /entries HTTP/1.1
> Host: localhost:3000
< HTTP/1.1 200 OK

[{
  "id": 1,
  "title": "This is our first post",
  "body": "Bla bla bla bla bla",
  "image": "http://lorempixel.com/400",
  "created_at": "2018-04-15T17:42:13.911Z"
}]

```

我们刚刚确认对我们的`/entries`端点的请求成功执行了数据库中的读写操作。这意味着我们的 Nest.js 应用现在可以使用，因为几乎任何服务器应用程序的基本功能（即存储数据并根据需要检索数据）都正常工作。

# 关系

虽然 MongoDB 不是关系数据库，但它允许进行“类似于连接”的操作，以一次检索两个（或更多）相关文档。

幸运的是，Mongoose 包含了一层抽象，允许我们以清晰、简洁的方式在对象之间建立关系。这是通过在模式属性中使用`ref`以及`.populate()`方法（触发所谓的“填充”过程的方法）来实现的；稍后会详细介绍。

## 建模关系

让我们回到我们的博客示例。记住到目前为止我们只有一个定义博客条目的模式。我们将创建一个第二个模式，它将允许我们为每个博客条目创建评论，并以一种允许我们稍后检索博客条目以及属于它的评论的方式保存到数据库中，所有这些都可以在单个数据库操作中完成。

因此，首先，我们创建一个像下面这样的`CommentSchema`：

**`src/comments/comment.schema.ts`**

```js
import * as mongoose from 'mongoose';

export const CommentSchema = new mongoose.Schema({
  _id: Schema.Types.ObjectId,
  body: String,
  created_at: Date,
  entry: { type: Schema.Types.ObjectId, ref: 'Entry' },
});

```

在这一点上，这个模式是我们之前的`EntrySchema`的“精简版本”。实际上，它是由预期的功能决定的，所以我们不应该太在意这个事实。

再次，我们依赖于名为`_id`的属性作为命名约定。

一个值得注意的新东西是`entry`属性。它将用于存储每个评论所属的条目的引用。`ref`选项告诉 Mongoose 在填充期间使用哪个模型，我们的情况下是`Entry`模型。我们在这里存储的所有`_id`都需要是 Entry 模型的文档`_id`。

**注意：** 为了简洁起见，我们将忽略`Comment`接口；这对你来说应该足够简单。不要忘记完成它！

其次，我们需要更新我们原始的`EntrySchema`，以便允许我们保存属于每个条目的`Comment`实例的引用。看下面的示例如何做到这一点：

**`src/entries/entry.schema.ts`**

```js
import * as mongoose from 'mongoose';

export const EntrySchema = new mongoose.Schema({
  _id: Schema.Types.ObjectId,
  title: String,
  body: String,
  image: String,
  created_at: Date,
  comments: [{ type: Schema.Types.ObjectId, ref: 'Comment' }],
});

```

请注意，我们刚刚添加的`comments`属性是*对象数组*，每个对象都有一个 ObjectId 以及一个引用。关键在于包含*相关对象*的数组，因为这个数组使我们可以称之为“一对多”关系，如果我们处于关系数据库的上下文中。

换句话说，每个条目可以有多个评论，但每个评论只能属于一个条目。

## 保存关系

一旦我们的关系被建模，我们需要提供一个方法将它们保存到我们的 MongoDB 实例中。

在使用 Mongoose 时，存储模型实例及其相关实例需要一定程度的手动嵌套方法。幸运的是，`async/await`将使任务变得更加容易。

让我们修改我们的`EntryService`，以保存接收到的博客条目和与之关联的评论；两者将作为不同的对象发送到`POST`端点。

**`src/entries/entries.service.ts`**

```js
import { Component } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model, Types } from 'mongoose';

import { EntrySchema } from './entry.schema';
import { Entry } from './entry.interface';

import { CommentSchema } from './comment.schema';
import { Comment } from './comment.interface';

@Injectable()
export class EntriesService {
  constructor(
    @InjectModel(EntrySchema) private readonly entryModel: Model<Entry>,
    @InjectModel(CommentSchema) private readonly commentModel: Model<Comment>
  ) {}

  // this method retrieves all entries
  findAll() {
    return this.entryModel.find().exec();
  }

  // this method retrieves only one entry, by entry ID
  findById(id: string) {
    return this.entryModel.findById(id).exec();
  }

  // this method saves an entry and a related comment in the database
  async create(input) {
    const { entry, comment } = input;

    // let's first take care of the entry (the owner of the relationship)
    entry._id = new Types.ObjectId();
    const entryToSave = new this.entryModel(entry);
    await entryToSave.save();

    // now we are ready to handle the comment
    // this is how we store in the comment the reference
    // to the entry it belongs to
    comment.entry = entryToSave._id;

    comment._id = new Types.ObjectId();
    const commentToSave = new this.commentModel(comment);
    commentToSave.save();

    return { success: true };
  }
}

```

修改后的`create()`方法现在是：

1.  为条目分配一个 ID。

1.  将条目保存并分配给`const`。

1.  为评论分配一个 ID。

1.  使用我们之前创建的条目的 ID 作为评论的`entry`属性的值。*这是我们之前提到的引用。*

1.  保存评论。

1.  返回成功状态消息。

通过这种方式，我们确保在评论中成功存储了对评论所属的条目的引用。顺便说一句，注意我们通过条目的 ID 来存储引用。

显然，下一步应该是提供一种从数据库中读取我们现在能够保存到其中的相关项目的方法。

## 阅读关系

正如前面几节所介绍的，Mongoose 提供的从数据库一次检索相关文档的方法称为“population”，并且通过内置的`.populate()`方法调用它。

我们将看到如何通过再次更改`EntryService`来使用这种方法；在这一点上，我们将处理`findById()`方法。

**`src/entries/entries.service.ts`**

```js
import { Component } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model, Types } from 'mongoose';

import { EntrySchema } from './entry.schema';
import { Entry } from './entry.interface';

import { CommentSchema } from './comment.schema';
import { Comment } from './comment.interface';

@Injectable()
export class EntriesService {
  constructor(
    @InjectModel(EntrySchema) private readonly entryModel: Model<Entry>,
    @InjectModel(CommentSchema) private readonly commentModel: Model<Comment>
  ) {}

  // this method retrieves all entries
  findAll() {
    return this.entryModel.find().exec();
  }

  // this method retrieves only one entry, by entry ID,
  // including its related documents with the "comments" reference
  findById(id: string) {
    return this.entryModel
      .findById(id)
      .populate('comments')
      .exec();
  }

  // this method saves an entry and a related comment in the database
  async create(input) {
    ...
  }
}

```

我们刚刚包含的`.populate('comments')`方法将把`comments`属性值从 ID 数组转换为与这些 ID 对应的实际文档数组。换句话说，它们的 ID 值被替换为通过执行单独查询从数据库返回的 Mongoose 文档。

# 摘要

NoSQL 数据库是“传统”关系数据库的一个强大替代品。MongoDB 可以说是当今使用的 NoSQL 数据库中最知名的，它使用 JSON 变体编码的文档。使用诸如 MongoDB 之类的基于文档的数据库允许开发人员使用更灵活、松散结构的数据模型，并可以提高在快速移动的项目中的迭代时间。

著名的 Mongoose 库是一个适配器，用于在 Node.js 中使用 MongoDB，并在查询和保存操作时抽象了相当多的复杂性。

在本章中，我们涵盖了与 Mongoose 和 Nest.js 一起工作的许多方面，比如：

+   如何使用 Docker Compose 启动本地 MongoDB 实例。

+   如何在我们的根模块中导入@nestjs/mongoose 模块并连接到我们的 MongoDb 实例。

+   什么是模式，以及如何为建模我们的数据创建一个模式。

+   建立一个管道，使我们能够以对 Nest.js 端点发出的请求的反应来写入和读取我们的 MongoDB 数据库。

+   如何在不同类型的 MongoDB 文档之间建立关系，以及如何以有效的方式存储和检索这些关系。

在下一章中，我们将介绍 Web 套接字。
