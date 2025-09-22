# 第八章。掌握异步性

我们的 JavaScript 入门指南(第三章)涵盖了所有重要的概念，使我们能够开始构建我们的应用程序。但 JavaScript 编程的一个基本方面值得更深入地探索：异步性。

第一章，*为什么选择 Node.js？*，讨论了 Node.js 的异步编程模型。它描述了 Node.js API 和第三方库中使用的统一方法。回想一下，每个异步方法都接受一个回调函数，该函数传递错误和结果参数，例如我们在第一章，*为什么选择 Node.js？*中看到的`fs.stat`函数：

```js
fs.stat('/hello/world', function (error, stats) {
  console.log('File last updated at: ' + stats.mtime);
});
```

然而，回调模式有一些弱点。执行错误处理和组合多个异步操作的结果可能会变得相当笨拙。JavaScript 中有一些替代的异步模式可以解决这些问题。多个竞争模式本身可能看起来令人担忧。在第一章，*为什么选择 Node.js？*中讨论的 Node.js 的一个好处是拥有一个单一的一致方法。

我们还应该重新审视 Node.js API 和库始终异步这一想法。我们需要考虑这如何应用到我们自己的代码中。这不仅仅是在为第三方编写模块时需要担心的事情。即使在我们的应用程序内部，大多数模块也需要通过异步接口公开其功能。如果不这样做，我们将严重限制我们实现这些模块的自由度。

在本章中，我们将涵盖以下主题：

+   向我们的模块引入异步接口

+   观察回调模式的某些弱点

+   通过重构移除回调，使我们的异步代码更易读

+   看看我们如何仍然可以从 Node.js 异步编程模型的一致性中受益

# 使用回调模式进行异步代码

让我们看看我们游戏服务中的一种方法：

```js
module.exports.get = (id) => games.find(game => game.id === id);
```

这个函数的接口是同步的：你调用它并返回一个值。第四章，*介绍 Node.js 模块*，介绍了游戏服务作为负责我们存储游戏的模块。如果我们更改存储实现，接口不需要改变。然而，目前情况并非如此。

如前所述，大多数 Node.js 库都是异步的。同步接口无法利用异步实现。假设`get`函数想要在第三方`datastore`库中使用异步方法。那会是什么样子？以下（非工作）代码中的注释描述了问题：

```js
module.exports.get = (id) => {
    datastore.getById(id, (err, result) => {
        // Result available, but outer method has already returned
    });
    return ???; // Need to return here, but have no result yet
};
```

这是一个普遍存在的问题，不仅仅是在 JavaScript 中。在其他平台上，你可以延迟返回，直到异步操作完成。这会将异步操作转换为阻塞操作。在 Node.js（以及其他 JavaScript 环境中），这种方式是不可行的。它将与单线程、非阻塞、事件驱动的执行模型不兼容。

## 暴露回调模式

为了使我们的游戏服务能够使用异步库，我们需要给它一个异步接口。请注意，Node.js 生态系统中的几乎所有库都是异步的。如果不是，它们将受到与我们的游戏服务当前相同的限制。

我们可以将我们的`get`函数的接口重写为遵循标准的异步回调模式。让我们看看这会对使用异步第三方`datastore`库产生什么影响（再次强调，以下代码是无效的，包含一个虚构的`datastore`对象）：

```js
module.exports.get = (id, callback) => {
  datastore.getById(id, (err, result) => {
    // Can now make use of the result by passing to the callback
    callback(err, result);
  }
  // No longer need to return here
}
```

当然，在这种情况下，我们可以将前面的代码简化如下：

```js
module.exports.get = (id, callback) => {
    datastore.getById(id, callback);
}
```

然而，通常我们可能想要对第三方库的结果进行更多处理。因此，我们的函数可能看起来更像是这样：

```js
module.exports.get = (id, callback) => {
    datastore.getById(id, (err, result) => {
        if (err) {
            callback(err);
        } else {
            callback(null, processResult(result));
        }
    }
}
```

假设`processResult`是我们模块内部的一个函数，它现在有一个同步接口是完全可以的。如果它需要稍后执行异步工作，我们可以更改其接口，而不会影响我们模块的消费者。

我们的游戏服务模块的**公共**接口确实需要完全异步。我们还没有真正改变模块的实现。这使得更新接口变得相当直接。我们可以在`src/services/games.js`中做出以下更改：

```js
'use strict';

const games = [];
let nextId = 1;

class Game {
    ...

 remove(callback) {
        games.splice(games.indexOf(this), 1);
 callback();
    }
}

module.exports.create = (userId, word, callback) => {
    const newGame = new Game(nextId++, userId, word); 
    games.push(newGame);
 callback(newGame);
};
module.exports.get = (id, callback) =>
 callback(null,
 games.find(game => game.id === parseInt(id, 10)));
module.exports.createdBy = (userId, callback) =>
 callback(null, games.filter(game => game.setBy === userId));
module.exports.availableTo = (userId, callback) =>
 callback(null, games.filter(game => game.setBy !== userId));

```

注意，这虽然有些不切实际。通常，在异步方法完成之前，控制权会返回给调用者。我们可以通过使用`process.nextTick`在事件循环的下一次 tick 上安排回调的执行来实现这一点（如果你想要复习事件循环，请参阅第一章，*为什么 Node.js?*）：

```js
'use strict';

const games = [];
let nextId = 1;

const asAsync = (callback, result) =>
 process.nextTick(() => callback(null, result));

class Game {
    ...

    remove(callback) {
        games.splice(games.indexOf(this), 1);
 asAsync(callback);
    }
}

module.exports.create = (userId, word, callback) => {
    let game = new Game(nextId++, userId, word);
    games.push(game);
 asAsync(callback);
};
module.exports.get = (id, callback) =>
 asAsync(callback,
 games.find(game => game.id === parseInt(id, 10)));
module.exports.createdBy = (userId, callback) =>
 asAsync(callback, games.filter(game => game.setBy === userId));
module.exports.availableTo = (userId, callback) =>
 asAsync(callback, games.filter(game => game.setBy !== userId));

```

将我们的应用程序的其余部分更新为消费这个异步接口是一个更复杂的任务。这就是为什么始终编写从开始就是异步的模块接口是值得的。我们绝对应该在进一步扩展我们的应用程序之前解决这个问题。

## 消费异步接口

游戏服务被游戏路由、索引路由以及我们的测试调用。让我们依次查看这些更改。以下代码来自`src/routes/games.js`：

```js
'use strict';

const express = require('express');
const router = express.Router();
const service = require('../service/games.js');

router.post('/', function(req, res, next) {
    let word = req.body.word;
    if (word && /^[A-Za-z]{3,}$/.test(word)) {
 service.create(req.user.id, word, (err, game) => {
 if (err) {
 next(err);
 } else {
 res.redirect(`/games/${game.id}/created`);
 }
 });
    } else {
        res.status(400).send('Word must be at least three characters long and contain only letters');
    }
});

const checkGameExists = function(id, res, onSuccess, onError) {
 service.get(id, function(err, game) {
 if (err) {
 onError(err);
 } else {
 if (game) {
 onSuccess(game);
 } else {
 res.status(404).send('Non-existent game ID');
 }
 }
 });
};

router.get('/:id', function(req, res, next) {
    checkGameExists(
        req.params.id,
        res,
        game => { ... },
 next);
});

router.post('/:id/guesses', function(req, res, next) {
    checkGameExists(
        req.params.id,
        res,
        game => { ... },
 next);
});

router.delete('/:id', function(req, res, next) {
    checkGameExists(
        req.params.id,
        res,
        game => {
            if (game.setBy === req.user.id) {
 game.remove((err) => {
 if (err) {
 next(err);
 } else {
 res.send();
 }
 });
            } else {
                res.status(403).send('You don't have permission...');
            }
        },
 next);
});

router.get('/:id/created', function(req, res, next) {
    checkGameExists(
        req.params.id,
        res,
        game => res.render('createdGame', game),
 next);
});

module.exports = router;
```

在这种情况下，更改很简单。现在对游戏服务函数的每次调用都传递一个回调。回调包含原本跟随游戏服务函数调用的逻辑。每个回调还需要处理错误值的可能性。在这种情况下，我们只需将其传递给 Express 的`next`回调，这样它就会被我们的全局错误处理器处理。

虽然这些变化很简单，但它们在我们的代码中引入了一些重复的模板代码。在索引路由中，这个问题更为严重；看看`src/routes/index.js`中的代码：

```js
var express = require('express');
var router = express.Router();
var games = require('../service/games.js');

router.get('/', function(req, res, next) {
 games.createdBy(req.user.id, (err, createdGames) => {
 if (err) {
 next(err);
 } else {
 games.availableTo(req.user.id, (err, availableGames) => {
 if (err) {
 next(err);
 } else {
 res.render('index', {
 title: 'Hangman',
 userId: req.user.id,
 createdGames: createdGames,
 availableGames: availableGames,
 partials: { createdGame: 'createdGame' }
 });
 }
 });
 }
  });});

module.exports = router;
```

在这里，我们需要合并两个不同异步调用的结果。这导致了嵌套回调。我们还得在每个阶段重复错误处理代码。注意，我们只有在第一个异步操作完成后才开始第二个异步操作。并行启动操作会更好。

回想一下，虽然 JavaScript 本身是单线程的，但异步操作可以在并行中执行工作，例如网络、磁盘和其他 I/O 操作。并行运行多个操作需要更复杂的（且容易出错的）模板代码。为了说明这可能如何工作，考虑将游戏服务测试中的`beforeEach`函数变为异步操作所做的更改。以下代码来自`src/test/services/games.js`：

```js
describe('Game service', function() {
    let firstUserId = 'user-id-1';
    let secondUserId = 'user-id-2';

 beforeEach(function(done) {
 service.availableTo('not-a-user', (err, gamesAdded) => {
 let gamesDeleted = 0;
 if (gamesAdded.length === 0) {
 done();
 }
 gamesAdded.forEach(game => {
 game.remove(() => {
 if (++gamesDeleted === gamesAdded.length) {
 done();
 }
 });
 });
 });
 });

    ...
});
```

在这里，我们需要对异步的移除方法进行未知数量的调用。当所有调用都完成时，必须调用`done`回调。有几种实现方式，但它们都涉及额外的模板代码。这里的方法是最简单的，通过计数完成操作的次数。另外，请注意，我们省略了错误处理，因为这是测试代码。在生产代码中，我们还需要考虑错误处理，这使得事情变得更加复杂。

### 注意

测试中还有一些其他的变化，以利用游戏服务的新的异步接口。为了简洁，这里省略了这些变化。它们与`index.js`中的变化类似。您可以通过查看 Git 仓库中该章节的第一个提交来查看完整的变更集，Git 仓库地址为[`github.com/NodeJsForDevelopers/chapter08`](https://github.com/NodeJsForDevelopers/chapter08)。

所有这些都显得相当令人不满意。我们的代码变得更加复杂、重复，且难以阅读。幸运的是，我们可以通过使用不同的方法来编写异步代码来解决这些问题。

# 使用承诺（promises）编写更干净的异步代码

**承诺（Promises**）是编写异步代码的回调模式的替代方案。承诺代表一个尚未完成但预期将来会完成的操作。正如其名“承诺”所暗示的，承诺是一个最终提供值或失败原因（即错误）的合同。您可能已经从.NET 中的任务或 Java 中的未来（Futures）中熟悉了这种模式。承诺有三个可能的状态：

+   **pending** 表示一个进行中的操作

+   **fulfilled** 表示一个成功的操作，带有结果值

+   **rejected** 表示一个不成功的操作，带有失败原因

当执行单个操作时，基于回调和基于承诺的方法看起来非常相似。承诺的力量在于组合异步操作。

考虑一个例子，我们有一个异步库函数用于获取、处理和聚合数据。我们希望依次执行这些操作，然后显示结果，并在执行过程中处理错误。使用回调，代码可能看起来像这样（以下为不可运行的虚构代码）：

```js
lib.getInitialData(function(e, initialData) {
  if (e) {
    console.log('Error: ' + e);
  } else {
    lib.processData(initialData, (e, processedData) => {
      if (e) {
        console.log('Error: ' + e);
      } else {
        lib.aggregateData(processedData, (e, aggregatedData) => {
          if (e) {
            console.log('Error: ' + e);
          } else {
            console.log('Success! Result=' + aggregatedData);
          }
        });
      }
    });
  }
});
```

这与我们之前章节中自己代码中遇到的问题有很多相同之处：嵌套回调、额外的样板代码和重复的错误处理。如果这些函数返回承诺，那么上述代码的等效代码如下：

```js
lib.getInitialData()
    .then(lib.processData)
    .then(lib.aggregateData)
    .then(function(aggregatedData) {
        console.log('Success! Result=' + result);
    }, function(error) {
        console.log('Error: ' + error);
    });
```

`then`函数将一个函数应用到承诺的结果值上，并返回一个新的承诺。通过这种方式，我们构建了一系列操作表示的承诺链。

`then`函数接受两个参数，这两个参数都是回调。如果异步操作返回错误，则将调用第二个参数。在上面的例子中，如果`library.aggregateData`调用失败，我们将记录一个错误。

如果省略了第二个`then`回调参数，任何错误都会沿着承诺链传播。在上面的例子中，这意味着如果`library.processData`调用失败，则不会调用`library.aggregateData`，并且我们的错误日志回调仍然会被调用。

如果你只关心错误情况，你可以直接使用`catch`函数指定一个错误回调，而不是使用`then`。你还可以结合传播机制来更清晰地重写前面的代码：

```js
library.getInitialData()
    .then(library.processData)
    .then(library.aggregateData)
    .then(function(aggregatedData) {
        console.log('Success! Result=' + result);
 })
 .catch(function(error) {
 console.log('Error: ' + error);
    });
```

在这里，任何位置的错误都会传播到最后一个承诺，我们检查是否有错误。请注意，这个重写的版本也会捕获成功日志回调抛出的任何错误，而前面的版本则不会。你应该始终在承诺链的末尾调用`catch`，除非你正在返回结果承诺对象以供其他地方消费。

## 实现基于承诺的异步代码

让我们将承诺模式应用到我们现有的应用程序中。首先，我们需要更新我们的游戏服务 API，以暴露承诺而不是回调。和之前一样，这很简单，因为我们的游戏服务在实现中实际上并没有使用任何异步操作（目前还没有）。我们的游戏服务的基于承诺的版本如下（在`src/services/games.js`中）：

```js
'use strict';

const games = [];
let currentId = 1;

class Game {
    ...

 remove() {
        games.splice(games.indexOf(this), 1);
 return Promise.resolve();
    }
}

module.exports.create = (userId, word) => {
    const newGame = new Game(nextId++, userId, word); 
    games.push(newGame);
 return Promise.resolve(newGame);
};
module.exports.get = (id) =>
 Promise.resolve(
 games.find(game => game.id === parseInt(id, 10)));
module.exports.createdBy = (userId) =>
 Promise.resolve(games.filter(game => game.setBy === userId));
module.exports.availableTo = (userId) =>
 Promise.resolve(games.filter(game => game.setBy !== userId));

```

创建基于承诺的接口甚至比基于回调的接口更简单。我们可以使用`Promise.resolve()`函数为已知值创建一个承诺。我们游戏服务中的每个函数看起来都和原始的同步版本很相似，只是多了一个调用`Promise.resolve`。

### 注意

如果你将承诺参数传递给`Promise.resolve`，那么你将得到一个行为与原始参数相同的承诺。如果你传递任何其他值，你将得到一个已解析的承诺，该承诺对应于该值。这在你需要操作可能是一个承诺或值的变量时很有用。你可以将其传递给`Promise.resolve`，然后一致地将其作为承诺处理。

### 消费承诺模式

现在我们需要更新我们的代码库的其余部分以使用承诺。让我们看看之前的相同文件，从游戏路由开始。请看以下来自 `src/routes/games.js` 的代码：

```js
'use strict';

const express = require('express');
const router = express.Router();
const service = require('../service/games.js');

router.post('/', function(req, res, next) {
    let word = req.body.word;
    if (word && /^[A-Za-z]{3,}$/.test(word)) {
 service.create(req.user.id, word)
 .then(game =>
 res.redirect(`/games/${game.id}/created`))
 .catch(next);
    } else {
        res.status(400).send('Word must be at least three characters long and contain only letters');
    }
});

const checkGameExists = function(id, res, onSuccess, onError) {
 service.get(id)
 .then(game => {
 if (game) {
 onSuccess(game);
 } else {
 res.status(404).send('Non-existent game ID');
 }
 })
 .catch(onError);
};

...

router.delete('/:id', function(req, res, next) {
    checkGameExists(
        req.params.id,
        res,
        game => {
            if (game.setBy === req.user.id) {
 game.remove()
 .then(() => res.send())
 .catch(next);
            } else {
                res.status(403).send('You do not have permission to delete this game');
            }
        },
        next);
});
```

这个文件之前是最简单的，所以这里的变化最少。我们仍然有一些样板代码的重复（例如，`catch` 调用）。尽管如此，基于承诺的方法比回调更紧凑且易于阅读。现在让我们看看来自 `src/routes/index.js` 的索引路由代码：

```js
var express = require('express');
var router = express.Router();
var games = require('../service/games.js');

router.get('/', function(req, res, next) {
 games.createdBy(req.user.id)
 .then(gamesCreatedByUser => 
 games.availableTo(req.user.id)
 .then(gamesAvailableToUser => {
 res.render('index', {
 title: 'Hangman',
 userId: req.user.id,
 createdGames: gamesCreatedByUser,
 availableGames: gamesAvailableToUser
 });
 }))
 .catch(next);
});

module.exports = router;
```

这有点改进。重复较少，但仍然有一些嵌套和样板代码。注意最外层的 `then` 回调返回一个承诺（从 `games.availableTo` 连接）。当一个 `then` 回调返回一个承诺时，这实际上会被扁平化，因此整体承诺返回内部承诺的值。这种扁平化也适用于错误的传播，因此我们不需要在内部承诺上显式调用 `catch`。

这段代码仍然有点难以理解。实际上有一种方法可以使它更容易阅读，我们稍后会回到这个问题。首先，让我们看看以下来自 `test/service/games.js` 的游戏服务测试中的 `beforeEach` 函数：

```js
describe('Game service', function() {
    let firstUserId = 'user-id-1';
    let secondUserId = 'user-id-2';

    beforeEach(function(done) {
 service.availableTo('non-existent-user')
 .then(games => games.map(game => game.remove()))
 .then(gamesRemoved => Promise.all(gamesRemoved))
 .then(() => done(), done);
    });
});
```

这已经变得非常短且线性。让我们分析每一行的作用：

+   `service.availableTo` 返回一个包含游戏数组的承诺

+   第一个 `then` 回调使用 `array.map` 将其转换为 *包含删除操作承诺的数组承诺*

+   下一个 `then` 回调使用 `Promise.all` 将其转换为整个删除操作数组的单个承诺

    ### 注意

    `Promise.all` 函数接受一个承诺数组，并在数组中所有承诺都解析或数组中的任何承诺被拒绝时立即拒绝返回一个承诺。

+   当 `Promise.all` 返回的承诺解析时，即所有删除操作完成时，将调用 Mocha 的 `done` 回调

注意，与基于回调的方法不同，错误处理也非常简单。我们只需将 `done` 回调作为错误处理程序（第二个参数）传递给最终的 `then` 调用。我们可以在测试本身中采取类似的方法，就像我们在 `beforeEach` 回调中所做的那样。再次提醒，为了简洁，省略了测试的更新，但你可以从书籍的配套代码中找到它们。

## 使用承诺并行化操作

我们还可以利用 `Promise.all` 函数简化索引路由。回想一下，我们的代码是依次调用两个异步操作。在基于回调的方法中，尝试并行执行这些操作会使代码更加复杂。使用承诺，实际上使我们的代码更易于阅读：

```js
var express = require('express');
var router = express.Router();
var games = require('../service/games.js');

router.get('/', function(req, res, next) {
 Promise.all([
 games.createdBy(req.user.id),
 games.availableTo(req.user.id)
 ])
 .then(results => {
 res.render('index', {
 title: 'Hangman',
 userId: req.user.id,
 createdGames: results[0],
 availableGames: results[1]
 });
 })
 .catch(next);
});

module.exports = router;
```

这更短，更容易理解。我们启动两个异步操作来加载数据，然后在两个操作都完成后立即使用这些数据。

### 小贴士

前述方法的唯一小缺点是我们必须通过索引从数组中获取两个值。在 Node.js v6 或更高版本中，我们可以通过使用 **解构** 来从数组中的值分配两个命名参数，从而避免这种情况，并使代码更加易于阅读，如下所示：

```js
        .then(([created, available]) => { ...
```

这在上述示例中没有用于与 Node.js v4 的向后兼容性。我们将在第十四章（[part0081.xhtml#aid-2D7TI1 "第十四章. Node.js 和更远"]）中更详细地讨论解构，*Node.js 和更远*。

# 组合异步编程模式

承诺（Promises）使我们能够解决回调模式的一些不足，并编写更易于阅读的代码。然而，我们遇到了一个新的问题。Node.js 的一大优点是它对异步编程的一致性方法。通过引入承诺以及传统的回调模式，我们似乎已经否定了这一点。

此外，尽管原生的承诺是 ECMAScript 2015 中的新特性，但这个概念并不新鲜。有许多现有的库提供了它们自己的承诺实现。

幸运的是，这些异步编程的竞争方法实际上非常一致。Node.js 风格回调模式一致性的最大价值来自于以下方面：

+   所有库函数默认都是异步的（非阻塞的）

+   所有异步操作都返回单个值或错误

承诺与上述点完全一致。JavaScript 中不同承诺实现的兼容性也非常好。这要归功于 Promises/A+ 规范（[`promisesaplus.com`](http://promisesaplus.com)）。这本质上定义了 `then` 方法的行为。你可能会遇到的任何承诺库都将遵循此规范。原生的 JavaScript 承诺也设计为与之兼容。这意味着所有这些库和原生 JavaScript 承诺都是可互操作的。

因此，所有使用回调的库都遵循相同的约定，所有承诺库都遵循相同的规范。唯一剩下的问题是转换承诺和回调。有几个承诺库可以为我们完成这项工作。

如果你只想将几个标准回调函数转换为承诺，可以使用 `denodeify`，它可以通过 npm 安装。我们之前提到的 `fs.stat` 示例将看起来像这样：

```js
const denodeify = require('denodeify');
const stat = denodeify(require('fs').stat));
stat('/hello/world')
    .then(stats => console.log('File last updated at: ' + stats.mtime));
```

你还会发现许多库公开了可以返回承诺或接受回调的函数，因此可以使用任一模式调用。

# 摘要

在本章中，我们看到了如何在我们的模块中公开标准的 Node.js 回调接口。我们使用了承诺来生成更易于阅读的异步代码。最后，我们看到了如何结合使用承诺和标准的 Node.js 回调。

现在我们能够实现自己的异步 API，我们可以扩展我们的应用程序并开始使用提供异步接口的其他库。在下一章中，我们将利用这一点向我们的应用程序引入持久存储。
