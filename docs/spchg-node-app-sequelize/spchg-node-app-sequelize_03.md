

# 第三章：验证模型

在数据库中保持一致性和完整性非常重要。数据库通常使用某种形式的约束规定来确保一致性。通常，这些约束包括检查值范围，例如最小字符串长度、唯一性或存在性。数据库的完整性涉及管理共生记录之间的关联和关系。这包括级联更新和删除引用记录（例如，当引用记录被删除时，将关联的标识列设置为`NULL`）。一致性和完整性不是相互排斥的，但这两个模式有助于确保组织。

注意

一致性这个术语指的是确保只有有效数据会被写入和从数据库中读取（尤其是在并发访问数据的情况下）。完整性指的是在插入或读取之前符合一组规则、约束或触发器的数据。

虽然大多数数据库引擎都处理一致性和完整性，但在一致性方面存在一些限制。如果您想要对数据库范围之外的第三方源执行验证，您可能需要为数据库构建（或安装）一个扩展来添加支持，或者使用一个中央代码库来帮助管理这些验证。

Sequelize 为各种数据类型提供了内置验证，以帮助提高项目的易用性。某些验证需要手动配置，例如检查文本值是否匹配电子邮件模式，或者某些验证需要手动输入，例如数值（或日期）范围。

在 Sequelize 中，可以使用两种方法执行验证：

+   我们可以在涉及多个属性的整个记录上执行验证

+   我们可以为每个特定的属性调用验证

在本章中，我们将探讨 Sequelize 如何执行验证，以在数据库中保持一致性和完整性。本章将涵盖以下主题：

+   使用验证作为约束

+   创建自定义验证方法

+   在执行异步操作时执行验证

+   处理验证错误

注意

Sequelize 将内部使用一个名为`validator.js`的验证库。本章将介绍 Sequelize 明确扩展的验证。有关可以使用完整验证列表，您可以参考`validator.js`存储库[`github.com/validatorjs/validator.js`](https://github.com/validatorjs/validator.js)。

# 技术要求

您可以在以下位置找到本章的代码文件：[`github.com/PacktPublishing/Supercharging-Node.js-Applications-with-Sequelize/tree/main/ch3`](https://github.com/PacktPublishing/Supercharging-Node.js-Applications-with-Sequelize/tree/main/ch3)

# 使用验证作为约束

有些验证 Sequelize 既用作验证又用作约束。这些参数可以在属性的选项中作为`validate`参数的兄弟进行配置。约束由数据库定义并受保护，而验证将由 Sequelize 和 Node.js 运行时独家处理。以下是 Sequelize 提供的约束列表。

## allowNull

`allowNull`选项将确定是否将`NOT NULL`应用于数据库列的定义。默认值是`true`，这将允许列具有`null`值。在使用带有`allowNull`约束的验证时，有几个注意事项需要记住：

+   如果将`allowNull`参数设置为`false`且属性的值为`null`，则自定义验证将不会运行。相反，将返回一个**ValidationError**，而无需向数据库发出请求。

+   如果`allowNull`参数设置为`true`且属性的值为`null`，则内置验证器将不会被调用，但自定义验证器仍然会执行。

以下是对各种验证状态的解释，这些状态将根据`allowNull`参数的设置导致 Sequelize 相应地表现：

```js
User.init({
  age: {
    type: Sequelize.INTEGER,
    allowNull: true,
    // if the age value is null then this will be ignored
    validate: {
      min: 1
    }
  },
  name: {
    type: DataTypes.STRING,
    allowNull: true,
    validate: {
      // even if the name's value is null, the
         customValidator will still be invoked
      customValidator(value) {
        if (value === null && (this.age === null || 
            this.age < 18)) {
          throw new Error("A name is required unless the 
                           user is under 18 years old");
        }
      }
    }
  },
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    validate: {
      // if the email value is null then this will not be 
         invoked
      customValidator(value) {
        // ...
      }
    }
  }
}, { sequelize });
```

在前面的示例中，在第一列`age`中，Sequelize 将执行验证检查以确保数值不低于一。下一列`name`将调用一个自定义验证函数，该函数检查属性的新值是否为`null`，如果是，则检查用户的年龄。最后一列`email`演示了如果将`allowNull`标志设置为`false`且值本身为`null`，Sequelize 将不会调用验证。

您可以通过调整属性验证配置中的`notNull`参数来自定义`NOT NULL`错误，如下所示：

```js
User.init({
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    validate: {
      notNull: {
        msg: 'Please enter your e-mail address'
      }
    }
  }
}, { sequelize });
```

否则，Sequelize 将返回数据库发送的错误消息。

## unique

将此参数设置为`true`将在数据库中为适用列构建一个唯一约束，如果您正在使用 Sequelize 的`sync`选项。如果存在唯一约束违规，Sequelize 将返回一个类型为`SequelizeUniqueConstraintError`的错误。以下是如何使用`unique`的快速示例（您还可以为唯一性允许可空值）：

```js
MyModel.init({
  email: {
    type: DataTypes.STRING,
    // by default allowNull is true
    allowNull: false,
    unique: true
  }
}, { sequelize });
```

作为一般规则，您应该在适用的情况下使用约束而不是验证，因为此选项也将应用于数据库。在约束不适用的场合，我们可以使用 Sequelize 的内置验证之一。

注意

当在`unique`属性上将`allowNull`设置为`true`时，数据库将允许在该属性上具有相同`NULL`值的多个记录。这是数据库管理系统方面的故意行为，可以通过显式地向`unique`索引添加约束来缓解，如下所示：

`CREATE UNIQUE INDEX idx_tbl_uniq ON tbl (a, (b IS NULL)) WHERE b IS NULL`

## 内置验证

这些验证是在 Node.js 运行时内执行的，而不是从数据库中执行。Sequelize 将通过其自己的验证器集扩展 `validator.js` 的功能。

### is (regex), not (notRegex), 和 equals

`is` 和 `not` 验证参数可以是字面量正则表达式或一个数组，其中第一个条目是正则表达式的字符串字面量，第二个条目是正则表达式标志。`equals` 参数是一个字符串值，用于执行严格的匹配检查。

以下是一个如何使用这三个验证器的模型示例：

```js
MyModel.init({
  foo: {
    type: DataTypes.STRING,
    validate: {
      is: /^[a-z]+$/i
      // can also be written as:
      // is: ['^[a-z]+$', 'i']
    }
  },
  bar: {
    type: DataTypes.STRING,
    validate: {
      not: /^[a-z]+$/i
      // can also be written as:
      // not: ['^[a-z]+$', 'i']
    } 
  },
  foobar: {
    type: DataTypes.STRING,
    validate: {
      // ensure 'foobar' is always equaled to 'static
         value'
      equals: 'static value'
    }
  }
}, { sequelize });
```

### isEmail

此验证将确保属性值符合 RFC 2822 规则，该规则可在 [`datatracker.ietf.org/doc/html/rfc2822`](https://datatracker.ietf.org/doc/html/rfc2822) 查阅。

### isUrl

这将验证属性值是否是一个实际的 URL，具有各种协议、主机名（IP 和 FQDN）以及最大长度。

### isIP, isIPv4, 或 isIPv6

这验证属性值是否与 IP 值的外观相匹配。`isIP` 验证接受 v4 和 v6 格式。

### isAlphanumeric, isNumeric, isInt, isFloat, 和 isDecimal

所有用于验证的输入都作为字面量字符串发送到 `validator.js` 库。这些验证器将确保输入可以解析为相应的验证。

### max 或 min

这些仅适用于数值属性。它们分别为属性验证添加最大或最小数值值。

### isLowercase 或 isUppercase

这些检查属性值中的每个字母是否都使用了正确的格式。

### isNull, notNull, 或 notEmpty

这验证值是否为 `null` 或不是。`notEmpty` 验证器将验证值中是否有任何空格、制表符或换行符。

### contains, notContains, isIn, 或 notIn

这些相关验证器将对值执行子字符串检查。相关验证器接受数组参数内的任何值。例如，请参见以下内容：

```js
MyModel.init({
  foo: {
    type: DataTypes.STRING,
    validate: {
      isIn: [['red', 'yellow', 'green']]
    }
  },
  bar: {
    type: DataTypes.STRING,
    validate: {
      contains: 'foo'
    } 
  }
}, { sequelize });
```

### len

`len` 验证器接受一个包含两个参数的数组作为其输入。这些参数用于检查值长度与最小和最大数值分别对应。要创建一个具有最小长度为 `1` 和最大长度为 `40` 的值长度验证，它看起来如下所示：

```js
MyModel.init({
  foo: {
    type: DataTypes.STRING,
    validate: {
      len: [1, 40]
    }
  }
}, { sequelize });
```

### isUUID

此验证器可以检查一个值是否符合唯一标识符的要求。您可以指定版本（3、4 或 5）作为输入参数，或者使用字面量字符串值 `all` 以接受任何 UUID 版本。

### isDate, isAfter, 或 isBefore

`isDate` 验证器将确定一个值是否类似于日期。`isAfter` 和 `isBefore` 验证器将执行与您尝试验证的日期的时间比较。默认输入比较是 `new Date()`。以下是一个快速示例：

```js
MyModel.init({
  expiration: {
    type: DataTypes.DATE,
    validate: {
      isAfter: '2060-01-01'
      // for "now"
      // isAfter: true
    }
  }
}, { sequelize });
```

备注

`isBefore`和`isAfter`的输入是一个符合任何适用日期的字符串，该日期可以被 JavaScript 解析。有关兼容格式的示例，您可以参考此链接：[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/parse`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/parse)。

现在我们已经通过几个示例了解了如何将验证应用于 Sequelize 模型的属性，我们可以更新 Avalon Airlines 项目中的几个文件。

## 在我们的项目中应用验证

在以下示例中，我们将为我们的 Airplane 模型的`planeModel`和`totalSeats`属性添加验证。我们可以从打开`models/airplane.js`文件并添加以下验证开始：

+   对于`planeModel`属性，添加一个`notEmpty`验证，因为所有飞机型号都需要一个非`null`且非空字符串的值。

+   在`totalSeats`属性上，添加一个最小验证值为`1`作为参数值，因为每架飞机都必须至少有一个座位可供顾客使用。

更新后的文件应类似于以下内容：

```js
const { Model } = require('@sequelize/core');
module.exports = (sequelize, DataTypes) => {
  class Airplane extends Model {
    static associate(models) {
    }
  };
  Airplane.init({
    planeModel: {
      type: DataTypes.STRING,
      validate: {
        notEmpty: {
          msg: 'Plane types should not be empty'
        }
      }
    },
    totalSeats: {
      type: DataTypes.INTEGER,
      validate: {
        min: {
          args: 1,
          msg: 'A plane must have at least one seat'
        }
      }
    }
  }, {
    sequelize,
    modelName: 'Airplane',
  });
  return Airplane;
};
```

接下来，我们将想要修改`models/boardingticket.js`文件，并添加以下`notEmpty`验证器：

```js
const { Model } = require('@sequelize/core');
module.exports = (sequelize, DataTypes) => {
  class BoardingTicket extends Model {
    static associate(models) {
    }
  };
  BoardingTicket.init({
    seat: {
      type: DataTypes.STRING,
      validate: {
        notEmpty: {
          msg: 'Please enter in a valid seating arrangement'
        }
      }
    }
  }, {
    sequelize,
    modelName: 'BoardingTicket',
  });
  return BoardingTicket;
};
```

在本节中最后要编辑的文件将是`models/customer.js`文件。名称属性将需要一个`notEmpty`验证器，而电子邮件属性将需要一个`isEmail`验证器，如下所示：

```js
const { Model } = require('@sequelize/core');
module.exports = (sequelize, DataTypes) => {
  class Customer extends Model {
    static associate(models) {
    }
  };
  Customer.init({
    name: {
      type: DataTypes.STRING,
      validate: {
        notEmpty: true,
        msg: 'A name is required for the customer'
      }
    },
    email: {
      type: DataTypes.STRING,
      validate: {
        isEmail: true,
        msg: 'Invalid email format for the customer'
      }
    }
  }, {
    sequelize,
    modelName: 'Customer',
  });
  return Customer;
};
```

在经过内置验证列表之后，我们现在可以学习如何构建自己的验证，以及如何在整个模型中使用自定义验证。

# 创建自定义验证方法

Sequelize 允许我们通过向属性或模型选项的`validate`参数（`Model.init()`函数的第二个输入参数）中添加一个函数来创建自己的验证。

如果我们想要创建自己的验证来限制用户不能使用`password`作为密码，我们会编写一个类似以下的解决方案：

```js
MyModel.init({
  password: {
    type: DataTypes.STRING,
    validate: {
      notLiteralPassword(value) {
        if (value === 'password') {
          throw new Error("Your password cannot be 
                           'password'");
        }
      }
    }
  }
}, { sequelize });
```

尽管你可以在自定义属性验证器中的其他属性上检查值，但在涉及多个属性进行验证时，声明一个模型自定义验证器被认为是良好的实践，我们将在稍后演示这一点。

我们还有一个模型文件需要添加验证。`models/flightschedule.js`文件需要验证出发机场不能与目的地机场相同。首先，我们需要导入 Sequelize 并添加一个可用机场列表：

```js
const { Model } = require('@sequelize/core');
const availableAirports = [
  'MIA',
  'JFK',
  'LAX'
];
```

接下来，添加模块导出和模型类扩展行：

```js
module.exports = (sequelize, DataTypes) => {
  class FlightSchedule extends Model {
    static associate(models) {
    }
  };
```

## 创建自定义属性验证器

然后，我们可以使用具有相关验证的属性定义初始化我们的模型。我们希望向`originAirport`和`destinationAirport`属性添加`isIn`验证器：

```js
  FlightSchedule.init({
    originAirport: {
      type: DataTypes.STRING,
  // examples of custom attribute validators
      validate: {
        isIn: {
          args: [availableAirports],
          msg: 'Invalid origin airport'
        }
      }
    },
    destinationAirport: {
      type: DataTypes.STRING,
      validate: {
        isIn: {
          args: [availableAirports],
          msg: 'Invalid destination airport'
        }
      }
    },
    departureTime: {
      type: DataTypes.DATE,
      validate: {
        isDate: {
          args: true,
          msg: 'Invalid departure time'
        }
      }
    }
  }, {
    sequelize,
    modelName: 'FlightSchedule',
    validate: {
```

### 添加自定义模型验证器

现在，我们可以在这里添加我们的自定义模型验证器。我们将创建一个函数，该函数将检查值是否与 `originAirport` 和 `destinationAirport` 属性匹配。如果两个值都相同，我们将标记目的地为无效并抛出错误：

```js
      validDestination() {
        const hasAirportValues = this.originAirport !== 
        null && this.destinationAirport !== null;
        const invalidDestination = this.originAirport === 
        this.destinationAirport;;
        if (hasAirportValues && invalidDestination) {
          throw new Error("The destination airport cannot 
                           be the same as the origin");
        }
      }
```

对于最后一步，我们将关闭任何对象或函数，并将类返回到导出：

```js
    }
  });
  return FlightSchedule;
};
```

你可能已经注意到，如果两个值都是 `null`，我们就会跳过 `validDestination` 验证器。`isIn` 验证器仍然会执行并返回一个错误，因为没有有效的值。

# 在执行异步操作时执行验证

有时，你的验证可能需要你获取关联模型的记录，调用第三方应用程序，或进行其他形式的请求，这些请求需要等待响应。

假设我们处于这样一种情况，我们必须确保在创建或更新客户的会员积分之前，有一个完成且活跃的付款。只要任何付款仍然被认为是良好的，该客户应该能够更新他们的会员资格。我们会使用 `async` 和 `await` 关键字来帮助我们执行这些请求并等待验证的响应：

```js
Membership.init({
  points: {
    type: DataTypes.INTEGER,
  }
}, {
  sequelize,
  validate: {
    async accountIsActive() {
      const payments = await Payments.find({
        where: { status: 'complete', expired: false }
      });
      if (payments.length < 1) {
        throw new Error("Invalid membership");
      }
    }
  }
});
```

注意

`async` 和 `await` 关键字同样适用于自定义属性验证器。

需要注意的是，如果你在生命周期事件中运行 Sequelize 查询，那么这些查询将在与父模型不同的事务下执行。例如，如果我们开始了一个事务，并在其作用域内创建了付款条目，然后尝试创建会员条目，那么 `await Payments.find(…)` 这一行将无法看到最近创建的记录。为了解决这个问题，我们可以在调用 `create` 时将 Sequelize 事务传递给 `transaction` 参数。以下是一个非常通用但高级的示例：

```js
const tx = await sequelize.transaction();
try {
  await Payment.create({ status: 'complete', expired: false });
  await Membership.create({
    points: 100,
    // without the following line the `await    
Payments.find()` call in 
    // ...accountIsActive will not find the previously
created entry
    transaction: tx
  });
  await t.commit();
} catch (err) {
  await t.rollback();
}
```

# 处理验证错误

使用来自 *创建自定义验证方法* 部分的 `FlightSchedule`，我们将介绍如何在调用 `validate`、`update` 和 `create` 方法时处理验证错误。假设我们调用了一个 `createFlightSchedule` 方法，其外观如下：

```js
const { ValidationError } = require('@sequelize/core');
// other imports and code...
async function createFlightSchedule() {
  try {
    await FlightSchedule.create({
      originAirport: 'JAX',
      destinationAirport: 'JFK',
      departureTime: '2030-01-01T00:00:00.000+00:00'
    });
  } catch (err) {
    if (err instanceof ValidationError) {
      console.log(err.errors);
    } else {
      console.log(err);
    }
  }
}
  return FlightSchedule;
};
```

默认情况下，从 `ValidationError` 返回的错误应该类似于以下内容（你可能还会看到列出的其他字段）：

```js
[
  ValidationErrorItem {
    message: 'Invalid origin airport',
    type: 'Validation error',
    path: 'originAirport',
    value: 'JAX',
    origin: 'FUNCTION',
    instance: FlightSchedule {
      dataValues: [Object],
      _previousDataValues: [Object],
      uniqno: 1,
      _changed: [Set],
      _options: [Object],
      isNewRecord: true
    },
    validatorKey: 'isIn',
    validatorName: 'isIn',
    validatorArgs: [ [Array] ],
    original: Error: Invalid origin airport {
      validatorName: 'isIn',
      validatorArgs: [Array]
    }
  }
] 
          origin airport'}
]
```

或者，我们可以在尝试使用实例的 `validate()` 方法创建或修改记录之前手动检查验证，如下所示：

```js
async function createFlightSchedule() {
  try {
    const schedule = FlightSchedule.build({
      originAirport: 'JAX',
      destinationAirport: 'JFK',
      departureTime: '2030-01-01T00:00:00.000+00:00'
    });
    await schedule.validate();
  } catch (err) {
    console.log(err);
  }
}
```

结果将返回一个类似于以下错误对象：

```js
{
  errors: [
    ValidationErrorItem {
      message: 'Invalid origin airport',
      type: 'Validation error',
      path: 'originAirport',
      value: 'JAX',
      origin: 'FUNCTION',
      instance: [FlightSchedule],
      validatorKey: 'isIn',
      validatorName: 'isIn',
      validatorArgs: [Array],
      original: [Error]
    }
  ]
} 
                          the same as the origin']
}
```

因此，现在我们的数据已经 *一致*，那么 *完整性* 呢？在下一章中，我们将介绍 Sequelize 为我们提供创建（以及操作）关联和模型之间各种关联方式的功能。

# 摘要

在本章中，我们通过使用 Sequelize 内置验证器以及添加我们自己的自定义验证方法，对我们的模型添加了验证。然后，我们转向在自定义验证中处理和执行异步方法。一旦我们能够正确调用验证，我们就能练习处理验证错误。

在下一章中，我们将介绍另一个为我们的数据库添加一致性和完整性的部分，即处理和关联我们的模型。验证将确保数据库行级别的完整性，而关联可以用来确保跨表和其他行的完整性。
