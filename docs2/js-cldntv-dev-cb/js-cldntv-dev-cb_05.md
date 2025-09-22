# 第五章：保护云原生系统

在本章中，将介绍以下菜谱：

+   保护您的云账户

+   创建联合身份池

+   实现注册、登录和注销

+   使用 OpenID Connect 保护 API 网关

+   实现自定义授权器

+   授权基于 GraphQL 的服务

+   实现 JWT 过滤器

+   使用信封加密

+   为传输加密创建 SSL 证书

+   配置 Web 应用程序防火墙

+   复制数据湖以进行灾难恢复

# 简介

云中的安全基于共享责任模型。在栈的某个以下线是云提供商的责任，以上线是云消费者的责任。云原生和无服务器计算将这条线推得越来越高。这使得团队能够专注于他们最擅长的领域——他们的业务领域。借助云提供的安全机制，团队能够实践 *设计安全* 并专注于 *深度防御* 技术来保护他们的数据。到目前为止，在每一个菜谱中，我们都看到了无服务器计算如何要求我们在栈的每一层定义组件之间的安全策略。本章中的菜谱将涵盖保护我们的云账户、使用 OAuth 2.0/Open ID Connect 保护我们的应用程序、保护我们的静态数据以及通过将安全的一些方面委托给云边缘来在我们的云原生系统周围创建一个边界。

# 保护您的云账户

如果我们不努力保护我们的云账户，那么我们所做的一切来保护我们的云原生系统都是徒劳的。我们必须为每个创建的云账户实施一系列最佳实践。随着我们努力创建自主服务，我们应该利用云账户之间的自然隔板，通过将相关服务分组到更多、更细粒度的账户中，而不是更少、更粗粒度的账户中。在这个菜谱中，我们将看到将账户视为代码如何使我们能够通过应用我们用于管理许多自主服务的相同基础设施即代码实践，轻松地管理许多账户。

# 如何做到这一点...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch5/account-as-code --path cncb-account-as-code
```

1.  使用 `cd cncb-account-as-code` 切换到 `cncb-account-as-code` 目录。

1.  查看名为 `serverless.yml` 的文件，其内容如下：

```js
service: cncb-account-as-code

provider:
  name: aws
  # cfnRole: arn:aws:iam::${self:custom.accountNumber}:role/${opt:stage}-cfnRole

custom:
  accountNumber: 123456789012

resources:
  Resources:
    AuditBucket:
      Type: AWS::S3::Bucket
      DeletionPolicy: Retain
      ...
    CloudTrail: 
      Type: AWS::CloudTrail::Trail
      ...

    CloudFormationServiceRole:
      Type: AWS::IAM::Role
      Properties: 
        RoleName: ${opt:stage}-cfnRole
        ...
    ExecuteCloudFormationPolicy: 
      Type: AWS::IAM::ManagedPolicy
      ...
    CiCdUser: 
      Type: AWS::IAM::User
      Properties:
        ManagedPolicyArns:
          - Ref: ExecuteCloudFormationPolicy 

    AdminUserGroup: 
      Type: AWS::IAM::Group
      ...
    ReadOnlyUserGroup: 
      Type: AWS::IAM::Group
      ...
    PowerUserGroup: 
      Type: AWS::IAM::Group
      ...
    ManageAccessKey:
      Condition: IsDev
      Type: AWS::IAM::ManagedPolicy
      ...
    MfaOrHqRequired:
      Condition: Exclude
      Type: AWS::IAM::ManagedPolicy
      ...
  ... 
```

1.  在 `serverless.yml` 中更新 `accountNumber`。

1.  使用 `npm install` 安装依赖

1.  使用 `npm test` 运行测试

1.  查看在 `.serverless` 目录中生成的内容。

1.  部署栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-account-as-code@1.0.0 dp:lcl <path-to-your-workspace>/cncb-account-as-code
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
```

1.  在 AWS 控制台中查看栈和资源。

1.  取消注释 `cfnRole`，然后使用带有 `force` 标志的 `npm run dp:lcl -- -s $MY_STAGE --force` 再次部署。

1.  使用 `npm run rm:lcl -- -s $MY_STAGE` 删除栈

# 它是如何工作的...

我创建的每个账户都使用`serverless.yml`文件开始，例如本食谱中的文件。在创建此账户范围堆栈之前，我不会在账户中创建其他堆栈。除创建用户之外的所有进一步更改都作为对此堆栈的更改交付。此堆栈的首要责任是启用`CloudTrail`。在第七章*优化可观察性*中，我们将看到如何使用此审计跟踪来监控和警报关于安全策略意外更改的情况。"AuditBucket"也是复制到恢复账户的候选者，如*复制数据湖以进行灾难恢复*食谱中讨论的那样。

接下来，堆栈创建用于授予所有账户用户权限的用户组。`AdminUserGroup`、`PowerUserGroup`和`ReadOnlyUserGroup`组是一个良好的起点，同时使用 AWS 提供的托管策略。随着账户使用的成熟，这些组将使用第六章中讨论的相同方法进行演变，即*构建持续部署管道*。然而，只有安全策略被编码化。用户到组的分配是一个手动过程，应该遵循适当的审批流程。堆栈包括`MfaOrHqRequired`策略，要求**多因素认证**（**MFA**）并允许企业 IP 地址白名单，但最初是禁用的。对于所有生产账户，肯定应该启用它。在开发账户中，大多数开发者被分配到高级用户组，这样他们可以自由地实验云服务。高级用户组没有 IAM 权限，因此包含一个可选的`ManageAccessKey`策略，允许高级用户管理他们的访问密钥。请注意，控制访问密钥的使用并频繁轮换它们非常重要。

当执行`serverless.yml`文件时，我们需要一个访问密钥。作为额外的安全措施，CloudFormation 支持使用服务角色，允许 CloudFormation 假定具有临时凭证的特定角色。使用`serverless.yml`文件中的`cfnRole`属性启用此功能。此堆栈创建一个初始的`CloudFormationServiceRole`，所有堆栈都应使用此角色。随着账户的成熟，此角色应调整到尽可能少的权限。包含的`ExecuteCloudFormationPolicy`只有足够的权限来执行`serverless.yml`文件。此策略将由`CiCdUser`使用，我们将在第六章*构建持续部署管道*中使用它。

# 创建联合身份池

管理用户是几乎所有系统的要求。在漫长的职业生涯中，我确实可以证明反复创建身份管理功能。幸运的是，我们现在可以从许多提供商那里获得此功能，包括我们的云提供商。由于有这么多选项可用，我们需要一个联邦解决方案，它将委托给许多其他身份管理系统，同时向我们的云原生系统呈现一个单一、统一的模型。在此配方中，我们将展示如何创建一个 *AWS Cognito 用户池*，然后我们将在其他配方中使用它来保护我们的服务。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch5/cognito-pool --path cncb-cognito-pool
```

1.  使用 `cd cncb-cognito-pool` 命令进入 `cncb-cognito-pool` 目录。

1.  检查名为 `serverless.yml` 的文件，其内容如下：

```js
service: cncb-cognito-pool

provider:
  name: aws
  # cfnRole: arn:aws:iam::<account-number>:role/${opt:stage}-cfnRole

resources:
  Resources:
    CognitoUserPoolCncb:
      Type: AWS::Cognito::UserPool
      Properties:
        UserPoolName: cncb-${opt:stage}
        ...
        Schema:
          - AttributeDataType: 'String'
            DeveloperOnlyAttribute: false
            Mutable: true
            Name: 'email'
            Required: true
        ...
    CognitoUserPoolCncbClient:
      Type: AWS::Cognito::UserPoolClient
      Properties:
        UserPoolId:
          Ref: CognitoUserPoolCncb

  Outputs:
    ...

plugins:
  - cognito-plugin

custom:
  pool:
    domain: cncb-${opt:stage}
    allowedOAuthFlows: ['implicit']
    allowedOAuthFlowsUserPoolClient: true
    allowedOAuthScopes: ...
    callbackURLs: ['http://localhost:3000/implicit/callback']
    logoutURLs: ['http://localhost:3000']
    refreshTokenValidity: 30
    supportedIdentityProviders: ['COGNITO']
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行测试。

1.  检查 `.serverless` 目录中生成的内容。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-cognito-pool@1.0.0 dp:lcl <path-to-your-workspace>/cncb-cognito-pool
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...

Stack Outputs
userPoolArn: arn:aws:cognito-idp:us-east-1:123456789012:userpool/us-east-1_tDXH8JAky
userPoolClientId: 4p5p08njgmq9sph130l3du2b7q
loginURL: https://cncb-john.auth.us-east-1.amazoncognito.com/login?redirect_uri=http://localhost:3000/implicit/callback&response_type=token&client_id=4p5p08njgmq9sph130l3du2b7q
userPoolProviderName: cognito-idp.us-east-1.amazonaws.com/us-east-1_tDXH8JAky
userPoolId: us-east-1_tDXH8JAky
userPoolProviderURL: https://cognito-idp.us-east-1.amazonaws.com/us-east-1_tDXH8JAky
```

1.  在 AWS 控制台中检查堆栈和资源。

1.  从堆栈输出访问 `loginURL` 以验证用户池是否已成功创建。

1.  完成后，使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

我们将在本章的其他配方中引用此用户池，因此您完成本章后请删除此堆栈。

# 它是如何工作的...

此配方仅将 AWS Cognito 用户池的创建过程规范化。像任何其他基础设施资源一样，我们希望将其视为代码并在持续交付管道中管理。对于此配方，我们定义了最基本的内容。您可能需要定义任何希望池管理的附加属性。对于此配方，我们仅指定了 `email` 属性。

重要的是要理解，这些属性不能通过 CloudFormation 进行更改，除非创建一个新的用户池，这将对其依赖的用户池的任何资源产生连锁反应。因此，请预计您需要投入一些精力来提前实验正确的属性组合。

对于将依赖于此用户池的每个应用程序，我们需要定义一个 `UserPoolClient`。每个应用程序通常会将其定义在它管理的堆栈中。然而，过度使用单个用户池是很重要的。这再次是一个关于自主性的问题。如果用户池和那些用户使用的应用程序确实是独立的，那么它们应该分别管理在不同的 Cognito 用户池中，即使这需要一些重复的工作。例如，如果您发现自己正在使用 Cognito 用户组编写复杂的逻辑来不自然地隔离用户，那么您可能更适合使用多个用户池。一个误用的例子是将员工和客户混合在同一个用户池中。

截至撰写本章时，CloudFormation 对 Cognito API 的支持并不完整。因此，使用插件来处理 `UserPoolClient` 的附加设置，例如 `domain`、`callbackUrls` 和 `allowedOAuthFlows`。

# 实现注册、登录和注销

重复实现注册、登录和注销并不高效。在单页应用程序中实现此逻辑也不理想。在这个菜谱中，我们将了解如何在单页应用程序中实现*OpenID Connect 隐式流*以使用*AWS Cognito 托管 UI*对用户进行认证。

# 如何实现...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch5/cognito-signin --path cncb-cognito-signin
```

1.  使用`cd cncb-cognito-signin`导航到`cncb-cognito-signin`目录。

1.  查看名为`src/App.js`的文件，其内容如下，并使用上一个菜谱中`userPool`堆栈输出的值更新`clientId`和`domain`字段：

```js
import React, { Component } from 'react';
import { BrowserRouter as Router, Route } from 'react-router-dom';
import { CognitoSecurity, ImplicitCallback, SecureRoute } from './authenticate';
import Home from './Home';

class App extends Component {
  render() {
    return (
      <Router>
        <CognitoSecurity
          domain='cncb-<stage>.auth.us-east-1.amazoncognito.com'
          clientId='a1b2c3d4e5f6g7h8i9j0k1l2m3'
          ...
          redirectSignIn={`${window.location.origin}/implicit/callback`}
          redirectSignOut={window.location.origin}
        >
          <SecureRoute path='/' exact component={Home} />
          <Route path='/implicit/callback' component={ImplicitCallback} />
        </CognitoSecurity>
      </Router>
    );
  }
}

export default App;
```

1.  查看名为`src/Home.js`的文件，其内容如下：

```js
import React from 'react';
import { withAuth } from './authenticate';
...

const Home = ({ auth }) => (
  <div ...>
    ...
    <button onClick={auth.logout}>Logout</button>
    <pre ...>{JSON.stringify(auth.getSession(), null, 2)}</pre>
  </div>
);

export default withAuth(Home);
```

1.  使用`npm install`安装依赖项。

1.  使用`npm start`在本地运行应用程序。

1.  浏览到`http://localhost:3000`。

1.  点击`注册`链接并按照说明操作。

1.  点击`注销`链接，然后再次登录。

1.  查看页面上的`idToken`。

我们将使用这些屏幕在授权菜谱中创建有效的`idToken`。

# 工作原理...

将注册、登录和注销功能添加到单页应用中非常简单。我们包括一些额外的库，用适当的配置初始化它们，并装饰现有的代码。在这个菜谱中，我们对我们的简单 React 应用程序进行了这些更改。AWS 提供了`amazon-cognito-auth-js`库来简化这项任务，我们在`src/authenticate`文件夹中用一些 React 组件包装它。首先，我们在`src/App.js`中初始化`CognitoSecurity`组件。接下来，我们为`Home`组件设置`SecureRoute`，如果用户未认证，将重定向到*Cognito 托管 UI*。`ImpicitCallback`组件将在用户登录后处理重定向。最后，我们将`withAuth`装饰器添加到`Home`组件上。在更复杂的应用程序中，我们只需装饰更多的路由和组件。框架处理其他所有事情，例如将**JSON Web Token**（JWT）保存到本地存储，并在`auth`属性中使用它。例如，`Home`组件显示令牌（`auth.getSession()`）并提供注销按钮（`auth.logout`）。

# 使用 OpenID Connect 保护 API 网关

使用 API 网关的一个优点是将安全关注点，如授权，推到我们系统的外围，远离我们的内部资源。这简化了内部代码并提高了可扩展性。在这个菜谱中，我们将配置一个*AWS API 网关*以对*AWS Cognito 用户池*进行授权。

# 准备工作

您需要根据*创建联合身份池*菜谱中创建的 Cognito 用户池和*实现注册、登录和注销*菜谱中创建的示例应用程序来创建本菜谱中使用的身份令牌。

# 如何实现...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch5/cognito-authorizer --path cncb-cognito-authorizer
```

1.  使用`cd cncb-cognito-authorizer`进入`cncb-cognito-authorizer`目录。

1.  检查名为`serverless.yml`的文件，其内容如下：

```js
service: cncb-cognito-authorizer

provider:
  name: aws
  # cfnRole: arn:aws:iam::<account-number>:role/${opt:stage}-cfnRole
  ...

functions:
  hello:
    handler: handler.hello
    events:
      - http:
          ...
          authorizer:
            arn: ${cf:cncb-cognito-pool-${opt:stage}.userPoolArn}
```

1.  使用`npm install`安装依赖项。

1.  使用`npm test -- -s $MY_STAGE`运行测试。

1.  检查`.serverless`目录中生成的内容。

1.  部署栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-cognito-authorizer@1.0.0 dp:lcl <path-to-your-workspace>/cncb-cognito-authorizer
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
endpoints:
  GET - https://ff70szvc44.execute-api.us-east-1.amazonaws.com/john/hello
functions:
  hello: cncb-cognito-authorizer-john-hello
```

1.  在 AWS 控制台中检查栈和资源。

1.  在更新`<API-ID>`后，调用以下`curl`命令，尝试在不使用`Authorization`令牌的情况下访问服务：

```js
$ curl https://<API-ID>.execute-api.us-east-1.amazonaws.com/$MY_STAGE/hello | json_pp

{
   "message" : "Unauthorized"
}
```

1.  使用前一个菜谱中的应用程序登录并生成`idToken`，然后使用以下命令导出`idToken`：

```js
$ export CNCB_TOKEN=<idToken value>
```

1.  在更新`<API-ID>`后，调用以下`curl`命令，以使用`Authorization`令牌成功访问服务：

```js
$ curl -v -H "Authorization: Bearer $CNCB_TOKEN"  https://<API-ID>.execute-api.us-east-1.amazonaws.com/$MY_STAGE/hello | json_pp

{
   "message" : "JavaScript Cloud Native Development Cookbook! Your function executed successfully!",
   "input" : {
      "headers" : {
         "Authorization" : "...",
         ...
      },
      ...
      "requestContext" : {
         ...
         "authorizer" : {
            "claims" : {
               "email_verified" : "true",
               "auth_time" : "1528264383",
               "cognito:username" : "john",
               "event_id" : "e091cd96-694d-11e8-897c-e3fe55ba3d67",
               "iss" : "https://cognito-idp.us-east-1.amazonaws.com/us-east-1_tDXH8JAky",
               "exp" : "Wed Jun 06 06:53:03 UTC 2018",
               "sub" : "e4bdd021-a160-4aff-bce2-e652f9469e3e",
               "aud" : "4p5p08njgmq9sph130l3du2b7q",
               "email" : "john@example.com",
               "iat" : "Wed Jun 06 05:53:03 UTC 2018",
               "token_use" : "id"
            }
         }
      },
      ...
   }
}
```

1.  使用`npm run rm:lcl -- -s $MY_STAGE`删除栈，一旦您完成操作。

# 它是如何工作的...

AWS API 网关、AWS Cognito 和 Serverless Framework 的组合使得使用 OpenID Connect 保护服务变得极其简单。AWS API 网关可以使用授权器函数来控制对服务的访问。这些函数验证`Authorization`头中传递的 JWT，并返回 IAM 策略。我们将在*实现自定义授权器*菜谱中深入了解这些细节。AWS Cognito 提供了一个授权器函数，用于验证特定用户池生成的 JWT。在`serverless.yml`文件中，我们只需将`authorizer`设置为特定 Cognito 用户池的`userPoolArn`。一旦授权，API 网关将 JWT 的解码`claims`传递到`requestContext`中的 lambda 函数，以便在业务逻辑中（如果需要）使用这些数据。

# 实现自定义授权器

在*使用 OpenID Connect 保护 API 网关*的菜谱中，我们利用了 AWS 提供的 Cognito 授权器。这是使用 Cognito 的一个优点。然而，这并不是唯一的选择。有时我们可能希望对返回的策略有更多的控制。在其他情况下，我们可能需要使用第三方工具，如*Auth0*或*Okta*。在这个菜谱中，我们将通过实现自定义授权器来展示如何支持这些场景。

# 准备工作

您需要使用在*创建联合身份验证池*菜谱中创建的 Cognito 用户池和在*实现注册、登录和注销*菜谱中创建的示例应用程序来创建本菜谱中使用的身份令牌。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch5/custom-authorizer --path cncb-custom-authorizer
```

1.  使用`cd cncb-custom-authorizer`进入`cncb-custom-authorizer`目录。

1.  检查名为`serverless.yml`的文件，其内容如下：

```js
service: cncb-custom-authorizer

provider:
  name: aws
  # cfnRole: arn:aws:iam::<account-number>:role/${opt:stage}-cfnRole
  ...

functions:
  authorizer:
    handler: handler.authorize
    environment:
      AUD: ${cf:cncb-cognito-pool-${opt:stage}.userPoolClientId}
      ISS: ${cf:cncb-cognito-pool-${opt:stage}.userPoolProviderURL}
      JWKS: ${self:functions.authorizer.environment.ISS}/.well-known/jwks.json
      DEBUG: '*'

  hello:
    handler: handler.hello
    events:
      - http:
          ...
          authorizer: authorizer
```

1.  检查名为`handler.js`的文件，其内容如下：

```js
...
module.exports.authorize = (event, context, cb) => {
  decode(event)
    .then(fetchKey)
    .then(verify)
    .then(generatePolicy)
    ...
};

const decode = ({ authorizationToken, methodArn }) => {
  ...
  return Promise.resolve({
    ...
    token: match[1],
    decoded: jwt.decode(match[1], { complete: true }),
  });
};

const fetchKey = (uow) => {
  ...
  return client.getSigningKeyAsync(kid)
    .then(key => ({
      key: key.publicKey || key.rsaPublicKey,
      ...uow,
    }));
};

const verify = (uow) => {
  ...
  return verifyAsync(token, key, {
    audience: process.env.AUD,
    issuer: process.env.ISS
  })
    ...
};

const generatePolicy = (uow) => {
  ...
  return {
    policy: {
      principalId: claims.sub,
      policyDocument: {
        Statement: [{ Action: 'execute-api:Invoke', Effect, Resource }],
      },
      context: claims,
    },
    ...uow,
  };
};
...
```

1.  使用`npm install`安装依赖项。

1.  使用`npm test -- -s $MY_STAGE`运行测试。

1.  检查`.serverless`目录中生成的内容。

1.  部署栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-custom-authorizer@1.0.0 dp:lcl <path-to-your-workspace>/cncb-custom-authorizer
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
endpoints:
  GET - https://8iznazkhr0.execute-api.us-east-1.amazonaws.com/john/hello
functions:
  authorizer: cncb-custom-authorizer-john-authorizer
  hello: cncb-custom-authorizer-john-hello

Stack Outputs
...
ServiceEndpoint: https://8iznazkhr0.execute-api.us-east-1.amazonaws.com/john
```

1.  在 AWS 控制台中检查栈和资源。

1.  更新`<API-ID>`后，调用以下`curl`命令，尝试在不使用`Authorization`令牌的情况下访问服务：

```js
$ curl https://<API-ID>.execute-api.us-east-1.amazonaws.com/$MY_STAGE/hello | json_pp

{
   "message" : "Unauthorized"
}
```

1.  使用*实现注册、登录和注销*配方中的应用程序进行登录并生成`idToken`，然后使用以下命令导出`idToken`：

```js
$ export CNCB_TOKEN=<idToken value>
```

1.  更新`<API-ID>`后，调用以下`curl`命令，使用`Authorization`令牌成功访问服务：

```js
$ curl -v -H "Authorization: Bearer $CNCB_TOKEN"  https://<API-ID>.execute-api.us-east-1.amazonaws.com/$MY_STAGE/hello | json_pp

{
   "message" : "JavaScript Cloud Native Development Cookbook! Your function executed successfully!",
   "input" : {
      "headers" : {
         "Authorization" : "...",
         ...
      },
      ...
      "requestContext" : {
         ...
         "authorizer" : {
            "exp" : "1528342765",
            "cognito:username" : "john",
            "iss" : "https://cognito-idp.us-east-1.amazonaws.com/us-east-1_tDXH8JAky",
            "sub" : "e4bdd021-a160-4aff-bce2-e652f9469e3e",
            "iat" : "1528339165",
            "email_verified" : "true",
            "auth_time" : "1528339165",
            "email" : "john@example.com",
            "aud" : "4p5p08njgmq9sph130l3du2b7q",
            "event_id" : "fdcdf125-69fb-11e8-a6ef-ab31871bed60",
            "token_use" : "id",
            "principalId" : "e4bdd021-a160-4aff-bce2-e652f9469e3e"
         }
      },
      ...
   }
}
```

1.  完成使用`npm run rm:lcl -- -s $MY_STAGE`后，删除堆栈。

# 它是如何工作的...

将自定义授权器连接到 API 网关的过程与 Cognito 授权器相同。我们可以在与配方相同的项目中实现`authorizer`函数，或者它可以在另一个堆栈中实现，以便可以在服务之间共享。`jwks-rsa`和`jsonwebtoken`开源库实现了大部分逻辑。首先，我们断言令牌的存在并对其进行解码。然后，我们使用解码令牌中存在的键 ID（`kid`）检索发行者的`.well-known/jwks.json`公钥。然后，我们验证令牌的签名与密钥，并断言受众（`aud`）和发行者（`iss`）符合预期。最后，函数返回一个基于路径授予服务访问权限的 IAM 策略。令牌的`claims`也返回在`context`字段中，以便可以将其转发到后端函数。如果任何验证失败，则返回`Unauthorized`错误。

声明对象必须被展平，否则会抛出`AuthorizerConfigurationException`异常。

# 授权基于 GraphQL 的服务

我们已经看到了如何使用*JWT*来授权对服务的访问。除了这种粗粒度的访问控制之外，我们还可以利用 JWT 中的声明来执行细粒度、基于角色的访问控制。在此配方中，我们将展示如何使用指令创建用于在 GraphQL 模式中声明性定义基于角色的权限的注释。

# 准备工作

您需要根据*创建联合身份池*配方中创建的 Cognito 用户池和*实现注册、登录和注销*配方中创建的示例应用程序来创建在此配方中使用的身份令牌。您需要通过 Cognito 控制台将`Author`组分配给您在此配方中使用的用户。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch5/graphql-jwt --path cncb-graphql-jwt
```

1.  使用`cd cncb-graphql-jwt`命令导航到`cncb-graphql-jwt`目录。

1.  检查名为`serverless.yml`的文件，其内容如下：

```js
service: cncb-graphql-jwt

provider:
  name: aws
  runtime: nodejs8.10
  # cfnRole: arn:aws:iam::<account-number>:role/${opt:stage}-cfnRole

functions:
  graphql:
    handler: handler.graphql
    events:
      - http:
          ...
          authorizer:
            arn: ${cf:cncb-cognito-pool-${opt:stage}.userPoolArn}
```

1.  检查以下内容的`index.js`、`schema/thing/typedefs.js`和`directives.js`文件：

```js
index.js
...
const { directiveResolvers } = require('./directives');

const directives = `
  directive @hasRole(roles: [String]) on QUERY | FIELD | MUTATION
`;
...
module.exports = {
  typeDefs: [directives, schemaDefinition, query, mutation, thingTypeDefs],
  resolvers: merge({}, thingResolvers),
  directiveResolvers,
};

schema/thing/typedefs.js

module.exports = `
  ...
  extend type Mutation {
    saveThing(
      input: ThingInput
    ): Thing @hasRole(roles: ["Author"])
    deleteThing(
      id: ID!
    ): Thing @hasRole(roles: ["Manager"])
  }
`;

directives.js

const getGroups = ctx => get(ctx.event, 'requestContext.authorizer.claims.cognito:groups', '');

const directiveResolvers = {
  hasRole: (next, source, { roles }, ctx) => {
    const groups = getGroups(ctx).split(',');
    if (intersection(groups, roles).length > 0) {
      return next();
    }
    throw new UserError('Access Denied');
  },
}

module.exports = { directiveResolvers };
```

1.  使用`npm install`安装依赖项。

1.  使用`npm test -- -s $MY_STAGE`运行测试。

1.  检查`.serverless`目录中生成的内容。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-graphql-jwt@1.0.0 dp:lcl <path-to-your-workspace>/cncb-graphql-jwt
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
endpoints:
  POST - https://b33zxnw20b.execute-api.us-east-1.amazonaws.com/john/graphql
functions:
  graphql: cncb-graphql-jwt-john-graphql

Stack Outputs
...
ServiceEndpoint: https://b33zxnw20b.execute-api.us-east-1.amazonaws.com/john
```

1.  在 AWS 控制台中检查堆栈和资源。

1.  使用*实现注册、登录和注销*菜谱中的应用程序进行登录并生成`idToken`，然后使用以下命令导出`idToken`：

```js
$ export CNCB_TOKEN=<idToken value>
```

1.  在更新`<API-ID>`后，调用以下`curl`命令以使用`Authorization`令牌成功访问服务：

```js
$ curl -v -X POST -H "Authorization: Bearer $CNCB_TOKEN" -H 'Content-Type: application/json' -d '{"query":"mutation { saveThing(input: { id: \"55555555-6666-1111-1111-000000000000\", name: \"thing1\", description: \"This is thing one of two.\" }) { id } }"}' https://b33zxnw20b.execute-api.us-east-1.amazonaws.com/$MY_STAGE/graphql | json_pp

{
   "data" : {
      "saveThing" : {
         "id" : "55555555-6666-1111-1111-000000000000"
      }
   }
}

$ curl -v -X POST -H "Authorization: Bearer $CNCB_TOKEN" -H 'Content-Type: application/json' -d '{"query":"query { thing(id: \"55555555-6666-1111-1111-000000000000\") { id name description }}"}' https://b33zxnw20b.execute-api.us-east-1.amazonaws.com/$MY_STAGE/graphql | json_pp
{
   "data" : {
      "thing" : {
         "name" : "thing1",
         "id" : "55555555-6666-1111-1111-000000000000",
         "description" : "This is thing one of two."
      }
   }
}

$ curl -v -X POST -H "Authorization: Bearer $CNCB_TOKEN" -H 'Content-Type: application/json' -d '{"query":"mutation { deleteThing( id: \"55555555-6666-1111-1111-000000000000\" ) { id } }"}' https://b33zxnw20b.execute-api.us-east-1.amazonaws.com/$MY_STAGE/graphql | json_pp
{
   "data" : {
      "deleteThing" : null
   },
   "errors" : [
      {
         "message" : "Access Denied"
      }
   ]
}
```

1.  使用`npm run rm:lcl -- -s $MY_STAGE`命令完成操作后删除堆栈。

# 它是如何工作的...

服务配置了一个 Cognito `authorizer`，该`authorizer`验证令牌并转发`claims`。这些`claims`包括用户是成员的`groups`。在设计时，我们希望声明性地定义访问特权操作的所需角色。在 GraphQL 中，我们可以使用`directives`来注释`schema`。在这个菜谱中，我们定义了一个`hasRole`指令并实现了一个`resolver`，该`resolver`将注释中定义的允许`roles`与声明中存在的`groups`进行比较，然后允许或拒绝访问。`resolver`逻辑与`schema`和`schema`中的注释解耦，并且注释简单且清晰。

# 实现 JWT 过滤器

我们已经看到了如何使用 JWT 来授权访问服务，以及我们如何使用令牌中的声明在服务内部执行细粒度、基于角色的授权。我们通常还需要在数据实例级别上控制访问。例如，客户只能访问他或她的数据，或者员工只能访问特定部门的资料。为了实现这一点，我们通常根据用户的权限装饰查询。在 RESTful API 中，这些信息通常包含在 URL 中作为路径参数。使用路径参数进行查询是典型的做法。

然而，我们希望使用 JWT 中的声明来执行过滤，因为令牌中的值是由令牌签名的真实性断言的。在这个菜谱中，我们将演示如何使用 JWT 中的声明来创建查询过滤器。

# 准备工作

您需要使用在*创建联合身份池*菜谱中创建的 Cognito 用户池和在*实现注册、登录和注销*菜谱中创建的示例应用程序来创建本菜谱中使用的身份令牌。

# 如何做到...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch5/jwt-filter --path cncb-jwt-filter
```

1.  使用`cd cncb-jwt-filter`命令导航到`cncb-jwt-filter`目录。

1.  查看名为`serverless.yml`的文件，其内容如下：

```js
service: cncb-jwt-filter

provider:
  name: aws
  # cfnRole: arn:aws:iam::<account-number>:role/${opt:stage}-cfnRole
  ...

functions:
  save:
    handler: handler.save
    events:
      - http:
          ...
          authorizer:
            arn: ${cf:cncb-cognito-pool-${opt:stage}.userPoolArn}
  get:
    handler: handler.get
    events:
      - http:
          path: things/{sub}/{id}
          ...
          authorizer:
            arn: ${cf:cncb-cognito-pool-${opt:stage}.userPoolArn}

resources:
  Resources:
    Table:
      ...
        KeySchema:
          - AttributeName: sub
            KeyType: HASH
          - AttributeName: id
            KeyType: RANG
```

1.  查看名为`handler.js`的文件，其内容如下：

```js
module.exports.save = (request, context, callback) => {
  const body = JSON.parse(request.body);
  const sub = request.requestContext.authorizer.claims.sub;
  const id = body.id || uuid.v4();

  const params = {
    TableName: process.env.TABLE_NAME,
    Item: {
      sub,
      id,
      ...body
    }
  };

  ...
  db.put(params, (err, resp) => {
    ...
  });
};

module.exports.get = (request, context, callback) => {
  const sub = request.requestContext.authorizer.claims.sub;
  const id = request.pathParameters.id;

  if (sub !== request.pathParameters.sub) {
    callback(null, { statusCode: 401 });
    return;
  }

  const params = {
    TableName: process.env.TABLE_NAME,
    Key: {
      sub,
      id,
    },
  };

  ...
  db.get(params, (err, resp) => {
    ...
  });
};
```

1.  使用`npm install`命令安装依赖项。

1.  使用`npm test -- -s $MY_STAGE`命令运行测试。

1.  查看在`.serverless`目录中生成的内容。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-jwt-filter@1.0.0 dp:lcl <path-to-your-workspace>/cncb-jwt-filter
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
endpoints:
  POST - https://ho1q4u5hp6.execute-api.us-east-1.amazonaws.com/john/things
  GET - https://ho1q4u5hp6.execute-api.us-east-1.amazonaws.com/john/things/{sub}/{id}
functions:
  save: cncb-jwt-filter-john-save
  get: cncb-jwt-filter-john-get
```

1.  在 AWS 控制台中查看堆栈和资源。

1.  使用*实现注册、登录和注销*菜谱中的应用程序进行登录并生成`idToken`，然后使用以下命令导出`idToken`：

```js
$ export CNCB_TOKEN=<idToken value>
```

1.  更新 `<API-ID>` 后，调用以下 `curl` 命令，以在 `Authorization` 令牌的限制内成功访问数据：

```js
$ curl -v -X POST -H "Authorization: Bearer $CNCB_TOKEN" -H 'Content-Type: application/json' -d '{ "id": "55555555-7777-1111-1111-000000000000", "name": "thing1", "description": "This is thing one of two." }' https://<API-ID>.execute-api.us-east-1.amazonaws.com/$MY_STAGE/things

< HTTP/1.1 201 Created
< location: https://ho1q4u5hp6.execute-api.us-east-1.amazonaws.com/john/things/e4bdd021-a160-4aff-bce2-e652f9469e3e/55555555-7777-1111-1111-000000000000

$ curl -v -H "Authorization: Bearer $CNCB_TOKEN" <Location response header from POST> | json_pp

{
   "description" : "This is thing one of two.",
   "id" : "55555555-7777-1111-1111-000000000000",
   "name" : "thing1",
   "sub" : "e4bdd021-a160-4aff-bce2-e652f9469e3e"
}

$ curl -v -H "Authorization: Bearer $CNCB_TOKEN" https://<API-ID>.execute-api.us-east-1.amazonaws.com/$MY_STAGE/things/<An invalid value>/55555555-7777-1111-1111-000000000000 | json_pp

< HTTP/1.1 401 Unauthorized
```

1.  查看日志：

```js
$ sls logs -f save -r us-east-1 -s $MY_STAGE

$ sls logs -f get -r us-east-1 -s $MY_STAGE
```

1.  使用 `npm run rm:lcl -- -s $MY_STAGE` 完成后，删除堆栈。

# 它是如何工作的...

API 的客户端可以使用任何值来构建 URL，但不能篡改 JWT 令牌的内容，因为发行者已经签发了令牌。因此，我们需要用令牌中的值覆盖任何请求值。在这个菜谱中，我们根据用户的令牌中的主题或子声明保存和检索特定用户的数据。服务配置了一个 `authorizer`，用于验证令牌并转发 `claims`。

为了简化示例，主题被用作 `HASH` 键，数据 `uuid` 作为 `RANGE` 键。当数据被检索时，我们将查询参数与令牌中的值进行比较，如果它们不匹配，则返回 `401` `statusCode`。如果它们匹配，我们使用令牌中的值在实际查询中，以防止断言逻辑中的任何错误意外返回未经授权的数据。

# 使用信封加密

对于大多数系统来说，在静态数据上加密数据至关重要。我们必须确保我们客户的隐私和公司数据的隐私。不幸的是，我们经常打开基于磁盘的加密，然后勾选要求已完成。然而，这仅在磁盘从系统中断开时保护数据。当磁盘连接时，数据在从磁盘读取时自动解密。例如，创建一个启用服务器端加密的 DynamoDB 表，然后创建一些数据并在控制台中查看它。只要你有权限，你将能够以明文形式看到数据。为了真正确保静态数据的隐私，我们必须在应用程序级别加密数据，并有效地删除所有敏感信息。在这个菜谱中，我们使用 AWS **密钥管理服务**（**KMS**）和一种称为 *信封加密* 的技术来保护 DynamoDB 表中的静态数据。

# 如何做到这一点...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch5/envelope-encryption --path cncb-envelope-encryption
```

1.  使用 `cd cncb-envelope-encryption` 命令导航到 `cncb-envelope-encryption` 目录。

1.  查看名为 `serverless.yml` 的文件，其内容如下：

```js
service: cncb-envelope-encryption

provider:
  name: aws
  # cfnRole: arn:aws:iam::<account-number>:role/${opt:stage}-cfnRole
  runtime: nodejs8.10
  endpointType: REGIONAL
  iamRoleStatements:
    ...
  environment:
    TABLE_NAME:
      Ref: Table
    MASTER_KEY_ALIAS:
      Ref: MasterKeyAlias

functions:
  save:
    ...
  get:
    ...

resources:
  Resources:
    Table:
      ...

    MasterKey:
      Type: AWS::KMS::Key
      Properties:
        KeyPolicy:
          Version: '2012-10-17'
          ...

    MasterKeyAlias:
      Type: AWS::KMS::Alias
      Properties:
        AliasName: alias/${self:service}-${opt:stage}
        TargetKeyId:
          Ref: MasterKey
  ...
```

1.  查看名为 `handler.js` 的文件，其内容如下：

```js
const encrypt = (thing) => {
  const params = {
    KeyId: process.env.MASTER_KEY_ALIAS,
    KeySpec: 'AES_256',
  };

  ...
  return kms.generateDataKey(params).promise()
    .then((dataKey) => {
      const encryptedThing = Object.keys(thing).reduce((encryptedThing, key) => {
        if (key !== 'id')
          encryptedThing[key] = 
            CryptoJS.AES.encrypt(thing[key], dataKey.Plaintext);
        return encryptedThing;
      }, {});

      return {
        id: thing.id,
        dataKey: dataKey.CiphertextBlob.toString('base64'),
        ...encryptedThing,
      };
    });
};

const decrypt = (thing) => {
  const params = {
    CiphertextBlob: Buffer.from(thing.dataKey, 'base64'),
  };

  ...
  return kms.decrypt(params).promise()
    .then((dataKey) => {
      const decryptedThing = Object.keys(thing).reduce((decryptedThing, key) => {
        if (key !== 'id' && key !== 'dataKey')
          decryptedThing[key] = 
            CryptoJS.AES.decrypt(thing[key], dataKey.Plaintext);
        return decryptedThing;
      }, {});

      return {
        id: thing.id,
        ...decryptedThing,
      };
    });
};

module.exports.save = (request, context, callback) => {
  const thing = JSON.parse(request.body);
  const id = thing.id || uuid.v4();

  encrypt(thing)
    .then((encryptedThing) => {
      const params = {
        TableName: process.env.TABLE_NAME,
        Item: {
          id,
          ...encryptedThing,
        }
      };

      ...
      return db.put(params).promise();
    })
    ...
};

module.exports.get = (request, context, callback) => {
  const id = request.pathParameters.id;
  ...
  db.get(params).promise()
    .then((resp) => {
      return resp.Item ? decrypt(resp.Item) : null;
    })
    ...
};
```

1.  使用 `npm install` 安装依赖。

1.  使用 `npm test` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-envelope-encryption@1.0.0 dp:lcl <path-to-your-workspace>/cncb-envelope-encryption
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
endpoints:
  POST - https://7wpqdcsoad.execute-api.us-east-1.amazonaws.com/john/things
  GET - https://7wpqdcsoad.execute-api.us-east-1.amazonaws.com/john/things/{id}
functions:
  save: cncb-envelope-encryption-john-save
  get: cncb-envelope-encryption-john-get

Stack Outputs
...
MasterKeyId: c83de811-bda8-4bdc-83e1-32e8491849e5
MasterKeyArn: arn:aws:kms:us-east-1:123456789012:key/c83de811-bda8-4bdc-83e1-32e8491849e5
```

```js
ServiceEndpoint: https://7wpqdcsoad.execute-api.us-east-1.amazonaws.com/john
MasterKeyAlias: alias/cncb-envelope-encryption-john
```

1.  在 AWS 控制台中查看堆栈和资源。

1.  更新 `<API-ID>` 后，调用以下 `curl` 命令以加密和解密数据：

```js
$ curl -v -X POST -d '{ "id": "55555555-8888-1111-1111-000000000000", "name": "thing1", "description": "This is thing one of two." }' https://7wpqdcsoad.execute-api.us-east-1.amazonaws.com/$MY_STAGE/things

< HTTP/1.1 201 Created
< location: https://7wpqdcsoad.execute-api.us-east-1.amazonaws.com/john/things/55555555-8888-1111-1111-000000000000

$ curl -v https://7wpqdcsoad.execute-api.us-east-1.amazonaws.com/$MY_STAGE/things/55555555-8888-1111-1111-000000000000 | json_pp

{
   "name" : "thing1",
   "id" : "55555555-8888-1111-1111-000000000000",
   "description" : "This is thing one of two."
}
```

1.  查看在 DynamoDB 控制台中加密的数据。

1.  查看日志：

```js
$ sls logs -f save -r us-east-1 -s $MY_STAGE

$ sls logs -f get -r us-east-1 -s $MY_STAGE
```

1.  使用 `npm run rm:lcl -- -s $MY_STAGE` 完成后，删除堆栈。

KMS 不包含在 AWS 免费层中。

# 它是如何工作的...

信封加密本质上是一种使用一个密钥加密另一个密钥的实践；敏感数据首先使用数据密钥进行加密，然后数据密钥再使用主密钥进行加密。在这个配方中，`save`函数在将数据保存到 DynamoDB 之前对其进行加密，而`get`函数在从 DynamoDB 检索数据并在将其返回给调用者之前对其进行解密。在`serverless.yml`文件中，我们定义了一个 KMS `MasterKey`和一个`MasterKeyAlias`。别名便于主密钥的轮换。`save`函数调用`kms.generateDataKey`来为对象创建一个数据密钥。每个对象都有自己的数据密钥，每次对象被保存时都会生成一个新的密钥。再次强调，这种做法便于密钥轮换。遵循*设计安全*实践，我们在设计和开发服务时确定哪些字段是敏感的。在这个配方中，我们单独加密所有字段。数据密钥用于使用`AES`加密库在本地加密每个字段。当生成数据密钥时，数据密钥也被主密钥加密并返回到`CyphertextBlob`字段。加密后的数据密钥与数据一起存储，以便`get`函数可以对其进行解密。`get`函数在从数据库检索数据后可以直接访问加密的数据密钥。`get`函数已被授权调用`kms.decrypt`来解密数据密钥。这个环节至关重要。必须根据最小权限原则限制对主密钥的访问。字段使用 AES 加密库在本地解密，以便可以返回给调用者。

# 创建用于传输加密的 SSL 证书

对于包含敏感数据的系统，传输中的数据加密至关重要，这占到了当今大多数系统。完全管理的云服务，如函数即服务、云原生数据库和 API 网关，将数据传输加密作为一项常规操作。这有助于确保我们的数据在运动过程中在整个堆栈中得到保护，而我们几乎不需要付出任何努力。然而，我们最终希望通过自定义域名公开我们的云原生资源。为了做到这一点并支持 SSL，我们必须提供自己的 SSL 证书。这个过程可能很繁琐，我们必须确保在证书到期并导致系统故障之前及时轮换证书。幸运的是，越来越多的云服务提供商正在提供自动轮换的完全管理证书。在这个配方中，我们将使用*AWS 证书管理器*来创建证书并将其与*CloudFront*分发关联。

# 准备工作

您需要一个已注册的域名和一个可以用于此配方创建将要部署的站点的子域的 Route 53 托管区域，并且您需要有权批准证书的创建或有权访问具有该权限的人。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch5/ssl-cert --path cncb-ssl-cert
```

1.  使用`cd cncb-ssl-cert`导航到`cncb-ssl-cert`目录。

1.  查看以下内容的`serverless.yml`文件：

```js
service: cncb-ssl-cert

provider:
  name: aws
  # cfnRole: arn:aws:iam::<account-number>:role/${opt:stage}-cfnRole

plugins:
 - serverless-spa-deploy
 - serverless-spa-config

custom:
  spa:
    ...
  dns:
    hostedZoneId: ZXXXXXXXXXXXXX
    validationDomain: example.com
    domainName: ${self:custom.dns.validationDomain}
    wildcard: '*.${self:custom.dns.domainName}'
    endpoint: ${opt:stage}-${self:service}.${self:custom.dns.domainName}
  cdn:
    acmCertificateArn: 
      Ref: WildcardCertificate

resources:
  Resources:
    WildcardCertificate:
      Type: AWS::CertificateManager::Certificate
      Properties:
        DomainName: ${self:custom.dns.wildcard}
        DomainValidationOptions:
          - DomainName: ${self:custom.dns.wildcard}
            ValidationDomain: ${self:custom.dns.validationDomain}
        SubjectAlternativeNames:
          - ${self:custom.dns.domainName}

  ...
```

1.  使用您的`hostedZoneId`和`validationDomain`更新`serverless.yml`文件。

1.  使用`npm install`安装依赖项。

1.  使用`npm test`运行测试。

1.  查看`.serverless`目录中生成的内容。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-ssl-cert@1.0.0 dp:lcl <path-to-your-workspace>/cncb-ssl-cert
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...

Stack Outputs
...
WebsiteDistributionURL: https://d19i44112h4l3r.cloudfront.net
...
WebsiteURL: https://john-cncb-ssl-cert.example.com
WebsiteDistributionId: EQSJSWLD0F1JI
WildcardCertificateArn: arn:aws:acm:us-east-1:870671212434:certificate/72807b5b-fe37-4d5c-8f92-25ffcccb6f79

Serverless: Path: ./build
Serverless: File: index.html (text/html)
```

要完成此步骤，需要授权人员接收证书批准请求电子邮件并批准证书的创建。CloudFormation 堆栈将暂停，直到证书获得批准。

1.  在 AWS 控制台中查看堆栈和资源。

1.  使用堆栈输出中的`WebsiteURL`调用以下`curl`命令：

```js
$ curl -v -https://john-cncb-ssl-cert.example.com
```

1.  完成操作后，使用`npm run rm:lcl -- -s $MY_STAGE`删除堆栈。

# 它是如何工作的...

在`serverless.yml`文件的资源部分，我们定义了`WildcardCertificate`资源，以指示 AWS 证书管理器创建所需的证书。我们创建了一个通配符证书，以便它可以由许多服务使用。指定正确的`validationDomain`非常重要，因为这将用于在创建证书之前请求批准。您还必须为顶级域名提供`hostedZoneId`。其余一切由`serverless-spa-config`插件处理。该配方部署了一个由`index.html`页面组成的简单网站。该网站的`endpoint`用于在托管区域中创建 Route 53 记录集，并将其分配为网站的 CloudFront 分发的别名。通配符证书与端点匹配，其`acmCertificateArn`分配给分发。最终，我们可以使用`https`协议和自定义域名访问`WebsiteURL`。

# 配置 Web 应用防火墙

**Web 应用防火墙**（**WAF**）是控制云原生系统流量的重要工具。WAF 通过阻止来自常见漏洞，如恶意机器人、SQL 注入、**跨站脚本攻击**（**XSS**）、HTTP 洪水和已知攻击者的流量来保护系统。我们有效地在系统周围创建了一个外围，在流量能够影响系统资源之前，在云边缘阻止流量。我们可以使用本书中的许多技术来监控内部和外部资源，并根据流量变化动态更新防火墙规则。市场上也有可用的托管规则，因此我们可以利用第三方广泛的安全专业知识。在本配方中，我们将通过创建一个阻止来自企业网络外部的流量的规则，并将 WAF 与*CloudFront*分发关联起来，来展示这些组件是如何组合在一起的。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch5/waf --path cncb-waf
```

1.  使用`cd cncb-waf`导航到`cncb-waf`目录。

1.  查看以下内容的`serverless.yml`文件：

```js
service: cncb-waf

provider:
  name: aws
  # cfnRole: arn:aws:iam::<account-number>:role/${opt:stage}-cfnRole

plugins:
 - serverless-spa-deploy
 - serverless-spa-config

custom:
  spa:
    ...
  cdn:
    webACLId: 
      Ref: WebACL
    # logging:
    #   bucketName: ${cf:cncb-account-as-code-${opt:stage}.AuditBucketName}

resources:
  Resources:
    WhitelistIPSet: 
      Type: AWS::WAF::IPSet
      Properties: 
        Name: IPSet for whitelisted IP adresses
        IPSetDescriptors: 
          - Type: IPV4
            Value: 0.0.0.1/32

    WhitelistRule: 
      Type: AWS::WAF::Rule
      Properties: 
        Name: WhitelistRule
        MetricName: WhitelistRule
        Predicates: 
          - DataId: 
              Ref: WhitelistIPSet
            Negated: false
            Type: IPMatch  

    WebACL: 
      Type: AWS::WAF::WebACL
      Properties: 
        Name: Master WebACL
        DefaultAction: 
          Type: BLOCK
        MetricName: MasterWebACL
        Rules: 
          - Action: 
              Type: ALLOW
            Priority: 1
            RuleId: 
              Ref: WhitelistRule        

  Outputs:
    WebACLId:
      Value:
        Ref: WebACL
```

1.  通过登录 VPN 并执行以下命令来确定您的企业外部公共 IP 地址：

```js
$ curl ipecho.net/plain ; echo
```

1.  在`serverless.yml`文件中更新`WhitelistIPSet`，后跟你的 IP 地址和`/32`。

1.  使用`npm install`安装依赖项。

1.  使用`npm test`运行测试。

1.  查看在`.serverless`目录中生成的内容。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-waf@1.0.0 dp:lcl <path-to-your-workspace>/cncb-waf
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...

Stack Outputs
WebsiteDistributionURL: https://d3a9sc88i7431l.cloudfront.net
WebsiteS3URL: http://cncb-waf-john-websitebucket-1c13hrzslok5s.s3-website-us-east-1.amazonaws.com
WebsiteBucketName: cncb-waf-john-websitebucket-1c13hrzslok5s
WebsiteDistributionId: E3OLRBU9LRZBUE
WebACLId: 68af80b5-8eda-43d0-be25-6c65d5cc691e

Serverless: Path: ./build
Serverless: File: index.html (text/html)
```

1.  在 AWS 控制台中查看堆栈和资源。

1.  在连接 VPN 的同时浏览堆栈输出中列出的`WebsiteDistributionURL`，您将能够访问该页面。

1.  现在，断开 VPN 连接，一旦缓存清除，您将无法访问页面。

1.  完成使用`npm run rm:lcl -- -s $MY_STAGE`后，请移除堆栈。

# 它是如何工作的...

首先要注意的是，使用 AWS WAF 的前提是使用 CloudFront。`serverless-spa-config`插件创建 CloudFront 分布并分配`webACLId`。对于这个配方，我们正在保护一个简单的`index.html`页面。我们创建一个静态的`WebACL`来阻止除单个 IP 地址之外的所有内容。我们为单个地址定义`WhitelistIPSet`并将其与`WhitelistRule`关联。然后，我们将规则与访问控制列表（ACL）关联，并将默认操作定义为`BLOCK`所有访问，然后基于`WhitelistRule`定义一个允许访问的操作。

这个示例很有用，但它只是触及了可能性的表面。例如，我们可以取消注释 CloudFront 分布的日志配置，然后根据可疑活动处理访问日志并动态创建规则。在日常生活中，我们还可以检索公共声誉列表并更新规则集。定义和维护有效的规则可能是一项全职工作，因此 AWS 市场上可用的完全管理的规则是自定义规则的有吸引力的替代品或补充。

# 复制数据湖以进行灾难恢复

当我第一次阅读关于 Code Spaces（[`www.infoworld.com/article/2608076/data-center/murder-in-the-amazon-cloud.html`](https://www.infoworld.com/article/2608076/data-center/murder-in-the-amazon-cloud.html)）的故事时，我有点害怕，直到我意识到这家公司之所以倒闭，是为了让我们都能从它的经验中学习。Code Spaces 是一家使用 AWS 的公司，他们的账户被黑客入侵并被勒索。他们进行了反击，他们的整个账户内容，包括备份，都被删除了，他们就这样倒闭了。正确使用多因素认证（MFA）和良好的访问密钥卫生习惯对于防止此类攻击至关重要。同样重要的是，要维护一个完全独立且断开的备份账户，这样单个账户的泄露就不会给整个系统和公司带来灾难。在这个菜谱中，我们将使用 S3 复制功能将存储桶复制到专门的恢复账户。至少，这项技术应该用于复制数据湖的内容，因为数据湖中的事件可以被重放以重建系统。

# 如何操作...

1.  从以下模板创建源和恢复项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch5/dr/recovery-account --path cncb-dr-recovery-account

$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch5/dr/src1-account --path cncb-dr-src1-account
```

1.  使用`cd cncb-dr-recovery-account`导航到`cncb-dr-recovery-account`目录。

1.  检查名为`serverless.yml`的文件，其内容如下：

```js
service: cncb-dr-recovery-account

provider:
  name: aws
  # cfnRole: arn:aws:iam::<account-number>:role/${opt:stage}-cfnRole

custom:
  accounts:
    src1:
      accountNumber: '#{AWS::AccountId}' # using same account to simplify recipe
...
resources:
  Resources:
    DrSrc1Bucket1:
      Type: AWS::S3::Bucket
      DeletionPolicy: Retain
      Properties:
        BucketName: cncb-${opt:stage}-us-west-1-src1-bucket1-dr
        VersioningConfiguration:
          Status: Enabled
    DrSrc1Bucket1Policy: 
      Type: AWS::S3::BucketPolicy
      Properties: 
        Bucket: 
          Ref: DrSrc1Bucket1
        PolicyDocument:
          Statement:
            - Effect: Allow
              Principal:
                AWS: arn:aws:iam::${self:custom.accounts.src1.accountNumber}:root
              Action:
                - s3:ReplicateDelete
                - s3:ReplicateObject
                - s3:ObjectOwnerOverrideToBucketOwner
              Resource:
                ...
  ...
```

1.  使用`npm install`安装依赖项。

1.  使用`npm test`运行测试。

1.  检查`.serverless`目录中生成的内容。

1.  部署栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-dr@1.0.0 dp:lcl <path-to-your-workspace>/cncb-dr
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
Stack Outputs
DrSrc1Bucket1Name: cncb-john-us-west-1-src1-bucket1-dr
```

1.  使用`cd cncb-dr-src1-account`导航到`cncb-dr-src1-account`目录。

1.  检查名为`serverless.yml`的文件，其内容如下：

```js
service: cncb-dr-src1-account

provider:
  name: aws
  # cfnRole: arn:aws:iam::<account-number>:role/${opt:stage}-cfnRole
  ...

custom:
  replicationBucketArn: arn:aws:s3:::cncb-${opt:stage}-us-west-1-src1-bucket1-dr
  recovery:
    accountNumber: '#{AWS::AccountId}' # using same account to simplify recipe
...
resources:
  Resources:
    Src1Bucket1:
      Type: AWS::S3::Bucket
      DeletionPolicy: Retain
      Properties:
        BucketName: cncb-${opt:stage}-us-east-1-src1-bucket1
        VersioningConfiguration:
          Status: Enabled
        ReplicationConfiguration:
          Role: arn:aws:iam::#{AWS::AccountId}:role/${self:service}-${opt:stage}-${opt:region}-replicate
          Rules:
            - Destination:
                Bucket: ${self:custom.replicationBucketArn}
                StorageClass: STANDARD_IA
                Account: ${self:custom.recovery.accountNumber}
                AccessControlTranslation:
                  Owner: Destination
              Status: Enabled
              Prefix: ''

    Src1Bucket1ReplicationRole:
      DependsOn: Src1Bucket1
      Type: AWS::IAM::Role
      Properties:
        RoleName: ${self:service}-${opt:stage}-${opt:region}-replicate
        AssumeRolePolicyDocument:
          Statement:
            - Effect: Allow
              Principal:
                Service:
                  - s3.amazonaws.com
              Action:
                - sts:AssumeRole
        Policies:
          - PolicyName: replicate
            PolicyDocument:
              Statement:
                ...
                - Effect: Allow
                  Action:
                    - s3:ReplicateObject
                    - s3:ReplicateDelete
                    - s3:ObjectOwnerOverrideToBucketOwner
                  Resource: ${self:custom.replicationBucketArn}/*

  ...
```

1.  使用`npm install`安装依赖项。

1.  使用`npm test`运行测试。

1.  检查`.serverless`目录中生成的内容。

1.  部署栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-dr@1.0.0 dp:lcl <path-to-your-workspace>/cncb-dr
> sls deploy -v -r us-east-1 "-s" "john"

Serverless: Packaging service...
...
Serverless: Stack update finished...
...
Stack Outputs
Src1Bucket1Name: cncb-john-us-east-1-src1-bucket1
```

1.  在 AWS 控制台中检查栈和存储桶。

1.  使用以下命令加载数据：

```js
$ sls invoke -r us-east-1 -f load -s $MY_STAGE
```

1.  检查两个存储桶的内容，以确保数据已复制。

1.  完成后，使用`npm run rm:lcl -- -s $MY_STAGE`删除每个栈。

# 它是如何工作的...

AWS S3 负责将一个存储桶的内容复制到另一个存储桶的所有繁重工作。我们只需定义要复制的内容和位置，并正确设置所有权限即可。将数据复制到完全与任何其他账户断开连接的单独账户中非常重要，以确保另一个账户的漏洞不会影响恢复账户。为了简化这个流程，我们将使用单个账户，但这并不会改变除了账户编号值之外的 CloudFormation 模板。将数据复制到您通常不使用的区域也是一个好主意，这样您某个区域的故障就不会导致您的恢复区域也出现故障。接下来，我们需要为每个要复制的源存储桶在恢复账户中创建相应的存储桶。使用前缀和/或后缀的命名约定来轻松区分源存储桶和目标存储桶是一个好主意。所有存储桶都必须开启版本控制以支持复制。源存储桶授予 S3 服务执行复制的权限，而目标存储桶授予源账户向恢复账户写入的权限。我们还把复制的文件内容的所有权转让给恢复账户。最终，如果源存储桶中的内容被完全删除，目标存储桶仍然会包含这些内容以及删除标记。
