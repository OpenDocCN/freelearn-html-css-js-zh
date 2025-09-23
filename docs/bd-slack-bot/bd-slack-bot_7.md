# 第七章。发布您的应用

到目前为止，您已经拥有了构建一个能够提高您工作效率并改善团队间沟通的机器人的所有知识。希望到现在为止，您已经想到了一个机器人的想法，它不仅能让您的生活更轻松，也可能对他人有用。在本章中，您将学习如何使您的机器人对您自己的团队以外的用户和整个 Slack 社区开放。

我们将介绍将您的机器人添加到 Slack 应用目录并使其对他人可访问所需的步骤。我们将回顾以下步骤，以将您的机器人添加到 Slack 应用目录：

+   注册您的机器人和获取令牌

+   理解 OAuth 流程

+   配置“添加到 Slack”按钮

+   范围

+   将您的应用或机器人提交到应用目录

+   通过您的机器人盈利

# Slack 应用目录

为了让用户更容易添加应用，Slack 创建了应用目录([`slack.com/apps`](https://slack.com/apps))。这是一个购买应用和机器人以添加到您的 Slack 团队的地方。与其他应用商店一样，提交到应用目录的每个应用都受到控制，并且必须由 Slack 本身批准，以防止垃圾邮件和滥用。

如前一章所示，其他团队可以通过 webhooks 使用您的机器人。然而，如果您试图触及广泛的受众并可能通过您的机器人盈利，应用目录是最有效的方式。

![Slack 应用目录](img/B05384_07_01.jpg)

应用目录使添加新应用变得简单

本章的最终目标是允许用户通过点击**添加到 Slack**按钮将机器人添加到他们的 Slack 团队，我们将在稍后详细说明。

让我们从注册一个应用开始。在这个例子中，我们将添加*Wikibot*机器人，这是我们第三章“增加复杂性”中构建的。

### 注意

请注意，我们注册 Wikibot（以及使用维基百科 API）仅用于演示目的。在使用第三方 API 为打算发布的机器人之前，请始终检查其条款和条件。例如，对于 Wikibot，我们可以使用维基百科 API，但不允许发布名为“维基百科机器人”的机器人，因为我们不拥有该商标。

# 注册您的应用并获取令牌

为了成功通过 Slack 的 OAuth 服务器进行身份验证，需要某些独特的令牌。这是必要的，以便 Slack 可以确定我们是否是我们所说的我们，以及我们的应用或机器人是否实际上与我们试图获取访问权限的团队集成。

我们首先导航到 Slack 新应用注册页面[`api.slack.com/applications/new`](https://api.slack.com/applications/new)。通过选择机器人的名称、来源团队、机器人的描述、帮助页面链接和重定向 URI 来填写表格：

![注册您的应用和获取令牌](img/B05384_07_02.jpg)

在填写此表格时，尽可能详细。

保存您的设置后，您可以选择设置机器人用户、webhook 或 slash 命令。对于 Wikibot，我们将设置一个机器人用户。

![注册您的应用和获取令牌](img/B05384_07_03.jpg)

如果您指定的用户名已被占用，Slack 会稍作修改以避免冲突。

保存您的更改后，您应该在下一屏看到 OAuth 信息。首先，在继续之前，请确保保存此页面的**客户端 ID**和**客户端密钥**代码：

![注册您的应用和获取令牌](img/B05384_07_04.jpg)

永远不要与任何人分享您的客户端密钥。

### 注意

此过程不会使您的机器人对整个 Slack 用户群可见；它只是注册您开发应用的意图。您将通过 OAuth 过程测试您的应用。我们将在稍后的部分介绍如何将您的机器人提交到应用目录。

# 理解 OAuth 过程

为了在不是我们自己的团队中实现机器人用户，我们需要一个类似于我们之前为我们的团队创建的机器人令牌。我们可以请求此令牌，但首先我们必须使用 OAuth 过程证明我们是所说的那个人。**OAuth**（**开放认证**）是一个由许多公司使用的开放标准认证，无论大小。

认证过程通过以下步骤进行：

1.  用户点击**添加到 Slack**按钮。

1.  Slack 会将请求发送到我们在应用设置页面提供的重定向 URI。

1.  一旦我们的服务器收到请求，我们将将其重定向到授权 API 端点（[`slack.com/oauth/authorize`](https://slack.com/oauth/authorize)），并在查询字符串中包含以下参数：

    +   `client_id`：这是我们首次创建应用时给出的唯一 ID。

    +   `scope`：这包括我们应用所需的权限。我们将在本章稍后详细介绍权限。

    +   `redirect_uri`：这是一个可选参数。这是 Slack 将发送授权结果的 URI。如果留空，则使用应用设置页面中指定的`redirect_uri`。

    +   `State`：这是我们创建的字符串；它可以包含我们希望保留的数据或作为我们自己的识别方法。例如，我们可以用只有我们知道的秘密短语填充此字段，我们可以在以后使用它来确保此请求来自受信任的来源。

    +   `Team`：这是我们希望将我们的应用程序限制到的 Slack 团队 ID。这在调试我们的集成时很有用。

1.  Slack 会向我们在之前请求中提供的重定向 URI 发送一个 HTTP GET 请求。如果未提供，则默认使用我们在应用设置页面提供的 URI。请求包含以下参数：

    +   `code`：这是 Slack 生成的临时代码，用于确认我们的身份。

    +   `state`：这是我们之前创建的字符串，可以用来确保此请求是合法的。

1.  凭借我们需要的所有工具和代码，我们通过另一个 HTTP GET 请求向 Slack 请求机器人用户令牌，传递以下参数：

    +   `client_id`: 这是在应用设置页面给我们的唯一客户端 ID

    +   `client_secret`: 这是在应用设置页面给我们的唯一且秘密的 ID

    +   `code`: 这是第 4 步请求给我们的代码

    +   `redirect_uri`: 如果之前发送了`redirect_`uri，则此必须匹配；否则，它是可选的

1.  最后，如果一切顺利，我们将从 Slack 收到包含我们所需所有数据的响应。它看起来可能像这样：

    ```js
    { 
      ok: true,
      access_token: 'xoxp-xxxxxxxxxxx-xxxxxxxxxxx-xxxxxxxxxxx-xxxxxxxxxx',
      scope: 'identify,bot',
      user_id: 'Uxxxxxxxx',
      team_name: 'Building Bots',
      team_id: 'Txxxxxxxx',
      bot: { 
        bot_user_id: 'U136YALCW',
        bot_access_token: 'xoxb-xxxxxxxxxxx-xxxxxxxxxxxxxxxxxxxxxxxx' 
      }
    }
    ```

为了让这个过程更容易理解，让我们看看这些交易的图表：

![理解 OAuth 流程](img/B05384_07_05.jpg)

Slack 的 OAuth 授权流程

现在，让我们看看前面的代码示例。为了使我们的生活更加轻松，我们将使用 Express 网络框架([`expressjs.com/`](http://expressjs.com/))和熟悉的`superagent` AJAX 库。请确保使用以下命令安装它们：

```js
npm install –save express superagent

```

接下来，让我们构建我们的服务器；创建或重用`index.js`文件，并粘贴以下代码：

```js
const request = require('superagent');
const express = require('express');

const app = express();

const CLIENT_ID = 'YOUR_CLIENT_ID';
const CLIENT_SECRET = 'YOUR_CLIENT_SECRET';

app.get('/', (req, res) => {
 res.redirect(`https://slack.com/oauth/authorize?client_id=${CLIENT_ID}&scope=bot&redirect_uri=${escape('http://YOUR_REDIRECT_URI/bot')}`);
});

app.get('/bot', (req, res) => {
  let code = req.query.code;

  request
 .get(`https://slack.com/api/oauth.access?client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}&code=${code}&redirect_uri=${escape('http://YOUR_REDIRECT_URI/bot')}`)
    .end((err, res) => {
      if (err) throw err;
      let botToken = res.body.bot.bot_access_token;
      console.log('Got the token:', botToken);
    });

  res.send('received');
});

app.listen(8080, () => {
  console.log('listening');
});
```

突出的区域表示您应该填写自己的令牌和 URI。

### 注意

强烈建议使用`ngrok`等服务，以便您的本地服务器可以从互联网上访问。访问[`ngrok.com/`](https://ngrok.com/)获取更多详情和设置说明。您应仅将`ngrok`用于开发目的。在生产环境中，您应使用专用服务器。

导航到 Slack 按钮文档页面([`api.slack.com/docs/slack-button#button-widget`](https://api.slack.com/docs/slack-button#button-widget))并向下滚动，直到您看到以下测试界面：

![理解 OAuth 流程](img/B05384_07_06.jpg)

您可以使用这个区域来测试您的集成是否正确认证

点击**添加到 Slack**按钮，您应该会看到一个屏幕，要求您确认是否希望授权您的机器人在您的频道中使用。点击**授权**按钮，然后切换到您的终端。所需的机器人令牌将显示在日志中：

```js
listening
Got the token: xoxb-37236360438-xxxxxxxxxxxxxxxxxxxxxxxx

```

我们可以使用我们的令牌启动我们的机器人用户，并使其能够响应和与其他团队的用户互动。现在让我们用 Wikibot 来做这件事。我们将使用本书前面介绍过的 Wikibot 代码，并修改它以与之前概述的 OAuth 流程一起工作。将`index.js`的内容替换为以下内容：

```js
'use strict';

const Bot = require('./Bot');

const wikiAPI = 'https://en.wikipedia.org/w/api.php?format=json&action=query&prop=extracts&exintro=&explaintext=&titles=';
const wikiURL = 'https://en.wikipedia.org/wiki/';

const request = require('superagent');
const express = require('express');

const app = express();

const CLIENT_ID = 'YOUR_CLIENT_ID';
const CLIENT_SECRET = 'YOUR_CLIENT_SECRET';

app.get('/', (req, res) => {
 res.redirect(`https://slack.com/oauth/authorize?client_id=${CLIENT_ID}&scope=bot&redirect_uri=${escape('http://[YOUR_REDIRECT_URI]/bot')}`);
});

app.get('/bot', (req, res) => {
  let code = req.query.code;

  request
 .get(`https://slack.com/api/oauth.access?client_id=${CLIENT_ID}&client_secret=${CLIENT_SECRET}&code=${code}&redirect_uri=${escape('http://[YOUR_REDIRECT_URI]bot')}`)
    .end((err, result) => {
      if (err) {
        console.log(err);
        return res.send('An error occured! Please try again later');
      }
      console.log(res.body);

      let botToken = result.body.bot.bot_access_token;
      console.log('Got the token:', botToken);

      startWikibot(result.body.bot.bot_access_token);

      res.send('You have successfully installed Wikibot! You can now start using it in your Slack team, but make sure to invite the bot to your channel first with the /invite command!');
    });
});

app.listen(8080, () => {
  console.log('listening');
});

function startWikibot(token) {
  const bot = new Bot({
    token: token,
    autoReconnect: true,
    autoMark: true
  });

  // The rest of the familiar Wikibot code follows.
  // Visit https://github.com/PaulAsjes/BuildingBots for the 
  // complete source code
}
```

让我们试试这个。在确保您的 `client_id`、`client_secret` 和 `redirect_uri` 已插入先前代码中突出显示的部分后，运行 Node 应用程序。为了测试集成，导航到此处关于“添加到 Slack”按钮的文档：[`api.slack.com/docs/slack-button#button-widget`](https://api.slack.com/docs/slack-button#button-widget)。和之前一样，向下滚动直到您看到测试小部件，勾选“机器人”框，然后点击“添加到 Slack”按钮。

### 小贴士

在此测试小部件下方是您在网站上放置“添加到 Slack”按钮时应使用的嵌入代码。

![理解 OAuth 流程](img/B05384_07_07.jpg)

注意 Slack 如何自动将我们的机器人重命名为 @wikibot2 以避免冲突

一旦授权，你应该会看到以下信息：

**您已成功安装 Wikibot！现在您可以在您的 Slack 团队中使用它，但请确保首先使用 /invite 命令将机器人邀请到您的频道！**

在这个例子中，我们返回了一个简单的字符串。根据最佳实践，我们需要重定向到一个网页，上面有一些关于如何操作 Wikibot 的说明。

切换到 Slack 客户端和您想集成 Wikibot 的频道。正如我们在 第二章 中讨论的，“您的第一个机器人”，机器人用户必须手动邀请到频道，所以让我们这样做并测试我们的机器人：

![理解 OAuth 流程](img/B05384_07_08.jpg)

我们的机器人已成功集成并正在运行！

Wikibot 将在 Node 服务运行期间继续正常工作。

接下来，我们将查看我们可用的其他作用域。

# 作用域

OAuth 作用域允许您指定您的应用程序执行其功能所需的确切访问权限。在先前的例子中，我们请求了 `bot` 作用域，这使我们的机器人能够访问机器人用户可以执行的所有操作。例如，`channels:history` 作用域使我们能够访问频道的聊天历史，而 `users:read` 允许我们访问团队中用户的完整列表。作用域有很多（您可以在 [`api.slack.com/docs/oauth-scopes`](https://api.slack.com/docs/oauth-scopes) 上查看），但我们将重点关注我们应用程序中最可能使用的三个作用域：

+   `bot`：这提供了一个机器人令牌，允许我们以机器人用户身份连接到团队

+   `incoming-webhook`：这提供了一个传入 Webhook 令牌

+   `commands`：这提供了一个 Slack 令牌，我们可以使用它来确保传入的斜杠命令请求是有效的

### 注意

机器人类型的作用域自动包括机器人执行所需的其他子集作用域。更多信息，请访问 [`api.slack.com/bot-users#bot-methods`](https://api.slack.com/bot-users#bot-methods)。

可以无问题地请求多个作用域。以下是我们初始重定向中请求的机器人、传入 Webhook 和命令作用域的示例：

```js
app.get('/', (req, res) => {
  res.redirect(`https://slack.com/oauth/authorize?client_id=${CLIENT_ID}&scope=bot+incoming-webhook+commands&redirect_uri=${escape('http://YOUR_REDIRECT_URI/bot')}`);
});
```

注意请求的作用域是如何用`+`符号分隔的。认证后，这将返回以下对象：

```js
{ 
  ok: true,
  access_token: 'xoxp-xxxxxxxxxxx-xxxxxxxxxxx-xxxxxxxxxxx-xxxxxxxxxx',
  scope: 'identify,bot,commands,incoming-webhook',
  user_id: 'Uxxxxxxxx',
  team_name: 'Building Bots',
  team_id: 'Txxxxxxxx',
  incoming_webhook:
   { channel: '#bot-test',
     channel_id: 'Cxxxxxxxx',
     configuration_url: 'https://buildingbots.slack.com/services/xxxxxxxxx',
     url: 'https://hooks.slack.com/services/Txxxxxxxx/Bxxxxxxxx/xxxxxxxxxxxxxxxxxxxxxxxx' },
  bot:
   { 
     bot_user_id: 'Uxxxxxxxx',
     bot_access_token: 'xoxb-xxxxxxxxxxx-xxxxxxxxxxxxxxxxxxxxxxxx'        
   }
}
```

### 注意

除了使用`+`符号外，作用域也可以用逗号分隔。

我们现在已经拥有了创建机器人所需的所有组件（`bot_access`令牌）、一个入站 webhook（`incoming_webhook`对象中的`url`参数）以及用于斜杠命令的`access_token`。

# 将您的应用提交到应用目录

一旦您在团队频道内测试了您的集成，并且对您的机器人感到满意，就是时候将其提交到应用目录了。为此，首先确保您的应用程序符合 Slack 部署应用清单的要求（[`api.slack.com/docs/slack-apps-checklist`](https://api.slack.com/docs/slack-apps-checklist)）。简而言之，您的应用必须：

+   只请求实际使用的那些作用域。

+   在网页上显示**添加到 Slack**按钮。您需要拥有自己的网站，为新用户提供说明和帮助。

+   有一个合适的名称（例如，没有商标或版权侵权）。

+   有一个清晰且独特的应用或机器人图标。

+   拥有一个至少 512 x 512 像素大小的优质图标。

+   包含您机器人操作的简短和详细描述。

+   包含一个安装链接（这可以是一个显示**添加到 Slack**按钮和使用指南的网页）。

+   提供客户支持链接和电子邮件，以防用户在安装您的机器人时遇到问题。

+   包含一个隐私政策的链接。您的机器人可能会监听私密对话，因此您需要明确说明您的机器人将收集哪些数据（如果有的话）。

+   确保格式和拼写正确。您的机器人应该使用清晰的语言，并且不包含任何错误。

注意，我们的示例，Wikibot，未能通过**合适的名称**条款，因为维基百科显然是一个注册商标，我们并不拥有其权利。仅基于这一点，Wikibot 将被拒绝。

一旦您确认您的应用或机器人符合前面的要求，您就可以在[`api.slack.com/submit`](https://api.slack.com/submit)提交您的应用程序进行审查。

就像其他应用商店一样，所有新提交都必须经过审查流程。审查期的长度高度取决于您应用的复杂性以及 Slack 录取团队需要处理的提交数量。

### 注意

当您准备好将您的应用发布到 Slack 应用目录时，您需要托管。快速让您的机器人上线的一个好方法是使用 Beep Boop [`beepboophq.com/`](https://beepboophq.com/)。这是一个付费服务，Beep Boop 会为您托管 Slack 机器人，这样您就可以专注于开发而不是基础设施。

为了确保你的机器人达到目标受众，考虑将其提交到有用的网站，如 Botwiki ([`botwiki.org`](http://botwiki.org))、botlist ([`botlist.co`](http://botlist.co)) 和 Product Hunt ([`www.producthunt.com`](http://www.producthunt.com))，以获得最大曝光。

# 机器人盈利

当然，机器人盈利是可选的，你如何盈利取决于你的机器人功能以及是否存在市场。请注意，如果你的目标是按一次性价格出售你的机器人，Slack 应用目录不支持货币转账。

应用指令中的所有应用都可以免费安装，但如何将你的用户群转换为付费客户则取决于你。

有多种方法可以实现这一点，没有单一的正确方式或 Slack-**认可的**方法。Zoho Expense 等公司采用的一种流行方法是基于用户的支付计划（[`www.zoho.com/us/expense/slack-integration/`](https://www.zoho.com/us/expense/slack-integration/)）。对于小型团队，这项服务是免费的，但一旦需要超过三个用户访问，就必须迁移到付费层。

这里的想法与我们所遇到的 API 类似，例如 Wolfram Alpha。这意味着采用分层方法，其中存在免费层（与调用次数或到期日期相关联），但如果需要更多请求，则付费层是可选的。

记住，在尝试为你的机器人盈利时，“先试后买”的销售策略在这里至关重要。如果用户不知道你的机器人如何工作以及它是否真正对他们有益，他们不太可能成为付费客户。考虑提供一个免费试用期或带有有限功能的免费层。

最重要的是，确保你有一个真正值得付费的产品。尽管我们第四章中的*待办事项*机器人（Chapter 4）和*使用数据*很有用，但不太可能有人会为这样一个简单的机器人付费，因为免费替代品很容易获得或轻易复制。

因此，你的机器人应该首先关注解决特定问题，其次才是盈利。

# 摘要

在本章中，你了解了如何通过 Slack 应用目录使你的应用可供其他团队使用。你了解了如何从 Slack 请求作用域，以确保你的应用具有执行操作的正确权限。最后，你学习了如何正确地使用 Slack 验证你的应用并获得使你的机器人、webhooks 和 slash 命令工作所需的令牌。

通过遵循本书中的课程，你已经获得了创建世界级 Slack 机器人所需的所有知识和工具。现在，取决于你如何创造机器人技术的下一个飞跃，并推动我们与机器人互动以解决问题和实现最佳效率的界限。

为了进一步激发你的灵感，你应该知道，聊天机器人和特别地 Slack 机器人正享受着前所未有的流行和认可爆炸。

在 2016 年微软 Build 开发者大会的开幕式上，微软 CEO 萨蒂亚·纳德拉预言了机器人的未来：

> *"机器人是新的应用程序。人与人之间的对话，人与数字助手之间的对话，人与机器人之间的对话，甚至数字助手与机器人之间的对话。这就是你将在未来几年看到的这个世界。"*

他的论点非常吸引人：机器人可能会取代应用程序，成为公司与客户之间沟通的主要来源。

Facebook 也看到了机器人的潜力。2016 年 4 月，他们宣布了为他们的 Messenger 平台开发的机器人，预计在未来几个月和几年内将看到巨大的活动。

虽然这本书专注于为 Slack 平台构建机器人，但技术、最佳实践和理论对所有机器人平台都是有效的。有了这些知识，你将拥有成为这个新机器人革命中合格开发者所需的一切。

开心编码！

## 进一步阅读

在这本书中，我们直接使用了 Node Slack 客户端来构建我们的机器人。通过 GitHub 跟踪这个包是保持最新功能和对 Slack 生态系统变化了解的最佳方式。然而，使用官方 Node Slack 客户端有其他替代方案。Botkit（[`github.com/howdyai/botkit`](https://github.com/howdyai/botkit)）是一个出色的包，旨在抽象出许多底层概念并简化机器人创建过程。Botkit 还支持为 Facebook Messenger 创建机器人，便于跨平台机器人开发。如果你希望尽快启动并运行你的机器人，考虑使用 Botkit。
