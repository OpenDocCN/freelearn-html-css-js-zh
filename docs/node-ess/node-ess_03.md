# 第三章：认证

我们现在可以创建 RESTful API，但我们不希望每个人都能访问我们暴露的所有内容。我们希望路由是安全的，并且能够跟踪谁在做什么。

Passport 是一个很棒的模块，另一个中间件，帮助我们验证请求。

Passport 公开了一个简单的 API，供提供者扩展并创建策略来验证用户。在撰写本文时，有 307 个官方支持的策略；但是，您完全可以编写自己的策略并发布供他人使用。

# 基本身份验证

passport 最简单的策略是接受用户名和密码的本地策略。

我们将为这些示例引入 express 框架，现在您已经了解了它在底层的基本工作原理，我们可以将它们整合在一起。

您可以安装`express`、`body-parser`、`passport`和`passport-local`。Express 是一个内置电池的 Node.js Web 框架，包括路由和使用中间件的能力：

```js
[~/examples/example-19]$ npm install express body-parser passport passport-local

```

目前，我们可以将我们的用户存储在一个简单的对象中，以便以后引用，如下所示：

```js
var users = {
    foo: {
        username: 'foo',
        password: 'bar',
        id: 1
    },
    bar: {
        username: 'bar',
        password: 'foo',
        id: 2
    }
}
```

一旦我们有了一些用户，我们就需要设置 passport。当我们创建本地策略的实例时，我们需要提供一个`verify`回调，其中我们检查用户名和密码，同时返回一个用户：

```js
var Passport = require( 'passport' ),
    LocalStrategy = require( 'passport-local' ).Strategy;

var localStrategy = new LocalStrategy({
    usernameField: 'username',
    passwordField: 'password'
  },
  function(username, password, done) {
    user = users[ username ];

    if ( user == null ) {
        return done( null, false, { message: 'Invalid user' } );
    }

    if ( user.password !== password ) {
        return done( null, false, { message: 'Invalid password' } );    
    }

    done( null, user );
  }
)
```

在这种情况下，`verify`回调期望使用`done`调用用户。它还允许我们在用户无效或密码错误时提供信息。

现在，我们有了一个策略，我们可以将其传递给 passport，这允许我们以后引用它并用它来验证我们的请求，如下所示：

```js
Passport.use( 'local', localStrategy );
```

您可以在每个应用程序中使用多种策略，并通过您传递的名称引用每个策略，在这种情况下是`'local'`。

现在，让我们创建我们的服务器，如下所示：

```js
var Express = require( 'express' );

var app = Express( );
```

我们将不得不使用`body-parser`中间件。这将确保当我们发布到我们的登录路由时，我们可以读取我们的主体；我们还需要初始化 passport：

```js
var BodyParser = require( 'body-parser' );
app.use( BodyParser.urlencoded( { extended: false } ) );
app.use( BodyParser.json( ) );
app.use( Passport.initialize( ) );
```

要登录到我们的应用程序，我们需要创建一个使用身份验证的`post`路由作为处理程序之一。其代码如下：

```js
app.post(
    '/login',
    Passport.authenticate( 'local', { session: false } ),
    function ( request, response ) {

    }
);
```

现在，当我们向`/login`发送`POST`请求时，服务器将验证我们的请求。

经过身份验证后，`user`属性将填充在请求对象上，如下所示：

```js
app.post(
    '/login',
    Passport.authenticate( 'local', { session: false } ),
    function ( request, response ) {
        response.send( 'User Id ' + request.user.id );
    }
);
```

最后，我们需要监听请求，就像所有其他服务器一样：

```js
app.listen( 8080, function( ) {
    console.log( 'Listening on port 8080' );
});
```

让我们运行示例：

```js
[~/examples/example-19]$ node server.js
Listening on port 8080

```

现在，当我们向服务器发送`POST`请求时，我们可以验证用户。如果用户没有同时传递用户名和密码，服务器将返回`400 Bad Request`。

### 提示

如果您不熟悉`curl`，您可以使用诸如 Advanced REST Client 之类的工具：

[`chromerestclient.appspot.com/`](https://chromerestclient.appspot.com/)

在接下来的示例中，我将使用命令行界面`curl`。

我们可以通过执行`POST`到`/login`命令来执行登录请求：

```js
[~]$ curl -X POST http://localhost:8080/login -v
< HTTP/1.1 400 Bad Request

```

如果用户提供了错误的详细信息，那么将返回`401 Unauthorized`：

```js
[~]$ curl -X POST http://localhost:8080/login \
 -H 'Content-Type: application/json' \
 -d '{"username":"foo","password":"foo"}' \
 -v
< HTTP/1.1 401 Unauthorized

```

如果我们提供了正确的详细信息，那么我们可以看到我们的处理程序被调用，并且正确的数据被返回：

```js
[~]$ curl -X POST http://localhost:8080/login \
 -H 'Content-Type: application/json' \
 -d '{"username":"foo","password":"bar"}'
User Id 1
[~]$ curl -X POST http://localhost:8080/login \
 -H 'Content-Type: application/json' \
 -d '{"username":"bar","password":"foo"}'
User Id 2

```

# Bearer 令牌

现在我们有了一个经过身份验证的用户，我们可以生成一个令牌，以便在将来的请求中使用，而不是在任何地方都传递我们的用户名和密码。这通常被称为 Bearer 令牌，方便的是，passport 有一个策略可以实现这一点。

对于我们的令牌，我们将使用一种称为**JSON Web Token**（**JWT**）的东西。JWT 允许我们从 JSON 对象中编码令牌，然后解码和验证它们。存储在其中的数据是开放和简单的，因此不应该在其中存储密码；但是，它使验证用户变得非常简单。我们还可以为这些令牌提供到期日期，这有助于限制令牌被暴露的严重性。

您可以在[`jwt.io/`](http://jwt.io/)上阅读有关 JWT 的更多信息。

您可以使用以下命令安装 JWT：

```js
[~/examples/example-19]$ npm install jsonwebtoken

```

一旦用户经过身份验证，我们就可以安全地为他们提供一个令牌，以便在将来的请求中使用：

```js
var JSONWebToken = require( 'jsonwebtoken' ),
    Crypto = require( 'crypto' );

var generateToken = function ( request, response ) {

    // The payload just contains the id of the user
    // and their username, we can verify whether the claim
    // is correct using JSONWebToken.verify     
    var payload = {
        id: user.id,
        username: user.username
    };
    // Generate a random string
    // Usually this would be an app wide constant
    // But can be done both ways
    var secret = Crypto.randomBytes( 128 )
                       .toString( 'base64' );
    // Create the token with a payload and secret
    var token = JSONWebToken.sign( payload, secret );

    // The user is still referencing the same object
    // in users, so no need to set it again
    // If we were using a database, we would save
    // it here
    request.user.secret = secret

    return token;
}

var generateTokenHandler = function ( request, response  ) {
    var user = request.user;    
    // Generate our token
    var token = generateToken( user );
    // Return the user a token to use
    response.send( token );
};

app.post(
    '/login',
    Passport.authenticate( 'local', { session: false } ),
    generateTokenHandler
);
```

现在，当用户登录时，他们将收到一个我们可以验证的令牌。

让我们运行我们的 Node.js 服务器：

```js
[~/examples/example-19]$ node server.js
Listening on port 8080

```

现在我们登录时会收到一个令牌：

```js
[~]$ curl -X POST http://localhost:8080/login \
 -H 'Content-Type: application/json' \
 -d '{"username":"foo","password":"bar"}'
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZC
I6MSwidXNlcm5hbWUiOiJmb28iLCJpYXQiOjE0MzcyO
TQ3OTV9.iOZO7oCIceZl6YvZqVP9WZLRx-XVvJFMF1p
pPCEsGGs

```

我们可以将此输入调试器中的[`jwt.io/`](http://jwt.io/)并查看内容，如下所示：

```js
{
  "id": 1,
  "username": "foo",
  "iat": 1437294795
}
```

如果我们有密钥，我们可以验证令牌是否正确。签名每次请求令牌时都会更改：

```js
[~]$ curl -X POST http://localhost:8080/login \
 -H 'Content-Type: application/json' \
 -d '{"username":"foo","password":"bar"}'
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZC
I6MSwidXNlcm5hbWUiOiJmb28iLCJpYXQiOjE0MzcyO
TQ5OTl9.n1eRQVOM9qORTIMUpslH-ycTNEYdLDKa9lU
pmhf44s0

```

我们可以使用`passport-bearer`对用户进行身份验证；它的设置方式与`passport-local`非常相似。但是，与其从主体接受用户名和密码不同，我们接受一个持票人令牌；这可以通过查询字符串、主体或`Authorization`标头传递：

首先，我们必须安装`passport-http-bearer`：

```js
[~/examples/example-19]$ npm install passport-http-bearer

```

然后让我们创建我们的验证器。有两个步骤：第一步是确保解码的信息与我们的用户匹配，这通常是我们检索用户的地方；然后，一旦我们有一个用户并且它是有效的，我们可以根据用户的密钥检查令牌是否有效：

```js
var BearerStrategy = require( 'passport-http-bearer' ).Strategy;

var verifyToken = function( token, done ) {
    var payload = JSONWebToken.decode( token );
    var user = users[ payload.username ];
    // If we can't find a user, or the information
    // doesn't match then return false
    if ( user == null ||
         user.id !== payload.id ||
         user.username !== payload.username ) {
        return done( null, false );
    }
    // Ensure the token is valid now we have a user
    JSONWebToken.verify( token, user.secret, function ( error, decoded ) {
        if ( error || decoded == null ) {
            return done( error, false );
        }
        return done( null, user );
    });
}   
var bearerStrategy = new BearerStrategy(
    verifyToken
)
```

我们可以将此策略注册为持票人，以便以后使用：

```js
Passport.use( 'bearer', bearerStrategy );
```

我们可以创建一个简单的路由，用于检索经过身份验证的用户的用户详细信息：

```js
app.get(
    '/userinfo',
    Passport.authenticate( 'bearer', { session: false } ),
    function ( request, response ) {
        var user = request.user;
        response.send( {
            id: user.id,
            username: user.username
        });
    }
);
```

让我们运行 Node.js 服务器：

```js
[~/examples/example-19]$ node server.js
Listening on port 8080

```

一旦我们收到一个令牌：

```js
[~]$ curl -X POST http://localhost:8080/login \
 -H 'Content-Type: application/json' \
 -d '{"username":"foo","password":"bar"}'

```

我们可以在我们的请求中使用结果：

```js
[~]$ curl -X GET http://localhost:8080/userinfo \
 -H 'Authorization: Bearer <token>'
{"id":1,"username":"foo"}

```

# OAuth

OAuth 提供了许多优势；例如，它不需要处理用户的实际识别。我们可以让用户使用他们信任的服务登录，例如 Google、Facebook 或 Auth0。

在接下来的示例中，我将使用`Auth0`。他们提供了一个免费帐户供您使用：[`auth0.com/`](https://auth0.com/)。

您需要注册并创建一个`api`（选择`AngularJS + Node.js`），然后转到设置并记下域、客户端 ID 和客户端密钥。您需要这些来设置`OAuth`。

我们可以使用`passport-oauth2`使用 OAuth 进行身份验证：

```js
[~/examples/example-19]$ npm install --save passport-oauth2

```

与我们的持票人令牌一样，我们希望验证服务器返回的内容，这将是一个具有 ID 的用户对象。我们将与我们的数据中的用户匹配或创建一个新用户：

```js
var validateOAuth = function ( accessToken, refreshToken, profile, done ) {

    var keys = Object.keys( users ), user = null;

    for( var iKey = 0; iKey < keys.length; i += 1 ) {
        user = users[ key ];
        if ( user.thirdPartyId !== profile.user_id ) { continue; }
        return done( null, user );
    }

    users[ profile.name ] = user = {
        username: profile.name,
        id: keys.length,
        thirdPartyId: profile.user_id
    }
    done( null, user );

};
```

一旦我们有一个验证用户的函数，我们就可以为我们的 OAuth 策略组合选项：

```js
var oAuthOptions = {
    authorizationURL: 'https://<domain>.auth0.com/authorize',
    tokenURL: 'https://<domain>.auth0.com/oauth/token',
    clientID: '<client id>',
    clientSecret: '<client secret>',
    callbackURL: "http://localhost:8080/oauth/callback"
}
```

然后我们创建我们的策略，如下所示：

```js
var OAuth2Strategy = require( 'passport-oauth2' ).Strategy;
oAuthStrategy = new OAuth2Strategy( oAuthOptions, validateOAuth );
```

在使用我们的策略之前，我们需要使用我们自己的策略`userProfile`方法进行鸭子类型处理，这样我们就可以请求用户对象在`validateOAuth`中使用：

```js
var parseUserProfile = function ( done, error, body ) {
    if ( error ) {
        return done( new Error( 'Failed to fetch user profile' ) )
    }

    var json;
    try {
        json = JSON.parse( body );
    } catch ( error ) {
        return done( error );
    }
    done( null, json );
}

var getUserProfile = function( accessToken, done ) {
    oAuthStrategy._oauth2.get(
        "https://<domain>.auth0.com/userinfo",
        accessToken,
        parseUserProfile.bind( null, done )
    )
}
oAuthStrategy.userProfile = getUserProfile
```

我们可以将此策略注册为`oauth`，以便以后使用：

```js
Passport.use( 'oauth', oAuthStrategy );
```

我们需要创建两个路由来处理我们的 OAuth 身份验证：一个路由用于启动流程，另一个用于识别服务器返回：

```js
app.get( '/oauth', Passport.authenticate( 'oauth', { session: false } ) );
```

我们可以在这里使用我们的`generateTokenHandler`，因为我们的请求上会有一个用户。

```js
app.get( '/oauth/callback',
  Passport.authenticate( 'oauth', { session: false } ),
  generateTokenHandler
);
```

我们现在可以启动我们的服务器并请求`http://localhost:8080/oauth`；服务器将重定向您到`Auth0`。登录后，您将收到一个令牌，您可以在`/userinfo`中使用。

如果您使用会话，您可以将用户保存到会话中，并将其重定向回您的首页（或为已登录用户设置的默认页面）。对于单页应用程序，例如使用 Angular 时，您可能希望将用户重定向到 URL 中带有令牌，以便客户端框架抓取并保存。

# 总结

我们现在可以对用户进行身份验证；这很棒，因为我们现在可以弄清楚这些人是谁，然后限制用户访问某些资源。

在下一章中，我们将介绍调试，如果我们的用户没有被验证，我们可能需要使用它。

为 Bentham Chang 准备，Safari ID bentham@gmail.com 用户编号：2843974 © 2015 Safari Books Online，LLC。此下载文件仅供个人使用，并受到服务条款的约束。任何其他用途均需著作权所有者的事先书面同意。未经授权的使用、复制和/或分发严格禁止并违反适用法律。保留所有权利。
