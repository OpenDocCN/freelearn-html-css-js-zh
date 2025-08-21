# 第三章：应用考虑

现在是时候处理真实世界的应用程序了！正如之前提到的，Node.js 平台的杀手功能之一是丰富的模块和快速发展的社区。审计您使用的每个模块以确保安全仍然很重要，但使用模块很可能会成为工作流程中不可或缺的一部分。

由于其巨大的流行度，我将专门编写我的代码示例以针对**Express**应用程序。这应该涵盖今天大多数 Node.js 应用程序，但我们将涵盖的概念适用于任何平台。

# Express 简介

Express 是一个专注于保持小巧但强大的 Node.js 最小化 Web 开发框架。它是建立在另一个称为**Connect**的框架之上的，这是一个用于编写带有小插件（称为中间件）的 HTTP 服务器的平台。

Connect 和 Express 的架构允许您仅使用您需要的内容，而不是其他。这非常好地融入了安全讨论中，因为您不会整合大量不使用的功能，这为可能未经检查的安全漏洞敞开了大门。

Connect 捆绑了 20 多个常用的中间件，增加了日志记录、会话、cookie 解析、请求体解析等功能。在定义 Connect 或 Express 应用程序时，只需按照以下代码添加要使用的中间件：

```js
var connect = require("connect"),
    app = connect(); // create the application

// add a favicon for browsers
app.use(connect.favicon());

// require a simple username/password to access
app.use(connect.basicAuth("username", "password"));

// this middleware simply responds with "Hello World" to every request
// that isn't responded to by previous middleware (i.e. favicon)
app.use(function (req, res) {
    res.end("Hello World");
});

// app is a thin wrapper around Node's http.Server
// so many of the same methods are available
app.listen(3000);

console.log("Server available at http://localhost:3000/");
```

在这里，我们正在创建一个具有三个中间件的应用程序：`favicon`，`basicAuth`和我们自己的自定义中间件。前两个由 Connect 提供，它们都可以进行配置以指定其确切的行为。

### 提示

中间件总是按照附加的顺序执行，这是在确定何时以及何时附加时要记住的事情。

Connect 使用传递风格，这意味着每个中间件函数都被赋予控制权，并且在完成后必须将控制传递给继续中的下一个中间件。在我们这里的应用程序方面，每个中间件都被赋予请求和响应对象，并且对请求的生命周期具有完全控制权。

由于它们按顺序执行，让我们来看看这个应用程序的请求/响应循环是如何运作的。由于中间件具有完全控制权，它可以采取以下三种主要行动之一：

+   直接响应请求，结束继续

+   修改请求或响应对象以供继续中间件使用

+   不做任何操作，只需启动下一层中间件

幸运的是，我们在这里有所有三个的例子！首先，当一个应用程序进入这个服务器时，它会通过`favicon`中间件运行。它检查**统一资源标识符**（**URI**），如果匹配`/favicon.ico`，它会为浏览器响应一个`favicon`图标。如果 URI 不匹配，它就会简单地传递给下一个中间件。

接下来，如果请求继续，就是`basicAuth`中间件。这会提示用户使用**HTTP 基本身份验证**提供用户名和密码组合。如果用户未提供正确的凭据，服务器将以**401（未经授权）**响应并结束请求。如果用户成功提供了正确的用户名和密码，请求对象将被修改以包含用户信息，然后启动下一个中间件。

最后是我们的自定义中间件，这可能是您可能拥有的最简单的中间件。它的作用只是将**Hello World**作为响应主体发送。这意味着无论我们请求什么 URI（当然除了`/favicon.ico`），只要我们提供正确的凭据，我们就会看到**Hello World**。

现在您已经对中间件的工作原理有了基本的了解，让我们继续学习 Express 以及它对 Connect 的增强。Express 通过 Connect 系统添加了路由、HTTP 助手、视图系统、内容协商和其他功能。事实上，Express 应用程序看起来与 Connect 应用程序非常相似，如下面的代码所示：

```js
var express = require('express'),
    app = express();

app.use(express.favicon());
app.use(express.basicAuth("username", "password"));

app.get("/", function (req, res) {
    res.send('Hello World');
});

app.listen(3000);
```

Express 自动在其自己的命名空间中包含 Connect 中间件，因此您可以在不需要显式要求 Connect 的情况下使用它们。此外，它还添加了一些自己的强大功能，特别是我们在这里使用的路由功能。

Express 受到了**Ruby**的**Sinatra** Web 框架的启发。每个 HTTP 动词（`GET`，`POST`等）在应用对象上都有一个相应的函数。在这里，我们说 URL`/`的 HTTP `GET`请求将发送**Hello World**。任何其他 URL 都将得到**404（未找到）**错误，除了`/favicon.ico`，它由 favicon 中间件处理。

Express 是一种极简主义的方法，可以按照您的意愿开发应用程序。它不会将您锁定在 MVC 框架或特定的视图引擎中，并允许您包含任何 npm 模块来为您的应用程序提供动力。

# 身份验证

身份验证是确定用户在尝试通过您的应用程序执行某些操作时是否是他们声称的用户的过程。有许多方法可以实现这一点，我将在这里介绍一些更常见的方法。除了一些例外，我的示例将归结为几个可用的 npm 模块。您可以随时使用其他模块来实现相同的目标。

## HTTP 基本身份验证

第一个是**HTTP 基本身份验证**，它是可用的最简单的技术之一。它允许在 HTTP 请求中提交用户名和密码，并允许服务器在未发送预期凭据时限制访问。

在使用 Web 浏览器时，需要 HTTP 基本身份验证的页面将提示用户输入用户名和密码的对话框。用户输入信息后，浏览器通常会在一段时间内存储这些凭据，而不是在每个页面上不断提示用户。

这种方法的主要优点是非常简单实现。事实上，使用 Connect 可以在一行代码中完成。此外，这种方法完全是无状态的，不需要请求中的任何带外信息。

有一些重要的缺点，首先是它不是保密的。换句话说，基本的 HTTP 请求包括明文的用户名和密码。从技术上讲，它被编码为`base64`，但这不是一种加密方法。因此，这种技术必须与某种加密方法结合使用，例如 HTTPS。否则，请求可以被数据包嗅探器拦截，凭据就不再是秘密了。

此外，这种方法的效率不太理想。当请求页面需要 HTTP 基本身份验证时，服务器实际上必须处理第一次请求两次。在第一次尝试中，请求被拒绝，用户需要提供他们的凭据。在第二次尝试中，凭据与请求一起发送，服务器必须再次处理身份验证。根据用户名和密码的验证方式，这可能是每个请求都会产生不可接受的延迟。

此外，使用此方法时，浏览器没有实现注销的方式，除非关闭浏览器本身。凭据由浏览器存储，用户不会被提示控制存储时间的长短，或者何时应该过期。据我了解，只有 Internet Explorer 提供了这样的功能，但它需要 JavaScript 才能触发。

最后，作为开发者，你无法控制登录界面的外观；这完全取决于浏览器。虽然这可能归结为简单的美学，但可以说它比自定义解决方案更安全。如果你想要实现它，这是非常容易做到的。Connect（以及 Express）提供的捆绑中间件之一就是为了这个目的。它被称为`basicAuth`中间件，可以以多种方式进行配置。

### 提示

在使用中间件时，记住顺序非常重要！确保将身份验证中间件尽早放置在链中，这样你就可以对所有请求进行身份验证，而不是在验证用户身份之前运行不必要的处理。

首先，你可以简单地向中间件提供一个用户名和密码，为你的应用程序提供一个有效的凭据集，如下所示：

```js
app.use(express.basicAuth("admin", "123456"));
```

在这里，我们设置了我们的应用程序需要通过 HTTP 基本认证来要求用户名为`"admin"`，密码为`"123456"`。这是添加这种认证方法的最简单方法。

更高级的用法是提供一个同步回调函数，可以执行稍微复杂的身份验证方案，例如，你可以包含一个包含用户名和密码组合的 JavaScript 对象，以便执行内存查找。这在以下代码中有所体现：

```js
var users = {
    // username: "password"
    admin: "password",
    user: "123456"
};

app.use(express.basicAuth(function (user, pass) {
    return users.hasOwnProperty(user) && users[user] === pass;
}));
```

我们已经设置`basicAuth`来检查我们的`users`对象是否有相应的用户名和密码组合是有效的。如果回调函数返回`true`，则认证成功。否则，认证失败，服务器会做出相应的响应。

我们刚刚使用的两种方法都需要在应用程序源代码中硬编码凭据。最后一种方法很可能是你会使用的方法，如果你使用 HTTP 基本认证的话。这是异步回调验证。这允许你对请求进行验证，例如根据文本文件或数据库等外部来源。参考以下代码：

```js
app.use(express.basicAuth(function (user, pass, done) {
    User.authenticate({ username: user, password: pass }, done);
}));
```

在这个例子中，我们有一个类似的配置，我们使用了一个函数参数。这个函数，与之前的例子不同，有三个参数。它接收用户名和密码，但也接收一个回调函数，当它完成验证凭据时需要执行。出于简洁起见，我没有包括具体的实现细节。

重点是你可以异步执行操作，回调函数有自己的两个参数。按照 Node.js 的风格，如果认证失败，第一个参数是一个`Error`对象。第二个参数是用户的信息，将由中间件添加到`req.user`中，允许后续中间件函数访问用户的信息。

说到底，HTTP 基本认证可能对大多数应用程序来说是不够的。接下来，我们将讨论**HTTP 摘要认证**，它最初被设计为 HTTP 基本认证的继任者。

## HTTP 摘要认证

HTTP 摘要认证旨在比 HTTP 基本认证更安全，因为它不会以明文形式发送凭据。相反，它使用 MD5 单向哈希算法来加密用户的认证信息。值得注意的是，MD5 不再被认为是一种安全的算法，这是这种特定机制的一个缺点。

我只是为了完整起见才包括这个解释。它并不受欢迎，今天很少推荐使用，所以我不会再包括任何更多的细节或例子。

它以与 HTTP 基本认证相同的方式运作。首先，当需要认证时，客户端的初始请求被拒绝，服务器指示客户端需要使用 HTTP 摘要认证。客户端计算用户凭据和服务器认证领域的哈希值。根据规范，还有一些可选的功能可用于改进哈希算法并防止被恶意代理劫持。

HTTP 摘要认证的一个优点是密码不以明文形式在网络上传输。这种认证方法是在一个时代设计的，在那个时代，对所有网络事务运行 HTTPS/SSL 是非常昂贵的，无论是在金钱还是处理能力方面。现在那个时代已经过去，你应该在整个应用程序中一直使用 HTTPS。在这种情况下，HTTP 摘要认证相对于 HTTP 基本认证的优势几乎不存在。

## 介绍 Passport.js

现在，我将介绍一个非常受欢迎的用于 Connect 和 Express 应用程序的认证层项目。该项目是 Passport.js ([`passportjs.org/`](http://passportjs.org/))，实际上是一个旨在提供一致的 API 来进行认证的模块集合，使用许多不同的方法和提供者。本节的其余示例将使用 Passport.js API，并且我将在其中解释一些更常见的协议。

要在应用程序中使用 Passport.js，你需要配置以下三个部分：

1.  认证策略

1.  应用程序中间件

1.  会话（可选）

Passport.js 使用术语“策略”来指代认证请求的一种方法。这可以是用户名和密码，甚至第三方认证，比如 OpenID 或 OAuth。这是你将要配置的第一件事情，它将取决于你选择支持的认证方法。

作为一个起始示例，我们将看一下本地策略，其中你可以接受一个 HTTP `POST`请求，其中包含身份验证所需的用户名和密码，然后根据以下代码对其进行验证：

```js
// module dependencies
var passport = require("passport"),
    LocalStrategy = require("passport-local").Strategy;

// LocalStrategy means we perform the authentication ourselves
passport.use(new LocalStrategy(
    // this callback function performs the authentication check
    function (username, password, done) {
        // this is just a mock API call
        User.findOne({ username: username }, function (err, user) {
            // if a fatal error of some sort occurred, pass that along
            if (err) {
                done(err);
            // if we don't find a valid user
            } else if (!user || !user.validPassword(password)) {
                done(null, false, { message: "Incorrect username and password combination." });

            // otherwise, this was a successful authentication
            } else {
                done(null, user);
            }
        });
    }
));
```

为了简单起见，这不会连接到我们的应用程序，这只是演示了 Passport.js 中间件的 API。我们在这里配置了一个本地策略。这个策略接受一个验证回调，有三个参数：用户名、密码和一个回调函数，一旦认证完成就会被调用。（Passport.js 处理从`POST`请求中提取用户名和密码）回调函数有它自己的三个参数：一个`Error`对象（如果适用），用户的信息（如果适用，如果认证失败则为 false），以及一个选项哈希。

在这种情况下，验证回调调用某种用户 API（具体内容并不重要）来查找与提供的用户名匹配的用户，然后继续进行以下检查：

1.  如果发生致命错误（比如数据库宕机，或者网络断开连接），那么回调将发出`Error`对象作为它唯一的参数，这将被传递到 Passport.js 之外，由你的应用程序处理。

1.  如果该用户名不存在，或者密码无效，那么回调将以`null`作为第一个参数（因为没有错误发生），`false`作为第二个参数（因为认证本身失败了），以及一个具有单个`message`属性的对象，我们可以用它来向用户显示消息（这第三个参数是可选的）。

1.  如果用户通过了这些检查，那么认证就成功了。回调函数首先发出`null`，然后是用户的信息对象。

以这种方式使用回调允许 Passport.js 完全不知道底层实现。现在，让我们继续进行中间件配置步骤。Passport.js 专门设计用于在 Connect 和 Express 应用程序中使用，但它也适用于使用相同中间件风格的任何应用程序。

配置 Passport.js 和您的策略后，您需要附加至少一个中间件来在应用程序中初始化 Passport.js，如下所示的代码：

```js
var express = require("express"),
    app = express();

// application middleware
app.use(express.cookieParser());
app.use(express.bodyParser());
app.use(express.session({ secret: "long random string … " }));

// initialize passport
app.use(passport.initialize());
app.use(passport.session()); // optional session support

// more application middleware
app.use(app.router);
```

这是一个基本的 Express 应用程序，我们附加了两个与 Passport 相关的中间件：初始化和可选的会话支持。请记住，顺序很重要，所以您需要在像`bodyParser`和`session`这样的中间件之后初始化 Passport.js，但在应用程序路由之前。

会话支持中间件是可选的，但对于大多数应用程序来说是建议的，因为这是一个非常常见的用例，并且必须在 Express 自己的`session`中间件之后附加。最后，我们将配置会话支持本身如下所示的代码：

```js
passport.serializeUser(function (user, done) {
    // only store the user's ID in the session (to keep it light)
    done(null, user.id);
});

passport.deserializeUser(function (id, done) {
    // we can retrieve the user's information based on the ID
    User.findById(id, function (err, user) {
        done(err, user);
    });
});
```

存储所有可用的用户数据，特别是随着并发用户数量的增加，可能会很昂贵。因此，Passport.js 为开发人员提供了一种配置存储到会话中的内容以及检索用户数据的能力的方式（而不是在内存中持续保留）。这并不是必需的，因为使用共享数据库存储会话信息可以缓解这个问题。

在上面的例子中，`serializeUser`函数接收一个回调，当会话被初始化时执行。在这里，我们只将用户的 ID 存储到会话中，使其尽可能轻量，同时仍然为我们提供查找他们信息所需的信息。

相应的`deserializeUser`函数在每个后续请求上被调用，并将相应的用户数据添加到请求对象中。在这种情况下，我们使用一个通用 API 来查找用户，基于他们的 ID，并使用该数据发出回调。

正如您所看到的，配置和使用 Passport.js 非常简单，并且完全符合 Connect 和 Express 的方法论。Passport.js 有超过 120 种策略可用，您可以在他们的网站上找到更多文档和示例（[`passportjs.org/`](http://passportjs.org/)）。

## OpenID

OpenID 是一种用第三方服务在网络上进行身份验证的开放标准。其目的是允许用户在网络上拥有一个单一的身份，然后可以在许多应用程序中使用，而不需要在每个单独的应用程序中注册。OpenID 没有中央管理机构，每个提供商都是独立的，用户可以选择任何他信任的提供商。今天有许多主要的提供商，包括：Google、Yahoo！、PayPal 等。

OpenID 身份验证过程的操作大致如下（这是一个简化的解释）：用户由消费者呈现一个 OpenID 登录表单。用户输入他们提供商的 URL。消费者将用户重定向到他们的提供商，提供商对用户进行身份验证，并询问用户是否应与消费者共享任何信息。然后提供商将用户重定向回消费者，消费者允许用户使用他们的服务。

要在应用程序中包含 OpenID，我们将使用`passport-openid`模块。这个模块是 Passport.js 项目的一流模块，它为您提供了一种实现通用 OpenID 身份验证过程的策略。首先，让我们看看以下所需的 Passport.js 配置：

```js
var passport = require('passport'),
    OpenIDStrategy = require('passport-openid').Strategy;

// configure the OpenID Strategy
passport.use(new OpenIDStrategy(
    {
        // the URL that the provider will redirect the user to
        returnURL: 'http://www.example.com/auth/openid/return',
        // the realm should identify your application to the User
        realm: 'http://www.example.com/'
    },
    // this verify callback has 2 arguments:
    // identifier: the ID for your user (who they claim to be)
    // done: the callback to issue after you've looked the user up
    function (id, done) {
        // this is a generic API, it could be any async operation
        User.findOrCreate({ openId: id }, function (err, user) {
            done(err, user);
        });
    }
));
```

我们已经包含了`passport`和`passport-openid`模块，并配置了 OpenID 策略。配置对象（第一个参数）有两个必需属性：

+   `returnURL`：这是 OpenID 提供商将用户重定向回您的应用程序的 URL。

+   `领域`：这是提供者将向用户显示的内容，以识别您的应用程序

第二个参数是验证回调，只接受两个参数：

+   标识符：这是用户如何在您的应用程序中标识自己

+   `完成`：这是您的应用程序根据标识符查找用户后发出的回调

现在，您需要配置 Express 路由，以处理登录请求，如下面的代码所示：

```js
// this route accepts the user"s login request, passport handles the redirect
// over to the Provider for authentication
app.post("/auth/openid", passport.authenticate("openid"));

// the Provider will redirect back to this URL (based on our earlier
// configuration of the strategy) and it will tell us whether or not
// the authentication was successful
app.get("/auth/openid/return", passport.authenticate("openid", {
    // if successful, we'll redirect the user to the hame page
    successRedirect: "/",
    // otherwise, send back to the login page
    failureRedirect: "/login"
}));
```

我们已经配置了两个路由，第一个路由通过`POST`接收用户的登录请求，Passport.js 负责将用户重定向到提供者。提供者已配置为将用户发送回`returnURL`，该 URL 对应于我们之前配置的第二个路由。

接下来，您需要在登录页面上添加一个 HTML 表单，该表单`POST`到我们之前配置的路由。如下面的代码所示：

```js
<form action="/auth/openid" method="post">
    <div>
        <label>OpenID:</label>
        <input type="text" name="openid_identifier"/><br/>
    </div>
    <div>
        <input type="submit" value="Sign In"/>
    </div>
</form>
```

唯一需要的 HTML 输入是一个具有名称`"openid_identifier"`的输入。每种策略都有自己的要求，因此在实施它们时，请务必阅读每种策略的文档。

我们在这里配置的是使用 Passport.js 的基本 OpenID 身份验证实现。现在，我们将继续配置基本的 OAuth 实现以进行身份验证。

OpenID 旨在允许您的身份由受信任的第三方进行身份验证，而 OAuth 旨在允许用户在不需要向每个单独的方提供凭据的情况下，在不同的应用程序之间共享信息。如果您的应用程序需要与另一个服务共享数据，那么您很可能会从该特定服务中使用 OAuth API。如果您只需要验证身份，OpenID 可能是该服务的首选机制。

## OAuth

OAuth 允许用户在不需要向两个服务共享其用户名和密码的情况下，从一个应用程序共享资源到另一个应用程序。此外，它还具有附加功能，可以提供有限的访问权限。此限制可以基于时间，即在经过一定时间后撤销访问权限。它还可以限制对特定数据集的访问，并可能使用户更多地控制他们决定分享什么。

这个过程通过使用几组不同的密钥（更精确地说是三组）来完成。授权过程的每个阶段都建立在前一组密钥的基础上，以构建下一步的密钥。此外，在每个步骤之间，用户在其他应用程序之间进行重定向，确保用户只向每个应用程序提供所需的最少信息。我在这里给出的解释是简化的，并没有涵盖诸如加密和签名等技术细节。

OAuth 的最佳隐喻是“代客泊车钥匙”。一些豪华汽车配有一把特殊的钥匙，其访问权限受限。我的意思是，这把特殊的钥匙只允许汽车行驶一小段距离，并且只允许代客泊车司机在拥有该钥匙的情况下访问汽车。这与 OAuth 所实现的非常相似，它允许所有者对他们拥有的资源进行临时和有限的访问，同时不放弃对该资源的完全控制。

通常涉及三方：`客户端`、`服务器`和`资源所有者`。客户端将代表资源所有者向服务器请求资源。

要使用 OAuth 规范使用的相同真实世界示例，想象一下简已经将一些个人照片上传到一个照片共享网站，并希望通过另一个在线服务将它们打印出来。

为了打印服务（客户端）能够访问存储在照片服务（服务器）中的照片，他们将需要来自 Jane（资源所有者）的批准。首先，任何客户端应用程序都需要向任何服务器应用程序注册自己，以获取第一组密钥，即客户端密钥。这些密钥被客户端和服务器都知道，并允许服务器首先验证客户端的身份。

Jane 准备好打印她的照片，所以她访问打印服务开始这个过程。她希望从照片服务中获取她的照片，而不是需要将它们上传到另一个服务，所以她告诉打印服务她希望使用照片服务的照片。

现在，打印服务通过安全的 HTTPS 请求将他们的客户端密钥发送给照片服务，以检索一组临时密钥。这些密钥用于在各种重定向中标识指定的授权请求。

一旦检索到临时密钥，打印服务将 Jane 重定向到照片服务。在那里，Jane 需要通过照片服务使用的任何方法验证她的身份。此外，照片服务可以向 Jane 提供选项，以限制授权的持续时间和范围。

一旦完成此验证，Jane 将被重定向回打印服务，并获得临时令牌。她已经授权打印服务访问照片服务，后者现在将临时密钥交换为最后一组密钥，即令牌密钥。

现在，打印服务可以使用这个“访问令牌”根据 Jane 允许的参数从照片服务请求信息，并且可以随时由 Jane 或照片服务撤销。在下面的示例中，我将坚持使用 Facebook 模块，该模块使用 OAuth v2.0，而不是使用通用的`passport-oauth`模块。我选择这条路线是为了避免需要展示当今使用的所有 OAuth 变体，因为每个实现可能都有自己的变体。此外，这里的示例将为您提供足够的 Passport API 介绍，以便您可以将这种方法应用到任何其他提供者。

首先，我们需要安装`passport-facebook`模块，然后根据以下代码配置 Passport.js 策略：

```js
var passport = require('passport'),
    FacebookStrategy = require('passport-facebook').Strategy;

// configuring the Facebook strategy (OAuth v2.0)
passport.use(new FacebookStrategy(
    {
        // developers must register their application with Facebook
        // this is where the ID/Secret are obtained
        clientID: FACEBOOK_APP_ID,
        clientSecret: FACEBOOK_APP_SECRET,

        // this is the URL that Facebook will redirect the user to
        callbackURL: "http://www.example.com/auth/facebook/callback"
    },

    // the verify callback has 4 arguments here:
    // accessToken: the token Facebook uses to verify authentication
    // refreshToken: used to extend the lifetime of the accessToken
    // profile: the user's shared information
    // done: the callback function
    function (accessToken, refreshToken, profile, done) {
        // here is where your application connects the 2 accounts
        User.findOrCreate(..., function (err, user) {
            done(err, user);
        });
    }
));
```

为了使用 Facebook 身份验证，您需要在 Facebook 开发者（[`developers.facebook.com/`](https://developers.facebook.com/)）注册并创建一个应用程序帐户。这对其他服务可能是类似的过程；您需要在他们那边进行某种注册，以便安全地与他们的用户协调。从那里，您可以获取`clientID`和`clientSecret`，并将其放入前面的配置中。您还需要指定一个`callbackURL`，它的行为非常类似于 OpenID 的`returnURL`。

接下来，您需要根据以下代码为您的 Express 应用程序配置路由：

```js
// redirects the User to Facebook for authentication
app.get("/auth/facebook", passport.authenticate("facebook"));

// Facebook will redirect back to this URL based on the strategy configuration
app.get("/auth/facebook/callback", passport.authenticate("facebook", {
    successRedirect: "/",
    failureRedirect: "/login"
}));
```

这与我们为 OpenID 设置的路由非常相似，但有一个主要区别。初始路由不是 HTML 表单`POST`；它是一个简单的 HTTP`GET`。这意味着您可以设置一个简单的 HTML 锚点，将它们指向这个路由，如下所示：

```js
<a href="/auth/facebook">Login with Facebook</a>
```

Passport 将用户发送到 Facebook 进行身份验证。当 Facebook 完成后，它将重定向回第二个路由，您可以根据需要重定向用户（就像 OpenID 实现一样）。

Passport.js 是一个很好的 API，可以抽象出所有您的身份验证需求，所以深入研究它的 API 文档（[`passportjs.org/`](http://passportjs.org/)），并利用他们提供的 120 多种策略的任意组合。

# 授权

授权是确定用户对应用程序中受限资源或操作的访问权限。身份验证专门处理用户是谁，而授权假设我们知道他们是谁，并且必须确定他们可以做什么。Express 为我们提供了一种优雅的方式，将授权直接构建到我们的路由中，这通常是授权发生的地方。

许多人最初没有意识到的是，关于 express 路由的是，您可以在定义路由时传递多个处理程序。它们中的每一个都像任何其他中间件一样，如下面的代码所示：

```js
function restrict(req, res, next) {
    if (req.user) {
        return next();
    } else {
        res.send(403); // Forbidden
    }
}

app.get("/restricted", restrict, function (req, res) {
    res.send("Hello, " + req.user);
});
```

我们的限制函数检查用户数据（假设它由我们的身份验证层设置），如果用户有效，则允许链继续进行。如果用户未登录，它将简单地响应**403（禁止）**。

关键在于，您可以使用多个路由处理程序作为处理预条件的机会，例如检查用户的身份验证状态、他们的角色或关于访问的任何其他规则。其中许多内容高度依赖于您如何构建应用程序以及您如何确定用户可以访问什么。

其中一个更受欢迎的授权方案是基于角色的授权。用户可以拥有任意数量的角色，例如："member"，"moderator"或"admin"。这些角色中的每一个都可以用来确定他们在每个操作上的访问权限。

```js
// dummy user data
var users = [
    { id: 1, name: "dominic", role: "admin" },
    { id: 2, name: "matthew", role: "member" },
    { id: 3, name: "gabriel", role: "member" }
];

// middleware for loading a user based on a :user param in the route
function loadUser(req, res, next) {
    req.userData = users[req.params.user];
    return next();
}

// middleware for restricting a route to only a specified role name
function requireRole(role) {
    // returns a function, closure allows us to access the role variable
    return function (req, res, next) {
        // check if the logged-in user's role is correct
        if (req.user.role === role) {
            return next();
        } else {
            return next(new Error("Unauthorized"));
        }
    };
}

// this route only loads a user's data (so it is loaded via middleware)
app.get("/users/:user", loadUser, function (req, res) {
    res.send(req.user.name);
});

// this route can only be called upon by an admin
app.del("/users/:user", requireRole("admin"), loadUser, function (req, res) {
    res.send("User deleted");
});
```

在上述代码中，我们有一个可用用户的列表。假设我们已经有一个身份验证层，当用户登录时加载用户配置数据，让我们看一下我们定义的两个中间件。

首先，`loadUser`是一个简单的中间件函数，用于为指定的路由加载用户（这可能是与登录用户不同的用户）。在这里，我们只有一个硬编码的列表，但它可以是我们异步进行的数据库调用。

其次，`requireRole`中间件对于不熟悉闭包或一级函数的人来说有点复杂。我们在这里做的是返回中间件函数，而不是简单地使用命名函数。通过闭包，我们可以在返回的函数内部访问`role`参数。这个中间件函数确保经过身份验证的用户具有我们要求的角色。

我们有两个路由，第一个（显示用户数据）是公开的，因此它只是通过中间件加载用户数据，不进行授权检查。第二个路由（删除用户）要求经过身份验证的用户是管理员。如果检查通过，用户的数据将被加载，并且路由将按预期进行。

有许多授权方法可供选择，有许多好的模块可供选择。基于角色的授权，正如我们在这里所展示的，易于在 Express 中实现，并且在逻辑上通常易于理解。与身份验证一样，您的实现取决于您最终如何构建应用程序。我在这里的主要目的是定义授权并向您展示一些示例，以帮助您尽可能将该机制与应用程序的其余部分区分开来。

# 安全日志记录

安全的另一个重要方面是日志记录，或者记录应用程序中的各种事件，以便对异常进行分析。这些异常可以被审查，以便检测攻击者试图绕过安全方法的地方，并且在实际入侵之前检测到这些活动，可以采取进一步的步骤来减轻这些风险。除了安全之外，日志记录还可以帮助检测程序中为用户造成问题的情况，并允许您更轻松地重现和解决这些问题。

您特定的应用程序和环境将驱动您的日志记录方法。通过方法，我指的是记录和存储日志的方式，例如在文件系统中使用平面文件，使用某种数据库，甚至使用第三方日志记录服务。虽然这些方法在项目之间可能有很大的不同，但记录的事件类型和相关信息应该在整个范围内保持相对一致。

**开放式网络应用安全项目**（**OWASP**）在其网站上有一个关于确定日志记录策略的很好的指南（访问[`www.owasp.org/index.php/Logging_Cheat_Sheet`](https://www.owasp.org/index.php/Logging_Cheat_Sheet) 获取更多信息）。他们建议为以下特定类型的事件记录日志：

+   输入验证失败（例如，协议违规，不可接受的编码，无效的参数名称和值）

+   输出验证失败（例如，数据库记录集不匹配和无效数据编码）

+   身份验证成功和失败

+   授权失败

+   会话管理失败（例如，cookie 会话标识值修改）

+   应用程序错误和系统事件（例如，语法和运行时错误，连接问题，性能问题，第三方服务错误消息，文件系统错误，文件上传病毒检测，以及配置更改）

+   应用程序和相关系统的启动和关闭，以及日志初始化（启动和停止）

+   使用更高风险功能（例如，网络连接，添加或删除用户，权限更改，将用户分配给令牌，添加或删除令牌，使用管理权限，应用管理员访问，访问付款卡持有人数据，使用数据加密密钥，密钥更改，创建和删除系统级对象，数据导入和导出，包括基于屏幕的报告，以及提交用户生成的内容，特别是文件上传）

+   法律和其他选择（例如，移动电话功能的权限，使用条款，条款和条件，个人数据使用同意，以及接收营销通讯的许可）

除了他们的建议，OWASP 还将以下事件作为可选事件呈现：

+   排序失败

+   过度使用

+   数据更改

+   欺诈和其他犯罪活动

+   可疑，不可接受或意外行为

+   配置修改

+   应用程序代码文件和/或内存更改

在确定要存储的日志数据时，OWASP 建议避免以下类型的数据：

+   应用程序源代码

+   会话标识值（如果需要跟踪特定会话事件，则考虑替换为哈希值）

+   访问令牌

+   敏感个人数据和某些形式的个人身份信息（PII）

+   身份验证密码

+   数据库连接字符串

+   加密密钥

+   银行账户或付款卡持有人数据

+   比日志系统允许存储的更高安全级别的数据

+   商业敏感信息

+   在相关司法管辖区内收集的非法信息

+   用户选择退出收集的信息，或者未同意收集的信息，例如使用不跟踪，或者同意收集已过期

在某些情况下，以下信息在调查过程中可能有用，但在包含在应用程序日志中之前应仔细审查：

+   文件路径

+   数据库连接字符串

+   内部网络名称和地址

+   非敏感个人数据（例如，个人姓名，电话号码，电子邮件地址）

由于每个应用程序和环境都不同，日志记录的方法也可以多种多样。我们将在这里看一下的 npm 模块旨在提供一个统一的 API，可以使用多种不同的方法，同时取决于上下文，允许您同时使用多种方法。

`winston`模块（[`github.com/flatiron/winston`](https://github.com/flatiron/winston)）提供了一个清晰易用的 API，用于编写日志。此外，它支持许多日志记录方法，包括添加自定义传输的功能。传输可以被描述为给定一组日志的存储或显示机制。

`winston`模块具有内置传输（也称为核心模块），用于将日志记录到控制台、将日志记录到文件以及通过 HTTP 发送日志。除了核心模块外，还有官方支持的传输模块，例如`CouchDB`、`Redis`、`MongoDB`、`Riak`和`Loggly`。最后，`winston` API 也有一个充满活力的社区，目前有超过 23 种不同的自定义传输，包括电子邮件传输和各种云服务，如亚马逊的**SimpleDB**和**Simple Notification Service**（**SNS**）。重点是，您可能需要的任何传输，可能已经有可用的模块，当然您也可以自己编写。

要开始使用`winston`，请通过 npm 安装它，然后您可以立即使用“默认记录器”，如下面的代码所示：

```js
var winston = require('winston');
winston.log("info", "Hello World");
winston.info("Hello Again");
```

这绝对是最快速开始使用`winston`的方法，但默认情况下只使用控制台传输。虽然默认记录器可以通过更多传输和配置进行扩展，但更灵活的方法是创建自己的`winston`实例，可以在应用程序中的各种上下文中使用。如下面的代码所示：

```js
var winston = require("winston");

var logger = new (winston.Logger)({
    transports: [
        new (winston.transports.Console)(),
        new (winston.transports.File)({ filename: 'somefile.log' })
    ]
});
```

在应用程序代码中，我通常将此类模块的样板代码放在它们自己的文件中。从那里，您可以导出一个预配置的对象，可以在整个应用程序中导入和使用，例如，您可以创建一个名为`lib/logger.js`的文件，看起来像下面的内容：

```js
var path = require("path"),
    winston = require("winston");

module.exports = new (winston.Logger)({
    transports: [
        // only logs errors to the console
        new (winston.transports.Console)({
            level: "error"
        }),
        // all logs will be saved to this app.log file
        new (winston.transports.File)({
            filename: path.resolve(__dirname, "../logs/app.log")
        }),
        // only errors will be saved to errors.log, and we can examine 
        // to app.log for more context and details if needed.
        new (winston.transports.File)({
            level: "error",
            filename: path.resolve(__dirname, "../logs/errors.log")
        })
    ]
});
```

然后在应用程序的其他部分中，您可以包含记录器并轻松使用它，如下所示：

```js
var logger = require("./lib/logger");
logger.log("info", "Hello World");
logger.info("Hello Again");
```

此外，`winston`还包括其他高级功能，如自定义日志级别、额外的传输配置和处理未处理的异常。此外，`winston`并不是 Node.js 中唯一可用的日志记录 API，还有其他可供您考虑的替代方案，具体取决于您自己的需求。更不用说开发自己的定制解决方案来完全控制了。

# 错误处理

任何应用程序的重要方面之一是如何处理错误。如前所述，未捕获的异常可能会导致应用程序崩溃，因此能够正确处理错误是开发周期的重要部分。

对自己应用程序中的错误做出响应是关键，因此请参阅第二章，*一般注意事项*，了解如何处理 Node.js 中的错误的一般介绍。在这里，我们将专门处理 Connect 和 Express。

首先，在路由处理程序中不要直接抛出错误。虽然 Express 足够聪明，可以直接在路由处理程序上尝试/捕获错误，但如果您正在执行某种异步操作（这在大多数情况下都是如此），则这对您没有帮助，如下面的代码所示：

```js
app.get("/throw/now", function (req, res) {
    // Express wraps the route handler invocation in try/catch, so
    // this will be handled without crashing the server
    throw new Error("I will not crash the server;
});

app.get("/throw/async", function (req, res) {
    // However, when performing some asynchronous operation
    // time) then you will lose your server if you throw
    setTimeout(function () {
        // try/catch does not work on callbacks/asynchronous code!
        throw new Error("I WILL crash the server");
    }, 100);
});
```

前面两个处理程序都会抛出异常。如前所述，Express 将在`try/catch`中执行处理程序，以处理处理程序本身中抛出的异常。但是，异步代码（例如第二个路由）无法使用典型的 try/catch，最终会变成未捕获的异常。简而言之，在处理错误时不要使用`throw`！

除了传递给处理程序的请求和响应对象之外，还有第三个参数可以像其他中间件一样使用。这通常被称为“next”回调，并且您可以像在中间件中一样使用它，传递给连续中的下一个项目。如下面的代码所示：

```js
app.get("/next", function (req, res, next) {
    // this is the correct way to handle errors, as Express will
    // delegate the error to special middleware
    return next(new Error("I'm passed to Express"));
});
```

如果您使用`Error`对象作为第一个参数执行下一个回调，那么 Connect 将接管该错误并委托给您配置的任何错误处理中间件。当您设置一个接受四个参数的中间件时，它总是被视为错误处理中间件。

```js
// 4 arguments tells Express that the middleware is for errors
// you can have more than 1 if necessary
app.use(function (err, req, res, next) {
    console.trace();
    console.error(err);

    // just responds with a 500 status code and the error message
    res.send(500, err.message);
});
```

这个特殊的错误处理中间件放在应用程序堆栈的最后，如果有必要，您可以设置多个。您可以像其他中间件一样通过 next 传递控制，例如，设置多层错误处理，其中一层可以发送电子邮件，一层可以记录到文件，最后一层可以向用户发送响应。

Connect 还有一个特殊的中间件，您可以利用它来处理错误，而无需硬编码自己的中间件。这是`errorHandler`中间件，当发生错误时，它将自动响应纯文本、JSON 或 HTML（取决于客户端的标头）。这个中间件表达如下：

```js
app.use(express.errorHandler());
```

通常，这个辅助程序只用于开发，因为您的生产应用程序可能在处理错误时需要更多的工作，您需要完全控制。

总之，始终在路由处理程序中使用“next”回调函数来传达错误，永远不要使用 throw。此外，始终通过添加一个带有四个参数的中间件函数来配置某种错误处理中间件。在开发中使用 Connect 的内置处理程序，并为生产环境设置自己的位置。

# 总结

在本章中，我们考虑了适用于应用程序的高级安全性考虑因素，如身份验证、授权和错误处理。在下一章中，我们将研究应用程序请求阶段出现的漏洞。
