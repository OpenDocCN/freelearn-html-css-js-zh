# 第六章 拍卖应用

本章将专注于构建一个类似拍卖的应用程序，它将依赖于之前构建的电子商务应用的 API。它将是一个小型概念验证应用。我们应用的后端解决方案将消费我们电子商务应用的后端 API。我希望最后一章能成为我们的游乐场，这样我们就可以通过这本书中使用的有趣技术，并且在一个较小但有趣的应用中有所乐趣。

# 设置基本应用

我们将从经典的 Express 应用程序样板开始。按照以下步骤设置基本项目：

1.  从 GitHub 克隆项目：[`github.com/robert52/express-api-starter`](https://github.com/robert52/express-api-starter)。

1.  将您的样板项目重命名为`auction-app`。

1.  如果您愿意，可以通过运行以下命令停止指向初始 Git 远程仓库：

    ```js
    git remote remove origin

    ```

1.  跳转到您的工作目录：

    ```js
    cd auction-app

    ```

1.  安装所有依赖项：

    ```js
    npm install

    ```

1.  创建开发配置文件：

    ```js
    cp config/environments/example.js config/environments/development.js

    ```

您的配置文件，`auction-app/config/environments/development.js`，应类似于以下内容：

```js
'use strict';

module.exports = {
  port: 3000,
  hostname: '127.0.0.1',
  baseUrl: 'http://localhost:3000',
  mongodb: {
    uri: 'mongodb://localhost/auction_dev_db'
  },
  app: {
    name: 'MEAN Blueprints - auction application'
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

# 我们要构建的内容

我们将构建一个英文拍卖网站。之前的电子商务应用将为我们提供产品，管理员可以使用这些产品创建拍卖。拍卖有不同的特性；我们不会逐一讨论它们，而是将描述一个英文拍卖。

最常见的拍卖是英文拍卖；它是一个单维度的拍卖，唯一考虑的是为商品提供的出价。通常它是以卖家为导向的，意味着它是单方面的。

通常，拍卖会设定一个起始价格；这被称为**保留价**，在此价格以下，卖家不会出售商品。每个买家都会出价，每个人都知道每个出价，因此是公开叫价。获胜者支付获胜价格。

没有低于当前最高出价。通常，当没有人对支付最新价格感兴趣时，拍卖结束。此外，可以为拍卖设置一个结束时间。

结束时间可以是绝对时间，在我们的案例中是一个标准日期时间，或者是一个相对于最后出价的时间，例如 120 秒。在章节的后面，我们将讨论相对时间的好处。

# 数据建模

在我们的应用中，拍卖是一个特殊事件，用户——更准确地说，是出价者——可以对可供销售的商品进行出价。一个项目是电子商务平台上的一个产品，但它只保留显示给用户所需的信息。让我们更详细地讨论每个模型。

## 拍卖

拍卖将包含关于事件的所有必要信息。如前所述，我们将实现一个英文拍卖，我们将从我们的主要电子商务应用中出售商品。

英式拍卖是公开叫价，这意味着每个人都知道每个出价。获胜者将支付获胜价格。每个出价都会提高商品的价格，下一个出价者必须支付更多才能赢得拍卖。

所有拍卖都将有一个保留价格，这是一个低于我们不会出售我们的产品的起始价值。换句话说，这是卖方可以接受的最低价格。

为了简化问题，我们将为我们的拍卖设置一个结束时间。最后出价最接近结束时间的出价将是获胜出价。你也可以选择相对时间，这意味着你可以从最后出价（即 10 分钟）设置一个时间限制，如果在那个时间段内没有出价，就宣布获胜者。这可以非常有效地防止出价狙击。

例如，假设你在一个产品上出价 39 美元作为起始价格。通常情况下，你拥有最高的出价。现在想象一下，拍卖即将结束，但在结束前只有几秒钟，另一个出价者试图以 47 美元的价格出价。这将让你没有时间反应，所以最后出价者赢得了拍卖。这就是通常出价狙击的工作方式。

让我们看看 Mongoose 拍卖模式：

```js
'use strict';

const mongoose = require('mongoose');
const Money = require('./money').schema;
const Schema = mongoose.Schema;
const ObjectId = Schema.ObjectId;
const Mixed = Schema.Types.Mixed;

var AuctionSchema = new Schema({
  item:           { type: Mixed },
  startingPrice:  { type: Money },
  currentPrice:   { type: Money },
  endPrice:       { type: Money },
  minAmount:      { type: Money },
  bids: [{
    bidder:       { type: ObjectId, ref: 'Bidder' },
    amount:       { type: Number, default: 0 },
    createdAt:    { type: Date, default: Date.now }
  }],
  startsAt:       { type: Date },
  endsAt:         { type: Date },
  createdAt:      { type: Date, default: Date.now }
});

module.exports = mongoose.model('Auction', AuctionSchema);
```

除了之前讨论的信息外，我们还将在我们的拍卖文档中嵌入所有出价。如果拍卖中有许多出价，这可能不是一个好主意，但既然我们打算进行固定时间的拍卖，出价将非常有限。对于热门拍卖，你只需将出价移动到单独的集合中，并在拍卖文档中保留引用。

## 出价者

我们正在使用我们电子商务应用程序的后端 API，因此我们不需要在我们的数据库中存储用户。但我们可以存储有关我们的出价用户的额外数据。为此，我们可以创建一个新的模型，称为`app/models/bidder.js`，并添加以下内容：

```js
'use strict';

const mongoose = require('mongoose');
const Money = require('./money').schema;
const Schema = mongoose.Schema;
const ObjectId = Schema.ObjectId;
const Mixed = Schema.Types.Mixed;

const BidderSchema = new Schema({
  profileId:      { type: String },
  additionalData: { type: Mixed },
  auctions: [{
    auction:      { type: ObjectId, ref: 'Auction' },
    status:       { type: String, default: 'active'},
    joinedAt:     { type: Date, default: Date.now }
  }],
  createdAt:      { type: Date, default: Date.now }
});

module.exports = mongoose.model('Bidder', BidderSchema);
```

`profileId`存储用户的`_id`，以便从电子商务平台引用用户文档。你还可以在这个模型中存储额外的数据，并将出价者参与的拍卖存储在模型中。

# 拍卖后端

在上一章中，我们在我们的架构中添加了一个服务层。我们将遵循相同的模式。此外，我们还将添加一个额外的组件，称为`Mediator`，它将作为一个单一入口点来帮助我们与不同的模块进行通信。

我们将在构建我们的模块时遵循中介设计模式，这是一种行为设计模式。这将是一个单一的中央控制点，通过它进行通信。

## 中介

我们的`Mediator`将是一个对象，它将通过通道协调与不同模块的交互。一个模块可以订阅一个给定的事件，并在该事件发生时收到通知。所有这些与事件相关的话题几乎让我们想到使用 Node.js 的事件核心模块，该模块用于发出命名事件，从而调用要执行的功能。

这是一个很好的起点。我们需要解决的一个问题是我们的 `Mediator` 需要是一个单点入口，并且在我们应用程序的执行时间只能存在一个实例。我们可以简单地使用单例设计模式。考虑到所有这些，让我们实现我们的中介：

```js
'use strict';

const EventEmitter = require('events');
let instance;

class Mediator extends EventEmitter {
  constructor() {
    super();
  }
}

module.exports = function singleton() {
  if (!instance) {
    instance = new Mediator();
  }

  return instance;
} 
```

这应该为我们模块的构建提供一个坚实的基础；目前这应该足够了。因为我们使用了 ES6 特性，我们可以直接扩展 `EventEmitter` 类。我们不是导出整个 `Mediator` 类，而是导出一个函数，该函数检查是否已经存在一个实例，如果没有，我们就创建 `Mediator` 类的新实例。

让我们看看我们将如何使用这种技术的例子：

```js
'use strict';

const mediator = require('./mediator')();

mediator.on('some:awesome:event', (msg) => {
  console.log(`received the following message: ${msg}`);
});

mediator.emit('some:awesome:event', 'Nice!');
```

我们只需要引入 `mediator` 实例，并使用 `.on()` 方法订阅事件并执行一个函数。使用 `.emit()` 方法发布命名事件，并传递一个消息作为参数。

记住在使用 ES6 中的 `箭头函数` 时，监听函数中的 `this` 关键字不再指向 `EventEmitter`。

## 拍卖管理器

我们不是在应用程序的控制器层实现所有业务逻辑，而是要构建另一个服务，称为 `AuctionManager`。这个服务将包含执行拍卖所需的所有必要方法。

使用这种技术，我们可以在以后轻松地决定如何导出我们应用程序的业务逻辑：使用传统的端点或通过 WebSockets。

让我们遵循几个步骤来实现我们的拍卖管理器：

1.  创建一个名为 `/app/services/auction-manager.js` 的新文件。

1.  添加必要的依赖项：

    ```js
    const MAX_LIMIT = 30;

    const mongoose = require('mongoose');
    const mediator = require('./mediator')();
    const Auction = mongoose.model('Auction');
    const Bidder = mongoose.model('Bidder');
    ```

1.  定义基类：

    ```js
    class AuctionManager {
      constructor(AuctionModel, BidderModel) {
        this._Auction = AuctionModel || Auction;
        this._Bidder = BidderModel || Bidder;
      }
    }
    module.exports = AuctionManager;
    ```

1.  获取所有拍卖的方法：

    ```js
      getAllAuctions(query, limit, skip, callback) {
        if (limit > MAX_LIMIT) {
          limit = MAX_LIMIT;
        }

        this._Auction
        .find(query)
        .limit(limit)
        .skip(skip)
        .exec(callback);
      }
    ```

1.  加入拍卖：

    ```js
      joinAuction(bidderId, auctionId, callback) {
        this._Bidder.findById(bidderId, (err, bidder) => {
          if (err) {
            return callback(err);
          }

          bidder.auctions.push({ auction: auctionId });
          bidder.save((err, updatedBidder) => {
            if (err) {
              return callback(err);
            }

            mediator.emit('bidder:joined:auction', updatedBidder);
            callback(null, updatedBidder);
          });
        });
      }
    ```

    如您所见，我们开始使用我们的中介来触发事件。在这个阶段，当一位竞标者加入拍卖时，我们会触发一个事件。目前这对我们来说并没有增加太多价值，但当我们开始尝试我们的实时通信解决方案时，这将会变得很有用。

1.  投标：

    ```js
      placeBid(auctionId, bidderId, amount, callback) {
        if (amount <= 0) {
          let err = new Error('Bid amount cannot be negative.');
          err.type = 'negative_bit_amount';
          err.status = 409;
          return callback(err);
        }

        let bid = {
          bidder: bidderId,
          amount: amount
        };

        this._Auction.update(
          // query
          {
            _id: auctionId.toString()
          },
          // update
          {
            currentPrice: { $inc: amount },
            bids: { $push: bid }
          },
          // results
          (err, result) => {
            if (err) {
              return callback(err);
            }

            if (result.nModified === 0) {
              let err = new Error('Could not place bid.');
              err.type = 'new_bid_error';
              err.status = 500;
              return callback(err);
            }

            mediator.emit('auction:new:bid', bid);
            callback(null, bid);
          }
        );
      }
    ```

当进行投标时，我们只想将投标添加到我们的拍卖的投标列表中，为此，我们将使用原子操作来更新 `currentPrice` 并添加当前的投标。此外，在成功投标后，我们将为该事件触发一个事件。

## 拍卖师

我们将为即将推出的模块起一个花哨的名字，我们将称之为 `Auctioneer`。为什么这个名字？嗯，我们正在构建一个拍卖应用，所以我们可以添加一些复古的感觉，并添加一个拍卖师，他将宣布新的投标和谁加入了拍卖。

如您所猜测的，这将是我们实时通信模块。该模块将使用 `SocketIO`，我们将要做的事情与第四章中的类似，*聊天应用*，在那里我们使用了该模块进行实时通信。

我们将只通过我们的模块中最重要的一部分来查看不同的概念在实际中的应用。让我们创建一个名为 `app/services/auctioneer.js` 的文件，并添加以下内容：

```js
'use strict';

const socketIO = require('socket.io');
const mediator = require('./mediator')();
const AuctionManager = require('./auction-manager');
const auctionManager =  new AuctionManager();

class Auctioneer {
  constructor(app, server) {
    this.connectedClients = {};
    this.io = socketIO(server);
    this.sessionMiddleware = app.get('sessionMiddleware');
    this.initMiddlewares();
    this.bindListeners();
    this.bindHandlers();
  }
}
module.exports = Auctioneer;
```

所以基本上，我们只是结构化我们的类并在构造函数中调用了一些方法。我们已经熟悉构造函数中的一些代码；例如，`.initMiddlewares()` 方法看起来与第四章，*聊天应用*相似，在那里我们使用中间件来授权和验证用户：

```js
  initMiddlewares() {
    this._io.use((socket, next) => {
      this.sessionMiddleware(socket.request, socket.request.res, next);
    });

    this.io.use((socket, next) => {
      let user = socket.request.session.passport.user;

      // authorize user
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

我们在调用 `.bindHandlers()` 方法时初始化了我们的 `SocketIO` 处理程序，并通过调用 `.bindListeners()` 方法将监听器附加到我们的中介者上。

因此，我们的 `.bindHandlers()` 方法将具有以下结构：

```js
  bindHandlers() {
    this.io.on('connection', (socket) => {
      // add client to the socket list to get the session later
      let userId = socket.request.session.passport.user;
      this.connectedClients[userId] = socket;

      // when user places a bid
      socket.on('place:bid', (data) => {
        auctionManager.placeBid(
          data.auctionId,
          socket.user._id,
          data.amount,
          (err, bid) => {
            if (err) {
              return socket.emit('place:bid:error', err);
            }

            socket.emit('place:bid:success', bid);
          }
        );

      });
    });
  }
```

记住，这只是一个部分代码，最终版本将包含更多的处理程序。所以，当新的客户端连接时，我们将一些处理程序附加到我们的 socket 上。例如，在先前的代码中，我们监听 `place:bid` 事件，当用户放置新的出价时，该事件将被调用，并且 `AuctionManager` 服务将持久化该出价。

显然，我们需要通知其他客户端关于发生的变化；我们不会在这里处理这一点。我们的 `.placeBid()` 方法在每次成功记录新的出价时通过 `Mediator` 发出事件。我们唯一需要做的是监听该事件，这在我们调用拍卖师构造方法中的 `.bindListeners()` 时已经完成了。

让我们看看 `.bindListeners()` 方法的一个部分代码示例：

```js
  bindListeners() {
    mediator.on('bidder:joined:auction', (bidder) => {
      let bidderId = bidder._id.toString();
      let currentSocket = this.connectedClients[bidderId];
      currentSocket.emit.broadcast('bidder:joined:auction', bidder);
    });

    mediator.on('auction:new:bid', (bid) => {
      this.io.sockets.emit('auction:new:bid', bid);
    });
  }
```

在前面的代码中，我们监听当一位竞标者加入拍卖时的情况，并向每个客户端广播一条消息，期望只触发事件的 socket 客户端。当放置新的出价时，我们向每个 socket 客户端发出事件。所以基本上，我们有两个类似的广播功能，但有一个主要区别；一个向每个客户端发送消息，期望触发事件的那个客户端，而第二个则向所有已连接的客户端广播。

## 从控制器中使用服务

如我们之前讨论的，我们的服务可以从任何模块中消费，并以不同的方式向客户端暴露。之前，我们使用了 `AuctionManager` 并通过 WebSockets 公开其业务逻辑。现在，我们将使用简单的端点来完成同样的工作。

让我们创建一个名为 `app/controllers/auction.js` 的控制器文件，并包含以下内容：

```js
'use strict';

const _ = require('lodash');
const mongoose = require('mongoose');
const Auction = mongoose.model('Auction');
const AuctionManager = require('../services/auction-manager');
const auctionManager = new AuctionManager();

module.exports.getAll = getAllAuctions;

function getAllAuctions(req, res, next) {
  let limit = +req.query.limit || 30;
  let skip = +req.query.skip || 0;
  let query = _.pick(req.query, ['status', 'startsAt', 'endsAt']);

  auctionManager.getAllAuctions(query, limit, skip, (err, auctions) => {
    if (err) {
      return next(err);
    }

    req.resources.auctions = auctions;
    next();
  });
}
```

我们在书中已经多次这样做，所以这里没有什么新的。控制器导出一个函数，该函数将附加从服务返回的所有拍卖，并且稍后响应将被转换为 JSON 响应。

## 从电子商务 API 获取数据

当创建拍卖时，我们需要关于我们添加到拍卖中的物品的额外信息。关于产品物品的所有信息都存储在上一章中构建的电子商务平台上。

我们在本章中没有涵盖拍卖的创建，但我们可以讨论与电子商务 API 的底层通信层。在数据建模阶段，我们没有讨论将用户存储在数据库中。

不包括用户管理的原因是我们将消耗第三方 API 来管理我们的用户。例如，验证和注册将通过电子商务平台处理。

## 电子商务客户端服务

为了与第三方 API 进行通信，我们将创建一个服务来代理请求。由于我们不会从 API 消费很多端点，我们可以创建一个单一的服务来处理所有事情。随着你的应用程序的增长，你可以轻松地根据域上下文对文件进行分组。

让我们创建一个名为`app/services/ecommerce-client.js`的新文件，并按照以下步骤操作：

1.  声明在服务中使用的常量并包含依赖项：

    ```js
    'use strict';

    const DEFAULT_URL = 'http://localhost:3000/api';
    const CONTENT_HEADERS = {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
    };

    const request = require('request');
    ```

1.  定义一个用于配置请求对象的自定义`RequestOptions`类：

    ```js
    class RequestOptions {
      constructor(opts) {
        let headers = Object.assign({}, CONTENT_HEADERS, opts.headers);

        this.method = opts.method || 'GET';
        this.url = opts.url;
        this.json = !!opts.json;
        this.headers = headers;
        this.body = opts.body;
      }

      addHeader(key, value) {
        this.headers[key] = value;
      }
    }
    ```

    为了减少使用`request`进行调用所需的代码结构，我们定义了一个自定义类来实例化默认的请求选项。

1.  添加`EcommerceClient`类：

    ```js
    class EcommerceClient {
      constructor(opts) {
        this.request = request;
        this.url = opts.url || DEFAULT_URL;
      }
    }
    ```

    `EcommerceClient`类将成为我们访问第三方 API 的主要入口点。它更像是一个门面，以不知道我们应用程序中使用的底层数据源。

1.  指定如何验证用户：

    ```js
      authenticate(email, password, callback) {
        let req = new RequestOptions({
          method: 'POST',
          url: `${this.url}/auth/basic`
        });
        let basic = btoa(`${email}:${password}`);

        req.addHeader('Authorization', `Basic ${basic}`);

        this.request(req, function(err, res, body) => {
          callback(err, body);
        })
      }
    ```

    API 服务器将为我们处理验证；我们只是使用在调用 API 时返回的令牌。我们的自定义`RequestOptions`类允许我们添加额外的头部数据，例如`Authorization`字段。

1.  添加`getProducts()`方法：

    ```js
      getProducts(opts, callback) {
        let req = new RequestOptions({
          url: `${this.url}/api/products`
        });
        req.addHeader('Authorization', `Bearer ${opts.token}`);

        this.request(req, function(err, res, body) => {
          callback(err, body);
        })
      }
    ```

如您所见，使用相同的原理，我们可以从我们的电子商务应用程序中检索数据。唯一不同的是，我们需要在我们的调用中添加一个令牌。我们不会讨论如何消费我们的服务，因为我们已经在本书中多次这样做过。

在控制器中使用它应该相当简单，并配置一个路由器来向客户端应用程序公开必要的端点。

# 前端服务

由于我们只触及我们应用程序的最重要部分，我们将讨论在 Angular 应用程序中使用的服务的实现。我认为理解与后端应用程序的底层通信层是很重要的。

## 拍卖服务

`AuctionService`将处理与后端 API 的所有通信，以获取特定拍卖的信息，或者简单地获取所有可用的拍卖。为此，我们将创建一个新的文件，`public/src/services/auction.service.ts`：

```js
import { Injectable } from 'angular2/core';
import { Response, Headers } from 'angular2/http';
import { Observable } from 'rxjs/Observable';
import { Subject } from 'rxjs/Subject';
import { BehaviorSubject } from 'rxjs/Subject/BehaviorSubject';
import { AuthHttp } from '../auth/index';
import { contentHeaders } from '../common/headers';
import { Auction } from './auction.model';
import { SubjectAuction, ObservableAuction, ObservableAuctions } from './types';

const URL = 'api/auctions';

@Injectable()
export class AuctionService { 
}
```

我们导入了我们的依赖项，并添加了一个`URL`常量以提高代码的可读性，但你可以按自己的意愿处理基本 URL 配置。在我们添加必要的方法之前，还有一些事情需要处理，所以让我们定义构造函数和类属性：

```js
  public currentAuction: SubjectAuction = new BehaviorSubject<Auction>(new Auction());
  public auctions: ObservableAuctions;
  public auction: ObservableAuction;

  private _http: Http;
  private _auctionObservers: any;
  private _auctionsObservers: any;
  private _dataStore: { auctions: Array<Auction>, auction: Auction };

  constructor(http: Http, bidService: BidService) {
    this._http = http;
    this.auction = new Observable(observer => this._auctionObservers = observer).share();
    this.auctions = new Observable(observer => this._auctionsObservers = observer).share();
    this._dataStore = { auctions: [], auction: new Auction() };
  }
```

我们导出了一个用于单个拍卖和拍卖列表的 Observable。我们还对当前拍卖感兴趣。除了所有熟悉的定义外，我们还添加了一个用于内部使用的第三个服务。

当获取单个拍卖或所有拍卖时，我们将更新观察者的下一个值，以便订阅者通过变化的发生得到通知：

```js
    public getAll() {
    this._authHttp
    .get(URL, { headers: contentHeaders })
    .map((res: Response) => res.json())
    .map((data) => {
      return data.map((auction) => {
        return new Auction(
          auction._id,
          auction.item,
          auction.startingPrice,
          auction.currentPrice,
          auction.endPrice,
          auction.minAmount,
          auction.bids,
          auction.status,
          auction.startsAt,
          auction.endsAt,
          auction.createdAt
        );
      });
    })
    .subscribe(auctions => {
      this._dataStore.auctions = auctions;
      this._auctionsObservers.next(this._dataStore.auctions);
    }, err => console.error(err));
  } 
```

要获取单个拍卖，我们可以使用以下方法：

```js
  public getOne(id) {
    this._authHttp
    .get(`${URL}/${id}`)
    .map((res: Response) => res.json())
    .map((data) => {
      return new Auction(
        data._id,
        data.item,
        data.startingPrice,
        data.currentPrice,
        data.endPrice,
        data.minAmount,
        data.bids,
        data.status,
        data.startsAt,
        data.endsAt,
        data.createdAt
      );
    })
    .subscribe(auction => {
      this._dataStore.auction = auction;
      this._auctionObservers.next(this._dataStore.auction);
    }, err => console.error(err));
  }
```

因此，这个服务将与我们 Node.js 应用程序通信，并将所有接收到的数据存储在内部存储中。除了从服务器获取数据外，我们最终还想存储当前的拍卖，因此这段代码应该处理它：

```js
  public setCurrentAuction(auction: Auction) {
    this.currentAuction.next(auction);
  }
```

## 套接字服务

套接字服务将处理与 SocketIO 服务器的通信。好处是，我们有一个单一的入口点，并且可以将底层逻辑抽象到应用程序的其余部分。

创建一个名为 `public/src/common/socket.service.ts` 的新文件，并添加以下内容：

```js
import { Injectable } from 'angular2/core';
import * as io from 'socket.io-client';
import { Observable } from 'rxjs/Rx';
import { ObservableBid } from '../bid/index';
import { ObservableBidder } from '../bidder/index' 

export class SocketService {
}
```

我们只导入 SocketIO 客户端和所有其他数据类型。另外，别忘了添加你类所需的其余必要代码：

```js
  public bid: ObservableBid;
  public bidder: ObservableBidder;
  private _io: any;

  constructor() {
    this._io = io.connect();
    this._bindListeners();
  }
```

我们在这里做的一件有趣的事情是，通过以下技术公开 Observables——应用程序的其余部分可以订阅数据流：

```js
  private _bindListeners() {
    this.bid = Observable.fromEvent(
      this._io, 'auction:new:bid'
    ).share();
    this.bidder = Observable.fromEvent(
      this._io, 'bidder:joined:auction'
    ).share();
  }
```

RxJs 的优点在于我们可以从事件中创建 Observables。当套接字发出事件时，我们只需从该事件创建一个 Observable。使用前面的代码，我们可以订阅来自后端的数据。

为了通过 SocketIO 向后端发送信息，我们可以公开一个 `.emit()` 方法，它将只是套接字客户端上的 `.emit()` 方法的包装器：

```js
  public emit(...args) {
    this._io.emit.apply(this, args);
  }
```

## 投标服务

要了解整体情况，我们可以查看位于以下路径下的 `BidService`：`public/src/bid/bid.service.ts`。该类将具有类似的结构：

```js
@Injectable()
export class BidService {
  public bid: any;
  public currentAuction: any;
  private _socketService: SocketService;
  private _auctionService: AuctionService;

  constructor(
    socketService: SocketService, 
    auctionService: AuctionService
  ) {    
    this._socketService = socketService;
    this._auctionService = auctionService;
    this.currentAuction = {};
    this._auctionService.currentAuction.subscribe((auction) => {
      this.currentAuction = auction;
    });
    this.bid = this._socketService.bid.filter((data) => {
      return data.auctionId === this.currentAuction._id;
    });
  }

  public placeBid(auctionId: string, bid: Bid) {
    this._socketService.emit('place:bid', {
      auctionId: auctionId,
      amount: bid.amount
    });
  }
}
```

`BidService` 将与 `SocketService` 交互以放置投标，这些投标将通过 Express 后端应用程序推送到所有已连接的客户端。我们还根据当前选定的拍卖过滤每个传入的投标。

当当前选定的拍卖发生变化时，我们想通过订阅 `AuctionService` 的 `currentAuction` 来更新我们的本地副本。

## 投标服务

`BidderService` 将是第一个使用 `SocketService` 并订阅 `bidder` 对象变化的。它将存储来自后端 Node.js 服务器的所有传入数据。

让我们创建一个名为 `public/src/services/bidder.service.ts` 的新文件，并添加以下基本内容：

```js
import { Injectable } from 'angular2/core';
import { Observable } from 'rxjs/Observable';
import { Subject } from 'rxjs/Subject';
import { BehaviorSubject } from 'rxjs/Subject/BehaviorSubject';
import { contentHeaders } from '../common/headers';
import { SocketService } from './socket.service';
import { Bidder } from '../datatypes/bidder';
import { ObservableBidders } from '../datatypes/custom-types';

@Injectable()
export class BidderService {
}
```

现在我们有了起点，我们可以定义我们的构造函数并声明所有必要的属性：

```js
  public bidders: ObservableBidders;

  private _socketService: SocketService;
  private _biddersObservers: any;
  private _dataStore: { bidders: Array<Bidder> };

  constructor() {
    this.bidders = new Observable(observer => this._biddersObservers = observer).share();
    this._dataStore = { bidders: [] };
  }
```

在这个概念验证中，我们不会从这个服务中进行任何 HTTP 调用，并且我们主要会在数据存储中存储信息。以下 `public` 方法将很有用：

```js
  public storeBidders(bidders: Array<Bidder>) {
    this._socketService = socketService;
    this._dataStore = { bidders: [] };
    this.bidders = new Observable(observer => {
      this._biddersObservers = observer;
    }).share();
    this._socketService.bidder.subscribe(bidder => {
      this.storeBidder(bidder);
    });    
  }

  public storeBidder(bidder: Bidder) {
    this._dataStore.bidders.push(bidder);
    this._biddersObservers.next(this._dataStore.bidders);
  }

  public removeBidder(id: string) {
    let bidders = this._dataStore.bidders;

    bidders.map((bidder, index) => {
      if (bidder._id === id) {
        this._dataStore.bidders.splice(index, 1);
      }
    });

    this._biddersObservers.next(this._dataStore.bidders);
  }
```

在前面的章节中，我们已经以类似的形式使用了这种逻辑。为了简化，我们只需在我们的数据结构中存储竞标者或单个竞标者，并更新观察者的下一个值，这样每个订阅者都会收到通知以获取最新值。

之前，我们使用了一个自定义数据类型 `Bidder`——或者如果你更熟悉，可以称之为模型。让我们快速看看它，位于以下路径——`public/src/datatypes/bidder.ts`：

```js
export class Bidder {
  _id:            string;
  profileId:      string;
  additionalData: any;
  auctions:       Array<any>;
  createdAt:      string

  constructor(
    _id?:            string,
    profileId?:      string,
    additionalData?: any,
    auctions?:       Array<any>,
    createdAt?:      string
  ) {
    this._id = _id;
    this.profileId = profileId;
    this.additionalData = additionalData;
    this.auctions = auctions;
    this.createdAt = createdAt;
  }
}
```

# 拍卖模块

我们已经采取了初步步骤并实现了我们的服务。现在我们可以开始在组件中使用它们了。在我们的 `Auction` 应用程序中有很多动态元素。应用程序中最具挑战性的部分将是拍卖详情页面。

上述代码将列出特定拍卖的详细信息，并列出当前的出价。当放置新的出价时，它将被推送到 `bids` 列表。

在我们的服务中，我们之前使用了 `Auction` 模型。让我们首先看看它，它位于 `public/src/auction/auction.model.ts`：

```js
import { Money } from '../common/index';

export class Auction {
  _id:            string;
  identifier:     string;
  item:           any;
  startingPrice:  any;
  currentPrice:   any;
  endPrice:       any;
  minAmount:      any;
  bids:           Array<any>;
  status:         string;
  startsAt:       string;
  endsAt:         string;
  createdAt:      string

  constructor(
    _id?:            string,
    item?:           any,
    startingPrice?:  any,
    currentPrice?:   any,
    endPrice?:       any,
    minAmount?:      any,
    bids?:           Array<any>,
    status?:         string,
    startsAt?:       string,
    endsAt?:         string,
    createdAt?:      string,
    identifier?:     string
  ) {
    this._id = _id;
    this.item = item || { slug: '' };
    this.startingPrice = startingPrice || new Money();
    this.currentPrice = currentPrice || this.startingPrice;
    this.endPrice = endPrice || new Money();
    this.minAmount = minAmount || new Money();
    this.bids = bids;
    this.status = status;
    this.startsAt = startsAt;
    this.endsAt = endsAt;
    this.createdAt = createdAt;
    this.identifier = identifier || `${this.item.slug}-${this._id}`;
  }
}
```

它有很多属性。当我们实例化模型时，我们进行一些初始化。我们使用一个自定义的 `Money` 模型，它反映了后端的自定义货币类型。

如果你记得，在 `Job Board` 应用程序中，我们使用了漂亮的 URL 来访问一家公司。我想保持同样的外观，但添加一点变化来尝试不同的结构。我们有相同的概念，但拍卖有一个不同的标识符。

我们正在使用产品的 slug 与拍卖的 `_id` 结合来作为我们的 `identifier` 属性。现在让我们看看 `Money` 模型，位于 `public/src/common/money.model.ts`：

```js
export class Money {
  amount: number;
  currency: string;
  display: string;
  factor: number;

  constructor(
    amount?: number,
    currency?: string,
    display?: string,
    factor?: number
  ) {
    this.amount = amount;
    this.currency = currency;
    this.display = display;
    this.factor = factor;
  }
}
```

如你所记，我们正在使用这些技术来为我们的对象提供初始值，并确保我们有必要的属性。为了刷新我们的记忆，`amount` 是通过将 `display` 值与 `factor` 相乘得到的。所有这些都是在服务器端完成的。

## 基本组件

我们将添加一个基本组件来配置我们的路由。我们的基本组件通常非常基础，没有太多逻辑；它只有与路由相关的逻辑。创建一个名为 `public/src/auction/components/auction-base.component.ts` 的新文件，并添加以下代码：

```js
import { Component } from 'angular2/core';
import { RouteConfig, RouterOutlet } from 'angular2/router';
import { AuctionListComponent } from './auction-list.component';
import { AuctionDetailComponent } from './auction-detail.component';

@RouteConfig([
  { path: '/', as: 'AuctionList', component: AuctionListComponent, useAsDefault: true },
  { path: '/:identifier', as: 'AuctionDetail', component: AuctionDetailComponent }
])
@Component({
    selector: 'auction-base',
    directives: [
      AuctionListComponent,
      AuctionDetailComponent,
      RouterOutlet
    ],
    template: `
      <div class="col">
        <router-outlet></router-outlet>
      </div>
    `
})
export class AuctionBaseComponent {
  constructor() {}
}
```

## 拍卖列表

为了显示当前可用的拍卖列表，我们将创建一个新的组件，名为 `public/src/auction/components/auction-list.component.ts`：

```js
import { Component, OnInit } from 'angular2/core';
import { AuctionService } from '../auction.service';
import { Router, RouterLink } from 'angular2/router';
import { Auction } from '../auction.model';

@Component({
    selector: 'auction-list',
    directives: [RouterLink],
    template: `
      <div class="auction-list row">
        <h2 class="col">Available auctions</h2>
        <div *ngFor="#auction of auctions" class="col col-25">
          <h3>
            <a href="#"
              [routerLink]="['AuctionDetail', { identifier: auction.identifier }]">
              {{ auction.item.title }}
            </a>
          </h3>
          <p>starting price: {{ auction.startingPrice.display }} {{ auction.startingPrice.currency }}</p>
        </div>
      </div>
    `
})
export class AuctionListComponent implements OnInit {
  public auctions: Array<Auction> = [];
  private _auctionService: AuctionService;

  constructor(auctionService: AuctionService) {
    this._auctionService = auctionService;
  }

  ngOnInit() {
    this._auctionService.auctions.subscribe((auctions: Array<Auction>) => {
      this.auctions = auctions;
    });
    this._auctionService.getAll();
  }
}
```

从这个组件，我们将链接到拍卖详情。正如你所见，我们使用了 `identifier` 作为路由参数。属性值是在 `Auction` 模型内部构建的。

## 详情页面

在这个应用程序中，详情页面将包含最多的动态部分。我们将显示拍卖的详细信息并列出所有新的出价。此外，用户也可以从该页面进行出价。为了实现这个组件，让我们遵循以下步骤：

1.  创建一个名为 `public/src/auction/components/auction-detail.component.ts` 的新文件。

1.  添加依赖项：

    ```js
    import { Component, OnInit } from 'angular2/core';
    import { AuctionService } from '../auction.service';
    import { RouterLink, RouteParams } from 'angular2/router';
    import { Auction } from '../auction.model';
    import { BidListComponent } from '../../bid/index';
    import { BidFormComponent } from '../../bid/index';
    ```

1.  配置 `Component` 注解：

    ```js
    @Component({
        selector: 'auction-detail,
        directives: [
          BidListComponent,
          BidFormComponent,
          RouterLink
        ],
        template: `
          <div class="col">
            <a href="#" [routerLink]="['AuctionList']">back to auctions</a>
          </div>
          <div class="row">
            <div class="col sidebar">
              <div class="auction-details">
                <h2>{{ auction.item.title }}</h2>
                <p>{{ auction.startingPrice.display }} {{ auction.startingPrice.currency }}</p>
                <p>{{ auction.currentPrice.dislpay }} {{ auction.startingPrice.currency }}</p>
                <p>minimal bid amount: {{ auction.minAmount.display }}</p>
              </div>
            </div>
            <div class="col content">
              <bid-list></bid-list>
              <bid-form></bid-form>
            </div>
          </div>
        `
    })
    ```

1.  添加类：

    ```js
    export class AuctionDetailComponent implements OnInit, OnDestroy {
      public auction: Auction;
      private _routeParams:RouteParams;
      private _auctionService: AuctionService;

      constructor(
        auctionService: AuctionService, 
        routeParams: RouteParams
      ) {    
        this._auctionService = auctionService;
        this._routeParams = routeParams;
      }
    }
    ```

1.  实现 `ngOnInit`：

    ```js
      ngOnInit() {
        this.auction = new Auction();
        const identifier: string = this._routeParams.get('identifier');
        const auctionId = this.getAuctionId(identifier);
        this._auctionService.auction.subscribe((auction: Auction) => {
          this.auction = auction;
        });
        this._auctionService.getOne(auctionId);
      }
    ```

1.  添加 `ngOnDestroy`：

    ```js
      ngOnDestroy() {
        this._auctionService.setCurrentAuction(new Auction());
      }
    ```

    当组件被销毁时，我们希望将 `currentAuction` 设置为空。

1.  定义私有的 `getAuctionId` 方法：

    ```js
      private getAuctionId(identifier: string) {
        const chunks = identifier.split('-');
        return chunks[chunks.length -1];
      }
    ```

我们使用 `RouterParams` 来获取标识符。因为我们有一个很好的 URI，我们只需要从标识符中剥离必要的信息。为此，我们使用了一个私有方法，该方法将 URL 组件分割成块，并只获取最后一部分。

URL 的最后一部分是拍卖的 `id`。在获得必要的 `id` 后，我们可以从我们的 API 中检索信息。

此组件使用了两个其他组件，`BidListComponent` 和 `BidFormComponent`。第一个用于显示出价列表，监听出价数据流，并更新出价列表。

第二个，`BidFormComponent`，用于出价。将所有功能封装到单独的组件中更容易。这样，每个组件都可以专注于其领域需求。

# 出价模块

我们将用 `bid` 模块来结束这一章，因为我们已经在之前的 `auction` 模块中使用了它的许多组件。这里只讨论 `bid listing`，因为它涉及到与底层套接字流的工作。

## 列出出价

从之前的 `AuctionDetailComponent` 我们可以看到，这个组件将以出价为输入。这些数据来自 `auction` 实体，它保存了之前放置的出价。

创建一个名为 `public/src/bid/components/bid-list.component.ts` 的新文件：

```js
import { Component, OnInit, OnDestroy } from 'angular2/core';
import { BidService } from '../bid.service';
import { Bid } from '../bid.model';
import { BidComponent } from './bid.component';

@Component({
    selector: 'bid-list',
    inputs: ['bids'],
    directives: [BidComponent],
    template: `
      <div class="bid-list">
        <div *ngIf="bids.length === 0" class="empty-bid-list">
          <h3>No bids so far :)</h3>
        </div>
        <bid *ngFor="#bid of bids" [bid]="bid"></bid>
      </div>
    `
})
export class BidListComponent implements OnInit, OnDestroy {
  public bids: Array<Bid>;
  private _bidService: BidService;
  private _subscription: any;

  constructor(bidService: BidService) {
    this._bidService = bidService;
  }

  ngOnInit() {
    this._subscription = this._bidService.bid.subscribe((bid) => {
      this.bids.push(bid);
    });
  }

  ngOnDestroy() {
    if (this._subscription) {
        this._subscription.unsubscribe();
    }
  }
}
```

我们订阅了来自 `BidService` 的 `bid` 数据流，以推送所有新的 incoming bids 并使用 `BidComponent` 显示它们。订阅也被存储起来，这样我们可以在组件销毁时取消订阅流。

## 出价组件

我们的出价组件将会相当简单。它将有一个 `bid` 输入，在视图初始化成功后，我们将滚动到出价列表视图的底部。让我们在 `public/src/bid/components/bid.component.ts` 下创建以下组件：

```js
import { Component, AfterViewInit } from 'angular2/core';
import { Bid } from '../bid.model';

@Component({
    inputs: ['bid'],
    selector: 'bid',
    template: `
      <div class="bid-item">
        <div class="">
          <span class="">{{bid_id}}</span>
          <span class="">{{bid.amount}}</span>
        </div>
      </div>
    `
})
export class BidComponent implements AfterViewInit {
  public bid: Bid;

  constructor() {}

  ngAfterViewInit() {
    var ml = document.querySelector('bid-list .bid-list');
    ml.scrollTop = ml.scrollHeight;
  }
}
```

还让我们看看我们的 `bid` 模型，`public/bid/bid.model.ts`：

```js
export class Bid {
  _id:            string;
  bidder:         any;
  amount:         any;
  createdAt:      string

  constructor(
    _id?:         string,
    bidder?:      any,
    amount?:      any,
    createdAt?:   string
  ) {
    this._id = _id;
    this.bidder = bidder;
    this.amount = amount;
    this.createdAt = createdAt;
  }
}
```

现在我们已经从后端到我们的前端组件完成了一次完整的往返。数据从 WebSocket 服务器流到我们的 Angular 2 应用程序。

此应用程序的目的是浏览书中使用的所有技术，我们有机会组装一个概念验证。本章的主要重点是查看底层模块，它们将如何组合，以及数据如何在各个模块之间建模和传输。

# 摘要

这是我们的最后一章，我们创建了一个小的概念验证应用程序。目的是浏览书中的一些最有趣的部分和方法，看看我们如何将激动人心的想法结合起来，创建出既小又强大的东西。

此外，我们使用了现有的电子商务 API 来检索有关产品项的信息并管理我们的用户。我们没有理由再次经历这个过程，因为我们可以在快速原型设计时依赖第三方 API。

在大多数章节中，我们只触及了最重要的部分。每个章节所需的所有代码都可以在 Packt Publishing 网站上找到（[`www.packtpub.com/`](https://www.packtpub.com/))。
