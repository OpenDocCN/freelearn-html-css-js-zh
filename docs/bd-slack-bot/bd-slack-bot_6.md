# 第六章. Webhooks 和 Slash Commands

到目前为止，我们创建的每个机器人都有两个共同特点：它们依赖于用户发出的命令，并且需要 Slack API 令牌。这在我们的机器人中非常有用，但如果我们想让一个机器人向 Slack 频道发送消息而不需要 API 令牌呢？再或者，如果我们想创建一个不需要 API 令牌与用户交互的机器人呢？GitHub Slack 集成就是一个例子，这是一个将特定仓库的 GitHub 活动发布到您选择的 Slack 频道的服务。

在本章中，我们将讨论如何使用 Webhooks 将数据传入和传出 Slack，以及如何创建用户可以在 Slack 中与之交互的 slash 命令。

我们将涵盖以下主题：

+   Webhooks

+   Incoming webhooks

+   Outgoing webhooks

+   Slash commands

+   频道内和临时响应

# Webhooks

Webhook 是一种通过 HTTP 方法修改或增强 Web 应用程序的方式。以前，我们在机器人中使用第三方 API 将数据传入和传出 Slack。然而，这并非唯一的方法。Webhooks 允许我们使用带有 JSON 有效负载的常规 HTTP 请求将消息从 Slack 发送到其他地方。使 Webhook 成为机器人的特性是它能够将消息发布到 Slack，就像它们是机器人用户一样。

这些 Webhook 可以分为传入 Webhook 和传出 Webhook，每种都有自己的目的和用途。

## Incoming webhooks

一个传入 Webhook 的例子是一个将信息从外部源转发到 Slack 频道的服务，而不需要明确请求。上述 GitHub Slack 集成就是一个例子：

![Incoming webhooks](img/B05384_06_01.jpg)

GitHub 集成发布关于我们感兴趣的仓库的消息

在前面的屏幕截图中，我们可以看到在团队监视的仓库上创建新分支后，消息是如何发送到 Slack 的。这些数据并非团队成员明确请求的，但它们是作为传入 Webhook 的结果自动发送到频道的。

其他流行的例子包括 Jenkins 集成，其中可以在 Slack 中监控基础设施更改（例如，如果 Jenkins 监视的服务器宕机，可以立即在相关的 Slack 频道发布警告消息）。

让我们从设置一个发送简单*Hello world*消息的传入 Webhook 开始：

1.  首先，导航到自定义集成 Slack 团队页面([`my.slack.com/apps/build/custom-integration`](https://my.slack.com/apps/build/custom-integration))。![Incoming webhooks](img/B05384_06_02.jpg)

    不同的自定义集成版本

1.  从列表中选择**Incoming WebHooks**，然后选择您希望 Webhook 应用发布消息到的频道：![Incoming webhooks](img/B05384_06_03.jpg)

    Webhook 应用将发布到您选择的频道

    自定义 webhook（即只为您的团队创建的 webhook）使用所选频道作为默认频道来发送消息。我们可以看到，可以使用相同的 webhook 向不同的频道发送消息。

1.  一旦您点击了**添加入站 WebHooks 集成**按钮，您将看到一个选项页面，允许您进一步自定义您的集成。![入站 webhook](img/B05384_06_04.jpg)

    名称、描述和图标都可以从这个菜单中设置

1.  为您的集成设置一个自定义图标（例如，本例中使用了`wave`表情符号）并记下 webhook URL，其格式如下：

    `https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX`

生成的 URL 对您的团队是唯一的，这意味着通过此 URL 发送的任何 JSON 有效负载都只会出现在您的团队 Slack 频道中。

现在，让我们在 Node 中快速测试我们的入站 webhook。启动一个新的 Node 项目（记住您可以使用`npm init`来创建`package.json`），然后在您的终端中运行以下命令来安装熟悉的`superagent` AJAX 库：

```js
npm install superagent –save

```

创建一个名为`index.js`的文件，并将以下 JavaScript 代码粘贴到其中：

```js
const WEBHOOK_URL = [YOUR_WEBHOOK_URL];

const request = require('superagent');

request
  .post(WEBHOOK_URL)
  .send({
    text: 'Hello! I am an incoming Webhook bot!'
  })
  .end((err, res) => {
    console.log(res);
  });
```

记住用您新生成的 URL 替换`[YOUR_WEBHOOK_URL]`，然后通过执行以下命令来运行程序：

```js
nodemon index.js

```

现在应该发生两件事：首先，在您的终端中记录一个长响应，其次，您应该在 Slack 客户端看到如下消息：

![入站 webhook](img/B05384_06_05.jpg)

入站 webhook 的“hello world”等效

我们在终端中记录的`res`对象是我们对 webhook URL 发出的 HTTP POST 请求的响应。它以大型 JavaScript 对象的形式出现，显示了我们对 webhook URL 发出的 HTTP POST 请求的信息。

观察在 Slack 客户端收到的消息，注意名称和图标是否与我们在团队管理员网站上设置的集成设置中的内容相同。请记住，如果没有提供，将使用默认图标、名称和频道，所以让我们看看当我们更改这些设置时会发生什么。将`index.js`中的`request` AJAX 调用替换为以下内容：

```js
request
  .post(WEBHOOK_URL)
  .send({
    username: "Incoming bot",
    channel: "#general",
    icon_emoji: ":+1:",
    text: 'Hello! I am different from the previous bot!'
  })
  .end((err, res) => {
    console.log(res);
  });
```

保存文件后，`nodemon`将自动重启程序。切换到 Slack 客户端，您应该在`#general`频道中看到如下消息：

![入站 webhook](img/B05384_06_06.jpg)

新名称、图标和消息

### 注意

在`icon_emoji`的位置，您也可以使用`icon_url`链接到您选择的特定图像。

如果您希望消息只发送给一个用户，可以将用户名作为`channel`属性的值：

```js
channel: "@paul"
```

这将导致消息从 Slackbot 直接消息中发送。消息的图标和用户名将与您在设置中配置的或设置在 POST 请求正文中的内容相匹配。

最后，让我们看看在我们的集成中发送链接；将`text`属性替换为以下内容，并保存`index.js`：

```js
text: 'Hello! Here is a fun link: <http://www.github.com|Github is great!>'
```

Slack 会自动解析它找到的任何链接，无论是`http://www.example.com`还是`www.example.com`的格式。通过将 URL 放在尖括号中并使用`|`字符，我们可以指定我们希望 URL 显示的内容：

![入站 webhooks](img/B05384_06_07.jpg)

格式化的链接比长 URL 更容易阅读

有关消息格式的更多信息，请访问[`api.slack.com/docs/formatting`](https://api.slack.com/docs/formatting)。

### 注意

注意，由于这是一个自定义 webhook 集成，我们可以更改集成的名称、图标和频道。如果我们将集成打包成 Slack 应用（其他团队可以安装的应用），则无法覆盖默认的频道、用户名和图标设置。

入站 webhook 由外部源触发——一个例子是当新用户注册到您的服务或产品被卖出时。入站 webhook 的目的是为您的团队提供易于访问和易于理解的信息。相反，如果您想从 Slack 中获取数据，这可以通过出站 webhook 的方式完成。

## 出站 webhooks

出站 webhook 与入站 webhook 不同，因为它们将数据从 Slack 发送到您选择的服务，然后该服务可以回复消息到 Slack 频道。

要设置出站 webhook，请再次访问您的 Slack 团队管理页面的自定义集成页面（[`my.slack.com/apps/build/custom-integration`](https://my.slack.com/apps/build/custom-integration)）。这次，请确保选择**Outgoing WebHooks**选项。

在下一屏幕上，请确保选择一个频道、一个名称和一个图标。注意有一个`target` URL 字段需要填写；我们很快就会填写它。

当 Slack 中的出站 webhook 被触发时，会向您提供的 URL（或 URLs，因为您可以指定多个）发送 HTTP POST 请求。因此，我们首先需要构建一个可以接受我们的 webhook 的服务器。

在`index.js`中，粘贴以下代码：

```js
'use strict';
const http = require('http');
// create a simple server with node's built in http module
http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});

  // get the data embedded in the POST request
  req.on('data', (chunk) => {
    // chunk is a buffer, so first convert it to 
    // a string and split it to make it more legible as an array
     console.log('Body:', chunk.toString().split('&'));
  });

  // create a response
  let response = JSON.stringify({
    text: 'Outgoing webhook received!'
  });

  // send the response to Slack as a message
  res.end(response);
}).listen(8080, '0.0.0.0');

console.log('Server running at http://0.0.0.0:8080/');
```

### 注意

注意我们要求使用`http`模块，尽管我们没有使用 NPM 安装它。这是因为`http`模块是 Node 的核心依赖项，并且会自动包含在您的 Node 安装中。

在此代码块中，我们在端口 8080 上启动一个简单的服务器，并监听传入的请求。

在这个例子中，我们将服务器设置为在`0.0.0.0`上运行，而不是`localhost`。这很重要，因为 Slack 正在向我们的服务器发送请求，因此它需要从互联网上可访问。将我们服务器的**互联网协议**（**IP**）设置为`0.0.0.0`告诉 Node 使用您的计算机的网络分配的 IP 地址。因此，通过将我们服务器的 IP 设置为`0.0.0.0`，Slack 可以通过在端口 8080 上击中您的 IP 地址（例如，`http://123.456.78.90:8080`）来访问您的服务器。

### 注意

如果你遇到 Slack 无法连接到你的服务器的问题，这很可能是由于你位于路由器或防火墙后面。为了解决这个问题，你可以使用像`ngrok`([`ngrok.com/`](https://ngrok.com/))这样的服务。或者，检查你的路由器或防火墙的**端口转发**设置。

让我们相应地更新我们的外部钩子设置：

![外部钩子](img/B05384_06_08.jpg)

带有目标 URL 的外部钩子设置

保存你的设置并运行你的 Node 应用；通过在钩子设置中指定的频道中输入消息来测试外部钩子是否工作。你应该在 Slack 中看到类似以下内容：

![外部钩子](img/B05384_06_09.jpg)

我们构建了一个垃圾邮件机器人

好消息是，我们的服务器正在接收请求并向 Slack 发送消息。这里的问题是我们在外部钩子设置页面中跳过了**触发词**字段。没有触发词，发送到指定频道的任何消息都会触发外部钩子。这导致我们的钩子最初由发送外部钩子的消息触发，从而形成一个无限循环。

为了解决这个问题，我们可以做两件事之一：

+   在监听所有频道消息时，请勿向频道返回消息

+   指定一个或多个触发词，以确保我们不会向频道发送垃圾邮件

返回消息是可选的，但为了确保更好的用户体验，鼓励这样做。即使是一条确认消息，例如**消息已接收**，也比没有消息要好，因为它向用户确认他们的消息已被接收并正在处理。

假设我们更喜欢第二个选项并添加一个触发词：

![外部钩子](img/B05384_06_10.jpg)

触发词帮助我们组织外部钩子

现在，让我们再次尝试，这次在消息开头发送带有触发词的消息。重新启动你的 Node 应用并发送一条新消息：

![外部钩子](img/B05384_06_11.jpg)

我们的外部钩子应用现在功能上与之前我们的机器人非常相似

太好了，现在切换到你的终端，看看那条消息记录了什么：

```js
Body: [ 'token=KJcfN8xakBegb5RReelRKJng',
  'team_id=T000001',
  'team_domain=buildingbots',
  'service_id=34210109492',
  'channel_id=C0J4E5SG6',
  'channel_name=bot-test',
  'timestamp=1460684994.000598',
  'user_id=U0HKKH1TR',
  'user_name=paul',
  'text=webhook+hi+bot%21',
  'trigger_word=webhook' ]
```

这个数组包含由 Slack 发送的 HTTP POST 请求的主体。其中包含一些有用的数据，例如用户的姓名、发送的消息和团队 ID。我们可以使用这些数据来自定义响应或执行一些验证，以确保用户有权使用此钩子。

在我们的响应中，我们只是发送了一个“**消息已接收**”字符串。然而，就像入站 webhooks 一样，我们可以设置自己的用户名和图标。频道不能与 webhook 设置中指定的频道不同。当 webhook 不是自定义集成时，同样适用这些限制。这意味着如果 webhook 作为另一个团队的 Slack 应用安装，则 webhook 只能以设置屏幕中指定的用户名和图标发布消息。我们将在第七章*发布您的应用*中详细讨论 Slack 应用。

需要注意的一个重要事项是，无论是入站还是出站的 webhooks，都只能在公共频道中设置。这主要是为了防止滥用并维护隐私，因为我们已经看到设置一个可以记录频道所有活动的 webhook 是极其简单的。

如果您想在私人组或 DM 中使用类似的功能，我们可以使用斜杠命令。

# 斜杠命令

以斜杠（`/`）开头的命令是可以在 Slack 客户端的任何地方使用的命令。你可能已经熟悉了 Slack 自己实现的更常见的那些命令。例如，使用`topic`命令：

```js
/topic Sloths are great
```

这将设置频道的主题为“树懒很棒。”就像处理入站和出站 webhooks 一样，Slack 允许团队配置他们自己的自定义斜杠命令。为了演示其用法，我们将构建一个使用流行的计算知识引擎 Wolfram Alpha ([`www.wolframalpha.com/`](http://www.wolframalpha.com/))的机器人。最终目标是创建一个机器人，它可以通过斜杠命令返回查询结果。

与 webhooks 不同，斜杠命令只能发送命令中包含的数据，因此你只能保证接收到的数据是故意发送的。正因为这个细微差别，我们使用斜杠命令还能获得额外的好处。它们可以在任何频道、DM 或私人组中使用。

首先，让我们设置斜杠命令集成并获取 Wolfram Alpha API 密钥。虽然我们不需要特定的 Slack 令牌，但我们确实需要它来访问 Wolfram Alpha 的服务。导航到您团队的集成设置([`buildingbots.slack.com/apps/manage/custom-integrations`](https://buildingbots.slack.com/apps/manage/custom-integrations))，选择**斜杠命令**，然后选择**添加配置**。我们将使用`wolfram`字符串作为我们的斜杠命令，所以让我们填写它并继续。

![斜杠命令](img/B05384_06_12.jpg)

斜杠命令必须对您的团队是唯一的

现在，指定斜杠命令将要发送请求的 URL，这与我们之前处理 webhooks 时所做的类似。

![斜杠命令](img/B05384_06_13.jpg)

斜杠命令可以以不同于 webhooks 的方式定制

我们可以选择在请求提供的 URL 时使用哪种 HTTP 方法。如果您希望向服务器发送数据，请使用**POST**方法。如果您希望检索数据而不发送任何内容，请使用**GET**方法。对于我们的 Wolfram Alpha 机器人，我们将使用**POST**，因为我们正在向之前创建的服务器发送查询。

特别注意生成的令牌。这是一个唯一的标识符，您可以使用它来确保所有发送到您服务器的请求都来自这个特定的 Slack 斜杠命令，从而允许您拒绝任何不想要的请求。我们稍后会回到令牌。

接下来，我们将填写自动完成详情。尽管这是可选的，但强烈建议您无论如何都填写它们，因为它们为用户提供清晰的说明，告诉他们如何使用您的斜杠命令。

![斜杠命令](img/B05384_06_14.jpg)

对于从未使用过您的命令的用户来说，帮助文本非常有帮助

与我们在本书中使用的其他第三方 API 类似，Wolfram Alpha API 需要 API 令牌才能访问他们的计算服务。要获取一个，请导航到以下 URL 并遵循屏幕上的注册说明：[`developer.wolframalpha.com/portal/apisignup.html`](https://developer.wolframalpha.com/portal/apisignup.html)。

### 注意

注意，Wolfram Alpha API 每月免费请求量只有 2000 次。如果您的斜杠命令超过这个数量，除非您支付更高级别的服务费用，否则您的请求将被拒绝。

Wolfram Alpha API 以 XML 格式发送响应，我们需要将其转换为 JSON 以便于使用。幸运的是，有一个 NPM 包可以为我们抽象这个问题：`node-wolfram` ([`www.npmjs.com/package/node-wolfram`](https://www.npmjs.com/package/node-wolfram))。通过运行以下命令安装`node-wolfram`包：

```js
npm install node-wolfram –save

```

一旦您有了密钥并且已经安装了`node-wolfram`，请将以下代码粘贴到`index.js`中：

```js
'use strict';

const http = require('http');
const request = require('superagent');

const WOLFRAM_TOKEN = [YOUR_WOLFRAM_API_TOKEN];
const SLACK_TOKEN = [YOUR_SLACK_TOKEN];

const Client = require('node-wolfram');
const wolfram = new Client(WOLFRAM_TOKEN);

// create a simple server with node's built in http module
http.createServer((req, res) => {
    res.writeHead(200, {'Content-Type': 'text/plain'});

    // get the data embedded in the POST request
    req.on('data', (chunk) => {
      // chunk is a buffer, so first convert it 
      // to a string and split it to make it legible
      console.log('Body:', chunk.toString().split('&'));

      let bodyArray = chunk.toString().split('&');
      let bodyObject = {};

      // convert the data array to an object
      for (let i = 0; i < bodyArray.length; i++) {
        // convert the strings into key value pairs
        let arr = bodyArray[i].split('=');
        bodyObject[arr[0]] = arr[1];
      }

      // if the token doesn't match ours, abort
      if (bodyObject.token !== SLACK_TOKEN) {
        return res.end('Invalid token');
      }

      queryWolfram(bodyObject.text.split('+').join(' '), (err, result) => {
        if (err) {
          console.log(err);
          return;
        }

        // send back the result to Slack
        res.end(result);
      });
    });
}).listen(8080, '0.0.0.0');

console.log('Server running at http://0.0.0.0:8080/');

// make sure to unescape the value so we don't get Unicode
let query = unescape(bodyObject.text.split('+').join(' '));

queryWolfram(query, (err, result) => {  wolfram.query(message, (err, result) => {
    if (err) {
      return done(err);
    }

    // if the query didn't fail, but the message wasn't understood 
    // then send a generic error message
    if (result.queryresult.$.success === 'false') {
      return done(null, 'Sorry, something went wrong, please try again');
    }
    let msg = '';

    for (let i = 0; i < result.queryresult.pod.length; i++) {
      let pod = result.queryresult.pod[i];
      msg += pod.$.title + ': \n';

      for (let j = 0; j < pod.subpod.length; j++) {
        let subpod = pod.subpod[j];

        for (let k = 0; k <subpod.plaintext.length; k++) {
          let text = subpod.plaintext[k];
          msg += '\t' + text + '\n';
        }
      }
    }

    done(null, msg);
  });
}
```

简而言之，这段代码监听 8080 端口的传入请求。一旦接收到数据（通过 POST 请求），我们将数据转换为 JavaScript 对象以便于使用。如果请求中发送的令牌与程序中硬编码的令牌匹配，我们将发送一个包含斜杠命令内容的请求到 Wolfram Alpha。幸运的是，Wolfram Alpha 运行自己的**自然语言处理**（**NLP**），因此我们可以直接发送用户的输入，让 Wolfram Alpha 完成繁重的工作。一旦我们从 Wolfram Alpha API 收到回调，我们将结果返回给 Slack，它会在 Slack 频道中发布。运行你的服务器，并在 Slack 中输入以下命令以查看其效果：

```js
/wolfram 2 x 2
```

几分钟后，您应该会看到结果：

![斜杠命令](img/B05384_06_15.jpg)

Wolfram Alpha 计算一个简单的数学问题

成功！现在让我们尝试一个更具挑战性的查询：

```js
/wolfram distance between earth and moon
```

那个请求应该会产生类似以下的结果：

![斜杠命令](img/B05384_06_16.jpg)

查询耗时过长

哎呀，看起来我们的查询超时了。如果我们给我们的应用程序添加一些日志记录，我们会看到虽然 Wolfram Alpha API 最终会返回结果，但它需要超过 Slack webhook 集成（3000 毫秒）的最大超时时间。这导致斜杠命令失败并显示前面的错误消息。

为了解决这个问题，让我们看看 Slack 初始接收到的数据；上一个斜杠命令的正文看起来像这样：

```js
Body: [ 'token=86oxKgPrkxrvPHpmleaP8Rbs',
  'team_id=T00000000',
  'team_domain=buildingbots',
  'channel_id=C0J4E5SG6',
  'channel_name=bot-test',
  'user_id=U0HKKH1TR',
  'user_name=paul',
  'command=%2Fwolfram',
  'text=distance+between+earth+and+moon',   'response_url=https%3A%2F%2Fhooks.slack.com%2Fcommands%2FT0HKKH1T9%2F35399194752%2Fm9mIVSHYjMdnwXWyCTYYTIZj' ]
```

我们感兴趣的是 `Body` 数组的最后一个索引——一个响应 URL。如果你的计算时间超过最大超时时间 3000 毫秒，Slack 会提供一个 URL，我们可以向其发送 POST HTTP 请求，就像我们发送 webhook 消息一样。

如果你的斜杠命令处理时间超过最大超时时间，并且你正在使用请求 URL，强烈建议你向 Slack 返回一条消息，告知用户他们的请求正在处理中。

将你的代码中的 `http.createServer` 块替换为以下内容，注意高亮区域：

```js
// create a simple server with node's built in http module
http.createServer((req, res) => {
    res.writeHead(200, {'Content-Type': 'text/plain'});

    // get the data embedded in the POST request
    req.on('data', (chunk) => {
      // chunk is a buffer, so first convert it to a string 
      // and split it to make it legible
      console.log('Body:', chunk.toString().split('&'));

      let bodyArray = chunk.toString().split('&');
      let bodyObject = {};

      // convert the data array to an object
      for (let i = 0; i < bodyArray.length; i++) {
        // convert the strings into key value pairs
        let arr = bodyArray[i].split('=');
        bodyObject[arr[0]] = arr[1];
      }

      // if the token doesn't match ours, abort
      if (bodyObject.token !== SLACK_TOKEN) {
        return res.end('Invalid token');
      }

 // send a message immediately to confirm that 
 // the request was receive it's possible that the 
 // query will take longer than the time Slack waits
 // for a response (3000ms), so we'll send a 
 // preliminary response and then send the results later
 res.end('Calculating response, be with you shortly!');

      // make sure to unescape the value so we don't get Unicode
      let query = unescape(bodyObject.text.split('+').join(' '));

      queryWolfram(query, (err, result) => {  wolfram.query(message, (err, result) => {
        if (err) {
          console.log(err);
          return;
        }

 // send the result from the wolfram alpha request,
 // which probably took longer than 3000ms to calculate
 request
 .post(unescape(bodyObject.response_url))
 .send({
 text: result
 })
 .end((err, res) => {
 if (err) console.log(err);
 });
      });
    });
}).listen(8080, '0.0.0.0');
```

在确认斜杠命令请求来自我们的团队之后，但在我们开始 Wolfram Alpha API 请求之前，我们向 Slack 频道返回一个确认消息，告知用户他们的请求正在进行中。

一旦 Wolfram Alpha 返回我们的数据，我们就向斜杠命令初始请求体中提供的响应 URL 发送 HTTP POST 请求。让我们再次尝试那个最后的命令：

```js
/wolfram distance between earth and moon
```

这应该返回一个确认消息：

![斜杠命令](img/B05384_06_17.jpg)

确认消息让用户知道事情正在进行

几秒钟后，我们应该看到斜杠命令查询的完整结果：

![斜杠命令](img/B05384_06_18.jpg)

我们的斜杠命令返回大量数据

在我们的斜杠命令按预期工作后，让我们看看返回输出的一个特性。

# 频道内和短暂响应

你可能已经注意到，当 Wolfram Alpha 机器人响应时，它的名字旁边有文本 **只有你能看到这条消息**。正如文本所暗示的，我们机器人的结果只对发起斜杠命令的用户可见。这是一个短暂响应的例子。请注意，原始斜杠命令的文本也只对执行它的用户可见。短暂响应的反面是频道内响应，它可以在频道中显示斜杠命令和结果，供所有人查看。

默认情况下，所有斜杠命令响应都由 Slack API 设置为短暂模式。让我们看看如何更改它，并发送频道内消息。再次，让我们替换 `http.createServer` 的内容。一步一步地查看更改：

```js
// create a simple server with node's built in http module
http.createServer((req, res) => {
    res.writeHead(200, {'Content-Type': 'application/json'});
```

这里的主要区别在于，我们将响应的头部内容类型更改为 `application/json`。这通知 Slack 预期一个字符串形式的 JSON 包。

代码如下：

```js
// get the data embedded in the POST request
req.on('data', (chunk) => {
     // chunk is a buffer, so first convert it to a string 
     // and split it to make it legible
  console.log('Body:', chunk.toString().split('&'));

  let bodyArray = chunk.toString().split('&');
  let bodyObject = {};

  // convert the data array to an object
  for (let i = 0; i < bodyArray.length; i++) {
    // convert the strings into key value pairs
    let arr = bodyArray[i].split('=');
    bodyObject[arr[0]] = arr[1];
  }

  // if the token doesn't match ours, abort
  if (bodyObject.token !== SLACK_TOKEN) {
 return res.end(JSON.stringify({
 response_type: 'ephemeral',
 text: 'Invalid token'
 }));
}
```

我们的错误响应现在需要以字符串化的 JSON 格式呈现。此外，我们添加了响应类型 `ephemeral`，这意味着错误消息只会对发起 slash 命令的用户可见：

```js
// send a message immediately to confirm that
// the request was receive it's possible that the
// query will take longer than the time Slack waits
// for a response (3000ms), so we'll send a
// preliminary response and then send the results later
res.end(JSON.stringify({
 response_type: 'in_channel',
 text: 'Calculating response, be with you shortly!'
}));
```

现在，我们特别想要一个 `in-channel` 响应。在这个上下文中，这意味着 slash 命令和处理的响应将对频道中的所有人可见：

![频道内和短暂响应](img/B05384_06_19.jpg)

原始的 slash 命令和中间响应都是可见的

最后，我们查询 **Wolfram|Alpha**：

```js
// make sure to unescape the value so we don't get Unicode
let query = unescape(bodyObject.text.split('+').join(' '));

queryWolfram(query, (err, result) => {
  if (err) {
    console.log(err);
    return;
  }

   // send the result from the wolfram alpha request,
   // which probably took longer than 3000ms to calculate
   request
     .post(unescape(bodyObject.response_url))
 .send({
 response_type: 'in_channel',
 text: result
 })
     .end((err, res) => {
       if (err) console.log(err);
     });
    });
  });
}).listen(8080, '0.0.0.0');
```

在这里，我们再次确保 Wolfram Alpha 的结果对整个频道可见。最后，让我们对我们的 `queryWolfram` 函数中数据的显示做一些改进：

```js
function queryWolfram(message, done) {
  wolfram.query(message, (err, result) => {
    if (err) {
      return done(err);
    }

    // if the query didn't fail, but the message wasn't understood
    // then send a generic error message
    if (result.queryresult.$.success === 'false') {
      return done(null, 'Sorry, something went wrong, please try again');
    }

 let msg = [];

    for (let i = 0; i < result.queryresult.pod.length; i++) {
      let pod = result.queryresult.pod[i];

      // print the title in bold
 msg.push(`*${pod.$.title}:*\n`);

      for (let j = 0; j < pod.subpod.length; j++) {
        let subpod = pod.subpod[j];

        for (let k = 0; k <subpod.plaintext.length; k++) {
          let text = subpod.plaintext[k];
 if (text) {
 // add a tab to the beginning
 msg.push('\t' + text + '\n');
 } else {
 // text is empty, so get rid of the title as well
 msg.pop();
 }
        }
      }
    }

    // join the msg array together into a string
    done(null, msg.join(''));
  });
}
```

这里的改进包括加粗章节标题和删除没有文本关联的章节。

现在我们已经把所有这些都整合在一起了，让我们来测试一下：

![频道内和短暂响应](img/B05384_06_20.jpg)

Wolfram Alpha 还可以用来获取流行算法的定义

请记住，slash 命令在您的 Slack 团队中是通用的。在我们的例子中，这意味着 **Wolfram|Alpha** 机器人可以从任何频道、DM 或私人组中触发。

# 使用 Webhooks 和 slash 命令

现在我们已经对 Webhooks 和 slash 命令有了明确的了解，我们应该确定何时使用它们。首先，我们应该考虑在什么情况下我们会使用 webhook 或 slash 命令而不是机器人用户，这是我们之前章节中学到的如何构建机器人用户。

机器人用户通常以一对一的方式进行操作；每个机器人都需要一个唯一的 Slack 令牌，这意味着机器人只能与该令牌关联的团队进行交互。这也允许机器人与 Slack 保持实时消息连接，并在连接失败的情况下重新连接。另一方面，Webhooks 和 slash 命令作为外部服务存在，可以被许多团队重复使用。通过消除对 Slack 令牌的需求，您可以让您的应用被许多其他团队使用。

使用此流程图来决定 webhook 或 slash 命令是否最适合您的需求：

![使用 Webhooks 和 slash 命令](img/B05384_06_21.jpg)

何时使用 Webhooks 或 slash 命令

在前面的图中，我们提到了 **主动** 和 **响应** 的概念。我们在 第三章 中讨论了这些概念，*增加复杂性*，但基本要点是，主动应用和机器人无需输入即可发布消息，而响应式机器人会以用户输入的形式对刺激做出反应。

# 摘要

在本章中，我们介绍了 Webhooks 是什么以及如何设置它们以将数据从 Slack 发送到第三方服务器，并通过第三方服务器将数据传入 Slack。我们还讨论了 slash 命令以及如何实现它们。

在下一章中，我们将介绍如何发布您的应用，以便其他团队可以使用您的机器人、Webhooks 和 slash 命令。
