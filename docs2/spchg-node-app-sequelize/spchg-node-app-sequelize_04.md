# 4

# 模型关联

除了使用验证来确保数据库内部的一致性之外，我们还可以在两个表之间创建关联，以确保共生关系得到维护和更新。数据库通过创建**外键引用**来维护这些关系，这些引用包含有关外键关联的表和列的元数据。这些元数据维护数据库的完整性。如果我们不进行适当的引用更新外键的值，我们就必须执行一个单独的查询来更新所有引用外键的新值的行。

例如，我们有三张表：`customers`、`products`和`receipts`。`receipts`表将有两个列（除了其他列之外），分别引用`customers`和`products`表上的列。如果我们想更新产品的标识列，我们只需修改相关产品的标识值。然后，引用`receipts`表中产品的行将自动更新。如果没有外键引用，我们必须在更新产品的标识后显式更新`receipts`表。

备注

传统上，外键会引用主键列或某种形式的标识列，但你并不局限于这些列。

模型之间的关系可以通过 Sequelize 自动管理，或者以可配置的方式管理，以采用现有的数据库。映射模型之间的关系可以帮助 ORM 根据环境通过**预加载**或**延迟加载**形成高效的查询。

本章将涵盖以下主题：

+   关联方法

+   关系模式

+   使用预加载和延迟加载查询关联

+   使用高级关联选项

+   将关联应用于 Avalon Airlines

# 技术要求

您可以在此处找到本章的代码文件：[`github.com/PacktPublishing/Supercharging-Node.js-Applications-with-Sequelize/tree/main/ch4`](https://github.com/PacktPublishing/Supercharging-Node.js-Applications-with-Sequelize/tree/main/ch4)

# 关联方法

在 ORM 中创建关系映射有几个选项。通过 ORM 定义关系可以帮助自动构建具有正确属性和列的数据库，管理关联验证（例如，检查是否严格只有一个相关记录），并在检索或插入数据时执行查询优化模式。Sequelize 支持四种关联模式：

+   **HasOne** – 一种一对一关联，其中*外键引用子*模型。该属性在父模型中定义。

+   **BelongsTo** – 一种一对一关联，其中*外键引用父*模型。该属性在子模型中定义。

+   **HasMany** – 一种一对一关联，其中*外键引用父*模型。该属性在子模型中定义。

+   **BelongsToMany** – 一种多对多关联，其中将使用单独的模型（称为 *连接表*）存储关联模型的引用。

Sequelize 将遵循在具有关联的模型上创建方法的模式。根据关系，方法名称前缀可以是 `get`、`set`、`add`、`create`、`remove` 和 `count`，后面跟着关联的名称（在适用的情况下可以是单数或复数形式）。

在本节中，我们将介绍关联及其对应方法的列表。一旦我们完成了关联概述，我们就可以开始关系概述的模式。在本节中，我们将使用演员和戏剧/电影/职位的概念来帮助我们掌握模型关联属性和行为的基本原理。这些示例仅用于说明目的，不应包含在我们的项目代码库中。

注意

你可以在 `get` 关联方法中添加 `where` 子句语句（以及其他查找参数），如下所示：

`Actor.getJobs({`

`where: { category: 'Action' },`

`limit: 10, offset: 20, /* etc. */`

`});`

## hasOne

`hasOne` 关联将为关联模型生成 `get`、`set` 和 `create` 方法。假设我们有以下关联和实例：

```js
Actor.hasOne(Job);
const actor = await Actor.create({ … });
```

我们可以使用 `createJob` 方法来插入并设置与演员相关的职位：

```js
await actor.createJob({ name: '…' });
```

`setJob` 方法将更新演员和职位之间的关联，从 `Job` 实例：

```js
const job = await Job.create({ name: '…' });
await actor.setJob(job);
```

你可以使用 `set` 前缀方法通过 `null` 值来移除关联：

```js
await actor.setJob(null);
```

## belongsTo

让我们改变之前示例中的模型为以下内容：

```js
Actor.belongsTo(Job);
```

`belongsTo` 关联将在与 `hasOne` 关联相同的模型上生成完全相同的方法。为了进一步解释，此关联不会在 `Job` 模型上创建 `setActor`，但仍然会在 `Actor` 模型上创建 `setJob` 方法。

## hasMany

`hasMany` 关联将生成以下前缀方法：`get`、`set`、`create`、`count`、`has`、`add` 和 `remove`。`get` 和 `set` 方法与之前的示例类似，除了方法名称的后缀将是模型名称的复数形式：

```js
Actor.hasMany(Job);
let jobs = await Job.findAll();
await Actor.setJobs(jobs);
await Actor.getJobs();
```

`create` 前缀方法仍然一次只创建一条记录，因此模型名称仍然是单数形式：

```js
await Actor.createJob({ name: '…' });
```

我们还可以使用 `has` 方法检查是否存在关系：

```js
const job = await Job.findOne();
// true or false boolean value
const hasJob = await Actor.hasJob(job);
// using jobs from our previous example
const hasAllJobs = await Actor.hasJobs(jobs);
```

我们可以使用 `add` 方法添加一个或多个关联，如下所示：

```js
await Actor.addJob(job);
await Actor.addJobs(jobs);
```

要检索模型有多少关联，我们可以调用 `count` 方法：

```js
// will return 2 following the examples in this section
await Actor.countJobs();
```

使用 `remove` 方法可以移除关联：

```js
await Actor.removeJob(job);
await Actor.removeJobs(jobs);
```

## belongsToMany

然后，我们将关联更改为 `belongsToMany`：

```js
Actor.belongsToMany(Job, { through: '...' });
```

`belongsToMany` 关联将在与之前 `hasMany` 示例相同的模型上生成相同的方法，类似于 `belongsTo` 生成与 `hasOne` 关联相同的方法。

## 重命名关联

您可以通过使用 `as` 参数创建关联的别名来修改 Sequelize 生成的方法名：

```js
Actor.hasOne(Job, {
  as: 'gig'
});
const actor = await Actor.create({ … });
const gig = await Job.create({ … });
actor.createGig({ … });
actor.setGig(gig);
actor.hasGig(gig);
```

注意

您可以直接将关系的标识符设置为属性，Sequelize 将使用 `save` 方法更新该值。然而，如果您在关联记录中进行了任何更改，通过调用关联实例的 `save` 方法，它们的信息将不会被更新。对实际关联的更改需要从它们自己的实例中进行。

现在我们知道了如何将关联方法应用于我们的模型，我们可以回顾各种关系模式，以帮助我们更好地理解这些关联在哪里以及何时被耦合。在下一节中，我们将详细介绍 Sequelize 支持的关系模式以及每个模式的示例。

# 关联模式

在本节中，我们将详细介绍每种类型的关系（除了将在下一节中讨论的超多对多关系），以及如何使用 Sequelize 定义关联。之后，我们将更新 Avalon Airlines 项目的模型以包含关联。

我们可以将几个关联模式组合起来定义一个关系模式。Sequelize 支持四种关系模式：

+   **一对一** – 我们将一起使用 *hasOne* 和 *belongsTo* 关联。

+   **一对多** – 使用 *hasMany* 和 *belongsTo* 关联来实现此模式。

+   **多对多** – 使用两个 *belongsToMany* 关联来实现此模式。

+   **超多对多** – 两个 *One-to-Many* 关系，其中 *One* 模型仍然被认为是相互依存的。这种关系将在 *创建超多对多关系* 部分中进一步详细说明。

## 一对一

一对一关系模式涉及模型之间的 `hasOne` 和 `belongsTo` 关联。这两个关联之间的区别在于哪个表将包含标识列。

例如，我们有一个 `Airplane` 和 `BoardingTicket` 模型。由于 `Airplane` 模型在航班完成后将不再与 `BoardingTicket` 相关，我们可以从 `Airplane` 模型的表中省略对登机牌的记忆。这意味着 `Airplane` 不需要 `hasOne` 关联，但 `BoardingTicket` 仍然需要一个 `belongsTo` 关联。

要定义一对一关系，我们会这样定义我们的模型：

```js
const A = sequelize.define('A', { … });
const B = sequelize.define('B', { … });
A.hasOne(B);
B.belongsTo(A);
```

使用 Sequelize 的 `sync` 命令会产生以下查询：

```js
CREATE TABLE IF NOT EXISTS "b" (
  /* ... */
);
CREATE TABLE IF NOT EXISTS "b" (
  /* ... */
  "aId" INTEGER REFERENCES "a" ("id") ON DELETE SET NULL ON UPDATE CASCADE
  /* ... */
);
```

注意

如果不调用 `A.hasOne(B)`，Sequelize 将不知道如何从模型 A 到 B 进行预加载（但可以从模型 B 到 A 进行预加载）。

有几种选项可以作为第二个参数传递以调整关联的行为：

+   `onUpdate` – 告诉数据库管理系统如何处理更新的外键关系。可能的值有 `CASCADE`、`RESTRICT`、`NO ACTION`、`SET DEFAULT` 和 `SET NULL`。此选项的默认值为 `CASCADE`。

+   `onDelete` – 与 `onUpdate` 相同，但用于处理已删除的外部关系。此选项的默认值为 `SET NULL`。

+   `foreignKey` – 接受一个字面字符串值或一个对象，该对象在定义模型时具有与属性相同的选项（`name`、`allowNull`、`unique` 等）。

+   `sourceKey` – 用于识别外键列值的源表列的名称。默认情况下，Sequelize 将使用具有 `primaryKey: true` 参数的源表属性。如果您的模型不包含显式的 `primaryKey` 属性，则 Sequelize 将使用默认的 `id` 属性。适用于 `hasOne` 和 `hasMany` 关联。

+   `targetKey` – 与 `sourceKey` 类似，但此值将引用目标表中的列，而不是源表中的列。适用于 `belongsTo` 关联。

这里有一些如何使用这些选项的示例：

```js
A.hasOne(B, {
    onUpdate: 'SET NULL',
    onDelete: 'CASCADE',
    foreignKey: 'otherId'
});
B.belongsTo(A);
A.hasOne(B, {
    onUpdate: 'CASCADE',
    onDelete: 'SET NULL',
    foreignKey: { name: 'otherId' }
});
B.belongsTo(A);
```

您可以在模型 A 和 B 之间互换使用第二个选项：

```js
A.hasOne(B);
B.belongsTo(A, {
    onUpdate: 'SET NULL',
    onDelete: 'CASCADE',
    foreignKey: 'otherId'
});
A.hasOne(B);
B.belongsTo(A, {
    onUpdate: 'CASCADE',
    onDelete: 'SET NULL',
    foreignKey: { name: 'otherId' }
});
```

默认情况下，Sequelize 会将一对一关系设置为可选，但如果我们想要在两个模型之间强制关系，那么我们会在关联选项中将 `allowNull` 定义为 `false`，如下所示：

```js
A.hasOne(B, {
  foreignKey: { allowNull: false }
});
```

## 一对多

这种关系模式将仅在 `belongsTo` 模型上创建外键引用列。使用 `hasMany` 关联定义属性有助于 Sequelize 进行数据检索优化，并为父模型添加辅助函数。第二个参数中的选项与一对一关系相同。

假设我们有一个 `Employees` 模型属于 `Organization`。使用 Sequelize，代码将类似于以下内容：

```js
Organization.hasMany(Employee);
Employee.belongsTo(Organization);
```

这将执行以下几个查询：

```js
CREATE TABLE IF NOT EXISTS "Organizations" (
  /* ... */
);
CREATE TABLE IF NOT EXISTS "Employees" (
  /* ... */
  "OrganizationId" INTEGER REFERENCES "Organizations" ("id") ON DELETE SET NULL ON UPDATE CASCADE,
  /* ... */
);
```

## 多对多

这种关系将使用关联实体来保持两个模型之间的引用。关联实体的其他名称包括连接表、连接模型、交叉引用表和配对表。使用 `sequelize.sync()`，Sequelize 将自动为您创建连接模型，但我们仍然可以选择定义自己的连接表，以便在需要添加更多属性、约束、生命周期事件等情况下使用。

在此示例中，我们有一些员工被分配了任务。员工可以处理多个任务，而任务可能需要多个员工：

```js
Employee.belongsToMany(Task, { through: 'EmployeeTasks' });
Task.belongsToMany(Employee, { through: 'EmployeeTasks' });
```

这将执行以下查询：

```js
CREATE TABLE IF NOT EXISTS "EmployeeTasks" (
    "createdAt" TIMESTAMP WITH TIME ZONE NOT NULL,
    "updatedAt" TIMESTAMP WITH TIME ZONE NOT NULL,
    "EmployeeId" INTEGER REFERENCES "Employees" ("id") ON DELETE CASCADE ON UPDATE CASCADE,
    "TaskId" INTEGER REFERENCES "Tasks" ("id") ON DELETE CASCADE ON UPDATE CASCADE,
    PRIMARY KEY ("EmployeeId","TaskId")
);
```

备注

多对多关系将使用 `CASCADE` 作为默认行为来管理更新和删除时的外键和关系。

如果我们希望在定义中更加明确，或者想要为连接模型添加自定义属性，我们可以这样定义连接模型和关系：

```js
// Employee and Task are pre-defined for brevity
const EmployeeTasks = sequelize.define('EmployeeTasks', {
    EmployeeId: {
      type: DataTypes.INTEGER,
      references: {
        model: Employee,
        key: 'id'
      }
    },
    TaskId: {
      type: DataTypes.INTEGER,
      references: {
        model: 'Tasks', // string literal values work here 
                           too
        key: 'id'
      }
    },
    SomeOtherColumn: {
        type: DataTypes.STRING
    }
});
Employee.belongsToMany(Task, {
    through: EmployeeTasks
});
Task.belongsToMany(Employee, {
    through: EmployeeTasks
});
```

除了 `through` 参数外，多对多关联还提供了一个名为 `uniqueKey` 的参数，这将允许你命名一个引用列。默认情况下，Sequelize 将创建一个由引用列（`employeeId` 和 `taskId`）组成的唯一键的连接表。如果你希望改变这种行为，你可以在连接模型定义中将 `unique` 属性参数设置为 `false`。

## 使用自定义外键定义

在何时以及如何正确使用 `sourceKey` 和 `targetKey` 可能一开始会让人困惑。`hasOne` 和 `hasMany` 关联将从父模型（目标模型）引用到子模型（源模型）；另一种说法是，“这个孩子通过名字、婚姻等是我的。”`belongsTo` 关联将引用父模型，或者说，“我通过名字、婚姻等属于这个父模型。”

我们将这些模型称为 *源* 和 *目标* 模型，因为父亲和子代可能会产生误导，并暗示某种形式的层次结构。关联不需要层次结构；它们只是形成关系。

注意

被定义为引用的属性必须应用唯一约束。这可以通过在属性选项中添加 `unique: true` 并使用 Sequelize 的 `sync()` 方法来实现。

为了说明如何配置源和目标键，我们首先定义我们的模型：

```js
const Actor = sequelize.define('Actors', {
    name: {
        type: DataTypes.TEXT,
        unique: true
    }
});
const Role = sequelize.define('Roles', {
    title: {
        type: DataTypes.TEXT,
        unique: true
    }
});
const Costume = sequelize.define('Costumes', {
    wardrobe: {
        type: DataTypes.TEXT,
        unique: true
    }
});
```

使用这些角色，我们将分别通过 `hasOne`、`hasMany`、`belongsTo` 和 `belongsToMany` 的示例来展示。

### 使用 hasOne

以下代码将在 `Roles` 模型上创建一个名为 `actorName` 的属性，该属性的值将与演员的名字相关联（而不是演员的 ID 属性）：

```js
Actor.hasOne(Role, {
    sourceKey: 'name',
    foreignKey: 'actorName'
});
```

### 使用 hasMany

我们将为 `hasMany` 关联使用相同的选项。以下代码将在 `Costumes` 模型上创建一个名为 `roleTitle` 的属性，它将与角色的标题相关联：

```js
Roles.hasMany(Costumes, {
    sourceKey: 'title',
    foreignKey: 'roleTitle'
});
```

### 使用 belongsTo

`belongsTo` 关联的工作方式略有不同。它不是从源模型引用，而是从目标模型引用，如下所示（这将产生与之前 `Actors.hasOne(Roles, ...)` 代码相同的结果）：

```js
Roles.belongsTo(Actors, {
    targetKey: 'name',
    foreignKey: 'actorName'
});
```

换句话说，当使用 `belongsTo` 时，外键将放置在创建关联的模型上，而对于 `hasOne`/`hasMany` 关联，外键将放置在未调用关联方法的另一个模型上。

### 使用 belongsToMany

`belongsToMany` 关联接受目标和源键作为参数来配置引用。在电影场景中，一个演员可能为他们的所有不同场景拥有多个服装。我们可以在 Sequelize 中这样表示这种关系：

```js
Costumes.belongsToMany(Actors, {
    through: 'actor_costumes',
    sourceKey: 'name',
    targetKey: 'wardrobe'
});
```

这将创建一个名为 `actor_costumes` 的连接表，其中包含两个引用列 `actorsName` 和 `costumesWardrobe`，分别引用 Actor 和 Costume 模型。

现在，我们已经讨论了 Sequelize 关联模式、选项、用例和示例的定义、选项、用例和示例，以及主要的三种关系模式，我们可以开始练习在选择或修改记录时包含这些关系。

# 使用急加载和懒加载查询关联

Sequelize 根据您希望如何查询数据提供两种不同的查询关联方法：急加载和懒加载。使用急加载，您将一次性加载所有关联数据。懒加载方法将根据代码中的调用逐个加载关联。与懒加载相比，急加载更容易解释，但要看到急加载的好处，我们首先需要了解懒加载。

注意

您可能听说过其他 ORM 框架中的“N+1 查询问题”；这指的是懒加载方法（尽管不是相互排斥）以及逐行选择关联如何可能对应用程序的性能造成危害。

## 懒加载

Sequelize 尽量不对您的意图做出假设，并且最初只选择模型数据。如果我们想遍历模型的相关数据，我们需要明确调用关联。懒加载的一个良好用例是条件查询相关数据（例如，我们可能不想在电影发布后获取电影评论）。懒加载看起来可能如下所示：

```js
const actor = await Actor.findOne();
// SELECT * FROM jobs WHERE actorId=?
const job = await Actor.getJob();
let reviews = [];
if (job.isDone) {
  // SELECT * FROM reviews WHERE jobId=?
  reviews = await job.getReviews({
    where: { published: true }
  });
}
```

## 急加载

通常，您会在有很多关联或主表返回大量行时使用这种加载形式。参考之前的示例，假设我们用 `getJobs` 替换了 `getJob`，并且对每个工作调用 `getReviews`，如下所示：

```js
const jobs = await Actor.getJobs();
let reviews = [];
jobs.map(async job => {
  if (job.isDone) {
    let jobReviews = await job.getReviews({
      where: { published: true }
    });
    reviews = reviews.concat(jobReviews);
  }
});
```

如果演员变得过于著名并且有数百个工作，我们可以看到查询数量如何迅速变得对系统来说过于繁琐。防止这种情况的一种方法是通过使用急加载方法，该方法将使用 `JOIN` 语句在顶级查询中包含关联数据。让我们通过开始定义我们的已完成工作关联来将之前的示例转换为 Sequelize 的急加载查询：

```js
const completedJobs = {
    model: Job,
    as: 'CompletedJobs',
    where: {
        completed: true
    },
    include: {
        model: Review,
        where: {
            published: true
        }
    }
}
```

这个第一个 `include` 参数将加载 `Job` 模型，设置别名为 `CompletedJobs`，添加一个针对 `completed` 标志的 `where` 子句，然后从 `Job` 到 `Review` 的嵌套关联调用（以及针对评论的 `published` `where` 子句）。

接下来，我们需要定义我们的不完整工作关联：

```js
const incompleteJobs = {
    model: Job,
    as: 'IncompleteJobs',
    where: {
        completed: false
    }
}
```

第二个参数是一个更简单的 `Job` 别名关联，它从 `CompletedJobs` 中反转了 `where` 子句。

这里的想法是分别查询已完成和未完成的工作，因为我们只想包括已完成的工作的评论。现在，我们可以使用工作和评论来查询我们的演员：

```js
const actor = await Actor.findOne({
    include: [ completedJobs, incompleteJobs ]
});
```

这将生成一个类似的 SQL 查询（为了简洁，省略了一些选定的列）：

```js
SELECT
    `Actor`.*,
    `CompletedJobs`.`title` AS `CompletedJobs.title`,
    `CompletedJobs`.`completed` AS `CompletedJobs.completed`,
    `CompletedJobs->Reviews`.`id` AS `CompletedJobs.Reviews.     id`,
    `CompletedJobs->Reviews`.`published` AS `CompletedJobs.
     Reviews.published`,

    `IncompleteJobs`.`title` AS `IncompleteJobs.title`,
    `IncompleteJobs`.`completed` AS `IncompleteJobs.completed`

FROM (
    SELECT `Actor`.`id`, `Actor`.`name`, `Actor`.`createdAt`, 
    `Actor`.`updatedAt`
    FROM `Actors` AS `Actor`
    LIMIT 1
) AS `Actor`
LEFT OUTER JOIN `Jobs` AS `CompletedJobs` ON
    `Actor`.`id` = `CompletedJobs`.`ActorId` AND
    `CompletedJobs`.`completed` = true
LEFT OUTER JOIN `Reviews` AS `CompletedJobs->Reviews` ON
    `CompletedJobs`.`id` = `CompletedJobs->Reviews`.`JobId` AND
    `CompletedJobs->Reviews`.`published` = true
LEFT OUTER JOIN `Jobs` AS `IncompleteJobs` ON
    `Actor`.`id` = `IncompleteJobs`.`ActorId` AND
    `IncompleteJobs`.`completed` = false;
```

然后，将已完成和未完成的工作连接起来：

```js
const jobs = [].concat(
  actor.CompletedJobs,
  actor.IncompleteJobs
);
```

现在我们可以遍历工作，并在适用的情况下显示评论：

```js
jobs.forEach(job => {
  const reviews = job.Reviews || [];
  // display reviews here
});
```

现在我们已经了解了 Sequelize 两种加载类型的基础，我们可以开始探索在关联数据时更高级的概念。在下一节中，我们将讨论关联的更高级查询模式：超级多对多关联和多态关联。

# 使用高级关联选项

Sequelize 提供了一系列技巧来帮助正确地与数据库的关系进行通信。其中一些方法将帮助以更有组织和更舒适的方式查询关联。其他方法将为我们提供一种方式来组合数据库的架构，以实现更高级的关系模式。在本节中，我们将讨论如何使用超级多对多模式管理复杂的多对多关系、定义作用域关联以及查询多态关联的示例。

## 使用关联的作用域

作用域是一种定义命名空间的方式，它包含一组默认参数，或者覆盖之前应用于查询的参数。关联可以定义作用域以帮助组织代码库。关联作用域与模型作用域之间的一个关键区别是，关联作用域的参数适用于`WHERE`子句。模型作用域可以定义`WHERE`、`LIMIT`和`OFFSET`子句等。

以下是一个查询具有作用域的关联的示例：

```js
const Worker = sequelize.define('worker', { name: DataTypes.STRING });
const Task = sequelize.define('task', {
  title: DataTypes.STRING,
  completed: DataTypes.BOOLEAN
});
Worker.hasMany(Task, {
    scope: {
        completed: true
    },
    as: 'completedTasks'
});
const worker = await Worker.create({ name: "Bob" });
await worker.getCompletedTasks();
```

Sequelize 将为`worker`实例添加一个名为`getCompletedTasks()`的混入，该混入将执行一个类似于以下查询的操作：

```js
SELECT `id`, `completed`, `workerId`
FROM `tasks` AS `task`
WHERE `task`.`completed` = true AND `task`.`workerId` = 1;
```

`task.completed = true`这一部分是 Sequelize 根据作用域定义自动添加的。定义相同作用域的另一种方式如下所示：

```js
Task.addScope('completed', {
    where: { completed: true }
});
Worker.hasMany(Task.scope('completed'), {
  as: 'completedTasks'
});
```

当创建具有作用域的关联时，Sequelize 会在调用`create`或`add`混入函数时自动为这些参数添加默认值。例如，我们知道一个工人已经完成了一个任务，并希望插入关联，如下所示：

```js
const worker = Worker.findOne();
await worker.createCompletedTask({ title: 'Repair Cluster' });
```

当通过`worker`创建一个完成的任务时，Sequelize 将执行一个类似的查询：

```js
INSERT INTO "tasks" (
    "id", "title", "completed"
) VALUES (
    DEFAULT, 'Repair Cluster', true, 1
) RETURNING *;
```

当我们想要添加一个完成的任务时，`completed`属性已被自动设置为`true`。如果我们使用`add`混入，也会表现出相同的行为。

在`belongsToMany`关联中使用作用域参数会将作用域应用于目标模型而不是连接模型。如果您希望将作用域应用于连接模型，您可以在`through`配置内部添加作用域参数，如下所示：

```js
const WorkerTask = sequelize.define('WorkerTask', {
  published: DataTypes.BOOLEAN
});
Worker.belongsToMany(Task.scope('completed'), {
  through: {
    model: WorkerTasks,
    scope: { published: true }
  },
  as: 'CompletedAndPublishedTask'
});
```

## 创建超级多对多关系

假设我们拥有一家商店，并希望通过事务维护员工与客户之间的关联列表。通常，我们可以通过在`Employee`和`Customer`模型上添加一个`belongsToMany`关联，并使用`Transaction`模型作为连接表来定义这种关系。

让我们从本节中使用的这些模型定义开始：

```js
const Employee = sequelize.define('employee', {
  name: DataTypes.STRING,
});
const Customer = sequelize.define('customer', {
  name: DataTypes.STRING
});
const Transaction = sequelize.define('transaction', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true,
    allowNull: false
  },
  couponCode: DataTypes.STRING
});
```

你可能已经注意到，`Transaction` 模型的 `id` 属性被明确定义为主键。这将防止 Sequelize 使用 `employeeId` 和 `customerId` 的组合键作为主键，这对于建立超级多对多关系是必需的。

有两种方法可以创建这三个模型之间的多对多关系。常见的方法是使用 `belongsToMany` 关联，如下所示：

```js
Employee.belongsToMany(Customer, { through: Transaction });
Customer.belongsToMany(Employee, { through: Transaction });
```

另一种方法是同时在关联模型上使用 `hasMany` 和 `belongsTo`：

```js
Employee.hasMany(Transaction);
Transaction.belongsTo(Employee);
Customer.hasMany(Transaction);
Transaction.belongsTo(Customer);
```

这两种方法将为连接表产生相同的图示结果（在连接模型上创建 `employeeId` 和 `customerId`）。然而，当你尝试预加载数据时，可能会遇到几个问题，具体取决于你如何查询数据。

使用 `belongsToMany` 关联，我们可以按以下方式查询我们的模型（然而，这不会对 `hasMany` 和 `belongsTo` 方法起作用）：

```js
Employee.findAll({ include: Customer });
Customer.findAll({ include: Employee });
```

从关联模型中包含连接模型对于 `belongsToMany` 方法不起作用，但以下代码对于 `hasMany` 和 `belongsTo` 方法是有效的：

```js
Employee.findAll({ include: Transaction });
Customer.findAll({ include: Transaction });
```

尝试通过连接模型包含关联模型仅适用于 `hasMany` 和 `belongsTo` 方法。以下代码对 `belongsToMany` 模式不起作用：

```js
Transaction.findAll({ include: Employee });
Transaction.findAll({ include: Customer });
```

为了能够使用这些模型的所有各种预加载形式，我们可以通过组合两种关联方法（如下所示）来实现超级多对多模式：

```js
Employee.belongsToMany(Customer, { through: Transaction });
Customer.belongsToMany(Employee, { through: Transaction });
Employee.hasMany(Transaction);
Transaction.belongsTo(Employee);
Customer.hasMany(Transaction);
Transaction.belongsTo(Customer);
```

以这种方式声明我们的关联将允许我们通过连接模型或关联模型本身查询关联数据，而不受约束或规定。深度嵌套的包含也通过超级多对多关系原生支持。

## 使用多态关联

当我们有两个或更多关联模型针对连接表上的同一外键时，我们可以为该场景使用多态关联模式。你可以将多态关联视为关联数据的某种通用接口。

假设我们拥有一家在线零售店，并希望将 widget 和 gizmo 的评论存储在一个评论表中。最初，我们可能想使用 `hasMany` 和 `belongsTo` 关联，但这会导致 Sequelize 在连接模型上生成两个列（`widgetId` 和 `gizmoId`），而不是一个通用的模式。从语义上讲，这也不合理，因为评论既不与 widget 相关，也不与 gizmo 产品相关。

首先，我们需要定义我们的 `Widget` 和 `Gizmo` 模型：

```js
const Widget = sequelize.define('Widget', {
  sku: DataTypes.STRING,
  url: DataTypes.STRING
});
const Gizmo = sequelize.define('Gizmo', {
  name: DataTypes.STRING
});
```

接下来，我们将定义我们的 `Review` 模型如下：

```js
const Review = sequelize.define('Review', {
  message: DataTypes.STRING,
  entityId: DataTypes.INTEGER,
  entityType: DataTypes.STRING
}, {
    instanceMethods: {
        getEntity(options) {
if (!this.entityType) return Promise.resolve(null);
const mixinMethodName = `get${this.entityType}`;
            return thismixinMethodName;
        }
    }
});
```

`instanceMethods` 参数将为每个实例创建一个 `getEntity` 函数，该函数将检查 `entityType` 是否为空。如果有实体类型，则可以通过在实体类型的值前添加 `get` 前缀来调用关联的混合函数。

现在我们可以建立我们的关系，如下所示：

```js
Widget.hasMany(Review, {
  foreignKey: 'entityId',
  constraints: false,
  scope: {
    entityType: 'Widget'
  }
});
Review.belongsTo(Widget, { foreignKey: 'entityId', 
                           constraints: false });
Gizmo.hasMany(Review, {
  foreignKey: 'entityId',
  constraints: false,
  scope: {
    entityType: 'Gizmo'
  }
});
Review.belongsTo(Gizmo, { foreignKey: 'entityId', 
                          constraints: false });
```

对于每个关联，我们将使用`entityId`作为`foreignKey`，由于连接模型引用了多个表，我们无法在该表上设置引用约束（这就是为什么我们将`constraints`设置为`false`）：

```js
Review.addHook("afterFind", findResult => {
if (!Array.isArray(findResult)) findResult = [findResult];
  for (const instance of findResult) {
    if (instance.entityType === "Widget" && instance.Widget 
        !== undefined) {
      instance.entity = instance.Widget;
    } else if (instance.entityType === "Gizmo" && in
               stance.Gizmo !== undefined) {
      instance.entity = instance.Gizmo;
    }
  }
});
```

要查询关联，请执行以下操作：

```js
const widget = await Widget.create({ sku: "WID-1" });
const review = await widget.createReview({ message: "it works!" });
// the following should be true
console.log(review.entityId === widget.id);
```

多态允许我们通过调用实例方法`getEntity`来检索小部件或小玩意，而无需在我们的查询中预先确定：

```js
const entity = await review.getEntity();
// widget and entity should be the same object and return 
   "true" for deep comparison checking
const isDeepEqual = require('deep-equal');
console.log(isDeepEqual(widget, entity));
```

为了懒加载我们的数据，我们将包括关联模型，就像任何其他懒加载查询一样：

```js
const reviews = await Review.findAll({
    include: [Widget, Gizmo]
});
for (const review of reviews) {
    console.log('Found a review with the following entity: 
                 ', review.entity.toJSON());
}
```

`afterFind`钩子将自动将`create`实例关联到每个评论的`entity`键。

由于我们引用了多个表到一个目标列，所以在查询连接模型时需要格外小心。例如，如果`Widget`和`Gizmo`都有一个 ID 为`5`，并且一个评论有`entityType`为`Gizmo`，如果我们查询带有`Review.findAll({ include: Widget })`的评论，那么`Widget`实例将被懒加载到`Review`中，无论`entityType`是什么。

Sequelize 不会自动从模型名称中推断实体的类型。幸运的是，我们的`afterFind`生命周期事件将正确分配实体值。建议始终使用抽象方法（例如，`getEntity`）而不是 Sequelize 的混入（例如，`getWidget`、`getGizmo`等），以避免歧义。

到目前为止，我们已经展示了一个一对多关系的示例，但多对多关系呢？使用之前的示例模型，我们可以添加一个名为`Categories`的关联模型，其外观如下：

```js
const Category = sequelize.define('Category', {
    name: DataTypes.STRING
}, {
    instanceMethods: {
        getEntities(options) {
            const widgets = await this.getWidgets(options);
            const gizmos = await this.getGizmos(options);
            return [].concat(widgets, gizmos);
        }
    }
});
```

现在，我们可以通过分配两个外键列来创建用于多对多关系的连接模型：

```js
const CategoryEntity = sequelize.define('CategoryEntity', {
    categoryId: {
        type: DataTypes.INTEGER,
        unique: 'ce_unique_constraint'
    },
    entityId: {
        type: DataTypes.INTEGER,
        unique: 'ce_unique_constraint',
        references: null
    },
    entityType: {
        type: DataTypes.STRING,
        unique: 'ce_unique_constraint'
    }
});
```

`ce_unique_constraint`行将告诉 Sequelize 这三个属性属于同一个复合唯一键。`entityId`的空引用将确保 Sequelize 不会为该列创建引用约束。

接下来，我们可以定义`Widget`和`Gizmo`的关系以及一个辅助方法。我们将分配一个公共参数配置以及一个用于修改作用域的函数：

```js
const throughJunction = {
    through: {
      model: CategoryEntity,
      unique: false
    },
    foreignKey: 'entityId',
    constraints: false
};
function scopeJunction(scope) {
    let opts = throughJunction;
    opts.through.scope = {
        entityType: scope
    };
    return opts;
}
```

然后，我们可以将我们的关系分配给适用的模型：

```js
Widget.belongsToMany(Category, scopeJunction('Widget'));
Category.belongsToMany(Widget, throughJunction);

Gizmo.belongsToMany(Category, scopeJunction('Gizmo'));
Category.belongsToMany(Gizmo, throughJunction);
```

调用一个方法，例如`widget.getCategories()`，将执行一个类似于以下查询的操作：

```js
SELECT
    `Category`.`id`,
    `Category`.`name`,
    `CategoryEntity`.`categoryId` AS `CategoryEntity.
     categoryId`,
    `CategoryEntity`.`entityId` AS `CategoryEntity.entityId`,
    `CategoryEntity`.`entityType` AS `CategoryEntity.
     entityType`,
FROM `Categories` AS `Category`
INNER JOIN `CategoryEntities` AS `CategoryEntity` ON
    `Category`.`id` = `CategoryEntity`.`categoryId` AND
    `CategoryEntity`.`entityId` = 1 AND
    `CategoryEntity`.`entityType` = 'Widget';
```

现在我们已经学会了如何使用 Sequelize 操作关联和关系，我们可以开始对我们的 Avalon 航空公司项目进行一些修改。

# 将关联应用于 Avalon 航空公司

幸运的是，这个项目的模型很简单，不需要像定义一个超级多对多关系那样付出大量的努力。在接下来的几个模型更新中，这本书将展示只需要修改`class`模型块的内容（每个文件中的其余内容应保持不变）。

从字母顺序开始，我们将通过添加与`FlightSchedule`的关系来修改`models/airplane.js`文件的`class`块：

```js
class Airplane extends Model {
  static associate(models) {
this.FlightSchedules =
this.hasMany(models.FlightSchedule);
  }
};
```

接下来，我们可以编辑 `models/boardingticket.js` 文件的 `class` 块并添加 `Customer` 和 `FlightSchedule` 关联：

```js
class BoardingTicket extends Model {
  static associate(models) {
    this.Customer = this.belongsTo(models['Customer']);
    this.FlightSchedule = this.belongsTo(models['FlightSchedule']);
  }
};
```

客户现在将有许多登机牌；`models/customer.js` 文件的 `class` 块现在应该看起来像这样：

```js
class Customer extends Model {
  static associate(models) {
this.BoardingTickets = 
this.hasMany(models.BoardingTicket);
  }
};
```

飞行计划将属于特定的飞机，并且将有许多登机牌。我们可以编辑 `models/flightschedule.js` 文件的 `class` 块以匹配以下示例：

```js
class FlightSchedule extends Model {
  static associate(models) {
    this.Airplane = this.belongsTo(models['Airplane']);
    this.BoardingTickets = this.hasMany(models['BoardingTicket']);
  }
};
```

由于我们在初始化 Sequelize 时没有执行 `sync({ force: true })`，我们需要生成一个迁移文件并为相应的模型添加必要的引用。我们可以使用 Sequelize CLI 工具通过 `migration:generate` 子命令来生成一个新的迁移文件：

```js
sequelize migration:generate --name add-references
```

Sequelize 将通过类似以下的消息通知你它已生成迁移文件：

```js
New migration was created at /Users/book/migrations/20211031155604-add-references.js .
```

文件名上的前缀数字将不同于你屏幕上显示的数字，但如果我们查看 `migrations` 目录，我们将看到一个新的生成文件。我们可以快速覆盖文件的内容。

在文件顶部，我们将希望包含 `DataTypes` 并开始迁移的 `up` 块：

```js
const { DataTypes } = require("@sequelize/core");
module.exports = {
  up: async (queryInterface, Sequelize) => { 
```

我们现在可以通过先添加列然后添加约束来为 `FlightSchedule` 模型添加对 `Airplane` 模型的引用：

```js
    await queryInterface.addColumn('FlightSchedules', 'AirplaneId', {
      type: DataTypes.INTEGER,
    });
    await queryInterface.addConstraint('FlightSchedules', {
      type: 'foreign key',
      fields: ['AirplaneId'],
      references: {
        table: 'Airplanes',
        field: 'id'
      },
      name: 'fkey_flight_schedules_airplane',
      onDelete: 'set null',
      onUpdate: 'cascade'
    });
```

接下来，我们希望对 `BoardingTicket` 及其相关模型 `Customer` 和 `FlightSchedule` 做同样的处理：

```js
    await queryInterface.addColumn('BoardingTickets', 'CustomerId', {
      type: DataTypes.INTEGER,
    }
);    await queryInterface.addConstraint('BoardingTickets', {
      type: 'foreign key',
      fields: ['CustomerId'],
      references: {
        table: 'Customers',
        field: 'id'
      }
,      name: 'fkey_boarding_tickets_customer',
      onDelete: 'set null',
      onUpdate: 'cascade'
    });
    await queryInterface.addColumn('BoardingTickets',  
                                   'FlightScheduleId', {
      type: DataTypes.INTEGER,
    });
    await queryInterface.addConstraint('BoardingTickets', {
      type: 'foreign key',
      fields: ['FlightScheduleId'],
      references: {
        table: 'FlightSchedules',
        field: 'id'
      },
      name: 'fkey_boarding_tickets_flight_schedule',
      onDelete: 'set null',
      onUpdate: 'cascade'
    });
```

现在，我们可以关闭 `up` 块并开始我们的 `down` 块以支持迁移反转：

```js
  },
  down: async (queryInterface, Sequelize) => {
```

我们需要首先删除约束，然后是列，最后关闭 `down` 块和导出对象，如下所示：

```js
    await queryInterface.removeConstraint(
      'FlightSchedules', 'fkey_flight_schedules_airplane'
    );
    await queryInterface.removeConstraint(
      'BoardingTickets', 'fkey_boarding_tickets_customer'
    );
    await queryInterface.removeConstraint(
      'BoardingTickets', 
      'fkey_boarding_tickets_flight_schedule'
    );
    await queryInterface.removeColumn('FlightSchedules', 
       'AirplaneId');
    await queryInterface.removeColumn('BoardingTickets', 
       'CustomerId');
    await queryInterface.removeColumn('BoardingTickets', 
    'FlightScheduleId');
  }
};
```

在我们的控制台中，我们可以使用以下命令来迁移这些新更改：

```js
sequelize db:migrate
```

Sequelize 将确认迁移已完成，并且我们的模型通过关联正式相互关联。在本书的整个过程中，我们将使用本章学到的知识在查询中包含关联数据，但到目前为止，我们将继续进行下一课。

# 摘要

在本章中，我们讨论了使用关联属性以及一些高级选项和关系模式来定义模型的关系，并探讨了 eager loading 和 lazy loading 之间的区别。在本章的结尾，我们从前一章的“验证模型”中汲取经验，为 Avalon Airlines 项目添加了验证、关联和迁移。

注意

如果你曾在关联上遇到困难并需要快速参考，相关材料可以在这里找到：[`sequelize.org/docs/v6/core-concepts/assocs/`](https://sequelize.org/docs/v6/core-concepts/assocs/)。

在下一章中，我们将讨论 Sequelize 的钩子功能（也称为生命周期事件），如何为模型定义钩子，以及生命周期事件的一些良好用例。
