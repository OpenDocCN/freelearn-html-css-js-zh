# 第二十二章：状态

现在我们已经在不同的框架上创建了我们的微服务基础布局，是时候更仔细地阅读我们的代码，看看一切是否看起来都很好了。只是继续编写代码而停止思考我们在做什么是很容易的，但后来当我们停下来的时候，我们会浪费时间删除重复的代码和重新组织我们的服务。

在我们编码之前总是先思考更好。这是随着时间的推移你会学到的东西，那就是重视你用来规划服务或思考新功能的时间。直接开始编码从来不是一个好主意。理论上，你的**服务**应该位于**安全**层内部，与**状态**有良好且稳定的连接：

![图片](img/0fafb30f-c4b5-4361-a42c-fc1ca74f7259.png)

# 状态

将状态视为一个人的记忆。通常，一个服务具有状态，这意味着它有它所提供的行为和信息记忆。我们的服务将无限期运行的设想是，但有时我们被迫重新启动它，甚至因为维护或升级而停止它一段时间。

理想情况下，一个服务应该在不丢失状态的情况下恢复，给用户一种它从未停止的印象。这是通过做以下两件事之一来实现的：

+   将状态存储在持久存储中

+   在停止之前将状态保存在持久存储中，并在重启后加载该状态

第一种选择会使你的服务稍微慢一些（没有比系统内存中的状态更快的东西），但应该在你重启时提供更一致的状态。

第二种选择更复杂，因为有时我们的服务可能会突然停止，无法保存该状态，但对于这些用例，你可能并不关心状态。这取决于你。

存储状态有很多选择；这取决于你想要存储什么。对于一个微服务，你应该避免文件系统，以便使你的服务与多个操作系统更兼容。

# 存储状态

根据你的服务，你可以使用以下方式存储状态：

+   **关系型数据库管理系统**（**RDBMS**），例如 MySQL 或 PostgreSQL

+   **非关系型数据库管理系统**，或 NoSQL，例如 MongoDB 或 RethinkDB

+   **内存数据库**（**IMDB**），例如 Redis 或 Memcached

第一种选择仍然是使用最广泛的一种。你将依赖于稳定且经过充分证明的数据库系统，这些系统在多个系统中运行，并且你可以在任何你想要部署微服务的云服务中找到。除了大多数解决方案的成熟度之外，如果设置得当，关系数据库应该给你一致性。

与第一种选择相比，第二种选择更近。通常，没有像 RDBMS 那样的固定表，你通常与文档集合一起工作，这些文档只是常见的 JSON 结构。由于通常没有限制，并且每个文档可能具有不同的结构，因此它更灵活。越灵活，一致性越低。

根据您选择的特定系统，所有三种选项都支持复制，这应该能够实现容错并提高地理上分散的实例的速度。

让我们尝试使用建议的系统之一来使用这三种选项中的每一个。让我们从关系型数据库开始，并使用 MySQL。

# MySQL

安装 MySQL 非常简单。只需访问官方网站并按照说明操作。您通常会被要求为 root 用户设置密码，您以后可以使用它来管理服务器设置和用户账户。

使用 Node.js 连接到 MySQL 服务器有一些选项，但最好的工具是 `mysql` 和 `mysql2` 模块。它们都满足所需的功能，并且都不是对方的下一个版本，它们只是在设计和支持的功能上略有不同。

首先，让我们将依赖项添加到我们的服务中。在终端中，进入我们的服务文件夹并输入：

```js
npm install mysql --save
```

我们现在可以包含我们的依赖项并配置数据库连接。为了避免在代码中包含凭证，我们可以创建一个单独的文件，并将设置放在那里，我们可以在将来更改这些设置，而且这些设置不应该包含在代码中。我们可以利用 Node.js 能够包含 JSON 文件的能力，并直接将设置写入 JSON。

创建一个名为 `settings.json` 的文件，并添加以下内容：

```js
{
    "db": "mysql://root:test@localhost/imagini"
}
```

我们定义了一个名为 `db` 的设置，它有一个数据库 URI，这是一种方便的方法，使用类似于任何网站地址的地址来定义我们的数据库访问和凭证。我们的数据库使用 `mysql`；它在 `localhost`（使用默认端口），可以通过用户名 `root` 和密码 `test` 访问，我们的数据库名称为 `imagini`。

我们现在可以包含模块和设置，并创建连接：

```js
const settings = require("./settings");
const mysql    = require("mysql");
const db       = mysql.createConnection(settings.db);
```

此模块仅在您进行查询时连接到数据库。这意味着服务将启动，您直到进行第一次查询之前都不会知道您的连接设置是否正确。我们不希望在服务稍后使用时才发现无法连接到数据库，所以让我们强制连接并检查服务器是否正在运行并接受我们的连接：

```js
db.connect((err) => {
    if (err) throw err;

    console.log("db: ready");

    // ...
    // the rest of our service code
    // ...

    app.listen(3000, () => {
        console.log("app: ready");
    });
});
```

这样，如果数据库有任何问题，服务将无法启动并抛出异常，这会通知你检查哪里出了问题。以下是一个可能的错误示例：

```js
Error: ER_ACCESS_DENIED_ERROR: Access denied for user 'root'@'localhost' (using password: YES)
```

这表明你可能输入了错误的密码，或者用户不匹配，甚至主机名或数据库名可能错误。在设置服务之前确保您连接到数据库意味着您的服务在没有适当状态的情况下不会被公开。

说到我们的微服务，它的状态非常简单。我们的状态是之前上传的图片。我们不仅可以使用文件系统，现在还可以使用数据库并创建一个表来存储它们：

```js
db.query(
    `CREATE TABLE IF NOT EXISTS images
    (
        id INT(11) UNSIGNED NOT NULL AUTO_INCREMENT,
        date_created TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
        date_used TIMESTAMP NULL DEFAULT NULL,
        name VARCHAR(300) NOT NULL,
        size INT(11) UNSIGNED NOT NULL,
        data LONGBLOB NOT NULL,

        PRIMARY KEY (id),
        UNIQUE KEY name (name)
    )
    ENGINE=InnoDB DEFAULT CHARSET=utf8`
);
```

我们可以在服务每次启动时执行此查询，因为它只会在表不存在时创建图像表。如果我们不更改其结构，总是这样做是可以的。

你可以看到我们正在创建一个具有唯一识别号（`id`）、创建日期（`date_created`）、知道我们的图像何时被使用的日期（`date_used`）、图像的`name`、以字节为单位的图像`size`和图像`data`的表。大小在这里有点多余，因为我们可以直接检查数据长度，但请耐心，这只是一个例子。

我们也将我们的名称定义为一个唯一的键，这意味着它有一个索引，可以快速通过名称查找图像，同时也确保我们的名称不会重复，并且没有人可以覆盖一个图像（除非先删除它）。

将图像以这种方式存储在数据库表中，给你带来几个优势，例如：

+   你有多少张图像

+   每张图像的大小和总大小

+   图像的创建时间和最后使用时间

它还使你能够改进你的服务；例如，你可以删除长时间未使用的图像。你还可以根据图像大小来设置这个条件。稍后，你可以添加认证（强制或非强制）并设置用户特定的规则。

这也很容易备份并将状态复制到另一个站点。有许多用于备份数据库的工具，你可以拥有另一个充当从属服务器的 MySQL 服务器，并将你的图像实时复制到另一个地理位置。

让我们将我们的服务从上一章改为使用我们的表，而不是之前在文件系统上使用的文件夹。

我们可以移除我们的`fs`模块依赖（现在不要移除路径依赖）：

```js
app.param("image", (req, res, next, image) => {
    if (!image.match(/\.(png|jpg)$/i)) {
        return res.status(403).end();
    }

    db.query("SELECT * FROM images WHERE name = ?", [ image ], (err, 
    images) => {
        if (err || !images.length) {
            return res.status(404).end();
        }

        req.image = images[0];

        return next();
    });
});
```

我们的`app.param`完全不同。我们现在验证`image`是否与我们的`image`表匹配。如果没有找到，它返回代码`404`。如果找到了，它将`image`信息存储在`req.image`中。我们现在可以将我们的`image`上传改为将`image`存储在我们的表中：

```js
app.post("/uploads/:name", bodyparser.raw({
    limit : "10mb",
    type : "image/*"
}), (req, res) => {
    db.query("INSERT INTO images SET ?", {
        name : req.params.name,
        size : req.body.length,
        data : req.body,
    }, (err) => {
        if (err) {
            return res.send({ status : "error", code: err.code });
        }

        res.send({ status : "ok", size: req.body.length });
    });
});
```

上传图像不再使用文件系统，而是在我们的表中创建一个新的行。我们不需要指定`id`，因为它会自动生成。我们的创建日期也是自动的，默认为当前时间戳。我们的使用日期默认为`NULL`，这意味着我们还没有使用过`image`：

```js
app.head("/uploads/:image", (req, res) => {
    return res.status(200).end();
});
```

我们的`image`检查方法现在变得极其简单，因为它依赖于之前的`app.param`来检查图像是否存在，所以，如果我们到达这个点，我们已经知道图像存在（它在`req.image`上），所以我们只需要返回代码`200`。

在更新我们的图像`fetch`方法之前，让我们尝试我们的服务。如果你在控制台启动它，你可以立即打开任何 MySQL 管理工具并检查我们的数据库。我使用的是 Sequel Pro for macOS。尽管名字中有 Pro，但它是一款免费软件，而且非常好用：

![](img/65b81b98-1531-4dfd-9cc2-1987f3662fcc.png)

我们的表已创建，你可以检查它是否具有我们定义的所有属性和索引。现在让我们再次上传一个`image`：

![图片](img/51f3e97d-d8ed-4e1f-9d58-0fb7277a8c99.png)

和之前一样，它返回一个包含成功状态和`image`大小的 JSON 响应。如果你再次查看 Sequel，在内容分隔符上，你会看到我们的图片数据：

![图片](img/8abacb08-a0fd-4724-9b17-2280b0cd71ab.png)

让我们再次尝试上传`image`。之前，我们的服务会直接覆盖它。现在，由于我们的唯一索引，它应该拒绝具有相同名称的`INSERT`操作：

![图片](img/44849b11-4ec0-40e4-92cb-d6604f5515cc.png)

太好了！`ER_DUP_ENTRY`是 MySQL 的重复插入代码。我们可以依赖它并拒绝覆盖图片。

我们还可以使用`check`方法来检查我们的`image`是否存在：

![图片](img/b2f9f4a5-884f-444e-b8ae-2de6bcf310ba.png)

如果我们使用另一个名称，我们会得到一个代码`404`：

![图片](img/c33c9c5a-0af8-4b77-b65b-08cdd3acd9b6.png)

看起来一切都很顺利。现在让我们改变我们的最终方法，即`image`操作方法。这个方法几乎一样；我们只是不需要读取`image`文件，因为它已经可用：

```js
app.get("/uploads/:image", (req, res) => {
    let image     = sharp(req.image.data);
    let width     = +req.query.width;
    let height    = +req.query.height;
    let blur      = +req.query.blur;
    let sharpen   = +req.query.sharpen;
    let greyscale = [ "y", "yes", "true", "1", 
    "on"].includes(req.query.greyscale);
    let flip      = [ "y", "yes", "true", "1", 
    "on"].includes(req.query.flip);
    let flop      = [ "y", "yes", "true", "1", 
    "on"].includes(req.query.flop);

    if (width > 0 && height > 0) {
        image.ignoreAspectRatio();
    }

    if (width > 0 || height > 0) {
        image.resize(width || null, height || null);
    }

    if (flip)        image.flip();
    if (flop)        image.flop();
    if (blur > 0)    image.blur(blur);
    if (sharpen > 0) image.sharpen(sharpen);
    if (greyscale)   image.greyscale();

    db.query("UPDATE images " +
             "SET date_used = UTC_TIMESTAMP " +
             "WHERE id = ?", [ req.image.id ]);

    res.setHeader("Content-Type", "image/" + path.extname(req.image.name).substr(1));

    image.pipe(res);
});
```

你可以看到我们如何使用路径依赖来获取`image`名称的扩展名。其余的都是一样的。我们只是在每次请求此方法时更新我们的`image`。

我们可以使用网络浏览器来测试我们的方法并查看我们之前上传的图片：

![图片](img/88605934-5962-4af5-b5fb-821ef77a428d.png)

一切都应该和之前一样正常工作，因为我们没有更改我们的图像处理依赖项，所以模糊和其他操作应该按预期工作：

![图片](img/6cd78552-7907-4854-b1fc-27fd4eb936a4.png)

现在，我们可以改进我们的服务并添加一个我们之前没有公开的方法：删除`image`。为此，我们可以使用 HTTP 的`DELETE`动词并从我们的表中删除`image`：

```js
app.delete("/uploads/:image", (req, res) => {
    db.query("DELETE FROM images WHERE id = ?", [ req.image.id ], (err) 
    => {
        return res.status(err ? 500 : 200).end();
    });
});
```

我们只需要检查查询是否导致错误。如果是这样，我们以代码`500`（内部服务器错误）响应。如果不是，我们以通常的代码`200`响应。

让我们重新启动我们的微服务并尝试删除我们的`image`：

![图片](img/f332e424-4eb1-4153-9c9e-0807469373fe.png)

看起来它工作了；它以代码`200`响应。如果我们尝试在网页浏览器中打开我们的图片，我们应该看到类似这样的内容：

![图片](img/eced36f5-8a89-4228-ad2f-779c7b69b60a.png)

在 Sequel 中，表现在也应该为空：

![图片](img/480a78b6-72cc-4286-bb0e-719f3fea3f71.png)

现在，我们有一个具有跨重启持久状态的实用微服务，正如我们预期的。你现在可以部署到任何云服务，无需依赖文件系统，只需数据库即可。

你可以轻松地将 MySQL 更改为另一个数据库，或者使用**对象关系映射**（**ORM**）模块来使你能够在不更改代码的情况下更改数据库服务器。ORM 是一个库，它允许你使用一个通用接口来访问不同类型的数据库。通常，这种抽象涉及根本不使用 SQL，并将你与数据库的交互简化为更简单的查询（以允许数据库服务器之间的互操作性）。

让我们抓住这个机会，更进一步，添加一些由于这次迁移而被简化的数据库中的方法。让我们创建一个公开数据库统计信息的方法，并删除旧图像。

我们的第一种统计方法应该只返回一个包含一些有用信息的 JSON 结构。让我们公开以下内容：

+   图像的总数

+   图像的总大小

+   我们的服务运行了多久

+   上次我们上传图像的时间

这里是一个我们的统计方法可能看起来像的例子：

```js
app.get("/stats", (req, res) => {
    db.query("SELECT COUNT(*) total" +
             ", SUM(size) size " +
             ", MAX(date_created) last_created " +
             "FROM images",
    (err, rows) => {
        if (err) {
            return res.status(500).end();
        }

        rows[0].uptime = process.uptime();

        return res.send(rows[0]);
    });
});
```

重新启动服务，让我们试试：

![图片](img/277cd1a8-085e-410c-b712-0e6d1f58d1f2.png)

如我们所见，我们没有图像，因为我们刚刚删除了我们的图像。没有大小，因为我们没有图像。也没有使用日期，服务运行时间为 5 秒。

如果我们上传之前的图像，我们将得到不同的结果，就像以下截图所示：

![图片](img/ac4b4b76-7cad-4ba4-b35e-40b11c7ca310.png)

现在，对于我们的第二个任务，删除旧图像，我们需要定期检查我们的数据库。我们将使用间隔计时器并运行一个`DELETE`查询。以下查询中提到的间隔只是一个例子；你可以编写你想要的条件。

```js
setInterval(() => {
    db.query("DELETE FROM images " +
             "WHERE (date_created < UTC_TIMETSTAMP - INTERVAL 1 WEEK 
             AND date_used IS NULL) " +
             " OR (date_used < UTC_TIMETSTAMP - INTERVAL 1 MONTH)");
}, 3600 * 1000);
```

这个查询删除了在过去一个月内未使用（但之前使用过）的`images`或在过去一周内未使用（且从未使用过）的图像。这意味着上传的图像至少需要使用一次，否则它们将很快被删除。

你可以想一个不同的策略，或者如果你愿意，不使用策略并手动删除。现在我们已经看到了 MySQL，让我们继续看看另一种类型的数据库服务器。

# 添加代码覆盖率

现在我们已经有一个正在工作的测试套件和一个测试，让我们引入代码覆盖率。从开发初期开始添加这个功能非常容易，并将帮助我们关注需要测试的代码部分，特别是涉及特定条件的一些用例（例如我们代码中的`if-then-else`语句）。从开发初期开始设置它很容易。另一方面，如果你有一个完全工作的代码，并想添加测试和覆盖率，这将更困难，并且需要相当长的时间。

为了增加代码覆盖率，我们将引入另一个模块。我们将全局安装它，以便能够直接使用它运行测试：

```js
npm install -g nyc
```

现在，我们可以使用以下仪器运行我们的测试：

```js
nyc npm test
```

这应该会运行我们的测试，同时安装了仪器。最后，你将得到一个漂亮的控制台报告。

![图片](img/99619f27-c68e-49d6-8b6a-a3ad0178c91a.png)

覆盖率结果存储在`.nyc_output`文件夹中。这使得你可以在不再次运行测试的情况下查看最后的测试结果。如果你的测试套件很大且需要一些时间才能完成，这很有用。

要查看结果，只需运行`nyc report`：

![图片](img/bd9cb784-7cdb-4a9f-8136-5c8c2e133c2a.png)

结果是一个控制台报告。还有几种其他类型的报告。其中一个特别有用的是`html`报告。让我们生成它：

```js
nyc report --reporter=html
```

你现在应该有一个包含`index.html`文件的`coverage`文件夹。在浏览器中打开它，你应该会看到以下截图：

![图片](img/90266f17-41c9-456b-ae43-f238f88d56bb.png)

我们只有一个文件代表我们的微服务。如果有更多，它们将按层次列出。每个文件都有全局平均统计数据。

有三组重要的列：

+   **语句**：代表代码语句（条件、赋值、断言、调用等）

+   **分支**：代表可能的代码控制工作流程，例如 if-then-else 或 switch-case 语句的可能性

+   **函数**：代表我们的实际代码函数和回调

你可以在我们的文件上点击，查看其具体细节，更具体地说，可以逐行查看代码和信息：

![图片](img/9331f44e-72e3-4ca6-ab78-8be70c2f63e8.png)

在每一行数字的右侧，你看到一个灰色区域，在这种情况下，你会在一些行中看到`2x`。这是该行的执行次数。该行被执行了两次。这实际上并不重要，除非你在寻找大量执行的代码片段并想进行某种优化。

你还可以看到第 12 行有两个变化。首先，在`throw err`后面有一个粉红色的背景。这意味着该语句从未被执行，这在目前是正常的，因为我们总是成功连接到数据库。`if`语句前的标记表示该条件从未被执行：

![图片](img/0945b756-7b35-43c7-ae99-dd753f2e554f.png)

如果你向下滚动几行，我们会看到更多带有标记的行。例如，我们可以看到我们的图片上传方法几乎被完全覆盖。唯一缺失的是错误处理。

由于我们在运行测试之前删除了测试图像，我们的图像删除方法也得到了覆盖。再次强调，唯一缺失的分支是如果数据库返回错误到我们的`DELETE`查询。

在进一步处理图片上传之前，让我们添加另一个名为`image-parameter.js`的`integration`测试文件，并添加一些测试来提高我们的覆盖率：

```js
const chai = require("chai");
const http = require("chai-http");
const tools = require("../tools");

chai.use(http);

describe("The image parameter", () => {
    beforeEach((done) => {
        chai
        .request(tools.service)
        .delete("/uploads/test_image_parameter.png")
        .end(() => {
            return done();
        });
    });

    it("should reply 403 for non image extension", (done) => {
        chai
        .request(tools.service)
        .get("/uploads/test_image_parameter.txt")
        .end((err, res) => {
            chai.expect(res).to.have.status(403);

            return done();
        });
    });

    it("should reply 404 for non image existence", (done) => {
        chai
        .request(tools.service)
        .get("/uploads/test_image_parameter.png")
        .end((err, res) => {
            chai.expect(res).to.have.status(404);

            return done();
        });
    });
});
```

让我们运行我们的测试套件并看看结果如何：

![图片](img/44cfd9bd-5ed2-43df-9b75-925c62c0c5da.png)

刷新 HTML 报告页面，查看我们的参数方法：

![图片](img/feb5c546-faf8-48dd-9033-b7d921f98985.png)

如你所见，我们现在覆盖了以下条件：

```js
if (!image.match(/\.(png|jpg)$/i)) {
```

以下条件：

```js
if (err || !images.length) {
```

我们现在对这个方法有了全面的覆盖。

有其他一些更难测试的覆盖行，例如计时器（你可以在*第 28 行*看到一个），`catch`语句，或者来自数据库或其他存储源的外部错误。有方法来模拟这些事件，我们稍后会讨论。

# RethinkDB

让我们看看使用 RethinkDB 的非关系型数据库的差异。如果你没有它，只需按照官方文档（[`www.rethinkdb.com/docs/`](https://www.rethinkdb.com/docs/)）进行安装。让我们先启动服务器：

```js
rethinkdb
```

这将启动服务器，它附带一个非常好的管理控制台，端口为`8080`。你可以在网页浏览器中打开它：

![图片](img/8a25af14-fa51-4597-9eff-d47bcde127d9.png)

前往顶部的“表”部分查看数据库：

![图片](img/b2a5cad5-13fc-40b5-9aa5-c36694406605.png)

使用“添加数据库”按钮创建一个名为`imagini`的数据库。现在你应该已经有了我们的数据库准备就绪。这里你不需要其他任何东西：

![图片](img/5be39ecd-34f7-42c4-8266-fbad8ad61e0b.png)

要使用我们新的数据库，我们需要安装`rethinkdb`依赖项。你可以移除 MySQL 依赖项：

```js
npm uninstall mysql --save
npm install rethinkdb -–save
```

现在，让我们改变我们的`settings`文件。这个模块不接受连接字符串，所以我们将使用 JSON 结构：

```js
{
    "db": {
        "host" : "localhost",
        "db" : "imagini"
    }
}
```

要包含我们的依赖项，我们只需要包含这个模块：

```js
const rethinkdb = require("rethinkdb");
```

然后，使用这个来连接到我们的服务器：

```js
rethinkdb.connect(settings.db, (err, db) => {
    if (err) throw err;

    console.log("db: ready");

    // ...
    // the rest of our service code
    // ...

    app.listen(3000, () => {
        console.log("app: ready");
    });
});
```

连接后，我们可以像之前一样创建我们的表。这次，我们不需要指定任何结构：

```js
rethinkdb.tableCreate("images").run(db);
```

`rethinkdb`对象是我们用来操作我们的表的，而`db`对象是一个连接对象，用于引用连接并指示在哪里运行我们的操作。

如果你像这样重启我们的服务，你会在之前创建的数据库中看到一个新表：

![图片](img/976d103d-a00f-4bc1-b830-ada32e107310.png)

如果你再次重启我们的服务，尝试创建已经存在的表时会出错。我们需要检查它是否已经存在，并且只有在不存在的情况下才发出命令：

```js
rethinkdb.tableList().run(db, (err, tables) => {
    if (err) throw err;

    if (!tables.includes("images")) {
        rethinkdb.tableCreate("images").run(db);
    }
});
```

接下来，我们的上传方法应该稍微改变一下，如下所示：

```js
app.post("/uploads/:name", bodyparser.raw({
    limit : "10mb",
    type : "image/*"
}), (req, res) => {
    rethinkdb.table("images").insert({
        name : req.params.name,
        size : req.body.length,
        data : req.body,
    }).run(db, (err) => {
        if (err) {
            return res.send({ status : "error", code: err.code });
        }

        res.send({ status : "ok", size: req.body.length });
    });
});
```

如果你像这样重启服务器，你应该能够上传一个`image`：

![图片](img/5591f6a1-af2d-414b-9d22-9b5defb7c872.png)

我们收到相同的响应，就像 MySQL 一样。我们可以进入管理控制台中的数据探索器部分，获取我们的记录以查看它是否在那里：

![图片](img/b05c2929-88d4-46bb-aa48-61055e014f06.png)

看起来不错。注意我们的记录 ID 不是一个数字，它是一个**通用唯一标识符**（**UUID**）。这是因为 RethinkDB 支持分片（如果有多于一个服务器，我们的表默认是分片的）并且相对于递增数字，分片唯一标识符更容易。

接下来是我们的 Express 参数：

```js
app.param("image", (req, res, next, image) => {
    if (!image.match(/\.(png|jpg)$/i)) {
        return res.status(403).end();
    }

    rethinkdb.table("images").filter({
        name : image
    }).limit(1).run(db, (err, images) => {
        if (err) return res.status(404).end();

        images.toArray((err, images) => {
            if (err) return res.status(500).end();
            if (!images.length) return res.status(404).end();

            req.image = images[0];

            return next();
        });
    });
});
```

通过这个改变，我们现在可以重启我们的服务并查看我们的`image`是否存在：

![图片](img/cf087587-648d-4091-aa9f-e0d9a8a15fd7.png)

我们需要稍微改变一下下载方式。我们需要删除之前的查询以更新我们的使用日期，并用一个新的查询替换它：

```js
app.get("/uploads/:image", (req, res) => {
    let image     = sharp(req.image.data);
    let width     = +req.query.width;
    let height    = +req.query.height;
    let blur      = +req.query.blur;
    let sharpen   = +req.query.sharpen;
    let greyscale = [ "y", "yes", "true", "1", 
    "on"].includes(req.query.greyscale);
    let flip      = [ "y", "yes", "true", "1", 
    "on"].includes(req.query.flip);
    let flop      = [ "y", "yes", "true", "1", 
    "on"].includes(req.query.flop);

    if (width > 0 && height > 0) {
        image.ignoreAspectRatio();
    }

    if (width > 0 || height > 0) {
        image.resize(width || null, height || null);
    }

    if (flip)        image.flip();
    if (flop)        image.flop();
    if (blur > 0)    image.blur(blur);
    if (sharpen > 0) image.sharpen(sharpen);
    if (greyscale)   image.greyscale();

    rethinkdb.table("images").get(req.image.id).update({ date_used : 
    Date.now() }).run(db);

    res.setHeader("Content-Type", "image/" + 
    path.extname(req.image.name).substr(1));

    image.pipe(res);
});
```

我们现在可以使用网络浏览器下载我们的图片：

![图片](img/454e1973-472e-414f-af6e-2b0abbb621ea.png)

接下来，我们需要更新我们的图片删除方法。它就像我们的上传一样简单：

```js
app.delete("/uploads/:image", (req, res) => {
    rethinkdb.table("images").get(req.image.id).delete().run(db, (err) 
    => {
        return res.status(err ? 500 : 200).end();
    });
});
```

这次，我们使用了图片的唯一 ID 来删除它。如果我们再次使用 `curl` 命令尝试，我们会收到一个代码 `200`：

![图片](img/e8c95a69-2e76-4e9d-b83f-7990119543e7.png)

如果我们尝试获取我们表的第一条记录，我们会看到那里什么也没有：

![图片](img/600662dc-1ded-4417-a286-899f773f6fc1.png)

最后，这是我们引入 MySQL 后添加的两个额外功能：统计和删除旧的不用图片。

我们的统计方法并不像运行带有聚合的 SQL 查询那样简单。我们必须计算我们每个统计量：

```js
app.get("/stats", (req, res) => {
    let uptime = process.uptime();

    rethinkdb.table("images").count().run(db, (err, total) => {
        if (err) return res.status(500).end();

        rethinkdb.table("images").sum("size").run(db, (err, size) => {
            if (err) return res.status(500).end();

            rethinkdb.table("images").max("date_created").run(db, (err, 
            last_created) => {
                if (err) return res.status(500).end();

                last_created = (last_created ? new 
                Date(last_created.date_created) : null);

                return res.send({ total, size, last_created, uptime });
            });
        });
    });
});
```

我们应该得到与之前相似的结果：

![图片](img/81ddd5ab-115d-42c0-867b-95a7319a2555.png)

删除旧图片相对容易；我们只需要过滤掉我们想要删除的图片，然后删除它们：

```js
setInterval(() => {
    let expiration = Date.now() - (30 * 86400 * 1000);

    rethinkdb.table("images").filter((image) => {
        return image("date_used").lt(expiration);
    }).delete().run(db);
}, 3600 * 1000);
```

我简化了之前的策略，只是删除了 1 个月（30 天，每天 86,400 秒，每天 1,000 毫秒）之前的 `images`。

# Redis

内存数据库与前面两种类型不同，因为它们通常不是结构化的，这意味着你没有任何表。你通常有一些可以查找和操作的类型列表，或者简单的哈希表。

利用我们之前为 Hydra 安装的 Redis 实例，让我们看看这种数据库的另一个缺点，或者实际上是特性。让我们连接到我们的 Redis 实例并执行以下指令序列：

![图片](img/2b204f94-7078-4020-9b1d-bf1f62f402cc.png)

我们在这里所做的是：

1.  使用 `redis-cli` 连接到 Redis 服务。

1.  获取计数器的内容，它是 nil（无），因为我们还没有定义它。

1.  增加计数器，现在是自动定义并设置为 `1`。

1.  再次增加计数器，现在是 `2`。

1.  获取计数器的内容，当然是 `2`。

1.  关闭 Redis 服务。

1.  启动 Redis 服务。

1.  再次连接到 Redis 服务。

1.  获取计数器的内容，它是 nil（无）。

我们的计数器在哪里？嗯，这是一个内存数据库，所以当我们关闭 Redis 服务时，一切都会消失。这是几乎所有内存数据库的设计。

它们被设计成快速和内存中的。它们的目的通常是缓存那些获取成本高昂的数据，比如一些复杂的计算，或者下载量大的数据，我们希望它们能够更快地可用（内存中）。

我对 Redis 并不完全公平，因为它实际上允许你的数据在服务重启之间被保存。所以，让我们看看我们可以用它来保存我们的微服务状态有多远。

如前所述，让我们卸载 `rethinkdb` 并安装 `redis` 模块：

```js
npm uninstall rethinkdb --save
npm install redis --save
```

让我们忽略我们的`settings.json`文件（如果你喜欢，你可以将其删除）并假设 Redis 将运行在我们的本地机器上。

首先，我们需要包含`redis`模块并创建一个`Client`实例：

```js
const redis = require("redis");
const db    = redis.createClient(); 
```

然后，我们需要等待它连接：

```js
db.on("connect", () => {
    console.log("db: ready");

    // ...
    // the rest of our service code
    // ...

    app.listen(3000, () => {
        console.log("app: ready");
    });
});
```

我们可以使用 Redis 存储我们的数据的方式有很多种。为了简单起见，因为我们没有表，让我们使用散列来存储我们的图像。每个图像将有一个不同的散列，散列的名称将是图像的名称。

由于这种类型的数据库中没有表，我们的初始化代码可以简单地删除。

接下来，让我们更改我们的上传方法，以便在 Redis 上存储数据。正如我提到的，让我们将其存储在以`image`命名的散列中：

```js
app.post("/uploads/:name", bodyparser.raw({
    limit : "10mb",
    type : "image/*"
}), (req, res) => {
    db.hmset(req.params.name, {
        size : req.body.length,
        data : req.body.toString("base64"),
    }, (err) => {
        if (err) {
            return res.send({ status : "error", code: err.code });
        }

        res.send({ status : "ok", size: req.body.length });
    });
});
```

`hmset`命令允许我们设置散列的多个字段，在我们的例子中，是`size`和`data`。注意我们以`base64`编码存储我们的图像内容，否则我们会丢失数据。如果我们重启我们的服务并尝试上传我们的测试`image`，它应该可以正常工作：

![](img/69d28e98-498a-4a5f-8dca-9532bb9027e6.png)

然后，我们可以使用`redis-cli`查看我们的图像是否在那里。嗯，我们正在检查散列是否具有字段大小并且与我们的图像大小匹配：

![](img/3112df7b-958f-4b55-97c6-d3623d694963.png)

太好了！现在我们可以更改我们的 Express 参数，以查找`image`散列：

```js
app.param("image", (req, res, next, name) => {
    if (!name.match(/\.(png|jpg)$/i)) {
        return res.status(403).end();
    }

    db.hgetall(name, (err, image) => {
        if (err || !image) return res.status(404).end();

        req.image      = image;
        req.image.name = name;

        return next();
    });
});
```

我们的`image`检查方法现在应该可以工作。而且，为了我们的下载方法能够工作，我们只需要将图像加载更改为解码我们之前的`base64`编码：

```js
app.get("/uploads/:image", (req, res) => {
    let image     = sharp(Buffer.from(req.image.data, "base64"));
    let width     = +req.query.width;
    let height    = +req.query.height;
    let blur      = +req.query.blur;
    let sharpen   = +req.query.sharpen;
    let greyscale = [ "y", "yes", "true", "1", "on"].includes(req.query.greyscale);
    let flip      = [ "y", "yes", "true", "1", "on"].includes(req.query.flip);
    let flop      = [ "y", "yes", "true", "1", "on"].includes(req.query.flop);

    if (width > 0 && height > 0) {
        image.ignoreAspectRatio();
    }

    if (width > 0 || height > 0) {
        image.resize(width || null, height || null);
    }

    if (flip)        image.flip();
    if (flop)        image.flop();
    if (blur > 0)    image.blur(blur);
    if (sharpen > 0) image.sharpen(sharpen);
    if (greyscale)   image.greyscale();

    db.hset(req.image.name, "date_used", Date.now());

    res.setHeader("Content-Type", "image/" + path.extname(req.image.name).substr(1));

    image.pipe(res);
});
```

我们现在正从 Redis 中提供图像。作为额外的好处，我们在`image`散列中添加/更新了一个`date_used`字段，以指示它最后一次被使用的时间：

![](img/c617c24a-e4df-4878-a442-2ce3ff909f1b.png)

删除我们的`image`就像删除我们的散列一样简单：

```js
app.delete("/uploads/:image", (req, res) => {
    db.del(req.image.name, (err) => {
        return res.status(err ? 500 : 200).end();
    });
});
```

然后，我们可以尝试删除我们的`test`图像：

![](img/3557032f-1cf1-42f1-9943-00c31ef5fd66.png)

使用`redis-cli`检查散列是否存在，我们看到它已经消失了：

![](img/716dfce9-3eaa-405a-ac5f-91f5bdd7424a.png)

缺少的只有两个功能：统计信息和删除旧图像。

对于统计信息，这可能很困难，因为我们使用的是通用的散列表，我们无法确定定义了多少个散列表，以及是否所有或任何散列表都有图像数据。我们可能需要扫描所有散列表，这对于大型集合来说很复杂。

要删除旧图像，问题与查找具有特定条件（如字段值）的散列表的方式相同。

仍然有其他路径可以解决这个问题。例如，我们可以有一个包含我们图像名称的另一个散列表，并使用日期。但是，复杂性会增加，而且由于我们通过不同的散列表分割信息而没有确保**原子性**、**一致性**、**隔离性**和**持久性**（**ACID**）操作，完整性可能会受到风险。

# 结论

正如我们所见，有许多选项可以存储我们的微服务状态。根据我们操作的信息类型，有一些数据库更适合处理我们的数据。

这一切都取决于我们应该问自己的一些不同的问题：

+   我们的数据完整性重要吗？

+   我们的数据结构复杂吗？

+   我们需要获取哪些类型的信息？

如果我们的数据完整性很重要或数据结构复杂，不要使用内存数据库。根据复杂度，看看你是否需要一个非关系型数据库，或者是否可以使用能够处理更复杂操作和数据聚合的关系型数据库，这将有助于你实现最后一个要点。

# 安全

一个好的做法是迭代性地编写代码，每次我们添加一个新的小功能或改进时都进行测试，并且始终编写考虑到我们为服务设想的所有功能的代码。

考虑服务的路线图可以帮助你为未来的改进做好准备，减少以后浪费或替换的代码量。

例如，在安全方面：

+   我们的服务是否安全？它是否为某些类型的恶意攻击做好了准备？

+   我们的服务是否私密？它是否应该有一些认证或授权机制？

幸运的是，我们的框架允许我们的代码进行组合，并允许我们后来添加安全层。例如，使用 Express 或 Hydra，我们可以添加一个先导路由功能，该功能将在我们的服务方法之前运行，使我们能够强制执行，例如，认证。

查看我们的服务，因为它使用 HTTP 公开其方法，我们可以对其进行一些改进，例如：

+   **认证**：强制任何使用它的人进行身份验证。或者，仅对上传和删除方法进行强制。这取决于你。也可能有用户账户，每个用户都会看到他们各自的图片列表。

+   **授权**：限制例如哪些网络可以访问服务，无论是否有有效的认证。

+   **机密性**：保护用户免受网络流量中窥探者的侵害。

+   **可用性**：限制每个客户端对服务的最大使用频率，以确保单个客户端不会阻塞你的整个服务。

为了引入这些改进，你可以添加一个认证模块，例如 Passport 模块，并使用证书为用户提供更安全的 HTTPS 体验。

其他类型的不安全性直接来自你的代码，并且通过添加证书或强制认证并不能得到改善。我指的是：

+   缺陷、编程逻辑错误以及未正确测试的使用案例，可能导致小或严重的问题。

+   依赖性错误，你可能没有意识到，但仍然可能破坏你的服务，并可能迫使你寻找替代依赖项，这从来都不是一项愉快的任务。

为了最小化这些事件，你应该始终不断进化你的测试套件，随着用例的出现添加它们，确保解决的新问题不会在以后再次出现。关于依赖性问题，你可以订阅 Node Security Project，甚至将其集成到你的代码中，以便始终知道你的依赖项中是否有任何一个是风险的。

如果有源代码戒律，接下来的四条肯定会在列表上：

+   保持代码简单。如果代码变得复杂，就停下来，回顾一下，并将代码拆分成更简单的部分。

+   验证外部输入，无论是用户还是另一个服务。永远不要相信来自外部的数据。

+   默认拒绝而不是相反，检查某人是否有权访问资源，并拒绝任何无权访问的人。

+   从项目开始时就添加测试用例。

# 摘要

状态是任何服务的一部分，状态建立在数据之上。为了获得更云原生（cloud-native）的体验，服务不能依赖于传统的文件系统，而需要使用其他类型的存储结构来存储数据。数据库是一个自然的发展，根据我们的数据的重要性和复杂性，我们可以选择一些数据库类型。

假设我们的状态已安全存储在某些类型的数据库服务中，确保我们的数据不能通过我们的服务被破坏也同样重要。我们的服务中可能存在安全漏洞和错误，可能会使我们的数据处于风险之中，因此，在规划服务路线图时，编写简单的代码、验证输入和考虑安全性是很重要的。

为了推进我们的服务，让我们引入我们尚未做但应该做的事情，那就是一个合适的测试套件。在下一章中，我们将看到一些好的选择，并创建一个测试套件，确定是否需要改变以使我们的服务尽可能安全。
