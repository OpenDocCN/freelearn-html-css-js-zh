# 第九章：实际示例 - 构建静态服务器

在过去的几章中，我们已经了解了 Node.js 及其提供的功能。虽然我们没有涵盖每个模块或 Node.js 提供的所有内容，但我们已经有了所有的要素来构建一个静态内容/生成器站点。这意味着我们将设置一个服务器来监听请求，并根据该请求构建页面。

为了实现这个服务器，我们需要了解站点生成的工作原理，以及如何将其作为即时操作实现。除此之外，我们还将研究缓存，以便我们不必在每次请求页面时重新编译。总的来说，在本章中，我们将查看并实现以下内容：

+   理解静态内容

+   设置我们的服务器

+   添加缓存和集群

# 技术要求

+   一个文本编辑器或**集成开发环境**（**IDE**），最好是 VS Code

+   支持 Node.js 的操作系统

+   本章的代码可以在以下网址找到：[`github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter09/microserve`](https://github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter09/microserve)。

# 理解静态内容

静态内容就是不变的内容。这可以是 HTML 页面、JavaScript、图像等。任何不需要通过数据库或某些外部系统进行处理的内容都可以被视为静态内容。

虽然我们不会直接实现静态内容服务器，但我们将实现一个即时静态内容生成器。对于不了解的人来说，静态内容生成器是一个构建静态内容然后提供该内容的系统。内容通常由某种模板系统构建。

一些常见的模板系统包括 Mustache、Handlebars.js 和 Jade。这些模板引擎寻找某种标记，并根据一些变量替换内容。虽然我们不会直接查看这些模板引擎，但要知道它们存在，并且它们对于诸如代码文档生成或甚至根据某些 API 规范创建 JavaScript 文件等方面非常有用。

我们将实现自己的模板系统版本，而不是使用其中一个常见格式，以了解模板的工作原理。我们将尽量保持简单，因为我们希望为我们的服务器使用最少的依赖项。我们将使用一个名为`Remarkable`的 Markdown 到 HTML 转换器作为依赖项：[`github.com/jonschlinkert/remarkable`](https://github.com/jonschlinkert/remarkable)。它依赖于两个库，每个库又依赖于一个库，因此我们将导入总共五个库。

虽然即时创建所有页面将使我们能够轻松进行更改，但除非我们处于开发环境中，否则我们不希望一直这样做。为了确保我们不一遍又一遍地构建 HTML 文件，我们将实现一个内存缓存来存储被请求最多的文件。

有了这些，让我们继续开始构建我们的应用程序，通过设置我们的服务器并发送响应。

# 启动我们的应用程序

首先，让我们通过在我们选择的文件夹中创建我们的`package.json`文件来设置我们的项目。我们可以从以下基本的`package.json`文件开始：

```js
{
    "version" : "0.0.1",
    "name"    : "microserver",
    "type"    : "module"
}
```

现在应该相当简单了。主要的是将类型设置为`module`，这样我们就可以在 Node.js 中使用模块。接下来，让我们继续通过在放置`package.json`文件的文件夹中运行`npm install remarkable`来添加`Remarkable`依赖项。有了这个，我们现在应该在我们的`package.json`文件中列出`remarkable`作为一个依赖项。接下来，让我们继续设置我们的服务器。为此，创建一个`main.js`文件并执行以下操作：

1.  导入`http2`和`fs`模块，因为我们将使用它们来启动我们的服务器和读取我们的私钥和证书文件，如下所示：

```js
import http2 from 'http2'
import fs from 'fs'
```

1.  创建我们的服务器并读取我们的密钥和证书文件。我们将在设置主文件后生成这些文件，就像这样：

```js
const server = http2.createSecureServer({
    key: fs.readFileSync('selfsignedkey.pem'),
    cert: fs.readFileSync('selfsignedcertificate.pem')
});
```

1.  通过崩溃我们的服务器来响应错误事件（我们可能应该更好地处理这个问题，但现在这样做就可以了）。我们还将通过简单的消息和状态码`200`（表示一切正常）来处理传入的请求，就像这样：

```js
server.on('error', (err) => {
    console.error(err);
    process.exit();
});
server.on('stream', (stream, headers) => {
    stream.respond({
       'content-type': 'text/html',
        ':status': 200
    });
    stream.end("A okay!");
});
```

1.  最后，我们将开始监听端口`50000`（这里可以使用一个随机端口号）。

现在，如果我们尝试运行这个，我们应该会被类似以下的一个令人讨厌的错误消息所打招呼：

```js
Error: ENOENT: no such file or directory, open 'selfsignedkey.pem'
```

我们还没有生成自签名的私钥和证书。请记住从第六章中了解到，我们不能在不安全的通道（HTTP）上提供任何内容；相反，我们必须使用 HTTPS。为此，我们需要从证书颁发机构获取证书，或者我们需要自己生成一个。从第六章中了解到，我们应该在我们的计算机上安装`openssl`应用程序。

1.  让我们继续通过运行以下命令来生成它，并只需通过命令提示符按*Enter*：

```js
> openssl req -newkey rsa:2048 -nodes -keyout selfsignedkey.pem -x509 -days 365 -out selfsignedcertificate.pem
```

现在我们应该在当前目录中有这两个文件，现在，如果我们尝试运行我们的应用程序，我们应该有一个在端口`50000`上监听的服务器。我们可以通过访问以下地址来检查：`127.0.0.1:50000`。如果一切正常，我们应该看到消息 A okay！

虽然像端口、私钥和证书这样的变量在开发过程中硬编码是可以的，但我们仍然应该将它们移到我们的`package.json`文件中，这样另一个用户可以在一个地方进行更改，而不是必须进入代码并进行更改。让我们继续进行这些更改。在我们的`package.json`文件中，让我们添加以下字段：

```js
"config" : {
    "port" : 50000,
    "key"  : "selfsignedkey.pem",
    "certificate" : "selfsignedcertificate.pem",
    "template" : "template",
    "body_files" : "publish"
},
"scripts" : {
   "start": "node --experimental-modules main.js"   
}
```

`config`部分将允许我们传递各种变量，让包的用户使用`package.json`的`config`部分设置，或者在运行我们的文件时使用`npm config set tinyserve:<variable>`命令设置。正如我们从第五章中看到的，`scripts`部分允许我们访问这些变量，并允许我们的包的用户现在只需使用`npm start`，而不是使用`node --experimental-modules main.js`。有了这个，我们可以通过在我们的`main.js`文件中声明所有这些变量来改变我们的`main.js`文件，就像这样：

```js
const ENV_VARS = process.env;
const port = ENV_VARS.npm_package_config_port || 80;
const key  = ENV_VARS.npm_package_config_key || 'key.pem';
const cert = ENV_VARS.npm_package_config_certificate || 'cert.pem';
const templateDirectory = ENV_VARS.npm_package_config_template || 'template';
const publishedDirectory = ENV_VARS.npm_package_config_bodyFiles || 'body';
```

所有配置变量都可以在我们的`process.env`变量中找到，因此我们在文件顶部声明了一个快捷方式。 然后，我们可以访问各种变量，就像我们在第五章中看到的那样，*切换上下文-没有 DOM，不同的 Vanilla*。 我们还设置了默认值，以防用户没有使用我们声明的`npm start`脚本运行我们的文件。 用户还会注意到我们声明了一些额外的变量。 这些是我们稍后会讨论的变量，但它们涉及到我们要超链接到的位置以及我们是否要启用缓存（开发变量）。 接下来，我们将看一下我们将如何访问我们想要设置的模板系统。

# 设置我们的模板系统

我们将使用 Markdown 来托管我们想要托管的各种内容，但我们将希望在所有文章中使用某些部分。 这些将是我们页面的页眉、页脚和侧边栏等内容。 我们可以将这些内容模板化，而不必将它们插入到我们为文章创建的所有 Markdown 文件中。

我们将把这些部分放在一个在运行时将被知道的文件夹中，通过我们声明的`templateDirectory`变量。 这也将允许我们包的用户更改我们的静态站点服务器的外观和感觉，而无需做任何太疯狂的事情。 让我们继续创建模板部分的目录结构。 这应该看起来像下面这样：

+   **模板**：我们应该在所有页面中查找静态内容

+   **HTML**：我们所有静态 HTML 代码将放在这里

+   **CSS**：我们的样式表将存放在这里

有了这个目录结构，我们现在可以创建一些基本的页眉、页脚和侧边栏 HTML 文件，以及一些基本的**层叠样式表**（**CSS**），以获得一个对每个人都应该熟悉的页面结构。 所以，让我们开始，如下所示：

1.  我们将编写`header` HTML，如下所示：

```js
<header>
    <h1>Our Website</h1>
    <nav>
        <a href="/all">All Articles</a>
        <a href="/contact">Contact Us</a>
        <a href="/about">About Us</a>
    </nav>
</header>
```

有了这个基本结构，我们有了网站的名称，然后是大多数博客网站都会有的一些链接。

1.  接下来，让我们创建`footer`部分，就像这样：

```js
<footer>
    <p>Created by: Me</p>
    <p>Contact: <a href="mailto:me@example.com">Me</a></p>
</footer>
```

1.  再次，相当容易理解。 最后，我们将创建侧边栏，如下所示：

```js
<nav>
    <% loop 5
    <a href="article/${location}">${name}</a>
    %>
</nav>
```

这就是我们的模板引擎发挥作用的地方。 首先，我们将使用`<% %>`字符模式来表示我们要用一些静态内容替换它。 接下来，`loop <number>`将让我们的模板引擎知道我们计划在停止引擎之前循环一定次数的下一个内容。 最后，`<a href="article/${location}">${name}</a>`模式将告诉我们的模板引擎这是我们要放入的内容，但我们将要用我们在代码中传递的对象中的变量替换`${}`标签。

接下来，让我们继续创建我们页面的基本 CSS，如下所示：

```js
*, html {
    margin : 0;
    padding : 0;
}
:root {
   --main-color : "#003A21"; 
   --text-color : "#efefef";
}
/* header styles */
header {
    background : var(--main-color);
    color      : var(--text-color);
}
/* Footer styles */
footer {
    background : var(--main-color);
    color  : var(--text-color);
}
```

由于大部分是样板代码，CSS 文件已经被剪切了。 值得一提的是自定义变量。 使用 CSS，我们可以通过使用模式`--<name> : <content>`声明自定义变量，然后我们可以在 CSS 文件中使用`var()`声明来使用它。 这使我们能够重用变量，如颜色和高度，而无需使用预处理器，如**SASS**。

CSS 变量是有作用域的。 这意味着如果您为`header`部分定义变量，它将仅在`header`部分中可用。 这就是为什么我们决定将我们的颜色放在`:root`伪元素级别，因为它将在整个页面中可用。 只需记住，CSS 变量的作用域类似于我们在 JavaScript 中声明的`let`和`const`变量。

有了我们的 CSS 布局，我们现在可以在我们的`template`文件中编写我们的主 HTML 文件。我们将把这个文件移到 HTML 文件夹之外，因为这是我们想要的主文件，以便把所有东西放在一起。这也会让我们的包的用户知道这是我们将用来组合所有部分的主文件，如果他们想要改变它，他们应该在这里做。现在，让我们创建一个看起来像下面这样的`main.html`文件：

```js
<!DOCTYPE html>
<html>
    <head>
        <link rel="stylesheet"  type="text/css" href="css/main.css" />
    </head>
    <body>
        <% from html header %>
        <% from html sidebar %>
        <% from html footer %>
    </body>
</html>
```

顶部部分应该看起来很熟悉，但是现在我们有了一个新的模板类型。`from`指令让我们知道我们将从其他地方获取这个文件。下一个语句表示它是一个`HTML`文件，所以我们将在`HTML`文件夹中查找。最后，我们看到文件的名称，所以我们知道我们要引入`header.html`文件。

有了所有这些，我们现在可以编写我们将用来构建页面的模板系统。我们将利用`Transform`流来实现我们的模板系统。虽然我们可以利用类似`Writable`流的东西，但是利用`Transform`流更有意义，因为我们根据一些输入条件改变输出。

要实现`Transform`流，我们需要跟踪一些东西，这样我们才能正确处理我们的键。首先，让我们读取并发送适当的块进行处理。我们可以通过实现`transform`方法并输出我们要替换的块来实现这一点。为此，我们将执行以下操作：

1.  我们将扩展一个`Transform`流并设置基本结构，就像我们在第七章中所做的那样，*流-理解流和非阻塞 I/O*。我们还将创建一个自定义类来保存缓冲区的开始和结束位置。这将允许我们知道我们是否在同一个循环中得到了模式匹配的开始。我们以后会需要这个。我们还将为我们的类设置一些私有变量，比如`begin`和`end`模板缓冲区，以及`#pattern`变量等状态变量，如下所示：

```js
import { Transform } from 'stream'
class Pair {
    start = -1
    end = -1
}
export default class TemplateBuilder extends Transform {
    #pattern = []
    #pair = new Pair()
    #beforePattern = Buffer.from("<%")
    #afterPattern = Buffer.from("%>")
    constructor(opts={}) {
        super(opts);
    }
    _transform(chunk, encoding, cb) {
        // process data
    }
}
```

1.  接下来，我们将不得不检查我们的`#pattern`状态变量中是否保存了数据。如果没有，那么我们知道要寻找模板的开始。一旦我们检查到模板语句的开始，我们可以检查它是否实际上在这个数据块中。如果是，我们将`#pair`的`start`属性设置为这个位置，这样我们的循环就可以继续进行；否则，我们在这个块中没有模板，我们可以开始处理下一个块，如下所示：

```js
// inside the _transform function
if(!this.#pattern.length && !this.#pair.start) {
    location = chunk.indexOf(this.#beforePattern, location);
    if( location !== -1 ) {
        this.#pair.start = location;
        location += 2;
    } else {
        return cb();
   }
}
```

1.  要处理另一个条件（我们正在寻找模板的结尾），我们需要处理更多的状态。首先，如果我们的`#pair`变量的`start`不是`-1`（我们设置它），我们知道我们仍在处理当前的块。这意味着我们需要检查我们是否可以在当前块中找到`end`模板缓冲区。如果我们找到了，那么我们可以处理模式并重置我们的`#pair`变量。否则，我们只是将当前块从`#pair`的`start`成员位置推送到我们的`#pattern`持有者的块末端，如下所示：

```js
if( this.#pair.start !== -1 ) {
    location = chunk.indexOf(this.#afterPattern, location);
    if( location !== -1 ) {
        this.#pair.end = location;
        this.push(processPattern(chunk.slice(this.#pair.start,this.#pair.end)));
        this.#pair = new Pair();
    } else {
        this.#pattern.push(chunk.slice(this.#pair.start));
    }
}
```

1.  最后，如果`#pair`的`start`成员被设置，我们检查`end`模板模式。如果我们找不到它，我们只是将整个块推送到`#pattern`数组。如果我们找到它，我们就从它的开头切割块，直到我们找到我们的`end`模板字符串。然后我们将所有这些连接在一起并进行处理。然后我们还将我们的`#pattern`变量重置为什么都不持有，就像这样：

```js
location = chunk.indexOf(this.#afterPattern, location);
if( location !== -1 ) {
    this.#pattern.push(chunk.slice(0, location));
    this.push(processPattern(Buffer.concat(this.#pattern)));
    this.#pattern = [];
} else {
    this.#pattern.push(chunk);
}
```

1.  所有这些都将包装在一个`do`/`while`循环中，因为我们至少要运行这段代码一次，当我们的`location`变量是`-1`时，我们就知道我们已经完成了（这是从`indexOf`检查返回的，当它找不到我们想要的时）。在`do`/`while`循环之后，我们运行回调，告诉我们的流我们已经准备好处理更多数据，如下所示：

```js
do {
  // transformation code
} while( location !== -1 );
cb();
```

将所有这些放在一起，我们现在有一个`transform`循环，应该处理几乎所有情况来获取我们的模板系统。我们可以通过将我们的`main.html`文件传递进去并将以下代码放入我们的`processPattern`方法中来测试这一点，就像这样：

```js
console.log(pattern.toString('utf8'));
```

1.  我们可以创建一个测试脚本来运行我们的`main.html`文件。继续创建一个`test.js`文件，并将以下代码放入其中：

```js
import TemplateStream from './template.js';
const file = fs.createReadStream('./template/main.html');
const tStream = new TemplateStream();
file.pipe(tStream);
```

有了这个，我们应该得到一个漂亮的输出，其中包含我们正在寻找的模板语法，比如`from html header`*.* 如果我们通过`sidebar.html`文件运行它，它应该看起来像下面这样：

```js
loop 5
    <a href="article"/${location}">${name}</a>
```

现在我们知道我们的`Transform`流的模板查找代码是有效的，我们只需要编写我们的处理块系统来处理我们之前的情况。

现在要处理这些块，我们需要知道在哪里查找文件。还记得之前我们在`package.json`文件中声明各种变量吗？现在，我们将利用`templateDirectory`。让我们将其作为流的参数传递进去，就像这样：

```js
#template = null
constructor(opts={}) {
    if( opts.templateDirectory ) {
        this.#template = opts.templateDirectory;
    }
    super(opts);
}
```

现在，当我们调用`processPattern`时，我们可以将块和`template`目录作为参数传递。从这里，我们现在可以实现`processPattern`方法。我们将处理两种情况：当我们找到一个`for`循环和当我们找到一个`find`语句。

要处理`for`循环和`find`语句，我们将按以下步骤进行：

1.  我们将构建一个缓冲区数组，除了`for`循环之外，它将是模板保存的内容。我们可以使用以下代码来实现这一点：

```js
const _process = pattern.toString('utf8').trim();
const LOOP = "loop";
const FIND = "from";
const breakdown = _process.split(' ');
switch(breakdown[0]) {
    case LOOP:
        const num = parseInt(breakdown[1]);
        const bufs = new Array(num);
        for(let i = 0; i < num; i++) {             
           bufs[i] = Buffer.from(breakdown.slice(2).join(''));
        }
        break;
   case FIND:
        console.log('we have a find loop', breakdown);
        break;
   default:
        return new Error("No keyword found for processing! " + 
         breakdown[0]);
}
```

1.  我们将查找循环指令，然后获取第二个参数，它应该是一个数字。如果我们打印出来，我们会看到我们有一堆填满相同数据的缓冲区。

1.  接下来，我们需要确保填写所有的模板字符串位置。这些看起来像`${<name>}`的模式。为此，我们将在这个循环中添加另一个参数，用于指定我们想要使用的变量的名称。让我们将其添加到`sidebar.html`文件中，如下所示：

```js
<% loop 5 articles
    <a href="article/${location}">${name}</a>
%>
```

1.  有了这个，我们现在应该传入一个我们想要在模板系统中使用的变量列表——在这种情况下，一个名为`articles`的数组，其中包含具有`location`和`name`键的对象。这可能看起来像下面这样：

```js
const tStream = new TemplateStream({
    templateDirectory,
    templateVariables : {
        sidebar : [
            {
                location : temp1,
                name     : 'article 1'
            }
        ]
    }
}
```

满足我们`for`循环条件的条件足够多，现在我们可以回到`Transform`流，并将其作为我们在构造函数中要处理的项目之一，并将其发送到我们的`processPattern`方法。一旦我们在这里添加了这些项目，我们将在`for`循环内更新我们的循环情况，使用以下代码：

```js
const num = parseInt(breakdown[1]);
const bufs = new Array(num);
const varName = breakdown[2].trim();
for(let i = 0; i < num; i++) {
    let temp = breakdown.slice(3).join(' ');
    const replace = /\${([0-9a-zA-Z]+)}/
    let results = replace.exec(temp);           
    while( results ) {
        if( vars[varName][i][results[1]] ) {
            temp = temp.replace(results[0], vars[varName][i][results[1]]);
        }
       results = replace.exec(temp);                
    }
    bufs[i] = Buffer.from(temp);
}
return Buffer.concat(bufs);
```

我们的临时字符串包含我们认为是模板的所有数据，而`varName`变量告诉我们在我们传递给`processPattern`的对象中查找的位置以执行我们的替换策略。接下来，我们将使用正则表达式提取变量的名称。这个特定的正则表达式表示查找`${<name>}`模式，同时也表示捕获`<name>`部分的内容。这样我们就可以轻松地获取变量的名称。我们还将继续循环遍历模板，看看是否有更多的正则表达式符合这些条件。最后，我们将用我们存储的变量替换模板代码。

完成所有这些后，我们将所有这些缓冲区连接在一起并返回它们。这对于那段代码来说是很多的；幸运的是，我们的模板的`from`部分要容易处理得多。我们的模板代码的`from`部分只需从我们的`templateDirectory`变量中查找具有该名称的文件，并将其返回为缓冲形式。

它应该看起来像下面这样：

```js
case FIND: {
    const type = breakdown[1];
    const HTML = 'html';
    const CSS  = 'css';
    if(!(type === HTML || type === CSS)) return new Error("This is not a
     valid template type! " + breakdown[1]);
    return fs.readFileSync(path.join(templateDirectory, type, `${breakdown[2]}.${type}`));
}
```

首先，我们从第二个参数中获取文件类型。如果不是`HTML`或`CSS`文件，我们将拒绝它。否则，我们将尝试读取文件并将其发送到我们的流中。

你们中的一些人可能会想知道我们将如何处理其他文件中的模板。现在，如果我们在`main.html`文件上运行我们的系统，我们将得到所有单独的块，但我们的`sidebar.html`文件没有填充。这是我们模板系统的一个弱点。解决这个问题的一种方法是创建另一个函数，它将调用我们的`Transform`流一定次数。这将确保我们为这些单独的部分完成模板。让我们现在就创建这个函数。

这不是处理这个问题的唯一方法。相反，我们可以利用另一个系统：当我们在文件中看到模板指令时，我们将该缓冲区添加到需要处理的项目列表中。这将允许我们的流处理指令，而不是一遍又一遍地循环缓冲区。这会导致它自己的问题，因为有人可能会编写一个无限递归的模板，这将导致我们的流中断。一切都是一种权衡，现在，我们选择编码的简易性而不是使用的简易性。

首先，我们需要从`events`模块中导入`once`函数和从`stream`模块中导入`PassThrough`流。让我们现在更新这些依赖关系，就像这样：

```js
import { Transform, PassThrough } from 'stream'
import { once } from 'events'
```

接下来，我们将创建一个新的`Transform`流，它将带入与以前相同的信息，但现在，我们还将添加一个循环计数器。我们还将响应`transform`事件，并将其推送到一个私有变量，直到我们读取完整的起始模板为止，如下所示：

```js
export class LoopingStream extends Transform {
    #numberOfRolls = 1
    #data = []
    #dir = null
    #vars = null
    constructor(opts={}) {
        super(opts);
        if( 'loopAmount' in opts ) {
            this.#numberOfRolls = opts.loopAmount
        }
        if( opts.vars ) {
            this.#vars = opts.vars;
        }
        if( opts.dir) {
            this.#dir = opts.dir;
        }
    }
    _transform(chunk, encoding, cb) {
        this.#data.push(chunk);
        cb();
    }
    _flush(cb) {
    }
}
```

接下来，我们将使我们的`flush`事件`async`，因为我们将利用一个异步`for`循环，就像这样：

```js
async _flush(cb) {
    let tData = Buffer.concat(this.#data);
    let tempBuf = [];
    for(let i = 0; i < this.#numberOfRolls; i++) {
        const passThrough = new PassThrough();
        const templateBuilder = new TemplateBuilder({ templateDirectory :
        this.#dir, templateVariables : this.#vars });
        passThrough.pipe(templateBuilder);
        templateBuilder.on('data', (data) => {
            tempBuf.push(data);
        });
        passThrough.end(tData);
        await once(templateBuilder, 'end');
        tData = Buffer.concat(tempBuf);
        tempBuf = [];
    }
    this.push(tData);
    cb();
}
```

基本上，我们将把所有的初始模板数据放在一起。然后，我们将通过我们的`TemplateBuilder`运行这些数据，构建一个新的模板来运行。我们利用`await once(templateBuilder, ‘end')`系统让我们以同步的方式处理这段代码。一旦我们完成了计数，我们将输出数据。

我们可以使用旧的测试工具来测试这一点。让我们继续设置它来利用我们的新的`Transform`流，并将数据输出到文件，如下所示：

```js
const file = fs.createReadStream('./template/main.html');
const testOut = fs.createWriteStream('test.html');
const tStream = new LoopingStream({
    dir : templateDirectory,
    vars : { //removed for simplicity sake },
    loopAmount : 2
});
file.pipe(tStream).pipe(testOut);
```

如果我们现在运行这个，我们会注意到`test.html`文件包含了我们完全构建的`template`文件！我们现在有一个可以使用的模板系统。让我们把它连接到我们的服务器上。

# 设置我们的服务器

有了我们的模板系统工作，让我们继续把所有这些连接到我们的服务器上。现在不再简单地回复“一切正常！”，而是用我们的模板回复。我们可以通过运行以下代码轻松实现这一点：

```js
stream.respond({
        'content-type': 'text/html',
        ':status': 200
    });
    const file = fs.createReadStream('./template/main.html');
    const tStream = new LoopingStream({
        dir: templateDirectory,
        vars : { //removed for readability }
},
        loopAmount : 2
    })
    file.pipe(tStream).pipe(stream);
});
```

这应该几乎和我们的测试工具一模一样。如果我们现在转到`https://localhost:50000`，我们应该会看到一个非常基本的 HTML 页面，但我们已经创建了我们的模板文件！如果我们现在进入开发工具并查看源代码，我们会看到一些奇怪的东西。CSS 表明我们加载了我们的`main.css`文件，但文件的内容看起来和我们的 HTML 文件完全一样！

我们的服务器对每个请求都以我们的 HTML 文件进行响应！我们需要做的是一些额外的工作，让我们的服务器能够正确地响应请求。我们将通过将请求的 URL 映射到我们拥有的文件来实现这一点。为了简单起见，我们只会响应 HTML 和 CSS 请求（我们不会发送任何 JavaScript），但是这个系统可以很容易地添加返回类型的图片，甚至文件。我们将通过以下方式添加所有这些：

1.  我们将为我们的文件结尾设置一个查找表，就像这样：

```js
const FILE_TYPES = new Map([
    ['.css', path.join('.', templateDirectory, 'css')],
    ['.html', path.join('.', templateDirectory, 'html')]
]);
```

1.  接下来，我们将使用这个映射根据请求的`headers`来拉取文件，就像这样：

```js
const p = headers[':path'];
for(const [fileType, loc] of FILE_TYPES) {
    if( p.endsWith(fileType) ) {
        stream.respondWithFile(
            path.join(loc, path.posix.basename(p)),
            {
                'content-type': `text/${fileType.slice(1)}`,
                ':status': 200
            }
        );
        return;
    }     
}
```

基本思想是循环遍历我们支持的文件类型，看看我们是否有这些文件。如果有，我们将用文件进行响应，并通过`content-type`头告诉浏览器它是 HTML 文件还是 CSS 文件。

1.  现在，我们需要一种方法来判断请求是否良好。目前，我们可以转到任何 URL，我们将一遍又一遍地得到相同的响应。我们将利用`publishedDirectory`环境变量来实现这一点。根据其中的文件名，这些将是我们的端点。对于每个子 URL 模式，我们将寻找遵循相同模式的子目录。如下所示：

```js
https:localhost:50000/articles/1 maps to <publishedDirectory>/articles/1.md
```

`.md`扩展名表示它是一个 Markdown 文件。这就是我们将编写页面的方式。

1.  现在，让我们让这个映射工作。为此，我们将在我们的`for`循环下面放入以下代码：

```js
try {
    const f = fs.statSync(path.join('.', publishedDirectory, p));
    stream.respond({
        'content-type': 'text/html',
        ':status': 200
    });
    const file = fs.createReadStream('./template/main.html');
    const tStream = new LoopingStream({
        dir: templateDirectory,
        vars : { },
        loopAmount : 2
    })
    file.pipe(tStream).pipe(stream);
} catch(e) {
    stream.respond({
        'content-type': 'text/html',
        ':status' : 404
    });
    stream.end('File Not Found! Turn Back!');
    console.warn('following file requested and not found! ', p);
}
```

我们将用`try`/`catch`块包装我们查找文件的方法(`fs.statSync`)。如果出现错误，这通常意味着我们没有找到文件，我们将向用户发送一个`404`消息。否则，我们将发送我们一直发送的内容：我们的示例`template`。如果我们现在运行服务器，我们将收到以下消息：文件未找到！回头吧！我们在那个目录中什么都没有！

让我们继续创建目录，并添加一个名为`first.md`的文件。如果我们添加这个目录和文件并重新运行服务器，如果我们转到`https://localhost:50000/first`，我们仍然会收到错误消息！我们之所以会收到这个消息，是因为在检查文件时我们没有添加 Markdown 文件扩展名！让我们继续将其添加到`fs.statSync`检查中，如下所示：

```js
const f = fs.statSync(path.join('.', publishedDirectory, `${p}.md`));
```

现在，当我们重新运行服务器时，我们将看到以前的正常模板。如果我们向`first.md`文件添加内容，我们将得不到该文件。现在我们需要将此添加到我们的模板系统中。

还记得在本章开头我们添加了`npm`包`remarkable`吗？现在我们将添加 Markdown 渲染器`remarkable`，以及我们的模板语言将寻找的新关键字，以渲染 Markdown，如下所示：

1.  让我们将`Remarkable`作为一个导入添加到我们的`template.js`文件中，就像这样：

```js
import Remarkable from 'remarkable'
```

1.  我们将寻找以下指令来将 Markdown 文件包含到`<% file <filename> %>`模板中，就像这样：

```js
const processPattern = function(pattern, templateDir, publishDir, vars=null) {
    const process = pattern.toString('utf8').trim();
    const LOOP = "loop";
    const FIND = "from";
    const FILE = "file";
    const breakdown = process.split(' ');
    switch(breakdown[0]) {
      // previous case statements removed for readability
        case FILE: {
            const file = breakdown[1];
            return fs.readFileSync(path.join(publishDir, file));
        }
        default:
            return new Error("Process directory not found! " +  
             breakdown[0]);
    }
}
```

1.  现在，我们需要在构造函数中的`Transform`流的可能选项中添加`publishDir`变量，如下所示：

```js
export default class TemplateBuilder extends Transform {
    #pattern = []
    #publish = null
    constructor(opts={}) {
        super(opts);
        if( opts.publishDirectory ) {
            this.#publish = opts.publishDirectory;
        }
    }
    _transform(chunk, encoding, cb) {
        let location = 0;
        do {
            if(!this.#pattern.length && this.#pair.start === -1 ) {
                // code from before
            } else {
                if( this.#pair.start !== -1 ) {
                        this.push(processPattern(chunk.slice(this.#pair.start,
this.#pair.end), this.#template, this.#publish, this.#vars)); //add publish to our processPattern function
                } 
            } 
        } while( location !== -1 );
    }
}
```

**记住**：为了使其更易于阅读，这些示例中删除了大量代码。要获取完整的示例，请转到本书的代码存储库。

1.  创建一个`LoopingStream`类，它将循环并运行`TemplateBuilder`：

```js
export class LoopingStream extends Transform {
    #publish = null
    constructor(opts={}) {
        super(opts);
        if( opts.publish ) {
            this.#publish = opts.publish;
        }
    }
    async _flush(cb) {
        for(let i = 0; i < this.#numberOfRolls; i++) {
            const passThrough = new PassThrough();
            const templateBuilder = new TemplateBuilder({
                templateDirectory : this.#dir,
                templateVariables : this.#vars,
                publishDirectory  :this.#publish
            });
        }
        cb();
    }
}
```

1.  我们需要使用以下模板化行更新我们的模板：

```js
<!DOCTYPE html>
<html>
    <head>
        <link rel="stylesheet"  type="text/css" href="css/main.css" />
    </head>
    <body>
        <% from html header %>
        <% from html sidebar %>
        <% file first.md %>
        <% from html footer %>
    </body>
</html>
```

1.  最后，我们需要将`publish`目录传递给服务器的流。我们可以通过以下代码进行此操作：

```js
const tStream = new LoopingStream({
        dir: templateDirectory,
        publish: publishedDirectory,
        vars : {
}});
```

有了所有这些，我们应该从服务器那里得到一些不仅仅是我们的基本模板。如果我们向文件中添加了一些 Markdown，我们应该只看到带有我们模板的 Markdown。现在我们需要确保这个 Markdown 被处理。让我们回到我们的转换方法，并调用`Remarkable`方法，以便它处理 Markdown 并以 HTML 的形式返回给我们，如下面的代码块所示：

```js
const MarkdownRenderer = new Remarkable.Remarkable();
const processPattern = function(…) {
      switch(breakdown[0]) {
            case FILE: {
                  const file = breakdown[1];
                  const html =
MarkdownRenderer.render(fs.readfileSync(path.join(publishDir, file)
).toString('utf8'));
            return Buffer.from(html);
            }
      }
}
```

通过这个改变，我们现在有了一个通用的 Markdown 解析器，它使我们能够获取我们的模板文件，并将它们与我们的`main.html`文件一起发送。为了使模板系统和静态服务器正常运行，我们需要做的最后一个改变是，确保`main.html`文件不再具有精确的模板，而是具有我们想要的指令状态，以便在那里放置一个文件，并且我们的模板系统将放置在我们流构造函数中声明的文件。我们可以通过以下更改轻松实现这一点：

1.  在我们的`template.js`文件中，我们将利用一个名为`fileToProcess`的独特变量。我们以与我们通过传递的`vars`获取`sidebar.html`文件要处理的变量相同的方式获取它。如果我们没有来自`fileToProcess`变量的文件，我们将利用我们在`template`指令的第二部分中拥有的文件，如下面的代码块所示：

```js
case FILE: {
    const file = breakdown[1];
    const html =
    MarkdownRenderer.render(fs.readFileSync(path.join(publishDir,  
    vars.fileToProcess || file)).toString('utf8'));
    return Buffer.from(html);
}
```

1.  我们需要将这个变量从我们的服务器传递到流中，就像这样：

```js
const p = headers[':path'];
const tStream = new LoopingStream({
    dir: templateDirectory,
    publish: publishedDirectory,
    vars : {
        articles : [ ],
        fileToProcess : `${p}.md`
    },
    loopAmount : 2
});
```

1.  我们将进行的最后一个改变是改变`html`文件，为我们没有的页面创建一个新的基本 Markdown 文件。这可以让我们为根 URL 创建一个基本页面。我们不会实现这一点，但这是我们可以这样做的一种方式：

```js
<body>
    <% from html header %>
    <% from html sidebar %>
    <% file base.md %>
    <% from html footer %>
</body>
```

有了这个改变，如果我们现在运行我们的服务器，我们就有了一个完全功能的模板系统，支持 Markdown！这是一个了不起的成就！然而，我们需要向我们的服务器添加两个功能，以便它能够处理更多的请求并快速处理相同的请求。这些功能是缓存和集群。

# 添加缓存和集群

首先，我们将通过向我们的服务器添加缓存来开始。我们不希望不断重新编译我们以前已经编译过的页面。为此，我们将实现一个围绕地图的类。这个类将同时跟踪 10 个文件。我们还将实现文件上次使用的时间戳。当我们达到第十一个文件时，我们将看到它不在缓存中，并且我们已经达到了我们可以在缓存中保存的文件的最大数量。我们将用时间戳最早的文件替换编译后的页面。

这被称为**最近最少使用**（**LRU**）缓存。还有许多其他类型的缓存策略，比如**生存时间**（**TTL**）缓存。这种缓存类型将消除在缓存中时间过长的文件。这是一种很好的缓存类型，当我们一遍又一遍地使用相同的文件，但当服务器有一段时间没有被访问时，我们最终希望释放空间。LRU 缓存将始终保留这些文件，即使服务器已经有好几个小时没有被访问。我们可以实现两种缓存策略，但现在我们只实现 LRU 缓存。

首先，我们将创建一个名为`cache.js`的新文件。在这里，我们将执行以下操作：

1.  创建一个新的类。我们不需要扩展任何其他类，因为我们只是在 JavaScript 内置的`Map`数据结构周围编写一个包装器，如下面的代码块所示：

```js
export default class LRUCache {
    #cache = new Map()
}
```

1.  然后我们将有一个构造函数，它将接受我们想要在缓存中存储的文件数量，然后使用我们的策略来替换其中一个文件，就像这样：

```js
#numEntries = 10
constructor(num=10) {
    this.#numEntries = num
}
```

1.  接下来，我们将向我们的缓存添加`add`操作。它将接受我们页面的缓冲形式和我们用来获取它的 URL。键将是 URL，值将是我们页面的缓冲形式，如下面的代码块所示：

```js
add(file, url) {
    const val = {
        page : file,
        time : Date.now()
    }
    if( this.#cache.size === this.#numEntries ) {
        // do something
        return;
    }
    this.#cache.set(url, val);
}
```

1.  然后，我们将实现`get`操作，通过它我们尝试根据 URL 获取文件。如果我们没有它，我们将返回`null`。如果我们检索到一个文件，我们将更新时间，因为这将被视为最新的页面抓取。如下所示：

```js
get(url) {
    const val = this.#cache.get(url);
    if( val ) {
        val.time = Date.now();
        this.#cache.set(url, val);
        return val.page;
    }
    return null;
}
```

1.  现在，我们可以更新我们的`add`方法的`if`语句。如果我们达到了限制，我们将遍历我们的地图，看看最短的时间是什么。我们将删除那个文件，并用新创建的文件替换它，就像这样：

```js
if( this.#cache.size === this.#numEntries ) {
    let top = Number.MAX_VALUE;
    let earliest = null;
    for(const [key, val] of this.#cache) {
        if( val.time < top ) {
            top = val.time;
            earliest = key;
        }
    }
    this.#cache.delete(earliest);
}
```

现在我们已经为我们的文件建立了一个基本的 LRU 缓存。要将其附加到我们的服务器上，我们需要将其放在我们的管道中间：

1.  让我们回到主文件并导入这个文件：

```js
import cache from './cache.js'
const serverCache = new cache();
```

1.  现在我们将稍微改变我们的流处理程序中的逻辑。如果我们注意到 URL 是我们在缓存中有的东西，我们将只是获取数据并将其传送到我们的响应中。否则，我们将编译模板，将其设置在我们的缓存中，并将编译后的版本传送下来，就像这样：

```js
const cacheHit = serverCache.get(p);
if( cacheHit ) {
    stream.end(cacheHit);
} else {
    const file = fs.createReadStream('./template/main.html');
    const tStream = new LoopingStream({
        dir: templateDirectory,
        publish: publishedDirectory,
        vars : { /* shortened for readability */ },
        loopAmount : 2
    });
    file.pipe(tStream);
    tStream.once('data', (data) => {
        serverCache.add(data, p);
        stream.end(data);
    });
}
```

如果我们尝试运行上述代码，我们现在将看到如果我们两次访问相同的页面，我们将从缓存中获取文件；如果我们第一次访问它，它将通过我们的模板流进行编译，然后将其设置在缓存中。

1.  为了确保我们的替换策略有效，让我们将缓存的大小设置为只有`1`，看看如果我们访问一个新的 URL，我们是否不断替换文件，如下所示：

```js
const serverCache = new cache(1);
```

如果我们现在在每个方法被调用时记录我们的缓存，我们将看到当我们访问新页面时，我们正在替换文件，但如果我们停留在同一个页面，我们只是发送缓存的文件回去。

现在我们已经添加了缓存，让我们在服务器上再添加一个部分，这样我们就可以处理大量的连接。我们将添加`cluster`模块，就像我们在第六章中所做的那样，*消息传递-了解不同类型*。我们将按照以下步骤进行：

1.  让我们在`main.js`文件中导入`cluster`模块：

```js
import cluster from 'cluster'
```

1.  现在我们将在主进程中初始化服务器。对于其他进程，我们将处理请求。

1.  现在，让我们改变策略，处理子进程内部的传入请求，就像这样：

```js
if( cluster.isMaster ) {
    const numCpus = os.cpus().length;
    for(let i = 0; i < numCpus; i++) {
        cluster.fork();
    }
    cluster.on('exit', (worker, code, signal) => {
        console.log(`worker ${worker.process.pid} died`);
    });
} else {
    const serverCache = new cache();
    // all previous server logic
}
```

通过这个单一的改变，我们现在可以在四个不同的进程之间处理请求。就像我们在第六章中学到的那样，*消息传递-了解不同类型*，我们可以为`cluster`模块共享一个端口。

# 总结

虽然还有一个部分需要添加（将我们的侧边栏连接到实际文件），但这应该是一个非常通用的模板服务器。需要做的就是修改我们的`FILE`模板，并将其连接到我们模板系统的侧边栏。通过我们对 Node.js 的学习，我们应该能够处理几乎任何类型的服务器端应用程序。我们还应该能够理解像 Express 这样的 Web 服务器是如何从这些基本构建块中创建的。

从这里，我们将回到浏览器，并将书中这部分学到的一些概念应用到接下来的几章中。我们将首先看一下浏览器中的工作线程，即专用工作线程。然后我们将看一下共享工作线程，以及我们如何从这些工作线程中获益，但仍然能够从中获取数据。最后，我们将看一下服务工作者，并看看它们如何帮助我们进行各种优化，比如在浏览器中进行缓存。
