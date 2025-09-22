# 第七章：模块化我们的代码

在上一章中，我们遵循了 TDD 工作流程并实现了我们 API 的第一个端点——创建用户端点。我们使用 Gherkin 编写了端到端（E2E）测试，使用 *Cucumber* 测试运行器运行它们，并使用它们来驱动开发。一切正常，但所有代码都包含在一个单一的、单体文件（`src/index.js`）中；这不是模块化的，使得我们的项目难以维护，尤其是在我们添加更多端点时。因此，在本章中，我们将把我们的应用程序代码分解成更小的模块。这将允许我们在第八章 编写单元/集成测试中为它们编写 **单元** 和 **集成** 测试。

通过遵循本章，你将能够做到以下几件事：

+   将大块代码分解成更小的模块

+   使用 **JSON Schema** 和 Ajv 定义和验证 JavaScript 对象

# 模块化我们的代码

如果你查看 `src/index.js` 文件，你会看到有三个顶级中间件函数——`checkEmptyPayload`、`checkContentTypeIsSet` 和 `checkContentTypeIsJson`——以及一个匿名错误处理函数。这些都是我们可以提取到它们自己的模块中的理想候选者。所以，让我们开始吧！

# 模块化我们的中间件

让我们在名为 `create-user/refactor-modules` 的新分支中执行这个重构过程：

```js
$ git checkout -b create-user/refactor-modules
```

然后，在 `src/middlewares` 中创建一个目录；这是我们存储所有中间件模块的地方。在其内部，创建四个文件——每个中间件函数一个：

```js
$ mkdir -p src/middlewares && cd src/middlewares
$ touch check-empty-payload.js \
 check-content-type-is-set.js \
 check-content-type-is-json.js \
 error-handler.js
```

然后，将中间件函数从 `src/index.js` 移动到它们对应的文件中。例如，`checkEmptyPayload` 函数应该移动到 `src/middlewares/check-empty-payload.js`。然后，在每个模块的末尾，将函数作为默认导出导出。例如，`error-handler.js` 文件将看起来像这样：

```js
function errorHandler(err, req, res, next) {
  ...
}

export default errorHandler;
```

现在，回到 `src/index.js` 并导入这些模块以恢复之前的行为：

```js
import checkEmptyPayload from './middlewares/check-empty-payload';
import checkContentTypeIsSet from './middlewares/check-content-type-is-set';
import checkContentTypeIsJson from './middlewares/check-content-type-is-json';
import errorHandler from './middlewares/error-handler';
...
app.use(errorHandler);
```

现在，再次运行我们的 E2E 测试，以确保我们没有破坏任何东西。同时，别忘了提交你的代码！

```js
$ git add -A && git commit -m "Extract middlewares into modules"
```

通过提取中间件函数，我们提高了 `src/index.js` 文件的可读性。由于我们正确地命名了函数，代码的意图和流程一目了然——你可以从函数名中理解函数的功能。接下来，让我们对请求处理器也做同样的处理。

# 模块化我们的请求处理器

目前，我们只为`POST /users`端点有一个请求处理程序，但到本章结束时，我们将实现更多。将它们全部定义在`src/index.js`文件中会导致一个庞大且难以阅读的混乱。因此，让我们将每个请求处理程序定义为其自己的模块。让我们首先在`src/handlers/users/create.js`中创建一个文件，并将创建用户请求处理程序提取到其中。之前，请求处理程序是一个匿名箭头函数；现在它在自己的模块中，让我们给它命名为`createUser`。最后，以与我们处理中间件相同的方式`export`函数。你应该得到类似以下内容：

```js
function createUser(req, res) {
  if (
    !Object.prototype.hasOwnProperty.call(req.body, 'email')
    || !Object.prototype.hasOwnProperty.call(req.body, 'password')
  ) {
    res.status(400);
    res.set('Content-Type', 'application/json');
    return res.json({ message: 'Payload must contain at least the email and password fields' });
  }
  ...
}

export default createUser;
```

然后，将`createUser`处理程序重新导入到`src/index.js`中，并在`app.post`内部使用它：

```js
...
import createUser from './handlers/users/create';
...
app.post('/users', createUser);
...
```

然而，我们的请求处理程序需要一个 Elasticsearch 客户端才能工作。解决这一问题的方法之一是将以下行移动到`src/handlers/users/create.js`模块的顶部：

```js
import elasticsearch from 'elasticsearch';
const client = new elasticsearch.Client({ host: ... });
```

然而，从长远来看，由于我们将有许多请求处理程序，我们不应该为每个处理程序实例化一个单独的客户端实例。相反，我们应该创建一个 Elasticsearch 客户端实例，并将其通过引用传递给每个请求处理程序。

要做到这一点，让我们在`src/utils/inject-handler-dependencies.js`中创建一个实用函数，它接受一个请求处理程序函数和 Elasticsearch 客户端，并返回一个新的函数，该函数将调用请求处理程序，并将客户端作为参数之一传递：

```js
function injectHandlerDependencies(handler, db) {
  return (req, res) => { handler(req, res, db); };
}

export default injectHandlerDependencies;
```

这是一个**高阶函数**的例子，它操作其他函数或返回其他函数。这是可能的，因为函数是 JavaScript 中的一种对象类型，因此被视为**一等公民**。这意味着你可以像传递任何其他对象一样传递函数，甚至作为函数参数。

要使用它，将其导入我们的`src/index.js`文件：

```js
import injectHandlerDependencies from './utils/inject-handler-dependencies';
```

然后，而不是直接使用`createUser`请求处理程序，传递`injectHandlerDependencies`返回的处理程序：

```js
# Change this
app.post('/users', createUser);

# To this
app.post('/users', injectHandlerDependencies(createUser, client));
```

最后，更新请求处理程序本身以使用客户端：

```js
function createUser(req, res, db) {
  ...
  db.index({ ... });
}
```

再次运行端到端测试，以确保我们没有引入任何错误，然后提交我们的更改：

```js
$ git add -A && git commit -m "Extract request handlers into modules"
```

# 单一职责原则

我们已经提取了请求处理程序并将其迁移到自己的模块中。然而，它并不像它本可以的那样模块化；目前，处理程序执行三个功能：

+   验证请求

+   写入数据库

+   生成响应

如果你已经学习过面向对象的设计原则，你无疑会遇到**SOLID**原则，这是一个代表**单一职责**、**开闭原则**、**里氏替换原则**、**接口隔离原则**和**依赖倒置原则**的记忆法缩写。

单一职责原则指出，一个模块应该执行一个，并且只有一个，功能。因此，我们应该将验证和数据库逻辑提取到它们自己的专用模块中。

# 解耦我们的验证逻辑

然而，我们不能直接从`src/handlers/users/create.js`复制现有的验证代码而不做修改。这是因为验证代码直接修改了响应对象`res`，这意味着验证逻辑和响应逻辑是**紧密耦合**的。

为了解决这个问题，我们必须在验证逻辑和响应处理器之间定义一个公共的**接口**。而不是直接修改响应，验证逻辑将生成一个符合此接口的对象，响应处理器将消费此对象以生成适当的响应。

当请求验证失败时，我们可以将其视为一种错误，因为客户端提供了错误的数据负载。因此，我们可以扩展原生的`Error`对象来创建一个新的`ValidationError`对象，该对象将作为接口。我们不需要提供状态或设置头信息，因为那是请求处理器的工作。我们只需要确保`ValidationError`实例将包含`message`属性。由于这是`Error`的默认行为，我们不需要做太多其他的事情。

# 创建`ValidationError`接口

在`src/validators/errors/validation-error.js`中创建一个新的文件，并添加`ValidationError`类的定义：

```js
class ValidationError extends Error {
  constructor(...params) {
    super(...params);

    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, ValidationError);
    }
  }
}

export default ValidationError;
```

上述代码扩展了`Error`类以创建自己的类。我们需要这样做，以便区分验证错误（应返回`400`响应）和代码中的错误（应返回`500`响应）。

# 将验证逻辑模块化

接下来，在`src/validators/users/create.js`中创建一个新的文件，并将请求处理器中的验证块复制到该文件中，将其包裹在其自己的函数内并导出该函数：

```js
function validate (req) {
  if (
    !Object.prototype.hasOwnProperty.call(req.body, 'email')
    || !Object.prototype.hasOwnProperty.call(req.body, 'password')
  ) {
    res.status(400);
    res.set('Content-Type', 'application/json');
    return res.json({ message: 'Payload must contain at least the email and password fields' });
  }
  ...
}

export default validate;
```

接下来，从`src/validators/errors/validation-error.js`导入`ValidationError`类。然后，而不是修改`res`对象（它不在作用域内），返回`ValidationError`实例。最终的`src/validators/users/create.js`文件可能看起来像这样：

```js
import ValidationError from '../errors/validation-error';

function validate(req) {
  if (
    !Object.prototype.hasOwnProperty.call(req.body, 'email')
    || !Object.prototype.hasOwnProperty.call(req.body, 'password')
  ) {
    return new ValidationError('Payload must contain at least the email and password fields');
  }
  if (
    typeof req.body.email !== 'string'
    || typeof req.body.password !== 'string'
  ) {
    return new ValidationError('The email and password fields must be of type string');
  }
  if (!/^[\w.+]+@\w+\.\w+$/.test(req.body.email)) {
    return new ValidationError('The email field must be a valid email.');
  }
  return undefined;
}

export default validate;
```

接下来，我们需要将此函数导入到请求处理器中，并使用它来验证创建用户请求的数据负载。如果验证结果是`ValidationError`的实例，则生成`400`响应；否则，继续索引用户文档：

```js
import ValidationError from '../../validators/errors/validation-error';
import validate from '../../validators/users/create';
function createUser(req, res, db) {
  const validationResults = validate(req);
  if (validationResults instanceof ValidationError) {
    res.status(400);
    res.set('Content-Type', 'application/json');
    return res.json({ message: validationResults.message });
  }
  db.index({ ... })
}

export default createUser;
```

通过提供公共接口，我们已经成功地将验证逻辑从其他代码中解耦。现在，运行端到端测试，如果它们是绿色的，提交我们的更改！

```js
$ git add -A && git commit -m "Decouple validation and response logic"
```

# 创建引擎

尽管大部分验证逻辑已经被抽象成一个独立的模块，但请求处理器仍然在处理验证器的结果，与数据库交互，并发送响应；它仍然没有遵循单一职责原则。

请求处理器的唯一任务应该是将请求传递给一个 *引擎*，该引擎将处理请求，并使用操作的结果进行响应。根据操作的结果，请求处理器应随后向客户端发出适当的响应。

因此，让我们在 `src/engines/users` 中创建一个新的目录并添加一个 `create.js` 文件；在内部，定义一个 `create` 函数并将其导出。这个 `create` 函数将验证我们的请求并将写入数据库，将操作的结果返回给请求处理器。由于写入数据库是一个异步操作，我们的 `create` 函数应该返回一个承诺。

尝试自己实现 `create` 函数，并在此处查看我们的实现：

```js
import ValidationError from '../../validators/errors/validation-error';
import validate from '../../validators/users/create';

function create(req, db) {
  const validationResults = validate(req);
  if (validationResults instanceof ValidationError) {
    return Promise.reject(validationResults);
  }
  return db.index({
    index: process.env.ELASTICSEARCH_INDEX,
    type: 'user',
    body: req.body,
  });
}

export default create;
```

然后，在 `src/handlers/users/create.js` 中，导入引擎模块并使用结果生成响应。最终的文件应该看起来像这样：

```js
import ValidationError from '../../validators/errors/validation-error';
import create from '../../engines/users/create';

function createUser(req, res, db) {
  create(req, db).then((result) => {
    res.status(201);
    res.set('Content-Type', 'text/plain');
    return res.send(result._id);
  }, (err) => {
    if (err instanceof ValidationError) {
      res.status(400);
      res.set('Content-Type', 'application/json');
      return res.json({ message: err.message });
    }
    return undefined;
  }).catch(() => {
    res.status(500);
    res.set('Content-Type', 'application/json');
    return res.json({ message: 'Internal Server Error' });
  });
}

export default createUser;
```

运行测试以确保它们仍然通过，然后将这些更改提交到 Git：

```js
$ git add -A && git commit -m "Ensure Single-Responsibility Principle for handler"
```

太棒了！我们现在已经重构了代码，使其更加模块化，并确保每个模块与其他模块解耦！

# 添加用户配置文件

如果我们回顾创建用户的必要条件，有一个仍然未完成——“用户可以可选地提供配置文件；否则，将为他们创建一个空配置文件”。所以，让我们实现这个要求！

# 将规范作为测试来编写

我们将首先编写端到端测试开始开发。在前一章中，我们已经测试了一个未提供配置文件的场景。在这些新的端到端测试中，我们将添加两个更多场景，其中客户端提供了一个配置文件对象——一个使用无效配置文件，另一个使用有效配置文件。

因此，我们必须首先决定什么构成了一个有效的配置文件；换句话说，我们的配置文件对象的结构应该是什么？没有正确或错误答案，但为了这本书，我们将使用以下结构：

```js
{
  "name": {
    "first": <string>,
    "last": <string>,
    "middle": <string>
  },
  "summary": <string>,
  "bio": <string>
}
```

所有字段都是可选的，但如果提供了，它们必须是正确的类型。

让我们从测试无效配置文件场景开始。在 `spec/cucumber/features/users/create/main.feature` 中添加以下场景概述：

```js
Scenario Outline: Invalid Profile

  When the client creates a POST request to /users/
  And attaches <payload> as the payload
  And sends the request
  Then our API should respond with a 400 HTTP status code
  And the payload of the response should be a JSON object
  And contains a message property which says "The profile provided is invalid."

  Examples:

  | payload                                                                          |
  | {"email":"e@ma.il","password":"abc","profile":{"foo":"bar"}}                     |
  | {"email":"e@ma.il","password":"abc","profile":{"name":{"first":"Jane","a":"b"}}} |
  | {"email":"e@ma.il","password":"abc","profile":{"summary":0}}                     |
  | {"email":"e@ma.il","password":"abc","profile":{"bio":0}}                         |
```

这些示例涵盖了属性类型不正确的情况，以及/或者提供了不受支持的属性。

当我们运行这些测试时，`And attaches <payload> as the payload` 显示为未定义。这个步骤定义应该允许我们将任何任意有效负载附加到请求上。尝试在 `spec/cucumber/steps/index.js` 中实现这一点，并对照以下解决方案检查你的解决方案：

```js
When(/^attaches (.+) as the payload$/, function (payload) {
  this.requestPayload = JSON.parse(payload);
  this.request
    .send(payload)
    .set('Content-Type', 'application/json');
});
```

再次运行端到端测试，这次，新定义的测试应该会失败。红-绿-重构。我们现在已经编写了一个失败的测试；下一步是实现功能，使其通过测试。

# 基于模式的验证

我们的测试失败是因为我们的 API 实际上正在将（无效的）配置文件对象写入数据库；相反，我们期望我们的 API 返回一个 `400` 错误。因此，我们必须为 `profile` 子文档实现额外的验证步骤。

目前，我们正在使用 `if` 条件块来验证电子邮件和密码字段。如果我们对我们的新用户对象使用相同的方法，我们就必须编写一个非常长的 `if` 语句列表，这对可读性很不利。有人也可能认为我们当前的 `validation` 函数实现已经相当难以阅读，因为它并不立即明显地表明用户对象应该是什么样子。因此，我们需要找到更好的方法。

一种更声明性的验证方式是使用 **模式**，它只是描述数据结构的一种正式方式。定义了模式之后，我们可以使用验证库来测试请求负载与模式是否匹配，如果不匹配，则返回适当的错误消息。

因此，在本节中，我们将使用模式来验证我们的配置文件对象，然后重构我们现有的所有验证代码以使用基于模式的验证。

# 模式类型

在 JavaScript 中最常用的模式是 **JSON Schema** ([json-schema.org](http://json-schema.org/))。要使用它，你首先定义一个用 JSON 编写的模式，然后使用模式验证库将感兴趣的对象与模式进行比较，以查看它们是否匹配。

但在我们解释 JSON Schema 的语法之前，让我们看看两个主要的 JavaScript 库，它们支持模式验证但不使用 JSON Schema：

+   `joi` ([`github.com/hapijs/joi`](https://github.com/hapijs/joi)) 允许你以可组合、可链式的方式定义要求，这意味着代码非常易于阅读。它在 GitHub 上有超过 9,000 个星标，并且被超过 94,000 个存储库和 3,300 个包所依赖：

```js
const schema = Joi.object().keys({
    username: Joi.string().alphanum().min(3).max(30).required(),
    password: Joi.string().regex(/^[a-zA-Z0-9]{3,30}$/),
    access_token: [Joi.string(), Joi.number()],
    birthyear: Joi.number().integer().min(1900).max(2013),
    email: Joi.string().email()
}).with('username', 'birthyear').without('password', 'access_token');

const result = Joi.validate({ username: 'abc', birthyear: 1994 }, schema);
```

+   `validate.js` ([`validatejs.org/`](https://validatejs.org/)) 是另一个非常表达式的验证库，它允许你定义自己的自定义验证函数。它在 GitHub 上有 1,700 个星标，并且被超过 2,700 个存储库和 290 个包所依赖：

```js
var constraints = {
  username: {
    presence: true,
    exclusion: {
      within: ["nicklas"],
      message: "'%{value}' is not allowed"
    }
  },
  password: {
    presence: true,
    length: {
      minimum: 6,
      message: "must be at least 6 characters"
    }
  }
};

validate({password: "bad"}, constraints);
```

# 选择对象模式和验证库

所以在三个选项中，我们应该使用哪一个？为了回答这个问题，我们首先应该考虑它们的 **互操作性** 和 **表达性**。

# 互操作性

互操作性涉及到不同框架、库和语言消费模式有多容易。在这个标准下，JSON Schema 无疑是赢家。

使用标准化模式（如 JSON Schema）的好处是，同一个模式文件可以被多个代码库使用。例如，随着我们平台的增长，我们可能有多个内部服务，每个服务都需要验证用户数据；其中一些甚至可能用另一种语言（例如 Python）编写。

与在不同语言中拥有多个用户模式定义相比，我们可以使用相同的模式文件，因为所有主要语言都有 JSON Schema 验证器：

+   Swift: `JSONSchema.swift` ([`github.com/kylef-archive/JSONSchema.swift`](https://github.com/kylef-archive/JSONSchema.swift))

+   Java: `json-schema-validator` ([github.com/java-json-tools/json-schema-validator](https://github.com/java-json-tools/json-schema-validator))

+   Python: `jsonschema` ([pypi.python.org/pypi/jsonschema](https://pypi.python.org/pypi/jsonschema))

+   Go: `gojsonschema` ([github.com/xeipuuv/gojsonschema](https://github.com/xeipuuv/gojsonschema))

您可以在 [json-schema.org/implementations.html](http://json-schema.org/implementations.html) 查看完整的验证器列表。

# 表达能力

JSON Schema 支持许多验证关键词和数据格式，如 IETF 记忆录 *JSON Schema Validation: A Vocabulary for Structural Validation of JSON* 所定义（[json-schema.org/latest/json-schema-validation.html](http://json-schema.org/latest/json-schema-validation.html)）；然而，由于 JSON 本身的限制，JSON Schema 缺乏以函数形式定义自定义验证逻辑的能力。

例如，JSON Schema 不提供表达以下逻辑的方法：“如果 `age` 属性小于 `18`，则 `hasParentalConsent` 属性必须设置为 `true`。” 因此，如果您想执行更复杂的检查，这些必须在 JavaScript 中的单独函数中完成。或者，一些基于 JSON Schema 的验证库扩展了 JSON Schema 语法，并允许开发者实现自定义验证逻辑。例如，`ajv` 验证库支持定义自定义关键词。

对于非 JSON Schema 验证库，`joi` 和 `validate.js` 都允许您定义自定义验证函数。

因此，尽管在理论上 JSON Schema 的表达能力较弱，但在实践中，所有解决方案的表达能力和灵活性都是相同的。因为 JSON Schema 是一个成熟的行业标准，并且具有更好的互操作性，所以我们将使用它来验证我们的有效载荷。

在撰写本书时，JSON Schema 规范仍处于草案阶段（具体为 draft-07，可在 [tools.ietf.org/html/draft-handrews-json-schema-00](https://tools.ietf.org/html/draft-handrews-json-schema-00) 找到）。最终规范可能与此处描述的略有不同。请参考 [json-schema.org](http://json-schema.org/) 上的最新版本。

# 创建我们的配置文件模式

因此，让我们使用 JSON Schema 构建我们的用户配置对象模式。为此，我们首先必须理解其语法。

首先要注意的是，一个 JSON Schema 本身就是一个 JSON 对象。因此，最简单的 JSON Schema 只是一个空对象：

```js
{}
```

这个空对象模式将允许任何类型的数据，所以它几乎毫无用处。为了使其有用，我们必须描述我们期望的数据类型。这可以通过`type`关键字来完成。`type`关键字期望其值要么是一个字符串，其中只允许一个类型，要么是一个数组，其中允许数组中指定的任何类型。

我们期望用户个人资料对象的输入仅是对象，因此我们可以指定一个`"object"`类型的`type`：

```js
{ "type": "object" }
```

`type`是最基本的关键字。还有许多其他适用于所有类型的常见关键字，例如`title`；也有特定于类型的关键字，例如`maximum`，它仅适用于`number`类型的数据。

对于对象类型，我们可以使用特定于类型的`properties`关键字来描述我们期望我们的对象包含哪些属性。`properties`的值必须是一个对象，其中属性名作为键，另一个有效的 JSON Schema 作为值，称为**子模式**。在我们的例子中，我们期望`bio`和`summary`属性是`string`类型，而`name`属性是`object`类型，因此我们的模式看起来像这样：

```js
{
  "type": "object",
  "properties": {
    "bio": { "type": "string" },
    "summary": { "type": "string" },
    "name": { "type": "object" }
  }
}
```

# 拒绝额外属性

最后，我们将设置对象特定的`additionalProperties`关键字为`false`。这将拒绝包含在`properties`下未定义的键的对象（例如，`isAdmin`）：

```js
{
  "type": "object",
  "properties": {
    "bio": { "type": "string" },
    "summary": { "type": "string" },
    "name": { "type": "object" }
  },
  "additionalProperties": false
}
```

将`additionalProperties`设置为`false`非常重要，尤其是在 Elasticsearch 中。这是因为 Elasticsearch 使用一种称为**动态映射**的技术来推断其文档的数据类型，并使用它来生成其索引。

# Elasticsearch 中的动态映射

要在关系型数据库中创建一个表，你必须指定一个**模型**，该模型存储有关每个列的名称和数据类型的信息。在插入任何数据之前，必须提供这些信息。

Elasticsearch 有一个类似的概念称为**类型映射**，它存储有关文档中每个属性名称和数据类型的信息。区别在于我们不需要在插入任何数据之前提供类型映射；事实上，我们根本不需要提供它！这是因为当 Elasticsearch 尝试从正在索引的文档中推断数据类型时，它将将其添加到类型映射中。这种自动检测数据类型并将其添加到类型映射的过程就是我们所说的动态映射。

动态映射是 Elasticsearch 提供的一种便利，但也意味着我们必须在将其索引到 Elasticsearch 之前对数据进行清理和验证。如果我们允许用户向他们的文档添加任意字段，类型映射可能会推断出错误的数据类型，或者被无关字段充斥。此外，由于 Elasticsearch 默认索引每个字段，这可能导致许多无关的索引。

你可以在[`www.elastic.co/guide/en/elasticsearch/guide/current/dynamic-mapping.html`](https://www.elastic.co/guide/en/elasticsearch/guide/current/dynamic-mapping.html)了解更多关于动态映射的信息。

# 为子模式添加具体性

目前，我们对`name`属性施加的唯一约束是它必须是一个对象。这还不够具体。因为每个属性的值都是另一个有效的 JSON Schema，我们可以为`name`属性定义一个更具体的模式：

```js
{
  "type": "object",
  "properties": {
    "bio": { "type": "string" },
    "summary": { "type": "string" },
    "name": {
      "type": "object",
      "properties": {
        "first": { "type": "string" },
        "last": { "type": "string" },
        "middle": { "type": "string" }
      },
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}
```

此 JSON Schema 满足我们对创建用户请求负载施加的每一个约束。然而，它看起来就像一个任意的 JSON 对象；查看它的人不会立即理解它是一个模式。因此，为了使我们的意图更加明确，我们应该向其中添加标题、描述和一些元数据。

# 添加标题和描述

首先，我们应该为模式和每个可能需要澄清的属性提供`title`和`description`关键字。这些关键字不在验证中使用，仅用于为您的模式用户提供上下文：

```js
"title": "User Profile Schema",
"description": "For validating client-provided user profile object when creating and/or updating an user",
```

# 指定元模式

接下来，我们应该包括`$schema`关键字，它声明 JSON 对象是一个 JSON Schema。它指向一个 URL，该 URL 定义了当前 JSON Schema 必须遵守的元模式。我们选择了[`json-schema.org/schema#`](http://json-schema.org/schema#)，它指向 JSON Schema 规范的最新草案：

```js
{ "$schema": "http://json-schema.org/schema#" }
```

# 指定一个唯一标识符

最后，我们应该包括`$id`关键字，它定义了我们模式的唯一 URI。这个 URI 可以被其他模式用来引用我们的模式，例如，当使用我们的模式作为子模式时。目前，只需将其设置为有效的 URL，最好使用你控制的域名：

```js
"$id": "http://api.hobnob.social/schemas/users/profile.json"
```

如果您不知道如何购买域名，我们将在第十章部署您的应用程序到 VPS 中向您展示。现在，只需使用一个虚拟域名，如`example.com`。

我们完成的 JSON Schema 应该看起来像这样：

```js
{
  "$schema": "http://json-schema.org/schema#",
  "$id": "http://api.hobnob.social/schemas/users/profile.json",
  "title": "User Profile Schema",
  "description": "For validating client-provided user profile object 
   when creating and/or updating an user",
  "type": "object",
  "properties": {
    "bio": { "type": "string" },
    "summary": { "type": "string" },
    "name": {
      "type": "object",
      "properties": {
        "first": { "type": "string" },
        "last": { "type": "string" },
        "middle": { "type": "string" }
      },
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}
```

将此文件保存到`/src/schema/users/profile.json`。

# 为创建用户请求负载创建一个模式

目前，我们现有的代码仍然使用自定义定义的`if`语句来验证创建用户请求负载对象的电子邮件和密码字段。由于我们将为我们的配置文件对象使用 JSON Schema 验证库，因此我们也应该将现有的验证逻辑迁移到 JSON Schema 以保持一致性。因此，让我们为整个创建用户请求负载对象创建一个模式。

在`src/schema/users/create.json`中创建一个新文件，并插入以下模式：

```js
{
  "$schema": "http://json-schema.org/schema#",
  "$id": "http://api.hobnob.social/schemas/users/create.json",
  "title": "Create User Schema",
  "description": "For validating client-provided create user object",
  "type": "object",
  "properties": {
    "email": {
      "type": "string",
      "format": "email"
    },
    "password": { "type": "string" },
    "profile": { "$ref": "profile.json#"}
  },
  "required": ["email", "password"],
  "additionalProperties": false
}
```

这里有几个需要注意的地方：

+   我们使用`format`属性来确保电子邮件属性是一个有效的电子邮件，如 RFC 5322 第 3.4.1 节定义。然而，我们还想排除某些语法上有效的电子邮件，如`daniel@127.0.0.1`，这些很可能是垃圾邮件。在本章的后面部分，我们将向您展示如何覆盖此默认格式。

+   我们使用了一个 JSON 引用(`$ref`)来引用我们之前定义的配置文件模式。`$ref`语法在[`tools.ietf.org/html/draft-pbryan-zyp-json-ref-03`](https://tools.ietf.org/html/draft-pbryan-zyp-json-ref-03)中指定，它允许我们从现有的模式中组合出更复杂的模式，从而消除了重复的需求。

+   我们已将`email`和`password`属性标记为必填。

# 选择 JSON Schema 验证库

下一步是选择一个 JSON Schema 验证库。json-schema.org ([`json-schema.org/`](https://json-schema.org/))提供了一个验证器列表，您可以在[json-schema.org/implementations.html](http://json-schema.org/implementations.html)上阅读。在选择模式验证库时，我们寻找两个东西：性能（有多快）和一致性（与规范有多接近）。

来自丹麦的开源开发者 Allan Ebdrup 创建了一套基准测试，用于比较这些库。您可以在[github.com/ebdrup/json-schema-benchmark](https://github.com/ebdrup/json-schema-benchmark)找到它。基准测试显示，*动态 JSON Schema 验证器*（djv，[github.com/korzio/djv](https://github.com/korzio/djv)）是最快的，并且失败测试最少（只有 1 个）。第二快的库是*另一个 JSON Schema 验证器*（ajv，[github.com/epoberezkin/ajv](https://github.com/epoberezkin/ajv)），它也只有单个失败测试：

| 库 | 相对速度 | 失败测试数量 |
| --- | --- | --- |
| `djv v2.0.0` (最快) | 100% | 1 |
| `ajv v5.5.1` | 98% | 1 |
| `is-my-json-valid v2.16.1` | 50.1% | 14 |
| `tv4 v1.3.0` | 0.2% | 33 |

因此，djv 似乎是一个明显的选择。然而，开发者和社区支持也是需要考虑的重要因素。所以，让我们看看一些最受欢迎的库，并检查它们的 GitHub 星标数量、从[npmjs.com](https://www.npmjs.com)每周的下载量以及依赖的仓库和包数量*：

| **库** | **GitHub 仓库** | **版本** | **GitHub 星标** | **每周下载量** | **贡献者数量** | **依赖** |
| --- | --- | --- | --- | --- | --- | --- |
| **仓库** | **包** |
| `ajv` | [epoberezkin/ajv](https://github.com/epoberezkin/ajv) | 6.5.3 | 4,117 | 12,324,991 | 74 | 1,256,690 | 2,117 |
| `tv4` | [geraintluff/tv4](https://github.com/geraintluff/tv4) | 1.3.0 | 1,001 | 342,094 | 22 | 8,276 | 486 |
| `jsonschema` | [tdegrunt/jsonschema](https://github.com/tdegrunt/jsonschema) | 1.2.4 | 889 | 214,902 | 39 | 18,636 | 727 |
| `is-my-json-valid` | [mafintosh/is-my-json-valid](https://github.com/mafintosh/is-my-json-valid) | 2.19.0 | 837 | 2,497,926 | 23 | 463,005 | 267 |
| `JSV` | [garycourt/JSV](https://github.com/garycourt/JSV) | 4.0.2 | 597 | 211,573 | 6 | 9,475 | 71 |
| `djv` | [korzio/djv](https://github.com/korzio/djv) | 2.1.1 | 134 | 1,036 | 6 | 36 | 10 |

* 这些数据截至 2018 年 9 月 6 日是正确的。

如您所见，虽然 djv 在技术上是最优解决方案，但 Ajv 拥有最多的下载量和贡献者数量——这是项目得到社区广泛支持的迹象。

除了这些指标之外，您还可能想检查以下内容：

+   它对`master`分支的最后一次有意义的提交日期（这排除了版本升级和格式更改）——例如，`JSV`库的最后提交是在 2012 年 7 月 11 日；因此，尽管它可能仍然有大量的活跃用户，但我们不应使用不再维护的库

+   开放问题的数量

+   发布频率

所有这些因素都将为您提供有关工具是否正在积极开发的指示。

考虑到所有因素，Ajv 似乎是显而易见的选择，因为它在性能、一致性和社区支持之间取得了正确的平衡。

# 使用 Ajv 对 JSON Schema 进行验证

因此，让我们首先将 Ajv 添加到我们项目的依赖项中：

```js
$ yarn add ajv
```

然后，在`src/validators/users/create.js`中，导入`ajv`库以及我们的两个 JSON 模式：

```js
import Ajv from 'ajv';
import profileSchema from '../../schema/users/profile.json';
import createUserSchema from '../../schema/users/create.json';
import ValidationError from '../errors/validation-error';
...
```

我们需要导入这两个模式，因为我们的创建用户模式引用了配置文件模式，而 Ajv 需要这两个模式才能解析此引用。

然后，移除整个`validate`函数，并用以下内容替换它：

```js
function validate(req) {
  const ajvValidate = new Ajv()
    .addFormat('email', /^[\w.+]+@\w+\.\w+$/)
    .addSchema([profileSchema, createUserSchema])
    .compile(createUserSchema);

  const valid = ajvValidate(req.body);
  if (!valid) {
    // Return ValidationError
  }
  return true;
}
```

接下来，我们将创建一个 Ajv 实例，并运行`addFormat`方法来覆盖`email`格式的默认验证函数；现在，`validate`函数将使用我们提供的正则表达式来验证任何具有`email`格式的属性。

接下来，我们使用`addSchema`方法向 Ajv 提供任何引用的子模式。这允许 Ajv 跟踪引用并生成一个非引用的扁平化模式，该模式将用于验证操作。最后，我们运行`compile`方法以返回实际的验证函数。

当我们运行验证函数时，它将返回`true`（如果它是有效的）或`false`（如果它是无效的）。如果无效，`ajvValidate.errors`将填充一个包含错误数组的数组，其外观可能如下所示：

```js
[
  {
    "keyword": "type",
    "dataPath": ".bio",
    "schemaPath": "#/properties/bio/type",
    "params": {
      "type": "string"
    },
    "message": "should be string"
  }
]
```

默认情况下，Ajv 以短路方式工作，并在遇到第一个错误时立即返回`false`。因此，`ajvValidate.errors`数组默认是一个包含第一个错误详细信息的单元素数组。要指示 Ajv 返回所有错误，您必须在 Ajv 构造函数中设置`allErrors`选项，例如，`new Ajv({allErrors: true})`。

# 生成验证错误消息

当我们的对象验证失败时，我们应该生成与之前相同的人类可读错误消息。为此，我们必须处理存储在`ajvValidate.errors`中的错误，并使用它们来生成人类可读的消息。因此，在`src/validators/errors/messages.js`中创建一个新的模块，并复制以下消息生成器：

```js
function generateValidationErrorMessage(errors) {
  const error = errors[0];

  if (error.dataPath.indexOf('.profile') === 0) {
    return 'The profile provided is invalid.';
  }
  if (error.keyword === 'required') {
    return 'Payload must contain at least the email and password fields';
  }
  if (error.keyword === 'type') {
    return 'The email and password fields must be of type string';
  }
  if (error.keyword === 'format') {
    return 'The email field must be a valid email.';
  }
  return 'The object is invalid';
}

export default generateValidationErrorMessage;
```

`generateValidationErrorMessage`函数从`ajvValidate.errors`数组中提取第一个错误对象，并使用它来生成适当的错误消息。如果没有条件适用，还有一个通用的默认错误消息。

# 函数泛化

目前，`generateValidationErrorMessage`函数产生的是针对创建用户操作的特定消息。这意味着尽管代码是分离的，但逻辑仍然高度耦合到创建用户端点。这种耦合破坏了模块的目的；这是一个应该被消除的代码异味。

相反，我们应该编程`generateValidationErrorMessage`函数，使其能够为所有验证错误生成错误消息。这样做也提供了本地一致性，因为所有验证器现在都将有它们错误消息的一致结构/格式。

因此，让我们通过替换我们的`generateValidationErrorMessage`函数来做出更改：

```js
function generateValidationErrorMessage(errors) {
  const error = errors[0];
  if (error.keyword === 'required') {
    return `The '${error.dataPath}.${error.params.missingProperty}' field is missing`;
  }
  if (error.keyword === 'type') {
    return `The '${error.dataPath}' field must be of type ${error.params.type}`;
  }
  if (error.keyword === 'format') {
    return `The '${error.dataPath}' field must be a valid ${error.params.format}`;
  }
  if (error.keyword === 'additionalProperties') {
    return `The '${error.dataPath}' object does not support the field '${error.params.additionalProperty}'`;
  }
  return 'The object is not valid';
}
```

由于这个更改将破坏我们的当前实现和测试，我们必须获得产品经理的批准。如果他们批准，我们必须然后更新 E2E 测试以反映这个更改：

```js
Scenario Outline: Bad Request Payload
  ...
  And contains a message property which says "<message>"

  Examples:

  | missingFields | message                          |
 | email         | The '.email' field is missing    |
 | password      | The '.password' field is missing |

Scenario Outline: Request Payload with Properties of Unsupported Type
  ...
 And contains a message property which says "The '.<field>' field must be of type <type>"
  ...

Scenario Outline: Request Payload with invalid email format
  ...
  And contains a message property which says "The '.email' field must be a valid email"
  ...

Scenario Outline: Invalid Profile
  ...
  And contains a message property which says "<message>"

  Examples:

  | payload | message                                                   |
  | ...     | The '.profile' object does not support the field 'foo'    |
  | ...     | The '.profile.name' object does not support the field 'a' |
  | ...     | The '.profile.summary' field must be of type string       |
  | ...     | The '.profile.bio' field must be of type string           |
```

接下来，将`generateValidationErrorMessage`函数导入到我们的`src/validators/users/create.js`文件中，并更新`validateRequest`函数，以便我们可以使用它来返回一个包含错误消息的对象，如果验证失败：

```js
import generateValidationErrorMessage from '../errors/messages';

function validate(req) {
  ...
  const valid = ajvValidate(req.body);
  if (!valid) {
    return new ValidationError(generateValidationErrorMessage(ajvValidate.errors));
  }
  return true;
}
```

# 更新 npm 构建脚本

看起来一切都很顺利，但如果运行测试，它们将返回以下错误：

```js
Error: Cannot find module '../../schema/users/profile.json'
```

这是因为 Babel 默认只处理`.js`文件。因此，我们的`.json`模式文件没有被处理或复制到`dist/`目录，这导致了前面的错误。为了修复这个问题，我们可以更新我们的`build`npm 脚本来使用 Babel 的`--copy-files`标志，这将把任何不可编译的文件复制到`dist/`目录：

```js
"build": "rimraf dist && babel src -d dist --copy-files",
```

现在，如果我们再次运行我们的测试，它们应该都能通过。

# 测试成功场景

由于我们添加了验证步骤，我们现在需要确保携带有效用户负载的请求将被添加到数据库中，就像以前一样。因此，在`spec/cucumber/features/users/create/main.feature`的末尾添加以下场景概述：

```js
Scenario Outline: Valid Profile

  When the client creates a POST request to /users/
  And attaches <payload> as the payload
  And sends the request
  Then our API should respond with a 201 HTTP status code
  And the payload of the response should be a string
  And the payload object should be added to the database, grouped under the "user" type
  And the newly-created user should be deleted

  Examples:

  | payload                                                                         |
  | {"email":"e@ma.il","password":"password","profile":{}}                          |
  | {"email":"e@ma.il","password":"password","profile":{"name":{}}}                 |
  | {"email":"e@ma.il","password":"password","profile":{"name":{"first":"Daniel"}}} |
  | {"email":"e@ma.il","password":"password","profile":{"bio":"bio"}}               |
  | {"email":"e@ma.il","password":"password","profile":{"summary":"summary"}}       |
```

再次运行你的测试以确保它们通过。

# 重置我们的测试索引

目前，我们的 Elasticsearch 测试索引中充满了虚拟用户。尽管现在这并不是一个问题，但将来可能会成为问题（例如，如果我们决定更改模式）。无论如何，在测试完成后清理副作用始终是一个好习惯，以便为后续的测试运行留下空白。因此，在每个测试结束时，我们应该删除 Elasticsearch 索引。这不是问题，因为索引将由测试代码自动重新创建。

因此，将以下行添加到我们的`e2e.test.sh`脚本中；这将清理测试索引（你应该在 Elasticsearch 响应后，但在运行 API 服务器之前放置它）：

```js
# Clean the test index (if it exists)
curl --silent -o /dev/null -X DELETE "$ELASTICSEARCH_HOSTNAME:$ELASTICSEARCH_PORT/$ELASTICSEARCH_INDEX"
```

再次运行测试，它们应该仍然通过。现在，我们可以将我们的更改提交到 Git：

```js
$ git add -A && git commit -m "Fully validate Create User request payload"
```

# 摘要

在本章中，我们将我们的单体应用程序拆分成许多更小的模块，并实现了我们创建用户功能的所有要求。我们将 JSON Schema 和 Ajv 集成到我们的验证模块中，这迫使我们更一致地处理错误消息的结构。这反过来又提高了我们最终用户的使用体验。

在下一章中，我们将使用 Mocha 和 Sinon 编写单元和集成测试，这将增强我们对代码的信心。
