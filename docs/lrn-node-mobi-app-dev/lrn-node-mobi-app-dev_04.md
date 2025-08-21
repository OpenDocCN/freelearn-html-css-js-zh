# 第四章：保护您的后端

在之前的章节中，我们逐层构建了一个基本的后端层，为一个类似商店的应用程序提供基本服务。到目前为止，我们还没有过多关注安全性；即使涉及删除整个产品数据库，只要能够访问服务器的人都可以执行我们的 API 公开的任何命令！

在本章中，我们将通过构建基本的安全机制来控制用户访问权限。具体来说，我们将处理基于令牌的身份验证，并向您展示如何通过这种方式轻松限制对后端的访问。通过这样做，我们将介绍角色的概念以及它们在我们的身份验证方案中的作用。

# 理解基于令牌的身份验证的结果

阅读完本章后，您将了解令牌验证是什么，以及如何使用它来根据用户状态限制 API 的功能。您还将了解角色是什么，以及它们如何影响验证。最后，您将知道如何仅使用我们迄今介绍的技术来实现此身份验证机制。

## 理论部分

在我们开始编写代码之前，让我们先概述涉及的概念以及它们如何与安全和彼此相关。

## 信任的小令牌

安全一直是软件开发各个领域中最紧迫的关注点之一。如果系统没有足够的机制来保护免受恶意用户的侵害，即使它快速、可扩展和稳健，也几乎永远不够。

在像我们这样的公开可访问的服务器环境中，安全性更加紧迫，因为我们的 API 将暴露给整个可能有恶意的人的地球。不知何故，我们需要确保从中请求服务的人是他们所说的那个人，并且被允许做他们想做的事情。

一个简单而强大的技术已经出现，用于实现这一点，即**基于令牌的身份验证**。在这种情况下，每个合法用户都会被分配一个访问令牌（通常是哈希），用于唯一标识服务器的用户。用户需要在需要身份验证的每个请求中提交令牌，服务器反过来验证令牌以确定是否应该授予访问权限。

为了获得访问令牌，用户首先需要以某种方式最初向服务器进行身份验证。通常，这是通过正常的用户名密码检查完成的。如果提供了正确匹配的用户名和密码，服务器会生成一个访问令牌，证明用户已经经过身份验证可以访问服务器。

## 扮演你的角色

在大多数软件系统中，并非所有用户都是平等的。一些用户，比如管理员，意图上具有比普通用户更广泛的系统访问权限。有几种方案可以限制用户在系统中可以访问的功能，但最常见的可能是使用角色。简而言之，角色是授予持有者对系统一定级别访问权限的属性。例如，具有**管理员**角色的用户可能具有完全访问读取和写入系统记录的权限，而具有**读者**角色的用户可能只能读取它们。此外，具有**书虫**角色的用户可能只能访问被分类为**书籍**的数据记录，依此类推。

## 将所有内容整合在一起

现在，角色和令牌在我们想要创建的身份验证方案中的作用可能是显而易见的。经过身份验证的请求的生命周期将按以下方式进行：

1.  服务器接收 API 请求。

1.  服务器检查是否提供了令牌。

如果没有提供，它会返回*403*（即**禁止**）。

1.  服务器检查令牌是否在数据库中。

如果数据库中没有提供，它会返回*403*。

1.  服务器检索用户的角色。

1.  服务器验证用户的角色是否符合 API 调用的要求。

如果不匹配，它会返回 *403*。

1.  服务器处理请求并向用户返回适当的响应。

## 实施

我们现在准备编写认证系统的功能实现。

我们需要做的第一件事是扩展我们的数据库以容纳必要的文档。特别是，我们需要添加以下三个新集合：

+   **用户**：这些是可以通过 API 访问服务器的用户

+   **角色**：这些是可以分配给用户的角色

+   **访问令牌**：这些是经过认证的用户的访问令牌

我们还需要向我们的 API 添加一些基本逻辑来注册用户并使其能够登录。

# 添加新的集合

打开你的 MongoDB shell 并执行以下操作：

```js
 use OrderBase;
 db.createCollection('User');
 db.createCollection('Role');
 db.createCollection('AccessToken');

```

这将创建我们需要存储用户、角色和令牌的必要集合。新文档将具有以下结构：

```js
User:
{
    firstName,
    lastName,
    email,
    roleID,
    password
}
```

现在，我们不会添加任何用户或令牌（这将在我们扩展 API 时进行），但我们将添加我们将要使用的角色。为了简单起见，我们只会有两个角色：

+   **生产者**：这是在商店销售商品并可以向其添加额外产品的用户。

+   **顾客**：这是从商店购买物品并可以创建订单以及检索有关产品和当前用户创建的订单的信息的用户。

可以理解的是，由 MongoDB 生成的默认 `ObjectID` 将包含在前面的代码中。对于访问令牌实体，我们只需使用 `ObjectID` 作为令牌的哈希值，因为这个值在我们正在使用的数据库中是唯一的。

# 添加认证模块

为了保持模块化并简化认证过程，我们将创建一个单独的模块来验证给定用户的访问权限。

在你的项目目录中，添加名为 `authentication.js` 的文件。打开文件并插入以下内容：

```js
var db = require('./database');

module.exports = {
  database: 'OrderBase',
  collection: 'AccessTokens',
  generateToken: function (user, callback) {
    var token = {
      userID: user._id
    }
  }

  // Persist and return the token
  db.insert(this.database, this.collection, token, function (err, res) {
    if (err) {
      callback(err, null);
    } else {
      callback(null, res);
    }
  });
},
authenticate: function (user, password, callback) {
  if (user.password ==== password) {
    // Create a new token for the user
    this.generateToken(user, function (err, res) {
      callback(null, res);
  });});
  } else {
    callback({
      error: 'Authentication error',
      message: 'Incorrect username or password'
    }, null);
  }
}
}
```

接下来，将模块导入到你的入口模块中，如下所示：

```js
var authentication = require('./authentication');

```

## 创建注册和帮助用户登录的函数

我们需要为我们的 API 添加端点，用于创建和认证希望与其交互的用户。鉴于我们迄今为止所做的工作，这很容易实现。

### 注册用户

我们首先要添加一个用于添加用户的 URL 端点。这在添加 REST API 时将会非常熟悉；我们要做的就是为 `user` 集合创建一个 `POST` 方法。首先，添加以下实用方法：

```js
var insertUser = function (user, req, res) {
  insertResource('OrderBase', 'User', user, function (err, result) {
    res.writeHead(200, {"Content-Type": "application/json"});
    res.end(JSON.stringify(result));
  });
};
```

接下来，修改你的路由器以包括以下 `case` 语句：

```js
case 'api/users/register':
  if (req.method === 'POST') {
    var body = "";
    req.on('data', function (dataChunk) {
      body += dataChunk;
    });
    req.on('end', function () {

      // Done pulling data from the POST body.
      // Turn it into JSON and proceed to store.
      var postJSON = JSON.parse(body);

      // validate that the required fields exist
      if (postJSON.email
      && postJSON.password
      && postJSON.firstName
      && postJSON.lastName) {
        insertUser(postJSON, req, res);
      } else {
        res.end('All mandatory fields must be provided');
      }
    });
  }
  break;
```

这就是我们注册用户所需要的全部。现在可以通过对 `/api/users/register` 端点进行简单的 `POST` 请求来处理注册。

### 使用户能够登录

为了使用户能够通过我们的 API 登录，我们需要完成以下三件事：

+   确保用户存在

+   确保用户提供了匹配的密码

+   返回一个访问令牌，用户可以在将来使用

幸运的是，前面列表中除了第一个之外的所有内容都已经被我们之前设计的认证模块处理了。我们需要做的就是将其插入到我们的路由器中。为此，我们还需要为登录部分设计一个新的端点。

在你的路由器配置中添加以下 `case`：

```js
case 'api/users/login':
  if (req.method === 'POST') {
    var body = "";
    req.on('data', function (dataChunk) {
      body += dataChunk;
    });
    req.on('end', function () {

      var postJSON = JSON.parse(body);

      // make sure that email and password have been provided
      if (postJSON.email && postJSON.password) {
        findUserByEmail(postJSON.email, function (err, user) {
          if (err) {
            res.writeHead(404, {"Content-Type": "application/json"});
            res.end({
              error: "User not found",
              message: "No user found for the specified email"
            });
          } else {
            // Authenticate the user
            authenticator.authenticate(
            user, postJSON.password, function(err, token) {
              if(err) {
                res.end({
                  error: "Authentication failure",
                  message: "User email and password do not match"
                });
              } else {
                res.writeHead(200, {"Content-Type": "application/json"});
              res.end(JSON.stringify(token));
              }
            });
          }
        });
      });

    } else {
      res.end('All mandatory fields must be provided');
    }
  });
}
  break;
```

在前面的代码中，我们添加了以下简单的方法来处理通过电子邮件查找用户：

```js
var findUserByEmail = function (email, callback) {
  database.find('OrderBase', 'User', {email: email}, function (err, user) {
    if (err) {
      callback(err, null);
    } else {
      callback(null, user);
    }
  });
};
```

就目前而言，这就是我们所需要的关于用户管理的全部。现在，让我们添加最后的修饰，并为我们的端点设置实际的安全性。

## 扩展我们的 API

我们现在准备修改我们的 API，以添加我们迄今为止开发的认证功能。首先，让我们确定访问策略应该如何工作：

+   **客户**应能够创建（`insert`）订单并检索（`get`）有关产品的信息，除此之外什么都不能做。

+   **生产者**应能够检索有关订单和产品的信息，并插入新产品

我们将通过在每个端点上放置一个简单的令牌和角色检查来实现这一点。检查将简单验证以下内容：

+   令牌是合法的

+   与令牌关联的用户具有执行所请求操作所必需的角色

首先，我们将在`authentication`模块中添加一个新函数，该函数将负责检查给定的令牌是否与给定的角色相关联：

```js
tokenOwnerHasRole: function (token, roleName, callback) {
  var database = this.database;
  db.find(database, 'User', {_id: token.userID}, function (err, user) {
    db.find(database, 'Role', {_id: user.roleID}, function (err, role) {
      if(err){
        callback(err, false);
      }
      else if (role.name ==== roleName) {
        callback(null, true);
      }
      else {
        callback(null, false);
      }
    });
  });
}
```

这种方法是我们需要验证提供的令牌的角色（隐式检查拥有令牌的用户是否具有指定的角色）。

接下来，我们只需要在我们的路由器中使用它。例如，让我们保护我们的产品 API 的`POST`端点。使其看起来像下面这样：

```js
case '/api/products':
  if (req.method === 'GET') {
    // Find and return the product with the given id
    if (parsedUrl.query.id) {
      findProductById(id, req, res);
    }
    // There is no id specified, return all products
    else {
      findAllProducts(req, res);
    }
  }
  else if (req.method === 'POST') {
    var body = "";
    req.on('data', function (dataChunk) {
      body += dataChunk;
    });
    req.on('end', function () {
      var postJSON = JSON.parse(body);

      // Verify access rights
      getTokenById(postJSON.token, function (err, token) {
        authenticator.tokenOwnerHasRole(token, 'PRODUCER', function (err, result) {
          if (result) {
            insertProduct(postJSON, req, res);
          } else {
          res.writeHead(403, {"Content-Type": "application/json"});
            res.end({
              error: "Authentication failure",
              message: "You do not have permission to perform that action"
            });
          }
        });
      });
    });
  }
  break;
```

就是这样！其他端点的实现方式相同，我们将为您提供它们的完整示例源代码。

尽管我在这里涵盖了一些基础知识，但安全仍然是当代软件开发中最大和最多样化的领域之一。我们相信基于令牌的身份验证将解决您在职业生涯中遇到的大多数情况。我还想为未来的学习提供一些建议，以及对您在这里学习的主题的补充。

## OAuth

现代 Web 应用程序提供的最常见的身份验证标准之一是**OAuth**（**开放身份验证标准**），特别是它的第二版（**OAuth2**）。OAuth 大量使用访问令牌，并且被（包括但不限于）Facebook、Google、Twitter、Reddit 和 StackOverflow 使用。该标准的一部分强大之处在于它允许用户使用他们的 Google 或 Facebook 帐户，甚至是支持 OAuth2 的其他帐户，来使用您的服务进行登录。

有几个成熟的 NPM 包可用于在 Node.js 中使用 OAuth2。特别是，我们建议您研究 node-oauth2-server 包（[`github.com/thomseddon/node-oauth2-server`](https://github.com/thomseddon/node-oauth2-server)）。

## 时间戳访问令牌

为了简化事情并专注于主要概念，我们在此示例中允许我们的访问令牌是永久的。这是一种非常糟糕的安全实践，因为令牌和密码一样，可能会被泄露并用于授予未经授权的用户对系统的访问权限。

减少这种危险的常见方法是对每个访问令牌施加**生存时间**（**TTL**）值，指示令牌可以使用多长时间，直到用户必须再次进行身份验证以获取新令牌。

## 密码哈希

为了简单起见，我们允许在此示例中将密码存储和检索为纯文本。不用说，这是一种糟糕的安全实践，绝对不应该在生产服务器上执行。成熟的 Node.js 框架（如 Express.js）提供了用于哈希密码的内置机制，当可用时，您应该始终选择这些机制。如果您需要自行对密码进行哈希处理，请选择`bcrypt`模块以进行哈希和比较。以下是相同的示例：

```js
var bcrypt = require('bcrypt');

var userPlaintextPassword = "ISecretlyLoveUnicorns";
var userHashedPassword = "";

// First generate a salt value to hash the password with
bcrypt.genSalt(10, function(err, salt) {
  // Hash the password using the salt value
  bcrypt.hash(userPlaintextPassword, salt,
  function(err, hashedPassword) {
    // We now have a fully hashed password
    userHashedPassword = hashedPassword;
  });
});

// Use the same module to compare the hashed password with potential
//matches.
bcrypt.compare("ISecretlyLoveUnicorns", userHashedPassword,
  function(err, result) {
    // Result will simply be true if hashing succeeded.
  });
bcrypt.compare("ISecretlyHateUnicorns", userHashedPassword,
  function(err,  result) {
    // result will be false if the comparison fails
});
```

# 总结

在本章中，您了解了基于令牌的身份验证，并看到了它如何在实践中加强后端。为了将其付诸实践，我们编写了一个简单的基于令牌的访问系统，以保护对一组后端数据的访问。我们的服务器现在几乎完成了，但我们仍然必须处理一些现代应用程序需要面对的其他紧迫问题。

在下一章中，我们将探讨如何解决其中一个最重要的问题。
