# 第四章。聊天应用程序

在本章中，我们将构建一个聊天应用程序。我们将构建的应用程序将完美地作为公司内部沟通工具。团队可以创建频道来讨论与项目相关的某些事项，甚至可以发送自动删除的敏感数据消息，例如服务器的登录凭证等。

# 设置基本应用程序

我们将首先使用与上一章中相同的样板代码设置基本应用程序。按照以下简单步骤来实现这一点：

1.  从 GitHub 克隆项目：[`github.com/robert52/express-api-starter`](https://github.com/robert52/express-api-starter)。

1.  将你的样板项目重命名为`mean-blueprints-chatapp`。

1.  如果你想，你可以通过运行以下命令来停止指向初始 Git 仓库：

    ```js
    git remote remove origin

    ```

1.  跳转到你的工作目录：

    ```js
    cd mean-blueprints-chatapp

    ```

1.  安装所有依赖项：

    ```js
    npm install

    ```

1.  创建开发配置文件：

    ```js
    cp config/environments/example.js config/environments/development.js

    ```

你的配置文件`config/environments/development.js`应该类似于以下内容：

```js
module.exports = {
  port: 3000,
  hostname: '127.0.0.1',
  baseUrl: 'http://localhost:3000',
  mongodb: {
    uri: 'mongodb://localhost/chatapp_dev_db'
  },
  app: {
    name: 'MEAN Blueprints - chat application'
  },
  serveStatic: true,
  session: {
    type: 'mongo',                     
    secret: 'someVeRyN1c3S#cr3tHer34U',
    resave: false,                          
    saveUninitialized: true            
  },
  proxy: {
    trust: true
  },
  logRequests: false  
};
```

# 修改用户模型

我们不需要太多关于用户的信息，因此我们可以将`User`模式缩减到仅严格必要的信息。此外，我们可以添加一个`profile`字段，它可以存储关于用户的任何额外信息，例如社交媒体配置文件信息或其他账户数据。

让我们修改`User`模式从`app/models/user.js`如下：

```js
const UserSchema = new Schema({
  email:  {
    type: String,
    required: true,
    unique: true
  },
  name: {
    type: String
  },
  password: {
    type: String,
    required: true,
    select: false
  },
  passwordSalt: {
    type: String,
    required: true,
    select: false
  },
  profile: {
    type: Mixed
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

# 消息历史数据模型

消息历史将是一个用户通过聊天应用程序提交的消息集合。在 MongoDB 中存储此类数据时，我们可以选择多种方法。好事是，没有正确的实现，尽管我们有许多常见的实现方法和考虑因素。

我们的起点将是用户发送的消息是会话线程的一部分。当两个或更多用户互相聊天时，最初为他们创建一个会话线程。消息对那个会话是私有的。这意味着消息与另一个实体具有父子关系，在我们的例子中是线程实体。

考虑到我们应用程序的需求，我们可以探索以下实现来存储我们的消息：

+   **将每条消息存储在单独的文档中**：这是最容易实现的，也是最灵活的，但它伴随着一些应用程序级别的复杂性。

+   **将所有消息嵌入到线程文档中**：由于 MongoDB 对文档大小的限制，这不是一个可接受的解决方案。

+   **实现混合解决方案**：消息存储在与线程文档分开的地方，但以类似桶的方式保存，每个桶存储有限数量的文档。因此，我们不会在一个桶中存储线程的所有消息，而是将它们分散开来。

对于我们的应用程序，我们可以采用每条消息一个文档的实现。这将为我们提供最大的灵活性和易于实现性。此外，我们可以轻松地以时间顺序和线程顺序检索消息。

## 线程模式

每条消息都将成为对话线程的一部分。有关谁参与对话等信息将存储在线程集合的文档中。

我们将从只包含必要字段的简单模式开始，其中我们将存储有关线程的简单信息。创建一个名为 `/app/models/thread.js` 的新文件，并使用以下模式设计：

```js
'use strict';

const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const ObjectId = Schema.ObjectId;

const ThreadSchema = new Schema({
  participants: {
    type: [
      {
        type: ObjectId,
        ref: 'User'
      }
    ]
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('Thread', ThreadSchema);
```

目前对我们来说最重要的部分是 `participants` 字段，它描述了谁参与了当前的对话。按照设计，我们的应用程序将支持多个用户参与同一个对话线程。想象一下，它就像一个频道，你的团队可以讨论一个特定的项目。

## 消息模式

如我们之前所说，我们将使用每条消息一个文档的方法。目前，我们将为我们的消息使用相当简单的模式。这可以根据应用程序的复杂性而改变。

我们将在 `app/models/message.js` 中定义我们的模式：

```js
'use strict';

const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const ObjectId = Schema.ObjectId;

const MessageSchema = new Schema({
  sender: {
    type: ObjectId,
    required: true,
    ref: 'User'
  },
  thread: {
    type: ObjectId,
    required: true,
    ref: 'Thread'
  },
  body: {
    type: String,
    required: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model('Message', MessageSchema);
```

模式相当简单。我们有一个发送者，它引用了一个用户和一个线程。在线程实体中，我们将存储有关对话的附加数据。

# 线程后端

在 Node.js 后端应用程序中，我们将通过 Express 应用程序的路由定义提供与管理工作线程相关的端点。还应该有一种方法可以从特定的线程中获取消息历史。

## 线程控制器

我们将通过以下步骤在新的控制器文件 `app/controllers/thread.js` 中添加管理线程所需的所有业务逻辑：

1.  添加所需的模块：

    ```js
    'use strict';

    const mongoose = require('mongoose');
    const Thread = mongoose.model('Thread');
    ```

1.  导出模块的方法：

    ```js
    module.exports.allByUser = allThreadsByUser;
    module.exports.find = findThread;
    module.exports.open = openThread;
    module.exports.findById = findThreadById;
    ```

1.  查找特定用户的全部线程：

    ```js
    function allThreadsByUser(req, res, next) {
      Thread
      .find({
        participants: req.user._id
      })
      .populate('participants')
      .exec((err, threads) => {
        if (err) {
          return next(err);
        }

        req.resources.threads = threads;
        next();
      });
    }
    ```

1.  通过不同的标准查找线程，例如，通过当前登录的用户和参与对话的另一个用户的 ID：

    ```js
    function findThread(req, res, next) {
      let query = {};
      if (req.body.userId) {
        query.$and = [
          { participants: req.body.userId },
          { participants: req.user._id.toString() }
        ];
      }

      if (req.body.participants) {
        query.$and = req.body.participants.map(participant => {
          return { participants: participant };
        });
      }

      Thread
      .findOne(query)
      .populate('participants')
      .exec((err, thread) => {
        if (err) {
          return next(err);
        }

        req.resources.thread = thread;
        next();
      });
    }
    ```

1.  打开一个新的对话：

    ```js
    function openThread(req, res, next) {
      var data = {};

      //  If we have already found the thread 
      //  we don't need to create a new one
      if (req.resources.thread) {
        return next();
      }

      data.participants = req.body.participants || [req.user._id, req.body.user];

      Thread
      .create(data, (err, thread) => {
        if (err) {
          return next(err);
        }

        thread.populate('participants', (err, popThread) => {
          if (err) {
            return next(err);
          }

          req.resources.thread = popThread;
          next();
        });
      });
    }
    ```

1.  最后，通过其 ID 查找线程：

    ```js
    function findThreadById(req, res, next) {
      Thread
      .findById(req.params.threadId, (err, thread) => {
        if (err) {
          return next(err);
        }

        req.resources.thread = thread;
        next();
      });
    }
    ```

## 定义路由

所有必要的业务逻辑都在控制器文件中实现。我们只需要将控制器中的方法挂载到路由上，以便它们可以从外部调用。创建一个名为 `app/routes/thread.js` 的新文件。添加以下代码：

```js
const express = require('express');
const router = express.Router();
const threadCtrl = require('../controllers/thread');
const messageCtrl = require('../controllers/message');
const auth = require('../middlewares/authentication');
const authorize = require('../middlewares/authorization');
const response = require('../helpers/response');

module.exports = router;
```

在我们添加了必要的模块依赖项之后，我们可以逐个实现每个路由：

1.  获取所有用户的线程：

    ```js
    router.get(
      '/threads',
      auth.ensured,
      threadCtrl.allByUser,
      response.toJSON('threads')
    );
    ```

1.  打开一个新的线程。如果参与者已经有一个线程，它将被返回：

    ```js
    router.post(
      '/thread/open',
      auth.ensured,
      threadCtrl.find,
      threadCtrl.open,
      response.toJSON('thread')
    );
    ```

1.  通过 ID 获取线程：

    ```js
    router.get(
      '/threads/:threadId',
      auth.ensured,
      threadCtrl.findById,
      authorize.onlyParticipants('thread'),
      response.toJSON('thread')
    )
    ```

1.  获取线程的所有消息：

    ```js
    router.get(
      '/threads/:threadId/messages',
      auth.ensured,
      threadCtrl.findById,
      authorize.onlyParticipants('thread'),
      messageCtrl.findByThread,
      response.toJSON('messages')
    );
    ```

我们跳过了几个步骤，并已经使用了消息控制器的一个方法；不要担心，我们将在下一步实现它。

## 消息控制器

我们的 API 应该返回特定对话的消息历史。我们将保持简单，直接从 MongoDB 的 `Message` 集合中检索所有数据。创建一个新的控制器文件，`app/controllers/message.js`，并添加以下逻辑以查找线程的所有消息文档：

```js
'use strict';

const mongoose = require('mongoose');
const Thread = mongoose.model('Thread');
const Message = mongoose.model('Message');
const ObjectId = mongoose.Types.ObjectId;

module.exports.findByThread = findMessagesByThread;

function findMessagesByThread(req, res, next) {
  let query = {
    thread: req.resources.thread._id
  };

  if (req.query.beforeId) {
    query._id = { $lt: new ObjectId(req.query.sinceId) };
  }

  Message
  .find(query)
  .populate('sender')
  .exec(function(err, messages) {
    if (err) {
      return next(err);
    }

    req.resources.messages = messages;
    next();
  });
}
```

由于我们有很多内容要覆盖，我们既不会在后端也不会在前端处理消息历史的分页。但在前面的代码中，我已经添加了一些帮助。如果发送了 `beforeId` 查询字符串，则可以通过最后已知的 ID 容易地进行分页。还请记住，如果 `_id` 字段存储了 `ObjectId` 值，则按其排序几乎等同于按创建时间排序。

让我们更深入地探讨一下这个 `_id` 字段。大多数 MongoDB 客户端会自动生成 `ObjectId` 值并将其赋给 `_id` 字段。如果文档中没有发送 `_id` 字段，`mongod`（MongoDB 的主要守护进程）将添加该字段。

我们可能会遇到的一个问题是，如果消息文档是由单个秒内的多个进程或系统生成的。在这种情况下，插入顺序将不会严格保留。

# 后端聊天服务

到目前为止，我们只是触及了我们后端应用程序的表面。我们将在服务器上添加一个服务层。这个抽象层将实现所有业务逻辑，例如即时消息。服务层将处理与其他应用程序模块和层的交互。

至于应用程序的 `WebSockets` 部分，我们将使用 `socketIO`，这是一个实时通信引擎。他们有一个非常棒的聊天应用程序示例。如果你还没有听说过它，可以查看以下链接：

[`socket.io/get-started/chat/`](http://socket.io/get-started/chat/)

## 聊天服务实现

现在我们已经熟悉了 `socketIO`，我们可以继续并实现我们的聊天服务。我们将首先创建一个名为 `app/services/chat/index.js` 的新文件。这将是我们的聊天服务的主要文件。添加以下代码：

```js
'use strict';

const socketIO = require('socket.io');
const InstantMessagingModule = require('./instant-messaging.module');

module.exports = build;

class ChatService {
}

function build(app, server) {
  return new ChatService(app, server);
}
```

不要担心 `InstantMessagingModule`。我们只是将其添加为参考，以免忘记。我们稍后会回来揭示这个谜团。我们的类应该有一个构造函数。现在让我们添加它：

```js
  constructor(app, server) {
    this.connectedClients = {};
    this.io = socketIO(server);
    this.sessionMiddleware = app.get('sessionMiddleware');
    this.initMiddlewares();
    this.bindHandlers();
  }
```

在构造函数中，我们初始化 `socketIO`，获取会话中间件，并最终将所有处理程序绑定到我们的 `socketIO` 实例。有关会话中间件的更多信息，可以在我们的 Express 配置文件 `config/express.js` 中找到。寻找类似的内容：

```js
  var sessionOpts = {
    secret: config.session.secret,
    key: 'skey.sid',
    resave: config.session.resave,
    saveUninitialized: config.session.saveUninitialized
  };

  if (config.session.type === 'mongodb') {
    sessionOpts.store = new MongoStore({
      url: config.mongodb.uri
    });
  }

  var sessionMiddleware = session(sessionOpts);
  app.set('sessionMiddleware', sessionMiddleware);
```

好处在于我们可以将这个会话逻辑与 `socketIO` 共享，并通过 `.use()` 方法挂载。这将在 `.initMiddlewares()` 方法中完成：

```js
  initMiddlewares() {
    this.io.use((socket, next) => {
      this.sessionMiddleware(socket.request, socket.request.res, next);
    });

    this.io.use((socket, next) => {
      let user = socket.request.session.passport.user;

      //  authorize user
      if (!user) {
        let err = new Error('Unauthorized');
        err.type = 'unauthorized';
        return next(err);
      }

      // attach user to the socket, like req.user
      socket.user = {
        _id: socket.request.session.passport.user
      };
      next();
    });
  }
```

首先，我们将会话中间件挂载到我们的实例上，这将会在类似挂载到我们的 Express 应用程序上的操作。其次，我们检查用户是否存在于套接字会话中，换句话说，即用户是否已经认证。

能够添加中间件是一个非常酷的特性，并使我们能够为每个已连接的套接字执行有趣的事情。我们还应该添加构造函数中的最后一个方法：

```js
  bindHandlers() {
    this.io.on('connection', socket => {
      // add client to the socket list to get the session later
      this.connectedClients[socket.request.session.passport.user] = socket;
      InstantMessagingModule.init(socket, this.connectedClients, this.io);
    });
  }
```

对于每个成功连接的客户端，我们将初始化即时通讯模块并将已连接的客户端存储在映射中，以供以后参考。

## 即时通讯模块

为了稍微模块化一些，我们将把表示已连接客户端的功能性拆分到单独的模块中。目前只有一个模块，但将来你可以轻松地添加新的模块。`InstantMessagingModule`将位于主聊天文件的同一文件夹中，更确切地说，是`app/services/chat/instant-messaging.module.js`。你可以安全地向其中添加以下代码：

```js
'use strict';

const mongoose = require('mongoose');
const Message = mongoose.model('Message');
const Thread = mongoose.model('Thread');

module.exports.init = initInstantMessagingModule;

class InstantMessagingModule {
}

function initInstantMessagingModule(socket, clients) {
  return new InstantMessagingModule(socket, clients);
}
```

该服务将使用`Message`和`Thread`模型来验证和持久化数据。我们导出一个初始化函数而不是整个类。你可以轻松地向导出的函数添加额外的初始化逻辑。

类构造函数将会相当简单，看起来会类似于以下这样：

```js
  constructor(socket, clients) {
    this.socket = socket;
    this.clients = clients;
    this.threads = {};
    this.bindHandlers();
  }
```

我们只是将必要的依赖项分配给每个属性，并将所有处理程序绑定到已连接的套接字。让我们继续使用`.bindHandlers()`方法：

```js
  bindHandlers() {
    this.socket.on('send:im', data => {
      data.sender = this.socket.user._id;

      if (!data.thread) {
        let err = new Error('You must be participating in a conversation.')
        err.type = 'no_active_thread';
        return this.handleError(err);
      }

      this.storeIM(data, (err, message, thread) => {
        if (err) {
          return this.handleError(err);
        }

        this.socket.emit('send:im:success', message);

        this.deliverIM(message, thread);
      });
    });
  }
```

当通过 WebSockets 发送新消息时，它将通过`.storeIM()`方法存储，并通过`.deliverIM()`方法分发给每个参与者。

我们稍微抽象了发送即时消息的逻辑，所以让我们定义我们的第一个方法，它用于存储消息：

```js
  storeIM(data, callback) {
    this.findThreadById(data.thread, (err, thread) => {
      if (err) {
        return callback(err);
      }

      let user = thread.participants.find((participant) => {
        return participant.toString() === data.sender.toString();
      });

      if (!user) {
        let err = new Error('Not a participant.')
        err.type = 'unauthorized_thread';
        return callback(err);
      }

      this.createMessage(data, (err, message) => {
        if (err) {
          return callback(err);
        }

        callback(err, message, thread);
      });
    });
  }
```

因此，基本上，`.storeIM()`方法找到对话线程并创建一条新消息。我们还添加了存储消息时的简单授权。发送者必须是给定对话的参与者。你可以将这部分逻辑移动到更合适的模块中。我将把它留给你作为练习。

让我们添加之前使用过的下两个方法：

```js
  findThreadById(id, callback) {
    if (this.threads[id]) {
      return callback(null, this.threads[id]);
    }

    Thread.findById(id, (err, thread) => {
      if (err) {
        return callback(err);
      }

      this.threads[id] = thread;
      callback(null, thread);
    });
  }

  createMessage(data, callback) {
    Message.create(data, (err, newMessage) => {
      if (err) {
        return callback(err);
      }

      newMessage.populate('sender', callback);
    });
  }
```

最后，我们可以将我们的消息传递给其他参与者。实现可以在以下类方法中找到：

```js
  deliverIM(message, thread) {
    for (let i = 0; i < thread.participants.length; i++) {
      if (thread.participants[i].toString() === message.sender.toString()) {
        continue;
      }

      if (this.clients[thread.participants[i]]) {
        this.clients[thread.participants[i]].emit('receive:im', message);
      }
    }
  }
```

我们已经完成了后端应用程序的开发。它应该实现了所有必要的功能，以便开始开发客户端 Angular 应用程序。

# 引导 Angular 应用程序

是时候开始使用 Angular 2 构建我们的客户端应用程序了。我们将集成`SocketIO`与客户端以与我们的后端应用程序通信。我们将展示应用程序最重要的部分，但你可以在任何时候查看最终版本。

## 引导文件

```js
boot.ts file will be the final version, with all the necessary data added to it:
```

```js
import { bootstrap } from 'angular2/platform/browser';
import { provide } from 'angular2/core';
import { HTTP_PROVIDERS } from 'angular2/http';
import { ROUTER_PROVIDERS, LocationStrategy, HashLocationStrategy } from 'angular2/router';
import { AppComponent } from './app.component';
import { ChatService }  from './services/chat.service';
import { ThreadService }  from './services/thread.service';
import { MessageService }  from './services/message.service';
import { UserService } from './services/user.service';

import 'rxjs/add/operator/map';
import 'rxjs/add/operator/share';
import 'rxjs/add/operator/combineLatest';
import 'rxjs/add/operator/distinctUntilChanged';
import 'rxjs/add/operator/debounceTime';

bootstrap(AppComponent, [
  HTTP_PROVIDERS, ROUTER_PROVIDERS,
  ChatService,
  ThreadService,
  MessageService,
  UserService,
  provide(LocationStrategy, {useClass: HashLocationStrategy})
]); 
```

我们将为这个特定的应用程序实现四个服务，并且我们将从一个应用程序组件开始。为了简化，我们将使用基于哈希的位置策略。

## 应用程序组件

我们应用程序的主要组件是`app`组件。目前我们将保持其简单性，只向其中添加一个路由出口，并配置我们应用程序的路由。

创建一个名为`public/src/app.component.ts`的新文件，包含以下代码：

```js
import { Component } from 'angular2/core';
import { RouteConfig, RouterOutlet } from 'angular2/router';
import { Router } from 'angular2/router';
import { ChatComponent } from './chat/chat.component';

@RouteConfig([
  { path: '/messages/...', as: 'Chat', component: ChatComponent, useAsDefault: true }
])
@Component({
    selector: 'chat-app',
    directives: [
      RouterOutlet
    ],
    template: `
      <div class="chat-wrapper row card whiteframe-z2">
        <div class="chat-header col">
          <h3>Chat application</h3>
        </div>
        <router-outlet></router-outlet>
      </div>
    `
})
export class AppComponent {
  constructor() {
  }
}
```

我们创建了主应用程序组件并配置了一个将具有子路由的路由。默认情况下，`ChatComponent`将被挂载。所以，这非常基础。在我们继续应用程序的组件之前，让我们休息一下并定义自定义数据类型。

# 自定义数据类型

为了将类似的功能分组并具有自定义类型检查，我们将为应用程序中使用的每个实体定义类。这将使我们能够在创建实体时访问自定义初始化和默认值。

## 用户类型

我们在前端 Angular 应用程序中使用的第一个自定义数据类型将是用户。您可以使用接口来定义自定义类型或一个常规类。如果您需要默认值或自定义验证，请使用常规类定义。

创建一个名为`public/src/datatypes/user.ts`的新文件，并添加以下类：

```js
export class User {
  _id: string;
  email: string;
  name: string;
  avatar: string;
  createdAt: string;

  constructor(_id?: string, email?: string, name?: string, createdAt?: string) {
    this._id = _id;
    this.email = email;
    this.name = name;
    this.avatar = 'http://www.gravatar.com/avatar/{{hash}}?s=50&r=g&d=retro'.replace('{{hash}}', _id);     this.createdAt = createdAt;
  }
}
```

在实例化新用户时，`user`实例将预先填充`avatar`属性，包含一个特定的头像图片链接。我使用了`gravatar`并添加了用户的 ID 作为哈希来生成图像。通常，您必须使用用户的电子邮件作为`md5`哈希。显然，头像图像可以由任何服务提供。您甚至可以尝试向此应用程序添加文件上传和资料管理。

## 线程类型

接下来，我们将定义一个线程类，包含一些自定义初始化逻辑。创建一个名为`public/src/datatypes/thread.ts`的新文件：

```js
import { User } from './user';

export class Thread {
  _id: string;
  name: string;
  participants: Array<User>;
  createdAt: string;

  constructor(_id?: string, name?: string, participants?: Array<User>, createdAt?: string) {
    this._id = _id;
    this.name = name || '';
    this.participants = participants || [];
    this.createdAt = createdAt;
  }

  generateName(omittedUser) {
    let names = [];
    this.participants.map(participant => {
      if (omittedUser._id !== participant._id) {
        names.push(participant.name);
      }
    });

    return (names[1]) ? names.join(', ') : names[0];
  }
}
```

如您所见，用户数据类型已被导入并用于表示给定线程的参与者必须是一个用户数组。还定义了一个类方法，用于根据对话线程中的参与用户生成特定线程的名称。

## 消息类型

最后，我们将定义在我们的应用程序中消息的结构。为此，我们将创建一个名为`public/src/datatypes/message.ts`的新文件，包含以下逻辑：

```js
export class Message {
  _id: string;
  sender: any;
  thread: string;
  body: string;
  createdAt: string;
  time: string;
  fulltime: string;

  constructor(_id?: string, sender?: any, thread?: string, body?: string, createdAt?: string) {
    this._id = _id;
    this.sender = sender;
    this.body = body;
    this.createdAt = createdAt;
    this.time = this._generateTime(new Date(createdAt));
    this.fulltime = this._generateDateTime(new Date(createdAt));
  }

  private _generateTime(date) {
    return  date.getHours() + ":"
          + date.getMinutes() + ":"
          + date.getSeconds();
  }

  private _generateDateTime(date) {
    return date.getDate() + "/"
          + (date.getMonth()+1)  + "/"
          + date.getFullYear() + " @ "
          + this._generateTime(date);
  }
}
```

你可能已经在想，“为什么不包括`User`数据类型并将发送者标记为用户？”坦白说，这并不是必须的。您可以包含您喜欢的任何类型，代码仍然有效。这取决于您想向代码中添加多少粒度。

回到我们的代码，我们向`Message`类中添加了两个额外的方法，以生成两个时间戳，一个显示消息创建的时间，另一个显示包含日期和时间的完整时间戳。

# 应用程序服务

在初始章节中，我们根据它们的领域上下文对文件进行了分组。这次我们做了一些不同的处理，以强调你也可以从更扁平的方法开始。如果需要，你可以根据它们的领域上下文而不是类型来对文件进行分组。

尽管如此，我们还是将我们的组件根据它们的上下文分组，以便更快地定位它们。想象一下，你可以将整个应用程序加载到不同的应用程序中，并且拥有更扁平的文件夹结构将减少不必要的导航麻烦。

## 用户服务

我们将从一个简单的服务开始，该服务将处理所有用户应用程序逻辑。创建一个名为 `public/src/services/user.service.ts` 的新文件，并包含以下代码：

```js
import { Injectable } from 'angular2/core';
import { Http, Response, Headers } from 'angular2/http';
import { Observable } from 'rxjs/Observable';
import { contentHeaders } from '../common/headers';
import { User } from '../datatypes/user';

type ObservableUsers = Observable<Array<User>>;

@Injectable()
export class UserService {
  public users: ObservableUsers;
  public user: User;
  private _http: Http;
  private _userObservers: any;
  private _dataStore: { users: Array<User> };

  constructor(http: Http) {
    this._http = http;
    this.users = new Observable(observer => this._userObservers = observer).share();
    this._dataStore = { users: [] };
    this.getAll();
  }
}
```

我们公开了一个 `users` 属性，它是一个可观察对象，并将其转换为 `hot` 可观察对象。我们为服务定义了一个内部数据存储，它是一个简单的对象。差点忘了提！记得导入你的依赖项。作为构造函数的结尾，我们从后端检索所有用户。我们在服务中这样做，这样我们就不需要从组件中显式调用它。

实际上，`.getAll()` 方法尚未实现，所以让我们给这个类添加以下方法：

```js
  getAll() {
    return this._http
    .get('/api/users', { headers: contentHeaders })
    .map((res: Response) => res.json())
    .subscribe(users => this.storeUsers(users));
  }
```

我们还将数据持久性移动到另一个方法，以防我们想在别处使用它。将以下方法添加到 `UserService` 类中：

```js
  storeUsers(users: Array<User>) {
    this._dataStore.users = users;
    this._userObservers.next(this._dataStore.users);
  }
```

目前，我们的应用程序已经拥有了来自 `UserService` 的所有必要功能，我们可以继续实现其他应用程序组件。

## 线程服务

线程服务将处理并共享与线程相关的数据。我们将存储从后端检索到的线程。此外，当前活动线程也将存储在这个服务中。

让我们先创建服务文件，命名为 `public/src/services/thread.service.ts`。之后，遵循以下步骤来实现服务的核心逻辑：

1.  加载必要的依赖项：

    ```js
    import { Injectable } from 'angular2/core';
    import { Http, Response, Headers } from 'angular2/http';
    import { Subject } from 'rxjs/Subject';
    import { BehaviorSubject } from 'rxjs/Subject/BehaviorSubject';
    import { Observable } from 'rxjs/Observable';
    import { contentHeaders } from '../common/headers';
    import { Thread } from '../datatypes/thread';
    import { User } from '../datatypes/user';
    ```

1.  定义几个自定义数据类型：

    ```js
    type ObservableThreads = Observable<Array<Thread>>;
    type SubjectThread = Subject<Thread>;
    ```

1.  定义基类：

    ```js
    @Injectable()
    export class ThreadService {
      public threads: ObservableThreads;
      public currentThread: SubjectThread = new BehaviorSubject<Thread>(new Thread());
      private _http: Http;
      private _threadObservers: any;
      private _dataStore: { threads: Array<Thread> };
      private _currentUser: any;
    }
    ```

1.  添加构造函数：

    ```js
      constructor(http: Http) {
        this._http = http;
        this._dataStore = { threads: [] };
        this.threads = new Observable(
          observer => this._threadObservers = observer
        ).share();
      }
    ```

1.  添加必要的获取所有线程的方法：

    ```js
      getAll() {
        return this._http
        .get('/api/threads', { headers: contentHeaders })
        .map((res: Response) => res.json())
        .map(data => {
          return data.map(thread => {
            return new Thread(thread._id, thread._id, thread.participants, thread.createdAt)
          });
        })
        .subscribe(threads => {
          this._dataStore.threads = threads;
          this._threadObservers.next(this._dataStore.threads);
        });
    ```

    此方法将从后端服务检索所有线程。它将它们存储在服务的数据存储中，并将最新值推送到线程观察者，以便所有订阅者都可以获取最新值。

1.  定义如何打开一个新线程：

    ```js
      open(data: any) {
        return this._http
        .post('/api/thread/open', JSON.stringify(data), { headers: contentHeaders })
        .map((res: Response) => res.json())
        .map(data => {
          return new Thread(data._id, data.name, data.participants, data.createdAt);
        });
      }
    ```

    `open()` 方法将返回一个可观察对象，而不是在服务内部处理数据。

1.  我们还需要能够设置当前线程：

    ```js
      setCurrentThread(newThread: Thread) {
        this.currentThread.next(newThread);
      }
    ```

    `currentThread` 是一个 `BehaviorSubject`，它将只保留最后一个值并与任何新的订阅者共享。这在存储当前线程时很有用。记住，你需要用初始值初始化这个主题。

1.  暴露一个方法来从外部数据源存储线程：

    ```js
      storeThread(thread: Thread) {
        var found = this._dataStore.threads.find(t => {
          return t._id === thread._id;
        });

        if (!found) {
          this._dataStore.threads.push(thread);
          this._threadObservers.next(this._dataStore.threads);
        }
      }
    ```

我们不想存储相同的线程两次。我们可以改进的一件事是更新已更改的线程，但在这个应用程序中我们不需要这个功能。这是你在改进此应用程序时应记住的事情。

使用最后实现的逻辑，我们应该有对话线程所需的最小功能。

## 消息服务

消息服务将会有一些不同，因为我们打算使用 socket.io 通过`WebSockets`从之前在本章中设置的 socket 服务器发送和接收数据。别担心！这种差异不会反映在应用程序的其他部分。服务应该始终抽象底层逻辑。

我们将首先创建一个名为`public/src/services/message.service.ts`的服务文件，并导入必要的依赖项：

```js
import { Injectable } from 'angular2/core';
import { Http, Response, Headers } from 'angular2/http';
import { Observable } from 'rxjs/Observable';
import { ThreadService } from './thread.service';
import { contentHeaders } from '../common/headers';
import { Message } from '../datatypes/message';
import { User } from '../datatypes/user';
import * as io from 'socket.io-client';
```

你可以看到我们已将`socket.io-client`库中的所有内容导入为`io`。接下来，我们将添加类定义：

```js
type ObservableMessages = Observable<Array<Message>>;

@Injectable()
export class MessageService {
  public messages: ObservableMessages;

  private _http: Http;
  private _threadService: ThreadService;
  private _io: any;
  private _messagesObservers: any;
  private _dataStore: { messages: Array<Message> };

  constructor(http: Http, threadService: ThreadService) {
    this._io = io.connect();
    this._http = http;
    this._threadService = threadService;
    this.messages = new Observable(observer => this._messagesObservers = observer).share();
    this._dataStore = { messages: [] };
    this._socketOn();
  }
}
```

在构造函数中，我们将初始化并连接到 socket 服务器。因为我们同时在服务器端和客户端使用默认配置，所以我们可以直接调用`.connect()`方法。`_socketOn()`私有方法将定义所有 socket 事件绑定；让我们添加这个方法：

```js
  private _socketOn() {
    this._io.on('receive:im', message => this._storeMessage(message));
    this._io.on('send:im:success', message => this._storeMessage(message));
  }
```

我们刚刚定义了两个要监听的事件并调用`_storeMessage()`方法。对于每个事件，将通过 socket 接收到一条新消息。在此之后，我们应该在我们的`MessageSerivce`类中添加该方法：

```js
  private _storeMessage(message: Message) {
    let sender = new User(
      message.sender._id,
      message.sender.email,
      message.sender.name,
      message.sender.createdAt
    );
    let m = new Message(
      message._id,
      new User(sender._id, sender.email, sender.name, sender.createdAt),
      message.thread,
      message.body,
      message.createdAt
    );
    this._dataStore.messages.push(m);
    this._messagesObservers.next(this._dataStore.messages);
  }
```

当存储一条新消息时，我们将创建一个新的`User`实例，以便拥有关于消息发送者的所有必要数据。此方法将仅在服务内部使用，但我们需要公开一个发送消息的方法，它将被其他组件访问：

```js
  sendMessage(message: Message) {
    this._io.emit('send:im', message);
  }
```

发送消息并不难。我们只需要发出`send:im`事件并附加消息本身。除了发送和接收消息外，我们还需要获取特定线程的消息历史并存储在服务的数据存储中。让我们现在就做这件事：

```js
  getByThread(threadId) {
    this._http
    .get('/api/threads/'+threadId+'/messages', { headers: contentHeaders })
    .map((res: Response) => res.json())
    .map(res => {
      return res.map(data => {
        let sender = new User(
          data.sender._id,
          data.sender.email,
          data.sender.name,
          data.sender.createdAt
        );
        return new Message(
          data._id,
          sender,
          data.thread,
          data.body,
          data.createdAt
        );
      });
    })
    .subscribe(messages => {
      this._dataStore.messages = messages;
      this._messagesObservers.next(this._dataStore.messages);
    });
  }
```

前面的方法应该为我们从 Express 应用程序中检索必要的数据。我们像之前存储传入消息时一样，为每条消息做同样的事情。更确切地说，我们正在使用发送者的信息实例化一个新的用户。对于消息服务来说，这就足够了。

# 聊天组件

现在我们有了所有必要的数据类型和服务，我们可以回到应用程序的组件。回顾一下`app`组件，一个好的开始是先从`chat`组件开始。创建一个名为`public/src/chat/chat.component.ts`的新文件，并添加以下导入：

```js
import { Component } from 'angular2/core';
import { RouteConfig, RouterOutlet } from 'angular2/router';
import { ChatService } from '../services/chat.service';
import { ThreadListComponent } from '../thread/thread-list.component';
import { MessageListComponent } from '../message/message-list.component';
import { MessageFormComponent } from '../message/message-form.component';
import { UserListComponent } from '../user/user-list.component';
import { ChatHelpComponent } from './chat-help.component';
```

在导入所有所需的模块后，我们实际上可以实施我们的组件：

```js
@RouteConfig([
  { path: '/',            as: 'ThreadMessagesDefault', component: ChatHelpComponent, useAsDefault: true },
  { path: '/:identifier', as: 'ThreadMessages', component: MessageListComponent }
])
@Component({
  selector: 'chat',
  directives: [
    ThreadListComponent,
    MessageFormComponent,
    UserListComponent,
    RouterOutlet
  ],
  template: `
    <div class="threads-container col sidebar">
      <user-list></user-list>
      <thread-list></thread-list>
    </div>

    <div class="messages-container col content">
      <router-outlet></router-outlet>

      <div class="message-form-container">
        <message-form></message-form>
      </div>
    </div>
  `
})
export class ChatComponent {
  private _chatService: ChatService;

  constructor(chatService: ChatService) {
    this._chatService = chatService;
  }
}
```

在组件的模板中包含了我们将要实现的组件，例如，线程列表组件，它将显示与其他用户的所有当前对话。聊天组件将是我们的较小组件的容器，但我们还添加了一个 `RouterOutlet` 来动态加载与当前路由匹配的组件。

默认路由将在路由参数中没有添加线程 ID 时加载一个辅助组件。我们可以将此与组件的主页关联起来。你可以使默认视图尽可能复杂；例如，你可以添加一个链接到最近打开的线程。我们现在将保持简单。创建一个名为 `public/src/chat-help.component.ts` 的新文件，并添加以下代码：

```js
import { Component } from 'angular2/core';

@Component({
  selector: 'chat-help',
  template: `
    <h2>Start a new conversation with someone</h2>
  `
})
export class ChatHelpComponent {
  constructor() {
  }
}
```

这里没有太多花哨的东西！只是一个简单的组件，它有一个内联模板，为用户显示一条友好的消息。现在我们已经覆盖了这一点，我们可以继续并实现其余的组件。

# 用户列表组件

用户列表组件将为我们提供搜索用户和与他们开始新对话的能力。我们需要显示用户列表并根据搜索标准进行过滤。此外，点击列表中的用户应打开一个新的对话线程。所有这些都应该相对简单易行。让我们首先创建组件文件。创建一个名为 `public/src/user/user.component.ts` 的新文件，并具有以下基本结构：

```js
import { Component } from 'angular2/core';
import { Router } from 'angular2/router'
import { Subject } from 'rxjs/Subject';
import { ReplaySubject } from 'rxjs/Subject/ReplaySubject';
import { UserService } from '../services/user.service';
import { ThreadService } from '../services/thread.service';
import { User } from '../datatypes/user';

@Component({
    selector: 'user-list',
    template: ``
})
export class UserListComponent {
  public users: Array<User>;
  public filteredUsers: Array<User>;
  public selected: boolean = false;
  public search: Subject<String> = new ReplaySubject(1);
  public searchValue: string = '';
  private _threadService: ThreadService;
  private _userService: UserService;
  private _router: Router;

  constructor(userService: UserService, threadService: ThreadService, router: Router) {
    this._userService = userService;
    this._threadService = threadService;
    this._router = router;
  }
}
```

我们导入了必要的依赖项并定义了组件的基础。组件将包含两个主要部分，用户列表和输入字段，用于从列表中搜索指定的用户。让我们定义组件的模板：

```js
      <div class="user-search-container">
        <input
          type="text"
          class="form-control block"
          placeholder="start a conversation"
          [(ngModel)]="searchValue"
          (focus)="onFocus($event)"
          (input)="onInput($event)"
          (keydown.esc)="onEsc($event)"
        />
      </div>
      <div class="user-list-container">
        <div class="users-container" [ngClass]="{active: selected }">
          <div class="user-list-toobar">
            <a href="#" (click)="onClose($event)" class="close-button">
              <span>×</span>
              <span class="close-text">esc</span>
            </a>
          </div>
          <div *ngFor="#user of filteredUsers">
            <a href="javascript:void(0);" (click)="openThread($event, user)">{{user.name}}</a>
          </div>
        </div>
      </div>
```

为了显示用户列表，我们将订阅用户服务中的 `users` 可观察对象。将以下代码添加到构造函数中：

```js
    this._userService.users.subscribe(users => {
      this.filteredUsers = this.users = users;
    });
```

要显示过滤后的用户列表，我们将使用以下逻辑。将以下代码添加到构造函数中：

```js
    this.search
      .debounceTime(200)
      .distinctUntilChanged()
      .subscribe((value: string) => {
        this.filteredUsers = this.users.filter(user => {
          return user.name.toLowerCase().startsWith(value);
        });
      });
```

为了从输入中获取数据，我们可以使用类似以下的方法，该方法仅在用户开始输入时执行：

```js
  onInput(event) {
    this.search.next(event.target.value);
  }
```

为了打开一个新的线程，我们为列表中显示的每个用户绑定了一个点击事件。让我们添加执行此操作所需的方法：

```js
  openThread(event, user: User) {
    this._threadService.open({ userId: user._id }).subscribe(thread => {
      this._threadService.storeThread(thread);
      this._router.navigate(['./ThreadMessages', { identifier: thread._id}]);
      this.cleanUp();
    });
  }
```

上述代码将仅调用线程服务的 `.open()` 方法，并在成功后导航到返回的线程。我们还从这个组件中调用了 `cleanUp()` 方法，这将重置我们的组件到初始状态。

最后，让我们添加组件中缺失的所有逻辑。只需将方法附加到组件中：

```js
  onFocus() {
    this.selected = true;
  }

  onClose(event) {
    this.cleanUp();
    event.preventDefault();
  }

  onEsc(event) {
    this.cleanUp();
    let target: HTMLElement = <HTMLElement>event.target;
    target.blur();
    event.preventDefault();
  }
  cleanUp() {
    this.searchValue = '';
    this.selected = false;
    this.search.next('');
  }
```

快速回顾一下，我们创建了一个用户列表组件，当聚焦到搜索输入时，它会显示列表中的所有用户；默认情况下，列表是隐藏的。我们添加了一些特殊事件；例如，按 *Esc* 键时，列表应该被隐藏。

# 显示线程

用户必须知道他/她参与的是哪个对话，因此我们需要向用户显示此信息。为此，我们将实现一个线程列表组件。

## 线程组件

为了显示线程列表，我们将为每个线程使用一个组件来封装显示给用户的所有信息和功能。要创建所需的组件，请遵循以下步骤：

1.  创建组件文件，命名为 `public/src/thread/thread.component.ts`。

1.  导入必要的依赖项：

    ```js
    import { Component, OnInit } from 'angular2/core';
    import { RouterLink } from 'angular2/router';
    import { ThreadService } from '../services/thread.service';
    import { Thread } from '../datatypes/thread';
    ```

1.  添加组件注解：

    ```js
    @Component({
        inputs: ['thread'],
        selector: 'thread',
        directives: [ RouterLink ],
        template: `
          <div class="thread-item">
            <a href="#" [routerLink]="['./ThreadMessages', { identifier: thread._id }]" data-id="{{thread._id}}">
              {{thread.name}}
              <span *ngIf="selected"> &bull; </span>
            </a>
          </div>
        `
    })
    ```

1.  定义组件的类：

    ```js
    export class ThreadComponent implements OnInit {
      public thread: Thread;
      public selected: boolean = false;
      private _threadService: ThreadService;

      constructor(threadService: ThreadService) {
        this._threadService = threadService;
      }

      ngOnInit() {
        this._threadService.currentThread.subscribe( (thread: Thread) => {
          this.selected = thread && this.thread && (thread._id === this.thread._id);
        });
      }
    }
    ```

我们创建了一个单独的线程组件，用于在列表中显示信息。我们使用 `routerLink` 导航到所需的对话线程。在初始化时，我们检查当前是哪个线程，以便我们可以标记选定的线程。

## 线程列表组件

现在我们有了线程组件，我们可以将它们显示在列表中。为此，将使用一个新的组件。让我们做类似的事情：

1.  创建组件文件，命名为 `public/src/thread/thread-list.component.ts`。

1.  导入必要的依赖项，包括 `thread` 组件：

    ```js
    import { Component, ChangeDetectionStrategy } from 'angular2/core';
    import { Observable } from 'rxjs/Observable';
    import { ThreadService } from '../services/thread.service';
    import { Thread } from '../datatypes/thread';
    import { ThreadComponent } from './thread.component';
    ```

1.  构建组件注解：

    ```js
    @Component({
        selector: 'thread-list',
        directives: [ThreadComponent],
        // changeDetection: ChangeDetectionStrategy.OnPushObserve,
        // changeDetection: ChangeDetectionStrategy.OnPush,
        template: `
          <h4>Recent <span class="muted">({{threads.length}})</span></h4>
          <thread
            *ngFor="#thread of threads"
            [thread]="thread">
          </thread>
        `
    })
    ```

1.  定义 `ThreadListComponent` 类：

    ```js
    export class ThreadListComponent {
      public threads: Array<Thread> = [];
      private _threadService: ThreadService;

      constructor(threadService: ThreadService) {
        this._threadService = threadService;
        this._threadService.threads.subscribe(threads => {
          this.threads = threads;
        });
        this._threadService.getAll();
      }
    }
    ```

这应该向用户显示一个打开的对话线程列表。我们使用 `thread` 服务获取组件所需的数据。如果服务的 `threads` 集合发生变化，更新策略将为我们处理必要的更新。

# 消息传递

现在我们可以开始和恢复对话，我们需要能够向该对话的参与者发送消息。我们将专注于实现此功能。

发送消息的流程相当简单。首先，我们将所需的消息发送到后端应用程序，为历史记录存储消息，并通知收件人有新消息。发送者和收件人都会在他们的设备上显示消息。

## 发送消息

在之前的 `chat` 组件中，我们使用了 `message form` 组件。此组件将允许我们输入消息并将它们发送到 Node.js 后端服务。让我们保持简单，只添加必要的功能。创建一个新的组件文件，命名为 `public/src/message/message-form.component.ts`。

我们将在组件中导入两个服务。为我们的依赖项添加以下代码：

```js
import { Component, OnInit } from 'angular2/core';
import { ThreadService } from '../services/thread.service';
import { MessageService } from '../services/message.service';
import { Message } from '../datatypes/message';
import { User } from '../datatypes/user';
import { Thread } from '../datatypes/thread';
```

现在，我们将添加组件注解并定义组件的类：

```js
@Component({
    selector: 'message-form',
    // changeDetection: ChangeDetectionStrategy.OnPush,
    template: `
      <input
        class="message-form form-control"
        autocorrect="off" autocomplete="off" spellcheck="true"
        (keydown.enter)="onEnter($event)"
        [(ngModel)]="draftMessage.body"
      >
    `
})
export class MessageFormComponent implements OnInit {

  constructor() {
  }

  ngOnInit() {
  }
}
```

为了定义其余必要的逻辑，我们将遵循以下步骤：

1.  向该类添加以下属性：

    ```js
      public draftMessage: Message;
      private _messageService: MessageService;
      private _threadService: ThreadService;
      private _thread: Thread;
    ```

1.  将构造函数更改为以下类似的形式：

    ```js
      constructor(messageService: MessageService, threadService: ThreadService) {
        this._messageService = messageService;
        this._threadService = threadService;
        this._threadService.currentThread.subscribe(thread => this._thread = thread);
      }
    ```

1.  修改组件的初始化以重置 `草稿消息` 值：

    ```js
      ngOnInit() {
        this.draftMessage = new Message();
      }
    ```

1.  添加发送消息的逻辑：

    ```js
      sendMessage() {
        let message: Message = this.draftMessage;
        message.thread = this._thread._id;
        this._messageService.sendMessage(message);
        this.draftMessage = new Message();
      }
    ```

    这将简单地调用 `thread` 服务的 `.sendMessage()` 方法。

1.  定义当用户按下 *Enter* 键时会发生什么：

    ```js
      onEnter(event: any) {
        this.sendMessage();
        event.preventDefault();
      }
    ```

从技术上讲，我们现在可以向后端发送消息并将它们持久化到 MongoDB 以构建消息历史。

## 消息组件

要显示所有消息，我们将从小处着手，创建消息组件。此组件将是消息列表中的一个条目。我们应该已经有了显示单个消息所需的数据和逻辑。

要实现`message`组件，创建一个名为`public/src/message/message.component.ts`的新文件，并附加以下代码：

```js
import { Component, AfterViewInit } from 'angular2/core';

@Component({
    inputs: ['message'],
    selector: 'message',
    template: `
      <div class="message-item">
        <div class="message-identifier">
          <img src="img/{{message.sender.avatar}}" width="36" height="36"/>
        </div>
        <div class="message-content">
          <div class="message-sender">
            <span class="user-name">{{message.sender.name}}</span>
            <span class="message-timestamp" title={{message.fulltime}}>{{message.time}}</span>
          </div>
          <div class="message-body">
            {{message.body}}
          </div>
        </div>
      </div>
    `
})
export class MessageComponent implements AfterViewInit {
  constructor() {
  }

  ngAfterViewInit() {
    var ml = document.querySelector('message-list .message-list');
    ml.scrollTop = ml.scrollHeight;
  }
}
```

该组件相当简单。它仅显示单个消息的数据，并在视图初始化后，将消息列表滚动到最底部。通过最后一个功能，如果实际可以放入视图中的消息多于实际消息，它将简单地滚动到最后一条消息。

## 消息列表组件

在我们实现单个消息组件之前，这是我们的消息列表组件所必需的，该组件将向用户显示所有消息。我们将使用与线程列表类似的模式来实现此组件。

按照以下步骤实现消息列表组件：

1.  导入必要的依赖：

    ```js
    import { Component } from 'angular2/core';
    import { RouteParams } from 'angular2/router';
    import { MessageService } from '../services/message.service';
    import { ThreadService } from '../services/thread.service';
    import { Thread } from '../datatypes/thread';
    import { Message } from '../datatypes/message';
    import { MessageComponent } from './message.component';
    ```

1.  添加 Angular 组件注解：

    ```js
    @Component({
        selector: 'message-list',
        directives: [MessageComponent],
        template: `
          <div class="message-list">
            <div *ngIf="messages.length === 0" class="empty-message-list">
              <h3>No messages so far :)</h3>
            </div>
            <message
              *ngFor="#message of messages"
              [message]="message"
              ></message>
          </div>
        `
    })
    ```

1.  定义组件的类：

    ```js
    export class MessageListComponent {
      public messages: Array<Message> = [];
      private _messageService: MessageService;
      private _threadService: ThreadService;
      private _routeParams:RouteParams;
    }
    Add the constructor:
      constructor(
        messageService: MessageService,
        threadService: ThreadService,
        routeParams: RouteParams
      ) {
        this._routeParams = routeParams;
        this._messageService = messageService;
        this._threadService = threadService;
        this._messageService.messages.subscribe(messages => this.messages = messages);
        let threadId: string = this._routeParams.get('identifier');
        this._threadService.setCurrentThread(new Thread(threadId));
        this._messageService.getByThread(threadId);
      }
    ```

由于我们每次导航到匹配的路由时都会重新加载组件，我们可以获取当前的`identifier`参数并通过当前线程 ID 加载消息。我们还设置了当前线程 ID，以便其他订阅者可以相应地采取行动。例如，在`thread`组件中，我们检查当前线程是否与组件的线程匹配。

# 摘要

我们已经到达了本章的结尾。本章是关于构建实时聊天应用程序。我们使用了`WebSockets`进行实时通信，将消息历史存储在 MongoDB 中，并创建了线程式对话。我们还为改进或添加新功能留出了空间。

在下一章，我们将尝试构建一个电子商务应用程序。
