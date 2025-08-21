# 第四章：请求层考虑

一些漏洞出现在应用程序的请求阶段。如前所述，Node.js 默认情况下为您做的很少，让您完全自由地构建满足您需求的服务器。

# 限制请求大小

常常在 Node.js 应用程序中被忽略的一个主要请求处理功能是大小限制。**Express**（可选）处理请求体数据的缓冲和将请求体解析为有意义的数据结构。当请求仍在被满足时，整个请求体的内容都在内存中。如果不设置限制，恶意用户有多种方法来影响您的系统，例如耗尽内存限制，上传占用不必要磁盘空间的文件。

根据您的需求，您需要确定应用程序的合理限制。虽然您的需求可能不同，但您应该始终设置某种限制，Connect 和 Express 为此目的专门提供了一个中间件，称为 limit：

```js
app.use(express.limit("5mb"));
```

此中间件需要尽早添加到堆栈中，否则直到太迟才会被捕获。它需要一个单独的配置，即请求大小的上限。如果发送一个数字，它将被转换为字节数。您还可以发送一个更可读的字符串，例如`"5mb"`或`"1gb"`。

如果超出限制，此中间件将响应**413（请求实体太大）**错误。首先，检查请求的`Content-Length`标头，如果太大，则直接拒绝请求。当然，标头可能是伪造的，甚至不存在，因此中间件还监视传入数据，如果实际请求体大小达到限制，则触发错误。

`bodyParser`中间件用于解析特定内容类型的传入请求体。实际上，`bodyParser`中间件具体来说只是三个不同中间件的简写，即`json`，`urlencoded`和`multipart`。每个中间件对应不同的内容类型。通过限制中间件设置绝对大小是有帮助的，但并不总是足够的。一些请求体应该有不同的限制。

例如，您可能希望允许最多 100MB 的文件上传。但是，同样大小的 JSON 将在`JSON.parse()`函数运行时使您的应用程序停止，因为它是一个阻塞操作。因此，强烈建议除了多部分（因为它处理文件上传）之外，为请求体设置一个更小的限制。

因此，我建议避免使用`bodyParser`中间件，以便更明确，并允许您为每个子中间件设置不同的限制。

```js
// module dependencies
var express = require("express"),
    app = express();

// limiting the allowed size of request bodies (by content-type)
app.use(express.urlencoded({ limit: "1kb" })); // application/x-www-form-urlencoded
app.use(express.json({ limit: "1kb" }));       // application/json
app.use(express.multipart({ limit: "5mb" }));  // multipart/form-data
app.use(express.limit("2kb"));                 // everything else
```

### 提示

像我们在这里讨论的为不同内容类型设置不同限制一样，如果您对中间件的选择顺序不小心，结果可能会出乎意料。

如果首先使用限制中间件，它将导致其他中间件忽略它们自己的大小限制。确保将全局限制中间件放在最后，这样它就可以作为任何其他内容类型的通用处理，而不是由`bodyParser`中间件系列处理。

## 使用流而不是缓冲

Node.js 包含一个名为**streams**的模块，其中包含广泛用于 Node.js 平台自身核心模块的实现。流很像 Unix 管道，它们可以被读取，写入，或者根据上下文甚至两者都可以。我不会在这里详细介绍，但流是 Node.js 的一个杀手功能，您应该尽可能在应用程序和任何 npm 模块中使用它们。

如果您正在实现更多的 RESTful API，例如接受文件上传作为`PUT`请求，那么在请求处理程序中使用流。以下代码显示了处理将请求体放入文件的低效方法：

```js
var fs = require("fs");

// handle a PUT request against /file/:name
app.put("/file/:name", function (req, res, next) {
    var data = "", // data buffer
        filename = req.params.name; // the URL parameter

    req.on("data", function (chunk) {
        data += chunk; // each data event appends to the buffer
    });

    req.on("end", function () {
        // write the buffered data to a file
        fs.writeFile(filename, data, function (err) {
            if (err) return next(err); // handle a write error

            res.send("Upload Successful"); // success message
        });
    });
});
```

在这里，我们将整个请求体缓冲到内存中，然后将其写入磁盘。在小尺寸时，这不是问题，但攻击者可能同时发送许多大型请求体，通过缓冲将自己置于不必要的风险中。在 Node.js 中，使用流来处理数据是一种长期的方法（谢天谢地，更短的方法也是最好的方法！）。

以下代码是相同请求的示例，只是使用流将数据传送到目的地。

```js
var fs = require("fs");

// handle a PUT request against /file/:name
app.put("/file/:name", function (req, res, next) {
    var filename = req.params.name, // the URL parameter
        // open a writable stream for our uploaded data
        destination = fs.createWriteStream(filename);

    // if our destination could not be written to, throw an error
    destination.on("error", next);

    req.pipe(destination).on("end", function () {
        res.send("Upload Successful"); // success message
    });
});
```

我们的示例设置了一个可写流，表示上传数据的目的地。数据将被直接传送到文件中，而不是在内存中缓冲整个请求体。需要注意的是，这个示例没有正确过滤用户输入；这完全是为了专注于示例的主题，不应直接应用于生产代码。

流是在许多情境下处理数据的一种经过验证和有效的模式，并充分利用了 Node.js 的事件驱动模型。

当处理许多同时用户，特别是在出现意想不到的交通高峰时，准备好应对灾难情景是很重要的，其中负载变得过重，超出服务器的处理能力。这也适用于缓解拒绝服务（DoS）攻击，这些攻击试图用比服务器可能处理的更多请求来淹没服务器，使其完全崩溃（或者只是减慢到爬行速度）。

# 监视事件循环的响应性

构建一个在重载下不会崩溃的服务器是可行的。一个有用的模式是监视事件循环的响应性，并立即拒绝一些请求，如果服务器负载过重，无法快速响应。有一个模块叫做 node-toobusy（https://github.com/lloyd/node-toobusy）就是这样做的。

一旦初始化，toobusy 将轮询事件循环，并监视延迟或事件循环中超过预期时间的请求。在应用程序中，您设置一个中间件层，简单地查询监视器，以确定是否将请求添加到服务器的当前处理队列。如果服务器太忙，它将以 503（服务器当前不可用）的方式进行响应，而不是承担超出其满足能力的负载。这种模式允许您继续尽可能多地提供服务请求，而不是使服务器崩溃，如下面的代码所示。

```js
var toobusy = require("toobusy"),
    express = require("express"),
    app = express();

// middleware which blocks requests when we're too busy
app.use(function(req, res, next) {
    if (toobusy()) {
        res.send(503, "I'm busy right now, sorry.");
    } else {
        next();
    }
});

app.get("/", function(req, res) {
    // each request blocks the event loop
    var start = (new Date()).getTime(), now;
    while (((new Date()).getTime() - start) <= 5000); // run for 5 seconds
    res.send("Hello World");
});

var server = app.listen(3000);
process.on("SIGINT", function() {
    server.close();
    // calling .shutdown allows your process to exit normally
    toobusy.shutdown();
    process.exit();
});
```

前面的示例是在 node toobusy 的 github 页面上找到的。它设置了一个使用 toobusy 模块的简单服务器中间件。它还设置了一个阻塞事件循环的单个路由，运行了五秒钟。如果出现一些同时请求足够长时间阻塞事件循环的情况，服务器将开始以 503（服务器当前不可用）错误进行响应，而不是承担超出其承受范围的负载。最后，这还包括了进程的优雅关闭。

这个示例还演示了关于 Node.js 中事件循环的一个非常重要的观点，值得重复。您的代码与事件循环调度器之间的约定是，所有代码应该快速执行，以避免阻塞事件循环的其他代码。这意味着要避免在应用程序代码中进行 CPU 密集型计算，不像前面的示例，在其 while 循环迭代期间阻塞 CPU。

Node.js 在应用程序主要是 I/O 绑定时效果最佳，因此应避免 CPU 密集型操作，比如复杂的计算或非常大的数据集迭代。如果系统需要这样的操作，考虑将阻塞部分作为单独的进程进行分离，以避免占用应用程序的事件循环。

有几种方法可以实现这一点，比如使用 HTML5 Web Worker API for node ([`github.com/pgriess/node-webworker`](https://github.com/pgriess/node-webworker))。此外，一个更基本的方法是利用 Node 的`child_process`模块与**进程间通信**（**IPC**）结合使用。关于 IPC 的具体内容可能严重依赖于您的平台和架构，这超出了本讨论的范围。

# 跨站点请求伪造

**跨站点请求伪造**（**CSRF**）是一种攻击向量，它利用了应用程序对特定用户浏览器的信任。在用户不知情的情况下，应用程序代表用户发出请求，从而使应用程序在假定受信任的用户发出请求的情况下执行某些操作，尽管实际上并非如此。

有许多方法可以实现这一点。一个例子是，一个 HTML 图像标签（例如，`<img>`）以某种方式被注入到页面中，无论是合法的还是非法的，比如通过 XSS，这是我们将在下一章中讨论的一个漏洞。浏览器隐式地向`src`属性中指定的 URL 发送请求，并在 HTTP 请求的一部分中发送任何 cookie。许多跟踪用户身份的应用程序通过包含某种会话标识符的 cookie 来实现，这样对服务器来说，就好像用户发出了请求。

预防措施非常简单；最常见的方法是要求在修改状态的每个请求中包含一个生成的用户特定令牌。事实上，Connect 已经包含了`csrf`中间件，就是为了这个目的。

它通过向当前用户的会话添加一个生成的令牌来工作，该令牌可以作为一个隐藏的输入字段或任何具有副作用的链接中的查询字符串值包含在 HTML 表单中。当处理后续请求时，中间件会检查用户会话中的值是否与请求提交的值匹配，如果不匹配，则会失败并返回**403（禁止）**。

```js
var express = require("express"),
    app = express();

app.use(express.cookieParser()); // required for session support
app.use(express.bodyParser());   // required by csrf
app.use(express.session({ secret: "secret goes here" })); // required by csrf
app.use(express.csrf());

// landing page, just links to the 2 different sample forms
app.get("/", function (req, res) {
    res.send('<a href="/valid">Valid</a> <a href="/invalid">Invalid</a>')
});

// valid form, includes the required _csrf token in the HTML Form (hidden input)
app.get("/valid", function (req, res) {
    var output = "";
    output += '<form method="post" action="/">'
    output += '<input type="hidden" name="_csrf" value="' + req.csrfToken() + '">';
    output += '<input type="submit">';
    output += '</form>';
    res.send(output);
});
// invalid form, does not have the required token
// throws a "Forbidden" error when submitted
app.get("/invalid", function (req, res) {
    var output = "";
    output += '<form method="post" action="/">'
    output += '<input type="submit">';
    output += '</form>';
    res.send(output);
});

// POST target, redirects back to home if successful
app.post("/", function (req, res) {
    res.redirect("/");
});

app.listen(2500);
```

这个示例应用程序有一些定义好的中间件，即`bodyParser`，`cookieParser`和`session`。这些都是`csrf`所需的，这就是为什么它们在顺序中排在第一位。此外，还有一些路由，如下所示：

+   主页，只提供两个示例表单的链接

+   表单操作/目标，只需在成功提交时将用户重定向到主页

+   有效的表单，包括令牌作为隐藏输入，并成功提交

+   无效的表单，不包括令牌，因此在提交时失败（带有(**403 Forbidden)** HTTP 响应）

这种方法可以防止攻击者成功发出虚假请求，因为所需的令牌对于每个表单提交都是不同的。

# 输入验证

在保护许多攻击向量，比如我们将在下一章中处理的 XSS 时，重要的是在接收用户输入时对其进行过滤和清理。这发生在 Web 应用程序的请求阶段，所以我们将在这里进行讨论。一个基本的经验法则是始终验证输入并转义输出。

用于验证用户输入的流行库是 node-validator ([`github.com/chriso/node-validator`](https://github.com/chriso/node-validator))。这个库绝不是唯一的选择，但它是我们在示例中将要使用的选择。

输入验证有几个目标，首先是验证传入的用户输入是否符合我们应用程序及其工作流程的标准；例如，您可能希望确保用户提交有效的电子邮件地址。我指的不是发送电子邮件进行确认以测试电子邮件地址是否真实，而是确保他们一开始就不输入错误的值。另一个例子是确保数字匹配特定范围，比如大于零。

其次，输入过滤旨在防止不良数据进入系统，可能会损害另一个子系统；例如，如果您接受某个数字输入，然后将其传递给另一个子系统进行一些额外的处理，比如报告或其他远程 API。如果您的用户故意或无意地提交其他意外的值，比如符号或字母字符，可能会在未来的操作中造成问题。在很大程度上，计算机是垃圾进，垃圾出，因此我们需要确保我们对任何用户输入都要小心谨慎。

第三，正如之前简要提到的，输入过滤是一种有用的（尽管不完整的）预防措施，可以防止**跨站脚本攻击**（**XSS**）等攻击。在 HTML、CSS 和 JavaScript 中的 XSS 攻击存在访问控制的严重问题，这意味着任何脚本都具有与其他脚本相同的访问权限。这意味着如果攻击者能够找到一种方式将进一步的代码注入到您的页面中，他们将拥有很大程度的控制权，这对您的用户可能是有害的。输入过滤可以通过删除可能巧妙嵌入其他用户输入的恶意代码来帮助。

除了基本的 node-validator 库之外，还有一个中间件插件（express-validator：[`github.com/ctavan/express-validator`](https://github.com/ctavan/express-validator)），专门为 Express.js 制作，我们将在示例中使用它。

我们的第一个示例将是一个接受各种输入的表单，只是为了尽可能地进行演示。考虑以下 HTML 表单：

```js
<form method="post">
    <div>
        <label>Name</label>
        <input type="text" name="name">
    </div>
    <div>
        <label>Email</label>
        <input type="email" name="email">
    </div>
    <div>
        <label>Website</label>
        <input type="url" name="website">
    </div>
    <div>
        <label>Age</label>
        <input type="number" name="age">
    </div>
    <div>
        <label>Gender</label>
        <select name="gender">
            <option>-- choose --</option>
            <option value="M">Male</option>
            <option value="F">Female</option>
        </select>
    </div>

    <button type="submit">Validate</button>
</form>
```

这个示例代码设置了一个带有五个字段的 HTML 表单：`name`，`e-mail`，`website`，`age`和`gender`。用户可以在提供的输入框中输入值，并`POST`到相同的 URL。在处理`POST`请求时，我们将验证数据并给出某种响应。下一个代码示例将是我们的应用程序代码：

```js
// module dependencies
var express = require("express"),
    app = module.exports = express();

app.use(express.bodyParser());           // required by csrf
app.use(require("express-validator")()); // the validation middleware

// an HTML form to be validated
app.get("/", function (req, res) {
    res.sendfile(__dirname + "/views/validate-input.html");
});

/**
 * Validates the input, will either:
 *  - sends a 403 Forbidden response in the event of validation errors
 *  - send a 200 OK response if the data validates successfully
 */
app.post("/", function (req, res, next) {
    // validation
    req.checkBody("name").notEmpty().is(/\w+/);
    req.checkBody("email").notEmpty().isEmail();
    req.checkBody("website").isUrl();
    req.checkBody("age").isInt().min(0).max(100);
    req.checkBody("gender").isIn([ "M", "F" ]);
    // filtering
    req.sanitize("name").trim();
    req.sanitize("email").trim();
    req.sanitize("age").toInt();

    var errors = req.validationErrors(true);

    if (errors) {
        res.json(403, {
            message: "There were validation errors",
            errors: errors
        });
    } else {
        res.json({
            name: req.param("name"),
            email: req.param("email"),
            website: req.param("website"),
            age: req.param("age"),
            gender: req.param("gender")
        });
    }
});
```

这个示例设置了一个基本的 Web 服务器，只有两个路由，一个是`GET /`，它只是发送之前显示的 HTML 表单作为响应。第二个路由是`POST /`，它接受从上述表单提交的数据，并首先根据以下规则进行验证：

| 字段 | 规则 |
| --- | --- |
| `name` | 该字段不能为空。它必须匹配一个正则表达式（这意味着它只能是字母、数字、空格和一些特定符号）。 |
| `e-mail` | 这必须是一个有效的电子邮件地址。 |
| `website` | 这必须是一个有效的 URL。 |
| `age` | 这必须是一个数字。它必须大于或等于 0。它必须小于或等于 100。 |
| `gender` | 这必须是"M"或"F"。 |

除了验证输入，它还根据以下规则执行一些过滤和转换以进行输出：

| 字段 | 规则 |
| --- | --- |
| `name` | 去除前导和尾随空格。 |
| `e-mail` | 去除前导和尾随空格。 |
| `age` | 转换为整数。 |

根据验证的结果，它要么以**403（禁止）**的状态响应，并附带验证错误列表，要么以**200（OK）**的状态响应，并附带过滤后的输入。

这应该表明，向应用程序添加输入验证和过滤非常简单，并且收益是非常值得的。您可以确保数据与各种工作流程的预期格式匹配，并有助于预防性地防范一些攻击向量。

# 摘要

在本章中，我们特别研究了请求漏洞，并提供了一些避免和处理这些漏洞的方法。在下一章中，我们将研究应用程序的响应阶段以及出现的漏洞。
