# 第七章：实施安全性、加密和身份验证

在本章中，我们将涵盖：

+   实施基本身份验证

+   密码加密哈希

+   实施摘要身份验证

+   设置 HTTPS Web 服务器

+   防止跨站点请求伪造

# 介绍

在生产 Web 服务器方面，安全性是至关重要的。安全性的重要性与我们提供的数据或服务的重要性相关。但即使对于最小的项目，我们也希望确保我们的系统不容易受到攻击。

许多 Web 开发框架提供了内置的安全性，这是一个双刃剑。一方面，我们不必过分关注细节（除了基本的事情，比如在将用户输入传递到 SQL 语句之前清理用户输入），但另一方面，我们隐含地相信供应商已经堵住了所有的漏洞。

如果发现一个广泛使用的服务器端脚本平台，比如 PHP，包含安全漏洞，这可能会很快成为公开的知识，运行易受攻击版本的框架的每个站点都会面临攻击。

对于 Node 来说，服务器端的安全性几乎完全取决于我们自己。因此，我们只需要教育自己关于潜在的漏洞，并加固我们的系统和代码。

在大多数情况下，Node 是最简约的：如果我们没有明确地概述某些事情，它就不会发生。这使得我们的系统或模糊的配置设置的未知部分很难被利用，因为我们是通过手工编码和配置我们的系统。

攻击发生在两个方面：利用技术缺陷和利用用户的天真。我们可以通过教育自己并认真检查和反复检查我们的代码来保护我们的系统。我们也可以通过教育用户来保护我们的用户。

在本章中，我们将学习如何实施各种类型的用户身份验证登录，如何保护这些登录，并加密任何传输的数据，以及一种防止经过身份验证的用户成为浏览器安全模型漏洞的受害者的技术。

# 实施基本身份验证

基本身份验证标准自上世纪 90 年代以来一直存在，并且可以是提供用户登录的最简单方式。当在 HTTP 上使用时，它绝对不安全，因为明文密码会从浏览器发送到服务器的连接中。

### 注意

有关基本身份验证的信息，请参见[`en.wikipedia.org/wiki/Basic_authentication`](http://en.wikipedia.org/wiki/Basic_authentication)。

然而，当与 SSL（HTTPS）结合使用时，基本身份验证可以是一种有用的方法，如果我们不关心自定义样式的登录表单。

### 注意

我们将在本章的*设置 HTTPS Web 服务器*配方中讨论 SSL/TLS（HTTPS）。有关更多信息，请参见[`en.wikipedia.org/wiki/SSL/TLS`](http://en.wikipedia.org/wiki/SSL/TLS)。

在这个配方中，我们将学习如何在纯 HTTP 上启动和处理基本访问身份验证请求。在接下来的配方中，我们将实施一个 HTTPS 服务器，并看到基本身份验证（摘要身份验证）的进展。

## 准备工作

我们只需要在一个新的文件夹中创建一个新的`server.js`文件。

## 如何做...

基本身份验证指定用户名、密码和领域，并且它在 HTTP 上工作。因此，我们将需要 HTTP 模块，并设置一些变量：

```js
var http = require('http');

var username = 'dave',
  password = 'ILikeBrie_33',
  realm = "Node Cookbook";

```

现在我们将设置我们的 HTTP 服务器：

```js
http.createServer(function (req, res) {
  var auth, login;

  if (!req.headers.authorization) {
    authenticate(res);
    return;
  }

  //extract base64 encoded username:password string from client
  auth = req.headers.authorization.replace(/^Basic /, '');
  //decode base64 to utf8
  auth = (new Buffer(auth, 'base64').toString('utf8'));

  login = auth.split(':'); //[0] is username [1] is password

  if (login[0] === username && login[1] === password) {
    res.end('Someone likes soft cheese!');
    return;
  }

  authenticate(res);

}).listen(8080);

```

请注意，我们对名为`authenticate`的函数进行了两次调用。我们需要创建这个函数，并将其放在我们的`createServer`调用之上：

```js
function authenticate(res) {
  res.writeHead(401,
     {'WWW-Authenticate' : 'Basic realm="' + realm + '"'});
  res.end('Authorization required.');
}

```

当我们在浏览器中导航到`localhost:8080`时，我们被要求为`Node Cookbook`领域提供用户名和密码。如果我们提供了正确的详细信息，我们对软奶酪的热情就会显露出来。

## 它是如何工作的...

基本身份验证通过服务器和浏览器之间发送的一系列标头进行工作。当浏览器访问服务器时，`WWW-Authenticate`标头被发送到浏览器，浏览器会打开一个对话框让用户登录。

浏览器的登录对话框会阻止加载其他内容，直到用户取消或尝试登录。如果用户按下**取消**按钮，他们会看到`authenticate`函数中使用`res.end`发送的**需要授权**消息。

然而，如果用户尝试登录，浏览器会向服务器发送另一个请求。这次响应`WWW-Authenticate`头部包含一个`Authorization`头部。我们在`createServer`回调的顶部检查它是否存在，使用`req.headers.authorization`。如果头部存在，我们跳过对`authenticate`的调用，继续验证用户凭据。`Authorization`头部如下所示：

```js
Authorization: Basic ZGF2ZTpJTGlrZUJyaWVfMzM=

```

在`Basic`后面的文本是一个 Base64 编码的字符串，其中包含用冒号分隔的用户名和密码，因此解码后的 Base64 文本是：

```js
dave:ILikeBrie_33

```

在我们的`createServer`回调中，我们首先从中剥离`Basic`部分，然后将其加载到一个将 Base64 转换为二进制的缓冲区中，然后对结果运行`toString`，将其转换为 UTF8 字符串。

有关 Base64 和 UTF-8 等字符串编码的信息，请参阅[`en.wikipedia.org/wiki/Base64`](http://en.wikipedia.org/wiki/Base64)和[`en.wikipedia.org/wiki/Comparison_of_Unicode_encodings`](http://en.wikipedia.org/wiki/Comparison_of_Unicode_encodings)。

最后，我们用冒号`split`登录详细信息，如果提供的用户名和密码与我们存储的凭据匹配，用户将获得对授权内容的访问权限。

## 还有更多...

基本身份验证作为中间件捆绑在 Express 框架中。

### 使用 Express 进行基本身份验证

Express（通过 Connect）提供了`basicAuth`中间件，它为我们实现了这种模式。要在 Express 中实现相同的功能：

```js
var express = require('express');

var username = 'dave',
  password = 'ILikeBrie_33',
  realm = "Node Cookbook";

var app = express.createServer();

app.use(express.basicAuth(function (user, pass) {
  return username === user && password === pass;
}, realm));

app.get('/:route?', function (req, res) {
  res.end('Somebody likes soft cheese!');
});

app.listen(8080);

```

如果我们现在转到`http://localhost:8080`，我们的 Express 服务器将与我们的主要示例表现出相同的行为。

### 提示

有关使用 Express 开发 Web 解决方案的信息，请参阅第六章*使用 Express 加速开发*。

## 另请参阅

+   *在第一章中讨论了*设置路由*，制作 Web 服务器

+   本章讨论了*实现摘要身份验证*。

+   本章讨论了*设置 HTTPS Web 服务器*

# 密码加密哈希

有效的加密是在线安全的基本部分。Node 提供了`crypto`模块，我们可以使用它来为用户密码生成我们自己的 MD5 或 SHA1 哈希。诸如 MD5 和 SHA1 之类的加密哈希被称为消息摘要。一旦输入数据被摘要（加密），它就不能被放回其原始形式（当然，如果我们知道原始密码，我们可以重新生成哈希并将其与我们存储的哈希进行比较）。

我们可以使用哈希来加密用户的密码，然后再存储它们。如果我们存储的密码被攻击者窃取，他们无法用来登录，因为攻击者没有实际的明文密码。然而，由于哈希算法总是产生相同的结果，攻击者可能通过将其与从密码字典生成的哈希进行匹配来破解哈希（有关缓解此问题的方法，请参见*还有更多...*部分）。

### 注意

有关哈希的更多信息，请参阅[`en.wikipedia.org/wiki/Cryptographic_hash_function`](http://en.wikipedia.org/wiki/Cryptographic_hash_function)。

在这个例子中，我们将创建一个简单的注册表单，并使用`crypto`模块来生成通过用户输入获得的密码的 MD5 哈希。

与基本身份验证一样，我们的注册表单应该通过 HTTPS 发布，否则密码将以明文形式发送。

## 准备工作

在一个新的文件夹中，让我们创建一个新的`server.js`文件，以及一个用于注册表单的 HTML 文件。我们将称其为`regform.html`。

我们将使用 Express 框架来提供外围机制（解析 POST 请求，提供`regform.html`等），因此应该安装 Express。我们在上一章中更详细地介绍了 Express 以及如何安装它。

## 如何做...

首先，让我们组合我们的注册表单（`regform.html`）：

```js
<form method=post>
  User  <input name=user>
  Pass <input type=password name=pass>
  <input type=submit>
</form>

```

对于`server.js`，我们将需要`express`和`crypto`。然后创建我们的服务器如下：

```js
var express = require('express');
var crypto = require('crypto');

var userStore = {},
  app = express.createServer().listen(8080);

app.use(express.bodyParser());

```

`bodyParser`给了我们 POST 的能力，我们的`userStore`对象用于存储注册用户的详细信息。在生产中，我们会使用数据库。

现在设置一个 GET 路由，如下所示的代码：

```js
app.get('/', function (req, res) {
  res.sendfile('regform.html');
});

```

这使用 Express 的`sendfile`方法来流式传输我们的`regform.html`文件。

最后，我们的 POST 路由将检查`user`和`pass`输入的存在，将用户指定的密码转换为 MD5 哈希值。

```js
app.post('/', function (req, res) {
  if (req.body && req.body.user && req.body.pass) {  
    var hash = crypto
      .createHash("md5")
      .update(req.body.pass)
      .digest('hex');

    userStore[req.body.user] = hash;
    res.send('Thanks for registering ' + req.body.user);
    console.log(userStore);
  }
});

```

当我们使用我们的表单进行注册时，控制台将输出`userStore`对象，其中包含所有注册的用户名和密码哈希。

## 它是如何工作的...

这个配方的密码哈希部分是：

```js
    var hash = crypto
      .createHash("md5")
      .update(req.body.pass)
      .digest('hex');

```

我们使用点符号将一些`crypto`方法链接在一起。

首先，我们使用`createHash`创建一个普通的 MD5 哈希（请参阅*还有更多...*部分，了解如何创建唯一的哈希）。我们也可以通过传递`sha1`作为参数来创建（更强大的）SHA1 哈希。对于 Node 捆绑的`openssl`版本（截至 Node 0.6.17 为止）支持的任何其他加密方法也是一样的。

有关不同哈希函数的比较，请参见[`ehash.iaik.tugraz.at/wiki/The_Hash_Function_Zoo`](http://ehash.iaik.tugraz.at/wiki/The_Hash_Function_Zoo)。

### 注意

该网站将某些哈希函数标记为破解，这意味着已经发现并发布了一个弱点。然而，利用这种弱点所需的工作量通常远远超过我们正在保护的数据的价值。

然后我们调用`update`来将用户的密码传递给初始哈希。

最后，我们调用`digest`方法，它返回一个完成的密码哈希。没有任何参数，`digest`会以二进制格式返回哈希值。我们传递`hex`（二进制数据的十六进制表示格式，参见[`en.wikipedia.org/wiki/Hexadecimal)`](http://en.wikipedia.org/wiki/Hexadecimal)）以使其在控制台上更易读。

## 还有更多...

`crypto`模块提供了一些更高级的哈希方法，用于创建更强大的密码。

### 使用 HMAC 使哈希值唯一化

**HMAC**代表**基于哈希的消息认证码**。这是一个带有秘密密钥的哈希（认证码）。

要将我们的配方转换为使用 HMAC，我们将我们的`crypto`部分更改为：

```js
    var hash = crypto
        .createHmac("md5",'SuperSecretKey')
        .update(req.body.pass)
        .digest('hex');

```

使用 HMAC 可以保护我们免受彩虹表（从大量可能的密码列表中预先计算的哈希）的影响。秘密密钥会改变我们的哈希值，使得彩虹表无效（除非攻击者发现我们的秘密密钥，例如，通过某种方式获得了对我们服务器操作系统的根访问权限，在这种情况下，彩虹表将不再是必要的）。

### 使用 PBKDF2 进行加固的哈希值

**PBKDF2**是基于密码的密钥派生函数的第二个版本，它是基于密码的加密标准的一部分。

PBKDF2 的一个强大特性是它生成数千次的哈希值。多次迭代哈希值可以增强加密，指数级增加了可能结果的数量，使得生成或存储所有可能的哈希值所需的硬件变得不可行。

`pbkdf2`需要四个组件：所需的密码，盐值，所需的迭代次数和生成哈希值的指定长度。

盐在概念上类似于 HMAC 中的秘密密钥，因为它与我们的哈希混合在一起，创建了一个不同的哈希。然而，盐的目的不同。盐只是为哈希添加了独特性，并不需要像秘密一样受到保护。一个强大的方法是使每个盐对生成的哈希是唯一的，并将其存储在哈希旁边。如果数据库中的每个哈希都是从不同的盐生成的，攻击者就被迫为每个哈希基于其盐生成一个彩虹表，而不是整个数据库。有了 PBKDF2，由于我们的盐，我们有了唯一哈希的唯一哈希，这为潜在攻击者增加了更多的复杂性。

对于强盐，我们将使用`crypto`的`randomBytes`方法生成 128 字节的随机数据，然后通过`pbkdf2`方法将其与用户提供的密码迭代 7,000 次，最终创建一个 256 字节长的哈希。

为了实现这一点，让我们修改配方中的 POST 路由。

```js
app.post('/', function (req, res) {
  if (req.body && req.body.user && req.body.pass) {
    crypto.randomBytes(128, function (err, salt) {
      if (err) { throw err;}
      salt = new Buffer(salt).toString('hex');
      crypto.pbkdf2(req.body.pass, salt, 7000, 256, function (err, hash) {
        if (err) { throw err; }
        userStore[req.body.user] = {salt : salt,
          hash : (new Buffer(hash).toString('hex')) };
        res.send('Thanks for registering ' + req.body.user);
        console.log(userStore);
      });
    });
  }
});

```

`randomBytes`和`pbkdf2`是异步的，这很有帮助，因为它允许我们执行其他任务，或者通过立即将用户带到新页面来改善用户体验，而他们的凭据正在被加密。这是通过简单地将`res.send`放在回调之外来完成的（我们在这里没有这样做，但这可能是一个好主意，因为这样大量的加密可能需要大约一秒钟来计算）。

一旦我们有了哈希和盐值，我们将它们放入我们的`userStore`对象中。要实现相应的登录，我们只需以相同的方式使用用户存储的盐来计算哈希。

我们选择迭代 7,000 次。当 PBKDF2 被标准化时，推荐的迭代次数是 1,000。然而，我们需要更多的迭代来考虑技术进步和设备成本的降低。

## 参见

+   在本章中讨论的*实现摘要身份验证*

+   *设置 HTTPS Web 服务器*在本章中讨论

+   *生成 Express 脚手架*在第六章中讨论，使用 Express 加速开发

# 实现摘要身份验证

**摘要身份验证**将基本身份验证与 MD5 加密结合在一起，从而避免传输明文密码，在普通 HTTP 上实现了更安全的登录方法。

单独使用摘要身份验证仍然不安全，没有 SSL/TLS 安全的 HTTPS 连接。在普通 HTTP 上的任何内容都容易受到中间人攻击的威胁，对手可以拦截请求并伪造响应。攻击者可以冒充服务器，用基本身份验证响应替换预期的摘要响应，从而获得明文密码。

尽管在没有 SSL/TLS 的情况下，摘要身份验证至少为我们提供了一些在明文密码方面需要更高级规避技术的防御。

因此，在这个配方中，我们将创建一个摘要身份验证服务器。

## 准备工作

首先，我们只需创建一个新的文件夹和一个新的`server.js`文件。

## 如何做到...

就像基本身份验证配方中我们创建了一个 HTTP 服务器一样，我们还将使用`crypto`模块来处理 MD5 哈希：

```js
var http = require('http');
var crypto = require('crypto');

var username = 'dave',
  password = 'digestthis!',
  realm = "Node Cookbook",
  opaque;

function md5(msg) {
  return crypto.createHash('md5').update(msg).digest('hex');
}

opaque = md5(realm);

```

我们创建了一个`md5`函数作为`crypto`哈希方法的简写接口。`opaque`变量是`Digest`标准的必要部分。它只是`realm`的 MD5 哈希（也用于基本身份验证）。客户端将 opaque 值返回给服务器，以提供额外的响应验证手段。

现在我们将创建两个额外的辅助函数，一个用于身份验证，另一个用于解析`Authorization`头，如下所示：

```js
function authenticate(res) {
  res.writeHead(401, {'WWW-Authenticate' : 'Digest realm="' + realm + '"'
    + ',qop="auth",nonce="' + Math.random() + '"'
    + ',opaque="' + opaque + '"'});

  res.end('Authorization required.');
}

function parseAuth(auth) {
  var authObj = {};
  auth.split(', ').forEach(function (pair) {
    pair = pair.split('=');
    authObj[pair[0]] = pair[1].replace(/"/g, '');
  });
  return authObj;
}

```

最后，我们按照以下代码实现服务器：

```js
http.createServer(function (req, res) {
  var auth, user, digest = {};

  if (!req.headers.authorization) {
    authenticate(res);
    return;
  }
  auth = req.headers.authorization.replace(/^Digest /, '');
  auth = parseAuth(auth); //object containing digest headers from client
  //don't waste resources generating MD5 if username is wrong
  if (auth.username !== username) { authenticate(res); return; }

  digest.ha1 = md5(auth.username + ':' + realm + ':' + password);
  digest.ha2 = md5(req.method + ':' + auth.uri);
  digest.response = md5([
    digest.ha1,
    auth.nonce, auth.nc, auth.cnonce, auth.qop,
    digest.ha2
  ].join(':'));

  if (auth.response !== digest.response) { authenticate(res); return; }
  res.end('You made it!');

}).listen(8080);

```

在浏览器中，这看起来与基本身份验证完全相同，这是不幸的，因为摘要和基本对话框之间的明显区别可能会提醒用户可能发生攻击。

## 它是如何工作的...

当服务器向浏览器发送`WWW-Authenticate`头时，包括几个属性，包括：`realm, qop, nonce`和`opaque`。

`realm`与基本身份验证相同，`opaque`是`realm`的 MD5 哈希。

`qop`代表 Quality of Protection，设置为`auth. qop`也可以设置为`auth-int`或者简单地省略。通过将其设置为`auth`，我们使浏览器计算出更安全的最终 MD5 哈希。`auth-int`更加强大，但浏览器对其的支持很少。

`nonce`类似于盐的概念，它使最终的 MD5 哈希在攻击者的视角下更不可预测。

当用户通过浏览器的身份验证对话框提交其登录详细信息时，将返回一个包含服务器发送的所有属性以及`username, uri, nc, cnonce`和`response`属性的`Authorization`头。

`username`是用户指定的别名，`uri`是正在访问的路径（我们可以使用它来基于路由进行安全保护），`nc`是一个序列计数器，每次认证尝试时都会递增，`cnonce`是浏览器自己生成的`nonce`值，`response`是最终计算出的哈希。

为了确认经过身份验证的用户，我们的服务器必须匹配`response`的值。为此，它删除`Authorization`头中的`Digest`字符串（包括前面的空格），然后将剩下的内容传递给`parseAuth`函数。`parseAuth`将所有属性转换为一个方便的对象，并将其加载回我们的`auth`变量中。

我们对`auth`的第一件事是检查用户名是否正确。如果没有匹配，我们会再次要求进行身份验证。这可以节省我们的服务器一些不必要的 MD5 哈希运算。

最终计算出的 MD5 哈希是由两个先前加密的 MD5 哈希以及服务器的`nonce`和`qop`值以及客户端的`cnonce`和`nc`值的组合生成的。

我们称第一个哈希为`digest.ha1`。它包含一个以冒号（:）分隔的字符串，其中包括`username, realm`和`password`的值。`digest.ha2`是`request`方法（GET）和`uri`属性，同样用冒号分隔。

`digest.response`属性必须与浏览器生成的`auth.response`匹配，因此排序和特定元素必须精确。为了创建我们的`digest.response`，我们将`digest.ha1`、`nonce, nc`、`cnonce, qop`和`digest.ha2`分别用冒号分隔，然后将这些值放入数组中，运行 JavaScript 的`join`方法生成最终字符串，然后传递给我们的`md5`函数。

如果给定的用户名和密码正确，并且我们正确生成了`digest.response`，它应该与浏览器的`response`头属性（`auth.response`）匹配。如果不匹配，用户将被要求进行另一次身份验证对话框。如果匹配，我们到达最终的`res.end`。我们成功了！

## 还有更多...

让我们解决注销问题。

### 退出经过身份验证的区域

在基本或摘要身份验证下，浏览器几乎没有任何官方注销方法的支持，除了关闭整个浏览器。

然而，我们可以通过更改`WWW-Authenticate`头中的`realm`属性来强制浏览器基本上失去其会话。

在多用户情况下，如果我们更改全局`realm`变量，将导致所有用户注销（如果有多个用户）。因此，如果用户希望注销，我们必须为他们分配一个唯一的领域，这将导致只有他们的会话退出。

为了模拟多个用户，我们将删除我们的`username`和`password`变量，并用一个`users`对象替换它们：

```js
var users = {
              'dave' : {password : 'digestthis!'},
              'bob' : {password : 'MyNamesBob:-D'},
            },
  realm = "Node Cookbook",
  opaque;

```

我们的子对象（当前包含`password`）可能会获得三个额外的属性：`uRealm, uOpaque`和`forceLogOut`。

接下来，我们将修改我们的`authenticate`函数如下：

```js
ffunction authenticate(res, username) {
  var uRealm = realm, uOpaque = opaque;
  if (username) {
    uRealm = users[username].uRealm;
    uOpaque = users[username].uOpaque;
  }
  res.writeHead(401, {'WWW-Authenticate' :
     'Digest realm="' + uRealm + '"'
    + ',qop="auth",nonce="' + Math.random() + '"'
    + ',opaque="' + uOpaque + '"'});

  res.end('Authorization required.');
}

```

我们向`authenticate`函数添加了一个可选的`username`参数。如果`username`存在，我们加载该用户的唯一`realm`和相应的`opaque`值，并将它们发送到标头中。

在我们的服务器回调中，我们替换了这段代码：

```js
  //don't waste resources generating MD5 if username is wrong
  if (auth.username !== username) { authenticate(res); return; }

```

使用以下代码：

```js
  //don't waste resources generating MD5 if username is wrong
  if (!users[auth.username]) { authenticate(res); return; }

  if (req.url === '/logout') {
    users[auth.username].uRealm = realm + ' [' + Math.random() + ']';
    users[auth.username].uOpaque = md5(users[auth.username].uRealm);
    users[auth.username].forceLogOut = true;
    res.writeHead(302, {'Location' : '/'});
    res.end();
    return;
  }

  if (users[auth.username].forceLogOut) {
    delete users[auth.username].forceLogOut;
    authenticate(res, auth.username);
  }

```

我们检查指定的用户名是否存在于我们的“用户”对象中，如果不存在，则无需进行进一步处理。只要用户有效，我们就检查路由（我们将为用户提供注销链接）。如果已命中`/logout`路由，我们会在已登录用户的对象上设置一个`uRealm`属性和一个包含`uRealm`的 MD5 哈希的相应`uOpaque`属性。我们还添加一个`forceLogOut`布尔属性，并将其设置为`true`。然后我们将用户从`/logout`重定向到`/`。

重定向触发另一个请求，服务器检测到当前经过身份验证的用户的`forceLogOut`属性的存在。然后，从“用户”子对象中删除`forceLogOut`，以防止它在以后造成干扰。最后，我们将控制权交给具有特殊`username`参数的`authenticate`函数。

因此，`authenticate`在`WWW-Authenticate`标头中包含了与用户关联的`uRealm`和`uOpaque`值，从而中断了会话。

最后，我们进行了一些微不足道的调整。

`digest.ha1`需要“密码”和`realm`值，因此更新如下：

```js
  digest.ha1 = md5(auth.username + ':'
    + (users[auth.username].uRealm || realm) + ':'
    + users[auth.username].password);

```

通过我们的新“用户”对象输入“密码”值，并根据我们已登录的用户是否设置了唯一领域（`uRealm`属性）来选择`realm`值。

我们将服务器代码的最后一部分更改为以下内容：

```js
if (auth.response !== digest.response) {
    users[auth.username].uRealm = realm + ' [' + Math.random() + ']';
    users[auth.username].uOpaque = md5(users[auth.username].uRealm);
    authenticate(res, (users[auth.username].uRealm && auth.username));
    return;
  }
  res.writeHead(200, {'Content-type':'text/html'});
  res.end('You made it! <br> <a href="logout"> [ logout ] </a>');

```

注意到包含了注销链接，这是最后一块拼图。

如果哈希不匹配，则会生成新的`uRealm`和`uOpaque`属性。这可以防止浏览器和服务器之间的永久循环。如果没有这个，当我们以有效用户身份登录然后注销时，我们将被呈现另一个登录对话框。如果我们输入一个不存在的用户，新的登录尝试将像正常情况下一样被服务器拒绝。然而，浏览器试图提供帮助，并回到我们第一个已登录用户和原始领域的旧身份验证详细信息。但是，当服务器接收到旧的登录详细信息时，它将用户与其唯一领域进行匹配，要求对`uRealm`而不是`realm`进行身份验证。浏览器看到`uRealm`值并将我们的不存在的用户与其匹配，尝试再次作为该用户进行身份验证，从而重复循环。

通过设置新的`uRealm`，我们打破了循环，因为引入了一个浏览器没有记录的额外领域，所以它会要求用户输入。

## 另请参阅

+   本章讨论了*实施基本身份验证*

+   本章讨论了*密码哈希加密*

+   本章讨论了*设置 HTTPS Web 服务器*

# 设置 HTTPS Web 服务器

在大多数情况下，HTTPS 是解决许多安全漏洞（如网络嗅探和中间人攻击）的解决方案。

感谢核心的`https`模块。设置起来非常简单。

## 准备就绪

更大的挑战可能在于实际获取必要的 SSL/TLS 证书。

为了获得证书，我们必须生成加密的私钥，然后从中生成证书签名请求。然后将其传递给证书颁发机构（一个专门受到浏览器供应商信任的商业实体——这自然意味着我们必须为此付费）。或者，CA 可以代表您生成您的私钥和证书签名请求。

### 注意

StartSSL 公司提供免费证书。关于如何在 Node 上使用 StartSSL 证书的文章可以在[`www.tootallnate.net/setting-up-free-ssl-on-your-node-server`](https://www.tootallnate.net/setting-up-free-ssl-on-your-node-server)找到。

经过验证过程后，证书颁发机构（CA）将颁发一个公共证书，使我们能够加密我们的连接。

我们可以简化这个过程并授权我们自己的证书（自签名），将自己命名为 CA。不幸的是，如果 CA 对浏览器不知情，它将明确警告用户我们的网站不值得信任，他们可能受到攻击。这对于积极的品牌形象并不好。因此，虽然在开发过程中我们可以自签名，但在生产中我们很可能需要一个受信任的 CA。

对于开发，我们可以快速使用`openssl`可执行文件（在 Linux 和 Mac OS X 上默认可用，我们可以从[`www.openssl.org/related/binaries.html`](http://www.openssl.org/related/binaries.html)获取 Windows 版本）来生成必要的私钥和公共证书：

```js
openssl req -new -newkey rsa:1024 -nodes -subj '/O=Node Cookbook' -keyout key.pem -out csr.pem && openssl x509 -req -in csr.pem -signkey key.pem -out cert.pem 

```

这在命令行上执行`openssl`两次：一次用于生成基本的私钥和证书签名请求，再次用于自签名私钥，从而生成证书（`cert.pem`）。

在实际生产场景中，我们的`-subj`标志将包含更多细节，我们还需要从合法的 CA 获取我们的`cert.pem`文件。但这对于私人、开发和测试目的来说是可以的。

现在我们有了密钥和证书，我们只需要创建一个新的`server.js`文件来启动我们的服务器。

## 如何做...

在`server.js`中，我们编写以下代码：

```js
var https = require('https');
var fs = require('fs');

var opts = {key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem')};

https.createServer(opts, function (req, res) {
  res.end('secured!');
}).listen(4443); //443 for prod

```

就是这样！

## 工作原理...

`https`模块依赖于`http`和`tls`模块，而这些模块又依赖于`net`和`crypto`模块。SSL/TLS 是传输层加密，这意味着它在 HTTP 的下层，在 TCP 层起作用。`tls`和`net`模块一起提供 SSL/TLS 加密的 TCP 连接，HTTPS 则在其之上。

当客户端通过 HTTPS 连接（在我们的情况下，地址为`https://localhost:4443`）时，它会尝试与我们的服务器进行 TLS/SSL 握手。`https`模块使用`tls`模块来响应握手，通过一系列的信息交换来确认浏览器和服务器之间的握手。（例如，你支持什么 SSL/TLS 版本？你想使用什么加密方法？我可以拿到你的公钥吗？）

在这个初始的交换结束时，客户端和服务器有了一个约定的共享密钥。这个密钥用于加密和解密双方之间发送的内容。这就是`crypto`模块发挥作用的地方，提供所有的数据加密和解密功能。

对我们来说，只需引入`https`模块，提供我们的证书，然后像使用`http`服务器一样使用它。

## 还有更多...

让我们看一些 HTTPS 使用案例。

### 在 Express 中使用 HTTPS

在 Express 中启用 HTTPS 同样简单：

```js
var express = require('express'),
  fs = require('fs');

var opts = {key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem')};

var app = express.createServer(opts).listen(8080);

app.get('/', function (req, res) {
  res.send('secured!');
});

```

### 使用 SSL/TLS 保护基本身份验证

我们可以在我们的`https`服务器中构建任何东西，就像在`http`服务器中一样。要在我们的基本身份验证配方中启用 HTTPS，我们只需修改：

```js
https.createServer(function (req, res) {

```

以下是：

```js
var opts = {key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem')};

https.createServer(opts, function (req, res) {

```

## 另请参阅

+   *本章讨论的密码哈希* 

+   *在本章讨论的基本身份验证中实施*

# 防止跨站点请求伪造

每个浏览器的安全模型都存在问题，作为开发者，我们必须意识到这一点。

当用户登录到网站后，通过经过身份验证的浏览器发出的任何请求都被视为合法的——即使这些请求的链接来自电子邮件，或者在另一个窗口中执行。一旦浏览器有了会话，所有窗口都可以访问该会话。

这意味着攻击者可以通过特制的链接或自动的 AJAX 调用来操纵用户在已登录的网站上的操作，而无需用户交互，只需在包含恶意 AJAX 的页面上。

例如，如果一个银行网站应用程序没有得到适当的 CSRF 保护，攻击者可以说服用户在登录到在线银行时访问另一个网站。然后，这个网站可以运行一个 POST 请求，将资金从受害者的账户转移到攻击者的账户，而受害者并不知情也没有同意。

这被称为**跨站点请求伪造（CSRF）**攻击。在这个 recipe 中，我们将实现一个带有 CSRF 保护的安全 HTML 登录系统。

## 准备工作

我们将从第六章中讨论的*制作一个 Express Web 应用程序*recipe 中保护我们的 Profiler Web 应用程序。我们将想要获取我们的`profiler`应用程序，打开`profiler/app.js`和`profiler/login/app.js`文件进行编辑。

没有 SSL/TLS 加密，基于 HTML 的登录至少会受到与基本授权相同的漏洞的影响。因此，为了基本安全，我们将在我们的应用程序中添加 HTTPS。因此，我们需要从上一个 recipe 中获取我们的`cert.pem`和`key.pem`文件。

我们还需要让 MongoDB 运行，并从第六章中的 recipes 中存储用户数据，因为我们的`profiler`应用程序依赖于它。

```js
sudo mongod --dbpath [PATH TO DB] 

```

## 如何做...

首先，让我们用 SSL 保护整个应用程序，`profiler/app.js`的顶部应该如下所示的代码：

```js
var express = require('express')
  , routes = require('./routes')
    , fs = require('fs');

var opts = {key: fs.readFileSync('key.pem'),
  cert: fs.readFileSync('cert.pem')};

var app = module.exports = express.createServer(opts);

```

`profiler`的`admin`部分是一个 CSRF 攻击可能发生的地方，所以让我们打开`profiler/login/app.js`并添加`express.csrf`中间件。`profiler/login/app.js`中`app.configure`回调的顶部应该如下所示的代码：

```js
app.configure(function(){
  app.set('views', __dirname + '/views');
  app.set('view engine', 'jade');

  app.use(express.bodyParser());
  app.use(express.methodOverride());

  app.use(express.cookieParser()); 
  app.use(express.session({secret: 'kooBkooCedoN'}));

  app.use(express.csrf());
//rest of configure callback

```

### 提示

**Express 3.x.x**

不要忘记，在 Express 3.x.x 中，秘密作为字符串进入`cookieParser`而不是作为对象进入`session: express.cookieParser('kooBkooCedoN')`。

`csrf`中间件依赖于`bodyParser`和`session`中间件，因此必须放在这些中间件之下。

现在，如果我们导航到`https://localhost:3000/admin`并尝试登录（dave, expressrocks），我们将收到一个`403 Forbidden`的响应，即使我们使用了正确的详细信息。

这是因为我们的登录应用程序现在在所有的 POST 表单中寻找一个名为`_csrf`的额外的 POST 参数，它必须与用户`session`对象中存储的`_csrf`值匹配。

我们的视图需要知道`_csrf`的值，这样它就可以被放置在我们的表单中作为一个隐藏元素。

我们将使用`dynamicHelper`为所有登录视图提供`req.session._csrf`。

```js
app.dynamicHelpers({ //Express 3.x.x would use app.locals.use instead
  user: function (req, res) { //see Chapter 6 for details. 
    return req.session.user;
  },
  flash: function (req, res) {
    return req.flash();
  },
  _csrf: function (req) {
    return req.session._csrf;
  }
});

```

接下来，我们将在`login/views`文件夹中创建一个名为`csrf.jade`的视图，如下所示：

```js
input(type="hidden", name="_csrf", value=_csrf);

```

现在我们在每个 POST 表单中包含`csrf.jade`。

`login.jade:`

```js
//prior login jade code above
if user
  form(method='post')
    input(name="_method", type="hidden", value="DELETE")
    include csrf
    p Hello #{user.name}!
      a(href='javascript:', onClick='forms[0].submit()') [logout]

  include admin
  include ../../views/profiles

else
  p Please log in
  form(method="post")
    include csrf
    fieldset
      legend Login
//rest of login.jade

```

`addfrm.jade:`

```js
fields = ['Name', 'Irc', 'Twitter', 'Github', 'Location', 'Description'];
form#addfrm(method='post', action='/admin/add')
  include csrf
  fieldset
    legend Add
//rest of addfrm.jade

```

### 提示

更新和维护一个有许多不同 POST 表单的网站可能会带来挑战。我们将不得不手动修改每一个表单。看看我们如何在*还有更多...*部分为所有表单自动生成 CSRF 值。

现在我们可以登录，添加配置文件，并且不会收到`403 Forbidden`的响应。

然而，我们的`/del`路由仍然容易受到 CSRF 的攻击。GET 请求通常不应该触发服务器上的任何更改。它们只是用来检索信息。然而，像许多其他应用程序一样，开发人员（也就是我们）在构建这个特定功能时懒惰，并决定强迫 GET 请求来执行他们的命令。

我们可以将这个转换为一个 POST 请求，然后用 CSRF 进行保护，但是对于一个有数百个这种异常 GET 请求的应用程序呢？

让我们找出如何保护我们的`/del`路由。在`login/routes/index.js`中添加以下代码：

```js
exports.delprof = function (req, res) {
  if (req.query._csrf !== req.session._csrf) {
    res.send(403);
    return;
  };
    profiles.del(req.query.id, function (err) {
      if (err) { console.log(err); }
        profiles.pull(req.query.p, function (err, profiles) {
          req.app.helpers({profiles: profiles});
          res.redirect(req.header('Referrer') || '/');
        });
    });

}

```

我们的更改使得我们不能删除配置文件，直到我们在查询字符串中包含`_csrf`，所以在`views/admin.jade:`中：

```js
mixin del(id)
  td
    a.del(href='/admin/del?id=#{id}&p=#{page}&_csrf=#{_csrf}') 
      &#10754;
//rest of admin.jade

```

## 工作原理...

`csrf`中间件生成一个唯一的令牌，保存在用户的会话中。这个令牌必须包含在任何操作请求（登录、登出、添加或删除）中，作为名为`_csrf`的属性。

如果请求体中（或 GET 的查询字符串中）的`_csrf`值与`session`对象中存储的`_csrf`令牌不匹配，服务器将拒绝访问该路由，从而阻止操作的发生。

这如何防止 CSRF 攻击？在普通的 CSRF 攻击中，攻击者无法知道`_csrf`值是多少，因此他们无法伪造必要的 POST 请求。

我们的`/del`路由保护不够安全。它在地址中暴露了`_csrf`值，可能为攻击者抓取`_csrf`值创造了一个非常小但仍然可信的机会。这就是为什么最好是我们坚持使用 POST/DELETE/PUT 请求来处理所有与操作相关的努力，将 GET 请求留给简单的检索。

### 跨站脚本（XSS）规避

在伴随的 XSS 攻击事件中，这种保护变得无效，攻击者能够在网站中植入自己的 JavaScript（例如，通过利用输入漏洞）。JavaScript 可以读取页面上的任何元素，并查看`document.cookie`中的非 HttpOnly cookie。

## 还有更多...

我们将研究一种自动生成登录表单 CSRF 令牌的方法，但我们也应该记住 CSRF 保护只有我们编写严密的能力才能做到。

### 自动保护带有 CSRF 元素的 POST 表单

确保我们应用程序中的所有 POST 表单都包含一个隐藏的`_csrf`输入元素可能是一个艰巨的任务。

我们可以直接与一些 Jade 内部交互，自动包含这些元素。

首先，在`login/app.js`中，我们向配置中添加了以下设置：

```js
  app.set('view options', {compiler: require('./customJadeCompiler')});

```

Express 允许我们将特定选项推送到我们正在使用的视图引擎。Jade 选项之一（我们的视图引擎）是`compile`，它允许我们定义我们自己的自定义 Jade 解释器。

让我们创建`customJadeCompiler.js`并将其放在`login`目录中。

首先，我们需要一些模块并设置我们的新编译器类如下：

```js
var jade = require('jade');
var util = require('util');

//inherit from Jades Compiler
var CompileWithCsrf = function (node, options) {
  jade.Compiler.call(this, node, options);
};

```

接下来，我们使用`util.inherits`从 Jades 编译器继承我们新编译器的原型。

```js
//inherit from the prototype
util.inherits(CompileWithCsrf, jade.Compiler);

```

然后我们修改 Jade 的内部`visitTag`方法（我们从`jade.Compiler`那里继承的）：

```js
CompileWithCsrf.prototype.visitTag = function (tag) {

   if (tag.name === 'form' && tag.getAttribute('method').match(/post/i)) {

    var csrfInput = new jade.nodes.Tag('input')
      .setAttribute('type', '"hidden"')
      .setAttribute('name', '"csrf"')
      .setAttribute('value', '_csrf');

    tag.block.push(csrfInput);

  }
  jade.Compiler.prototype.visitTag.call(this, tag);
};

```

最后，我们将我们的新编译器加载到`module.exports`中，以便通过`require`传递给`app.js`中`view options`设置的`compiler`选项：

```js
module.exports = CompileWithCsrf;

```

我们创建了一个新的类类型函数，将`call`方法应用于`jade.Compiler`。当我们将`this`对象传递给`call`方法时，我们实质上将`jade.Compiler`的主要功能继承到我们自己的`CompileWithCsrf`类类型函数中。这是重复使用代码的好方法。

然而，`jade.Compiler`还有一个修改过的原型，必须合并到我们的`CompileWithCsrf`中，以便完全模仿`jade.Compiler`。

我们使用了`util.inherits`，但我们也可以这样说：

```js
CompileWithCsrf.prototype = new jade.Compiler();

```

或者：

```js
CompileWithCsrf = Object.create(jade.Compiler);

```

甚至可以说：

```js
CompileWithCsrf.prototype.__proto__ = jade.Compiler.prototype;

```

`Object.create`是 Ecmascript5 的方法，`new`是旧的方法，`__proto__`是应该避免使用的非标准方法。它们都继承了`jade.Compiler`的附加方法和属性。但是，`util.inherits`是首选的，因为它还添加了一个特殊的`super`属性，其中包含我们最初继承的对象。

使用`call`和`util.inherits`本质上允许我们将`jade.Compiler`对象克隆为`CompileWithCsrf`，这意味着我们可以修改它而不影响`jade.Compiler`，然后允许它代替`jade.Compiler`运行。

我们修改`visitTag`方法，该方法处理 Jade 视图中的每个标签（例如，`p，div`等）。然后，我们使用正则表达式寻找`method`属性设置为`post`的`form`标签，因为`method`属性可能是大写或小写，用双引号或单引号括起来。

如果我们找到一个使用 POST 格式的`form`，我们使用`jade.Nodes`构造函数创建一个新的输入`node`（在这种情况下作为 HTML 元素），然后调用`setAttribute`（一个内部的 Jade 方法）三次来设置`type, name`和`value`字段。注意`name`设置为`'"_csrf"'`但`value`包含`'_csrf'`。内部的双引号告诉 Jade 我们打算使用一个字符串。没有它们，它会将第二个参数视为一个变量，这正是我们在`value`的情况下想要的。因此，`value`属性根据`app.js`中定义的`_csrf dynamicHelper`（同样是从`express.csrf`中间件生成的`req.session._csrf`中获取）进行渲染。

现在我们的`_csrf`令牌已经自动包含在每个 POST 表单中，我们可以从`login.jade`和`addfrm.jade`视图中删除`csrf.jade`的包含。

### 消除跨站脚本（XSS）漏洞

跨站脚本攻击通常是可以预防的，我们所要做的就是确保任何用户输入都经过验证和编码。棘手的部分在于我们在不正确或不充分地对用户输入进行编码的地方。

当我们接受用户输入时，大部分时间我们会在稍后的阶段将其输出到浏览器中，这意味着我们必须将其嵌入到我们的 HTML 中。

XSS 攻击主要是破坏上下文。例如，想象一下，我们有一些 Jade 代码，通过用户名链接到用户个人资料：

```js
a (href=username) !{username}

```

这段代码有两种可利用的方式。首先，我们在锚链接元素的文本部分使用了`!{username}`而不是`#{username}`。在 Jade 中，`#{}插值`会转义给定变量中的任何 HTML。因此，如果攻击者能够插入：

```js
<script>alert('This is where you get hacked!')</script>

```

作为他们的用户名，`#{username}`会渲染为：

```js
&lt;script&gt;alert('This is where you get hacked!')&lt;/script&gt;

```

而`!{username}`将不会被转义（例如，HTML 不会被转换为转义字符，如`&lt;`代替`<`）。攻击代码可以从一个无辜（尽管活泼）的`alert`消息改变为成功发起的伪造请求，而我们的 CSRF 保护将是徒劳的，因为攻击是从同一个页面操作的（JavaScript 可以访问页面上的所有数据，攻击者通过 XSS 获得了对我们页面的 JavaScript 的访问权限）。

Jade 默认对 HTML 进行转义，这是一件好事。然而，适当的转义必须具有上下文意识，简单地将 HTML 语法转换为相应的实体代码是不够的。

我们糟糕的 Jade 代码中的另一个易受攻击的区域是`href`属性。属性是一个与简单嵌套 HTML 不同的上下文。未引用的属性特别容易受到攻击，例如，考虑以下代码：

```js
<a href=profile>some text</a>

```

如果我们将`profile`设置为`profile onClick=javascript:alert('gotcha')`，我们的 HTML 将会显示：

```js
<a href=profile onClick=javascript:alert('gotcha')>some text</a>

```

同样，Jade 在这方面部分保护了我们，通过自动引用插入到属性中的变量。然而，我们的易受攻击的属性是`href`属性，这是 URL 类型的另一个子上下文。由于它没有任何前缀，攻击者可能会将他们的用户名输入为`javascript:alert('oh oh!')`，因此输出为：

```js
a (href=username) !{username}

```

将是：

```js
<a href="javascript:alert('oh oh!')"> javascript:alert('oh oh!') </a>

```

`javascript:`协议允许我们在链接级别执行 JavaScript，当一个无意中的用户点击一个恶意链接时，可以发起 CSRF 攻击。

### 注意

这些简单的例子是基础的。XSS 攻击可能会更加复杂和复杂。然而，我们可以遵循 Open Web Application Security Projects 8 输入消毒规则，这些规则提供了对 XSS 的广泛保护：

[`www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet`](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet)

### 提示

**验证器模块**

一旦我们了解了如何清理用户输入，我们就可以使用正则表达式快速应用特定的验证和净化方法。然而，为了简化生活，我们也可以使用第三方的`validator`模块，该模块可以通过`npm`安装。文档可在 Github 页面上找到：[`www.github.com/chriso/node-validator`](https://www.github.com/chriso/node-validator)。

## 另请参阅

+   *在本章讨论的设置 HTTPS Web 服务器*

+   *初始化和使用会话*在第六章中讨论，使用 Express 加速开发

+   *创建 Express Web 应用程序*在第六章中讨论，使用 Express 加速开发
