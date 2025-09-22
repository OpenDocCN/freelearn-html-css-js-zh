# 数据存储和检索

在前两章中，我们构建了一个小型且有一定用途的应用程序来存储笔记，然后使其在移动设备上工作。虽然应用程序工作得相当好，但它并没有在任何长期基础上存储那些笔记，这意味着当你停止服务器时，笔记就会丢失，如果你运行多个笔记实例，每个实例都有自己的笔记集。典型的下一步是引入数据库层。

在本章中，我们将探讨 Node.js 中的数据库支持，这样用户在任何访问的笔记实例中都能看到相同的笔记集，并且能够可靠地存储笔记以供长期检索。

我们将从上一章中使用的*笔记*应用程序代码开始。我们从一个简单的、使用数组存储笔记的内存数据模型开始，然后使其适应移动设备。在本章中，我们将：

+   发现日志操作和调试信息

+   开始使用 ES6 模块格式

+   使用多个数据库引擎实现笔记对象的持久化

让我们开始吧！

第一步是复制上一章中的代码。例如，如果你在`chap06/notes`中工作，将其复制到`chap07/notes`。

# 数据存储和异步代码

根据定义，外部数据存储系统在 Node.js 架构中需要异步代码。从磁盘、另一个进程或数据库中检索数据访问时间总是足够长，以至于需要延迟执行。

现有的`Notes`数据模型是一个内存数据存储。从理论上讲，内存数据访问不需要异步代码，因此，现有的模型模块可以使用常规函数而不是`async`函数。

我们知道笔记必须迁移到使用数据库，并且需要异步 API 来访问笔记数据。因此，现有的笔记模型 API 使用`async`函数，这样我们就可以在本章中将笔记数据持久化到数据库中。

# 日志

在我们进入数据库之前，我们必须解决一个高质量软件系统的一个属性：管理日志信息，包括正常系统活动、系统错误和调试信息。日志使我们能够深入了解系统的行为。它获得了多少流量？如果是一个网站，人们最多访问哪些页面？发生了多少错误，以及错误的类型是什么？是否发生了攻击？是否发送了格式不正确的请求？

日志管理也是一个问题。日志轮转意味着定期将日志文件移开，以便开始一个新的日志文件。你应该处理日志数据以生成报告。对安全漏洞的筛查必须具有高优先级。

十二因素应用程序模型建议简单地将日志信息发送到控制台，然后其他软件系统捕获该输出并将其导向日志服务。遵循他们的建议可以通过减少可能出错的事物来降低系统复杂性。在后面的章节中，我们将使用 PM2 来实现这个目的。

让我们先完成对笔记中当前信息日志的浏览。

当我们使用 Express Generator 最初创建 *Notes* 应用程序时，它配置了一个使用 `morgan` 的活动日志系统：

```js
const logger = require('morgan'); 
.. 
app.use(logger('dev')); 
```

这是在终端窗口中打印请求的内容。有关更多信息，请访问 [`github.com/expressjs/morgan`](https://github.com/expressjs/morgan)。

在内部，Express 使用 **Debug** 包进行调试跟踪。您可以使用 `DEBUG` 环境变量来开启这些功能。我们应该尽量在我们的应用程序代码中使用这个包。更多信息，请访问 [`www.npmjs.com/package/debug`](https://www.npmjs.com/package/debug)。

最后，应用程序可能会生成未捕获的异常。需要捕获、记录并适当处理`uncaughtException`错误。

# 使用 Morgan 进行请求日志记录

Morgan 包有两个主要的配置区域：

+   日志格式

+   日志位置

目前，Notes 使用的是 `dev` 格式，它被描述为一种简洁的状态输出，旨在为开发者使用。这可以用来记录网络请求，作为衡量网站活动和受欢迎程度的一种方式。Apache 日志格式已经拥有一个庞大的报告工具生态系统，而且确实，Morgan 可以生成这种格式的日志文件。

要更改格式，只需更改`app.js`中的这一行：

```js
app.use(logger(process.env.REQUEST_LOG_FORMAT || 'dev')); 
```

然后按照以下方式运行 *Notes*：

```js
$ REQUEST_LOG_FORMAT=common npm start

> notes@0.0.0 start /Users/david/chap07/notes
> node ./bin/www
::1 - - [12/Feb/2016:05:51:21 +0000] "GET / HTTP/1.1" 304 -
::1 - - [12/Feb/2016:05:51:21 +0000] "GET /vendor/bootstrap/css/bootstrap.min.css HTTP/1.1" 304 -
::1 - - [12/Feb/2016:05:51:21 +0000] "GET /stylesheets/style.css HTTP/1.1" 304 -
::1 - - [12/Feb/2016:05:51:21 +0000] "GET /vendor/bootstrap/js/bootstrap.min.js HTTP/1.1" 304 -  
```

要恢复到之前的日志输出，只需不设置此环境变量。如果您查看过 Apache 访问日志，这种日志格式看起来会熟悉。行首的 `::1` 表示法是 `localhost` 的 IPV6 表示法，您可能更熟悉的是 `127.0.0.1`。

我们可以在请求日志记录上宣布胜利，然后转向调试消息。然而，让我们看看如何将日志直接记录到文件中。虽然可以通过一个单独的进程捕获 `stdout`，但 Morgan 已经安装在 Notes 中，并且它确实提供了将输出直接导向文件的能力。

Morgan 文档建议如下：

```js
// create a write stream (in append mode) 
var accessLogStream = fs.createWriteStream(__dirname + '/access.log', {flags: 'a'}) 

// setup the logger 
app.use(morgan('combined', {stream: accessLogStream})); 
```

但这有一个问题；在不杀死和重新启动服务器的情况下，无法执行日志轮转。因此，我们将使用他们的 `rotating-file-stream` 包。

首先，安装该包：

```js
$ npm install rotating-file-stream --save  
```

然后我们将此代码添加到`app.js`中：

```js
const fs = require('fs-extra');
...
const rfs = require('rotating-file-stream');
var logStream;
// Log to a file if requested
if (process.env.REQUEST_LOG_FILE) {
  (async () => {
    let logDirectory = path.dirname(process.env.REQUEST_LOG_FILE); 
    await fs.ensureDir(logDirectory);
    logStream = rfs(process.env.REQUEST_LOG_FILE, {
        size: '10M',     // rotate every 10 MegaBytes written
        interval: '1d',  // rotate daily
        compress: 'gzip' // compress rotated files
    });
  })().catch(err => { console.error(err); });
}
.. 
app.use(logger(process.env.REQUEST_LOG_FORMAT || 'dev', { 
    stream: logStream ? logStream : process.stdout 
})); 
```

在这里，我们使用环境变量 `REQUEST_LOG_FILE` 来控制是否将日志发送到 `stdout` 或文件。日志可以放入目录中，如果目录不存在，代码将自动创建该目录。通过使用 `rotating-file-stream` ([`www.npmjs.com/package/rotating-file-stream`](https://www.npmjs.com/package/rotating-file-stream))，我们确保了日志文件轮转，无需额外的系统。

使用 `fs-extra` 模块是因为它向 `fs` 模块添加了基于 Promise 的函数 ([`www.npmjs.com/package/fs-extra`](https://www.npmjs.com/package/fs-extra))。在这种情况下，`fs.ensureDir` 检查指定的目录结构是否存在，如果不存在，则创建目录路径。

# 调试信息

通过这种方式运行 *Notes*，你可以生成 Express 所做的详细跟踪：

```js
$ DEBUG=express:* npm start  
```

如果你想调试 Express，这非常有用。但，我们也可以在我们的代码中使用它。这类似于插入 `console.log` 语句，但无需记住要注释掉调试代码。

启用模块中的调试非常简单：

```js
const debug = require('debug')('module-name'); 
.. 
debug('some message'); 
.. 
debug(`got file ${fileName}`); 
```

# 捕获 stdout 和 stderr

重要信息可以打印到 `process.stdout` 或 `process.stderr`，如果未捕获该输出，则可能会丢失。十二要素模型建议使用系统功能来捕获这些输出流。在笔记中，我们将使用 PM2 来实现这一目的，这将在第十章 “部署 Node.js 应用程序” 中介绍。

`logbook` 模块 ([`github.com/jpillora/node-logbook`](https://github.com/jpillora/node-logbook)) 在捕获 `process.stdout` 和 `process.stderr` 方面提供了一些有用的功能，并且可以将输出发送到有用的位置。

# 未捕获的异常

未捕获的异常是另一个可能会丢失重要信息的地方。在 *Notes* 应用程序中修复这个问题很容易：

```js
const error = require('debug')('notes:error'); 

process.on('uncaughtException', function(err) { 
  error("I've crashed!!! - "+ (err.stack || err)); 
}); 
.. 
if (app.get('env') === 'development') { 
  app.use(function(err, req, res, next) { 
    // util.log(err.message); 
    res.status(err.status || 500); 
    error((err.status || 500) +' '+ error.message); 
    res.render('error', { 
      message: err.message, 
      error: err 
    }); 
  }); 
} 
.. 
app.use(function(err, req, res, next) { 
  // util.log(err.message); 
  res.status(err.status || 500); 
  error((err.status || 500) +' '+ error.message); 
  res.render('error', { 
    message: err.message, 
    error: {} 
  }); 
}); 
```

`debug` 包有一个我们遵循的约定。对于具有多个模块的应用程序，所有调试对象都应该使用命名模式 `app-name:module-name`。在这种情况下，我们使用了 `notes:error`，它将用于所有错误消息。我们也可以使用 `notes:memory-model` 或 `notes:mysql-model` 来调试不同的模型。

当我们设置未捕获异常的处理程序时，将错误记录添加到错误处理程序中也是一个好主意。

# 未处理的 Promise 拒绝

使用 Promise 和 `async` 函数自动将错误引导到有用的方向。错误会导致 Promise 转换为 *拒绝* 状态，这最终必须在 `.catch` 方法中处理。由于我们都是人类，我们难免会忘记确保所有代码路径都处理了它们的拒绝的 Promise。

目前，如果 Node.js 检测到未处理的 Promise 拒绝，它会打印以下警告：

```js
(node:4796) UnhandledPromiseRejectionWarning: Unhandled promise rejection 
```

警告继续说明，未处理的 Promise 拒绝的默认处理程序已被弃用，并且此类 Promise 拒绝将使 Node 进程崩溃而不是打印此消息。在这种情况下，内置的`process`模块确实会发出事件，因此添加处理程序很容易：

```js
import util from 'util';
...
process.on('unhandledRejection', (reason, p) => {
  error(`Unhandled Rejection at: ${util.inspect(p)} reason: ${reason}`);
});
```

至少，我们可以打印如下错误消息：

```js
notes:error Unhandled Rejection at: Promise {
  notes:error <rejected> TypeError: model(...).keylist is not a function
  ... full stack trace
} reason: TypeError: model(...).keylist is not a function +3s
```

# 使用 ES6 模块格式

我们使用 CommonJS 模块编写了*笔记*应用程序，这是传统的 Node.js 模块格式。虽然应用程序可以继续使用该格式，但 JavaScript 社区已经选择在浏览器和 Node.js 代码中切换到 ES6 模块，因此切换到 ES6 模块非常重要，这样我们就可以使用统一的模块格式。让我们使用 ES6 模块重写应用程序，然后为任何新添加的内容编写 ES6 模块。

需要的更改很大，需要将`require`语句替换为`import`语句，并将文件从`foo.js`重命名为`foo.mjs`。让我们开始吧。

# 将 app.js 重写为 ES6 模块

让我们从`app.js`开始，将其名称更改为`app.mjs`：

```js
$ mv app.js app.mjs
```

将顶部的`require`语句块更改为以下内容：

```js
import fs from 'fs-extra';
import url from 'url';
import express from 'express';
import hbs from 'hbs';
import path from 'path';
import util from 'util';
import favicon from 'serve-favicon';
import logger from 'morgan';
import cookieParser from 'cookie-parser';
import bodyParser from 'body-parser';
import DBG from 'debug';
const debug = DBG('notes:debug'); 
const error = DBG('notes:error'); 
import { router as index } from './routes/index';
// const users = require('./routes/users');
import { router as notes } from './routes/notes'; 

// Workaround for lack of __dirname in ES6 modules
const __dirname = path.dirname(new URL(import.meta.url).pathname);

const app = express();

import rfs from 'rotating-file-stream';
```

然后，在脚本的底部进行以下更改：

```js
export default app;
```

让我们简要谈谈这里提到的解决方案。Node.js 自动注入到 CommonJS 模块中的有几个全局变量。这些变量不受 ES6 模块支持。对于*笔记*应用程序来说，关键的变量是`__dirname`，它在`app.mjs`中多处使用。这里显示的代码更改包括基于 Node.js 10.x 开始可用的全新 JavaScript 功能的一个解决方案，即`import.meta.url`变量。

`import.meta`对象旨在将有用信息注入到 ES6 模块中。正如其名所示，`import.meta.url`变量包含描述模块从何处加载的 URL。对于 Node.js 来说，目前，ES6 模块只能从本地文件系统上的`file://` URL 加载。这意味着，如果我们提取该 URL 的`pathname`，我们可以轻松计算出包含模块的目录，如下所示。

为什么选择这个解决方案？为什么不使用以`./`开头的路径名？答案是，`./`文件名是相对于进程的当前工作目录进行评估的。这个目录通常与正在执行的 Node.js 模块所在的目录不同。因此，Node.js 团队添加`import.meta.url`功能是非常方便的。

在大多数情况下，遵循以下更改模式：

```js
const moduleName = require('moduleName');  // in CommonJS modules
import moduleName from 'moduleName';       // in ES6 modules
```

记住，Node.js 在 ES6 和 CommonJS 模块中都使用相同的模块查找算法。Node.js 的`require`语句是同步的，这意味着在`require`完成时，它已经执行了模块并返回了其`module.exports`。相比之下，ES6 模块是异步的，这意味着模块可能还没有完成加载，你可以只导入所需的模块部分。

这里显示的大多数模块导入是为了在 `node_modules` 目录中安装的常规 Node.js 模块，其中大多数是 CommonJS 模块。使用 `import` 与 CommonJS 模块的规则是，`module.exports` 对象被处理为如果它是默认导出。前面显示的 `import` 语句将默认导出（或 `module.exports` 对象）命名为 `import` 语句中所示。对于以这种方式导入的 CommonJS 模块，您将像在 CommonJS 上下文中一样使用它，即 `moduleName.functionName()`。

`debug` 模块的使用实际上是相同的，但编码方式不同。在 CommonJS 上下文中，我们被告知如下使用该模块：

```js
const debug = require('debug')('notes:debug'); 
const error = require('debug')('notes:error'); 
```

换句话说，该模块的 `module.exports` 是一个函数，我们立即调用它。ES6 模块没有使用 `debug` 模块的那种语法的语法。因此，我们必须像下面那样将其拆分，并显式调用该函数。

讨论的最后一个要点是两个路由模块的 `import`。最初尝试让这些模块将 `router` 作为默认值导出，但 Express 在那种情况下抛出了错误。因此，我们将重写这些模块，将 `router` 作为命名导出，然后像下面这样使用它。

# 将 `bin/www` 重写为 ES6 模块

记住 `bin/www` 是一个用于启动应用程序的脚本。它被编写为一个 CommonJS 脚本，但由于 `app.mjs` 现在是一个 ES6 模块，因此 `bin/www` 也必须被重写为 ES6 模块。在编写本文时，CommonJS 模块不能导入 ES6 模块。

更改文件名：

```js
$ mv bin/www bin/www.mjs
```

然后，在顶部，将 `require` 语句更改为 `import` 语句：

```js
import app from '../app.mjs';
import DBG from 'debug';
const debug = DBG('notes:server-debug'); 
const error = DBG('notes:server-error'); 
import http from 'http';
```

我们已经讨论了这里的一切，除了 `app.mjs` 将其 `app` 对象作为默认导出。因此，我们像下面这样使用它。

# 将模型代码重写为 ES6 模块

模型目录包含两个模块：`Note.js` 定义了 `Note` 类，而 `notes-memory.js` 包含一个内存中的数据模型。这两个模块都很容易转换为 ES6 模块。

更改文件名：

```js
$ cd models
$ mv Note.js Note.mjs
$ mv notes-memory.js notes-memory.mjs
```

在 `Note.mjs` 中，只需进行以下更改：

```js
export default class Note {
  ...
}
```

这使得 `Note` 类成为默认导出。

然后，在 `notes-memory.mjs` 中进行以下更改：

```js
import Note from './Note';

var notes = [];

async function crupdate(key, title, body) {
  notes[key] = new Note(key, title, body);
  return notes[key];
}

export function create(key, title, body) { return crupdate(key, title, body); }
export function update(key, title, body) { return crupdate(key, title, body); }

export async function read(key) {
  if (notes[key]) return notes[key];
  else throw new Error(`Note ${key} does not exist`);
}

export async function destroy(key) {
  if (notes[key]) {
    delete notes[key];
  } else throw new Error(`Note ${key} does not exist`);
}

export async function keylist() { return Object.keys(notes); }
export async function count() { return notes.length; }
export async function close() { }

```

这是对将函数分配给 `module.exports` 并使用命名导出的直接转写。

通过将 `Note` 类定义为 `Note.mjs` 模块的默认导出，它可以很好地导入使用该类的任何模块。

# 将路由模块重写为 ES6 模块

`routes` 目录包含两个路由模块。目前，每个路由模块创建一个 `router` 对象，向该对象添加路由函数，然后将它分配给 `module.exports` 字段。这表明我们应该将 `router` 作为默认导出，但如我们之前所说，这并没有成功。相反，我们将 `router` 作为命名导出。

更改文件名：

```js
$ cd routes
$ mv index.js index.mjs
$ mv notes.js notes.mjs
```

然后，在每个文件的顶部，将 `require` 语句块更改为以下内容：

```js
import util from 'util';
import express from 'express';
import * as notes from '../models/notes-memory';

export const router = express.Router();
```

这两个文件将会一样。然后在每个文件的底部，删除将 `router` 赋值给 `module.exports` 的那行代码。

让我们转向 `app.mjs` 并更改导入路由模块的方式。

因为 `router` 是一个命名导出，默认情况下你会在 `app.mjs` 中如下导入 `router` 对象：

```js
import { router } from './routes/index';
```

但然后我们会遇到冲突，因为这两个模块都定义了一个 `router` 对象。相反，我们使用 `as` 子句更改了这个对象的名称：

```js
import { router as index } from './routes/index';
import { router as notes } from './routes/notes'; 
```

每个模块的 `router` 对象因此被赋予了合适的名称。

# 在文件系统中存储笔记

文件系统是一个经常被忽视的数据库引擎。虽然文件系统没有数据库引擎支持的查询功能，但它们是存储文件的可靠位置。笔记模式足够简单，以至于文件系统可以轻松地作为其数据存储层。

让我们从向 `Note.mjs` 添加一个函数开始：

```js
export default class Note {
   ...
   get JSON() { 
      return JSON.stringify({ 
        key: this.key, title: this.title, body: this.body 
      }); 
   }

   static fromJSON(json) { 
       var data = JSON.parse(json); 
       var note = new Note(data.key, data.title, data.body); 
       return note; 
   } 
}
```

`JSON` 是一个获取器，这意味着它获取对象的值。在这种情况下，`note.JSON` 属性/获取器，没有括号，将简单地给出 Note 的 JSON 表示形式。我们将在写入 JSON 文件时使用这个。

`fromJSON` 是一个静态函数，或者工厂方法，用于在拥有 JSON 字符串的情况下辅助构建 `Note` 对象。区别在于 `JSON` 与 `Note` 类的实例相关联，而 `fromJSON` 与类本身相关联。它们可以如下使用：

```js
const note = new Note("key", "title", "body"); 
const json = note.JSON;    // produces JSON text
const newnote = Note.fromJSON(json); // produces new Note instance
```

现在，让我们创建一个新的模块，`models/notes-fs.mjs`，来保存文件系统模型：

```js
import fs from 'fs-extra';
import path from 'path';
import util from 'util';
import Note from './Note';
import DBG from 'debug';
const debug = DBG('notes:notes-fs');
const error = DBG('notes:error-fs');

async function notesDir() { 
    const dir = process.env.NOTES_FS_DIR || "notes-fs-data"; 
    await fs.ensureDir(dir);
    return dir;
} 

function filePath(notesdir, key) { return path.join(notesdir, `${key}.json`); } 

async function readJSON(notesdir, key) { 
    const readFrom = filePath(notesdir, key); 
    var data = await fs.readFile(readFrom, 'utf8');
    return Note.fromJSON(data);
}
```

`notesDir` 函数将在 `notes-fs` 中被使用，以确保目录存在。为了简化这个过程，我们使用了 `fs-extra` 模块，因为它向 `fs` 模块添加了基于 Promise 的函数（[`www.npmjs.com/package/fs-extra`](https://www.npmjs.com/package/fs-extra)）。在这种情况下，`fs.ensureDir` 验证指定的目录结构是否存在，如果不存在，则创建目录路径。

环境变量 `NOTES_FS_DIR` 配置了一个存储笔记的目录。我们将为每篇笔记创建一个文件，并将笔记存储为 JSON 格式。如果没有指定环境变量，我们将回退到使用 `notes-fs-data` 作为目录名称。

因为我们要添加另一个依赖：

```js
$ npm install fs-extra --save 
```

每个数据文件的文件名是带有 `.json` 后缀的 `key`。这带来一个限制，即文件名不能包含 `/` 字符，因此我们使用以下代码进行测试：

```js
async function crupdate(key, title, body) { 
    var notesdir = await notesDir();
    if (key.indexOf('/') >= 0) 
        throw new Error(`key ${key} cannot contain '/'`); 
    var note = new Note(key, title, body); 
    const writeTo = filePath(notesdir, key); 
    const writeJSON = note.JSON; 
    await fs.writeFile(writeTo, writeJSON, 'utf8');
    return note;
}

export function create(key, title, body) { return crupdate(key, title, body); }
export function update(key, title, body) { return crupdate(key, title, body); }
```

与 `notes-memory` 模块一样，`create` 和 `update` 函数使用完全相同的代码。`notesDir` 函数用于确保目录存在，然后我们创建一个 `Note` 对象，然后将数据写入文件。

注意代码因为使用了 `async` 函数而非常直接。我们不需要检查错误，因为它们将被 `async` 函数自动捕获并传递给我们的调用者：

```js
export async function read(key) { 
    var notesdir = await notesDir();
    var thenote = await readJSON(notesdir, key);
    return thenote; 
}
```

使用 `readJSON` 从磁盘读取文件。它已经生成了 `Note` 对象，所以我们只需返回该对象即可：

```js
export async function destroy(key) { 
    var notesdir = await notesDir();
    await fs.unlink(filePath(notesdir, key)); 
}
```

`fs.unlink`函数删除我们的文件。因为这个模块使用文件系统，所以删除文件就是删除`note`对象所必需的：

```js
export async function keylist() { 
    var notesdir = await notesDir();
    var filez = await fs.readdir(notesdir);
    if (!filez || typeof filez === 'undefined') filez = []; 
    var thenotes = filez.map(async fname => { 
        var key = path.basename(fname, '.json');
        var thenote = await readJSON(notesdir, key);
        return thenote.key; 
    }); 
    return Promise.all(thenotes); 
}
```

`keylist`的合约是返回一个 Promise，该 Promise 将解析为现有笔记对象的键数组。由于它们作为单独的文件存储在`notesdir`中，我们必须读取该目录中的每个文件来检索其键。

`Array.map`从一个现有的数组（即由`fs.readdir`返回的文件名数组）构造一个新的数组。构造数组中的每个条目都是一个`async`函数，它读取笔记，返回`key`：

```js
export async function count() { 
    var notesdir = await notesDir();
    var filez = await fs.readdir(notesdir); 
    return filez.length;
}

export async function close() { }
```

计算笔记的数量只是简单地计算`notesdir`中文件的数量。

# ES6 模块的动态导入

在我们开始修改路由函数之前，我们必须考虑如何处理多个模型。我们目前有两个数据模型模块，`notes-memory`和`notes-fs`，并且我们将在本章结束时实现更多。我们需要一个简单的方法来选择正在使用的模型。

有几种可能的方法可以做到这一点。例如，在一个 CommonJS 模块中，可以这样做：

```js
const path  = require('path'); 
const notes = require(process.env.NOTES_MODEL  
                  ? path.join('..', process.env.NOTES_MODEL)  
                  : '../models/notes-memory'); 
```

这让我们可以设置一个环境变量`NOTES_MODEL`来选择用于数据模型的模块。

这种方法不适用于常规的`import`语句，因为`import`语句中的模块名不能是这种表达式。现在 Node.js 中的动态导入功能确实提供了一个类似于刚才显示的片段的机制。

动态导入是一个返回 Promise 的`import()`函数，该 Promise 将解析为导入的模块。作为一个返回 Promise 的函数，`import()`在模块的顶层代码中不会很有用。但是，考虑以下情况：

```js
var NotesModule;

async function model() {
  if (NotesModule) return NotesModule;
  NotesModule = await import(`../models/notes-${process.env.NOTES_MODEL}`);
  return NotesModule;
}

export async function create(key, title, body) { 
    return (await model()).create(key, title, body); 
}
export async function update(key, title, body) { 
    return (await model()).update(key, title, body); 
}
export async function read(key) { return (await model()).read(key); }
export async function destroy(key) { return (await model()).destroy(key); }
export async function keylist() { return (await model()).keylist(); }
export async function count() { return (await model()).count(); }
export async function close() { return (await model()).close(); }
```

将该模块保存在文件`models/notes.mjs`中。该模块实现了我们将用于所有笔记模型模块的相同 API。`model()`函数是动态选择基于环境变量的笔记模型实现的关键。

这是一个`async`函数，因此它的返回值是一个 Promise。该 Promise 的值是`import()`加载的所选模块。因为`import()`返回一个 Promise，所以我们使用`await`来知道它是否正确加载。

每个 API 方法都遵循以下模式：

```js
export async function methodName(args) { 
    return (await model()).methodName(args); 
}
```

因为`model()`返回一个 Promise，所以最简洁的方法是使用一个`async`函数，并使用`await`来解析 Promise。一旦 Promise 解析完成，我们只需调用`methodName`函数并继续我们的业务。否则，那些 API 方法函数将如下所示：

```js
export function methodName(args) {
    return model().then(notes => { return notes.*methodName*(args); });
}
```

这两种实现是等效的，并且很明显哪种更简洁。

在所有这些对`async`函数返回的 Promise 的`await`等待中，讨论开销是值得的。最坏的情况是在第一次调用`model()`时，因为选定的笔记模型尚未加载。第一次调用时，调用流程如下：

+   API 方法调用`model()`，然后调用`import()`，然后`await`模块以完成加载

+   API 方法`await`等待`model()`返回的 Promise，获取模块对象，然后调用 API 函数

+   调用者也在使用`await`来接收最终结果

第一次运行时，时间主要被`import()`加载模块的过程所占据。在后续调用中，模块已经被加载，第一步就是简单地形成一个包含模块的已解析的 Promise。然后 API 方法可以快速地委托给实际的 API 方法。

要使用这个功能，在`routes/index.mjs`和`routes/notes.mjs`中，我们进行以下更改：

```js
import util from 'util';
import express from 'express';
import * as notes from '../models/notes';

export const router = express.Router();
```

# 使用文件系统存储运行笔记应用程序

在`package.json`中，在`scripts`部分添加以下内容：

```js
"start-fs": "DEBUG=notes:* NOTES_MODEL=fs node --experimental-modules ./bin/www.mjs", 
```

当你在`package.json`中放入这些条目时，请确保使用正确的 JSON 语法。特别是，如果你在`scripts`部分的末尾留下逗号，它将无法解析，并且 npm 将抛出一个错误信息。

在放置好这段代码后，我们现在可以按照以下方式运行*笔记*应用程序：

```js
$ DEBUG=notes:* npm run start-fs

> notes@0.0.0 start-fs /Users/david/chap07/notes
> NOTES_MODEL=models/notes-fs node --experimental-modules./bin/www.mjs

  notes:server Listening on port 3000 +0ms
  notes:fs-model keylist dir=notes-fs-data files=[  ] +4s  
```

然后，我们可以在之前一样的方式使用`http://localhost:3000`上的应用程序。因为我们没有更改任何模板或 CSS 文件，所以应用程序将看起来与你在第六章*实现移动优先范式*结束时的样子完全一样。

因为`notes:*`的调试被打开，我们会看到*笔记*应用程序正在进行的日志。通过简单地不设置`DEBUG`变量，很容易关闭这个功能。

你现在可以终止并重新启动*笔记*应用程序，并看到完全相同的笔记。你还可以使用常规文本编辑器，如`vi`在命令行中编辑笔记。你现在可以在不同的端口上启动多个服务器，并看到完全相同的笔记：

```js
"server1": "NOTES_MODEL=fs PORT=3001 node --experimental-modules./bin/www.mjs", 
"server2": "NOTES_MODEL=fs PORT=3002 node --experimental-modules./bin/www.mjs", 
```

然后，你在单独的命令窗口中启动`server1`和`server2`，就像我们在第五章*你的第一个 Express 应用程序*中做的那样。然后，在单独的浏览器窗口中访问这两个服务器，你将看到两个浏览器窗口都显示了相同的笔记。

最后的检查是创建一个键中包含`/`字符的笔记。记住，键用于生成存储笔记的文件名，因此键不能包含`/`字符。在浏览器打开的情况下，点击“添加笔记”并输入笔记，确保你在`key`字段中使用`/`字符。点击提交按钮时，你会看到一个错误信息，说明这是不允许的。

# 使用 LevelUP 数据存储存储笔记

要开始使用实际的数据库，让我们看看一个非常轻量级、小体积的数据库引擎：**LevelUP**。这是一个围绕 Google 开发的 LevelDB 引擎的 Node.js 友好型包装器，通常用于在网页浏览器中进行本地数据持久化。它是一个非索引的 NoSQL 数据存储，最初是为在浏览器中使用而设计的。Node.js 模块 Level 使用 LevelDB API，并支持多个后端，包括`LevelDOWN`，它将 C++ LevelDB 数据库集成到 Node.js 中。

访问 [`www.npmjs.com/package/level`](https://www.npmjs.com/package/level) 获取有关模块的信息。`level` 包自动设置 `levelup` 和 `leveldown` 包。

要安装数据库引擎，请运行以下命令：

```js
$ npm install level@2.1.x --save
```

然后，开始创建 `models/notes-level.mjs` 模块：

```js
import fs from 'fs-extra';
import path from 'path';
import util from 'util';
import Note from './Note';
import level from 'level';
import DBG from 'debug';
const debug = DBG('notes:notes-level'); 
const error = DBG('notes:error-level'); 

var db;

async function connectDB() { 
    if (typeof db !== 'undefined' || db) return db;
    db = await level(
        process.env.LEVELDB_LOCATION || 'notes.level', { 
            createIfMissing: true, 
            valueEncoding: "json" 
    }); 
    return db;
} 
```

`level` 模块通过 `db` 对象为我们提供了一个与数据库交互的方式。我们为了方便使用，将此对象作为模块中的全局变量存储。如果 `db` 对象已设置，我们就可以立即返回它。否则，我们将使用 `createIfMissing` 打开数据库，以便在需要时创建数据库。

数据库的位置默认为当前目录中的 `notes.level`。可以通过设置环境变量 `LEVELDB_LOCATION`（顾名思义）来指定数据库位置：

```js
async function crupdate(key, title, body) { 
    const db = await connectDB();
    var note = new Note(key, title, body); 
    await db.put(key, note.JSON);
    return note;
}

export function create(key, title, body) {
    return crupdate(key, title, body);
}

export function update(key, title, body) {
    return crupdate(key, title, body);
}
```

调用 `db.put` 要么创建一个新的数据库条目，要么替换现有的一个。因此，`update` 和 `create` 都被设置为相同的函数。我们将笔记转换为 JSON，以便它可以轻松地存储在数据库中：

```js
export async function read(key) {
    const db = await connectDB();
    var note = Note.fromJSON(await db.get(key));
    return new Note(note.key, note.title, note.body);
}
```

读取笔记很简单：只需调用 `db.get`，它将检索数据，该数据必须从 JSON 表示形式中解码。

注意，`db.get` 和 `db.put` 没有使用回调函数，我们使用 `await` 来获取结果值。`level` 导出的函数可以接受回调函数，其中回调将被调用。如果没有提供回调函数，则 `level` 函数将返回一个 Promise 以与 `async` 函数兼容：

```js
export async function destroy(key) { 
    const db = await connectDB();
    await db.del(key);
}
```

`db.destroy` 函数从数据库中删除记录：

```js
export async function keylist() { 
    const db = await connectDB();
    var keyz = [];
    await new Promise((resolve, reject) => { 
        db.createKeyStream()
        .on('data', data => keyz.push(data)) 
        .on('error', err => reject(err)) 
        .on('end', () => resolve(keyz));
    }); 
    return keyz;
}

export async function count() { 
    const db = await connectDB();
    var total = 0;
    await new Promise((resolve, reject) => { 
        db.createKeyStream()
        .on('data', data => total++) 
        .on('error', err => reject(err)) 
        .on('end', () => resolve(total));
    }); 
    return total;
}

export async function close() {
    var _db = db;
    db = undefined;
    return _db ? _db.close() : undefined;
}
```

`createKeyStream` 函数使用类似于 Streams API 的事件驱动接口。它将流经每个数据库条目，在过程中发出事件。对于数据库中的每个键都会发出 `data` 事件，在数据库结束时发出 `end` 事件，在出错时发出 `error` 事件。结果是，没有简单的方法来将其表示为一个简单的 Promise。相反，我们调用 `createKeyStream`，让它运行，在运行过程中收集数据。我们必须将其包装在一个 Promise 对象中，并在 `end` 事件上调用 resolve。

然后将此内容添加到 `package.json` 的 `scripts` 部分：

```js
"start-level": "DEBUG=notes:* NOTES_MODEL=level node --experimental-modules ./bin/www.mjs",
```

最后，您可以运行 *Notes* 应用程序：

```js
$ DEBUG=notes:* npm run start-level
> notes@0.0.0 start /Users/david/chap07/notes
> node ./bin/www

  notes:server Listening on port 3000 +0ms 
```

控制台中的打印输出将与应用程序相同。您可以对其进行测试，以查看一切是否正常工作。

由于 `level` 不支持多个实例同时访问数据库，您将无法使用多个 *Notes* 应用程序场景。然而，您将能够随意停止和重新启动应用程序而不会丢失任何笔记。

# 使用 SQLite3 在 SQL 中存储笔记

要开始使用更常见的数据库，让我们看看如何在 Node.js 中使用 SQL。首先，我们将使用 SQLite3，这是一个轻量级、易于设置的数据库引擎，非常适合许多应用程序。

要了解该数据库引擎，请访问[`www.sqlite.org/`](http://www.sqlite.org/)。

要了解 Node.js 模块，请访问[`github.com/mapbox/node-sqlite3/wiki/API`](https://github.com/mapbox/node-sqlite3/wiki/API)或[`www.npmjs.com/package/sqlite3`](https://www.npmjs.com/package/sqlite3)。

SQLite3 的主要优势是它不需要服务器；它是一个自包含的、无需设置的 SQL 数据库。

第一步是安装模块：

```js
$ npm install sqlite3@3.x --save
```

# SQLite3 数据库模式

接下来，我们需要确保我们的数据库已配置。我们使用以下 SQL 表定义作为模式（将其保存为`models/schema-sqlite3.sql`）：

```js
CREATE TABLE IF NOT EXISTS notes (
    notekey VARCHAR(255),
    title VARCHAR(255),
    body TEXT
);
```

我们如何在编写代码之前初始化这个模式？一种方法是通过您的操作系统包管理系统确保安装了`sqlite3`包，例如在 Ubuntu/Debian 上使用`apt-get`，在 macOS 上使用 MacPorts。一旦安装完成，您就可以运行以下命令：

```js
$ sqlite3 chap07.sqlite3 
SQLite version 3.21.0 2017-10-24 18:55:49
Enter ".help" for usage hints.
sqlite> CREATE TABLE IF NOT EXISTS notes (
 ...> notekey VARCHAR(255),
 ...> title VARCHAR(255),
 ...> body TEXT
 ...> );
sqlite> .schema notes
CREATE TABLE notes (
 notekey VARCHAR(255),
 title VARCHAR(255),
 body TEXT
);
sqlite> ^D
$ ls -l chap07.sqlite3 
-rwx------ 1 david staff 8192 Jan 14 20:40 chap07.sqlite3  
```

虽然我们可以这样做，但十二要素应用程序模型表示我们必须以这种方式自动化任何管理过程。为此，我们应该编写一个小脚本，在 SQLite3 上运行 SQL 操作，并使用它来初始化数据库。

幸运的是，`sqlite3`命令为我们提供了这样做的方法。将以下内容添加到`package.json`的`scripts`部分：

```js
"sqlite3-setup": "sqlite3 chap07.sqlite3 --init models/schema-sqlite3.sql", 
```

运行设置脚本：

```js
$ npm run sqlite3-setup

> notes@0.0.0 sqlite3-setup /Users/david/chap07/notes
> sqlite3 chap07.sqlite3 --init models/schema-sqlite3.sql 

-- Loading resources from models/schema-sqlite3.sql

SQLite version 3.10.2 2016-01-20 15:27:19
Enter ".help" for usage hints.
sqlite> .schema notes
CREATE TABLE notes (
    notekey VARCHAR(255),
    title   VARCHAR(255),
    body    TEXT
);
sqlite> ^D
```

我们可以编写一个小的 Node.js 脚本来完成这个任务，这样做也很容易。然而，通过使用包提供的工具，我们在自己的项目中需要维护的代码更少。

# SQLite3 模型代码

现在，我们可以编写代码来在*笔记*应用程序中使用这个数据库。

创建`models/notes-sqlite3.mjs`文件：

```js
import util from 'util';
import Note from './Note';
import sqlite3 from 'sqlite3';
import DBG from 'debug';
const debug = DBG('notes:notes-sqlite3'); 
const error = DBG('notes:error-sqlite3'); 

var db; // store the database connection here 

async function connectDB() { 
    if (db) return db; 
    var dbfile = process.env.SQLITE_FILE || "notes.sqlite3"; 
    await new Promise((resolve, reject) => {
        db = new sqlite3.Database(dbfile, 
            sqlite3.OPEN_READWRITE | sqlite3.OPEN_CREATE, 
            err => { 
                if (err) return reject(err); 
                resolve(db);
        });
    });
    return db;
}
```

这与`notes-level.mjs`中的`connectDB`函数具有相同的目的：管理数据库连接。如果数据库没有打开，它会继续打开，并确保数据库文件被创建（如果不存在）。但如果数据库已经打开，它将立即返回：

```js
export async function create(key, title, body) { 
    var db = await connectDB();
    var note = new Note(key, title, body); 
    await new Promise((resolve, reject) => { 
        db.run("INSERT INTO notes ( notekey, title, body) "+ 
            "VALUES ( ?, ? , ? );", [ key, title, body ], err => { 
                if (err) return reject(err); 
                resolve(note); 
        }); 
    });
    return note;
}

export async function update(key, title, body) { 
    var db = await connectDB();
    var note = new Note(key, title, body); 
    await new Promise((resolve, reject) => { 
        db.run("UPDATE notes "+ 
            "SET title = ?, body = ? WHERE notekey = ?", 
            [ title, body, key ], err => { 
                if (err) return reject(err); 
                resolve(note); 
        }); 
    });
    return note;
}
```

这些是我们的`create`和`update`函数。正如承诺的那样，我们现在有理由定义笔记模型，使其具有`create`和`update`操作的单独函数，因为每个的 SQL 语句都不同。

调用`db.run`执行 SQL 查询，给我们机会将参数插入到查询字符串中。

`sqlite3`模块使用在 SQL 编程接口中常见的参数替换范式。程序员将 SQL 查询放入字符串中，然后在要插入值的地方放置一个问号。查询字符串中的每个问号都必须与程序员提供的数组中的值匹配。模块负责正确编码值，以确保查询字符串格式正确，同时防止 SQL 注入攻击。

`db.run` 函数简单地运行它所给的 SQL 查询，并不检索任何数据。因为 `sqlite3` 模块不产生任何类型的 Promise，我们必须在函数调用中包裹一个 `Promise` 对象：

```js
export async function read(key) {
  var db = await connectDB();
  var note = await new Promise((resolve, reject) => {
    db.get("SELECT * FROM notes WHERE notekey = ?", [key], (err,row) => {
        if (err) return reject(err);
        const note = new Note(row.notekey, row.title, row.body);
        resolve(note);
     });
  });
  return note;
}
```

要使用 `sqlite3` 模块检索数据，你使用 `db.get`、`db.all` 或 `db.each` 函数。这里使用的 `db.get` 函数只返回结果集的第一行。`db.all` 函数一次性返回结果集的所有行，如果结果集很大，这可能会成为内存问题。`db.each` 函数逐行检索，同时仍然允许处理整个结果集。

对于 *笔记* 应用程序，使用 `db.get` 来检索笔记是足够的，因为每个 `notekey` 只有一个笔记。因此，我们的 `SELECT` 查询无论如何最多只返回一行。但如果你的应用程序会在结果集中看到多行呢？我们将在下一分钟看看如何处理这个问题。

顺便说一句，这个 `read` 函数中有一个错误。看看你是否能找到错误。我们将在第十一章 *单元测试和功能测试* 中了解更多，当我们的测试工作揭示这个错误时：

```js
export async function destroy(key) {
  var db = await connectDB();
  return await new Promise((resolve, reject) => {
    db.run("DELETE FROM notes WHERE notekey = ?;", [key], err => {
        if (err) return reject(err);
        resolve();
    });
  });
}
```

要删除笔记，我们只需执行 `DELETE FROM` 语句：

```js
export async function keylist() {
    var db = await connectDB();
    var keyz = await new Promise((resolve, reject) => {
        var keyz = [];
        db.all("SELECT notekey FROM notes", (err, rows) => {
                if (err) return reject(err);
                resolve(rows.map(row => row.notekey ));
            });
    });
    return keyz;
}
```

`db.all` 函数检索结果集的所有行。

这个函数的合约是返回一个笔记键数组。`rows` 对象是从数据库中返回的结果数组，它包含我们要返回的数据，但格式不同。因此，我们使用 `map` 函数将数组转换为满足合约的格式：

```js
export async function count() {
    var db = await connectDB();
    var count = await new Promise((resolve, reject) => {
        db.get("select count(notekey) as count from notes",(err, row) 
        => {
                if (err) return reject(err);
                resolve(row.count);
            });
    });
    return count;
}

export async function close() {
    var _db = db;
    db = undefined;
    return _db ? new Promise((resolve, reject) => {
            _db.close(err => {
                if (err) reject(err);
                else resolve();
            });
        }) : undefined;
}
```

我们可以简单地使用 SQL 来为我们计算笔记的数量。在这种情况下，`db.get` 返回一个包含单个列 `count` 的行，这是我们想要返回的值。

# 使用 SQLite3 运行 Notes

最后，我们准备好使用 SQLite3 运行 *笔记* 应用程序。将以下代码添加到 `package.json` 的 `scripts` 部分：

```js
"start-sqlite3": "SQLITE_FILE=chap07.sqlite3 NOTES_MODEL=sqlite3 node --experimental-modules ./bin/www.mjs",
```

运行 *笔记* 应用程序：

```js
$ DEBUG=notes:* npm run start-sqlite3

> notes@0.0.0 start-sqlite3 /Users/david/chap07/notes
> SQLITE_FILE=chap07.sqlite3 NOTES_MODEL=models/notes-sqlite3 node ./bin/www.mjs

  notes:server Listening on port 3000 +0ms
  notes:sqlite3-model Opened SQLite3 database chap07.sqlite3 +5s  
```

你现在可以浏览应用程序在 `http://localhost:3000`，并且像之前一样运行它。

因为 SQLite3 支持来自多个实例的并发访问，你可以通过将此添加到 `package.json` 的 `scripts` 部分来运行多服务器示例：

```js
"server1-sqlite3": "SQLITE_FILE=chap07.sqlite3 NOTES_MODEL=sqlite3 PORT=3001 node ./bin/www.mjs", 
"server2-sqlite3": "SQLITE_FILE=chap07.sqlite3 NOTES_MODEL=sqlite3 PORT=3002 node ./bin/www.mjs", 
```

然后，像之前一样，在单独的命令窗口中运行这些命令。

因为我们没有对视图模板或 CSS 文件进行任何更改，应用程序看起来和之前一样。

当然，你可以使用 `sqlite` 命令，或其他 SQLite3 客户端应用程序来检查数据库：

```js
$ sqlite3 chap07.sqlite3 
SQLite version 3.10.2 2016-01-20 15:27:19
Enter ".help" for usage hints.
sqlite> select * from notes;
hithere|Hi There||ho there what there
himom|Hi Mom||This is where we say thanks  
```

# 使用 Sequelize 的 ORM 方法存储笔记

有几种流行的 SQL 数据库引擎，例如 PostgreSQL、MySQL ([`www.npmjs.com/package/mysql`](https://www.npmjs.com/package/mysql)) 和 MariaDB ([`www.npmjs.com/package/mariasql`](https://www.npmjs.com/package/mariasql))。每个数据库引擎都对应着类似我们刚刚使用的 `sqlite3` 模块的 Node.js 客户端模块。程序员可以接近 SQL，这就像驾驶手动挡汽车一样有趣。但如果我们想要一个更高层次的数据库视图，以便我们可以从对象的角度而不是数据库表中的行来思考呢？**对象关系映射**（**ORM**）系统提供了这样的高级接口，甚至可以提供使用相同数据模型与多个数据库的能力。

**Sequelize** 模块 ([`www.sequelizejs.com/`](http://www.sequelizejs.com/)) 基于 Promise，提供了强大、成熟的 ORM 功能，并且可以连接到 SQLite3、MySQL、PostgreSQL、MariaDB 和 MSSQL。由于 Sequelize 基于 Promise，它将自然地与我们所编写的基于 Promise 的应用程序代码兼容。

大多数 SQL 数据库引擎的一个先决条件是能够访问数据库服务器。在上一节中，我们通过使用 SQLite3 来绕过这个问题，因为 SQLite3 不需要设置数据库服务器。虽然可以在您的笔记本电脑上安装数据库服务器，但我们想避免这种复杂性，并使用 Sequelize 来管理 SQLite3 数据库。我们还将看到，只需一个配置文件就可以运行相同的 Sequelize 代码，针对托管数据库，如 MySQL。在 第十章，*部署 Node.js 应用程序* 中，我们将学习如何使用 Docker 在笔记本电脑上轻松设置任何服务，包括数据库服务器，并将相同的配置部署到实时服务器。大多数网络托管提供商将 MySQL 或 PostgreSQL 作为服务的一部分提供。

在我们开始编写代码之前，让我们安装两个模块：

```js
$ npm install sequelize@4.31.x --save
$ npm install js-yaml@3.10.x --save
```

第一个模块显然是安装 Sequelize 包。第二个模块 `js-yaml` 是为了我们可以实现一个 YAML 格式的文件来存储 Sequelize 连接配置。YAML 是一种可读性强的 *数据序列化语言*，这意味着 YAML 是一种易于使用的文本文件格式，用于描述数据对象。也许了解 YAML 的最佳地方是它的维基百科页面 [`en.wikipedia.org/wiki/YAML`](https://en.wikipedia.org/wiki/YAML)。

# Notes 应用程序的 Sequelize 模型

让我们创建一个新的文件，`models/notes-sequelize.mjs`：

```js
import fs from 'fs-extra';
import util from 'util';
import jsyaml from 'js-yaml';
import Note from './Note';
import Sequelize from 'sequelize';
import DBG from 'debug';
const debug = DBG('notes:notes-sequelize'); 
const error = DBG('notes:error-sequelize'); 

var SQNote; 
var sequlz;

async function connectDB() { 
  if (typeof sequlz === 'undefined') {
    const YAML = await fs.readFile(process.env.SEQUELIZE_CONNECT,'utf8');
    const params = jsyaml.safeLoad(YAML, 'utf8'); 
    sequlz = new Sequelize(params.dbname, params.username,
                           params.password, params.params); 
  }
  if (SQNote) return SQNote.sync(); 
  SQNote = sequlz.define('Note', { 
        notekey: { type: Sequelize.STRING, primaryKey: true, unique: 
        true }, 
        title: Sequelize.STRING, 
        body: Sequelize.TEXT 
  }); 
  return SQNote.sync();
}
```

数据库连接存储在 `sequlz` 对象中，通过读取配置文件（我们稍后会介绍这个文件）并实例化一个 Sequelize 实例来建立连接。数据模型 `SQNote` 向 Sequelize 描述我们的对象结构，以便它可以定义相应的数据库表。如果 `SQNote` 已经定义，我们只需返回它，否则我们定义并返回 `SQNote`。

Sequelize 连接参数存储在我们通过 `SEQUELIZE_CONNECT` 环境变量指定的 YAML 文件中。`new Sequelize(..)` 这一行打开数据库连接。显然，参数包含任何需要的数据库名称、用户名、密码以及其他与数据库连接所需的选项。

`sequlz.define` 这一行是我们定义数据库模式的地方。我们不是将模式定义为创建数据库表的 SQL 命令，而是提供了一个字段及其特性的高级描述。Sequelize 将对象属性映射到表中的列。

我们告诉 Sequelize 调用这个模式 Note，但我们使用 `SQNote` 变量来引用该模式。这是因为我们已将 Note 定义为一个表示笔记的类。为了避免名称冲突，我们将继续使用 `Note` 类，并使用 SQNote 与数据库中存储的笔记交互。

在以下位置可以找到在线文档：

Sequelize 类：[`docs.sequelizejs.com/en/latest/api/sequelize/`](http://docs.sequelizejs.com/en/latest/api/sequelize/)。 [`docs.sequelizejs.com/en/latest/api/sequelize/`](http://docs.sequelizejs.com/en/latest/api/sequelize/) 定义模型：[`docs.sequelizejs.com/en/latest/api/model/`](http://docs.sequelizejs.com/en/latest/api/model/)。

将这些函数添加到 `models/notes-sequelize.mjs`：

```js
export async function create(key, title, body) { 
    const SQNote = await connectDB();
    const note = new Note(key, title, body); 
    await SQNote.create({ notekey: key, title: title, body: body });
    return note;
}

export async function update(key, title, body) { 
    const SQNote = await connectDB();
    const note = await SQNote.find({ where: { notekey: key } }) 
    if (!note) { throw new Error(`No note found for ${key}`); } else { 
        await note.updateAttributes({ title: title, body: body });
        return new Note(key, title, body);
    } 
}
```

在 Sequelize 中创建新的对象实例有几种方法。最简单的是调用一个对象的 `create` 函数（在这个例子中，`SQNote.create`）。这个函数将两个其他函数 `build`（用于创建对象）和 `save`（用于将其写入数据库）合并在一起。

更新对象实例略有不同。首先，我们必须使用 `find` 操作从数据库中检索其条目。`find` 操作提供了一个指定查询的对象。使用 `find`，我们检索一个实例，而 `findAll` 操作检索所有匹配的实例。

要了解 Sequelize 查询的文档，请访问 [`docs.sequelizejs.com/en/latest/docs/querying/`](http://docs.sequelizejs.com/en/latest/docs/querying/)。

和大多数或所有其他 Sequelize 函数一样，`SQNote.find` 返回一个 Promise。因此，在 `async` 函数内部，我们需要 `await` 操作的结果。

更新操作需要两个步骤，第一步是使用 `find` 操作找到相应的对象以从数据库中读取它。一旦找到实例，我们就可以使用 `updateAttributes` 函数简单地更新其值：

```js
export async function read(key) { 
    const SQNote = await connectDB();
    const note = await SQNote.find({ where: { notekey: key } }) 
    if (!note) { throw new Error(`No note found for ${key}`); } else { 
        return new Note(note.notekey, note.title, note.body); 
    } 
}
```

读取笔记时，我们再次使用 `find` 操作。可能存在空结果的可能性，我们必须抛出一个错误来匹配。

这个函数的合约是返回一个 `Note` 对象。这意味着使用 Sequelize 获取的字段来创建一个 `Note` 对象：

```js
export async function destroy(key) { 
    const SQNote = await connectDB();
    const note = await SQNote.find({ where: { notekey: key } }) 
    return note.destroy(); 
}
```

要销毁一个笔记，我们使用 `find` 操作检索其实例，然后调用其 `destroy()` 方法：

```js
export async function keylist() { 
    const SQNote = await connectDB();
    const notes = await SQNote.findAll({ attributes: [ 'notekey' ] });
    return notes.map(note => note.notekey); 
}
```

因为`keylist`函数作用于所有`Note`对象，我们使用`findAll`操作。我们查询所有笔记上的`notekey`属性。我们得到了一个名为`notekey`的字段的对象数组，我们使用`.map`函数将其转换为笔记键的数组：

```js
export async function count() { 
    const SQNote = await connectDB();
    const count = await SQNote.count();
    return count; 
}

export async function close() {
    if (sequlz) sequlz.close();
    sequlz = undefined;
    SQNote = undefined;
}
```

对于`count`函数，我们可以直接使用`count()`方法来计算所需的结果。

# 配置 Sequelize 数据库连接

Sequelize 在几个 SQL 数据库引擎上支持相同的 API。数据库连接是通过 Sequelize 构造函数的参数初始化的。十二要素应用程序模型建议，此类配置数据应保存在代码外部，并使用环境变量或类似机制注入。我们将使用 YAML 格式的文件来存储连接参数，并通过环境变量指定文件名。

Sequelize 库没有定义用于存储连接参数的此类文件。但开发这样的文件很简单。让我们这样做。

Sequelize 构造函数的 API 是：`constructor(database: String, username: String, password: String, options: Object)`。

在`connectDB`函数中，我们构造器编写如下：

```js
sequlz = new Sequelize(params.dbname, params.username, params.password, params.params); 
```

这个名为`models/sequelize-sqlite.yaml`的文件为我们提供了一个简单的映射，如下所示，用于 SQLite3 数据库：

```js
dbname: notes 
username: 
password: 
params: 
    dialect: sqlite 
    storage: notes-sequelize.sqlite3 
```

YAML 文件是 Sequelize 构造函数参数的直接映射。此文件中的`dbname`、`username`和`password`字段直接对应于连接凭证，而`params`对象提供了额外的参数。`params`字段中有许多可能的属性可以使用，你可以在 Sequelize 文档中阅读有关它们的更多信息，请参阅[`docs.sequelizejs.com/manual/installation/usage.html`](http://docs.sequelizejs.com/manual/installation/usage.html)。

`dialect`字段告诉 Sequelize 使用哪种类型的数据库。对于 SQLite 数据库，数据库文件名在`storage`字段中给出。

让我们先使用 SQLite3，因为不需要进一步设置。之后，我们将尝试使用 MySQL 重新配置我们的 Sequelize 模块。

如果你已经有一个不同的数据库服务器可用，创建相应的配置文件很简单。对于笔记本电脑上的一个合理的 MySQL 数据库，创建一个新的文件，例如`models/sequelize-mysql.yaml`，包含以下代码：

```js
dbname: notes 
username: .. user name 
password: .. password 
params: 
    host: localhost 
    port: 3306 
    dialect: mysql 
```

这很简单。`username`（用户名）和`password`（密码）必须与数据库凭证相匹配，而`host`（主机）和`port`（端口）将指定数据库托管的位置。设置数据库`dialect`和其他连接信息，然后就可以开始了。

要使用 MySQL，你需要安装基础 MySQL 驱动程序，这样 Sequelize 才能使用 MySQL：

```js
$ npm install mysql@2.x --save
```

使用 Sequelize 针对它支持的其它数据库，如 PostgreSQL，同样简单。只需创建一个配置文件，安装 Node.js 驱动程序，并安装/配置数据库引擎。

# 使用 Sequelize 运行 Notes 应用程序

现在，我们可以准备运行*Notes*应用程序使用 Sequelize。我们可以针对 SQLite3 和 MySQL 运行它，但让我们从 SQLite 开始。将以下条目添加到`package.json`中的`scripts`条目：

```js
"start-sequelize": "SEQUELIZE_CONNECT=models/sequelize-sqlite.yaml NOTES_MODEL=sequelize node  --experimental-modules ./bin/www.mjs" 
```

然后按照以下步骤运行它：

```js
$ DEBUG=notes:* npm run start-sequelize

> notes@0.0.0 start-sequelize /Users/david/chap07/notes
> SEQUELIZE_CONNECT=models/sequelize-sqlite.yaml NOTES_MODEL=sequelize node --experimental-modules./bin/www.mjs

  notes:server Listening on port 3000 +0ms 
```

如前所述，应用程序看起来完全一样，因为我们没有更改视图模板或 CSS 文件。对其进行测试，一切应该正常工作。

使用 Sequelize，多个*Notes*应用程序实例就像在`package.json`的`scripts`部分添加这些行一样简单，然后像以前一样启动这两个实例：

```js
"server1-sequelize": "SEQUELIZE_CONNECT=models/sequelize-sqlite.yaml NOTES_MODEL=sequelize PORT=3001 node --experimental-modules ./bin/www.mjs", 
"server2-sequelize": "SEQUELIZE_CONNECT=models/sequelize-sqlite.yaml NOTES_MODEL=sequelize PORT=3002 node --experimental-modules ./bin/www.mjs",
```

你将能够启动这两个实例，使用不同的浏览器窗口访问这两个实例，并看到它们显示相同的笔记集合。

再次强调，在给定的数据库服务器上使用基于 Sequelize 的模式：

1.  安装和配置数据库服务器实例，或者获取已配置数据库服务器的连接参数。

1.  安装相应的 Node.js 驱动程序。

1.  编写一个与连接参数相对应的 YAML 配置文件。

1.  在`package.json`中创建新的`scripts`条目来自动化针对该数据库启动笔记。

# 在 MongoDB 中存储笔记

MongoDB 在 Node.js 应用程序中得到了广泛的应用，其中一个标志是流行的 MEAN 缩写：MongoDB（或 MySQL），Express，Angular 和 Node.js。MongoDB 是领先的 NoSQL 数据库之一。它被描述为*可扩展的、高性能的、开源的、面向文档的数据库*。它使用 JSON 风格的文档，没有预定义的、刚性的模式，并具有大量高级功能。你可以访问他们的网站获取更多信息以及文档，网址为[`www.mongodb.org`](http://www.mongodb.org)。

MongoDB 的 Node.js 驱动程序的文档可以在[`www.npmjs.com/package/mongodb`](https://www.npmjs.com/package/mongodb)和[`mongodb.github.io/node-mongodb-native/`](http://mongodb.github.io/node-mongodb-native/)找到。

Mongoose 是 MongoDB 的一个流行 ORM（对象关系映射）工具([`mongoosejs.com/`](http://mongoosejs.com/))。在本节中，我们将使用原生的 MongoDB 驱动程序，但 Mongoose 也是一个值得考虑的替代方案。

你需要一个运行的 MongoDB 实例。`compose.io`([`www.compose.io/`](https://www.compose.io/))和`ScaleGrid.io`([`scalegrid.io/`](https://scalegrid.io/))托管服务提供商提供托管 MongoDB 服务。如今，将 MongoDB 作为 Docker 容器的一部分托管在由其他 Docker 容器构建的系统中的操作非常简单。我们将在第十一章，*单元测试和功能测试*中这样做。

有可能在笔记本电脑上设置一个临时的 MongoDB 实例进行测试。它在所有操作系统包管理系统中都可用，MongoDB 网站有说明([`docs.mongodb.org/manual/installation/`](https://docs.mongodb.org/manual/installation/))。

一旦安装，就没有必要将 MongoDB 设置为后台服务。相反，你可以运行几个简单的命令，在命令窗口的前台运行 MongoDB 实例，你可以随时将其终止和重启。

在一个命令窗口中，运行以下命令：

```js
$ mkdir data
$ mongod --dbpath data
```

在另一个命令窗口中，你可以按以下方式测试：

```js
$ mongo
MongoDB shell version: 3.0.8
connecting to: test
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
  http://docs.mongodb.org/
Questions? Try the support group
  http://groups.google.com/group/mongodb-user
> db.foo.save({ a: 1});
WriteResult({ "nInserted" : 1 })
> db.foo.find();
{ "_id" : ObjectId("56c0c98673f65b7988a96a77"), "a" : 1 }
> 
bye
```

这将在名为`foo`的集合中保存一个*文档*。第二个命令在`foo`中查找所有文档，并将它们打印出来供你使用。`_id`字段由 MongoDB 添加，作为文档标识符。这对于测试和调试很有用。对于实际部署，你的 MongoDB 服务器必须在服务器上正确安装。请参阅 MongoDB 文档以获取这些说明。

# Notes 应用程序的 MongoDB 模型

既然你已经证明你有了一个工作的 MongoDB 服务器，让我们开始工作。

安装 Node.js 驱动程序就像运行以下命令一样简单：

```js
$ npm install mongodb@3.x --save
```

现在创建一个新的文件，`models/notes-mongodb.mjs`：

```js
import util from 'util';
import Note from './Note';
import mongodb from 'mongodb'; 
const MongoClient = mongodb.MongoClient;
import DBG from 'debug';
const debug = DBG('notes:notes-mongodb'); 
const error = DBG('notes:error-mongodb'); 

var client;

async function connectDB() { 
    if (!client) client = await MongoClient.connect(process.env.MONGO_URL);
    return { 
        db: client.db(process.env.MONGO_DBNAME), 
        client: client
    };
}
```

使用`MongoClient`类与 MongoDB 实例连接。所需的 URL 将通过环境变量指定，格式简单：`mongodb://localhost/`。数据库名称通过另一个环境变量指定。

对应对象的文档可以在以下位置找到

[`mongodb.github.io/node-mongodb-native/2.2/api/MongoClient.html`](http://mongodb.github.io/node-mongodb-native/2.2/api/MongoClient.html) 用于 MongoClient 和 [`mongodb.github.io/node-mongodb-native/2.2/api/Db.html`](http://mongodb.github.io/node-mongodb-native/2.2/api/Db.html) 用于 Db

这创建了一个数据库客户端，然后打开数据库连接。这两个对象都从`connectDB`以匿名对象的形式返回。MongoDB 操作的一般模式如下：

```js
(async () => {
  const client = await MongoClient.connect(process.env.MONGO_URL);
  const db = client.db(process.env.MONGO_DBNAME);
  // perform database operations using db object
  client.close();
})();
```

因此，我们的模型方法需要同时使用`client`和`db`对象，因为它们都会用到它们。让我们看看这是如何完成的：

```js
export async function create(key, title, body) { 
    const { db, client } = await connectDB();
    const note = new Note(key, title, body); 
    const collection = db.collection('notes'); 
    await collection.insertOne({ notekey: key, title, body });
    return note;
}

export async function update(key, title, body) { 
    const { db, client } = await connectDB();
    const note = new Note(key, title, body); 
    const collection = db.collection('notes'); 
    await collection.updateOne({ notekey: key }, { $set: { title, body } });
    return note;
}
```

我们使用解构赋值将`db`和`client`检索到单独的变量中。

MongoDB 将所有文档存储在集合中。一个*集合*是一组相关的文档，集合在关系数据库中类似于一个表。这意味着创建一个新的文档或更新现有的文档，首先将其构造为一个 JavaScript 对象，然后请求 MongoDB 将该对象保存到数据库中。MongoDB 自动将该对象编码为其内部表示。

`db.collection`方法为我们提供了一个`Collection`对象，我们可以用它来操作命名集合。请参阅其文档[`mongodb.github.io/node-mongodb-native/2.2/api/Collection.html`](http://mongodb.github.io/node-mongodb-native/2.2/api/Collection.html)。

如方法名所示，`insertOne`将一个文档插入到集合中。同样，`updateOne`方法首先查找一个文档（在这种情况下，通过查找匹配的`notekey`字段），然后根据指定更改文档中的字段。

你会看到这些方法返回一个 Promise。`mongodb` 驱动支持回调和 Promise。许多方法如果提供了回调函数，将调用该函数，否则它返回一个将传递结果或错误的 Promise。当然，由于我们正在使用 `async` 函数，`await` 关键字使得这个过程非常简洁。

更详细的文档可以在以下链接中找到：

插入：[`docs.mongodb.org/getting-started/node/insert/`](https://docs.mongodb.org/getting-started/node/insert/). [更新：](https://docs.mongodb.org/getting-started/node/update/).

接下来，让我们看看如何从 MongoDB 读取笔记：

```js
export async function read(key) { 
    const { db, client } = await connectDB();
    const collection = db.collection('notes');
    const doc = await collection.findOne({ notekey: key });
    const note = new Note(doc.notekey, doc.title, doc.body);
    return note; 
}
```

`mongodb` 驱动支持多种 `find` 操作的变体。在这种情况下，*笔记* 应用确保恰好有一个文档与给定的键匹配。因此，我们可以使用 `findOne` 方法。正如其名所示，`findOne` 将返回第一个匹配的文档。

`findOne` 的参数是一个查询描述符。这个简单的查询寻找 `notekey` 字段与请求的 `key` 匹配的文档。当然，一个空查询将匹配集合中的所有文档。你可以以类似的方式匹配其他字段，并且查询描述符可以做更多的事情。有关查询的文档，请访问 [`docs.mongodb.org/getting-started/node/query/`](https://docs.mongodb.org/getting-started/node/query/)。

我们之前使用的 `insertOne` 方法也采用了相同的查询描述符。

为了满足这个函数的合约，我们创建一个 `Note` 对象，并将其返回给调用者。因此，我们使用从数据库检索到的数据创建一个 Note：

```js
export async function destroy(key) { 
    const { db, client } = await connectDB();
    const collection = db.collection('notes'); 
    await collection.findOneAndDelete({ notekey: key });
}
```

`find` 变体之一是 `findOneAndDelete`。正如其名所示，它会找到与查询描述符匹配的一个文档，然后删除该文档：

```js
export async function keylist() { 
    const { db, client } = await connectDB();
    const collection = db.collection('notes'); 
    const keyz = await new Promise((resolve, reject) => { 
        var keyz = []; 
        collection.find({}).forEach( 
            note => { keyz.push(note.notekey); }, 
            err => { 
                if (err) reject(err); 
                else resolve(keyz); 
            } 
        ); 
    }); 
    return keyz;
}
```

在这里，我们使用基本的 `find` 操作并给它一个空查询，以便匹配每个文档。我们要返回的是一个包含每个文档的 `notekey` 的数组。

所有的 `find` 操作都会返回一个 `Cursor` 对象。文档可以在 [`mongodb.github.io/node-mongodb-native/2.1/api/Cursor.html`](http://mongodb.github.io/node-mongodb-native/2.1/api/Cursor.html) 找到。

如其名所示，`Cursor` 对象是查询结果集的一个指针。它具有许多与操作结果集相关的有用函数。例如，你可以跳过结果集中的前几个项目，或者限制结果集的大小，或者执行 `filter` 和 `map` 操作。

`Cursor.forEach` 方法接受两个回调函数。第一个在结果集中的每个元素上调用。在这种情况下，我们可以用它将 `notekey` 记录在一个数组中。第二个回调在处理完结果集中的所有元素后调用。我们使用这个来指示成功或失败，并返回 `keyz` 数组。

因为`forEach`使用这种模式，所以它没有提供 Promise 的选项，我们必须自己创建 Promise，如下所示：

```js
export async function count() { 
    const { db, client } = await connectDB();
    const collection = db.collection('notes');
    const count = await collection.count({});
    return count;
}

export async function close() {
    if (client) client.close();
    client = undefined;
}
```

`count`方法接受一个查询描述符，正如其名称所暗示的，它计算匹配文档的数量。

# 使用 MongoDB 运行笔记应用程序

现在我们有了 MongoDB 模型，我们可以准备好使用它来运行*笔记*。

到现在为止，你已经知道了这个步骤；将其添加到`package.json`的`scripts`部分：

```js
"start-mongodb": "MONGO_URL=mongodb://localhost/ MONGO_DBNAME=chap07 NOTES_MODEL=mongodb node --experimental-modules ./bin/www.mjs", 
```

`MONGO_URL`环境变量是连接到你的 MongoDB 数据库的 URL。

你可以按照以下方式启动*笔记*应用程序：

```js
$ DEBUG=notes:* npm run start-mongodb
> notes@0.0.0 start-mongodb /Users/david/chap07/notes
> MONGO_URL=mongodb://localhost/ MONGO_DBNAME=chap07 NOTES_MODEL=mongodb node --experimental-modules ./bin/www

  notes:server Listening on port 3000 +0ms 
```

你可以在`http://localhost:3000`上浏览应用程序，并对其进行测试。你可以终止并重新启动应用程序，你的笔记仍然会保留在那里。

将以下内容添加到`package.json`的`scripts`部分：

```js
"server1-mongodb": "MONGO_URL=mongodb://localhost/ MONGO_DBNAME=chap07 NOTES_MODEL=mongodb PORT=3001 node --experimental-modules ./bin/www.mjs", 
"server2-mongodb": "MONGO_URL=mongodb://localhost/ MONGO_DBNAME=chap07 NOTES_MODEL=mongodb PORT=3002 node --experimental-modules ./bin/www.mjs",
```

你将能够启动两个*笔记*应用程序的实例，并看到它们共享同一组笔记。

# 摘要

在本章中，我们经历了一场关于不同数据库技术的真实旋风。虽然我们反复查看相同的七个功能，但接触各种数据存储模型和完成任务的方式是有用的。即便如此，我们也只是触及了从 Node.js 访问数据库和数据存储引擎选项的表面。

通过正确抽象模型实现，我们能够在不改变应用程序其余部分的情况下轻松切换数据存储引擎。我们确实跳过了设置数据库服务器的问题。正如承诺的那样，当我们在第十章*部署 Node.js 应用程序*中探索 Node.js 应用程序的生产部署时，我们将讨论这个问题。

通过将模型代码集中在存储数据的目的上，模型和应用程序都应该更容易进行测试。可以使用一个模拟数据模块来测试应用程序，该模块提供已知可预测的笔记，这些笔记可以可预测地进行检查。我们将在第十一章*单元测试和功能测试*中更深入地探讨这一点。

在下一章中，我们将专注于使用 OAuth2 对用户进行身份验证。
