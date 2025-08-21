# 第八章：集成网络范式

在本章中，我们将涵盖：

+   发送电子邮件

+   发送短信

+   与 TCP 通信

+   创建 SMTP 服务器

+   实施虚拟主机范式

# 介绍

Node 的能力远不止于简单地提供网页服务。Node 的核心重点是支持各种计算任务和网络目标，其简单易懂的界面使得像我们这样的开发人员能够释放创造力，创新日益相互连接的解决方案和想法。

在本章中，我们将以一些基本示例来看待这种相互连接的知识，我们知道我们可以将这些原型应用到更大的应用程序中。

了解如何实施网络范式可以帮助我们超越 Web 应用程序的正常界限，为我们的用户提供高级功能，并实现更多与我们服务连接的方式。

# 发送电子邮件

在许多平台上，发送电子邮件的能力是标准的，但 Node 的方法将电子邮件功能留给开发人员。

值得庆幸的是，Node 社区中有一些出色的模块创建者已经为发送电子邮件创建了模块。在本教程中，我们将使用功能齐全的第三方`nodemailer`模块向一组收件人发送虚构的通讯。

## 准备工作

为了发送电子邮件，我们需要一个可以连接的功能正常的 SMTP 服务器。在后面的教程中，我们将创建自己的 SMTP 服务器，但现在我们需要获取一些我们的 SMTP 的细节来使用我们的客户端。

如果我们有一个电子邮件地址，我们就可以访问 SMTP 服务器。我们可以从我们的提供商那里找到 SMTP 主机地址。

如果需要，我们可以通过注册 Gmail 帐户（在[mail.google.com)](http://mail.google.com)）来获得访问 SMTP 服务器的权限。一旦我们有了帐户，我们就可以使用`smtp.gmail.com`作为主机，使用我们的 Gmail 地址作为用户名。

我们将创建一个新的文件夹，其中包含一个名为`mailout.js`的新文件来保存我们的代码。

## 如何做...

使用`nodemailer`有三个主要元素。它们如下：

1.  设置 SMTP 传输

1.  组装消息对象（包括传输）

1.  将对象传递给`sendMail`方法。

让我们添加`nodemailer`模块，并按照以下代码创建传输：

```js
var nodemailer = require('nodemailer');

var transport = nodemailer.createTransport('SMTP', {
    host: 'smtp.gmail.com',
    secureConnection: true,
    port: 465,
    auth: {
      user: "ourGmailAddress@googlemail.com",
      pass: "ourPassword"
    }
  });

```

我们需要填写我们自己的 SMTP 设置，包括`user`和`pass`值。

我们已经使用了`secureConnection`设置，并将端口设置为`465`，因此我们可以使用 Gmail 的 SSL/TLS 安全的 SMTP 服务器。

现在我们将我们配置的传输合并到一个名为`msg`的对象中，如下所示：

```js
var msg = {
  transport: transport,
  text:    "Hello! This is your newsletter, :D",
  from:    "Definitely Not Spammers <spamnot@ok.com>",
  subject: "Your Newsletter"
};

```

请注意，我们还没有在对象上设置`to`属性。我们将要发送邮件到多个地址，所以`to`将动态设置。为了测试目的，我们将创建一个`mailinator`电子邮件地址的数组。**Mailinator** ([`www.mailinator.com`](http://www.mailinator.com))是一个免费服务，允许我们通过向虚构的地址发送电子邮件来快速创建临时电子邮件地址。

```js
var maillist = [
  'Mr One <mailtest1@mailinator.com>',
  'Mr Two <mailtest2@mailinator.com>',
  'Mr Three <mailtest3@mailinator.com>',
  'Mr Four <mailtest4@mailinator.com>',
  'Mr Five <mailtest5@mailinator.com>'
];

```

现在我们只需循环发送我们的通讯给每个地址，如下所示：

```js
var i = 0;
maillist.forEach(function (to) {
  msg.to = to;
  nodemailer.sendMail(msg, function (err) {

    if (err) { console.log('Sending to ' + to + ' failed: ' + err); }

    console.log('Sent to ' + to);

    i += 1
    if (i === maillist.length) { msg.transport.close();  }
  });
});

```

如果我们将浏览器指向[`mailtest1.mailinator.com`](http://mailtest1.mailinator.com)（或`mailtest2, mailtest3`等），我们应该在 Mailinator 的临时收件箱中看到我们的消息。

## 工作原理...

使用 Nodemailer 的`createTransport`方法，我们可以快速配置我们的应用程序所需的 SMTP 设置，然后将这些设置包含在`sendMail`方法使用的`msg`对象中。

我们没有设置初始的`to`属性，因为它在每次`maillist.forEach`迭代中被修改，然后传递到`sendMail`方法中。

`sendMail`是异步的，就像大多数带有回调的方法一样（`forEach`是一个例外）。每次调用`sendMail`后，`forEach`都会继续调用下一个`sendMail`，而不会等待`sendMail`调用完成。这意味着`forEach`循环将在所有`sendMail`调用完成之前结束。因此，为了知道所有邮件何时发送完毕，我们使用一个计数器（`i`）。

每次发送一封邮件，我们都会将我们的`i`变量增加`1`。一旦`i`等于我们的`maillist`数组的大小，所有邮件都已发送，所以我们调用`transport.close`。

Nodemailer 打开多个连接（连接池）到 SMTP 服务器，并重用这些连接发送所有的邮件。这确保了快速和高效的发送邮件，消除了为每封邮件打开和关闭连接的开销。`transport.close`关闭连接池，从而允许我们的应用程序完成执行。

## 还有更多...

Nodemailer 是一个功能齐全、高度可配置的邮件模块，我们将会看到。

### 使用 sendmail 作为替代传输

许多托管提供商都有一个`sendmail`服务，连接到默认的 SMTP 服务器，我们不需要知道其详细信息。如果我们简单地修改我们的`transport`对象为`sendmail`，Nodemailer 将与`sendmail`进行接口。

```js
var transport = nodemailer.createTransport("Sendmail");

```

如果`sendmail`不在我们主机服务器的环境`PATH`变量中（要找出来，只需在 SSH 提示符中键入`sendmail`），我们可以指定`sendmail`的位置如下：

```js
var transport = nodemailer.createTransport("Sendmail", "/to/sendmail");

```

### HTML 邮件

电子邮件可以包含 HTML，并在基本用户代理中优雅地降级为纯文本。要发送 HTML 电子邮件，我们只需将`html`属性添加到我们的`msg`对象中：

```js
var msg = {
//prior properties: transport
 text:    "Hello! This is your newsletter, :D",
 html: "<b>Hello!</b><p>This is your newsletter, :D</p>",
//following properties: from, subject
};

```

纯文本应该与 HTML 一起包含，为不支持 HTML 的电子邮件客户端提供备用。

如果我们不想单独编写文本部分，我们可以让 Nodemailer 从 HTML 中提取文本，使用`generateTextFromHtml`属性，如下面的代码所示：

```js
var msg = {
  transport: transport,
  html: "<b>Hello!</b><p>This is your newsletter, :D</p>",
  createTextFromHtml: true,
  from:    "Definitely Not Spammers <spamnot@ok.com>",
  subject: "Your Newsletter"
};

```

### 发送附件

如果我们想通过电子邮件附件讲一个非常糟糕的笑话，该怎么办？

我们将动态创建一个文本文件，并从磁盘加载一个图像文件，然后将它们附加到一封电子邮件中。

对于图片，我们将使用`deer.jpg`（可以在支持代码文件中找到）。这应该放在与我们的邮件文件相同的文件夹中（让我们称之为`mailout_attachments.js`）。

```js
var nodemailer = require('nodemailer');

var msg = {
  transport: nodemailer.createTransport('SMTP', {
    host: 'smtp.gmail.com',
    secureConnection: true,
    port: 465,
    auth: {
      user: "ourGmailAddress@googlemail.com",
      pass: "ourPassword"
    }
  }),
  text:    "Answer in the attachment",
  from:    "The Attacher <attached@files.com>",
  subject: "What do you call a deer with no eyes?",
  to: "anyemail@anyaddress.com",
  attachments: [
    {fileName: 'deer.txt', contents:'no eye deer.'},
    {fileName: 'deerWithEyes.jpg', filePath: 'deer.jpg'}
  ]
};

nodemailer.sendMail(msg, function (err) {
  if (err) { console.log('Sending to ' + msg.to + ' failed: ' + err); }
  console.log('Sent to ' + msg.to);
  msg.transport.close();
});

```

当然，这是一个附件的概念验证，并不是电子邮件的最佳用法。附件作为`msg`对象中的对象数组提供。每个附件对象必须有一个`fileName`属性，这是在电子邮件中附件的文件名。这不必与从磁盘加载的实际文件的名称匹配。

文件内容可以直接通过`contents`属性写入，使用字符串或`Buffer`对象，或者我们可以使用`filePath`从磁盘流式传输文件（我们也可以直接将流传递给`sourceStream`属性）。

## 另请参阅

+   *在本章中讨论的*发送短信

+   *在本章中讨论的*创建 SMTP 服务器

# 发送短信

能够向用户发送短信文本消息是我们与他们联系的另一种方式

我们可以将计算机连接到 GSM 调制解调器，与专门的库进行交互（如 Asterisk，`asterisk.org`，结合 ngSMS，`ozekisms.com`），并与库和电话设备进行接口，发送短信。

当然还有更简单的方法。像 Twilio 这样的服务提供网关短信服务，我们可以通过 HTTP REST API 与它们联系，它们会为我们处理短信发送。

在这个教程中，我们将使用`twilio`模块将我们的通讯通讯应用程序转换为一个通用的短信服务。

## 准备就绪

这需要一个 Twilio 账户（[`www.twilio.com/try-twilio`](https://www.twilio.com/try-twilio)）。一旦注册并登录，我们应该注意我们的账户 SID、Auth Token 和沙箱电话号码（我们可能需要选择我们的居住国家以获得适当的沙箱号码）。

我们需要一些电话号码用于测试发送短信。在沙盒模式下（这是我们在开发中将要使用的模式），我们想要发送短信或打电话的任何号码都必须经过验证过程。我们通过从**账户**部分选择**Numbers**链接，然后点击**Verify a Number**来完成这个过程。然后 Twilio 将拨打该号码，并期望在屏幕上输入提供的 PIN 以进行确认。

让我们创建一个新文件，`smsout.js`，并安装`twilio`助手模块：

```js
npm install twilio 

```

## 如何做...

首先，我们需要`twilio`并组合一些配置设置：

```js
var twilio = require('twilio');
var settings =  {
  sid : 'Ad054bz5be4se5dd211295c38446da2ffd',
  token: '3e0345293rhebt45r6erta89xc89v103',
  hostname : 'dummyhost',
  phonenumber: '+14155992671' //sandbox number
}

```

### 提示

**Twilio 电话号码**

在我们开始与 Twilio 服务进行交互之前，我们必须指定一个注册的 Twilio 电话号码，以创建我们的`phone`。在开发过程中，我们可以简单地使用沙盒号码，可以从 Twilio 仪表板中找到（[`www.twilio.com/user/account`](http://www.twilio.com/user/account)）。在生产环境中，我们需要升级我们的账户并从 Twilio 购买一个唯一的电话号码。

有了我们的`settings`正确，我们准备创建一个 Twilio 客户端，用它来初始化一个虚拟电话：

```js
var restClient = new (twilio.RestClient)(settings.sid, settings.token);

var client = new (twilio.Client)(settings.sid,
                                  settings.token,
                                  settings.hostname);

var phone = client.getPhoneNumber(settings.phonenumber);

```

我们在这里也创建了`restClient`，它提供了反映 Twilio 原始 REST API 的 API 调用。我们将使用`restClient`来检查我们的短信消息的状态，以确定消息是否已从 Twilio 发送到目标电话。

现在我们定义了一个要发送短信的号码列表（我们必须提供自己的号码），就像我们在上一个示例中的`maillist`数组一样：

```js
var smslist = [
  '+44770xxxxxx1',
  '+44770xxxxxx2',
  '+44770xxxxxx3',  
  '+44770xxxxxx4',  
  '+44770xxxxxx5'  
];

```

### 提示

除非我们升级了我们的账户，否则`smslist`上的任何号码都必须经过 Twilio 的预验证。这可以通过 Twilio Numbers 账户部分完成（[`www.twilio.com/user/account/phone-numbers/`](https://www.twilio.com/user/account/phone-numbers/)）。

然后，我们简单地循环遍历`smslist`并使用`phone`向每个接收者发送短信消息，如下所示：

```js
var msg = 'SMS Ahoy!';
smslist.forEach(function (to) {
  phone.sendSms(to, msg, {}, function(sms) { console.log(sms);  });
});

```

这应该可以正常工作，除了这个过程不会退出（因为`twilio`初始化了一个服务器来接收 Twilio 的回调），我们不知道 Twilio 何时向接收者发送短信。一种检查的方法是向 Twilio REST API 发出另一个请求，请求状态更新。`twilio RestClient`使我们可以轻松实现这一点，具体如下：

```js
  phone.sendSms(to, msg, {}, function(sms) {
      restClient.getSmsInstance(sms.smsDetails.sid, function (presentSms) {
		//process presentSms using it's status property.
      });
  });

```

如果我们的短信在第一次调用时没有发送，我们需要等待并再次检查。让我们进行一些最终的改进，如下面的代码所示：

```js
var msg = 'SMS Ahoy!',  i = 0;
smslist.forEach(function (to) {
  phone.sendSms(to, msg, {}, function (sms) {

  function checkStatus(smsInstance) {  
      restClient.getSmsInstance(smsInstance.sid, function (presentSms) {
       if (presentSms.status === 'sent') {  
          console.log('Sent to ' + presentSms.to);
        } else {
          if (isNaN(presentSms.status)) {
            //retry: if its not a number (like 404, 401), it's not an error
            setTimeout(checkStatus, 1000, presentSms);
            return;
          }
          //it seems to be a number, let's notify, but carry on
          console.log('API error: ', presentSms.message);
        }
        i += 1;
        if (i === smslist.length) { process.exit(); }
      });
    };

    checkStatus(sms.smsDetails);

  });
});

```

现在控制台将在每次确认发送号码后输出。当所有号码都已发送消息时，该过程退出。

## 它是如何工作的...

`twilio.RestClient`通过`twilio`助手为我们提供了对 Twitter API 的低级交互访问。这简单地为我们预设的授权设置包装了通用的 API 调用，代表我们进行 HTTP 请求。

`twilio.Client`是由`twilio`助手提供的更高级的 API，处理 Twilio 和客户端之间的双向交互。初始化新客户端时必须提供的第三个参数是`hostname`参数。`twilio`模块提供了这个参数给 Twilio，作为从 Twilio 服务器请求的回调 URL 的一部分，以确认短信消息是否已发送。

我们忽略了这种行为，并提供了一个虚拟的主机名，实现了我们自己的确认短信已发送的方法。我们的方法不需要我们拥有一个可以从 Web 访问的实时服务器地址（请参阅*还有更多..*部分，了解使用`hostname`属性作为预期的回调 URL 的实时服务器实现）。

我们使用我们创建的`phone`对象的`sendSms`方法通过`twilio`模块向 Twilio API 发出 HTTP 请求，传入所需的接收者、消息和回调函数。

一旦请求发出，我们的回调就会触发初始的`sms`对象。我们使用这个对象来确定 Twilio 给我们的`sendSMS`请求分配的 ID，即`smsInstance.sid`（即`sms.smsDetails.sid`）。

`smsInstance.sid`被传递给`restClient.getSmsInstance`，它为我们提供了我们的`smsIntance`对象的更新实例，我们称之为`presentSms`。这个调用是从一个名为`checkStatus`的自定义自调用函数表达式中进行的，它将我们的初始`sms.smsDetails`对象传递给它。

我们希望看到 Twilio 是否已经发送了我们的短信。如果是，`presentSms.status`将是`sent`。如果不是，我们希望等一会儿，然后要求 Twilio 再次更新我们排队的短信消息的状态。也就是说，除非返回的状态是`404`，在这种情况下出现了问题，我们需要通知用户，但继续处理下一个短信消息。

就像*发送电子邮件*的示例一样，我们会记录发送的消息数量。一旦它们总数达到收件人的数量，我们就会退出进程，因为`smsout.js`是一个命令行应用程序（在服务器场景中，我们不需要也不想这样做）。

## 还有更多...

Twilio 模块的多功能性不仅限于发送短信。它还可以通过发出事件透明地处理 Twilio 的回调。

### 使用 processed 事件监听器

在实时公共服务器上，最好向`twilio.Client`提供我们的主机名，以便它可以管理回调 URL 请求。

### 注意

为了使这段代码工作，它必须托管在一个实时公共服务器上。有关在实时服务器上托管 Node 的更多信息，请参阅第十章，携手共进

我们将删除`restClient`并将我们的`settings`对象更改为以下内容：

```js
var settings =  {
  sid : 'Ad054bz5be4se5dd211295c38446da2ffd',
  token: '3e0345293rhebt45r6erta89xc89v103',
  hostname : 'nodecookbook.com',
  phonenumber: '+14155992671' //sandbox number
};

```

然后我们的短信发送代码就是：

```js
var i = 0;
smslist.forEach(function (to) {
  phone.sendSms(to, msg, {}, function(sms) {
    sms.on('processed', function(req) {  
      i += 1;  
      console.log('Message to ' + req.To +
      ' processed with status: ' + req.SmsStatus);
      if (i === smslist.length) {process.exit();}
    });
  });
});

```

`twilio`客户端提供了一个`statusCallback` URL 给 Twilio。Twilio 将请求这个 URL（类似`http://nodecookbook.com:31337/autoprovision/0`）来确认，`twilio`辅助模块将发出一个`processed`事件来通知我们 Twilio 的确认。我们监听这个事件，通过`req`对象检查给定的`SmsStatus`来确认成功。

### 进行自动电话呼叫

为了使下一个示例工作，我们需要一个有效的主机名，并且在一个公开的服务器上运行我们的应用程序，就像在前一节中一样。

### 注意

为了使这段代码工作，它必须托管在一个实时公共服务器上。有关在实时服务器上托管 Node 的更多信息，请参阅第十章，携手共进

要进行呼叫，我们从通常的设置开始。

```js
var twilio = require('twilio');

var settings =  {
  sid : 'Ad054bz5be4se5dd211295c38446da2ffd',
  token: '3e0345293rhebt45r6erta89xc89v103',
  hostname : 'nodecookbook.com',
  phonenumber: '+14155992671' //sandbox number
};

var client = new (twilio.Client)(settings.sid,
                                  settings.token,
                                  settings.hostname);

var phone = client.getPhoneNumber(settings.phonenumber);

```

然后我们使用`makeCall`方法如下：

```js
phone.makeCall('+4477xxxxxxx1', {}, function(call) {
  call.on('answered', function (req, res) {
    console.log('answered');
    res.append(new (twilio.Twiml).Say('Meet us in the abandoned factory'));
    res.append(new (twilio.Twiml).Say('Come alone', {voice: 'woman'}));
    res.send();
  }).on('ended', function (req) {
    console.log('ended', req);
    process.exit();
  })
});

```

如果我们的帐户没有升级，我们提供给`makeCall`的任何号码都必须通过 Twilio 帐户部分的`Numbers`区域进行验证（[`www.twilio.com/user/account/phone-numbers/`](https://www.twilio.com/user/account/phone-numbers/)）。

`makeCall`调用一个带有`call`参数的回调。`call`是一个事件发射器，具有`answered`和`ended`事件。`twilio`模块将 Twilio 的确认回调透明地转换为这些事件。

`answered`事件将`req`和`res`对象传递给它的回调函数。`req`包含有关传出呼叫的信息，而`res`具有允许我们与呼叫交互的方法，即`res.append`和`res.send`。

要向接收者发送计算机化的文本到语音消息，我们实例化`twilio`模块的`Twiml`类的新实例，并使用`Say`方法（注意给非类的大写 S 的不寻常约定，使用小写 s 会引发错误）。

`TwiML`是 Twilio 的标记语言。它只是特定于 API 的 XML。`twilio`模块提供了`Twiml`类来处理形成必要 XML 的繁琐工作。我们用它来创建两个`Say`动词。在`twilio`的背后，两个`append`调用后跟着的发送调用将创建并发出以下`TwiML`到`twilio`：

```js
<?xml version="1.0" encoding="UTF-8" ?>
<Response>
  <Say>
	  Meet us in the abandoned factory
  </Say>
  <Say voice="woman">
	  Come alone
  </Say>
</Response>

```

`TwiML`代码由`twilio`接收并转换为语音。在女性的声音说“独自来”之后，电话结束，触发我们应用程序中的`ended`事件（`twilio`模块作为接收来自`twilio`的 HTTP 请求的结果发出的，表示通话已经结束）。

我们监听`ended`事件，确定`twilio`（或接收方）何时挂断电话。一旦触发`ended`，我们就退出进程，并输出`req`对象作为通话的概述。

## 另请参阅

+   本章讨论的发送电子邮件

+   本章讨论的 TCP 通信

+   第十章,实时运行

# 使用 TCP 通信

传输控制协议（TCP）提供了 HTTP 通信的基础。通过 TCP，我们可以在运行在不同服务器主机上的进程之间打开接口，并且与 HTTP 相比，可以更少的开销和更少的复杂性远程通信。

Node 为我们提供了`net`模块，用于创建 TCP 接口。在扩展、可靠性、负载平衡、同步或实时社交通信方面，TCP 是一个基本要素。

在这个示例中，我们将演示建立一个 TCP 连接所需的基础，以便通过网络远程监视和过滤网站访问的 HTTP 标头。

## 准备工作

我们需要两个新文件：`server.js`和`monitor.js`。让我们把它们放在一个新的文件夹里。

## 操作步骤...

首先，让我们在`server.js`中创建我们的第一个 TCP 服务器，如下所示：

```js
var net = require('net');
var fauxHttp = net.createServer(function(socket) {
    socket.write('Hello, this is TCP\n');
    socket.end();

   socket.on('data', function (data) {
    console.log(data.toString());
  });

}).listen(8080);

```

我们可以使用`nc`（netcat）命令行程序在另一个终端中进行测试，如下所示：

```js
echo "testing 1 2 3" | nc localhost 8080 

```

### 注意

如果我们使用 Windows，我们可以从[`www.joncraton.org/blog/netcat-for-windows`](http://www.joncraton.org/blog/netcat-for-windows)下载 netcat。

响应应该是`Hello, this is TCP\n`，`server.js`控制台应该输出`testing 1 2 3`。

请记住，HTTP 建立在 TCP 之上，因此我们可以向 TCP 服务器发出 HTTP 请求。如果我们将浏览器导航到`http://localhost:8080`并观察控制台，我们将看到浏览器的 HTTP 请求中的所有标头显示在控制台中，并且浏览器显示**Hello this is TCP**。

我们给 TCP 服务器命名为`fauxHttp`。我们将使用它来记录来自浏览器客户端的 HTTP 标头（通过一些调整，我们可以很容易地调整我们的代码以与实际的 HTTP 服务器一起工作）。

仍然在`server.js`中，我们将创建另一个 TCP 服务器，为`monitor.js`打开第二个端口，以便与我们的服务器通信。不过，在此之前，我们将创建一个新的`EventEmitter`对象，作为我们的两个`server.js` TCP 服务器之间的桥梁：

```js
var net = require('net'),
  stats = new (require('events').EventEmitter),
  filter = 'User-Agent';

var fauxHttp = net.createServer(function(socket) {
    socket.write('Hello, this is TCP\n');
    socket.end();    

   socket.on('data', function (data) {
    stats.emit('stats', data.toString());
  });

}).listen(8080);

```

我们在`socket`的`data`监听器中用新的`stats EventEmitter`替换了`console.log`，它将在接收到 TCP 数据时`emit`一个自定义的`stats`事件。

我们还在`server.js`的第二个 TCP 接口中包含了一个`filter`变量，如下所示：

```js
var monitorInterface = net.createServer(function(socket) {

  stats.on('stats', function (stats) {
    var header = stats.match(filter + ':') || stats.match('');
    header = header.input.substr(header.index).split('\r\n')[0];
    socket.write(header);
  });

  socket.write('Specify a filter [e.g. User-Agent]');
  socket.on('data', function(data) {
    filter = data.toString().replace('\n','');

    socket.write('Attempting to filter by: ' + filter);
  });

}).listen(8081);

```

我们的`monitorInterface`服务器监听我们的`stats`发射器，以确定第一个 TCP 服务器何时接收到信息，并将此信息（在经过过滤后）发送到端口`8081`上连接的客户端。

现在我们只需要创建这个客户端。在`monitor.js`中，我们编写以下代码：

```js
var net = require('net');
var client = net.connect(8081, 'localhost', function () {
  process.stdin.resume();
  process.stdin.pipe(client);
}).on('data', function (data) {
  console.log(data.toString());
}).on('end', function () {
  console.log('session ended');
});

```

当我们在两个终端中打开`server.js`和`monitor.js`，并在浏览器中导航到`http://localhost:8080`时，`server.js`会将每个请求的 HTTP 标头中的`User-Agent`字符串传输到`monitor.js`。

我们可以应用不同的过滤器，比如`Accept`。只需将其输入到运行中的`monitor.js`进程中，任何不匹配的过滤器都将默认返回初步请求行（`GET /, POST /route/here`等）。

要在不同的系统上运行，我们只需将`server.js`放在远程主机上，然后将`net.connect`的第二个参数从`localhost`更新为我们服务器的名称，例如：

```js
var client = net.connect(8081, 'nodecookbook.com', function () {

```

## 工作原理...

HTTP 层建立在 TCP 之上。因此，当我们的浏览器发出请求时，TCP 服务器接收所有 HTTP 头信息。`http.createServer`将处理这些头信息和其他 HTTP 协议交互。但是，`net.createServer`只是接收 TCP 数据。

`socket.write`和`socket.end`类似于 HTTP 服务器回调函数中的`response.write`和`response.end`，但没有参考 HTTP 协议的要求。尽管如此，这些相似之处足以使我们的浏览器能够从`socket.write`接收数据。

我们的`fauxHttp` TCP 服务器所做的就是通过端口`8080`接收请求，向客户端输出`Hello, this is TCP\n`，并直接通过我们的`stats`事件发射器读取客户端的任何数据。

我们的`monitorInterface` TCP 服务器监听一个单独的端口（`8081`），基本上给了我们一种（完全不安全的）管理员界面。在`monitorInterface`回调中，我们监听`stats`发射器，每当浏览器（或任何 TCP 客户端）访问`localhost:8080`时触发。

在`stats`的监听器回调中，我们检索所需的`header`，使用`filter`变量搜索具有`match`对象`index`和`input`属性的 HTTP 头，使我们能够提取指定的头。如果没有匹配项，我们将匹配一个空字符串，从而返回一个包含`index`为`0`的`match`对象，从而提取 HTTP 头的第一行（请求路径和方法）。

`monitorInterface` TCP 服务器回调的最后一部分监听`socket`的`data`事件，并将`filter`变量设置为客户端发送的内容。这使得`monitor.js`客户端可以通过将`process.stdin`流直接传输到 TCP`client`来改变`filter`。这意味着我们可以直接在`monitor.js`的运行过程中输入，并且`monitorInterface`的`socket`的`data`事件将在`server.js`中触发，接收`monitor.js`的 STDIN 中键入的任何内容。

`monitor.js`通过将`process.stdin`流直接传输到 TCP`client`来利用这个功能。这意味着我们可以直接在运行过程中输入，并且`monitor.js`的`socket`中的`data`事件，这将触发从`monitor.js`的 STDIN 中键入的任何内容。

## 还有更多...

让我们看看一些进一步利用 TCP 强大功能的方法。

### 端口转发

转发端口有各种原因。例如，如果我们希望通过移动连接 SSH 到我们的服务器，可能会发现端口`22`已被阻止。公司防火墙也可能会有同样的情况（这可能是因为所有特权端口都被屏蔽，除了最常见的端口，如`80`和`443）。

我们可以使用`net`模块将 TCP 流量从一个端口转发到另一个端口，从本质上绕过防火墙。因此，这应该只用于合法情况，并且需要任何必要的许可。

首先，我们将需要`net`并定义要转发的端口：

```js
var net = require('net');
var fromPort = process.argv[2] || 9000;
var toPort = process.argv[3] || 22;

```

因此，我们可以通过命令行定义端口，或者默认将任意端口`9000`转发到 SSH 端口。

现在我们创建一个 TCP 服务器，通过`fromPort`接收连接，创建一个到`toPort`的 TCP 客户端连接，并在这些连接之间传递所有数据，如下所示：

```js
net.createServer(function (socket) {
  var client;
  socket.on('connect', function () {
    client = net.connect(toPort);
    client.on('data', function (data) {
      socket.write(data);
    });
  })
  .on('data', function (data) {
    client.write(data);
   })
  .on('end', function() {
      client.end();
  });
}).listen(fromPort, function () {
  console.log('Forwarding ' + this.address().port + ' to ' + toPort);
});

```

我们使用`data`事件在`client`（我们的桥接连接）和`socket`（传入连接）之间接收和推送数据。

如果我们现在在远程服务器上运行我们的脚本（不带参数），我们可以使用端口`9000`从本地计算机登录到安全 shell。

```js
ssh -l username domain -p 9000 

```

### 使用 pcap 来监视 TCP 流量

使用第三方的`pcap`模块，我们还可以观察 TCP 数据包在系统内外的传输。这对于分析和优化预期行为、性能和完整性非常有用。

在命令行上：

```js
npm install pcap 

```

对于我们的代码：

```js
var pcap = require('pcap');
var pcapSession = pcap.createSession("","tcp"); //may need to put wlan0, 
                                                                             //eth0, etc. as 1st arg.
var tcpTracker = new pcap.TCP_tracker();

tcpTracker.on('end', function (session) {
    console.log(session);
});

pcapSession.on('packet', function (packet) {
  tcpTracker.track_packet(pcap.decode.packet(packet));
});

```

### 提示

如果`pcap`无法选择正确的设备，将没有输出（或者可能是无关的输出）。在这种情况下，我们需要知道要嗅探哪个设备。如果我们是无线连接，可能是`wlan0`或`wlan1`，如果我们是有线连接，可能是`eth0/eth1`。我们可以通过在命令行上输入`ifconfig`（Linux，Mac OS X）或`ipconfig`（Windows）来找出哪个设备具有与我们路由器 IP 地址的网络部分匹配的`inet 地址`（例如`192.168.1.xxx`）。

如果我们将此保存为`tcp_stats.js`，我们可以使用以下命令运行它：

```js
sudo node tcp_stats.js 

```

`pcap`模块与特权端口进行接口，并且必须以 root 身份运行（对于强制执行特权端口的 Linux 和 Mac OS X 等操作系统）。

如果我们导航到任何网站然后刷新页面，`pcap`的`tcpTracker end`事件会被触发，我们会输出`session`对象。

为了初始化`tcpTracker`，我们创建一个`pcap`会话，并为`packet`事件附加一个监听器，将每个解码的`packet`传递给`tcpTracker`。

在创建`pcap`会话时，我们传递一个空字符串，然后是`tcp`到`createSession`方法。空字符串会导致`pcap`自动选择一个接口（如果这不起作用，我们可以指定适当的接口，例如`eth0，wlan1`或`lo`，如果我们想要分析`localhost`的 TCP 数据包）。第二个参数`tcp`指示`pcap`仅监听 TCP 数据包。

## 另请参阅

+   *在本章中讨论的创建 SMTP 服务器*

+   *在本章中讨论的虚拟主机范式的实现*

# 创建 SMTP 服务器

我们不必依赖第三方 SMTP 服务器，我们可以创建我们自己的！

在这个配方中，我们将使用第三方的`simplesmtp`模块创建我们自己的内部 SMTP 服务器（就像第一个 SMTP 服务器一样），这是第一个配方中`nodemailer`模块的基础库。有关将内部 SMTP 服务器转换为外部公开的 MX 记录服务器的信息，请参阅本配方末尾的*还有更多..*部分。

## 准备工作

让我们创建一个文件并将其命名为`server.js`，然后创建一个名为`mailboxes`的新文件夹，其中包含三个子文件夹：`bob，bib`和`susie`。我们还需要准备好第一个配方中的`mailout.js`文件。

## 如何做...

首先，我们将设置一些初始变量：

```js
var simplesmtp = require('simplesmtp');
var fs = require('fs');
var path = require('path');
var users = [{user: 'node', pass: 'cookbook'}],
  mailboxDir = './mailboxes/',
  catchall = fs.createWriteStream(mailboxDir + 'caught', {flags : 'a'});

```

现在，我们启用带有身份验证的 SMTP 服务器：

```js
var smtp = simplesmtp
  .createServer({requireAuthentication: true})
  .on('authorizeUser', function (envelope, user, pass, cb) {
    var authed;
    users.forEach(function (userObj) {
      if (userObj.user === user && userObj.pass === pass) {
        authed = true;
      }
});
    cb(null, authed);
  });

```

接下来，我们将对一些`simplesmtp`事件做出反应，以处理传入的邮件，从`startData`事件开始：

```js
smtp.on('startData', function (envelope) {
  var rcpt, saveTo;
  envelope.mailboxes = [];
  envelope.to.forEach(function (to) {
    path.exists(mailboxDir + to.split('@')[0], function (exists) {
      rcpt = to.split('@')[0];
          if (exists) {
        envelope.mailboxes.unshift(rcpt);
        saveTo = mailboxDir + rcpt + '/' + envelope.from
          + ' - ' + envelope.date;
        envelope[rcpt] = fs.createWriteStream(saveTo, {flags: 'a'});
        return;
      }
      console.log(rcpt + ' has no mailbox, sending to caught file');
      envelope[rcpt] = catchall;  
    });
  });
});

```

然后`data`和`dataReady`事件将如下所示：

```js
smtp.on('data', function (envelope, chunk) {
  envelope.mailboxes.forEach(function (rcpt) {
    envelope[rcpt].write(chunk);
  });
}).on('dataReady', function (envelope, cb) {
  envelope.mailboxes.forEach(function (rcpt) {
      envelope[rcpt].end();
  });

  cb(null, Date.now());
});

```

为了更简洁的代码，我们使用点符号将这两个事件链接在一起。最后，我们告诉我们的 SMTP 服务器要监听哪个端口：

```js
smtp.listen(2525);

```

在生产环境中，指定端口为`25`（或在更高级的情况下为`465`或`587`）会更加方便。

现在让我们通过将*发送电子邮件*配方中的`mailout.js`文件转换来测试我们的服务器。

首先，我们修改我们的`createTransport`调用以反映我们自定义 SMTP 服务器的值：

```js
var transport = nodemailer.createTransport('SMTP', {
    host: 'localhost',
    secureConnection: false,
    port: 2525,
    auth: {
      user: "node",
      pass: "cookbook"
    }
  });

```

接下来，我们修改`maillist`数组以反映我们的邮箱，如下面的代码所示：

```js
var maillist = [
  'Bob <bob@nodecookbook.com>, Bib <bib@nodecookbook.com>',
  'Miss Susie <susie@nodecookbook.com>',
  'Mr Nobody <nobody@nodecookbook.com>',
];

```

Bob 和 Bib 一起发送。我们还添加了一个没有邮箱的地址（`<nobody@nodecookbook.com>`）以测试我们的捕获所有功能。

现在，如果我们在一个终端中运行`server.js`，在另一个终端中运行`mailout.js`，`mailout.js`的输出应该是这样的：

```js
Sent to Miss Susie <susie@nodecookbook.com>
Sent to Mr Nobody <nobody@nodecookbook.com>
Sent to Bob <bob@nodecookbook.com>, Bib <bib@nodecookbook.com> 

```

如果我们查看`mailboxes/bob`目录，我们会看到来自`spamnot@ok.com`的电子邮件，`susie`和`bib`也是一样。

`server.js`应该有以下输出：

```js
nobody has no mailbox, sending to caught file 

```

因此，当分析`mailboxes/caught`的内容时，我们将看到我们发送给 Mr Nobody 的电子邮件。

## 工作原理...

SMTP 基于 SMTP 客户端和服务器之间的一系列纯文本通信，在 TCP 连接上进行。`simplesmtp`模块为我们执行这些通信，为开发人员交互提供了更高级的 API。

当我们调用 `simplesmtp.createServer` 时，将 `requireAuthorization` 设置为 `true`，我们的新服务器（简称 `smtp`）将触发一个 `authorizeUser` 事件，并且在我们调用第四个参数 `cb`（回调）之前不会继续处理。`cb` 接受两个参数。第一个参数可以通过 `Error` 对象指定拒绝访问的原因（我们只是传递 `null`）。第二个参数是一个布尔值，表示用户是否被授权（如果没有，并且错误参数为 null，则会向邮件客户端发送一个通用的拒绝访问错误）。

我们通过循环遍历我们的 `users` 数组来确定第二个 `cb` 参数，找出用户名和密码是否正确（实际上，我们可能希望使用数据库来完成这部分）。如果匹配成功，我们的 `auth` 变量将设置为 `true` 并传递给 `cb`，否则它保持为 `false`，客户端将被拒绝。

如果客户端被授权，`smtp` 将为每个信封（一个包含该电子邮件所有收件人、正文文本、电子邮件头部、附件等的电子邮件包裹）发出多个事件。

在 `startData` 事件中，我们会得到一个 `envelope` 参数，我们使用 `envelope.to` 属性来检查我们的收件人是否有邮箱。SMTP 允许在一封电子邮件中指定多个收件人，因此 `envelope.to` 总是一个数组，即使它只包含一个收件人。因此，我们使用 `forEach` 循环遍历 `envelope.to`，以便为每个指定的收件人检查邮箱。

我们通过将地址按 `@` 字符拆分来找出预期的收件人邮箱，将其加载到我们的 `rcpt` 变量中。我们对地址的域部分不进行验证，尽管 `simplesmtp` 在触发任何事件之前会自动验证域是否真实。

`rcpt` 被添加到我们的 `envelope.mailboxes` 数组中，在循环遍历 `envelope.to` 之前，我们将其添加到 `envelope` 中。我们在后续的 `data` 和 `dataReady` 事件中使用 `envelope.mailboxes`。

在 `envelope.to forEach` 循环中，我们为 `envelope` 添加一个名为邮箱名称（`rcpt`）的最终属性。如果邮箱存在，我们创建 `writeStream` 来 `saveTo`（路径和文件名由 `envelope.from` 与 `envelope.date` 组合确定）。现在我们已经准备好接收每个收件人邮箱的数据。如果收件人的邮箱不存在，我们将 `envelope[rcpt]` 设置为 `catchall`。`catchall` 是我们在文件顶部设置的全局变量。它是一个带有 `a` 标志的 `writeStream`，以便 `caught` 文件累积孤立的电子邮件。我们在初始化时创建 `catchall writeStream`，然后重用相同的 `writeStream` 用于所有发送到不存在邮箱的邮件。这样可以避免为每封错误寄往的电子邮件创建 `writeStream`，从而节省资源。

`data` 事件会在服务器接收到电子邮件正文的每个块时触发，给我们提供 `envelope` 和 `chunk`。我们使用 `envelope[rcpt].write` 将每个 `chunk` 保存到相应的文件中，通过循环遍历我们自定义的 `envelope.mailboxes` 数组来确定 `rcpt`。

`dataReady` 事件表示所有数据已经接收完毕，数据已经准备好进行处理。由于我们已经存储了数据，我们使用这个事件来结束我们 `mailboxes` 中每个 `rcpt` 的相关 `writeStream`。`dataReady` 事件还需要一个回调（cb）。第一个参数可以是一个 `Error` 对象，允许最终拒绝电子邮件（例如，如果分析电子邮件的内容发现是垃圾邮件）。第二个参数期望一个队列 ID 发送到邮件客户端，在我们的情况下，我们只是给出 `Date.now`。

## 还有更多...

让我们看看如何将我们的 SMTP 服务器转换为公共邮件交换处理程序。

### 从外部 SMTP 服务器接收电子邮件

通过删除授权设置并远程托管我们的 SMTP 服务器，监听端口`25`，我们可以允许其他邮件服务器与我们的 SMTP 服务器通信，以便可以将电子邮件从一个网络传输到另一个网络（例如，从 Gmail 帐户到我们托管的 SMTP 服务器）。

让我们将文件保存为`mx_smtp.js`，并相应地修改以下内容：

```js
var simplesmtp = require('simplesmtp');
var fs = require('fs');
var path = require('path');
var  mailboxDir = './mailboxes/',
    catchall = fs.createWriteStream(mailboxDir + 'caught', {flags : 'a'});
var smtp = simplesmtp.createServer();

```

我们已经丢弃了`users`变量，并更改了`smtp`变量，因此具有`requireAuthentication`属性和相应的`authorizeUser`事件的对象已被删除。为了使外部邮件程序转发到我们的 SMTP 服务器，它必须能够连接。由于其他邮件程序没有认证详细信息，我们必须打开我们的服务器以允许它们这样做。

`startData`数据和`dataReady`事件都保持不变。最终的变化是端口：

```js
smtp.listen(25);

```

为了使其工作，我们必须拥有一个我们可以更改邮件交换（MX）记录的域的实时服务器（例如，亚马逊 EC2 微实例）和根访问权限。

例如，假设我们将我们的 SMTP 服务器托管在`mysmtpserver.net`上，并且我们想要接收`bob@nodecookbook.com`的电子邮件。我们将`nodecookbook.com`的 MX 记录指向`mysmtpserver.net`，优先级为 10。

### 注意

有关如何更改注册商的 DNS 记录的示例，请参见[`support.godaddy.com/help/article/680`](http://support.godaddy.com/help/article/680)。有关 MX 记录的更多信息，请查看[`en.wikipedia.org/wiki/MX_record`](http://en.wikipedia.org/wiki/MX_record)。

一旦更改完成，它们可能需要一段时间才能传播（最多 48 小时，尽管通常更快）。我们可以使用`dig mx`（Mac OS X 和 Linux）或`nslookup set q=MX`（Windows）来确定我们的 MX 记录是否已更新。

我们必须在我们的远程主机上安装 Node，并确保端口`25`是公开的，并且没有被其他程序使用。要检查其他程序是否使用端口`25`，请使用 SSH 登录并键入`netstat -l`。如果在**活动互联网连接**（仅服务器）部分中看到`*:smtp`，则表示已经有程序在使用该端口，并且必须停止（尝试`ps -ef`查找任何嫌疑人）。

在实际服务器上，我们创建包含`bob bib`和`susie`的`mailboxes`文件夹，将我们的`mx_smtp.js`文件复制过去，并安装`simplesmtp`。

```js
npm install simplesmtp 

```

现在，如果一切都设置正确，并且我们的 MX 记录已更新，我们可以在实际服务器上执行我们的`mx_smtp.js`文件。然后向`<bob@nodecookbook.com>`发送测试电子邮件（或者`@`我们已更改 MX 记录的任何域），等待几秒钟，然后检查`mailboxes/bob`文件夹。电子邮件应该已经出现。

## 另请参阅

+   发送电子邮件在本章中讨论

+   *部署到服务器环境*在第十章中讨论，上线

# 实施虚拟托管范式

如果我们希望在一个服务器上托管多个站点，我们可以使用虚拟托管来实现。虚拟托管是一种根据名称独特处理多个域名的方法。这种技术非常简单：我们只需查看传入的`Host`标头并相应地做出响应。在这个任务中，我们将为静态站点实现基于名称的简单虚拟托管。

## 准备工作

我们将创建一个名为`sites`的文件夹，其中`localhost-site`和`nodecookbook`作为子目录。在`localhost-site/index.html`中，我们将编写以下内容：

```js
<b> This is localhost </b>

```

在`nodecookbook/index.html`中，我们将添加以下代码：

```js
<h1>Welcome to the Node Cookbook Site!</h1>

```

对于本地测试，我们将希望配置我们的系统以使用一些额外的主机名，以便我们可以将不同的域指向我们的服务器。要做到这一点，我们在 Linux 和 Max OS X 上编辑`/etc/hosts`，或者在 Windows 系统上编辑`%SystemRoot%\system32\drivers\etc\hosts`。

在文件的顶部，它将我们的本地环回 IP`127.0.0.1`映射到`localhost`。让我们将这一行改为：

```js
127.0.0.1 localhost nodecookbook

```

最后，我们想要创建两个新文件：`mappings.js`和`server.js`。`mappings.js`文件将为每个域名提供静态文件服务器，而`server.js`将提供虚拟托管逻辑。

我们将使用`node-static`模块来提供我们的站点，我们的虚拟主机将只提供静态网站。如果我们还没有它，我们可以通过`npm`安装它，如下所示：

```js
npm install node-static 

```

## 如何做到...

让我们从`mappings.js`开始：

```js
var static = require('node-static');

function staticServe (dir) {
  return new (static.Server)('sites/' + dir)
}

exports.sites = {
  'nodecookbook' : staticServe('nodecookbook'),
  'localhost' : staticServe('localhost-site')
} ;

```

我们已经使用了系统`hosts`文件中列出的域。在生产场景中，域将通过 DNS 记录指向我们。

现在是`server.js`的时候了：

```js
var http = require('http');

var port = 8080,
  mappings = require('./mappings');

var server = http.createServer(function (req, res) {
  var domain = req.headers.host.replace(new RegExp(':' + port + '$'), ''),
    site = mappings.sites[domain] ||
      mappings.sites[mappings.aliases[domain]];

  if (site) { site.serve(req, res); return; }
  res.writeHead(404);
  res.end('Not Found\n');

}).listen(port);

```

现在当我们导航到`http://localhost:8080`或`http://localhost.localdomain:8080`时，我们会得到`sites/localhost-site/index.html`中的内容。而如果我们转到`http://nodecookbook:8080`，我们会得到大的 Node Cookbook 欢迎消息。

## 它是如何工作的...

每当我们的服务器收到一个请求，我们都会去掉端口号（对于端口`80`服务器来说是不必要的），以确定`domain`。

然后我们将`domain`与我们的`mappings.sites`对象进行交叉引用。如果找到一个站点，我们调用它的`serve`方法，该方法是从`node-static`库继承的。在`mappings.js`中，每个`exports.sites`属性都包含一个指向相关站点目录的`node-static Server`实例。我们使用我们的自定义`staticServer`函数作为包装器，以使代码更加整洁。

要使用静态`Server`实例，我们调用其`serve`方法，通过`req`和`res`对象传递，就像在`server.js`中一样：

```js
  if (site) { site.serve(req, res); return; }

```

`site`变量是指向给定域名的适当站点文件夹的`static.Server`实例。

如果`server.js`在请求的域中找不到`mapping.js`中的站点，我们只需向客户端传递`404`错误。

## 还有更多...

超越静态托管进入动态托管，或者如果我们想要在我们的网站上使用 SSL/TLS 证书呢？

### 虚拟托管 Express 应用程序

Express/Connect 带有`vhost`中间件，它允许我们轻松实现基于 Express 的动态虚拟托管。

首先，我们需要设置两个 Express 应用程序。让我们删除`nodecookbook`和`localhost-site`文件夹，并使用`express`二进制文件重新生成它们，如下所示：

```js
rm -fr nodecookbook && express nodecookbook
rm -fr localhost-site && express localhost-site
cd nodecookbook && npm -d install
cd ../localhost-site && npm -d install 

```

我们还需要修改每个站点文件中`app.js`的最后部分，将`app.listen`方法包装在`if module.parent`条件中：

```js
if (!module.parent) {

  app.listen(3000);
  console.log("Express server listening on port %d in %s mode",        	app.address().port, app.settings.env);

}

```

在我们的`nodecookbook`应用程序中，让我们在`index.jade`中添加以下内容：

```js
h1 Welcome to the Node Cookbook site!

```

在`localhost-site`应用程序中，我们将以下代码添加到`index.jade`中：

```js
b this is localhost

```

有了我们的站点设置，我们可以修改`mappings.js`如下：

```js
function appServe (dir) {
  return require('./sites/' + dir + '/app.js')
}

exports.sites = {
  'nodecookbook' : appServe('nodecookbook'),
  'localhost' : appServe('localhost-site')
};

```

我们已经删除了`node-static`模块，因为我们改用了 Express。我们的`staticServe`便利函数已被修改为`appServe`，它只是根据`exports.servers`中的映射使用`require`加载每个 Express 应用程序。

我们将更新`server.js`如下：

```js
var express = require('express'),
  mappings = require('./mappings'),
  app = express.createServer();

Object.keys(mappings.sites).forEach(function (domain) {
  app.use(express.vhost(domain, mappings.sites[domain]));
});

app.listen(8080);

```

我们创建一个主`app`，然后循环遍历`mappings.sites`，将每个子应用程序传递到`app.use`中与`express.vhost`一起使用。`vhost`中间件接受两个参数。第一个是域名。我们从`mappings.sites`键中获取每个`domain`。第二个是一个 Express 应用程序。我们从`mappings.sites`中的值中检索每个 Express 应用程序。

我们只需请求域和`vhost`中间件就会将相关域与相关应用程序对齐，以提供正确的站点。

### 服务器名称指示

在服务器名称指示（SNI）之前，基于名称的 SSL/TLS 虚拟托管对于网站是一个复杂的管理问题（需要将每个主机名存储在多域证书中）。

这是因为在服务器接收到任何 HTTP 头之前，基于指定域名的证书建立了加密连接。因此，服务器无法提供特定于一个域的证书。因此，浏览器会明确警告用户连接可能不安全，因为证书上列出的域名与正在访问的域名不匹配。为了避免这种情况，虚拟主机必须购买包含托管的每个域的证书，然后每次添加或删除新域时重新申请新证书。

SNI 在 SSL/TLS 握手开始时将请求的域名转发到服务器，允许我们的服务器为域名选择适当的证书，并防止浏览器告诉我们的用户他们可能受到攻击。

`https.Server`（继承自`tls.Server`）具有`addContext`方法，允许我们为多个单独的域指定主机名和证书凭据。

让我们通过进行一些更改来启用 TLS 兼容的虚拟主机。首先，在`mappings.js`中，我们将添加另一个方便函数，称为`secureShare:`

```js
function secureShare(domain) {
  var site = {
    content: staticServe(domain),
    cert: fs.readFileSync('sites/' + domain + '/certs/cert.pem'),
    key: fs.readFileSync('sites/' + domain + '/certs/key.pem')
  };
  return site;
} ;

```

接下来，我们将改变加载站点的方式，调用`secureShare`而不是`staticServe:`

```js
exports.sites = {
  'nodecookbook.com' : secureShare('nodecookbook.com'),
  'davidmarkclements.com' : secureShare('davidmarkclements.com')
};

```

为了使这个示例在生产环境中工作，我们必须用我们控制的域名替换示例域名，并获得由受信任的证书颁发机构签发的真正证书。

### 注意

我们可以通过按照本章的支持代码文件中的说明（在`secure_virtual_hosting/howto`下）在本地进行测试。

让我们通过将`sites`文件夹结构更改为符合`mappings.js`中所做的更改，将`nodecookbook`重命名为`nodecookbook.com`，将`localhost-site`重命名为`davidmarkclements.com`，并将后者的`index.html`文件更改为以下内容：

```js
<b>This is DavidMarkClements.com virtually AND secure</b>

```

每个`site`文件夹还需要一个包含我们的`cert.pem`和`key.pem`文件的`certs`文件夹。这些文件必须是专门为该域购买的证书。

在`server.js`中，我们将脚本顶部更改为以下内容：

```js
var https = require('https');
var fs = require('fs');

```

需要`fs`模块来加载我们的凭据。由于我们已经用`https`替换了`http`，我们将修改我们的`createServer`调用如下：

```js
var server = https.createServer({}, function (req, res) {

```

只需将`http`添加`s`即可。在这种情况下，没有 SSL/TLS 凭据，因为我们将使用`addContext`方法按域加载这些凭据。因此，我们只传递一个空对象。

在`mappings.js`中，我们的`secureShare`函数返回一个包含三个属性`content, key`和`cert`的对象，其中`content`保存静态服务器。因此，在`server.js`中，我们更新这一行：

```js
  if (site) { site.serve(req, res); return; }

```

至：

```js
  if (site) { site.content.serve(req, res); return; }

```

由于我们正在托管在实时服务器上，我们通过绑定到`0.0.0.0:`将其暴露给传入的 Web 连接

```js
}).listen(port, '0.0.0.0');

```

我们还可以将`port`变量更改为`443`，以直接通过 HTTPS 端口提供服务（我们必须以 root 身份运行服务器才能做到这一点，在实际环境中这具有安全风险，请参见第十章，*上线*，了解如何安全地执行此操作）。

最后，我们在`server.js`的底部添加以下内容：

```js
Object.keys(mappings.sites).forEach(function (hostname) {
  server.addContext(hostname, {key: mappings.sites[hostname].key,
    cert: mappings.sites[hostname].cert});    
});

```

这将根据`mappings.js`中的设置加载每个域的`key`和`cert`属性。

只要我们为每个指定的域拥有受信任的 CA 认证凭据，并且我们使用现代浏览器，我们就可以在不接收警告的情况下使用 HTTPS 访问每个站点。

### 提示

**注意**

有一个问题：服务器名称指示仅适用于现代浏览器。在这种情况下，现代浏览器不包括在 Windows XP 上运行时的 Internet Explorer 7/8 和 Safari，以及 Android Gingerbread（2.x 版本）和 Blackberry 浏览器。如果我们通过`https.createServer`的选项对象提供了默认证书，用户仍然可以在旧版浏览器上查看网站，但他们将收到与我们不使用 SNI 时相同的警告（旧版浏览器在 SSL/TLS 协商中不指示主机名，因此我们的 SNI 处理永远不会发生）。根据预期的市场，我们可能必须使用替代方法，直到这些旧版浏览器在与我们的目的相关的数量上使用得足够少。

## 另请参阅

+   *提供静态文件*在第一章中讨论，搭建 Web 服务器

+   *动态路由*在第六章中讨论，使用 Express 加速开发

+   *设置 HTTPS Web 服务器*在第七章中讨论，实现安全、加密和认证

+   *部署到服务器环境*在第十章中讨论，上线
