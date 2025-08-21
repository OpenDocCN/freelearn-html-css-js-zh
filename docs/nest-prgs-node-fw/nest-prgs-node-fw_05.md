# 第五章：TypeORM

几乎每次在现实世界中使用 Nest.js 时，您都需要某种持久性来保存数据。也就是说，您需要将 Nest.js 应用程序接收到的数据保存在某个地方，并且您需要从某个地方读取数据，以便随后将该数据作为响应传递给 Nest.js 应用程序接收到的请求。

大多数情况下，“某个地方”将是一个数据库。

TypeORM 是一个与多种不同关系数据库一起工作的对象关系映射（ORM）。对象关系映射是一个工具，用于在对象（例如“Entry”或“Comment”，因为我们正在构建一个博客）和数据库中的表之间进行转换。

这种转换的结果是一个实体（称为数据传输对象），它知道如何从数据库中读取数据到内存（这样您就可以将数据作为请求的响应使用），以及如何从内存写入数据库（这样您就能够存储数据以备后用）。

TypeORM 在概念上类似于 Sequelize。TypeORM 也是用 TypeScript 编写的，并且广泛使用装饰器，因此它非常适合 Nest.js 项目。

我们显然将专注于将 TypeORM 与 Nest.js 一起使用，但 TypeORM 也可以在浏览器和服务器端使用，使用传统的 JavaScript 以及 TypeScript。

TypeORM 允许您同时使用数据映射器模式和活动记录模式。我们将专注于活动记录模式，因为它大大减少了在典型 Nest.js 架构上使用所需的样板代码量，就像本书中所解释的那样。

TypeORM 也可以与 MongoDB 一起工作，不过在这种情况下，使用专门的 NoSQL ORM，如 Mongoose，是更常见的方法。

# 使用哪种数据库

TypeORM 支持以下数据库：

+   MySQL

+   MariaDB

+   PostgreSQL

+   MS SQL Server

+   sql.js

+   MongoDB

+   Oracle（实验性）

考虑到在本书中我们已经使用 Sequelize 和 Mongoose 分别使用 PostgreSQL 和 MongoDB，我们决定使用 TypeORM 与 MariaDB。 

## 关于 MariaDB

MariaDB 是一个由 MySQL 的一些原始开发人员领导的开源、社区驱动的项目。它是从 Oracle 收购后保持其自由和开放性的 GNU 通用公共许可证下的 MySQL 分支。

该项目的最初想法是作为 MySQL 的一个可替代品。这在 5.5 版本之前基本上是正确的，而 MariaDB 保持了与 MySQL 相同的版本号。

尽管如此，从 10.0 版本开始，较新的版本略微偏离了这种方法。不过，MariaDB 仍然专注于与 MySQL 高度兼容，并共享相同的 API。

# 入门

当然，TypeORM 作为一个 npm 包进行分发。您需要运行`npm install typeorm @nestjs/typeorm`。

您还需要一个 TypeORM 数据库驱动程序；在这种情况下，我们将使用`npm install mysql`安装 MySQL/MariaDB。

TypeORM 还依赖于`reflect-metadata`，但幸运的是，我们之前已经安装了它，因为 Nest.js 也依赖于它，所以我们无需做其他事情。请记住，如果您在 Nest.js 上下文之外使用 TypeORM，您还需要安装这个依赖。

**注意：** 如果您还没有安装 Node.js，现在安装是一个好主意：`npm install --save-dev @types/node`。

## 启动数据库

为了连接到数据库，我们将使用 Docker Compose，使用官方的 MariaDB Docker 镜像来设置我们的本地开发环境。我们将指向`latest` Docker 镜像标签，这在撰写本文时对应于版本 10.2.14。

```js
version: '3'

volumes:
  # for persistence between restarts
  mariadb_data:

services:
  mariadb:
    image: mariadb:latest
    restart: always
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: nestbook
      MYSQL_USER: nest
      MYSQL_PASSWORD: nest
    volumes:
        - mariadb_data:/var/lib/mysql

  api:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - NODE_ENV=development
    depends_on:
      - mariadb
    links:
      - mariadb
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

## 连接到数据库

现在我们有了一个连接 TypeORM 的数据库，让我们配置连接。

我们有几种配置 TypeORM 的方式。最直接的一种是在项目根文件夹中创建一个`ormconfig.json`文件，这对于入门非常有用。这个文件将在启动时被 TypeORM 自动抓取。

这是一个适合我们用例的示例配置文件（即使用 Docker Compose 与之前提出的配置）。

**ormconfig.json**

```js
{
  "type": "mariadb",
  "host": "mariadb",
  "port": 3306,
  "username": "nest",
  "password": "nest",
  "database": "nestbook",
  "synchronize": true,
  "entities": ["src/**/*.entity.ts"]
}

```

关于配置文件的一些说明：

+   属性`host`、`port`、`username`、`password`和`database`需要与`docker-compose.yml`文件中之前指定的属性匹配；否则，TypeORM 将无法连接到 MariaDB Docker 镜像。

+   `synchronize`属性告诉 TypeORM 在应用程序启动时是否创建或更新数据库模式，以便模式与代码中声明的实体匹配。将此属性设置为`true`很容易导致数据丢失，所以**在启用此属性之前，请确保你知道你在做什么**。

## 初始化 TypeORM

现在数据库正在运行，并且你能够成功地建立起它与我们的 Nest.js 应用之间的连接，我们需要指示 Nest.js 使用 TypeORM 作为一个模块。

由于我们之前安装的`@nest/typeorm`包，所以在我们的 Nest.js 应用程序中使用 TypeORM 就像在主应用程序模块（可能是`app.module.ts`文件）中导入`TypeOrmModule`一样简单。

```js
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot(),
    ...
  ]
})

export class AppModule {}

```

# 建模我们的数据

使用 ORM 最好的一点可能是，你可以利用它们提供的建模抽象：基本上，它们允许我们思考我们的数据，并用属性（包括类型和关系）来塑造它，生成我们可以直接使用和操作的“对象类型”（并将它们连接到数据库表）。

这个抽象层可以让你摆脱编写特定于数据库的代码，比如查询、连接等。很多人喜欢不必为选择和类似的事情而苦苦挣扎；所以这个抽象层非常方便。

## 我们的第一个实体

在使用 TypeORM 时，这些对象抽象被称为*实体*。

实体基本上是映射到数据库表的类。

说到这里，让我们创建我们的第一个实体，我们将其命名为`Entry`。我们将使用这个实体来存储博客的条目（帖子）。我们将在`src/entries/entry.entity.ts`创建一个新文件；这样 TypeORM 就能够找到这个实体文件，因为在我们的配置中我们指定了实体文件将遵循`src/**/*.entity.ts`文件命名约定。

```js
import { Entity } from 'typeorm';

@Entity()
export class Entry {}

```

`@Entity()`装饰器来自`typeorm` npm 包，用于标记`Entry`类为一个实体。这样，TypeORM 就会知道它需要在我们的数据库中为这种对象创建一个表。

`Entry`实体还有点太简单了：我们还没有为它定义一个属性。我们可能需要像标题、正文、图片和日期这样的东西来记录博客条目，对吧？让我们来做吧！

```js
import { Entity, Column } from 'typeorm';

@Entity()
export class Entry {
  @Column() title: string;

  @Column() body: string;

  @Column() image: string;

  @Column() created_at: Date;
}

```

不错！我们为实体定义的每个属性都标有`@Column`装饰器。再次，这个装饰器告诉 TypeORM 如何处理属性：在这种情况下，我们要求每个属性都存储在数据库的一列中。

遗憾的是，这个实体将无法使用这段代码。这是因为每个实体都需要至少一个主列，而我们没有将任何列标记为主列。

我们最好为每个条目创建一个`id`属性，并将其存储在主列上。

```js
import { Entity, Column, PrimaryColumn } from 'typeorm';

@Entity()
export class Entry {
  @PrimaryColumn() id: number;

  @Column() title: string;

  @Column() body: string;

  @Column() image: string;

  @Column() created_at: Date;
}

```

啊，好多了！我们的第一个实体现在可以工作了。让我们来使用它！

# 使用我们的模型

当需要将请求连接到数据模型时，在 Nest.js 中的典型方法是构建专门的服务，这些服务作为与每个模型的“接触点”，并构建控制器，将服务与到达 API 的请求连接起来。让我们在以下步骤中遵循`模型 -> 服务 -> 控制器`的方法。

## 服务

在典型的 Nest.js 架构中，应用程序的重要工作是由服务完成的。为了遵循这种模式，创建一个新的`EntriesService`，用它来与`Entry`实体交互。

所以，让我们在这里创建一个新文件：**`src/entries/entries.service.ts`**

```js
import { Component } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

import { Entry } from './entry.entity';

@Injectable()
export class EntriesService {
  constructor(
    // we create a repository for the Entry entity
    // and then we inject it as a dependency in the service
    @InjectRepository(Entry) private readonly entry: Repository<Entry>
  ) {}

  // this method retrieves all entries
  findAll() {
    return this.entry.find();
  }

  // this method retrieves only one entry, by entry ID
  findOneById(id: number) {
    return this.entry.findOneById(id);
  }

  // this method saves an entry in the database
  create(newEntry: Entry) {
    this.entry.save(newEntry);
  }
}

```

服务的最重要部分是使用`Repository<Entry>`创建 TypeORM 存储库，然后在构造函数中使用`@InjectRepository(Entry)`进行注入。

顺便说一句，如果你在想，当处理 ORM 时，存储库可能是最常用的设计模式，因为它允许你将数据库操作抽象为对象集合。

回到最新的服务代码，一旦你创建并注入了 Entry 存储库，就可以使用它从数据库中`.find()`和`.save()`条目，以及其他操作。当我们为实体创建存储库时，这些有用的方法会被添加进来。

既然我们已经处理了数据模型和服务，现在让我们为最后一个链接编写代码：控制器。

## 控制器

让我们为 Entry 模型创建一个控制器，通过 RESTful API 将其暴露给外部世界。代码非常简单，你可以看到。

继续，在以下位置创建一个新文件：**`src/entries/entries.controller.ts`**

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
  findOneById(@Param('entryId') entryId) {
    return this.entriesSrv.findOneById(entryId);
  }

  @Post()
  create(@Body() entry) {
    return this.entriesSrv.create(entry);
  }
}

```

和往常一样，我们使用 Nest.js 依赖注入使`EntryService`在`EntryController`中可用。

## 构建一个新的模块

我们新实体端点工作的最后一步是在应用模块中包含实体、服务和控制器。我们不会直接这样做，而是遵循“分离模块”的方法，为我们的条目创建一个新模块，在那里导入所有必要的部分，然后在应用模块中导入整个模块。

因此，让我们创建一个名为：**`src/entries/entries.module.ts`**的新文件。

```js
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

import { Entry } from './entry.entity';
import { EntriesController } from './entry.controller';
import { EntriesService } from './entry.service';

@Module({
  imports: [TypeOrmModule.forFeature([Entry])],
  controllers: [EntriesController],
  components: [EntriesService],
})
export class EntriesModule {}

```

还记得当我们在本章的最初步骤中在`AppModule`中包含了`TypeOrmModule`吗？我们在那里使用了`TypeOrmModule.forRoot()`公式。然而，在这里我们使用了不同的公式：`TypeOrmModule.forFeature()`。

Nest.js TypeORM 实现中的这种区别允许我们在不同的模块中分离不同的功能（“特性”）。这样你就可以根据本书的架构章节中提出的一些想法和最佳实践来调整你的代码。

无论如何，让我们将新的`EntriesModule`导入到`AppModule`中。如果忽略了这一步，你的主应用模块将不会意识到`EntriesModule`的存在，你的应用将无法正常工作。

**`src/app.module.ts`**

```js
import { TypeOrmModule } from '@nestjs/typeorm';
import { EntriesModule } from './entries/entries.module';

@Module({
  imports: [
    TypeOrmModule.forRoot(),
    EntriesModule,
    ...
  ]
})

export class AppModule {}

```

就是这样！现在你可以向`/entities`发送请求，端点将调用数据库的写入和读取操作。

是时候让我们的数据库试试了！我们将向之前链接到数据库的端点发送一些请求，看看是否一切都按预期工作。

我们将从向`/entries`端点发送 GET 请求开始。显然，由于我们还没有创建任何条目，我们应该收到一个空数组作为响应。

```js
> GET /entries HTTP/1.1
> Host: localhost:3000
< HTTP/1.1 200 OK

[]

```

让我们创建一个新的条目。

```js
> GET /entries HTTP/1.1
> Host: localhost:3000
| {
|   "id": 1,
|   "title": "This is our first post",
|   "body": "Bla bla bla bla bla",
|   "image": "http://lorempixel.com/400",
|   "created_at": "2018-04-15T17:42:13.911Z"
| }

< HTTP/1.1 201 Created

```

成功！让我们通过 ID 检索新条目。

```js
> GET /entries/1 HTTP/1.1
> Host: localhost:3000
< HTTP/1.1 200 OK

{
  "id": 1,
  "title": "This is our first post",
  "body": "Bla bla bla bla bla",
  "image": "http://lorempixel.com/400",
  "created_at": "2018-04-15T17:42:13.911Z"
}

```

是的！我们之前的 POST 请求触发了数据库中的写入，现在这个最后的 GET 请求触发了对数据库的读取，并返回先前保存的数据！

现在让我们再次尝试检索所有条目。

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

我们刚刚确认，对`/entries`端点的请求成功执行了数据库的读写操作。这意味着我们的 Nest.js 应用现在可以使用，因为几乎任何服务器应用程序的基本功能（即存储数据并根据需要检索数据）都正常工作。

# 改进我们的模型

尽管我们现在通过实体从数据库中读取和写入数据，但我们只编写了一个基本的初始实现；我们应该审查我们的代码，看看有什么可以改进的地方。

现在让我们回到实体文件`src/entries/entry.entity.ts`，看看我们可以做出什么样的改进。

## 自动生成的 ID

所有的数据库条目都需要有一个唯一的 ID。目前，我们只是依赖于客户端在创建实体时（发送 POST 请求时）发送的 ID，但这并不理想。

任何服务器端应用程序都将连接到多个客户端，所有这些客户端都无法知道哪些 ID 已经在使用，因此他们无法生成并发送每个 POST 请求的唯一 ID。

TypeORM 提供了几种为实体生成唯一 ID 的方法。第一种是使用`@PrimaryGeneratedColumn()`装饰器。通过使用它，您不再需要在 POST 请求的主体中包含 ID，也不需要在保存条目之前手动生成 ID。相反，每当您要求将新条目保存到数据库时，TypeORM 会自动为其生成 ID。

我们的代码看起来像下面这样：

```js
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Entry {
  @PrimaryGeneratedColumn() id: number;

  @Column() title: string;

  @Column() body: string;

  @Column() image: string;

  @Column() created_at: Date;
}

```

值得一提的是，这些唯一的 ID 将以顺序方式生成，这意味着每个 ID 将比数据库中已有的最高 ID 高一个数字（生成新 ID 的确切方法将取决于数据库类型）。

TypeORM 还可以更进一步：如果将`"uuid"`参数传递给`@PrimaryGeneratedColumn()`装饰器，生成的值将看起来像一串随机的字母和数字，带有一些破折号，确保它们是唯一的（至少*相对*唯一）。

```js
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Entry {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column() title: string;

  @Column() body: string;

  @Column() image: string;

  @Column() created_at: Date;
}

```

还要记得将`id`的类型从`number`更改为`string`！

## 条目是何时创建的？

在原始实体定义中，还预期从客户端接收`created_at`字段。然而，我们可以通过一些更多的 TypeORM 魔术装饰器轻松改进这一点。

让我们使用`@CreateDateColumn()`装饰器为每个条目动态生成插入日期。换句话说，您不需要在保存条目之前从客户端设置日期或手动创建日期。

让我们更新实体：

```js
import {
  Entity,
  Column,
  CreateDateColumn,
  PrimaryGeneratedColumn,
} from 'typeorm';

@Entity()
export class Entry {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column() title: string;

  @Column() body: string;

  @Column() image: string;

  @CreateDateColumn() created_at: Date;
}

```

不错，是吗？还想知道条目上次修改是什么时候，以及对其进行了多少次修订？同样，TypeORM 使这两者都很容易实现，并且不需要我们额外的代码。

```js
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn,
  VersionColumn,
} from 'typeorm';

@Entity()
export class Entry {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column() title: string;

  @Column() body: string;

  @Column() image: string;

  @CreateDateColumn() created_at: Date;

  @UpdateDateColumn() modified_at: Date;

  @VersionColumn() revision: number;
}

```

我们的实体现在将自动为我们处理修改日期，以及每次保存操作时的修订号。您可以跟踪对实体的每个实例所做的更改，而无需实现一行代码！

## 列类型

在我们的实体中使用装饰器定义列时，如上所述，TypeORM 将从使用的属性类型推断数据库列的类型。这基本上意味着当 TypeORM 找到以下行时

```js
@Column() title: string;

```

这将`string`属性类型映射到`varchar`数据库列类型。

这通常会很好地工作，但在某些情况下，我们可能需要更明确地指定要在数据库中创建的列的类型。幸运的是，TypeORM 允许使用非常少的开销来实现这种自定义行为。

要自定义列类型，请将所需类型作为字符串参数传递给`@Column()`装饰器。一个具体的例子是：

```js
@Column('text') body: string;

```

可以使用的确切列类型取决于您使用的数据库类型。

### `mysql` / `mariadb`的列类型

`int`、`tinyint`、`smallint`、`mediumint`、`bigint`、`float`、`double`、`dec`、`decimal`、`numeric`、`date`、`datetime`、`timestamp`、`time`、`year`、`char`、`varchar`、`nvarchar`、`text`、`tinytext`、`mediumtext`、`blob`、`longtext`、`tinyblob`、`mediumblob`、`longblob`、`enum`、`json`、`binary`、`geometry`、`point`、`linestring`、`polygon`、`multipoint`、`multilinestring`、`multipolygon`、`geometrycollection`

### `postgres`的列类型

`int`、`int2`、`int4`、`int8`、`smallint`、`integer`、`bigint`、`decimal`、`numeric`、`real`、`float`、`float4`、`float8`、`double precision`、`money`、`character varying`、`varchar`、`character`、`char`、`text`、`citext`、`hstore`、`bytea`、`bit`、`varbit`、`bit varying`、`timetz`、`timestamptz`、`timestamp`、`timestamp without time zone`、`timestamp with time zone`、`date`、`time`、`time without time zone`、`time with time zone`、`interval`、`bool`、`boolean`、`enum`、`point`、`line`、`lseg`、`box`、`path`、`polygon`、`circle`、`cidr`、`inet`、`macaddr`、`tsvector`、`tsquery`、`uuid`、`xml`、`json`、`jsonb`、`int4range`、`int8range`、`numrange`、`tsrange`、`tstzrange`、`daterange`

### `sqlite` / `cordova` / `react-native`的列类型

`int`、`int2`、`int8`、`integer`、`tinyint`、`smallint`、`mediumint`、`bigint`、`decimal`、`numeric`、`float`、`double`、`real`、`double precision`、`datetime`、`varying character`、`character`、`native character`、`varchar`、`nchar`、`nvarchar2`、`unsigned big int`、`boolean`、`blob`、`text`、`clob`、`date`

### `mssql`的列类型

`int`、`bigint`、`bit`、`decimal`、`money`、`numeric`、`smallint`、`smallmoney`、`tinyint`、`float`、`real`、`date`、`datetime2`、`datetime`、`datetimeoffset`、`smalldatetime`、`time`、`char`、`varchar`、`text`、`nchar`、`nvarchar`、`ntext`、`binary`、`image`、`varbinary`、`hierarchyid`、`sql_variant`、`timestamp`、`uniqueidentifier`、`xml`、`geometry`、`geography`

### `oracle`的列类型

`char`、`nchar`、`nvarchar2`、`varchar2`、`long`、`raw`、`long raw`、`number`、`numeric`、`float`、`dec`、`decimal`、`integer`、`int`、`smallint`、`real`、`double precision`、`date`、`timestamp`、`timestamp with time zone`、`timestamp with local time zone`、`interval year to month`、`interval day to second`、`bfile`、`blob`、`clob`、`nclob`、`rowid`、`urowid`

如果你还没有准备好承诺使用特定的数据库类型，并且希望为将来保持选择的开放性，那么使用不是每个数据库都可用的类型可能不是最好的主意。

## SQL 中的 NoSQL

TypeORM 还有一个最后的绝招：`simple-json`列类型，可以在每个支持的数据库中使用。使用它，你可以直接在关系数据库列中保存普通的 JavaScript 对象。是的，令人惊叹！

让我们在实体中使用一个新的`author`属性。

```js
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn,
  VersionColumn,
} from 'typeorm';

@Entity()
export class Entry {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column() title: string;

  @Column('text') body: string;

  @Column() image: string;

  @Column('simple-json') author: { first_name: string; last_name: string };

  @CreateDateColumn() created_at: Date;

  @UpdateDateColumn() modified_at: Date;

  @VersionColumn() revision: number;
}

```

`simple-json`列类型允许您直接存储甚至复杂的 JSON 树，而无需首先定义一个模型。在您欣赏比传统的关系数据库结构更灵活的情况下，这可能会派上用场。

# 数据模型之间的关系

如果您一直跟着本章节，那么您将有一种通过 API 将新的博客条目保存到数据库中，然后再读取它们的方法。

接下来是创建第二个实体来处理每个博客条目中的评论，然后以这样的方式创建条目和评论之间的关系，以便一个博客条目可以有属于它的多个评论。

然后创建`Comments`实体。

**`src/comments/comment.entity.ts`**

```js
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn,
  VersionColumn,
} from 'typeorm';

@Entity()
export class Comment {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column('text') body: string;

  @Column('simple-json') author: { first_name: string; last_name: string };

  @CreateDateColumn() created_at: Date;

  @UpdateDateColumn() modified_at: Date;

  @VersionColumn() revision: number;
}

```

您可能已经注意到`Comment`实体与`Entry`实体非常相似。

接下来的步骤将是在条目和评论之间创建一个“一对多”的关系。为此，在`Entry`实体中包含一个新的属性，使用`@OneToMany()`装饰器。

**`src/entries/entry.entity.ts`**

```js
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn,
  VersionColumn,
  OneToMany,
} from 'typeorm';

import { Comment } from '../comments/comment.entity';

@Entity()
export class Entry {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column() title: string;

  @Column('text') body: string;

  @Column() image: string;

  @Column('simple-json') author: { first_name: string; last_name: string };

  @OneToMany(type => Comment, comment => comment.id)
  comments: Comment[];

  @CreateDateColumn() created_at: Date;

  @UpdateDateColumn() modified_at: Date;

  @VersionColumn() revision: number;
}

```

“一对多”关系必须是双向的，因此您需要在`Comment`实体中添加一个反向关系“多对一”。这样，两者都将得到适当的“绑定”。

**`src/comments/comment.entity.ts`**

```js
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn,
  VersionColumn,
  ManyToOne,
} from 'typeorm';

import { Entry } from '../entries/entry.entity';

@Entity()
export class Comment {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column('text') body: string;

  @Column('simple-json') author: { first_name: string; last_name: string };

  @ManyToOne(type => Entry, entry => entry.comments)
  entry: Entry;

  @CreateDateColumn() created_at: Date;

  @UpdateDateColumn() modified_at: Date;

  @VersionColumn() revision: number;
}

```

我们传递给`@OneToMany()`和`@ManyToOne()`装饰器的第二个参数用于指定我们在另一个相关实体上创建的逆关系。换句话说，在`Entry`中，我们将相关的`Comment`实体保存在名为`comments`的属性中。这就是为什么在`Comment`实体定义中，我们将`entry => entry.comments`作为第二个参数传递给装饰器的原因，直到在`Entry`中存储评论。

**注意：**并非所有关系*需要*是双向的。“一对一”关系可以是单向的或双向的。在单向“一对一”关系的情况下，关系的所有者是声明它的一方，而另一个实体不需要知道关于第一个实体的任何信息。

就是这样！现在我们的每个条目都可以有多条评论。

## 如何存储相关实体

如果我们谈论代码，保存属于条目的评论的最直接的方法将是保存评论，然后保存包含新评论的条目。创建一个新的`Comments`服务来与实体交互，然后修改`Entry`控制器以调用该新的`Comments`服务。

让我们看看。这并不像听起来那么难！

这将是我们的新服务：

**`src/comments/comments.service.ts`**

```js
import { Component } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

import { Comment } from './comment.entity';

@Injectable()
export class CommentsService {
  constructor(
    @InjectRepository(Comment) private readonly comment: Repository<Comment>
  ) {}

  findAll() {
    return this.comment.find();
  }

  findOneById(id: number) {
    return this.comment.findOneById(id);
  }

  create(comment: Comment) {
    return this.comment.save(comment);
  }
}

```

代码看起来确实很熟悉，不是吗？这与我们已经拥有的`EntriesService`非常相似，因为我们为评论和条目提供了相同的功能。

这将是修改后的`Entries`控制器：

**`src/entries/entries.controller.ts`**

```js
import { Controller, Get, Post, Body, Param } from '@nestjs/common';

import { EntriesService } from './entries.service';
import { CommentsService } from '../comments/comments.service';

import { Entry } from './entry.entity';
import { Comment } from '../comments/comment.entity';

@Controller('entries')
export class EntriesController {
  constructor(
    private readonly entriesSrv: EntriesService,
    private readonly commentsSrv: CommentsService
  ) {}

  @Get()
  findAll() {
    return this.entriesSrv.findAll();
  }

  @Get(':entryId')
  findOneById(@Param('entryId') entryId) {
    return this.entriesSrv.findOneById(entryId);
  }

  @Post()
  async create(@Body() input: { entry: Entry; comments: Comment[] }) {
    const { entry, comments } = input;
    entry.comments: Comment[] = [];
    await comments.forEach(async comment => {
      await this.commentsSrv.create(comment);
      entry.comments.push(comment);
    });
    return this.entriesSrv.create(entry);
  }
}

```

简而言之，新的`create()`方法：

+   接收一个博客条目和属于该条目的评论数组。

+   在博客条目对象内创建一个新的空数组属性（名为`comments`）。

+   遍历接收到的评论，保存每一条评论，然后逐一将它们推送到`entry`的新`comments`属性中。

+   最后，保存了现在包含每条评论链接的`entry`。

### 以更简单的方式保存相关实体

我们上次编写的代码有效，但不太方便。

幸运的是，TypeORM 为我们提供了一种更简单的方法来保存相关实体：启用“级联”。

在实体中将`cascade`设置为`true`将意味着我们将不再需要单独保存每个相关实体；相反，将关系的所有者保存到数据库将同时保存这些相关实体。这样，我们以前的代码可以简化。

首先，让我们修改我们的`Entry`实体（它是关系的所有者）以启用级联。

**`src/entries/entry.entity.ts`**

```js
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn,
  VersionColumn,
  OneToMany,
} from 'typeorm';

import { Comment } from '../comments/comment.entity';

@Entity()
export class Entry {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column() title: string;

  @Column('text') body: string;

  @Column() image: string;

  @Column('simple-json') author: { first_name: string; last_name: string };

  @OneToMany(type => Comment, comment => comment.id, {
    cascade: true,
  })
  comments: Comment[];

  @CreateDateColumn() created_at: Date;

  @UpdateDateColumn() modified_at: Date;

  @VersionColumn() revision: number;
}

```

这真的很简单：我们只需为`@OneToMany()`装饰器的第三个参数添加一个`{cascade: true}`对象。

现在，我们将重构`Entries`控制器上的`create()`方法。

**`src/entries/entries.controller.ts`**

```js
import { Controller, Get, Post, Body, Param } from '@nestjs/common';

import { EntriesService } from './entries.service';

import { Entry } from './entry.entity';
import { Comment } from '../comments/comment.entity';

@Controller('entries')
export class EntriesController {
  constructor(private readonly entriesSrv: EntriesService) {}

  @Get()
  findAll() {
    return this.entriesSrv.findAll();
  }

  @Get(':entryId')
  findAll(@Param('entryId') entryId) {
    return this.entriesSrv.findOneById(entryId);
  }

  @Post()
  async create(@Body() input: { entry: Entry; comments: Comment[] }) {
    const { entry, comments } = input;
    entry.comments = comments;
    return this.entriesSrv.create(entry);
  }
}

```

请将新控制器与我们以前的实现进行比较；我们已经摆脱了对`Comments`服务的依赖，以及对`create()`方法的迭代器。这使我们的代码更短，更清晰，这总是好的，因为它减少了引入错误的风险。

在这一部分，我们发现了如何保存彼此相关的实体，同时保存它们的关系。这对于我们相关实体的成功至关重要。干得好！

## 批量检索相关实体

现在我们知道如何保存一个实体并包含它的关系，我们将看看如何从数据库中读取一个实体以及它们的所有相关实体。

在这种情况下的想法是，当我们从数据库请求博客条目（只有一个）时，我们还会得到属于它的评论。

当然，由于你对博客一般情况比较熟悉（它们已经存在一段时间了，对吧？），你会意识到并不是所有的博客都会同时加载博客文章和评论；很多博客只有在你滚动到页面底部时才加载评论。

为了演示功能，我们将假设我们的博客平台将同时检索博客文章和评论。

我们需要修改`Entries`服务来实现这一点。再次强调，这将非常容易！

**`src/entries/entries.service.ts`**

```js
import { Component } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

import { Entry } from './entry.entity';

@Injectable()
export class EntriesService {
  constructor(
    @InjectRepository(Entry) private readonly entry: Repository<Entry>
  ) {}

  findAll() {
    return this.entry.find();
  }

  findOneById(id: number) {
    return this.entry.findOneById(id, { relations: ['comments'] });
  }

  create(newEntry: Entry) {
    this.entry.save(newEntry);
  }
}

```

我们只在`Entry`存储库的`findOneById()`方法的第二个参数中添加了`{ relations: ['comments'] }`。选项对象的`relations`属性是一个数组，因此我们可以检索出我们需要的任意多个关系。它也可以与任何`find()`相关方法一起使用（即`find()`、`findByIds()`、`findOne()`等等）。

## 懒惰关系

在使用 TypeORM 时，常规关系（就像我们迄今为止写的那样）是*急切*关系。这意味着当我们从数据库中读取实体时，`find*()`方法将返回相关的实体，而无需我们编写连接或手动读取它们。

我们还可以配置我们的实体将关系视为*懒惰*，这样相关的实体在我们说之前不会从数据库中检索出来。

这是通过将保存相关实体的字段类型声明为`Promise`而不是直接类型来实现的。让我们看看代码上的区别：

```js
// This relationship will be treated as eager
@OneToMany(type => Comment, comment => comment.id)
comments: Comment[];

// This relationship will be treated as lazy
@OneToMany(type => Comment, comment => comment.id)
comments: Promise<Comment[]>;

```

当然，使用懒惰关系意味着我们需要改变保存实体到数据库的方式。下一个代码块演示了如何保存懒惰关系。请注意`create()`方法。

**`src/entries/entries.controller.ts`**

```js
import { Controller, Get, Post, Body, Param } from '@nestjs/common';

import { EntriesService } from './entries.service';
import { CommentsService } from '../comments/comments.service';

import { Entry } from './entry.entity';
import { Comment } from '../comments/comment.entity';

@Controller('entries')
export class EntriesController {
  constructor(
    private readonly entriesSrv: EntriesService,
    private readonly commentsSrv: CommentsService
  ) {}

  @Get()
  findAll() {
    return this.entriesSrv.findAll();
  }

  @Get(':entryId')
  findAll(@Param('entryId') entryId) {
    return this.entriesSrv.findOneById(entryId);
  }

  @Post()
  async create(@Body() input: { entry: Entry; comments: Comment[] }) {
    const { entry, comments } = input;
    const resolvedComments = [];
    await comments.forEach(async comment => {
      await this.commentsSrv.create(comment);
      resolvedComments.push(comment);
    });
    entry.comments = Promise.resolve(resolvedComments);
    return this.entriesSrv.create(entry);
  }
}

```

通过以下方式使`create()`方法变为“懒惰”：

1.  初始化一个新的`resolvedComments`空数组。

1.  遍历请求中收到的所有评论，保存每一条评论，然后将其添加到`resolvedComments`数组中。

1.  当所有评论都被保存时，我们将一个 promise 分配给`entry`的`comments`属性，然后立即用第 2 步中构建的评论数组解决它。

1.  将带有相关评论的`entry`保存为已解决的 promise。

在保存之前将一个立即解决的 promise 分配为实体的值的概念并不容易理解。但是，由于 JavaScript 的异步性质，我们仍然需要诉诸于这一点。

话虽如此，请注意 TypeORM 对懒惰关系的支持仍处于实验阶段，因此请谨慎使用。

# 其他类型的关系

到目前为止，我们已经探讨了“一对多”的关系。显然，TypeORM 也支持“一对一”和“多对多”的关系。

## 一对一

以防你不熟悉这种关系，其背后的想法是一个实体的一个实例，只属于另一个实体的一个实例，而且只属于一个。

举个更具体的例子，假设我们要创建一个新的`EntryMetadata`实体来存储我们想要跟踪的新事物，比如，假设博客文章从读者那里得到的喜欢数量和每篇博客文章的短链接。

让我们从创建一个名为`EntryMetadata`的新实体开始。我们将把文件放在`/entry`文件夹中，与`entry.entity.ts`文件相邻。

**`src/entries/entry_metadata.entity.ts`**

```js
import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';

@Entity()
export class EntryMetadata {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column() likes: number;

  @Column() shortlink: string;
}

```

我们刚刚创建的实体非常简单：它只有常规的`uuid`属性，以及用于存储条目的`likes`和`shortlink`的两个其他属性。

现在让我们告诉 TypeORM 在每个`Entry`实例中包含一个`EntryMetadata`实体的实例。

**`src/entries/entry.entity.ts`**

```js
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn,
  VersionColumn,
  OneToMany,
  OneToOne,
  JoinColumn,
} from 'typeorm';

import { EntryMetadata } from './entry-metadata.entity';
import { Comment } from '../comments/comment.entity';

@Entity()
export class Entry {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column() title: string;

  @Column('text') body: string;

  @Column() image: string;

  @Column('simple-json') author: { first_name: string; last_name: string };

  @OneToOne(type => EntryMetadata)
  @JoinColumn()
  metadata: EntryMetadata;

  @OneToMany(type => Comment, comment => comment.id, {
    cascade: true,
  })
  comments: Comment[];

  @CreateDateColumn() created_at: Date;

  @UpdateDateColumn() modified_at: Date;

  @VersionColumn() revision: number;
}

```

您可能已经注意到了`@JoinColumn()`装饰器。在“一对一”关系中使用这个装饰器是 TypeORM 所要求的。

### 双向一对一关系

此时，`Entry`和`EntryMetadata`之间的关系是单向的。在这种情况下，这可能已经足够了。

然而，假设我们想直接访问`EntryMetadata`实例，然后获取它所属的`Entry`实例的可能性。好吧，现在我们还不能做到；直到我们使关系双向为止。

因此，仅出于演示目的，我们将在`EntryMetadata`实例中包含到`Entry`实例的反向关系，以便你知道它是如何工作的。

**`src/entries/entry_metadata.entity.ts`**

```js
import { Entity, PrimaryGeneratedColumn, Column, OneToOne } from 'typeorm';

import { Entry } from './entry.entity';

@Entity()
export class EntryMetadata {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column() likes: number;

  @Column() shortlink: string;

  @OneToOne(type => Entry, entry => entry.metadata)
  entry: Entry;
}

```

确保不要在第二个条目中包含`@JoinColumn()`装饰器。该装饰器应该只用在拥有者实体中；在我们的情况下，就是`Entry`中。

我们需要做的第二个调整是指向原始`@OneToOne()`装饰器中相关实体的位置。记住，我们刚刚看到这需要通过向装饰器传递第二个参数来完成，就像这样：

**`src/entries/entry.entity.ts`**

```js
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn,
  VersionColumn,
  OneToMany,
  OneToOne,
  JoinColumn,
} from 'typeorm';

import { EntryMetadata } from './entry-metadata.entity';
import { Comment } from '../comments/comment.entity';

@Entity()
export class Entry {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column() title: string;

  @Column('text') body: string;

  @Column() image: string;

  @Column('simple-json') author: { first_name: string; last_name: string };

  @OneToOne(type => EntryMetadata, entryMetadata => entryMetadata.entry)
  @JoinColumn()
  metadata: EntryMetadata;

  @OneToMany(type => Comment, comment => comment.id, {
    cascade: true,
  })
  comments: Comment[];

  @CreateDateColumn() created_at: Date;

  @UpdateDateColumn() modified_at: Date;

  @VersionColumn() revision: number;
}

```

就是这样！现在我们有了一个美丽的、工作正常的`Entry`和`EntryMetadata`实体之间的双向一对一关系。

顺便说一句，如果你想知道我们如何保存然后检索这两个相关的实体，我有好消息告诉你：它的工作方式与我们在本章前面看到的一对多关系相同。因此，要么像在本章前面介绍的那样手动操作，要么（我个人的最爱）使用“级联”来保存它们，并使用`find*()`来检索它们！

## 多对多

我们可以为我们的实体建立的最后一种关系类型被称为“多对多”。这意味着拥有实体的多个实例可以包含拥有实体的多个实例。

一个很好的例子可能是我们想要为我们的博客条目添加“标签”。一个条目可能有几个标签，一个标签可以用在几个博客条目中，对吧。这使得关系属于“多对多”类型。

我们将节省一些代码，因为这些关系的声明方式与“一对一”关系完全相同，只需将`@OneToOne()`装饰器更改为`@ManyToMany()`。

# 高级 TypeORM

让我们来看看安全。

## 首先是安全

如果你在本书的 Sequelize 章节中阅读过，你可能对生命周期钩子的概念很熟悉。在那一章中，我们使用`beforeCreate`钩子在将用户密码保存到数据库之前对其进行加密。

如果你想知道 TypeORM 中是否也存在这样的东西，答案是肯定的！尽管 TypeORM 文档将它们称为“监听器”。

因此，为了演示其功能，让我们编写一个非常简单的`User`实体，其中包含用户名和密码，并且在将其保存到数据库之前，我们将确保加密密码。我们将在 TypeORM 中使用的特定监听器称为`beforeInsert`。

```js
@Entity
export class User {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column() username: string;

  @Column() password: string;

  @BeforeInsert()
  encryptPassword() {
    this.password = crypto.createHmac('sha256', this.password).digest('hex');
  }
}

```

## 其他监听器

一般来说，监听器是在 TypeORM 中特定事件发生时触发的方法，无论是与写相关还是与读相关。我们刚刚了解了`@BeforeInsert()`监听器，但我们还有其他一些可以利用的监听器：

+   `@AfterLoad()`

+   `@BeforeInsert()`

+   `@AfterInsert()`

+   `@BeforeUpdate()`

+   `@AfterUpdate()`

+   `@BeforeRemove()`

+   `@AfterRemove()`

## 组合和扩展实体

TypeORM 提供了两种不同的方式来减少实体之间的代码重复。其中一种遵循组合模式，而另一种遵循继承模式。

尽管很多作者都支持组合优于继承，但我们将在这里介绍这两种可能性，并让读者决定哪种更适合他/她自己的特定需求。

### 嵌入式实体

在 TypeORM 中组合实体的方式是使用一种称为嵌入式实体的工件。

嵌入式实体基本上是具有一些声明的表列（属性）的实体，可以包含在其他更大的实体中。

让我们以一个例子开始：在审查我们之前为`Entry`和`Comment`实体编写的代码之后，我们很容易看到（除其他外）有三个重复的属性：`created_at`，`modified_at`和`revision`。

创建一个“可嵌入”实体来保存这三个属性然后将它们嵌入到我们的原始实体中会是一个很好的主意。让我们看看如何做。

我们首先将创建一个`Versioning`实体（名称不太好，我知道，但应该能让您看到这个想法）带有这三个重复的属性。

**`src/common/versioning.entity.ts`**

```js
import { CreateDateColumn, UpdateDateColumn, VersionColumn } from 'typeorm';

export class Versioning {
  @CreateDateColumn() created_at: Date;

  @UpdateDateColumn() modified_at: Date;

  @VersionColumn() revision: number;
}

```

请注意，我们在这个实体中没有使用@Entity 装饰器。这是因为它不是一个“真正”的实体。把它想象成一个“抽象”实体，即一个我们永远不会直接实例化的实体，而是我们将用它来嵌入到其他可实例化的实体中，以便为它们提供一些可重用的功能。换句话说，从较小的部分组合实体。

因此，现在我们将把这个新的“可嵌入”实体嵌入到我们的两个原始实体中。

**`src/entries/entry.entity.ts`**

```js
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  OneToMany,
  OneToOne,
  JoinColumn,
} from 'typeorm';

import { EntryMetadata } from './entry-metadata.entity';
import { Comment } from '../comments/comment.entity';
import { Versioning } from '../common/versioning.entity';

@Entity()
export class Entry {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column() title: string;

  @Column('text') body: string;

  @Column() image: string;

  @Column('simple-json') author: { first_name: string; last_name: string };

  @OneToOne(type => EntryMetadata, entryMetadata => entryMetadata.entry)
  @JoinColumn()
  metadata: EntryMetadata;

  @OneToMany(type => Comment, comment => comment.id, {
    cascade: true,
  })
  comments: Comment[];

  @Column(type => Versioning)
  versioning: Versioning;
}

```

**`src/comments/comment.entity.ts`**

```js
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

import { Versioning } from '../common/versioning.entity';

@Entity()
export class Comment {
  @PrimaryGeneratedColumn('uuid') id: string;

  @Column('text') body: string;

  @Column('simple-json') author: { first_name: string; last_name: string };

  @Column(type => Versioning)
  versioning: Versioning;
}

```

即使在这个非常简单的例子中，我们已经将两个原始实体从三个不同的属性减少到了一个！在`Entry`实体和`Comment`实体中，当我们调用它们的读取或写入方法时，`versioning`列将被`Versioning`嵌入实体内的属性实际替换。

### 实体继承

TypeORM 为在我们的实体之间重用代码提供了第二种选择，即使用实体继承。

如果您已经熟悉 TypeScript，那么当您考虑到实体只是带有一些装饰器的常规 TS 类时，实体继承就很容易理解（和实现）。

对于这个特定的例子，让我们假设我们基于 Nest.js 的博客已经在线上一段时间了，并且它已经相当成功。现在我们想要引入赞助博客条目，这样我们就可以赚一些钱并将它们投资到更多的书籍中。

问题是，赞助条目将与常规条目非常相似，但会有一些新属性：赞助商名称和赞助商网址。

在这种情况下，经过一番思考后，我们可能决定扩展我们的原始`Entry`实体并创建一个`SponsoredEntry`。

**`src/entries/sponsored-entry.entity.ts`**

```js
import { Entity, Column } from 'typeorm';

import { Entry } from './entry.entity';

@Entity()
export class SponsoredEntry extends Entry {
  @Column() sponsorName: string;

  @Column() sponsorUrl: string;
}

```

就是这样。我们从`SponsoredEntry`实体创建的任何新实例都将具有来自扩展的`Entry`实体的相同列，以及我们为`SponsoredEntry`定义的两个新列。

## 缓存

TypeORM 默认提供了一个缓存层。我们可以利用它，只需稍微增加一点开销。如果您正在设计一个预期会有大量流量和/或您需要尽可能获得最佳性能的 API，这一层将特别有用。

这两种情况都会因为使用更复杂的数据检索场景（例如复杂的`find*()`选项，大量相关实体等）而越来越受益于缓存。

在连接到数据库时，缓存需要显式激活。到目前为止，在我们的情况下，这将是我们在本章开头创建的`ormconfig.json`文件。

**ormconfig.json**

```js
{
  "type": "mariadb",
  "host": "db",
  "port": 3306,
  "username": "nest",
  "password": "nest",
  "database": "nestbook",
  "synchronize": true,
  "entities": ["src/**/*.entity.ts"],
  "cache": true
}

```

在连接上激活缓存层之后，我们需要将`cache`选项传递给我们的`find*()`方法，就像下面的例子中那样：

```js
this.entry.find({ cache: true });

```

上面的代码将使`.find()`方法在缓存值存在且未过期时返回缓存值，否则返回相应数据库表中的值。因此，即使在过期时间窗口内调用该方法三千次，实际上只会执行一次数据库查询。

TypeORM 在处理缓存时使用了一些默认值：

1.  默认的缓存生命周期是 1,000 毫秒（即 1 秒）。如果我们需要自定义过期时间，我们只需要将所需的生命周期作为值传递给选项对象的`cache`属性。在上面的例子中，`this.entry.find({ cache: 60000 })`将设置 60 秒的缓存 TTL。

1.  TypeORM 将在您已经使用的同一数据库中为缓存创建一个专用表。该表将被命名为`query-result-cache`。这并不是坏事，但如果我们有一个可用的 Redis 实例，它可以得到很大的改进。在缓存中，我们需要在`ormconfig.json`文件中包含我们的 Redis 连接详细信息：

**ormconfig.json**

```js
{
  "type": "mariadb",
  ...
  "cache": {
    "type": "redis",
    "options": {
      "host": "localhost",
      "port": 6379
    }
  }
}

```

这样我们可以在高负载下轻松提高 API 的性能。

## 构建查询

TypeORM 的存储库方法极大地隔离了我们查询的复杂性。它们提供了一个非常有用的抽象，使我们不需要关心实际的数据库查询。

然而，除了使用这些不同的`.find*()`方法之外，TypeORM 还提供了手动执行查询的方法。这在访问我们的数据时极大地提高了灵活性，但代价是需要我们编写更多的代码。

TypeORM 执行查询的工具是`QueryBuilder`。一个非常基本的例子可能涉及重构我们旧有的`findOneById()`方法，使其使用`QueryBuilder`。

**`src/entries/entries.service.ts`**

```js
import {getRepository} from "typeorm";
...

findOneById(id: number) {
  return getRepository(Entry)
    .createQueryBuilder('entry')
    .where('entry.id = :id', { id })
    .getOne();
}

...

```

另一个稍微复杂一些的情景是构建一个连接，以便还检索相关的实体。我们将再次回到我们刚刚修改以包括相关评论的`findOneById()`方法。

**`src/entries/entries.service.ts`**

```js
import {getRepository} from "typeorm";
...

findOneById(id: number) {
  return getRepository(Entry)
    .createQueryBuilder('entry')
    .where('entry.id = :id', { id })
    .leftJoinAndSelect('entry.comments', 'comment')
    .getOne();
}

...

```

## 从现有数据库构建我们的模型

直到这一点，我们从一个“干净”的数据库开始，然后创建我们的模型，将模型转换为数据库列的任务交给了 TypeORM。

这是“理想”的情况，但是...如果我们发现自己处于相反的情况下怎么办？如果我们已经有一个填充了表和列的数据库呢？

有一个很好的开源项目可以用于这个：[typeorm-model-generator](https://github.com/Kononnable/typeorm-model-generator)。它被打包为一个命令行工具，可以使用`npx`运行。

**注意：**如果您对此不熟悉，`npx`是一个随`npm` > 5.2 一起提供的命令，它允许我们在命令行中运行 npm 模块，而无需先安装它们。要使用它，您只需要在工具的常规命令之前加上`npx`。例如，如果我们想要使用 Angular CLI 在命令行中创建一个新项目，我们将使用`npx ng new PROJECT-NAME`。

当它被执行时，typeorm-model-generator 将连接到指定的数据库（它支持大致与 TypeORM 相同的数据库），并将根据我们作为命令行参数传递的设置生成实体。

由于这是一个仅适用于一些非常特定用例的有用工具，我们将在本书中略去配置细节。但是，如果您发现自己在使用这个工具，请前往[其 GitHub 存储库](https://github.com/Kononnable/typeorm-model-generator)查看。

# 总结

TypeORM 是一个非常有用的工具，使我们能够在处理数据库时进行大量的繁重工作，同时大大抽象了数据建模、查询和复杂的连接，从而简化了我们的代码。

由于 Nest.js 通过`@nest/typeorm`包提供了很好的支持，因此它也非常适合用于 Nest.js 项目。

本章涵盖的一些内容包括：

+   TypeORM 支持的数据库类型以及如何选择其中一种的一些建议。

+   如何将 TypeORM 连接到您的数据库。

+   什么是实体以及如何创建您的第一个实体。

+   从您的数据库中存储和检索数据。

+   利用 TypeORM 使处理元数据（ID、创建和修改日期等）更容易。

+   自定义数据库中列的类型以匹配您的需求。

+   建立不同实体之间的关系以及在从数据库读取和写入时如何处理它们。

+   更高级的程序，如通过组合或继承重用代码；连接到生命周期事件；缓存；以及手动构建查询。

总的来说，我们真的认为你对 Nest.js 越熟悉，就越有可能开始感觉写 TypeORM 代码更舒适，因为它们在一些方面看起来很相似，比如它们广泛使用 TypeScript 装饰器。

在下一章中，我们将介绍 Sequelize，这是一个基于 Promise 的 ORM。
