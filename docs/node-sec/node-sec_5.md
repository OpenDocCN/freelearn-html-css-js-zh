# 第五章：响应层漏洞

您与用户请求的最后交互当然是响应。这里的讨论将集中在应用程序代码的这一部分的漏洞和最佳实践。这将包括**跨站脚本攻击**（**XSS**），一些**拒绝服务**（**DoS**）攻击的向量，甚至各种浏览器用于实施特定安全策略的 HTTP 标头。

# 跨站脚本攻击（XSS）

跨站脚本攻击（XSS）是处理 Web 应用程序时的一个更受欢迎的话题，因为在许多方面，这是 HTML/CSS/JavaScript 的默认行为。具体来说，XSS 是一种攻击向量，用于向 Web 页面注入不受信任且可能恶意的代码。通常，这被视为向您的页面注入 JavaScript 代码的机会，该代码现在可以访问特定 Web 页面中客户端几乎可以访问的任何内容。

默认情况下，JavaScript 在浏览器中以全局范围执行，包括由不受信任的来源注入的代码。这与您自己的受信任代码的行为相同，使其成为具有许多可能性的危险向量。恶意脚本可以找到用户的会话 ID（通常在 cookie 中），并使用 AJAX 将该信息发送给可以劫持用户会话的人。

注入通常来自未经过滤或消毒的用户输入，然后输出到浏览器。考虑以下示例代码：

```js
var express = require("express"),
    app = express();

app.get("/", function (req, res) {
    var output = "";
    output += '<form action="/test">';
    output += '<input name="name" placeholder="enter a name">';
    output += '</form>';

    res.send(output);
});

app.get("/test", function (req, res) {
    res.send("Hello, " + req.query.name);
});

app.listen(3000);
```

这个脚本创建了一个服务器，简单地发送一个 HTML 表单，该表单通过`GET`提交到另一个页面。第二个路由简单地将用户输入的值输出到浏览器。

如果用户输入他们的名字（比如 Dominic），一切都很好，用户在下一页上看到**"Hello, Dominic"**。但是，如果用户输入了其他内容，比如原始 HTML 呢？在这种情况下，它只是将 HTML 与我们自己的 HTML 一起输出，浏览器无法区分。

如果您在该文本字段中输入`<script>alert('hello!');</script>`，那么当您打开下一个页面时，您将看到**"Hello,"**，并且浏览器将触发一个带有**"hello!"**的警报框。这只是一个无害的例子，但这种漏洞有巨大的潜在危害。这些攻击是通过所谓的不受信任的数据完成的，这些数据可能是原始用户输入，存储在数据库中的信息，或者通过远程数据源访问的信息。然后，您的应用程序使用这些不受信任的数据来构造某种命令，然后执行该命令。当命令被操纵以执行开发人员原始意图之外的某些操作时，危险就会出现。

这种类型攻击的原型示例是 SQL 注入，其中不受信任的数据用于更改 SQL 命令。考虑以下代码：

```js
var sql = "SELECT * FROM users WHERE name = '" + username + "'";
```

假设用户名变量来自用户输入，重点是它是我们定义的不受信任的数据。如果用户输入了一些无害的东西，比如`'Dominic'`，那么一切都很好，生成的 SQL 看起来像以下代码：

```js
SELECT * FROM users WHERE name = 'Dominic'
```

如果有人输入了一些不那么无害的东西，比如：`'' OR 1=1`，那么生成的 SQL 就会变成以下样子：

```js
SELECT * FROM users WHERE name = '' OR 1=1
```

这完全改变了查询的含义，而不是限制为具有匹配名称的一个用户，现在返回了每一行。这可能会更加灾难性，考虑值：`''; DROP TABLE users;`，它将生成以下 SQL：

```js
SELECT * FROM users WHERE name = ''; DROP TABLE users;
```

没有任何额外的访问权限，用户已经导致了我们应用程序的严重数据损失，可能会使所有用户无法使用整个应用程序。

事实证明，XSS 是另一种类型的注入攻击，Web 浏览器和它们执行的 HTML、CSS 和 JavaScript 都针对这些类型的攻击进行了优化。我们需要了解每种语言中的许多不同上下文。考虑以下模板：

```js
<h2>User: <%= username %></h2>
```

使用我们不信任的数据，我们可以很容易地通过向该值注入额外的 HTML 来引起麻烦，比如`<script>alert('xss');</script>`，这将生成以下 HTML 代码：

```js
<h2>User: <script>alert('xss');</script></h2>
```

解决方法是在这个上下文中对任何添加到页面的不受信任的数据使用 HTML 转义。这种技术将 HTML 中重要的字符，比如尖括号和引号，转换为它们对应的 HTML 实体；防止它们改变嵌入其中的 HTML 结构。以下表格是这种转换的一个例子：

| 字符 | 实体 |
| --- | --- |
| 小于号 (`<`) | `&lt;` |
| 大于号 (`>`) | `&gt;` |
| 双引号 (`"`) | `&quot;` |
| 单引号 (`'`) | `'`（`&apos;`不是有效的 HTML，应该避免使用） |
| 和号 (`&`) | `&amp;` |
| 斜杠 (`/`) | `/` |

这种转义方法使得攻击者更难改变你的 HTML 结构，这是保护你的网页非常重要的技术。然而，不同的上下文将需要更多的转义技术，我们将很快讨论。

### 提示

许多流行的模板库默认包括自动的 HTML 转义，但有些则不包括。这对于选择模板框架或库对你来说应该是一个重要因素。

HTML 属性可以被注入其他 HTML，用于创建一个新的上下文，比如关闭属性并开始一个新的属性。更进一步，这个注入的 HTML 可以用来关闭 HTML 标签，并在另一个上下文中注入更多的 HTML。考虑以下模板：

```js
<img height=<%= height %> src="img/...">
```

考虑以下用于高度的注入值：`100 onload="javascript:alert('XSS');"`, 这将生成以下 HTML：

```js
<img height=100 onload="javascript:alert('XSS');" src="img/...">
```

结果是注入的 JavaScript 代码。在这种特定的上下文中，像我们之前使用的 HTML 编码是不够的，因为前面仍然是一个完全有效的 HTML。除了我们之前提到的 HTML 转义，你应该要求在所有 HTML 属性周围加上引号，特别是当涉及到不受信任的数据时。为了涵盖所有情况，甚至是未引用的属性，你可以将所有 ASCII 值低于 256 编码为它们的 HTML 实体格式或可用的命名实体，比如`&quot;`。

涉及 URL 的 HTML 属性，比如`href`和`src`，是另一个需要自己编码的上下文。考虑以下模板：

```js
<a href="<%= url %>">Home Page</a>
```

如果用户输入以下数据：`javascript:alert('XSS');`，那么将生成以下 HTML：

```js
<a href="javascript:alert('XSS');">Home Page</a>
```

在这里不适用 HTML 编码，因为前面是有效的 HTML 标记。相反，应该检查一个完全合格的 URL 是否包含意外的协议。在这里，我们使用了`javascript:`，这会让浏览器执行任意代码，就像`eval()`函数一样。最后，输出应该通过内置的 JavaScript 函数`encodeURI()`进行转义，该函数转义 URL 中无效的字符。

我将在这里展示的最后一个例子是在先前提到的属性中部分 URL。使用以下模板：

```js
<a href="/article?page=<%= nextPage %>">Next</a>
```

`nextPage`变量被用作 URL 的一部分，而不是 URL 本身。我们之前提到的`encodeURI()`函数有一个伴侣叫做`encodeURIComponent()`，它转义更多的字符，因为它是用来编码单个查询字符串参数的。

另一个常见的反模式是直接将 JSON 数据注入页面，以在渲染页面的同时在服务器和客户端之间共享数据。考虑以下模板：

```js
<script>
var clientData = <%= JSON.stringify(serverData); %>;
</script>
```

这种特定的技术，虽然方便，也可能导致 XSS 攻击。假设`serverData`对象有一个名为`username`的属性，反映了当前用户的名字。还假设这个值可以由用户设置，而没有任何过滤，直接在用户输入和页面显示之间（当然不应该发生）。

如果用户将他的名字改为`</script><script>alert('XSS')</script>`，那么输出的 HTML 将如下所示：

```js
<script>
var clientData = {"username":"</script><script>alert('XSS');</script>"};
</script>
```

根据 HTML 规范，`</`字符（即使在 JavaScript 字符串中，就像我们这里）将被解释为一个闭合标签，攻击者刚刚创建了一个全新的脚本标签，就像任何其他脚本标签一样，它对页面有完全控制权。

与其直接尝试转义 JSON 数据，减轻这个问题的最佳方法是使用另一种方法来注入你的 JSON 数据：

```js
<script id="serverData" type="application/json">
<%= html_escape(JSON.stringify(data)) %>
</script>

<script>
var dataElement = document.getElementById("serverData");
var dataText = dataElement.textContent || dataElement.innerText; // unescapes the content of the script
var data = JSON.parse(dataText);
 </script>
```

这种方法使用了一个带有预定义 ID 的脚本标签，我们可以用它来检索它。当浏览器遇到它不理解的脚本类型时，它将简单地不执行它，同时将其隐藏在用户面前。这个脚本标签的内容将是我们的 JSON 的 HTML 转义版本，这样可以确保我们没有上下文边界的交叉。

接下来，我们使用另一个脚本（最好是外部文件，但绝不是必需的），其中包含查找我们定义的脚本元素并检索其文本内容的代码。通过使用`textContent/innerText`属性而不是`innerHTML`，我们得到了浏览器为我们执行的额外转义，以防万一。最后，我们通过`JSON.parse`运行 JSON 数据来实际执行 JSON 解码。

虽然这种方法需要更多的宣传，而且比第一个例子要慢一些，但它会更安全，这是一个很好的权衡。

这些例子绝不是一个详尽的列表，但它们应该说明 HTML、CSS 和 JavaScript 各自都有上下文，允许各种类型的代码注入。永远不要相信用户输入，并确保根据上下文使用适当的转义方法。

**开放式 Web 应用安全项目**（**OWASP**）是一个维护维基（[`www.owasp.org/`](http://www.owasp.org/)）的基金会，专门针对所有 Web 应用程序的安全考虑。他们有关于许多攻击向量的文章，包括一个更全面的检查表，用于防止许多更多种类的 XSS 攻击。

# 拒绝服务

**拒绝服务**（**DoS**）攻击可以采用各种形式，但主要目的是阻止用户访问你的应用程序。一种方法是向服务器发送大量请求，占用服务器的资源，阻止合法请求得到满足。

请求洪水通常针对多线程服务器，比如**Apache**。这是因为为每个请求生成一个新线程的过程为同时请求的数量提供了一个容易达到的上限。对于 Node.js 平台的事件循环，这种特定类型的攻击通常不那么有效，尽管这并不意味着它是不可能的。

如果不正确使用事件循环，事件循环仍然可能会暴露应用程序，我无法强调理解它的重要性有多大，同时编写任何 Node.js 应用程序。你的应用程序代码与事件循环的约定是尽可能快地运行。一次只有一个应用程序的部分在运行，所以 CPU 密集型也可能占用资源。这适用于所有情况，但我在这一章中提到它是为了特别解决你的响应处理程序。

如前所述，尽可能使用流，特别是在处理网络请求或文件系统时。处理大块数据可能是耗时的，取决于你如何处理这些数据，使用流可以将这些大操作分解成许多小块，从而在过程中满足其他请求。

# 与安全相关的 HTTP 头

有一些可用的 HTTP 标头可以帮助我们的 Web 应用程序增加一些安全性。我们将看一下一个名为**头盔**的模块，它被编写为一个 Connect/Express 中间件的集合，根据您的配置添加这些标头。我们将检查头盔包括的每个中间件函数，以及它们的效果的简要解释。

## 内容安全策略

首先，头盔支持为 HTML 和 Web 应用程序的一种新的安全机制设置标头，称为**内容安全策略**（**CSP**）。XSS 攻击通过使用其他方法欺骗浏览器传递有害内容来规避**同源策略**（**SOP**）。

对于支持此功能的浏览器，您可以将资源（例如图像、框架或字体）限制为通过白名单域加载。这通过希望阻止访问不受信任的域加载恶意内容，从而限制了 XSS 攻击的影响。

CSP 通过一个或多个`Content-Security-Policy` HTTP 标头传达给浏览器，例如：

```js
Content-Security-Policy: script-src 'self'
```

此标头将指示浏览器要求所有脚本仅从当前域加载。浏览器检测到来自任何其他域的脚本将被直接阻止。

CSP 标头是由分号分隔的一系列指令构成的。实现多个 CSP 限制的标头示例如下：

```js
Content-Security-Policy: script-src 'self'; frame-src 'none'; object-src 'none'
```

此标头指示浏览器仅限制脚本到当前域（与我们之前的示例相同），并且完全禁止使用框架（包括 iframes）和对象。

每个指令都被命名为`*-src`，后面跟着一个以空格分隔的预定义关键字列表（必须用引号括起来）或域 URL。

可用的关键字包括以下内容：

+   `'self'`：这将脚本限制为当前域

+   `'none'`：这限制了所有域（根本不能加载）

+   `'unsafe-inline'`：这允许内联代码（强烈建议避免，稍后讨论）

+   `'unsafe-eval'`：这允许文本到 JavaScript 的机制，如`eval()`（同样强烈不建议）

可用的指令包括以下内容：

+   `connect-src`：这限制了可以通过 XHR 和 WebSockets 连接的域

+   `font-src`：这限制了可以用于下载字体文件的域

+   `frame-src`：这限制了可以加载框架（包括内联框架）的域

+   `img-src`：这限制了可以从中加载图像的域

+   `media-sr`：这限制了视频和音频的来源

+   `object-src`：这允许控制对象的来源（例如 Flash）

+   `script-src`：这限制了可以从中加载脚本的域

+   `style-src`：这限制了可以从中加载样式表的域

+   `default-src`：这充当所有指令的缩写

省略指令会使其策略完全开放（这是默认行为），除非您指定`default-src`指令。

头盔可以根据您传递给中间件的配置为每个支持的用户代理（例如浏览器）构造标头。默认情况下，它将提供以下 CSP 标头：

```js
Content-Security-Policy: default-src 'self'
```

这是一个非常严格的策略，因为它只允许从当前域加载外部资源，而不允许从其他任何地方加载。在大多数情况下，这太过严格，特别是如果您要使用 CDN 或允许外部服务与您自己通信。

您可以通过中间件定义函数来配置头盔，通过添加一个名为`defaultPolicy`的属性，其中包含您的指令作为对象哈希，例如：

```js
app.use(helmet.csp.policy({
    defaultPolicy: {
        "script-src": [ "'self'" ],
        "img-src": [ "'self'", "http://example.com/" ]
    }
}));
```

这将指示头盔发送以下标头：

```js
Content-Security-Policy: script-src 'self'; img-src 'self' http://example.com/
```

这将限制脚本和图像仅限于当前域以及域[`example.com/`](http://example.com/)。

CSP 还包括了一个报告功能，你可以用来审计自己的应用程序并快速检测漏洞。有一个专门用于此目的的`report-uri`指令，告诉浏览器发送违规报告的 URI。参考以下示例代码：

```js
Content-Security-Policy: default-src 'self'; ...; report-uri /my_csp_report_parser;
```

当浏览器发送报告时，它是一个具有以下结构的 JSON 文档：

```js
{
  "csp-report": {
    "document-uri": "http://example.org/page.html",
    "referrer": "http://evil.example.com/",
    "blocked-uri": "http://evil.example.com/evil.js",
    "violated-directive": "script-src 'self' https://apis.google.com",
    "original-policy": "script-src 'self' https://apis.google.com; report-uri http://example.org/my_amazing_csp_report_parser"
  }
}
```

这份报告包括了大部分你需要追踪违规行为的信息，即：

+   `document-uri`：发生违规的页面

+   `blocked-uri`：违规资源

+   `violated-directive`：违反的具体指令

+   `original-policy`：页面的策略（CSP 头的内容）

当刚开始使用 CSP 时，可能不明智立即设置策略并开始阻止。在详细说明应用程序策略的过程中，你可以设置 CSP 以尊重报告模式。

这允许你设置完整的策略，而不是立即阻止用户，你可以简单地接收详细违规报告。这为你提供了在实施之前微调策略的方法。

要启用报告模式，你只需更改 HTTP 头部名称。不再使用我们一直在使用的，而是简单地使用`Content-Security-Policy-Report-Only`，其他一切保持不变：

```js
Content-Security-Policy-Report-Only: default-src 'self'; ...; report-uri /my_csp_report_parser;
```

在 helmet 中，通过在配置对象中包含`reportOnly`参数来启用报告模式：

```js
express.use(helmet.csp.policy({
    reportOnly: true,
    defaultPolicy: {
        "script-src": [ "'self'" ],
        "img-src": [ "'self'", "http://example.com/" ]
    }
}));
```

这设置了我们之前使用的相同策略，只是增加了报告模式。

CSP 是一种出色的安全机制，你应该立即开始使用，尽管浏览器支持并不完全。截至本文撰写时，它是**W3C 候选推荐**，预计浏览器将以快速的速度实现这一功能。

## HTTP 严格传输安全（HSTS）

**HTTP 严格传输安全**（HSTS）是一种机制，向用户代理（例如，Web 浏览器）通信，特定应用程序应仅通过 HTTPS 访问，因为这是加密通信。如果你的应用程序理想情况下只存在于安全连接上，这允许你正式向浏览器声明。

这个头部只有两个参数，`max-age`指令告诉浏览器要尊重配置的时间（以秒为单位），以及`includeSubDomains`指令以相同方式处理当前域的子域。与 CSP 一样，这是通过 HTTP 头部通信的：

```js
Strict-Transport-Security: max-age=15768000
```

这告诉浏览器，大约六个月内，当前域从现在开始应该通过 HTTPS 访问（即使用户通过 HTTP 访问）。这是由 helmet 设置的默认配置，也是最简单的实现方式：

```js
app.use(helmet.hsts());
```

这使用先前说明的配置设置了 HSTS 的中间件，中间件定义函数还接受两个可选参数。首先，`max-age`指令可以设置为一个数字（应以秒表示）。其次，`includeSubDomains`指令可以设置为一个简单的布尔值：

```js
app.use(helmet.hsts(1234567, true));
```

这将设置以下头部：

```js
Strict-Transport-Security: max-age=1234567; includeSubdomains
```

浏览器支持目前并不像 CSP 那样完整，但预计会朝着这个方向前进。与此同时，将其添加到应用程序的安全详细信息中是值得的。

## X-Frame-Options

这个头部控制特定页面是否允许加载到`<frame>`或`<iframe>`元素中。这主要用于防止恶意用户劫持（或“点击劫持”）你的用户，从而欺骗他们执行他们本来没有打算执行的操作。

这是通过另一个 HTTP 头部通信给浏览器的，因此当浏览器加载一个框架/iframe 的 URL 时，它将检查这个头部以确定采取的行动。头部看起来像下面这样：

```js
X-Frame-Options: DENY
```

在这里，我们使用值`DENY`，这是通过头盔配置时的默认值。其他可用选项包括`sameorigin`，它只允许在当前域上加载域。最后一个选项是`allow-from`选项，允许您指定可以在框架中呈现当前页面的 URI 白名单。

在大多数情况下，默认设置应该工作得很好，您可以通过头盔这样设置：

```js
app.use(helmet.xframe());
```

这将添加我们之前看到的标头。要使用`sameorigin`选项进行配置，请使用以下配置：

```js
helmet.xframe('sameorigin');
```

最后，这将设置`allow-from`变体，还为您提供了设置允许的 URI 的第二个参数：

```js
helmet.xframe('allow-from', 'http://example.com');
```

对于这种安全机制，浏览器支持非常好，因此可以立即实施。`allow-from`标头是一个警告，不是均匀支持的，因此在使用之前，请确保根据您的要求研究具体情况。

## X-XSS-Protection

这个下一个标头是特定于 Internet Explorer 的，它启用了 XSS 过滤器。而不是我自己解释，这是来自**Microsoft Developer Network**（**MSDN**）的解释。

### 注意

XSS 过滤器作为 Internet Explorer 8 组件运行，可以查看浏览器中流动的所有请求/响应。当过滤器在跨站点请求中发现可能的 XSS 时，它会在服务器的响应中识别并中和攻击。有关更多信息，请访问：[`msdn.microsoft.com/en-us/library/dd565647(v=vs.85).aspx`](http://msdn.microsoft.com/en-us/library/dd565647(v=vs.85).aspx)

这个功能可能默认情况下已启用，但是如果用户自己禁用了它或在某些选择区域禁用了它，可以使用类似以下的简单标头来启用它：

```js
X-XSS-Protection: 1; mode=block
```

通过将标头设置为 0，强制禁用 XSS 过滤器，但该配置不通过头盔公开。实际上，它根本没有配置，因此其使用就像这样简单：

```js
app.use(helmet.iexss());
```

## X-Content-Type-Options

这是另一个标头，它阻止某些浏览器的特定行为（目前只有 Internet Explorer 和 Google Chrome 支持此功能）。在这种情况下，即使资源本身设置了有效的`Content-Type`标头，浏览器也会尝试“嗅探”（例如，猜测）返回资源的 MIME 类型。

这可能会导致浏览器被欺骗以执行或呈现开发人员意外的方式的文件，这取决于许多因素可能导致潜在的安全漏洞。关键是您的服务器的`Content-Type`标头应该是浏览器考虑的唯一因素，而不是试图自行猜测。

与前面的例子一样，没有真正的配置可用，以下标头将简单地添加到您的应用程序中：

```js
X-Content-Type-Options: nosniff
```

通过头盔配置此标头：

```js
app.use(helmet.contentTypeOptions());
```

## Cache-Control

头盔提供的最后一个中间件是用于将`Cache-Control`标头设置为`no-store`或`no-cache`。这可以防止浏览器缓存给定的响应。这个中间件也没有配置，并且是通过以下方式包含的：

```js
app.use(helmet.cacheControl());
```

您将使用此中间件和标头来防止浏览器存储和缓存可能包含敏感用户信息的页面。然而，这样做的折衷是当在整体应用程序中应用时，您可能会遇到严重的性能问题。

在处理静态文件和资源（例如样式表和图像）时，此标头只会减慢您的站点速度，并且可能不会增加任何安全性好处。确保小心地在整体应用程序中如何以及何处应用此特定中间件。

头盔模块是向您的应用程序添加这些有用的安全功能的快速方法，这是由 Connect 创建的强大中间件架构启用的。有很多这些安全功能中的许多无法在这里解决，并且可能会在将来发生变化，因此最好熟悉它们所有。

# 总结

在本章中，我们看到了在应用程序处理的响应阶段出现的漏洞，比如 XSS 和 DoS。我们还研究了如何通过防御性编码或利用更新的安全标准和政策来减轻这些特定问题。
