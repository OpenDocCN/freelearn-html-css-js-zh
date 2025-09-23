# 第三章 职位板

在本章中，我们将构建一个职位板应用程序。用户将能够创建个人资料并填写不同类型的信息，例如工作经历、他们参与的项目、认证信息，甚至是与教育相关的信息。此外，公司也将能够发布职位空缺，用户可以申请。

# 设置基础应用程序

在许多情况下，大多数开发者已经为他们使用的 Node 应用程序设置了他们自己的样板代码。这样做的一个原因可能是存在多种正确的方法来做事情。通常，你的样板代码将涵盖应用程序的初始功能，例如用户架构、登录和注册。

由于我们已经从最初的两章中建立了一个坚实的基础，我们可以重用大量的代码库。我已经创建了一个简单的基应用程序，我们可以从这里开始。只需按照以下步骤克隆项目：

1.  从 GitHub 克隆项目[`github.com/robert52/express-api-starter`](https://github.com/robert52/express-api-starter)。

1.  将您的样板项目重命名为`jobboard`。

1.  如果您愿意，可以通过运行以下命令停止指向初始 Git 仓库：

    ```js
    git remote remove origin

    ```

1.  跳转到您的工作目录：

    ```js
    cd jobboard

    ```

1.  安装所有依赖项：

    ```js
    npm install

    ```

1.  创建一个开发配置文件：

    ```js
    cp config/environments/example.js config/environments/development.js

    ```

您的配置文件`jobboard/config/environments/development.js`应类似于以下内容：

```js
'use strict';

module.exports = {
  port: 3000,
  hostname: '127.0.0.1',
  baseUrl: 'http://localhost:3000',
  mongodb: {
    uri: 'mongodb://localhost/jobboard_dev_db'
  },
  app: {
    name: 'Job board'
  },
  serveStatic: true,
  session: {
    type: 'mongo',                          
    secret: 'someVeRyN1c3S#cr3tHer34U',
    resave: false,                          
    saveUninitialized: true                 
  }
};
```

# 修改用户后端

用户后端逻辑需要稍作修改以适应我们的需求。例如，我们需要为我们的用户提供角色。我们将在讨论用户模型时详细说明这一点。我们必须添加授权策略。我们还需要为我们的用户提供一个个人资料。

## 修改用户模型

为了支持多种账户类型并最终为用户分配角色，我们需要对用户模型进行一些修改。这将告诉我们用户是注册了简单账户，其中他们可以定义带有工作经历的个人资料，还是创建一个想要发布职位机会的公司。

角色将定义用户可以执行的操作。例如，对于一家公司，我们可以有一个拥有完全控制账户的公司所有者，或者我们可以有一个是该公司的成员并发布可用的职位空缺的用户。

让我们使用以下内容修改`jobboard/app/models/user.js`中的用户架构：

```js
var UserSchema = new Schema({
  email:  {
    type: String,
    required: true,
    unique: true
  },
  name: {
    type: String
  },
  password: {
    type: String,
    required: true,
    select: false
  },
  passwordSalt: {
    type: String,
    required: true,
    select: false
  },
  active: {
    type: Boolean,
    default: true
  },
  roles: {
    type: [
      {
        type: String,
        enum: ['user', 'member', 'owner']
      }
    ],
    default: ['user']
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

我们在我们的用户架构中添加了一个额外的字段，更确切地说，是`roles`，它包含用户可以执行的操作。您可以将任何类型的角色添加到由枚举验证定义的有效角色列表中。

## 授权策略

为了授权我们的用户执行请求的操作，我们必须检查他们是否有权这样做。例如，只有公司所有者才能更改公司信息或添加新成员。

在项目的初始阶段，我喜欢将我的策略保持得尽可能简单和分离。换句话说，我不喜欢创建一个管理一切的东西，而是使用简单的函数来检查不同的场景。

让我们看看一个授权策略。创建一个名为 `jobboard/app/middlewares/authorization.js` 的文件，并添加以下内容：

```js
module.exports.onlyMembers = authorizeOnlyToCompanyMembers;

function authorizeOnlyToCompanyMembers(req, res, next) {
  // check if user is member of company
  const isMember = req.resources.company.members.find((member) => {
    return member.toString() === req.user._id.toString();
  });

  if (!isMember) {
    return res.status(403).json({ message: 'Unauthorized' });
  }

  next();
}
```

这个简单的函数将检查公司的所有者是否是认证用户。前面的策略可以用以下方式使用：

```js
router.put(
  '/companies/:companyId',
  auth.ensured,
  companyCtrl.findById,
  authorize.onlyOwner,
  companyCtrl.update,
  response.toJSON('company')
);
```

上述代码确保用户已认证，从 MongoDB 中通过 ID 获取公司，并检查我们之前实现的策略是否授权用户更新公司。

# 公司后端模块

我们将为我们应用程序实现第一个后端模块。这个模块将处理与公司相关的一切。

## 公司模型

我们将向公司模型添加一个简单但有趣的功能，它将从公司名称创建一个所谓的缩略名。在我们的语境中，缩略名是从公司名称生成的，以便作为有效的 URL 接受。它将用于以有意义的方式引用公司。例如，如果我们系统中有一个名为 `Your Awesome Company` 的公司，生成的缩略名将是 `your-awesome-company`。

为了生成缩略名，我们将实现一个简单的辅助函数，以便在必要时可以重用它。创建一个名为 `app/helpers/common.js` 的文件，并添加以下代码行：

```js
'use strict';

module.exports.createSlug = createSlug;

function createSlug(value) {
   return value
   .toLowerCase()
   .replace(/[^\w\s]+/g,'')
   .trim()
   .replace(/[\s]+/g,'-');
}
```

现在我们有了 `helper` 函数，我们可以定义 `company` 模型和它所需的模式。创建一个名为 `app/models/company.js` 的文件，并向其中添加以下代码：

```js
'use strict';

const mongoose = require('mongoose');
const commonHelper = require('../helpers/common');
const Schema = mongoose.Schema;
const ObjectId = Schema.ObjectId;

let CompanySchema = new Schema({
  name: {
    type: String,
    required: true
  },
  slug: {
    type: String
  },
  owner: {
    type: ObjectId,
    required: true,
    ref: 'User'
  },
  members: {
    type: Array,
    default: []
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

CompanySchema.pre('save', (next) => {
  this.slug = commonHelper.createSlug(this.name);
  next();
});

// compile Company model
module.exports = mongoose.model('Company', CompanySchema);
```

我们定义了公司的 mongoose 模式，并添加了一个预保存钩子来生成缩略名。在这个预保存钩子中，我们使用了来自通用辅助函数的 `createSlug()` 方法。中间件是按顺序运行的，因此我们需要调用 `next()` 来表示执行完成。

## 公司控制器

通过公司控制器，我们将公开管理公司所需的所有业务逻辑。我们将逐一讨论这些功能。

### 创建公司

用户成功注册公司类型账户后，他们可以创建一个新公司并成为所有者。我们将实现一个简单的创建功能并将其挂载到 Express 路由上。让我们创建一个名为 `jobboard/app/controllers/company.js` 的控制器文件，内容如下：

```js
'use strict';

const _ = require('lodash');
const mongoose = require('mongoose');
const Company = mongoose.model('Company');

module.exports.create = createCompany;

function createCompany(req, res, next) {
  let data = _.pick(req.body, ['name', 'country', 'address']);
  data.owner = req.user._id;
  data.members = [req.user._id];

  Company.create(data, (err, company) => {
    if (err) {
      return next(err);
    }

    res.status(201).json(company);
  });
}
```

当我们定义模式时，我们在公司模型中添加了验证。我们添加的一件事是为创建方法选择必要的数据。公司的所有者默认是创建它的用户。我们还把用户添加到成员列表中。在成功创建了一个新公司后，我们返回一个包含有关新创建公司的信息的 JSON。

### 通过 ID 获取公司

现在我们能够创建公司了，是时候通过 ID 检索一个公司了。我们将追加以下代码到 `app/controller/company.js` 控制器文件：

```js
module.exports.findById = findCompanyById;

function findCompanyById(req, res, next) {
  if (!ObjectId.isValid(id)) {
    res.status(404).send({ message: 'Not found.'});
  }

  Company.findById(req.params.companyId, (err, company) => {
    if (err) {
      return next(err);
    }

    req.resources.company = company;
    next();
  });
}
```

在前面的代码行中，我们使用了 mongoose 从公司模型提供的 `findById` 方法。在我们搜索 MongoDB 中的公司之前，我们想要确保 ID 是一个有效的 `ObjectId`。

我们在这里添加的另一个有趣的功能是在请求中添加了一个全局的 `resource` 对象。这次我们不是返回 JSON，而是将其作为属性添加到我们将携带在 Express 路由的回调管道中的对象。这将在我们想要在其他情况下重用相同功能时非常有用。

### 获取所有公司

我们还希望获取存储在 MongoDB 中的所有公司。对于这个用例，一个简单的查询应该就足够了。我们可以添加一个简单的按国家过滤，默认返回最多 50 家公司。以下代码将实现此功能：

```js
module.exports.getAll = getAllCompanies;

function getAllCompanies(req, res, next) {
  const limit = +req.query.limit || 50;
  const skip = +req.query.skip || 0;
  let query = _.pick(req.query, ['country']);

  Company
  .find(query)
  .limit(limit)
  .skip(skip)
  .exec((err, companies) => {
    if (err) {
      return next(err);
    }

    req.resources.companies = companies;
    next();
  });
}
```

### 更新公司

在更新公司时，我们只想从公司模型中更新一些字段。我们不想在更新公司时更改所有者或添加新成员。更改所有者功能将不会实现；只有添加新成员功能将实现，但它将由不同的模块处理。

将以下代码行追加到 `jobboard/app/controllers/company.js`:

```js
module.exports.update = updateCompany;

function updateCompany(req, res, next) {
  let data = _.pick(req.body, ['name', 'country', 'address']);
  _.assign(req.resources.company, req.body);

  req.resources.company.save((err, updatedCompany) => {
    if (err) {
      return next(err);
    }

    req.resources.company = updatedCompany;
    next();
  });
}
```

### 添加公司成员

公司成员将只能有限地访问公司。他们可以发布空缺职位并筛选申请空缺职位的用户简历。我们将向位于 `jobboard/app/controllers/company.js` 的同一公司控制器添加此功能：

```js
module.exports.addMember = addCompanyMember;

function addCompanyMember(req, res, next) {
  let includes = _.includes(req.resources.company.members, req.body.member);

  if (includes) {
    return res.status(409).json({
      message: 'User is already a member of your company',
      type: 'already_member'
    });
  }

  req.resources.company.members.push(req.body.member);
  req.resources.company.save((err, updatedCompany) => {
    if (err) {
      return next(err);
    }

    req.resources.company = updatedCompany;
    next();
  });
}
```

### 删除公司成员

我们还需要处理如何从公司中删除成员。我们将在添加成员逻辑之后追加此功能：

```js
module.exports.removeMember = removeCompanyMember;

function removeCompanyMember(req, res, next) {
  let includes = _.includes(req.resources.company.members, req.body.member);

  if (!includes) {
    return res.status(409).json({
      message: 'User is not a member of your company',
      type: 'not_member'
    });
  }

  _.pull(req.resources.company.members, req.body.member);
  req.resources.company.save((err, updatedCompany) => {
    if (err) {
      return next(err);
    }

    req.resources.company = updatedCompany;
    next();
  });
}
```

## 公司路由

接下来，我们将定义所有必要的路由，以便从公司控制器访问之前实现的功能。让我们创建我们的路由器文件，命名为 `jobboard/app/routes/companies.js`，并添加以下内容：

```js
'use strict';

const express = require('express');
const router = express.Router();
const companyCtrl = require('../controllers/company');
const auth = require('../middlewares/authentication');
const authorize = require('../middlewares/authorization');
const response = require('../helpers/response');
```

按照以下步骤定义端点：

1.  创建公司：

    ```js
    router.post(
      '/companies',
      auth.ensured,
      companyCtrl.checkUserCompany,
      companyCtrl.create
    );
    ```

    我们确保用户在系统中没有其他公司。

1.  获取所有公司：

    ```js
    router.get(
      '/companies',
      companyCtrl.getAll,
      response.toJSON('companies')
    );
    ```

1.  通过 ID 获取公司：

    ```js
    router.get(
      '/companies/:companyId',
      companyCtrl.findById,
      response.toJSON('company')
    );
    ```

1.  更新公司：

    ```js
    router.put(
      '/companies/:companyId',
      auth.ensured,
      companyCtrl.findById,
      authorize.onlyOwner,
      companyCtrl.update,
      response.toJSON('company')
    );
    ```

    公司的更新只能由所有者进行。

1.  添加公司成员：

    ```js
    router.post(
      '/companies/:companyId/members',
      auth.ensured,
      companyCtrl.findById,
      authorize.onlyOwner,
      companyCtrl.addMember,
      response.toJSON('company')
    );
    ```

    只有公司所有者可以添加成员。

1.  删除公司成员：

    ```js
    router.delete(
      '/companies/:companyId/members',
      auth.ensured,
      companyCtrl.findById,
      authorize.onlyOwner,
      companyCtrl.removeMember,
      response.toJSON('company')
    );
    ```

    我们也将限制这个动作只允许公司所有者执行。

1.  导出路由器：

    ```js
    module.exports = router;
    ```

# 职位后端模块

此模块将实现所有与职位相关的后端逻辑。我们将定义必要的模型和控制器。模块中最重要的部分将进行解释。

## 职位模型

职位模型将定义`Jobs`集合中的一个实体，并在创建新职位时处理必要的验证。至于公司模型，我们将使用自定义变量文件来处理职位行业和类型。这两个文件将分别位于`jobboard/config/variables/industries.js`和`jobboard/config/variables/jobtypes.js`。两者都导出一个对象列表。

为了实现`职位`模型，我们将遵循以下步骤：

1.  创建模型文件，命名为`jobboard/app/models/job.js`。

1.  添加必要的依赖项：

    ```js
    const mongoose = require('mongoose');
    const commonHelper = require('../helpers/common');
    const Industries = require('../../config/variables/industries');
    const Countries = require('../../config/variables/countries');
    const Jobtypes = require('../../config/variables/jobtypes');
    const Schema = mongoose.Schema;
    const ObjectId = Schema.ObjectId;
    ```

1.  仅从变量文件中检索验证值列表：

    ```js
    const indEnum = Industries.map(item => item.slug);
    const cntEnum = Countries.map(item => item.code);
    const jobEnum = Jobtypes.map(item => item.slug);
    ```

1.  定义 Mongoose 模式：

    ```js
    let JobSchema = new Schema({
      title: {
        type: String,
        required: true
      },
      slug: {
        type: String,
        required: true
      },
      summary: {
        type: String,
        maxlength: 250
      },
      description: {
        type: String
      },
      type: {
        type: String,
        required: true,
        enum: jobEnum
      },
      company: {
        type: ObjectId,
        required: true,
        ref: 'Company'
      },
      industry: {
        type: String,
        required: true,
        enum: indEnum
      },
      country: {
        type: String,
        required: true,
        enum: cntEnum
      },
      createdAt: {
        type: Date,
        default: Date.now
      }
    });
    ```

1.  添加一个预保存钩子：

    ```js
    JobSchema.pre('save', (next) => {
      this.slug = commonHelper.createSlug(this.name);
      next();
    });
    ```

1.  最后，编译模型：

    ```js
    module.exports = mongoose.model('Job', JobSchema);	
    ```

## 职位控制器

我们的控制将集成所有必要的业务逻辑来处理所有职位 CRUD 操作。之后，我们可以将控制器公开的方法挂载到特定的路由上，以便外部客户端可以与我们的后端通信。

### 为公司添加新职位

在创建新职位时，它应该为特定公司创建，因为职位代表公司的一个空缺职位。因此，在创建职位时，我们需要公司上下文。

创建一个名为`jobboard/app/controllers/job.js`的控制器文件，并添加以下创建逻辑：

```js
const MAX_LIMIT = 50;
const JOB_FIELDS = ['title', 'summary', 'description', 'type', 'industry', 'country'];

const _ = require('lodash');
const mongoose = require('mongoose');
const Job = mongoose.model('Job');
const ObjectId = mongoose.Types.ObjectId;

module.exports.create = createJob;

function createJob(req, res, next) {
  let data = _.pick(req.body, JOB_FIELDS);
  data.company = req.company._id;

  Job.create(data, (err, job) => {
    if (err) {
      return next(err);
    }

    res.status(201).json(job);
  });
}
```

如我们之前所说，我们需要添加职位的公司上下文。为此，我们将向 Express 路由器请求管道添加一个通过 ID 获取公司的功能。别担心；当我们定义路由时会看到这一点。

### 通过 ID 查找职位

我们还应该从 Mongo 中通过 ID 检索一个职位。这里将使用与公司控制器中相同的逻辑。将以下代码添加到职位控制器中：

```js
module.exports.findById = findJobById;

function findJobById(req, res, next) {
  if (!ObjectId.isValid(id)) {
    res.status(404).send({ message: 'Not found.'});
  }

  Job.findById(req.params.jobId, (err, job) => {
    if (err) {
      return next(err);
    }

    res.resources.job = job;
    next();
  });
}
```

### 获取所有职位

当检索所有可用职位时，应该有应用一些过滤器的可能性，例如职位类型、分配的行业或职位可用的国家。除了这些过滤器外，我们还需要获取公司中所有可用的开放职位。所有这些逻辑都将使用以下代码实现：

```js
module.exports.getAll = getAllJobs;

function getAllJobs(req, res, next) {
  const limit = +req.query.limit || MAX_LIMIT;
  const skip = +req.query.skip || 0;
  let query = _.pick(req.query, ['type', 'country', 'industry']);

  if (req.params.companyId) {
    query.company = req.params.companyId;
  }

  Job
  .find(query)
  .limit(limit)
  .skip(skip)
  .exec((err, jobs) => {
    if (err) {
      return next(err);
    }

    req.resources.jobs = jobs;
    next();
  });
}
```

### 更新特定职位

我们还希望更新公司发布的职位，但仅限于公司成员。这种限制将由中间件处理；目前，我们只将实现更新功能。将以下代码添加到`app/controllers/job.js`：

```js
module.exports.update = updateJob;
function updateJob(req, res, next) {
  var data = _.pick(req.body, JOB_FIELDS);
  _.assign(req.resources.job, data);

  req.resources.job.save((err, updatedJob) => {
    if (err) {
      return next(err);
    }

    res.json(job);
  });
}
```

## 职位路线

首先，我们将创建一个名为`app/routes/jobs.js`的路由文件，并添加以下代码：

```js
'use strict';

const express = require('express');
const router = express.Router();
const companyCtrl = require('../controllers/company');
const jobCtrl = require('../controllers/job');
const auth = require('../middlewares/authentication');
const authorize = require('../middlewares/authorization');
const response = require('../helpers/response');
```

### 获取一个或所有职位

现在我们有了基础，我们可以开始定义我们的路由。第一对路由将可供公共访问，因此检索一个或所有职位不需要进行身份验证。请添加以下代码：

```js
router.get(
  '/jobs',
  jobCtrl.getAll,
  response.toJSON('jobs')
);

router.get(
  '/jobs/:jobId',
  jobCtrl.findById,
  response.toJSON('job')
);
```

奖励——获取特定公司的职位！

```js
router.get(
  '/companies/:companyId/jobs',
  jobCtrl.getAll,
  response.toJSON('jobs')
);
```

### 创建路由

现在，在创建和更新职位时，事情会变得有些棘手。要创建一个职位，请添加以下代码：

```js
router.post(
  '/companies/:companyId/jobs',
  auth.ensured,
  companyCtrl.findById,
  authorize.onlyMembers,
  jobCtrl.create
);
```

当创建一个职位时，用户必须登录并且必须是发布该职位的公司成员。为此，我们从数据库中检索一个公司并使用授权中间件。我们比较并检查认证用户是否在成员列表中。如果一切顺利，用户就可以创建一个新的职位空缺。

可能还有其他完成这些任务的方法，但这个解决方案可以带来好处，因为我们只在需要时请求资源。例如，如果用户已认证，我们可以在每个请求的`req.user`对象上添加公司对象，但这将意味着每个请求都会有额外的 I/O 操作。

### 更新路由

对于更新功能，附加以下代码：

```js
router.put(
  '/companies/:companyId/jobs/:jobId',
  auth.ensured,
  companyCtrl.findById,
  authorize.onlyMembers,
  jobCtrl.findById,
  jobCtrl.update
);
```

如您所见，这里与创建路由相同的限制原则也存在。我们唯一额外添加的是检索一个职位 ID，这是更新功能所需要的。

通过这种方式，我们已经完成了`job`模块的后端逻辑实现。

# 职位申请

每个用户都可以申请职位，公司也想知道谁申请了他们可用的职位空缺。为了处理这些场景，我们将把所有职位的申请存储在 MongoDB 中单独的集合中。我们将描述后端 Node.js 应用程序逻辑。

## 应用程序模型

应用程序模型将非常简单和直接。我们本可以使用嵌入式数据模型。换句话说，我们可以在职位实体中保存所有申请。从我的观点来看，分开的集合给你更多的灵活性。

让我们创建一个名为`app/models/application.js`的文件，并将以下代码添加到定义模式中：

```js
'use strict';

const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const ObjectId = Schema.ObjectId;

let ApplicationSchema = new Schema({
  user: {
    type: ObjectId,
    required: true,
    ref: 'User'
  },
  status: {
    type: String,
    default: 'pending',
    enum: ['pending', 'accepted', 'processed']
  },
  job: {
    type: ObjectId,
    required: true,
    ref: 'Job'
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('Application', ApplicationSchema);
```

## 控制器逻辑

后端控制器将处理所有必要的逻辑，以管理与职位申请相关的端点上的传入请求。我们将把控制器中导出的每个方法挂载到特定的路由上。

### 申请职位

当候选人申请职位时，我们在 MongoDB 中存储该申请的引用。我们之前定义了`Application`模式。为了持久化申请，我们将在`app/controllers/application.js`控制器文件中使用以下后端逻辑：

```js
module.exports.create = createApplication;

function createApplication(req, res, next) {
  Application.create({
    user: req.user._id,
    job: req.params.jobId
  }, (err, application) => {
    if (err) {
      return next(err);
    }

    res.status(201).json(application);
  });
}
```

### 通过 ID 查找给定应用程序

在更新和从数据库中删除应用程序时，我们需要通过其 ID 找到应用程序。有一个通用的逻辑来检索数据是很好的；它可以在不同的场景中被重用。将以下代码附加到控制器文件中：

```js
module.exports.findById = findApplicationById;

function findApplicationById(req, res, next) {
  if (!ObjectId.isValid(id)) {
    res.status(404).send({ message: 'Not found.'});
  }

  Application.findById(req.params.applicationId, (err, application) => {
    if (err) {
      return next(err);
    }

    res.resources.application = application;
    next();
  });
}
```

再次强调，我们正在使用请求对象上的`resource`属性来填充查询的结果。

### 获取所有职位申请

每家公司都想看到他们列出的职位的所有申请。为了提供这个功能，职位控制器必须返回一个应用程序列表，并能够通过状态进行过滤。以下代码将实现这个功能：

```js
module.exports.getAll = getAllApplications;

function getAllApplications(req, res, next) {
  const limit = +req.query.limit || 50;
  const skip = +req.query.skip || 0;
  let query = {
    job: req.params.jobId
  };

  if (req.query.status) {
    query.status = req.query.status;
  }

  Application
  .find(query)
  .limit(limit)
  .skip(offset)
  .exec((err, applications) => {
    if (err) {
      return next(err);
    }

    req.resources.applications = applications;
    next();
  });
}
```

### 更新应用程序

为了更改申请的状态，我们必须使用特定的状态值更新它。控制器中的 `update` 方法将处理此用例。将更新逻辑附加到控制器文件：

```js
module.exports.update = updateApplication;

function updateApplication(req, res, next) {
  req.resources.application.status = req.body.status;

  req.resources.application.save((err, updatedApplication) => {
    if (err) {
      return next(err);
    }

    res.json(updatedApplication);
  });
}
```

### 从职位中删除申请

应聘者应该有能力删除空缺职位的申请。我们将不允许任何人除应聘者外删除申请。这种限制将由中间件处理。删除的后端逻辑应类似于以下内容：

```js
module.exports.remove = removeApplication;

function removeApplication(req, res, next) {
  req.resources.application.remove((err) => {
    if (err) {
      return next(err);
    }

    res.json(req.resources.application);
  });
}
```

现在，我们不会讨论如何添加路由。您可以在应用程序的最终源代码中找到所有可用的路由。

# 创建新公司

在成功注册后，用户可以创建新的公司。我们已经使用 Node.js 实现了后端逻辑，并且应该能够将公司存储在 MongoDB 的 companies 集合中。

## 公司服务

尽管我们正在讨论创建公司的功能，但我们将添加所有端点到服务中：

1.  让我们创建服务文件，命名为 `jobboard/public/src/company/company.service.ts`。

1.  导入必要的依赖项：

    ```js
    import { Injectable } from 'angular2/core';
    import { Http, Response, Headers } from 'angular2/http';
    import { AuthHttp } from '../auth/index';
    import { contentHeaders } from '../common/index';
    import { Company } from './company.model';
    ```

1.  创建 `service` 类：

    ```js
    @Injectable()
    export class CompanyService {
      private _http: Http;
      private _authHttp: AuthHttp;
    }
    ```

1.  添加 `constructor`：

    ```js
      constructor(http: Http, authHttp: AuthHttp) {
        this._http = http;
        this._authHttp = authHttp;
      }
    ```

1.  附加 `create` 方法：

    ```js
      create(company) {
        let body = JSON.stringify(company);

        return this._authHttp
        .post('/api/companies', body, { headers: contentHeaders })
        .map((res: Response) => res.json())  
      }
    ```

1.  定义 `findByid()` 函数：

    ```js
      findById(id) {
        return this._http
        .get(`/api/companies/${id}`, { headers: contentHeaders })
        .map((res: Response) => res.json())
      }
    ```

1.  从后端检索所有公司：

    ```js
      getAll() {
        return this._http
        .get('/api/companies', { headers: contentHeaders })
        .map((res: Response) => res.json())
      }
    ```

1.  更新公司：

    ```js
     update(company) {
        let body = JSON.stringify(company);

        return this._authHttp
        .put(`/api/companies/${company._id}`, body, { headers: contentHeaders })
        .map((res: Response) => res.json())
      }
    ```

## 创建公司组件

现在我们已经有一个完全功能化的服务，它与后端进行通信，我们可以开始实现我们的组件。创建公司组件将是第一个。

让我们创建一个新文件，命名为 `public/src/company/components/company-create.component.ts`，并添加组件的类和依赖项：

```js
import { Component, OnInit } from 'angular2/core';
import { Router, RouterLink } from 'angular2/router';
import { CompanyService } from '../company.service';
import { Company } from '../company.model';
export class CompanyCreateComponent implements OnInit {
  public company: Company;
  private _router: Router;
  private _companyService: CompanyService;

  constructor(companyService: CompanyService, router: Router) {
    this._router = router;
    this._companyService = companyService;
  }

  ngOnInit() {
    this.company = new Company();
  }
}
```

`Component` 注解应类似于以下内容：

```js
@Component({
    selector: 'company-create',
    directives: [
      RouterLink
    ],
    template: `
      <div class="login jumbotron center-block">
        <h1>Register</h1>
      </div>
      <div>
        <form role="form" (submit)="onSubmit($event)">
          <div class="form-group">
            <label for="name">Company name</label>
            <input type="text" [(ngModel)]="company.name" class="form-control" id="name">
          </div>
          <div class="form-group">
            <label for="email">Country</label>
            <input type="text" [(ngModel)]="company.country" class="form-control" id="country">
          </div>
          <div class="form-group">
            <label for="email">Address</label>
            <input type="text" [(ngModel)]="company.address" class="form-control" id="address">
          </div>
          <div class="form-group">
            <label for="password">Summary</label>
            <textarea [(ngModel)]="company.summary" class="form-control" id="summary"></textarea>
          </div>
          <button type="submit" class="button">Submit</button>
        </form>
      </div>
    `
})
```

为了将公司数据属性绑定到每个表单输入控件，我们使用了 `ngModel` 双向数据绑定。在提交表单时，执行 `onSubmit()` 方法。让我们添加前面的方法：

```js
  onSubmit(event) {
    event.preventDefault();

    this._companyService
    .create(this.company)
    .subscribe((company) => {
      if (company) {
        this.goToCompany(company._id, company.slug);
      }
    }, err => console.error(err));
  }
```

这将通过我们的服务尝试创建新公司。如果公司成功创建，我们将导航到公司详情页面。`goToCompany()` 方法描述如下：

```js
  goToCompany(id, slug) {
    this._router.navigate(['CompanyDetail', { id: id, slug: slug}]);
  }
```

我们使用路由器导航到公司的详细信息。路由器将构建所需的路径以进行导航。错误处理未涵盖。您还可以添加验证作为改进。

# 显示公司

我们从早期实现“添加新公司”功能时，对公司模块有一个良好的开端。因此，我们可以跳入并创建和实现其余的文件以显示所有公司。

为了在我们的应用程序中显示公司列表，我们创建了一个新的组件文件，命名为 `public/src/company/components/company-list.component.ts`：

```js
import { Component, OnInit } from 'angular2/core';
import { Router, RouterLink } from 'angular2/router';
import { CompanyService } from '../company.service';
import { Company } from '../company.model';

@Component({})
export class CompanyListComponent implements OnInit {
  public companies: Array<Company>;
  private _router: Router;
  private _companyService: CompanyService;

  constructor(companyService: CompanyService, router: Router) {
    this._router = router;
    this._companyService = companyService;
  }

  ngOnInit() {
    this._companyService
    .getAll()
    .subscribe((companies) => {
      this.companies = companies;
    });
  }
}
```

如您所见，我们有一个相当基本的组件。在初始化时，使用 `CompanyService` 从后端检索公司。我们直接订阅返回的 `Observable` 以更新组件的 `companies` 属性。

现在只剩下添加 `Component` 注解：

```js
@Component({
    selector: 'company-list',
    directives: [
      RouterLink
    ],
    template: `
      <div class="jumbotron center-block">
        <h2>Companies list</h2>
        <p class="lead">Here you can find all the registered companies.</p>
      </div>
      <div>
      <div *ngFor="#company of companies" class="col col-25">
        <img src="img/208x140?text=product+image&txtsize=18"/>
        <h3>
          <a href="#"
            [routerLink]="['CompanyDetail', { id: company._id, slug: company.slug }]">
            {{ company.name }}
          </a>
          </h3>
      </div>
      </div>
    `
})
```

使用 `ngFor`，我们遍历公司数据并相应地显示。你可以显示其他数据，但现阶段，公司名称应该足够了。另外，当点击名称时，我们使用 `RouterLink` 导航到目标公司。

# 求职模块

我们将继续构建 `job` 模块。这样做的原因是 `company` 模块使用 `job` 模块中的一个组件来显示公司的可用职位列表。

## 求职服务

求职服务将处理与后端的通信，主要是 CRUD 操作。我们将创建一个 Angular 工厂来完成这个任务。创建一个名为 `public/app/job/job.service.js` 的新文件，并按照以下步骤操作：

1.  定义基础结构和公开方法：

    ```js
    import { Injectable } from 'angular2/core';
    import { Http, Response, Headers } from 'angular2/http';
    import { AuthHttp } from '../auth/index';
    import { contentHeaders, serializeQuery } from '../common/index';
    import { Job } from './job.model';

    @Injectable()
    export class JobService {
      private _http: Http;
      private _authHttp: AuthHttp;

      constructor(http: Http, authHttp: AuthHttp) {
        this._http = http;
        this._authHttp = authHttp;
      }
    }
    ```

1.  实现 `创建职位` 方法：

    ```js
      create(job) {
        let body = JSON.stringify(job);

        return this._authHttp
        .post('/api/jobs', body, { headers: contentHeaders })
        .map((res: Response) => res.json())
      }
    ```

    我们使用 `AuthHttp` 服务，因为创建端点需要认证用户。

1.  添加按 ID 查找职位的代码：

    ```js
      findById(id) {
        return this._http
        .get(`/api/jobs/${id}`, { headers: contentHeaders })
        .map((res: Response) => res.json())
      }
    ```

1.  从后端查询所有职位：

    ```js
      getAll(criteria) {
        let query = '';
        let str = serializeQuery(criteria);

        if (str) {
          query = `?${str}`;
        }

        return this._http
        .get(`/api/jobs${query}`, { headers: contentHeaders })
        .map((res: Response) => res.json())
      }
    ```

`getAll()` 方法接受一个标准作为参数以过滤职位。在某些情况下，我们只想获取给定公司的职位列表。我们使用 `serializeQuery` 函数构建查询字符串，该函数位于 `public/src/common/query.ts` 下，内容如下：

```js
export function serializeQuery(query): string {
  var chunks = [];
  for(var key in query)
    if (query.hasOwnProperty(key)) {
      let k = encodeURIComponent(key);
      let v = encodeURIComponent(query[key]);
      chunks.push(`${k}=${v}`);
    }
  return chunks.join('&');
}
```

## 求职基础组件

我们将为我们 的 `job` 模块构建一个基础组件。它将包含显示子组件所需的全部 `RouteConfig`。创建一个名为 `public/src/job/components/job-base.component.ts` 的新文件：

```js
import { Component } from 'angular2/core';
import { RouterOutlet, RouteConfig } from 'angular2/router';
import { JobService } from '../job.service';
import { JobListComponent } from './job-list.component';
import { JobDetailComponent } from './job-detail.component';
import { JobCreateComponent } from './job-create.component';

@RouteConfig([
  { path: '/', as: 'JobList', component: JobListComponent, useAsDefault: true },
  { path: '/:id/:slug', as: 'JobDetail', component: JobDetailComponent },
  { path: '/create', as: 'JobCreate', component: JobCreateComponent }
])
@Component({
    selector: 'job-base',
    directives: [
      RouterOutlet
    ],
    template: `
      <router-outlet></router-outlet>
    `
})
export class JobBaseComponent {
  constructor() {}
} 
```

我们将每个子组件挂载到特定的路径上。我们将使用与 `JobDetail` 相同的 URL 结构来处理 `CompanyDetail`。我认为使用 URL 中的 slug 可以使其看起来既美观又简洁。

接下来，我们将逐一定义组件。

## 求职组件

`jobs` 组件将在整个应用程序中重用。它的目的是根据几个因素显示职位列表。

创建一个名为 `public/src/job/components/jobs.component.ts` 的文件，并包含以下内容：

```js
import { Component, OnInit } from 'angular2/core';
import { Router, RouterLink } from 'angular2/router';
import { JobService } from '../job.service';
import { Job } from '../job.model'; 

export class JobsComponent implements OnInit {
  public company: any;
  public jobs: Array<Job>;
  private _jobsService: JobService;
  private _router: Router;

  constructor(jobsService: JobService, router: Router) {
    this._router = router;
    this._jobsService = jobsService;
  }
}
```

添加 `ngOnInit` 方法以从 Express 应用程序中检索必要的数据，如下所示：

```js
  ngOnInit() {
    let query: any = {};

    if (this.company) {
      query.company = this.company;
    }

    this._jobsService
    .getAll(query)
    .subscribe((jobs) => {
      this.jobs = jobs;
    });
  }
```

我们的组件有一个 `company` 属性，当我们要查询与某个公司相关的所有职位时将使用它。同时，别忘了添加以下注解：

```js
@Component({
    selector: 'jobs',
    inputs: ['company'],
    directives: [RouterLink],
    template: `
      <div *ngFor="#job of jobs" class="col">
        <h3>
          <a href="#"
            [routerLink]="['/Jobs', 'JobDetail', { id: job._id, slug: job.slug }]">
            {{ job.title }}
          </a>
        </h3>
        <p>
          <a href="#"
            [routerLink]="['/Companies', 'CompanyDetail', { id: job.company._id, slug: job.company.slug }]">
            {{ job.company.name }}
          </a>
          <span>·</span>
          <span>{{ job.industry }}</span>
          <span>·</span>
          <span>{{ job.type }}</span>
          <span>·</span>
          <span>{{ job.createdAt }}</span>
        </p>
        <p>{{ job.summary }}</p>
      </div>
    `
})
```

我们的组件还有一个名为 `company` 的输入数据绑定属性。这将引用一个公司的 ID。同时创建一个链接到公司页面的链接。

## 求职列表组件

在此组件中，我们可以使用之前构建的 `jobs` 组件来列出系统中的所有可用职位。由于所有主要逻辑都位于 `jobs` 组件中，我们只需包含它。

创建一个名为 `public/src/job/componets/job-list.component.ts` 的新文件，并添加以下代码：

```js
import { Component } from 'angular2/core';
import { JobService } from '../job.service';
import { Job } from '../job.model';
import { JobsComponent } from './jobs.component';

@Component({
    selector: 'job-list',
    directives: [JobsComponent],
    template: `
      <div class="login jumbotron center-block">
        <h2>Job openings</h2>
        <p class="lead">Take a look, maybe you will find something for you.</p>
      </div>
      <div>
        <jobs></jobs>
      </div>
    `
})
export class JobListComponent {
  public jobs: Array<Job>;
  private _jobsService: JobService;

  constructor(jobsService: JobService) {
    this._jobsService = jobsService;
  }
}
```

## 求职详情

职位详情页面将显示用户所需职位的所有必要信息。我们将使用与公司详情中相同的用户友好路由。幸运的是，我们已经有了一个与后端 API 通信的服务。

创建一个名为`public/src/job/components/job-detail.component.ts`的文件，并添加以下代码：

```js
import { Component, OnInit } from 'angular2/core';
import { RouteParams, RouterLink } from 'angular2/router';
import { JobService } from '../job.service';
import { Job } from '../job.model';

@Component({})
export class JobDetailComponent implements OnInit {
  public job: Job;
  private _routeParams: RouteParams;
  private _jobService: JobService;

  constructor(jobService: JobService, routerParams: RouteParams) {
    this._routeParams = routerParams;
    this._jobService = jobService;
  }

  ngOnInit() {
    const id: string = this._routeParams.get('id');
    this.job = new Job();
    this._jobService
    .findById(id)
    .subscribe((job) => {
      this.job = job;
    });
  }
}
```

组件内的逻辑与`CompanyDetailComponent`中的几乎相同。使用`id`路由参数，我们从后端获取所需的职位。

`Component`注解应包含必要的模板和指令：

```js
@Component({
    selector: 'job-detail',
    directives: [
      RouterLink
    ],
    template: `
      <div class="job-header">
        <div class="col content">
          <p>Added on: {{ job.createdAt }}</p>
          <h2>{{ job.name }}</h2>
          <div class="job-description">
            <h4>Description</h4>
            <div>{{ job.description }}</div>
          </div>
        </div>
        <div class="sidebar">
          <h4>Country</h4>
          <p>{{ job.country }}</p>
          <h4>Industry</h4>
          <p>{{ job.industry }}</p>
          <h4>Job type</h4>
          <p>{{ job.type }}</p>
        </div>
      </div>
    `
})
```

## 添加新职位

现在我们能够列出所有可用的职位，我们可以实现添加新职位的功能。这将会与我们在公司模块中实现的功能相似。

可能感觉你一直在重复做同样的事情，但本章的目的是创建一个专注于 CRUD 操作的应用程序。许多企业级应用程序都有实现这些操作的庞大模块。所以不用担心！我们将有章节来实验不同的技术和架构。

让我们继续并创建一个名为`public/src/job/components/job-create.component.ts`的文件：

```js
import { Component, OnInit } from 'angular2/core';
import { Router, RouterLink } from 'angular2/router';
import { JobService } from '../job.service';
import { Job } from '../job.model';

export class JobCreateComponent implements OnInit {
  public job: Job;
  private _router: Router;
  private _jobService: JobService;

  constructor(jobService: JobService, router: Router) {
    this._router = router;
    this._jobService = jobService;
  }

  ngOnInit() {
    this.job = new Job();
  }

  onSubmit(event) {
    event.preventDefault();

    this._jobService
    .create(this.job)
    .subscribe((job) => {
      if (job) {
        this.goToJob(job._id, job.slug);
      }
    });
  }

  goToJob(id, slug) {
    this._router.navigate(['JobDetail', { id: id, slug: slug}]);
  }
}
```

在`Component`类前添加以下注解：

```js
@Component({
    selector: 'job-create',
    directives: [
      RouterLink
    ],
    template: `
      <div class="jumbotron center-block">
        <h1>Post a new job</h1>
        <p>We are happy to see that you are growing.</p>
      </div>
      <div>
        <form role="form" (submit)="onSubmit($event)">
          <div class="form-group">
            <label for="title">Job title</label>
            <input type="text" [(ngModel)]="job.title" class="form-control" id="title">
          </div>
          <div class="form-group">
            <label for="industry">Industry</label>
            <input type="text" [(ngModel)]="job.industry" class="form-control" id="industry">
          </div>
          <div class="form-group">
            <label for="country">Country</label>
            <input type="text" [(ngModel)]="job.country" class="form-control" id="country">
          </div>
          <div class="form-group">
            <label for="type">Job type</label>
            <input type="text" [(ngModel)]="job.type" class="form-control" id="type">
          </div>
          <div class="form-group">
            <label for="summary">Summary</label>
            <textarea [(ngModel)]="job.summary" class="form-control" id="summary"></textarea>
          </div>
          <div class="form-group">
            <label for="description">Description</label>
            <textarea [(ngModel)]="job.description" class="form-control" id="description"></textarea>
          </div>
          <button type="submit" class="button">Create a job</button>
        </form>
      </div>
    `
})
```

# 公司详情

可能你已经注意到，在我们之前列出所有公司时，我们创建了一些漂亮的 URL。我们将使用该路径来显示公司的所有详情以及可用的职位。

URL 还包含公司别名，这是公司名称的 URL 友好表示。它对用户没有好处，这只是我们添加的 URL 糖，以便更好地显示公司名称。查询后端数据时仅使用公司 ID。

既然我们已经有了所有必要的组件和服务，我们可以通过以下步骤实现我们的详情组件：

1.  创建一个名为`public/src/company/components/company-detail.component.ts`的新文件。

1.  添加必要的依赖项：

    ```js
    import { Component, OnInit } from 'angular2/core';
    import { RouteParams, RouterLink } from 'angular2/router';
    import { CompanyService } from '../company.service';
    import { Company } from '../company.model';
    import { JobsComponent } from '../../job/index';
    ```

1.  添加`Component`注解：

    ```js
    @Component({
        selector: 'company-detail',
        directives: [
          JobsComponent,
          RouterLink
        ],
        template: `
          <div class="company-header">
            <h2>{{ company.name }}</h2>
            <p>
              <span>{{ company.country }}</span>
              <span>·</span>
              <span>{{ company.address }}</span>
            </p>
          </div>
          <div class="company-description">
            <h4>Description</h4>
          </div>
          <div class="company-job-list">
            <jobs [company]=company._id></jobs>
          </div>
        `
    })
    ```

    在模板中，我们使用我们之前实现的`jobs`组件，通过发送公司的`id`来列出公司的所有可用职位。

1.  声明`component`类：

    ```js
    export class CompanyDetailComponent implements OnInit {
      public company: Company;
      private _routeParams: RouteParams;
      private _companyService: CompanyService;

      constructor(companyService: CompanyService, routerParams: RouteParams) {
        this._routeParams = routerParams;
        this._companyService = companyService;
      }

      ngOnInit() {
        const id: string = this._routeParams.get('id');
        this.company = new Company();
        this._companyService
        .findById(id)
        .subscribe((company) => {
          this.company = company;
        });
      }
    }
    ```

# 用户个人资料

在我们的系统中，我们没有账户类型。我们只为用户定义角色，例如公司所有者、公司成员或候选人。因此，任何注册用户都可以用不同的信息填写他们的个人资料。

记住我们在用户模式中定义了一个`profile`属性。它将保存有关用户工作经验、教育或用户想要添加的任何其他相关数据的所有信息。

用户的个人资料将通过块来构建。每个块将分组一个特定领域，例如经验，允许用户为每个块添加新的条目。

## 个人资料后端

管理配置数据的后端逻辑尚未实现。我想传达一种我们在扩展现有后端的同时添加新功能的感觉。因此，我们将首先创建一个新的控制器文件，`app/controllers/profile.js`。然后添加以下代码：

```js
'use strict';

const _ = require('lodash');
const mongoose = require('mongoose');
const User = mongoose.model('User');
const ProfileBlock = mongoose.model('ProfileBlock');
const ObjectId = mongoose.Types.ObjectId;

module.exports.getProfile = getUserProfile;
module.exports.createProfileBlock = createUserProfileBlock;
module.exports.updateProfile = updateUserProfile;
```

我们将导出三个函数来管理配置数据。让我们按照以下步骤定义它们：

1.  获取当前认证用户和整个配置数据：

    ```js
    function getUserProfile(req, res, next) {
      User
      .findById(req.user._id)
      .select('+profile')
      .exec((err, user) => {
        if (err) {
          return next(err);
        }

        req.resources.user = user;
        next();
      });
    }
    ```

1.  为用户创建新的配置文件块：

    ```js
    function createUserProfileBlock(req, res, next) {
      if (!req.body.title) {
        return res.status(400).json({ message: 'Block title is required' });
      }

      var block = new ProfileBlock(req.body);
      req.resources.user.profile.push(block);

      req.resources.user.save((err, updatedProfile) => {
        if (err) {
          return next(err);
        }

        req.resources.block = block;
        next();
      });
    }
    ```

    我们为 `ProfileBlock` 模式使用自定义模式来创建新的配置文件块并将其推送到用户的配置数据中。我们将回到我们的模式并定义它。

1.  更新现有配置文件块：

    ```js
    function updateUserProfile(req, res, next) {
      // same as calling user.profile.id(blockId)
      // var block = req.resources.user.profile.find(function(b) {
      //   return b._id.toString() === req.params.blockId;
      // });

      let block = req.resources.user.profile.id(req.params.blockId);

      if (!block) {
        return res.status(404).json({ message: '404 not found.'});
      }

      if (!block.title) {
        return res.status(400).json({ message: 'Block title is required' });
      }

      let data = _.pick(req.body, ['title', 'data']);
      _.assign(block, data);

      req.resources.user.save((err, updatedProfile) => {
        if (err) {
          return next(err);
        }

        req.resources.block = block;
        next();
      });
    }
    ```

当更新配置文件块时，我们需要搜索该特定块并使用新数据更新它。之后，更改将被保存并持久化到 MongoDB。

让我们看看我们的 `ProfileBlock` 模式，它位于 `app/models/profile-block.js` 下：

```js
'use strict';

const mongoose = require('mongoose');
const commonHelper = require('../helpers/common');
const Schema = mongoose.Schema;

let ProfileBlock = new Schema({
  title: {
    type: String,
    required: true
  },
  slug: String,
  data: []
});

ProfileBlock.pre('save', function(next) {
  this.slug = commonHelper.createSlug(this.title);
  next();
});

module.exports = mongoose.model('ProfileBlock', ProfileBlock);
```

上述文档模式将被嵌入到用户的文档 `profile` 属性中。`data` 属性将包含所有配置文件块，以及它们自己的数据。

为了公开我们之前实现的功能，让我们创建一个配置路由文件，称为 `app/routes/profile.js`：

```js
'use strict';

const express = require('express');
const router = express.Router();
const profileCtrl = require('../controllers/profile');
const auth = require('../middlewares/authentication');
const response = require('../helpers/response');

router.get(
  '/profile',
  auth.ensured,
  profileCtrl.getProfile,
  response.toJSON('user')
);

router.post(
  '/profile/blocks',
  auth.ensured,
  profileCtrl.getProfile,
  profileCtrl.createProfileBlock,
  response.toJSON('block')
);

router.put(
  '/profile/blocks/:blockId',
  auth.ensured,
  profileCtrl.getProfile,
  profileCtrl.updateProfile,
  response.toJSON('block')
);

module.exports = router;
```

## 同步配置数据

为了存储和检索与用户相关的配置数据，我们将创建一个 Angular 服务，该服务将处理与后端的通信。

前端 `profile` 模块将位于 `user` 模块中，因为它们相关，并且我们可以根据它们的领域上下文进行分组。创建一个名为 `public/src/user/profile/profile.service.ts` 的文件，并添加以下基线代码：

```js
import { Injectable } from 'angular2/core';
import { Http, Response, Headers } from 'angular2/http';
import { Subject } from 'rxjs/Subject';
import { BehaviorSubject } from 'rxjs/Subject/BehaviorSubject';
import { Observable } from 'rxjs/Observable';
import { contentHeaders } from '../../common/index';
import { AuthHttp } from '../../auth/index';
import { Block } from './block.model';

@Injectable()
export class ProfileService {
  public user: Subject<any> = new BehaviorSubject<any>({});
  public profile: Subject<Array<any>> = new BehaviorSubject<Array<any>>([]);
  private _http: Http;
  private _authHttp: AuthHttp;
  private _dataStore: { profile: Array<Block> };
}
```

这次，我们将使用 `Observables` 和 `Subject` 来处理数据流。在这种情况下，它们更合适，因为有很多动态部分。配置数据可以从许多不同的来源更新，并且更改需要传递给所有订阅者。

为了有一个本地数据副本，我们将在服务中使用数据存储。现在让我们逐一实现每个方法：

1.  添加类构造函数：

    ```js
      constructor(http: Http, authHttp: AuthHttp) {
        this._http = http;
        this._authHttp = authHttp;
        this._dataStore = { profile: [] };
        this.profile.subscribe((profile) => {
          this._dataStore.profile = profile;
        });
      }
    ```

1.  获取用户的配置信息：

    ```js
      public getProfile() {
        this._authHttp
        .get('/api/profile', { headers: contentHeaders })
        .map((res: Response) => res.json())
        .subscribe((user: any) => {
          this.user.next(user);
          this.profile.next(user.profile);
        });
      }
    ```

1.  创建新的配置文件块：

    ```js
      public createProfileBlock(block) {
        let body = JSON.stringify(block);

        this._authHttp
        .post('/api/profile/blocks', body, { headers: contentHeaders })
        .map((res: Response) => res.json())
        .subscribe((block: any) => {
          this._dataStore.profile.push(block);
          this.profile.next(this._dataStore.profile);
        }, err => console.error(err));
      }
    ```

1.  更新现有配置文件块：

    ```js
      public updateProfileBlock(block) {
        if (!block._id) {
          this.createProfileBlock(block);
        } else {
          let body = JSON.stringify(block);

          this._authHttp
          .put(`/api/profile/blocks/${block._id}`, body, { headers: contentHeaders })
          .map((res: Response) => res.json())
          .subscribe((block: any) => {
            this.updateLocalBlock(block);
          }, err => console.error(err));
        }
      }
    ```

    当更新配置文件块时，我们检查该块是否存在 ID。如果没有，这意味着我们想要创建一个新的块，因此我们将使用 `createProfileBlock()`。

1.  从本地存储更新块：

    ```js
      private updateLocalBlock(data) {
        this._dataStore.profile.forEach((block) => {
          if (block._id === data._id) {
            block = data;
          }
        });

        this.profile.next(this._dataStore.profile);
      }
    ```

## 编辑配置数据

要编辑用户的配置文件，我们将创建一个单独的组件。用户配置文件是使用块构建的。因此，我们应该为配置文件块创建另一个组件。

按照以下步骤实现 `ProfileEditComponent`：

1.  添加必要的依赖项：

    ```js
    import { Component, OnInit } from 'angular2/core';
    import { ProfileBlockComponent } from './profile-block.component';
    import { ProfileService } from '../profile.service';
    import { Block } from '../block.model';
    ```

1.  放置 `Component` 注解：

    ```js
    @Component({
        selector: 'profile-edit',
        directives: [ProfileBlockComponent],
        template: `
        <section>

          <div class="jumbotron">
            <h2>Hi! {{user.name}}</h2>
            <p class="lead">Your public e-mail is <span>{{user.email}}</span> <br> and this is your profile</p>
          </div>

          <div class="row">
            <div class="col-md-12">
              <div class="profile-block" *ngFor="#block of profile">
                <profile-block [block]="block"></profile-block>
              </div>
            </div>

            <form class="form-horizontal col-md-12">
              <div class="form-group">
                <div class="col-md-12">
                  <input [(ngModel)]="newBlock.title" type="text" class="form-control" placeholder="Block title">
                </div>
              </div>
              <div class="form-group">
                <div class="col-md-12">
                  <button (click)="onClick($event)" class="button">New block</button>
                </div>
              </div>
            </form>
          </div>

        </section>
        `
    })
    ```

1.  添加属性和构造函数：

    ```js
    export class ProfileEditComponent implements OnInit {
      public user: any;
      public profile: any;
      public newBlock: Block;
      private _profileService: ProfileService;

      constructor(profileService: ProfileService) {
        this._profileService = profileService;
      }
    }
    ```

1.  添加 `ngOnInit()` 方法：

    ```js
      ngOnInit() {
        this.user = {};
        this.newBlock = new Block();
        this._profileService.user.subscribe((user) => {
          this.user = user;
        });
        this._profileService.profile.subscribe((profile) => {
          this.profile = profile;
        });
        this._profileService.getProfile();
      }
    ```

1.  定义用户如何添加新块：

    ```js
      onClick(event) {
        event.preventDefault();
        let profile = this.profile.slice(0);  // clone the profile
        let block = Object.assign({}, this.newBlock); // clone the new block

        profile.push(block);
        this._profileService.profile.next(profile);
        this.newBlock = new Block();
      }
    ```

我们将订阅配置文件数据流并显示所有块。为了显示配置文件块，我们使用了一个单独的组件。该组件将块作为数据输入获取。

当用户添加一个新块时，我们将新创建的块推送到配置文件中。这相当简单，因为我们使用了 RxJS 中的`Subject`。通过这种方式，我们可以将我们的配置文件数据与所有组件同步。

## 配置文件块组件

因为配置文件是由块组成的，我们可以创建一个可重用且封装了所有块功能的单独组件。让我们按照以下步骤创建我们的组件：

1.  创建一个名为`public/src/user/profile/components/profile-block.component.ts`的新文件。

1.  添加必要的依赖项：

    ```js
    import { Component, OnInit } from 'angular2/core';
    import { ProfileService } from '../profile.service';
    import { Block } from '../block.model';
    import { Entry } from '../entry.model';
    ```

1.  配置`Component`注解：

    ```js
    @Component({
        selector: 'profile-block',
        inputs: ['block'],
        template: `
          <div class="panel panel-default">
            <div class="panel-heading">
              <h3 class="panel-title">{{block.title}}</h3>
            </div>
            <div class="panel-body">
              <div class="profile-block-entries">
                <div *ngFor="#entry of block.data">
                  <div class="form-group">
                    <label>Title</label>
                    <input class="form-control" type="text"
                      (keydown.enter)="onEnter($event)"
                      [(ngModel)]="entry.title">
                  </div>
                  <div class="form-group">
                    <label>Sub title</label>
                    <input class="form-control" type="text"
                      (keydown.enter)="onEnter($event)"
                      [(ngModel)]="entry.subTitle">
                  </div>
                  <div class="form-group">
                    <label>Description</label>
                    <textarea class="form-control"
                      (keydown.enter)="onEnter($event)"
                      [(ngModel)]="entry.description"></textarea>
                  </div>
                  <hr>
                </div>
              </div>
              <button class="btn btn-default btn-xs btn-block" (click)="addEntry($event)">
                <i class="glyphicon glyphicon-plus"></i> Add new entry
              </button>
            </div>
          </div>
        `
    })
    ```

1.  定义`ProfileBlockComponent`类：

    ```js
    export class ProfileBlockComponent implements OnInit {
      public block: any;
      private _profileService: ProfileService;

      constructor(profileService: ProfileService) {
        this._profileService = profileService;
      }

      ngOnInit() {
        console.log(this.block);
      }

      addEntry(event) {
        event.preventDefault();
        this.block.data.push(new Entry());
      }

      onEnter(event) {
        event.preventDefault();
        this._profileService.updateProfileBlock(this.block);
      }
    }
    ```

使用`addEntry()`方法，我们可以向我们的块添加更多条目。这是一个简单的操作，它将新条目推送到块的数据库中。为了保存更改，我们绑定到 keydown 事件，该事件将*Enter*键与`onEnter()`方法关联。此方法将使用之前实现的服务更新配置文件块。

如果一个块是新添加的并且没有`id`，`ProfileService`将处理这种情况，因此我们不需要在我们的组件中添加不同的方法调用。

## 额外模型

我们使用了一些额外的模型——这些模型在后端找不到——以帮助我们处理 Angular 部分。当创建初始值或为属性设置默认值时，它们非常有用。

`Entry`模型描述了配置文件块的单个条目。该模型可以在`public/src/user/profile/entry.model.ts`下找到：

```js
export class Entry {
  title: string;
  subTitle: string;
  description: string;

  constructor(
    title?: string,
    subTitle?: string,
    description?: string
  ) {
    this.title = title || '';
    this.subTitle = subTitle || '';
    this.description = description || '';
  }
}
```

我们还在我们的模块中使用了第二个辅助模型——`public/src/user/profile/block.model.ts`：

```js
import { Entry } from './entry.model';

export class Block {
  _id: string;
  title: string;
  slug: string;
  data: Array<any>;

  constructor(
    _id?: string,
    title?: string,
    slug?: string,
    data?: Array<any>
  ) {
    this._id = _id;
    this.title = title;
    this.slug = slug;
    this.data = data || [new Entry()];
  }
}
```

之前使用的模型使用了`Entry`模型来初始化`data`属性，以防没有数据。您也可以为您的模型添加验证。这取决于应用程序的复杂性。

剩余的功能可以在以下链接的最终项目仓库中找到：[`github.com/robert52/mean-blueprints-jobboard`](https://github.com/robert52/mean-blueprints-jobboard)

# 摘要

最后，我们到达了本章的结尾。

在本章中，我们从模板开始构建应用程序，扩展了一些功能，并添加了我们自己的新功能。我们创建了一个具有多种用户类型的系统，并添加了授权策略。此外，在最后几步中，我们扩展了我们的后端 API 以添加新功能，并为我们 Angular 2 应用程序添加了一个额外的模块。

在下一章中，我们将使用实时通信，看看用户如何在应用程序中相互交互。
