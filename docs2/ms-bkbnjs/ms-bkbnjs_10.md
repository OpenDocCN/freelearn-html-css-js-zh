# 第十章 认证

大多数网络应用程序都使用某种授权和认证子系统，以允许其用户访问应用程序的私有信息。然而，如果你对如何实现它没有清晰的想法，认证过程可能会变得复杂，因为 Backbone 并没有提供如何实现的提示。

Backbone 是认证无关的，这意味着它不提供用于实现认证策略的对象或工具。优点是 Backbone 不与任何认证机制耦合，缺点是你应该关注它。

由于 Backbone 是基于 REST API 设计的，因此你将不得不处理那种 API 中常见的认证机制。这也是 Backbone 不强制或提供用于认证用户的工具的一个很好的原因。

另一个需要注意的事情是，REST API 应该是无状态的，这意味着它们不会跟踪你之前发出的请求。对你来说，这意味着如果你发出登录请求，你将期望服务器在后续请求中识别你；然而，在无状态服务器上，它不会记住你。

如果你之前没有与 REST 网络服务合作过，这可能会听起来很疯狂；然而，你必须每次向服务器发送请求时都进行认证。这是必要的，并且有许多可行的方法可以实现；你应该查阅 API 文档，以了解认证算法的确切细节。

尽管有众多可用的选项，它们彼此之间非常相似，只是在一些细节上有所变化；然而，本质上，它们的工作方式非常相似。因此，不必担心可供使用的不同认证方式的数量；学习基础知识，然后修改细节即可。

# 无状态 API 认证

对无状态 API 进行认证意味着你应该在每次向服务器发送请求时进行认证；记住，无状态服务器不会跟踪之前的请求。这意味着每次你向服务器发送请求时，它都会将请求处理为第一个请求。

由于会话信息不会存储在服务器上，你应该将其放在其他地方。对于 Backbone 应用程序来说，存储会话数据的正确位置是浏览器，你可以使用 `localStorage` 来存储和检索会话数据，并使用 JavaScript 来管理会话。

## HTTP 基本认证

对 RESTful API 进行认证的最简单方式是使用 HTTP 基本认证。其背后的思想很简单；你应该在每次发送请求时包含你用户名和密码的编码版本。这可能听起来很危险，因为每次请求都发送用户名和密码，确实如此。因此，强烈建议只在启用了 HTTPS 连接的地方使用基本认证：

![HTTP 基本认证](img/B01962_10_01.jpg)

图 10.1 基本认证方案

用户名和密码应在请求的`Authentication`头下发送。考虑以下场景：

+   用户：myuser

+   密码：123456

为了编码`Authentication`头，用户名和密码应使用冒号`:`作为分隔符连接。

```js
myuser:123456
```

然后，该字符串应按如下方式编码为`base64`：

```js
$ echo myuser:123456 | base64
bXl1c2VyOjEyMzQ1Ngo=

```

生成的字符串应用于对服务器发出的每个请求：

```js
GET /api/contacts
Authorization: Basic bXl1c2VyOjEyMzQ1Ngo=
```

服务器将为每个请求解码并验证你的身份。请记住，你应该不要在没有 HTTPS 的情况下使用此机制。有人很容易拦截请求头并解码字符串，以发现你的用户名和密码。

## OAuth2 认证

OAuth2 协议是为了在服务之间共享资源而设计的，不使用用户名和密码。你可能已经使用过可以使用社交网络账户进行认证的应用程序。这就是 OAuth2 的实际应用。OAuth2 认证是 RFC 6749 中描述的授权框架，如下所示：

![OAuth2 认证](img/B01962_10_02.jpg)

图 10.2 OAuth2 抽象流程

在前面的图中，你可以看到使用 OAuth2 算法进行认证的抽象图。你可以识别以下实体，如下所示：

+   **资源所有者**是拥有受保护数据的实体。这通常是个人。

+   **Web 应用程序**是想要访问资源所有者私有数据的应用程序。

+   **授权服务器**用于识别和验证资源服务器（受保护数据所在的服务器）的用户。

+   **访问令牌**是在资源服务器中用于授权资源访问的数据。访问令牌通常有一个过期时间。

+   **资源服务器**是提供受保护数据的主机。

注意，资源服务器和授权服务器可以是同一主机。认证过程如下：

1.  应用程序请求资源所有者的授权。

1.  资源所有者授权并颁发授权密钥。

1.  应用程序使用授权密钥来交换访问令牌。

1.  授权服务器验证授权密钥和应用程序。

1.  授权服务器颁发访问令牌并将其返回给应用程序。

1.  应用程序可以使用访问令牌来访问受保护资源。

颁发的访问令牌通常会有过期时间，以便防止攻击者恶意使用。当令牌过期时，应用程序应重复认证过程。

然而，每次令牌过期时都进行登录并不实际。为了防止这种情况，授权服务器颁发另一个名为`刷新令牌`的令牌，当当前访问令牌过期时，可以使用它来颁发新的访问令牌。

### 服务应用程序

当你想访问像 Facebook、Twitter、Google 等服务的私有数据时，你必须首先将该服务中的应用程序注册到该服务上。当你将该服务中的应用程序注册到服务上时，他们将会要求你提供应用程序名称、描述、网站等信息。

当应用程序注册完成后，服务将为你提供一些令牌来识别你的应用程序，这些令牌包括以下两个关键数据：

+   **ClientID**：这个唯一标识符用于在服务中识别你的应用程序

+   **ClientSecret**：这用于验证使用给定 ClientID 发出的请求是否合法

如果你为应用程序构建的 REST 服务器只由你访问，你可以手动生成一个`ClientID`和`ClientSecret`作为应用程序中的常量值。

如果你的 REST 服务器将向任何想要玩弄应用程序数据的人公开公共 API，你应该开发某种应用程序注册（例如用户注册）以便允许其他人注册他们的应用程序。

## OAuth2 授权类型

在上一节中，你已经看到了 OAuth2 协议作为认证的抽象模式。RFC 6749 文档规范描述了四种不同的方式来获取访问令牌。

### 授权代码授权

授权代码授权是最完整的授权流程；其主要用途是从另一个服务器访问用户的私有资源：

![授权代码授权](img/B01962_10_03.jpg)

图 10.3 授权代码授权

参考前面的图示。服务器应用程序是一个应用程序服务器（例如 Node.js、Python 等），API 服务器是一个第三方服务器，其中包含私有资源（例如 Facebook、Google 等）。

在授权代码授权场景中，服务器应用程序想要代表**用户**从 API 服务器获取数据。这是通过服务器应用程序完成的；当用户与 Backbone 应用程序交互时，它会向服务器应用程序发出请求，然后服务器应用程序可以从 API 服务器获取数据，进行一些处理，并将响应返回给 Backbone 应用程序。

Backbone 应用程序永远不会与 API 服务器建立单个连接，这是服务器应用程序的责任，以确保 Backbone 应用程序只能看到单个服务器应用程序。

### 隐式授权

这是对授权代码授权的简化；隐式授权的使用是针对没有服务器或移动应用程序的纯前端应用程序：

![隐式授权](img/B01962_10_04.jpg)

图 10.4 隐式授权

在隐式授权中，App 服务器不存在，因此，Backbone 应用程序应直接与 API 服务器通信。尽管隐式授权很简单，但你应该注意其安全问题。

为了最小化这种风险，你的应用程序应该使用 HTTPS 进行安全保护，并且如果你没有启用此流类型，不要使用这种流类型。另一个相关问题是，此授权类型不会颁发刷新令牌，这意味着当访问令牌过期时，你应该重新登录。

### 资源所有者密码凭证授权

当 Backbone App 和 API 服务器是同一应用程序时，此授权类型非常有用。换句话说，前端应用程序和后端服务器是由您开发的，这意味着您不是在访问第三方资源。

由于您的应用程序拥有所有资源，您将需要应用程序的用户名和密码来对其进行身份验证：

![资源所有者密码凭证授权](img/B01962_10_05.jpg)

图 10.5 资源所有者密码

上述图表与隐式授权图表非常相似；然而，在这种情况下，您不需要使用 ClientID 和 ClientSecret 令牌，这简化了身份验证过程。

当您使用此授权类型时，感觉就像传统的身份验证方式；您应该将您的用户名和密码发送到服务器，然后服务器会告诉您凭证是否有效。如果有效，您将收到一个有效的访问令牌，您可以存储并按需使用。

### 客户端凭证授权

当您有一个信任的客户端访问服务器资源时，会使用客户端凭证授权。例如，一个商业伙伴。在这种授权类型中，您不是验证用户，而是验证应用程序，因此不需要用户名或密码。

在这个授权中，如果您信任客户端，应该使用 ClientID 和 ClientSecret，将颁发访问令牌。

![客户端凭证授权](img/B01962_10_06.jpg)

图 10.6 客户端凭证

### 摘要

在前面的章节中，您已经看到了如何使用 OAuth2 框架对 REST 服务器进行身份验证；在 OAuth2 中，规范以四种方式描述，使用任何一种取决于应用程序的需求。

然而，所有这些授权类型的目标都是获取一个可以用于下一个服务器请求的访问令牌。一旦您有了访问令牌，与 API 服务器的交互应该对 Backbone App 来说是透明的，令牌应该发送，而无需应用程序其他部分的了解。

# 实现 HTTP 基本身份验证

让我们在`Contacts App`中实现基本认证协议。正如您在前面的章节中学到的，您需要为向服务器发出的每个请求添加`Authorization`头，以便进行身份验证。从服务器端，您需要读取和解析此头。

已经开发了一个有用的`npm`包来解码`Authorization`头。使用`basic-auth`模块，您可以读取请求头并返回一个包含两个字段的对象：`name`和`pass`，这些字段可以用来验证用户。为了简单起见，我们将使用硬编码的用户名和密码，而不是真实的数据库：

```js
// server/basicAuthMiddleware.js
var basicAuth = require('basic-auth');

var authorizationRequired = function (req, res, next) {
  var credentials = basicAuth(req) || {};

  if (credentials.name === 'john' && credentials.pass === 'doe') {
    return next();
  } else {
    return res.sendStatus(401);
  }
};

module.exports = authorizationRequired;
```

中间件会检查用户名是否为`john`且密码是否为`doe`。如果不是，将向客户端发送 HTTP `401`错误。您可以为想要保护的每个资源使用中间件：

```js
var controller = require('./controller');
var authorizationRequired = require('./basicAuthMiddleware');

module.exports = routes = function(server) {
  server.post('/api/contacts',
authorizationRequired, controller.createContact);
  server.get('/api/contacts',
authorizationRequired, controller.showContacts);
  server.get('/api/contacts/:contactId',
authorizationRequired, controller.findContactById);
  server.put('/api/contacts/:contactId',
authorizationRequired, controller.updateContact);
  server.delete('/api/contacts/:contactId',
authorizationRequired, controller.deleteContact);
  server.post('/api/contacts/:contactId/avatar',
authorizationRequired, controller.uploadAvatar);
};
```

我们在 HTTP 401 响应中包含的 `WWW-Authenticate` 头部将确保浏览器弹出一个对话框要求你输入用户名和密码。你可以在对话框中使用 `john` 用户和 `doe` 密码，然后浏览器将为你构建并发送认证头：

![实现 HTTP 基本认证](img/B01962_10_07.jpg)

图 10.7 基本认证登录

为了更好地控制如何请求认证，你可以创建一个 `form` 视图并为认证目的添加一些路由：

```js
<div class="col-xs-12 col-sm-offset-4 col-sm-4">
<div class="panel">
<div class="panel-body">
<h4>
Login required
</h4>
<p>
Use 'john' as user and 'doe' as password.
</p>
<form>
<div class="form-group">
<label for="username">User</label>
<input type="user" class="form-control" id="username" placeholder="Username">
</div>
<div class="form-group">
<label for="password">Password</label>
<input type="password" class="form-control" id="password" placeholder="Password">
</div>
<p id="message" class="pull-left"></p>
<button type="submit" class="btn btn-primary pull-right">Login</button>
</form>
</div>
</div>
</div>
```

`LoginView` 方法应该在用户点击 **登录** 按钮时处理认证过程：

```js
// apps/login/views/loginView.js
'use strict';

var Common = require('../../../common');
var template = require('../templates/login.tpl');

class LoginView extends Common.ModelView {
  constructor(options) {
    super(options);
    this.template = template;
  }

  get className() {
    return 'row';
  }

  get events() {
    return {
      'click button': 'makeLogin'
    };
  }

  makeLogin(event) {
    event.preventDefault();

    var username = this.$el.find('#username').val();
    var password = this.$el.find('#password').val();

console.log('Will login the user', username,
                'with password', password);
  }
}

module.exports = LoginView; 
```

应该添加一个新的路由来显示 `#/login` 表单：

```js
// apps/login/router.js
'use strict';

var Backbone = require('backbone');
var LoginView = require('./views/loginView');

class LoginRouter extends Backbone.Router {
  constructor(options) {
    super(options);

    this.routes = {
      'login': 'showLogin'
    };

    this._bindRoutes();
  }

  showLogin() {
    var App = require('../../app');
    var login = new LoginView();

    App.mainRegion.show(login);
  }
}

module.exports = new LoginRouter();
```

当应用程序启动时，你需要包含这个新的路由，如下所示：

```js
// app.js
// ...

// Initialize all available routes
require('./apps/contacts/router');
require('./apps/login/router');

// ...
```

当未认证的用户访问 `#/contacts` 路由时，Backbone 应用程序应该将他们重定向到登录表单：

```js
Backbone.$.ajaxSetup({
  statusCode: {
    401: () =>{
      window.location.replace('/#login');

    }
  }
});
```

当服务器响应 HTTP 401 时，意味着用户未认证，此时你可以显示登录窗口。请记住，为了防止浏览器显示其登录对话框，需要移除 `WWW-Authenticate` 响应头：

```js
function unauthorized(res) {
  // res.set('WWW-Authenticate', 'Basic realm=Authorization Required');
  return res.sendStatus(401);
};
```

![实现 HTTP 基本认证](img/B01962_10_08.jpg)

图 10.8 登录表单

现在我们已经有了登录表单，我们可以将其中的认证代码放入其中。这将分为以下三个步骤进行：

1.  构建认证字符串。

1.  测试认证字符串是否有效。

1.  为未来的请求保存认证字符串。

认证字符串很容易构建，你可以使用 `btoa()` 函数将字符串转换为 `base64`，如下所示：

```js
class LoginView extends Common.ModelView {
  // ...

  makeLogin(event) {
    event.preventDefault();

    var username = this.$el.find('#username').val();
    var password = this.$el.find('#password').val();
    var authString = this.buildAuthString(
      username, password
    );

    console.log('Will use', authString);
  }

  buildAuthString(username, password) {
    return btoa(username + ':' + password);
  }
}
```

然后，你可以使用 `authString` 来测试是否可以成功获取联系人资源。如果服务器成功响应，则说明用户正在使用正确的凭据：

```js
class LoginView extends Common.ModelView {
  // ...

  makeLogin(event) {
    event.preventDefault();

    var username = this.$el.find('#username').val();
    var password = this.$el.find('#password').val();
    var authString = this.buildAuthString(
      username, password
    );

    Backbone.$.ajax({
      url: '/api/contacts',
      headers: {
        Authorization: 'Basic ' + authString
      },
      success: () => {
        var App = require('../../../app');
        App.router.navigate('contacts', true);
      },
      error: jqxhr => {
        if (jqxhr.status === 401) {
          this.showError('User/Password are not valid');
        } else {
          this.showError('Oops... Unknown error happens');
        }
      }
    });
  }

  buildAuthString(username, password) {
    return btoa(username + ':' + password);
  }

  showError(message) {
    this.$('#message').html(message);
  }
}
```

如果认证字符串有效，则用户将被重定向到联系人列表；然而，重定向可能不会按预期工作，因为联系人列表中没有发送 `Authorization` 头部。请记住，你应该为每个请求发送 `Authorization` 头部。

你需要将 `Authentication` 字符串保存在 `sessionStorage` 中，以便在未来的请求中使用。`sessionStorage` 与 `localStorage` 类似；然而，在 `sessionStorage` 中，数据将在浏览器关闭时被移除：

```js
class LoginView extends Common.ModelView {
  // ...

  makeLogin(event) {
// ...

    Backbone.$.ajax({
      url: '/api/contacts',
      headers: {
        Authorization: 'Basic ' + authString
      },
      success: () => {
        var App = require('../../../app');
        App.saveAuth('Basic', authSting);
        App.router.navigate('contacts', true);
      },
      error: jqxhr => {
        if (jqxhr.status === 401) {
          this.showError('User/Password are not valid');
        } else {
          this.showError('Oops... Unknown error happens');
        }
      }
    });
  }

// ...
}
```

`App` 对象将负责存储令牌：

```js
// app.js
var App = {
  // ...

  // Save an authentication token
  saveAuth(type, token) {
    var authConfig = type + ':' + token;

    sessionStorage.setItem('auth', authConfig);
    this.setAuth(type, token);
  },

  // ...
}
```

在将令牌保存在 `sessionStorage` 中后，你应该为每个未来的请求包含 `Authorization` 头部：

```js
// app.js
var App = {
  // ...

  // Set an authorization token
  setAuth(type, token) {
    var authString = type + ' ' + token;
    this.setupAjax(authString);
  },

  // Set Authorization header for authentication
  setupAjax(authString) {
    var headers = {};

    if (authString) {
      headers = {
        Authorization: authString
      };
    }

    Backbone.$.ajaxSetup({
      statusCode: {
        401: () => {
          App.router.navigate('login', true);
        }
      },
      headers: headers
    });
  }

  // ...
}
```

当应用程序启动时，它应该检查是否有活跃的会话打开；如果有，则应该使用该会话，如下所示：

```js
// app.js
var App = {
start() {
    // The common place where sub-applications will be showed
    App.mainRegion = new Region({el: '#main'});

    this.initializePlugins();

    // Load authentication data
    this.initializeAuth();

    // Create a global router to enable sub-applications
    // to redirect to
    // other URLs
    App.router = new DefaultRouter();
    Backbone.history.start();
  },

  // ...

  // Load authorization data from sessionStorage
  initializeAuth() {
    var authConfig = sessionStorage.getItem('auth');

    if (!authConfig) {
      return window.location.replace('/#login');
    }

    var splittedAuth = authConfig.split(':');
    var type = splittedAuth[0];
    var token = splittedAuth[1];

    this.setAuth(type, token);
  },

  // ...
}
```

用户应该能够注销。让我们在 App 路由器中添加一个用户注销的路由：

```js
// app.js

// General routes non sub-application dependant
class DefaultRouter extends Backbone.Router {
  constructor(options) {
    super(options);
    this.routes = {
      '': 'defaultRoute',
      'logout': 'logout'
    };
    this._bindRoutes();
  }

  // Redirect to contacts app by default
  defaultRoute() {
    this.navigate('contacts', true);
  }

  // Drop session data
  logout() {
    App.dropAuth();
    this.navigate('login', true);
  }
}
```

当从 `sessionStorage` 中移除 `auth` 字符串并且不再发送认证头时，会话将被移除：

```js
var App = {
  // ...

  // Remove authorization token
  dropAuth() {
    sessionStorage.removeItem('auth');
    this.setupAjax(null);
  },

  // …
}
```

这就是您可以使用 HTTP 基本认证协议实现授权的方法。为每个发送到服务器的请求生成并附加一个授权字符串，这是通过 jQuery 的 `ajaxSetup()` 方法完成的。在下一节中，我们将看到如何实现 OAuth2 协议。

# 实现 OAuth 认证

正如我们在基本认证中所做的那样，我们将构建 OAuth2 协议的服务端实现。由于 Backbone App 和 Server App 都是由我们构建的，因此最佳授权类型选择是 *资源所有者密码凭证授权*。

与基本认证不同，OAuth2 需要添加一个用于颁发访问和刷新令牌的端点。如 RFC-6749 所述，对此端点发出的请求应包括以下内容：

> *客户端通过使用 "application/x-www-form-urlencoded" 添加以下参数向令牌端点发起请求：*
> 
> *grant_type：必需。值必须设置为 "password"。*
> 
> *用户名：必需。资源所有者的用户名。*
> 
> *密码：必需。资源所有者的密码。*

一个有效的请求将如下所示：

```js
POST /api/oauth/token HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

grant_type=password&username=john&password=doe
```

然后，服务器将响应一个有效的访问令牌、可选的刷新令牌和令牌类型；它可能包含其他值，如下所示：

```js
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
    "access_token":"2YotnFZFEjr1zCsicMWpAA",
    "token_type":"example",
    "expires_in":3600,
    "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
    "example_parameter":"example_value"
}
```

`token_type` 值告诉客户端所颁发令牌的类型，在我们的情况下，它是 `Bearer`。我们可以通过创建必要的函数来开始实现颁发授权令牌：

```js
function authorize(data, callback) {
  var grantType = data.grant_type;
  var username = data.username;
  var password = data.password;

  if (grantType !== 'password') {
    return callback({error: 'invalid_grant'});
  }

  if (!username || !password) {
    return callback({error: 'invalid_request'});
  }

  if (username === 'john' && password === 'doe') {
    issueAuthorization(username, callback);
  } else {
    callback({error: 'invalid_grant'});
  }
}
```

如 RFC 文档中指定，如果未支持授权类型，则我们应该响应一个 `invalid_grant` 错误；如果请求中缺少参数，则我们应该响应一个 `invalid_request` 错误。

如果用户名和密码匹配，则我们可以颁发一个授权令牌：

```js
const DEFAULT_EXPIRATION_TIME = 3600; // seconds (1 hour)

// ...

function issueAuthorization(username, callback) {
  var accessToken = generateToken();
  var refreshToken = generateToken();
  var token = {
    access_token: accessToken,
    token_type: 'Bearer',
    expires_in: DEFAULT_EXPIRATION_TIME,
    refresh_token: refreshToken
  };

  saveValidToken(token, username);
  callback(token);
}
```

生成的令牌只是使用 `generateToken()` 函数生成的随机字符串，如下所示：

```js
const TOKEN_LENGTH = 20;

// ...

function generateToken() {
return crispy.base32String(TOKEN_LENGTH);
}
```

这些令牌应该存储在某个地方，以便在未来的请求中进行验证。为了简单起见，在这本书中，我们将令牌存储在内存对象中；然而，您可以使用如 Redis 这样的数据库进行实际项目：

```js
var validTokens = {};
var refreshTokens = {};

// ...

function saveValidToken(token, username) {
  var tokenCopy = _.clone(token);
  tokenCopy.username = username;

  validTokens[token.access_token] = tokenCopy;
  refreshTokens[token.refresh_token] = tokenCopy;

  setTimeout(function() {
    expireToken(tokenCopy.access_token);
  }, DEFAULT_EXPIRATION_TIME * 1000);
}

function expireToken(token) {
  delete validTokens[token];
}
```

`validTokens` 和 `refreshTokens` 是存储令牌的哈希表。在 **TTL** （**生存时间**）到期后，`validTokens` 中的令牌应该被移除，`setTimeout()` 调用将确保这些项自动移除。

要验证用户是否已认证，我们只需检查令牌是否在 `validTokens` 哈希表中有效，如下所示：

```js
function authenticate(token, callback) {
  if (_.has(validTokens, token)) {
    callback({valid: true, token: validTokens[token]});
  } else {
    callback({valid: false, token: null});
  }
}
```

使用本节中描述的函数，我们可以在我们的 Contacts App 项目中实现 OAuth2。让我们添加一个路由来生成访问令牌，并添加一个中间件来保护资源，如下所示：

```js
var controller = require('./controller');
var auth = require('./oauth2Middleware);

module.exports = routes = function(server) {
  server.post('/api/oauth/token', auth.authenticate);
  server.post('/api/contacts', auth.requireAuthorization,
    controller.createContact);
  server.get('/api/contacts', auth.requireAuthorization,
    controller.showContacts);
  server.get('/api/contacts/:contactId',
    auth.requireAuthorization, controller.findContactById);
  server.put('/api/contacts/:contactId',
    auth.requireAuthorization, controller.updateContact);
  server.delete('/api/contacts/:contactId',
    auth.requireAuthorization, controller.deleteContact);
  server.post('/api/contacts/:contactId/avatar',
    auth.requireAuthorization, controller.uploadAvatar);
};
```

`oauth2Middleware` 模块提供了 `requireAuthorization()` 中间件和 `authenticate()` 认证处理器，如下所述：

```js
module.exports = {
  authenticate(req, res) {
    authorize(req.body || {}, _.bind(res.json, res));
  }
}
```

为了颁发新的令牌，你需要调用 `authorize()` 函数，该函数返回一个符合 RFC 文档中指定的有效 OAuth2 响应：

```js
requireAuthorization(req, res, next) {
  var authorization = req.headers.authorization || '';

  if (!authorization) {
    return res.sendStatus(401);
  }

  var splitValues = authorization.split(' ');
  var tokenType = splitValues[0];
  var token = splitValues[1];

  if (!tokenType || tokenType !== 'Bearer' || !token) {
    return res.sendStatus(401);
  }

  authenticate(token, function(response) {
    if (response.valid) {
      next();
    } else {
      return res.sendStatus(401);
    }
  });
}
```

`requireAuthorization()` 中间件用于保护我们 OAuth2 协议实现中的资源。该中间件将令牌分为两部分：令牌类型和令牌本身；它验证令牌类型及其在活动访问令牌列表中的存在是否有效。

在 Backbone 应用程序中，我们可以重用为基本认证协议创建的对象；然而，我们需要进行一些小的修改。在 `LoginView` 对象中，你应该将 `url` 请求更改为 `/api/oauth/token` 并将方法更改为 `POST`，如下所示：

```js
class LoginView extends Common.ModelView {
  // ...

  makeLogin(event) {
    event.preventDefault();

    var username = this.$el.find('#username').val();
    var password = this.$el.find('#password').val();

    Backbone.$.ajax({
      method: 'POST',
      url: '/api/oauth/token',
      data: {
        grant_type: 'password',
        username: username,
        password: password
      },
      success: response => {
        var App = require('../../../app');
        var accessToken = response.access_token;
        var tokenType = response.token_type;

        App.saveAuth(tokenType, accessToken);
        App.router.navigate('contacts', true);
      },
      error: jqxhr => {
        if (jqxhr.status === 401) {
          this.showError('User/Password are not valid');
        } else {
          this.showError('Oops... Unknown error happens');
        }
      }
    });
  }

  buildAuthenticationString(token) {
    return 'Bearer ' + token;
  }

  showError(message) {
    this.$('#message').html(message);
  }
}
```

# 摘要

如果你没有清晰的关于 REST 服务器中认证工作原理的视野，Backbone 应用程序中的认证可能会变得复杂。由于 Backbone 对认证是中立的，它不会强迫你使用某种认证机制。作为开发者，创建一个或遵循现有的认证机制是你的责任。

在无状态服务器的支撑下，Backbone 应用程序中，你应该将会话处理代码移动到浏览器中。在本章展示的示例中，我们使用了 `sessionStorage` 来存储访问令牌；然而，你也可以使用其他存储解决方案，例如 `localStorage` 和 `indexeddb`，甚至 cookies。

然后，我们看到了如何在 Contacts 应用程序中将基本认证和 OAuth2 协议的理论与实践实现相结合。该实现对于应用程序的其他部分是透明的，因此，你可以轻松地在实现之间切换。
