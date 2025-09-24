# 第五章

# 用户模块的 REST API

# 简介

任何应用程序的核心都是用户模块，这是一个基础组件，负责管理以用户为中心的功能。此模块允许用户管理用户账户，启用身份验证和授权，以及各种用户特定操作，如添加或注册用户、更新用户资料、删除用户、密码管理、基于角色的权限、日志记录等。用户模块提升了用户体验并促进了应用程序的无缝运行。

# 结构

在本章中，我们将讨论以下主题：

+   基础控制器和基础服务用于 REST API 开发

+   带输入验证的角色管理

+   带输入验证的用户管理

    +   用户入职

    +   用户登录

    +   认证和授权机制

    +   更新用户数据

    +   删除用户账户

    +   带电子邮件通知的密码管理和恢复

# 基础控制器

在上一章中，我们看到了如何为每个模块构建单独的控制器。对于每个控制器，都有一些常见的方法，例如添加、获取所有、获取单个、更新等处理程序。由于每个实体都有一个控制器，因此这些通用函数必须由每个控制器实现。我们可以编写一个抽象类供控制器扩展，并强制它们提供实现。因此，遵循既定规范，我们现在介绍一个基础概念，称为 **基础控制器**。它是一个抽象类，作为其他类继承其预定义方法的蓝图，这些方法有助于创建、更新、检索所有、检索单个和删除数据。

让我们在 utils 目录中使用以下代码创建 `**base_controller.ts**`：

`// base_controller.ts`

`import { Request, Response } from 'express';`

`export abstract class BaseController {`

`public abstract addHandler(req: Request, res: Response): void;`

`public abstract getAllHandler(req: Request, res: Response): void;`

`public abstract getOneHandler(req: Request, res: Response): void;`

`public abstract updateHandler(req: Request, res: Response): void;`

`public abstract deleteHandler(req: Request, res: Response): void;`

`}`

这个抽象类的目的是提供一个统一的框架，其他类（控制器）在实现这些方法时可以遵循。当一个类扩展了 `**BaseController**` 时，它必须提供这些抽象方法的实现。这种做法保证了跨各种路由的控制器保持一致的方法名称和参数，尽管具体的执行细节可能有所不同。

请注意，单个控制器仍然可以编写自己的额外方法。

本章的代码可以从[`github.com/ava-orange-education/Hands-on-Rest-APIs-with-Node.js-and-Express/tree/main/ch-05-code-files`](https://github.com/ava-orange-education/Hands-on-Rest-APIs-with-Node.js-and-Express/tree/main/ch-05-code-files)下载。

# 基础服务

基础服务是一个基础结构，为应用程序中管理数据操作提供常用功能。它作为蓝图，其他服务可以继承以避免重复代码并确保数据操作的一致模式。基础服务的主要目的是封装常用的数据操作，如创建、读取、更新和删除，并将它们提供给其他服务。这减少了代码重复并强制执行应用程序不同模块中的一致实践。需要数据操作的其他服务可以继承基础服务。通过扩展基础服务，这些子服务可以访问基础服务中定义的常用方法。

让我们使用以下代码创建`**base_service.ts**`：

`[`github.com/ava-orange-education/Hands-on-Rest-APIs-with-Node.js-and-Express/blob/main/ch-05-code-files/pms-be/src/utils/base_service.ts`](https://github.com/ava-orange-education/Hands-on-Rest-APIs-with-Node.js-and-Express/blob/main/ch-05-code-files/pms-be/src/utils/base_service.ts)`

如果您已经下载了本章的源代码，那么您可以在其中找到`**base_service.ts**`。我们为每个方法添加了注释，以解释其功能和工作原理。这个基础服务类提供了常见的 CRUD 操作、`**findbyIds**`以及自定义查询执行器，同时处理 API 响应和错误情况。这个服务类可以被其他服务继承。

管理数据库连接效率非常重要。如果我们为每个操作创建单独的数据库连接，那么在高负载期间应用程序可能会崩溃。使用数据库池是最佳实践，以限制和重用连接池中的连接。

让我们使用以下代码更改`**db.ts**`文件，以添加数据库池并在整个应用程序中使用单个连接：

`[`github.com/ava-orange-education/Hands-on-Rest-APIs-with-Node.js-and-Express/blob/main/ch-05-code-files/pms-be/src/utils/db.ts`](https://github.com/ava-orange-education/Hands-on-Rest-APIs-with-Node.js-and-Express/blob/main/ch-05-code-files/pms-be/src/utils/db.ts)`

`**connectDatabase**`函数负责建立数据库连接，如果可用则返回现有连接。它首先检查是否已经存在有效的连接，如果没有，则初始化一个新的连接并将其存储以供将来使用。

`**getInstance**` 函数检索数据库实例，确保在提供访问之前连接已经建立。与 `**connectDatabase**` 函数不同，`**getInstance**` 在返回之前会等待连接建立，确保它只能在连接就绪后使用一次。

`**getRepository**` 函数旨在为给定实体检索存储库实例。它会检查是否存在有效的数据库连接，并在存储库实例不存在时创建它。如果没有有效的连接，它返回 null。

# 角色管理

角色管理是应用程序安全的一个关键方面，它涉及控制和定义系统内用户的访问和权限。它确保用户根据其角色或职责执行特定操作时拥有适当的权利。角色管理对于防止未授权访问和数据泄露以及维护应用程序的整体完整性至关重要。

不同用户有不同的访问级别和权限。例如，一个应用程序可能有不同的用户角色，如 `**超级管理员**`、`**项目经理**` 和 `**访客**`，每个角色都有不同的权限集。管理员通常可以访问所有功能和功能，项目经理有有限的访问权限，而访客可能有非常受限的访问权限。

# 角色服务

角色服务将用于在继承自基础服务的数据库中执行基于角色的操作。因此，让我们在角色组件中使用以下代码创建 `**roles_service.ts**` 文件：

`// role_service.ts`

`import { Repository } from 'typeorm';`

`import { BaseService } from '../../utils/base_service';`

`import { DatabaseUtil } from '../../utils/db';`

`import { Roles } from './roles_entity';`

`export class RolesService extends BaseService<Roles> {`

`constructor() {`

`// 创建 DatabaseUtil 的实例`

`const databaseUtil = new DatabaseUtil();`

`// 获取 Roles 实体的存储库`

`const roleRepository: Repository<Roles> = databaseUtil.getRepository(Roles);`

`// 调用 BaseService 类的构造函数，并将`

`repository` 作为参数

`super(roleRepository);`

`}`

`}`

`**RolesService**` 类旨在扩展 `**BaseService**` 类提供的功能。它使用 `**DatabaseUtil**` 类获取 `**Roles**` 实体的存储库，然后将该存储库传递给 `**BaseService**` 类的构造函数。这使得 `**RolesService**` 类能够继承并使用在 `**BaseService**` 类中定义的 CRUD 方法来处理 `**Roles**` 实体。

我们将按以下方式开发角色的 REST API：

+   添加角色

+   获取所有角色

+   获取单个角色

+   更新角色

+   删除角色

在开发实际的 **添加角色** API 之前，我们需要在将角色添加到数据库时定义输入验证。因此，让我们使用 Express Validator 来验证输入请求。

# 输入验证

首先，安装 Express Validator 模块，请在 `**cmd**` 中粘贴以下命令：

`npm i express-validator --save`

现在在 utils 目录中创建名为 `**validator.ts**` 的文件，使用以下代码：

`[`github.com/ava-orange-education/Hands-on-Rest-APIs-with-Node.js-and-Express/blob/main/ch-05-code-files/pms-be/src/utils/validator.ts`](https://github.com/ava-orange-education/Hands-on-Rest-APIs-with-Node.js-and-Express/blob/main/ch-05-code-files/pms-be/src/utils/validator.ts)`

提供的代码导出一个名为 `**validate**` 的函数，该函数使用 express-validator 包生成用于验证请求数据的 `**middleware**`。此中间件函数运行提供的验证函数，使用 `**validationResult**` 检查验证错误，如果验证失败，则发送带有 400 状态和错误消息的响应。错误消息的结构与 `**IValidationError**` 接口相匹配。这种方法通常用于处理 Express 应用程序中的请求验证。

接下来，在 `**roles_routers.ts**` 文件中创建 `**validRoleInput**`，使用以下代码：

`// roles_routers.ts`

`import { Express } from 'express';`

`import { RoleController, RolesUtil } from './roles_controller';`

`import { validate } from '../../utils/validator';`

`import { body } from 'express-validator';`

`const validRoleInput = [`

`body('name').trim().notEmpty().withMessage('It should be required'),`

`body('description').isLength({ max: 200 }).withMessage('It has`

`maximum limit of 200 characters'),`

`];`

`export class RoleRoutes {`

`private baseEndPoint = '/api/roles';`

`constructor(app: Express) {`

`const controller = new RoleController();`

`app.route(this.baseEndPoint)`

`.get(controller.getAllHandler)`

`.post(validate(validRoleInput), controller.addHandler);`

`app.route(this.baseEndPoint + '/:id')`

`.get(controller.getOneHandler)`

`.put(validate(validRoleInput), controller.updateHandler)`

`.delete(controller.deleteHandler);`

`}`

`}`

在前面的类中使用了 `**baseEndPoint**` 变量。这是 API 端点的一部分，对于所有角色 API 都将是相同的。

注意到 `**validRoleInput**` 变量，它是一个数组。这个数组包含了对角色中预期每个输入字段的一系列验证检查。这个数组的每个元素都是一个验证函数，用于检查数据的特定方面。

请求体中 `**name**` 字段的验证器将按以下方式处理：

`body('name').trim().notEmpty().withMessage('It should be required')`

此验证器应用于请求体中的 `**name**` 字段，并执行以下功能：

+   `**trim()**`: 从输入中删除任何前导和尾随空白字符。

+   `**notEmpty()**`: 检查输入是否为空。

+   `**withMessage('It should be required')**`: 如果验证失败，此消息将包含在错误响应中。

类似地，`**description**`字段的验证器如下所示：

`body('description').isLength({ max: 200 }).withMessage('It has a maximum limit of 200 characters')`

此验证器应用于请求体中的`**description**`字段。它检查输入的长度不超过`**200**`个字符。

总体而言，此代码定义了一组针对角色数据不同字段的验证规则，包括检查访问权限的存在、长度和有效性。如果任何验证失败，相应的错误消息将包含在状态码为`**400: bad**`的错误响应中。

每个角色都包含一组权限。这些权限不过是字符串键，这有助于我们了解登录用户是否被分配了特定的权限，以便让他们执行相应的任务。添加任务的权限可能非常简单，如`**add_task**`，这可以添加到分配给用户的角色中。

让我们在`**common.ts**`文件中定义所有必要的应用权限，如下所示：

`[`github.com/ava-orange-education/Hands-on-Rest-APIs-with-Node.js-and-Express/blob/main/ch-05-code-files/pms-be/src/utils/common.ts`](https://github.com/ava-orange-education/Hands-on-Rest-APIs-with-Node.js-and-Express/blob/main/ch-05-code-files/pms-be/src/utils/common.ts)`

这些权限是应用权限，用于根据分配的角色在用户登录时检查权限。当创建角色时，它有一个权限值，这些应用权限以逗号分隔的形式保存。

让我们在`**role_controller.ts**`文件中使用以下代码创建一个包含`**getAllPermissionsFromRights**`函数的`**RolesUtil**`类：

`// role_controller.ts`

`export class RolesUtil {`

`/**`

`* 从定义的权限对象中的权限中检索所有可能的权限。`

`* @returns {string[]} An array of permissions`

`*/`

`public static getAllPermissionsFromRights(): string[] {`

`// 初始化一个空数组以收集值；`

`let permissions = [];`

`// 遍历 Rights 对象的每个部分`

`for (const module in Rights) {`

`// 检查当前模块是否定义了 ALL 的权限`

`if (Rights[module]['ALL']) {`

`let sectionValues = Rights[module]['ALL'];`

`sectionValues = sectionValues.split(',');`

`permissions = […permissions, …sectionValues];`

`}`

`}`

`// 返回收集到的权限`

`return permissions;`

`}`

`}`

此函数有效地编译了从定义的权限对象中可用的所有权限，然后可以在应用程序中用于验证和其他目的。

现在请使用以下代码在`**validRoleInput**`中的`**rights**`字段添加验证：

`const validRoleInput = [`

`body('name').trim().notEmpty().withMessage('It should be required'),`

`body('description').isLength({ max: 200 }).withMessage('It has maximum limit of 200 characters'),`

`body('rights').custom((value: string) => {`

`const accessRights = value?.split(',');`

`if (accessRights?.length > 0) {`

`const validRights = RolesUtil.getAllPermissionsFromRights();`

`const areAllRightsValid = accessRights.every(right =>`

`validRights.includes(right));`

`if (!areAllRightsValid) {`

`throw new Error('无效权限');`

`}`

`}`

`return true; // 验证通过`

`})`

`];`

根据提供的代码，自定义验证函数确保在添加或更新角色过程中权限的有效性。它评估请求中接收到的权限，并验证它们是否有效。如果发现任何权限无效，将生成错误。

# 添加角色

当使用 REST API 添加角色时，你通常会在请求体中提供必要的数据，例如角色的名称、描述以及它应该拥有的权限。负责此操作的 API 端点旨在接收这些数据，根据预定义的规则进行验证，并根据提供的信息创建新的角色。

我们已经创建了`**roles_controller.ts**`作为骨架类。现在，让我们使用扩展的 Base Controller 来修改它，并使用基础服务通过以下代码执行数据库操作：

`// roles_controller.ts`

`import { Response, Request } from 'express';`

`import { RolesService } from './roles_service';`

`import { BaseController } from '../../utils/base_controller';`

`import { Rights } from '../../utils/common';`

`export class RoleController extends BaseController {`

`public async addHandler(req: Request, res: Response): Promise<void> {`

`const role = req.body;`

`const service = new RolesService();`

`const result = await service.create(role);`

`res.status(result.statusCode).json(result);`

`return;`

`}`

```js```````js` `public async getAllHandler(req: Request, res: Response) {}`    `public async getOneHandler(req: Request, res: Response) {}`    `public async updateHandler(req: Request, res: Response) {}`    `public async deleteHandler(req: Request, res: Response) {}`    `}`    In this context, the ``**addHandler**`**`** method captures the data provided within the incoming request body. Subsequently, it forwards this data to the Base service by invoking the create method, which is responsible for adding the information to the database. The Base service then returns a response to this handler, signifying either a successful or unsuccessful outcome. This response is effectively transmitted as the ultimate outcome for the associated REST API operation.    Now call `**addHandler**` in role routes with change in `**roles_router.ts**` file as follows:    `//roles_router.ts`    `export class RoleRoutes {`    `private baseEndPoint = '/api/roles';`    `constructor(app: Express) {`    `const controller = new RoleController();`    `app.route(this.baseEndPoint)`    `.post(validate(validRoleInput), controller.addHandler);`    `}`    We have established routes for adding roles and incorporated middleware to validate requests before inserting data into the database.    You can test the REST APIs in Postman, cURL, or .http file with the following request and get their responses. The easiest method is to simply use VSCode, install REST Client extension, and create a new file with the .http extension and put the request as shown in *Figure 5.1*:  ![](img/5.1.jpg)  **Figure 5.1:** Making a request using VSCode REST Client Extension    For testing the API for adding a role, we need to make a `**POST**` call to `**/api/roles**`. The preceding figure shows the way it can be called with the request body.    **REST API Add Role**    **Request**    `URL : http://127.0.0.1:3000/api/roles`    `Method: POST`    `body :`    `{`    `"name":"Super Admin",`    `"description":"Having All Rights",`    `"rights": "add_role,edit_role,get_all_roles,get_details_role,`    `delete_role,`    `add_user,edit_user,get_all_users,get_details_user,delete_user,`    `add_project,edit_project,get_all_projects,get_details_project,`    `delete_project,add_task,edit_task,get_all_tasks,get_details_task,`    `delete_task,add_comment,edit_comment,get_all_comments,`    `get_details_comment,delete_comment"`    `}`    **Response**    `{`    `"statusCode": 201,`    `"status": "success",`    `"data": {`    `"role_id": "b88cc70d-ab0a-4464-9562-f6320df519f6",`    `"name": "Super Admin",`    `"description": "Having All Rights",`    `"rights": "add_role,edit_role,get_all_roles,get_details_role,`    `delete_role,add_user,edit_user,get_all_users,get_details_user,`    `delete_user,add_project,edit_project,get_all_projects,`    `get_details_project,delete_project,add_task,edit_task,get_all_tasks,`    `get_details_task,delete_task,add_comment,edit_comment,`    `get_all_comments,get_details_comment,delete_comment",`    `"created_at": "2023-08-16T16:39:14.047Z",`    `"updated_at": "2023-08-16T16:39:14.047Z"`    `}`    `}`    In case of a unique role name, trying again with the same request gives an error as 409 conflict code:    `{`    `"statusCode": 409,`    `"status": "error",`    `"message": "Key (name)=(Super Admin) already exists."`    `}`    In another case, if you change the rights as "`**rights**`"`**:**`"`**no_rights**`", it gives an error for Bad Request with 400 status as follows:    `{`    `"statusCode": 400,`    `"status": "error",`    `"errors": [`    `{`    `"rights": "Invalid permission"`    `}`    `]`    `}`    # GetAll Roles    Once a role has been successfully added to the database, we can proceed to retrieve the newly inserted roles from the database. So, let's change the `**getAllHandler**` method in the `**roles_controller.ts**` file using the following code:    `// roles_controller.ts`    `public async getAllHandler(req: Request, res: Response): Promise<void> {`    `const service = new RolesService();`    `const result = await service.findAll(req.query);`    `res.status(result.statusCode).json(result);`    `}`    The ``**getAllHandler**`**`** method uses the `**RolesService**` class to retrieve all roles from the database based on the query parameters in the request. The resulting data is then sent back to the client with an appropriate HTTP status code and formatted as JSON.    This `**controller**` method call in routes with change in `**roles_routes.ts**` as follows:    `// roles_routes.ts`    `app.route(this.baseEndPoint)`    `.post(validate(validRoleInput), controller.addHandler)`    `.get(controller.getAllHandler);`    By employing this approach, we establish a `**GET**` route that fetches all roles stored in the database, effectively functioning as a REST API endpoint for retrieving role data.    **REST API GetAll Roles**    **Request**    `URL : http://127.0.0.1:3000/api/roles`    `Method: GET`    `Query Params: {}`    **Response**    `{`    `"statusCode": 200,`    `"status": "success",`    `"data": [`    `{`    `"role_id": "b88cc70d-ab0a-4464-9562-f6320df519f6",`    `"name": "Super Admin",`    `"description": "Having All Rights",`    `"rights": "add_role,edit_role,get_all_roles,get_details_role,delete_role,add_user,edit_user,get_all_users,get_details_user,`    `delete_user,add_project,edit_project,get_all_projects,`    `get_details_project,delete_project,add_task,edit_task,`    `get_all_tasks,get_details_task,delete_task,add_comment,`    `edit_comment,get_all_comments,get_details_comment,delete_comment",`    `"created_at": "2023-08-16T16:39:14.047Z",`    `"updated_at": "2023-08-16T16:39:14.047Z"`    `}, {`    `"role_id": "5f11a06b-e9a7-438d-9f49-757e8239e238",`    `"name": "visitor",`    `"description": null,`    `"rights": null,`    `"created_at": "2023-08-15T13:04:50.314Z",`    `"updated_at": "2023-08-15T13:04:50.314Z"`    `}`    `]`    `}`    If you want to filter or search by exact name, you can change query params as follows:    `URL : http://127.0.0.1:5000/api/roles?name=visitor`   ``````js```` `It gives only matched data as a response, as follows`   ```js`````` `{`    `"statusCode": 200,`    `"status": "success",`    `"data": [`    `{`    `"role_id": "5f11a06b-e9a7-438d-9f49-757e8239e238",`    `"name": "visitor",`    `"description": null,`    `"rights": null,`    `"created_at": "2023-08-15T13:04:50.314Z",`    `"updated_at": "2023-08-15T13:04:50.314Z"`    `}`    `]`    `}`    **注意**：在基础服务中，查询参数现在将仅适用于精确匹配。我们将在稍后探索搜索功能。    # 获取单个角色    获取单个角色端点是角色管理系统的一个基本部分，允许用户查看特定角色的信息，而无需检索所有角色的完整列表。这对于提供针对每个角色属性和权限的针对性见解至关重要。要实现获取单个角色 API，请在角色控制器中的`**getOneHandler**`代码中进行以下更改：    `// roles_controller.ts`    `public async getOneHandler(req: Request, res: Response): Promise<void> {`    `const service = new RolesService();`    `const result = await service.findOne(req.params.id);`    `res.status(result.statusCode).json(result);`    `}`    `**getOneHandler**`函数作为传入客户端请求、与数据库交互的服务层和传出 HTTP 响应之间的桥梁。它根据提供的角色 ID 从数据库中检索单个角色的详细信息，并将角色信息发送回客户端。    该方法通过在路由文件中创建一个新的路由来从路由文件中调用：    `// roles_routes.ts`    `app.route(this.baseEndPoint + '/:id')`    `.get(controller.getOneHandler);`    这里****，`**/:id**`将是一个请求参数，用于捕获用户想要检索的角色 ID。    **REST API 获取单个角色**    **请求**    `URL: http://127.0.0.1:3000/api/roles/5f11a06b-e9a7-438d-9f49-757e8239e238`    `方法：GET`    **响应**    `{`    `"statusCode": 200,`    `"status": "success",`    `"data": {`    `"role_id": "5f11a06b-e9a7-438d-9f49-757e8239e238",`    `"name": "visitor",`    `"description": null,`    `"rights": null,`    `"created_at": "2023-08-15T13:04:50.314Z",`    `"updated_at": "2023-08-15T13:04:50.314Z"`    `}`    `}`    提供有效的角色 ID 将产生成功响应，而输入一个与现有数据库条目不对应的 ID 将导致`**404 错误**`，表示请求的实体未找到。    `{`    `"statusCode": 404,`    `"status": "error",`    `"message": "Not Found"`    `}`    # 更新角色    更新角色涉及修改存储在数据库中的特定角色的现有数据。此过程允许您调整角色的属性，例如其名称、描述和关联的权限。通过执行更新，您可以确保角色的信息保持准确和最新。当角色的权限发生变化时，此操作特别有用，您需要将这些更改反映在数据库中。    要实现更新角色 API，请在角色控制器中的`**updateHandler**`代码中进行以下更改：    `// roles_controller.ts`    `public async updateHandler(req: Request, res: Response) : Promise<void> {`    `const role = req.body;`    `const service = new RolesService();`    `const result = await service.update(req.params.id, role);`    `res.status(result.statusCode).json(result);`    `}`    `**updateHandler**`函数旨在处理数据库中角色的更新。    它通过从传入的 HTTP 请求体中检索数据来操作，然后利用这些数据根据请求参数 ID（即角色的唯一标识符`**role_id**`）更新数据库中的相应角色数据。该函数随后生成一个响应，指示更新操作是成功还是失败，如果需要，提供有关更新数据或适当的错误消息。    该方法通过在路由文件中创建一个新的路由来从路由文件中调用：    `// roles_routes.ts`    `app.route(this.baseEndPoint + '/:id')`    `.
