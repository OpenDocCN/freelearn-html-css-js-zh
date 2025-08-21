# 第六章：Sequelize

Sequelize 是一个基于承诺的 ORM，适用于 Node.js v4 及更高版本。这个 ORM 支持许多方言，比如：

+   `PostgreSQL`

+   `MySQL`

+   `SQLite`

+   `MSSQL`

这为事务提供了可靠的支持。使用 Sequelize，您可以使用`sequelize-typescript`，它提供了装饰器来放置在您的实体中，并管理模型的所有字段，带有类型和约束。

此外，Sequelize 来自许多钩子，为您提供了重要的优势，可以在事务的任何级别检查和操作数据。

在本章中，我们将看到如何使用`postgresql`配置您的数据库以及如何配置到您的数据库的连接。之后，我们将看到如何实现我们的第一个实体，这将是一个简单的`User`实体，然后如何为此实体创建一个提供者，以便将实体注入到`UserService`中。我们还将通过`umzug`看到迁移系统，以及如何创建我们的第一个迁移文件。

您可以查看存储库的`src/modules/database`，`src/modules/user`，`/src/shared/config`和`/src/migrations` `/migrate.ts`。

# 配置 Sequelize

为了能够使用 Sequelize，我们首先必须设置 sequelize 和我们的数据库之间的连接。为此，我们将创建`DatabaseModule`，其中将包含 sequelize 实例的提供者。

为了设置这个连接，我们将定义一个配置文件，其中将包含连接到数据库所需的所有属性。此配置将必须实现`IDatabaseConfig`接口，以避免忘记一些参数。

```js
export interface IDatabaseConfigAttributes {
    username: string;
    password: string;
    database: string;
    host: string;
    port: number;
    dialect: string;
    logging: boolean | (() => void);
    force: boolean;
    timezone: string;
}

export interface IDatabaseConfig {
    development: IDatabaseConfigAttributes;
}

```

此配置应该设置为以下示例，并通过环境变量或默认值设置参数。

```js
export const databaseConfig: IDatabaseConfig = {
    development: {
        username: process.env.POSTGRES_USER ||             'postgres',
        password: process.env.POSTGRES_PASSWORD || null,
        database: process.env.POSTGRES_DB || 'postgres',
        host: process.env.DB_HOST || '127.0.0.1',
        port: Number(process.env.POSTGRES_PORT) || 5432,
        dialect: 'postgres',
        logging: false,
        force: true,
        timezone: '+02:00',
    }
};

```

配置完成后，您必须创建适当的提供者，其目的是使用正确的配置创建 sequelize 实例。在我们的情况下，我们只是设置了环境配置，但您可以使用相同的模式设置所有配置，只需要更改值。

这个实例是让你了解应该提供的不同模型。为了告诉 sequelize 我们需要哪个模型，我们在实例上使用`addModels`方法，并传递一个模型数组。当然，在接下来的部分中，我们将看到如何实现一个新模型。

```js
export const databaseProvider = {
    provide: 'SequelizeInstance',
    useFactory: async () => {
        let config;
        switch (process.env.NODE_ENV) {
            case 'prod':
            case 'production':
            case 'dev':
            case 'development':
            default:
                config = databaseConfig.development;
        }

        const sequelize = new Sequelize(config);
        sequelize.addModels([User]);
        return sequelize;
    }
};

```

此提供者将返回 Sequelize 的实例。这个实例将有助于使用 Sequelize 提供的事务。此外，为了能够注入它，我们在`provide`参数中提供了令牌`SequelizeInstance`的名称，这将用于注入它。

Sequelize 还提供了一种立即同步模型和数据库的方法，使用`sequelize.sync()`。这种同步不应该在生产模式下使用，因为它每次都会重新创建一个新的数据库并删除所有数据。

我们现在已经设置好了我们的 Sequelize 配置，并且需要设置`DatabaseModule`，如下例所示：

```js
@Global()
@Module({
    providers: [databaseProvider],
    exports: [databaseProvider],
})
export class DatabaseModule {}

```

我们将`DatabaseModule`定义为`Global`，以便将其添加到所有模块作为相关模块，让您可以将提供者`SequelizeInstance`注入到任何模块中，如下所示：

```js
@Inject('SequelizeInstance`) private readonly sequelizeInstance

```

我们现在有一个完整的工作模块来访问我们数据库中的数据。

# 创建一个模型

设置好 sequelize 连接后，我们必须实现我们的模型。如前一节所示，我们告诉 Sequelize 我们将使用此方法`sequelize.addModels([User]);`来拥有`User`模型。

您现在看到了设置它所需的所有功能。

## @Table

这个装饰器将允许您配置我们对数据的表示，以下是一些参数：

```js
{

    timestamps:  true,
    paranoid:  true,
    underscored:  false,
    freezeTableName:  true,
    tableName:  'my_very_custom_table_name'
}

```

`timestamp`参数将告诉你想要有`updatedAt`和`deletedAt`列。`paranoid`参数允许你软删除数据而不是删除它以避免丢失数据。如果你传递`true`，Sequelize 将期望有一个`deletedAt`列以设置删除操作的日期。

`underscored`参数将自动将所有驼峰命名的列转换为下划线命名的列。

`freezTableName`将提供一种避免 Sequelize 将表名变为复数形式的方法。

`tableName`允许你设置表的名称。

在我们的案例中，我们只使用`timestamp: true, tableName: 'users'`来获取`updatedAt`和`createdAt`列，并将表命名为`users`。

## @column

这个装饰器将帮助定义我们的列。你也可以不传递任何参数，这样 Sequelize 将尝试推断列类型。可以推断的类型包括`string`、`boolean`、`number`、`Date`和`Blob`。

一些参数允许我们在列上定义一些约束。比如，假设`email`列，我们希望这个电子邮件是一个字符串，并且不能为空，所以这个电子邮件必须是唯一的。Sequelize 可以识别电子邮件，但我们必须告诉它如何验证电子邮件，通过传递`validate#isUnique`方法。

看一下下面的例子。

```js
@Column({
    type: DataType.STRING,
    allowNull: false,
    validate: {
        isEmail: true,
        isUnique: async (value: string, next: any): Promise<any> => {
            const isExist = await User.findOne({ where: { email: value }});
            if (isExist) {
                const error = new Error('The email is already used.');
                next(error);
            }
            next();
        },
    },
})

```

在前面的示例中，我们传递了一些选项，但我们也可以使用一些装饰器，如`@AllowNull(value: boolean)`，`@Unique`甚至`@Default(value: any)`。

为了设置一个`id`列，`@PrimaryKey`和`@AutoIncrement`装饰器是设置约束的一种简单方法。

## 创建用户模型

现在我们已经看到了一些有用的装饰器，让我们创建我们的第一个模型，`User`。为了做到这一点，我们将创建一个类，该类必须扩展自基类`Model<T>`，这个类需要为自身的模板值。

```js
export class User extends Model<User> {...}

```

现在我们添加了`@Table()`装饰器来配置我们的模型。这个装饰器接受与接口`DefineOptions`对应的选项，正如我们在***@Table 部分***中描述的，我们将传递 timestamp 为 true 和表的名称作为选项。

```js
@Table({ timestamp: true, tableName: 'users' } as IDefineOptions)
export class User extends Model<User> {...}

```

现在我们需要为我们的模型定义一些列。为此，`sequelize-typescript`提供了`@Column()`装饰器。这个装饰器允许我们提供一些选项来配置我们的字段。你可以直接传递数据类型`DataType.Type`。

```js
@Column(DataTypes.STRING)
public email: string;

```

你还可以使用***@Column 部分***中显示的选项来验证和确保电子邮件的数据。

```js
@Column({
    type: DataType.STRING,
    allowNull: false,
    validate: {
        isEmail: true,
        isUnique: async (value: string, next: any): Promise<any> => {
            const isExist = await User.findOne({
                where: { email: value }
            });
            if (isExist) {
                const error = new Error('The email is already used.');
                next(error);
            }
            next();
        },
    },
})
public email: string;

```

现在你知道如何设置列，让我们为简单的用户设置模型的其余部分。

```js
@Table(tableOptions)
export class User extends Model<User> {
    @PrimaryKey
    @AutoIncrement @Column(DataType.BIGINT)
    public id: number;

    @Column({
        type: DataType.STRING,
        allowNull: false,
    })
    public firstName: string;

    @Column({
        type: DataType.STRING,
        allowNull: false,
    })
    public lastName: string;

    @Column({
        type: DataType.STRING,
        allowNull: false,
        validate: {
            isEmail: true,
            isUnique: async (value: string, next: any): Promise<any> => {
                const isExist = await User.findOne({
                    where: { email: value }
                });
                if (isExist) {
                    const error = new Error('The email is already used.');
                    next(error);
                }
                next();
            },
        },
    })
    public email: string;

    @Column({
        type: DataType.TEXT,
        allowNull: false,
    })
    public password: string;

    @CreatedAt
    public createdAt: Date;

    @UpdatedAt
    public updatedAt: Date;

    @DeletedAt
    public deletedAt: Date;
}

```

在所有添加的列中，你可以看到`TEXT`类型的密码，但当然，你不能将密码存储为明文，所以我们必须对其进行哈希处理以保护它。为此，使用 Sequelize 提供的生命周期钩子。

## 生命周期钩子

Sequelize 提供了许多生命周期钩子，允许你在创建、更新或删除数据的过程中操作和检查数据。

以下是 Sequelize 中一些有用的钩子。

```js
  beforeBulkCreate(instances, options)
  beforeBulkDestroy(options)
  beforeBulkUpdate(options)

  beforeValidate(instance, options)
  afterValidate(instance, options)

  beforeCreate(instance, options)
  beforeDestroy(instance, options)
  beforeUpdate(instance, options)
  beforeSave(instance, options)
  beforeUpsert(values, options)

  afterCreate(instance, options)
  afterDestroy(instance, options)
  afterUpdate(instance, options)
  afterSave(instance, options)
  afterUpsert(created, options)

  afterBulkCreate(instances, options)
  afterBulkDestroy(options)
  afterBulkUpdate(options)

```

在这种情况下，我们需要使用`@BeforeCreate`装饰器来对密码进行哈希处理，并在存储到数据库之前替换原始值。

```js
@Table(tableOptions)
export class User extends Model<User> {
    ...
    @BeforeCreate
    public static async hashPassword(user: User, options: any) {
        if (!options.transaction) throw new Error('Missing transaction.');

        user.password = crypto.createHmac('sha256', user.password).digest('hex');
    }
}

```

之前写的`BeforeCreate`允许你在将对象插入到数据库之前覆盖用户的`password`属性值，并确保最低限度的安全性。

# 将模型注入到服务中

我们的第一个`User`模型现在已经设置好了。当然，我们需要将其注入到服务或甚至控制器中。要在任何其他地方注入模型，我们必须首先创建适当的提供者，以便将其提供给模块。

这个提供者将定义用于注入的密钥，并将`User`模型作为值，我们之前已经实现了这个模型。

```js
export const userProvider = {
    provide: 'UserRepository',
    useValue: User
};

```

要将其注入到服务中，我们将使用`@Inject()`装饰器，它可以使用前面示例中定义的字符串`UserRepository`。

```js
@Injectable()
export class UserService implements IUserService {
    constructor(@Inject('UserRepository') private readonly UserRepository: typeof User) { }
    ...
}

```

在将模型注入服务之后，您可以使用它来访问和操作数据。例如，您可以执行`this.UserRepository.findAll()`来在数据库中注册数据。

最后，我们必须设置模块以将`userProvider`作为提供者，该提供者提供对模型和`UserService`的访问。`UserService`可以导出，以便在另一个模块中使用，通过导入`UserModule`。

```js
@Module({
    imports: [],
    providers: [userProvider, UserService],
    exports: [UserService]
})
export class UserModule {}

```

# 使用 Sequelize 事务

您可能会注意到这行代码，`if (!options.transaction) throw new Error('Missing transaction.');`，在使用`@BeforeCreate`装饰的`hashPassword`方法中。如前所述，Sequelize 提供了对事务的强大支持。因此，对于每个操作或操作过程，您都可以使用事务。要使用 Sequelize 事务，请查看以下`UserService`的示例。

```js
@Injectable()
export class UserService implements IUserService {
    constructor(@Inject('UserRepository') private readonly UserRepository: typeof User,
                @Inject('SequelizeInstance') private readonly sequelizeInstance) { }
    ...
}

```

我们在本章中提到的模型和 Sequelize 实例都已注入。

要使用事务来包装对数据库的访问，您可以执行以下操作：

```js
public async create(user: IUser): Promise<User> {
    return await this.sequelizeInstance.transaction(async transaction => {
        return await this.UserRepository.create<User>(user, {
            returning: true,
            transaction,
        });
    });
}

```

我们使用`sequelizeInstance`创建一个新的事务，并将其传递给`UserRepository`的`create`方法。

# 迁移

使用 Sequelize，您可以同步模型和数据库。问题是，此同步将删除所有数据，以便重新创建表示模型的所有表。因此，此功能在测试中很有用，但在生产模式下则不适用。

为了操作数据库，您可以使用`umzung`，这是一个与框架无关的库和迁移工具，适用于 Nodejs。它与任何数据库都无关，但提供了一个 API，用于迁移或回滚迁移。

当您使用命令`npm run migrate up`时，它会执行`ts-node migrate.ts`，您可以将`up/down`作为参数传递。为了跟踪已应用的所有迁移，将创建一个名为`SequelizeMeta`的新表，并将所有已应用的迁移存储在此表中。

我们的迁移文件可以在存储库中找到，名称为`migrate.ts`。此外，所有迁移文件将存储在存储库示例的`migrations`文件夹中。

## 配置迁移脚本

为了配置 umzung 实例，您可以设置一些选项：

+   `storage`，对应于我们的`sequelize`字符串键

+   `storageOptions`，它将使用 Sequelize，并且您可以在此选项中更改用于存储已应用迁移的名称的列的默认名称`modelName`，`tableName`和`columnName`属性。

还可以进行其他一些配置，以设置`up`方法名称和`down`方法名称，传递日志函数。`migrations`属性将允许您提供一些参数以传递给 up/down 方法，并提供要应用的迁移的路径以及适当的模式。

```js
const umzug = new Umzug({
    storage: 'sequelize',
    storageOptions: { sequelize },

    migrations: {
        params: [
            sequelize,
            sequelize.constructor, // DataTypes
        ],
        path: './migrations',
        pattern: /\.ts$/
    },

    logging: function () {
        console.log.apply(null, arguments);
    }
});

```

## 创建迁移

要执行迁移脚本，请提供要应用的迁移。假设您想使用迁移创建`users`表。您必须设置`up`和`down`方法。

```js
export async function up(sequelize) {
    // language=PostgreSQL
    sequelize.query(`
        CREATE TABLE "users" (
            "id" SERIAL UNIQUE PRIMARY KEY NOT NULL,
            "firstName" VARCHAR(30) NOT NULL,
            "lastName" VARCHAR(30) NOT NULL,
            "email" VARCHAR(100) UNIQUE NOT NULL,
            "password" TEXT NOT NULL,
            "birthday" TIMESTAMP,
            "createdAt" TIMESTAMP NOT NULL,
            "updatedAt" TIMESTAMP NOT NULL,
            "deletedAt" TIMESTAMP
        );
    `);

    console.log('*Table users created!*');
}

export async function down(sequelize) {
    // language=PostgreSQL
    sequelize.query(`DROP TABLE users`);
}

```

在每个方法中，参数将是`sequelize`，这是配置文件中使用的实例。通过此实例，您可以使用查询方法来编写我们的 SQL 查询。在前面的示例中，函数`up`将执行查询以创建`users`表。`down`方法的目的是在回滚时删除此表。

# 总结

在本章中，您已经看到了如何通过实例化 Sequelize 实例来设置与数据库的连接，并使用工厂直接在另一个地方注入实例。

另外，您已经看到了 sequelize-typescript 提供的装饰器，以便设置一个新的模型。您还看到了如何在列上添加一些约束，以及如何在保存之前使用生命周期钩子来对密码进行哈希处理。当然，这些钩子可以用来验证一些数据或在执行其他操作之前检查一些信息。但您也已经看到了如何使用`@BeforeCreate`钩子。因此，您已经准备好使用 Sequelize 事务系统。

最后，您已经看到了如何配置 umzug 来执行迁移，并且如何创建您的第一个迁移以创建用户表。

在下一章中，您将学习如何使用 Mongoose。
