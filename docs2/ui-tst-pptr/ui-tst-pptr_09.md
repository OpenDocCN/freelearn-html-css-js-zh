# *第九章*：抓取工具

在 *第一章* *使用 Puppeteer 入门* 中，我们讨论了网络自动化的不同用途。在所有这些用例中，网络抓取是让开发者最兴奋的一个。当我谈论自动化时，我知道当我开始谈论任务自动化时，我会得到观众的全部注意力，但当我深入到 **网络抓取** 这个话题时，我会得到更多的关注。

请不要误解我，我认为 UI 测试很重要。正如我们在前面的章节中看到的，这不仅仅是运行自动化测试，也是照顾客户。但网络抓取有那种有趣的火花，一种黑客的感觉，我不想在这个话题上遗漏任何内容。

几个月前，我读了一本关于网络抓取的书，书中在书的结尾处包含了一个关于 UI 测试的章节。我们将做同样的事情，但方向相反。这是一本以抓取章节结尾的 UI 测试书。

我们将从这个章节的开始，通过定义和揭秘网络抓取。它只是针对黑客的吗？它甚至合法吗？我们还将讨论抓取伦理，比如什么时候可以抓取，什么时候不可以。

章节的第二部分将讨论使用 Puppeteer 可用的不同抓取工具。

本章我们将涵盖以下主题：

+   网络抓取简介

+   创建抓取器

+   并行运行抓取器

+   如何避免被检测为机器人

+   处理身份验证和授权

到本章结束时，你将能够将你在本书的学习过程中学到的所有概念应用于一个全新的网络自动化领域。

让我们开始吧。

# 技术要求

你可以在 GitHub 仓库（[`github.com/PacktPublishing/UI-Testing-with-Puppeteer`](https://github.com/PacktPublishing/UI-Testing-with-Puppeteer)）的 `Chapter9` 目录下找到本章的所有代码。请记住在那个目录下运行 `npm install`。

# 网络抓取简介

介绍一个新概念的最佳方式是通过给出一些具体和直接的定义。让我们首先定义 **数据抓取**。根据维基百科 ([`www.hardkoded.com/ui-testing-with-puppeteer/data-scraping`](https://www.hardkoded.com/ui-testing-with-puppeteer/data-scraping))，“*数据抓取是一种计算机程序从另一个程序的人类可读输出中提取数据的技术*。”任何从计算机输出的信息都可以被提取和处理。最早的抓取器被称为“屏幕抓取器”。屏幕抓取器就像一个可以捕获屏幕的应用程序。然后，通过运行 **光学字符识别**（**OCR**），它从该图像中提取文本以进行进一步处理。

网络抓取将这个想法提升到了新的水平。*网络抓取是一种使用软件从一个或多个网站中提取数据的技巧*。

你可能会想：这甚至合法吗？亚马逊是一个公开的网站。我可以自由地浏览网站；为什么我不能运行一个脚本来提取已经公开的数据呢？好吧，这取决于。让我与你分享一些现实世界中的场景，这些场景与网络爬取有类似的伦理困境。

**第一种情况**

一个小型杂货店老板去一个大商场比较他们的产品和价格。她不能拿一箱牛奶走，不付钱，但她可以四处走动，记下产品价格，然后把这份清单带到她的店里，比较价格。价格不是产品。她没有偷任何东西。而且，商场太大，他们无法控制每个四处走动记笔记的人。但如果同一个人去下一个街区的小杂货店开始记笔记呢？我敢打赌店主已经认识她了，而且她记笔记的行为太明显了，这可能会威胁到他们的生意。这是非法的吗？不是。但她可能会陷入纠纷。

**第二种情况**

一些高端家具店不允许你在店内拍照。

**最后一种情况**

你在赌场不能数牌！他们会把你踢出去，并且终身禁止你进入。

这些是现实生活中的“爬虫”。人们在现实世界中试图提取信息。爬取网络与此类似。只要满足以下条件，你就能爬取一个网站：a) 网站欢迎（隐式或显式）爬虫，b) 在爬取时你的态度是考虑周到的，以及 c) 你所爬取的内容是被允许的。让我们来分析这些概念。

## 网站是否允许爬虫？

对于这个问题，第一个反应可能是：“不！网站所有者为什么要允许爬虫？”但这并不一定正确。如果你拥有一家酒店呢？如果一个聚合网站爬取你的预订页面，然后在他们的网站上显示这些结果，并附上链接回到你的网站，他们将从这中获得一些利润，而你将吸引更多客户：双赢。或者，如果你拥有一家如维基百科或政府网站这样的非营利网站，爬取可能不是问题。作为一个非营利网站，你不应该太在意机器人来到你的网站提取数据，除非它们影响了你网站的性能。但如果你网站的内容是关于歌词，你不会希望任何人来到你的网站提取歌词。歌词是你的产品，你的资产。

在本章中，我们将看到许多绕过一些验证的技术，但我的个人规则是：如果网站不想被爬取，我就不会爬取它。不意味着不。

那么，我们如何知道我们是否可以爬取一个网站呢？网站的所有者可以通过至少四种不同的方式来表达这一点。

### 条款和条件

在爬取一个网站之前，你应该首先检查其条款和条件。网站所有者可以非常明确地表示他们不希望被爬取。

使用条款和条件是我们经常在安装应用程序时忽略的那块大文本。我敢打赌你也收到过邮件告诉你一个网站已经更改了其使用条款和条件，而你只是说“随便吧”然后存档了那封邮件。

但我们不应该这样做。根据 iubenda ([`www.hardkoded.com/ui-testing-with-puppeteer/iubenda-terms`](https://www.hardkoded.com/ui-testing-with-puppeteer/iubenda-terms))，"'*使用条款和条件'是服务提供者与其用户之间合同关系的规范文件。使用条款和条件实际上就是一份合同，其中所有者明确了其服务的使用条件*。"有时我们可能会认为，当我们在一个网站上购买数字内容（软件、音乐、电子书）时，我们就拥有了那个产品，而实际上，如果你阅读了使用条款和条件，你只是购买了使用该产品的权利，但并不拥有它。

使用条款和条件还说明了你可以在网站上做什么。许多网站都对此非常明确。以 www.ebay.com 的使用条款和条件为例：

*3. 使用 eBay*

*在使用或访问服务的过程中，你将不会：*

*使用任何机器人、蜘蛛、抓取器或其他自动化手段访问我们的服务以任何目的；*

*绕过我们的机器人排除头信息，干扰我们服务的运行，或对我们的基础设施施加不合理或不成比例的大负载。*

正如我们所见，eBay 表述得非常明确。你不能抓取他们的网站。eBay 诉 Bidder's Edge ([`www.hardkoded.com/ui-testing-with-puppeteer/ebay-vs-edge`](https://www.hardkoded.com/ui-testing-with-puppeteer/ebay-vs-edge)) 是 2000 年代的一个著名案例。eBay 指控 Bidder's Edge 的抓取活动构成了对 eBay 财产的侵犯 ([`www.hardkoded.com/ui-testing-with-puppeteer/Trespass-to-chattels`](https://www.hardkoded.com/ui-testing-with-puppeteer/Trespass-to-chattels))。换句话说，Bidder's Edge 的抓取影响了 eBay 的服务器。我敢打赌你不想与 eBay 对簿公堂。

现在，让我们来看看 Ryanair 的使用条款和条件 ([`www.hardkoded.com/ui-testing-with-puppeteer/ryanair-terms`](https://www.hardkoded.com/ui-testing-with-puppeteer/ryanair-terms))：

*使用任何自动化系统或软件（无论是由第三方操作还是其他方式），从本网站提取任何数据用于商业目的（“屏幕抓取”）是严格禁止的。*

Ryanair 也不喜欢抓取器，但它说“出于商业目的”，这意味着你可以编写你的抓取器代码来寻找你下一次度假的最佳价格。

如果使用条款和条件没有明确说明抓取器，网站所有者可以通过 `robots.txt` 文件表达他们与抓取器的关系的另一种方式。

### robots.txt 文件

维基百科再次给出了对 robots 文件的精彩定义。根据维基百科([`www.hardkoded.com/ui-testing-with-puppeteer/Robots-exclusion-standard`](https://www.hardkoded.com/ui-testing-with-puppeteer/Robots-exclusion-standard))，robots 排除协议"*是网站用来与网络爬虫和其他网络机器人通信的标准。该标准指定了如何通知网络爬虫哪些网站区域不应被处理或扫描。网络爬虫通常被搜索引擎用来对网站进行分类*。”

那个定义中的关键词是“通知”。网站所有者可以在 robots 文件中表达哪些网站部分可以被抓取。大多数网站只使用`robots.txt`文件来告诉搜索引擎它们可以在哪里找到用于抓取的 Sitemap：

```js
User-agent: *
Sitemap: https://www.yoursite.com/sitemap.xml
```

这两行简单的代码告诉搜索引擎，如谷歌，获取那个 Sitemap 文件并抓取那些页面。但你可以在这个文件中找到更复杂的定义。例如，维基百科上的`robots.txt`文件有超过 700 行！这告诉我们该网站被大量抓取。让我们看看这个文件中我们可以找到的一些示例：

```js
# Please note: There are a lot of pages on this site, and there are
# some misbehaved spiders out there that go _way_ too fast. If you're
# irresponsible, your access to the site may be blocked.
```

我喜欢这个文件以一条信息开始！他们希望我们来到这个页面阅读它。下一部分很有趣：

```js
# Crawlers that are kind enough to obey, but which we'd rather not have
# unless they're feeding search engines.
User-agent: UbiCrawler
Disallow: /
User-agent: DOC
Disallow: /
User-agent: Zao
Disallow: /
# Some bots are known to be trouble, particularly those designed to copy
# entire sites. Please obey robots.txt.
User-agent: sitecheck.internetseer.com
Disallow: /
```

在这里，维基百科通知说，他们不希望与用户代理**UbiCrawler**、**DOC**、**Zao**和**sitecheck.internetseer.com**相关的抓取器抓取网站。并且文件以适用于所有用户代理的通用规则结束：

```js
User-agent: *
Allow: /w/api.php?action=mobileview&
Allow: /w/load.php?
Allow: /api/rest_v1/?doc
Disallow: /w/
Disallow: /api/
Disallow: /trap/
Disallow: /wiki/Special:
…
```

他们基本上说，除了某些 URL（如`/w/`、`/api/`等）之外，所有剩余的（`User-agent: *`）都可以抓取整个网站。

如果我们在条款和条件或`robots.txt`文件中找不到任何有用的信息，我们可能在页面响应中找到一些线索。

### 你是人类吗？

我打赌现实生活中没有人问过你你是不是人类，但许多网站总是问我们这个问题：

![reCAPTCHA 机器人检测](img/Figure_9.01_B16113.jpg)

reCAPTCHA 机器人检测

最初只是一个简单的“输入你看到的单词”的挑战，变成了越来越复杂的挑战：

![复杂的 CAPTCHA](img/Figure_9.02_B16113.jpg)

复杂的 CAPTCHA

我们应该检查第一行的第三个方块吗？谁知道……但他们的目标是明确的。以一种友好的方式，使用 UI，他们想将抓取器踢出去。

当一个网站放置一个像之前截图中的 CAPTCHA 时，这并不总是因为他们不想被抓取。也许他们想保护他们的用户。他们不希望恶意机器人测试用户名和密码或被盗的信用卡号码。但意图是明确的：那个页面是为人类准备的。

我把这些验证称为“友好”的方式将机器人踢出去。但您也可能从服务器获得一些不友好的响应：

![页面返回 403 HTTP 错误](img/Figure_9.03_B16113.jpg)

页面返回 403 HTTP 错误

如果你通常可以访问的页面在抓取时返回 403 错误，这意味着服务器认为你的行为是对服务器的攻击，并禁止了你的 IP。

这是个坏消息。这意味着你将无法使用当前的 IP 访问该网站。禁令可能是几个小时或永远。这也意味着，如果你在组织内部与其他计算机共享公共 IP，那么没有人将能够访问该网站。有两种方法可以解决这个问题。一种是通过联系网站所有者，道歉并请求他们从禁止列表中移除你的 IP。或者你可以继续糟糕地玩，尝试更改你的公共 IP 或使用代理。但如果你被抓到一次，你将被抓到两次。

网站告诉你不要抓取的最后一种方式是提供 API。

### 使用网站 API

在这个背景下，API 是网站公开的一组 URL，这些 URL 在返回页面而不是返回数据时，通常以 JSON 格式返回数据：

![Wikipedia API](img/Wikipedia_API.jpg)

![Figure_9.04_B16113.jpg](img/Figure_9.04_B16113.jpg)

维基百科 API

这里的问题是，如果你可以通过点击 URL 获取所有需要的信息，为什么还要浪费时间解析 HTML 元素，等待网络调用等等？

我认为这是一种很好的方式，让网站告诉你：“嘿，别抓取我，这里是你可能需要的数据。欢迎你来这里获取你需要的信息。”

API 还给了网站所有者设置规则的机会，例如速率限制（每秒或每分钟可以发送多少请求）、API 公开的数据以及对于消费者来说将保持隐藏的内容。

这些是一些网站可以传达我们可以做什么和不能做什么的方式。但我们对被抓取网站的看法也很重要。

### 我们的态度

这部分相当简单，可以简化为两个词：友好。你必须想到，在你试图抓取的页面另一边，有像你一样的人，他们的目标是保持网站运行。他们必须为服务器和网络带宽付费。记住，他们不是你的敌人。他们是你的朋友。他们拥有你需要的数据，所以请友好对待。

你应该首先查看可用的 API。如果一个网站公开了包含你想要的数据的 API，那么就没有必要抓取该网站。

接下来，你应该考虑你的抓取速率/速度。虽然你可能希望尽可能快地抓取网站，但我建议尽量使抓取过程接近真实用户交互。在本章的后面部分，我们将看到用于并行抓取页面的工具，但我会非常小心地使用这些工具。

许多使用 Puppeteer 的抓取器会忘记一件事。你应该始终表明自己是一个机器人。在下一节中，我们将看到这个想法是告诉服务器你是一个机器人而不是真实用户。

我们最后需要考虑的是评估我们正在提取的数据。

### 我们正在抓取哪些数据？

我认为这是常识。我们在提取受版权保护的数据吗？我们在提取网站的资产吗？例如，如果我们去一个歌词网站并提取歌词，我们就是在剥夺网站的目的。但如果我们去航空公司的网站查看航班价格，价格并不是公司的资产。他们卖的是航班，而不是价格。

另一件事要考虑的是我们提取的数据将用于什么。我们应该考虑我们的行为是否会增强抓取的网站，例如，在酒店预订网站的情况下，他们可能会获得更多客户，或者威胁它，例如，如果我们抓取歌词来创建我们自己的歌词网站。

诚实地讲，我们今天所知道的许多网站都使用了爬虫技术来播种他们的网站。你可能会在房地产网站上看到这一点。谁会想去一个空荡荡的房地产网站呢？没有人。所以，这些网站会通过其他网站的帖子来播种，以吸引新客户。

理论已经足够了。让我们创建一些爬虫。

# 创建爬虫

让我们尝试从 Packt 网站抓取书籍价格。条款和条件对爬虫没有任何说明([`www.hardkoded.com/ui-testing-with-puppeteer/packtpub-terms`](https://www.hardkoded.com/ui-testing-with-puppeteer/packtpub-terms))。但是`robots.txt`文件有一些明确的规则：

```js
User-agent: *
Disallow: /index.php/
Disallow: /*?
Disallow: /checkout/
Disallow: /app/
Disallow: /lib/
Disallow: /*.php$
Disallow: /pkginfo/
Disallow: /report/
Disallow: /var/
Disallow: /catalog/
Disallow: /customer/
Disallow: /sendfriend/
Disallow: /review/
Disallow: /*SID=
```

他们不希望我们去那些页面。但是网站有一个相当庞大的`sitemap.xml`，超过 9,000 行。如果`robots.txt`是爬虫的“不要去这里”的标志，那么`sitemap.xml`就是“请查看这个”的标志。这些是`sitemap.xml`文件上的第一个条目：

```js
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
    xmlns:image="http://www.google.com/schemas/sitemap-image/1.1">
    <url>
        <loc>https://www.packtpub.com/web-development</loc>
        <lastmod>2020-12-15T12:22:50+00:00</lastmod>
        <changefreq>daily</changefreq>
        <priority>0.5</priority>
    </url>
    <url>
        <loc>https://www.packtpub.com/web-development/ecommerce</loc>
        <lastmod>2019-09-13T07:29:53+00:00</lastmod>
        <changefreq>daily</changefreq>
        <priority>0.5</priority>
    </url>
    ...
</urlset>
```

基于这个 XML，我们将构建一个**爬虫**。爬虫是一个程序，它将导航网站并抓取所有页面。这是我们计划：

+   我们将构建一个书籍类别 URL 数组。为了使运行更短，我们将只抓取类别页面，例如[`www.packtpub.com/web-development`](https://www.packtpub.com/web-development)。我们还将限制列表为 10 个类别，这样我们就对服务器友好。

+   一旦我们得到这个列表，我们将导航每个页面，获取书籍链接。我们不希望其中出现重复，所以我们需要小心。

+   一旦我们得到书籍列表，我们将导航每个书籍页面，获取平装书的定价以及电子书和平装书的单独定价：

![书籍详情](img/Figure_9.05_B16113.jpg)

书籍详情

+   这些价格将被收集到一个 JSON 数组中，并在过程结束时发送到磁盘。

让我们开始吧！你可以在这个部分的`crawler.js`文件中找到这段代码。我们将获取`sitemap.xml`文件并获取前 10 个类别：

```js
const https = require('https');
(async function() {
    const sitemapxml = await getSitemap();
})();
function getSitemap() {
    let resolve;
    const promise = new Promise(r => resolve = r);
    https.get('https://www.packtpub.com/sitemap.xml', function(res) {
        let body = '';
        res.on('data', (chunk)  => body += chunk);
        res.on('end', () => resolve(body));
    });
    return promise;
};
```

在这里，我们使用原生的 `https` 包下载 `sitemap.xml` 文件。我们创建一个当调用 `resolve` 函数时将会解决的 Promise。然后我们调用 `get` 函数来获取文件。我们在 `data` 事件中收集正在下载的信息，并在接收到 `end` 事件时，通过返回我们收集的 `body` 字符串来解决 Promise。理解代码的流程可能需要一些时间，但一旦习惯了，它就非常直接。

`sitemapxml` 变量是一个字符串。我们首先需要解析 XML，然后从中获取一个 JavaScript 模型。`xml2js` 将为我们完成大部分工作。让我们在终端中使用 `npm install` 安装该模块：

```js
npm install xml2js   
```

一旦我们有了这个模块，我们就可以开始在代码中使用它了：

```js
const xmlParser = require('xml2js').parseString;
```

我们可以通过调用 `xmlParser` 函数来解析 sitemap：

```js
(async function() {
    const sitemapxml = await getSitemap();
    const categories = await getCategories(sitemapxml);
})();
function getCategories(sitemapxml) {
    let resolve;
    const promise = new Promise(r => resolve = r);
    xmlParser(sitemapxml, function (err, result) {
        const output = result.urlset.url
          .filter(url => url.loc[0].match(/\//g).length === 3)
          .slice(0, 10)
          .map(url => url.loc[0]);
        resolve(output);
    });
    return promise;
};
```

正如你所见，我们正在使用之前用过的相同的 `Promise` 模式。当我们调用 `xmlParser` 时，我们将在刚刚传入的回调函数的结果参数中获取解析后的 `result`。一旦我们得到结果，我们就准备输出。在查看 `sitemap.xml` 文件的同时阅读代码可能会有所帮助，以获取更多上下文。我们从 `result.urlset.url` 数组中获取 URL 元素。然后，我们使用带有三个斜杠的 `loc`（例如 [`www.packtpub.com/web-development`](https://www.packtpub.com/web-development)）来 `filter` URL 元素。然后，我们使用 `slice` 函数仅获取前 10 个元素。最后，我们使用 `map` 函数仅返回结果 URL，返回一个包含类别 URL 的字符串数组。

现在是时候使用 Puppeteer 了。我们将导航从 `sitemap.xml` 中获取的每个类别，并返回书籍 URL。我们不会只抓取类别页面的第一页。我将这个功能留给你作为作业。

让我们从创建一个将在整个程序中使用的浏览器开始：

```js
(async function() {
    /*Previous code*/
    const books = [];
    const page = await getPuppeteerPage();
})();
async function getPuppeteerPage() {
    const browser = await puppeteer.launch({
        headless: false, 
        slowMo: 500
    });
    const userAgent = await browser.userAgent();
    const page = await browser.newPage();
    await page.setUserAgent(userAgent + ' UITestingWithPuppeteerDemoBot');
    return page;
}
```

我敢打赌你对这段代码非常熟悉。在这里我们能看到的第一个新事物是我们正在传递 `slowMo` 选项。这意味着在 Puppeteer 执行的每个动作之后，我们将等待 500 毫秒。我们还将使用 `userAgent` 函数从浏览器中获取用户代理。然后，我们获取那个字符串，并追加 `' UITestingWithPuppeteerDemoBot'`，这样出版商的服务器管理员就会知道那是我们。

让我们开始抓取吧！

```js
(async function() {
    /*Previous code*/
    for(const categoryURL of categories) {
        const newBooks = await getBooks(categoryURL, page);
        if(newBooks) {
            books.push(...newBooks);
        }
    }
    page.browser().close();
})();
async function getBooks(categoryURL, page) {
    try {
        await page.goto(categoryURL);
        await page.waitForSelector('a.card-body');
        return await page.evaluate(() => {
            const links = document.querySelectorAll('a.card-body');
            return Array.from(links).map(l => l.getAttribute('href')).slice(0, 10);
        });
    }
    catch {
        console.log(`Unable to get books from ${categoryURL}`);
    }
}
```

我们将在主函数中遍历 `categories` 列表，并调用 `getBooks`，传递 `categoryURL` 和一个 Puppeteer 页面。该函数将返回一个包含书籍 URL 的列表，我们将使用 `push` 函数将其追加到我们的 `books` 数组中。我们使用 `slice(0, 10)`，所以只返回前 10 个项目。

我们将所有代码包裹在一个 `try/catch` 块中，因为我们不希望如果某个类别失败时代码会失败。

现在我们来看看`getBooks`函数。它看起来相当直接。我们进入`categoryURL`，使用`a.card-body` CSS 选择器等待一个元素。这个选择器将给我们书籍的 URL。一旦书籍加载，我们将调用`evaluate`，这样我们就可以获取所有带有`a.card-body`的链接，然后，使用`map`函数，我们将返回链接的`href`属性，这将给我们 URL。

抓取书籍不会有太大不同：

```js
(async function() {
    /*Previous code*/
    const prices = [];
    for(const bookURL of books) {
        const price = await getPrice(bookURL, page);
        if(price) {
            prices.push(price);
        }
    }
    fs.writeFile('./prices.json', prices);
    page.browser().close();
})();
async function getPrice(bookURL, page) {
    try
    {
        await page.goto(bookURL);
        await page.waitForSelector('.price-list__item .price-list__price');
        return await page.evaluate(() => {
            const prices = document.querySelectorAll('.price-list__item .price-list__price');
            if(document.querySelectorAll('.price-list__name')[1].innerText.trim() == 'Print + eBook') {
                return {
                    book: document.querySelector('.product-info__title').innerText,
                    print: prices[1].innerText,
                    ebook: prices[2].innerText,
                }
            }
        });
    }
    catch {
        console.log(`Unable to get price from ${bookURL}`);
    }
}
```

在这里，我们正在应用这本书中学到的所有内容。我们进入一个页面，等待选择器，然后我们将调用`evaluate`函数，它将返回一个对象。我们还没有以这种方式使用`evaluate`函数。

在`evaluate`内部，我们使用`.price-list__item .price-list__price` CSS 选择器获取价格，使用`.product-info__title` CSS 选择器获取书籍标题。然后，如果产品名称是`"Print + eBook"`，因为该网站还提供视频，我们返回一个具有三个属性的对象：`book`、`print`和`ebook`。

最后要强调的是，我们将代码包裹在`try/catch`块中。如果我们抓取一本书失败，我们不希望整个程序失败。

主要函数将收集这些结果，然后使用`fs.writeFile`将它们保存到文件中。为了使用该函数，您需要在程序的第一行添加`const fs = require('fs');`来导入`fs`。

如果一切如预期进行，我们将得到一个`prices.json`文件，内容如下：

```js
[
  {
    "book": "Kubernetes and Docker - An Enterprise Guide",
    "print": "$39.99",
    "ebook": "$5.00 Was $27.99"
  },
  {
    "book": "The Docker Workshop",
    "print": "$39.99",
    "ebook": "$5.00 Was $27.99"
  },
]
```

我们有我们的第一个抓取器。从那里，您在文件系统中有了数据，可以由其他工具进行分析。

这可以做得更好吗？是的，可以。我们可以尝试看看是否可以进行一些并行抓取。

# 并行运行抓取器

我不是因为我编写了它才这么说，但我们的抓取器结构相当好。每个部分都被分离到不同的函数中，这使得很容易识别哪些部分可以并行运行。

我不想重复，但请记住，我们正在抓取的网站，在这个例子中，Packt，是我们的朋友，甚至是我的出版商。我们不会影响网站；我们想看起来像正常用户。我们不希望并行运行 1,000 次调用。我们不需要这样做。所以，我们将尝试以谨慎的方式并行运行我们的抓取器。

好消息是，我们不需要编写并行架构的代码来解决这个问题。我们将使用一个名为**puppeteer-cluster**（[`www.npmjs.com/package/puppeteer-cluster`](https://www.npmjs.com/package/puppeteer-cluster)）的包。这是这个库根据 npmjs 上的描述所做的工作：

+   处理爬取错误

+   在崩溃的情况下自动重启浏览器

+   如果任务失败，可以自动重试

+   提供不同的并发模型可供选择（页面、上下文、浏览器）

+   使用简单，小样板

+   提供进度视图和监控统计信息（见以下代码片段）

听起来很有前景。让我们看看我们如何实现它。首先，我们需要安装这个包：

```js
npm install puppeteer-cluster
```

这将使我们的包准备好使用。你可以在`crawler-with-cluster.js`文件中找到这一节的代码。让我们通过在我们的代码第一行调用`require`来在我们的爬虫中导入集群：

```js
const { Cluster } = require("puppeteer-cluster");
```

现在我们已经导入了`Cluster`类，我们可以在主函数中创建一个新的**集群**：

```js
const cluster = await Cluster.launch({
    concurrency: Cluster.CONCURRENCY_PAGE,
    maxConcurrency: 2,
    retryLimit: 1,
    monitor: true,
    puppeteerOptions: {
        headless : false, 
        slowMo: 500
    }
});
```

`Cluster.launch`函数有很多选项，但我认为现在我们只需要了解这些选项：

+   `concurrency`将告诉集群我们想要使用的隔离级别。默认值是`Cluster.CONCURRENCY_CONTEXT`。这些都是可用的选项：

    a) 使用`Cluster.CONCURRENCY_CONTEXT`，每个作业将有自己的上下文。

    b) 使用`Cluster.CONCURRENCY_PAGE`，每个作业将有自己的页面，但所有作业将共享相同的上下文。

    c) 使用`Cluster.CONCURRENCY_BROWSER`，每个作业将有自己的浏览器。

+   `maxConcurrency`将帮助我们设置我们想要同时运行多少个任务。

+   通过`retryLimit`，我们可以设置集群在任务失败时将运行任务多少次。默认值是 0，但我们将给它一次重试任务的机会，将其设置为 1。

+   如果我们将`monitor`选项设置为`true`，我们将得到一个漂亮的控制台输出，显示当前进程。

+   我们在这里要讨论的最后一个选项是`puppeteerOptions`。集群将把这个对象传递给`puppeteer.launch`函数。

包说明中提到的一件事是它支持错误处理。让我们添加示例中他们有的错误处理：

```js
cluster.on('taskerror', (err, data, willRetry) => {
    if (willRetry) {
      console.warn(`Encountered an error while crawling ${data}. ${err.message}\nThis job will be retried`);
    } else {
      console.error(`Failed to crawl ${data}: ${err.message}`);
    }
});
```

这看起来相当稳固。当任务失败时，集群将触发一个`taskerror`事件。在那里我们可以看到错误、数据和是否将重试操作。

我们不需要改变我们下载和处理`sitemap.xml`的方式。那里没有需要改变的地方。但一旦我们有了类别，而不是调用`getBooks`函数，我们将使用`queue`来完成这个任务：

```js
for(const categoryURL of categories) {
    cluster.queue(categoryURL, getBooks);
}
```

我们正在告诉集群我们需要通过传递那个`categoryURL`来运行`getBooks`。

我还有更多好消息。我们的爬取函数几乎准备好在集群中使用——**几乎准备好**。我们需要改变四件事：

```js
async function getBooks({page, data}) {
    const userAgent = await page.browser().userAgent();
    await page.setUserAgent(userAgent + ' UITestingWithPuppeteerDemoBot');
    await page.goto(data);
    await page.waitForSelector('a.card-body');

    const newBooks = await page.evaluate(() => {
        const links = document.querySelectorAll('a.card-body');
        return Array.from(links).map(l => l.getAttribute('href')).slice(0, 10);
    });
    for(const book of newBooks) {
        cluster.queue(book, getPrice);
    }
}
```

首先，我们对签名做了一点修改。不再是期望`(page, categoryURL)`，而是期望一个具有`page`属性和`data`属性的对象，其中`page`将是集群创建和管理的页面，而`data`属性将是我们在排队任务时传递的`categoryURL`实例。

提示

传递给`queue`函数的第一个参数不需要是 URL。甚至不需要是字符串。你可以传递任何对象，函数将获取该对象在`data`属性中的内容。

我们必须做的第二件事是在页面创建时添加对`setUserAgent`的调用，因为页面是由集群本身创建的。

然后，我们不是返回书籍列表，而是向队列中添加了更多任务，但在这个情况下，我们排队的函数是`getPrice`，传递了`book` URL。

我们最后需要做的就是移除`try/catch`块，因为集群会为我们处理这一点。

现在是时候更新`getPrice`函数了：

```js
async function getPrice({ page, data}) {
    const userAgent = await page.browser().userAgent();
    await page.setUserAgent(userAgent + ' UITestingWithPuppeteerDemoBot');
    await page.goto(data);
    await page.waitForSelector('.price-list__item .price-list__price');
    prices.push(await page.evaluate(() => {
        const prices = document.querySelectorAll('.price-list__item .price-list__price');
        if(document.querySelectorAll('.price-list__name')[1].innerText.trim() == 'Print + eBook') {
            return {
                book: document.querySelector('.product-info__title').innerText,
                print: prices[1].innerText,
                ebook: prices[2].innerText,
            }
        }
    }));
}
```

我们几乎做了同样的事情。我们更改了签名，添加了对`setUserAgent`的调用，移除了`try/catch`，并且不再返回价格，而是在函数内部将价格推送到`prices`数组中。

最后，我们需要等待集群完成其工作：

```js
await cluster.idle();
await cluster.close();
```

对`idle`的调用将等待所有任务完成，然后`close`函数将关闭浏览器和集群。让我们看看这一切是否都能正常工作！

```js
> node crawler-with-cluster.js
== Start:     2020-12-29 08:50:14.475
== Now:       2020-12-29 08:51:28.078 (running for 1.2 minutes)
== Progress:  6 / 70 (8.57%), errors: 0 (0.00%)
== Remaining: 13.1 minutes (@ 0.08 pages/second)
== Sys. load: 6.1% CPU / 95.3% memory
== Workers:   2
   #0 WORK https://www.packtpub.com/security
   #1 WORK https://www.packtpub.com/all-products
```

`puppeteer-cluster`的输出非常出色。我们可以看到已消耗的时间、进度以及工作者正在处理的内容。

到目前为止，我们一直在遵守规则。但如果我们想避免被检测为抓取器呢？让我们找出答案。

# 如何避免被检测为机器人

在提到抓取伦理之后，我犹豫是否要添加这一节。我认为我在说当所有者说不的时候，我的观点已经很明确了。但如果我要写一个关于抓取的章节，我认为我需要向你展示这些工具。然后，如何使用你到目前为止所学的信息，就取决于你了。

不希望被抓取的网站，如果正在被积极抓取，将会投入大量时间和金钱来尝试避免被抓取。如果抓取器不仅损害了网站的性能，还损害了业务，那么这种努力将变得更加重要。

负责处理机器人的开发者不会仅仅依赖于用户代理，因为我们看到，这很容易被操纵。他们应该只依赖于评估来自一个 IP 地址的请求数量，因为我们也看到，抓取器可以减慢其脚本，模拟一个感兴趣的用户的操作。

如果网站不能通过检查用户代理和监控流量峰值来阻止抓取器，他们将会尝试使用不同的技术来捕捉抓取器。

他们会首先引入 CAPTCHA。但是，正如我们将在本节中看到的，抓取器可以解决其中的一些。

然后，他们会尝试评估请求之间的时间间隔。你在 500 毫秒后点击了链接吗？你在不到 1 秒内填写了表单吗？你可能是一个机器人。

他们还可以添加 JavaScript 代码来检查你的浏览器功能。你没有插件吗？连 Chrome 默认提供的都没有？你没有设置语言吗？你可能是一个机器人。

最后，他们将会设置陷阱来捕捉你。例如，Packt 知道你可能正在使用`a.card-body` CSS 选择器抓取链接。他们可能会添加一个带有该选择器的隐藏链接，但那样的话，链接的 URL 可能是 https://www.packtpub.com/bot-detected。如果你访问了 bot-detected URL，你就会被发现。在表单的情况下，他们可能会添加一些隐藏的输入字段，一个典型的用户不会完成这些字段，因为它们是隐藏的。如果服务器收到了那个隐藏输入字段中的值，抱歉——你又再次被捕捉到了。

这是一个猫捉老鼠的游戏。老鼠总是会试图找到新的方法偷偷进入，而猫会努力填补墙上的漏洞。

话虽如此，让我们看看如果我们是游戏中这只老鼠，我们有哪些工具可用。

安东尼·瓦斯特尔有一个出色的机器人检测演示页面([`arh.antoinevastel.com/bots/areyouheadless`](https://arh.antoinevastel.com/bots/areyouheadless))。你可以在这个部分的`bot.js`文件中找到这段代码。让我们尝试使用 Puppeteer 截取该页面的屏幕截图：

```js
const puppeteer = require('puppeteer');
(async function() {
    const browser = await puppeteer.launch({});
    const page = await browser.newPage();
    await page.goto('https://arh.antoinevastel.com/bots/areyouheadless');
    await page.screenshot({ path : './bot.png'});
    browser.close()
})();
```

简单的 Puppeteer 代码。我们以无头模式打开浏览器，导航到页面，并截图。让我们看看截图的样子：

![安东尼·瓦斯特尔的机器人检测](img/Figure_9.06_B16113.jpg)

安东尼·瓦斯特尔的机器人检测

安东尼抓住了我们。我们被检测为机器人，但这并不是游戏的结束。我们还有一些事情可以做。让我们首先引入`puppeteer-extra`([`github.com/berstend/puppeteer-extra`](https://github.com/berstend/puppeteer-extra))。这个`puppeteer-extra`包允许我们向 Puppeteer 添加插件。这个包将允许我们使用`puppeteer-extra-plugin-stealth`插件([`www.npmjs.com/package/puppeteer-extra-plugin-stealth`](https://www.npmjs.com/package/puppeteer-extra-plugin-stealth))。在这个游戏中，这个包就像是一个老鼠大师。它将添加所有技巧（或许多技巧），这样我们的代码就不会被检测为机器人。

我们首先需要做的是在终端中安装这两个包：

```js
npm install puppeteer-extra
npm install puppeteer-extra-plugin-stealth
```

现在我们可以替换这一行：

```js
const puppeteer = require('puppeteer');
```

我们用这三行代码替换它：

```js
const puppeteer = require('puppeteer-extra');
const StealthPlugin = require('puppeteer-extra-plugin-stealth');
puppeteer.use(StealthPlugin());
```

在那里，我们导入`puppeteer`，但来自`puppeteer-extra`。然后，我们导入隐蔽插件并使用`use`函数安装它。就这样！

![安东尼·瓦斯特尔的机器人检测绕过](img/Figure_9.07_B16113.jpg)

安东尼·瓦斯特尔的机器人检测绕过

`puppeteer-extra-plugin-stealth`包并不是坚不可摧的。正如我之前提到的，这是一个猫捉老鼠的游戏。还有很多其他的额外工具可以使用。你可以在包的存储库中看到完整的列表([`www.hardkoded.com/ui-testing-with-puppeteer/puppeteer-extra-packages`](https://www.hardkoded.com/ui-testing-with-puppeteer/puppeteer-extra-packages))。在那里，你可以找到`puppeteer-extra-plugin-anonymize-ua`，它将在所有页面上更改用户代理，或者`puppeteer-extra-plugin-recaptcha`，它将尝试解决 reCAPTCHA([`www.google.com/recaptcha/about/`](https://www.google.com/recaptcha/about/))挑战。

如果我们不至少稍微谈谈如何处理授权，那么一个爬取章节就不会完整。

# 处理授权

认证和授权是网络开发中的一个广泛话题。认证是网站如何识别你的方式。简单来说，就是登录。另一方面，授权是在你认证后可以在网站上执行的操作，例如，检查你是否可以访问特定页面。

有许多种认证模式。我们在本书中介绍了最简单的一种：用户名和密码登录页面。但事情可能会更复杂。测试与 Facebook 或单点登录的集成可能相当具有挑战性，但它们将涉及自动化用户交互。

有一种认证方法，您无法通过自动化 DOM 来执行——**HTTP 基本认证**：

![HTTP 基本认证](img/Figure_9.08_B16113.jpg)

HTTP 基本认证

这种登录弹出窗口现在并不流行。事实上，我认为它们从未流行过。但您可能已经看到过它们，如果您设置过路由器。这个模态框就像我们在*第五章*中看到的对话框，*等待元素和网络调用*。Puppeteer 无法帮助我们完成这种认证，因为没有 HTML 可以自动化。幸运的是，自动化这个过程很容易。您可以在`authentication.js`文件中找到这个部分的代码：

```js
(async function() {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    await page.authenticate({username: 'user', password: 'password'});
    await page.goto('https://ptsv2.com/t/ui-testing-puppeteer/post');
    await page.screenshot({ path : './authentication.png'});
    browser.close()
})();
```

我们要认证进入 https://ptsv2.com/t/ui-testing-puppeteer/post 的唯一事情是在调用`goto`之前调用`authenticate`函数。`authenticate`函数期望一个包含两个属性的对象：`username`和`password`。

一旦我们完成认证，我们需要在每次请求中告诉服务器我们是谁，这样它们才能授权我们（或拒绝）执行某些任务。从理论上讲，Web 服务器是无状态的。除非 1）它们在响应中注入一些信息，使用 cookie，或者 2）我们告诉它们，否则它们没有方法知道我们是谁。最常见的方式是通过 HTTP 头。但这可以通过传递一个作为查询字符串参数或作为 HTTP POST 数据一部分的键来解决。

当您想要更改认证数据时，您需要从其他地方获取这些信息。您可能需要打开浏览器，登录您想要抓取的网站，并从那里提取认证数据，然后您可以在您的抓取器中使用这些数据。

假设您想要抓取 Packt 网站，但这次您想在登录状态下抓取。因此，您打开浏览器，登录，然后您可以使用像*Export cookie JSON file for Puppeteer*这样的工具扩展（您可以在 Chrome 网络商店中用这个名字找到它）来导出网站生成的所有 cookie。一旦我们有了包含所有 cookie 的名为`account.packtpub.com.cookies.json`的 JSON 文件，您可以将该文件复制到您的工区，并执行如下操作：

```js
const puppeteer = require('puppeteer');
const cookies = require('./account.packtpub.com.cookies.json');
(async function() {
    const browser = await puppeteer.launch({defaultViewport : { width: 1024, height: 1024}});
    const page = await browser.newPage();
    await page.setCookie(...cookies);
    await page.goto('https://account.packtpub.com/account/details');
    await page.waitForSelector('[autocomplete="given-name"]');
    await page.screenshot({ path : './cookies.png'});
    browser.close()
})();
```

代码中的新元素是对`setCookie`函数的调用。该函数期望一个 cookie 列表。由于我们所有的 cookie 都在一个 JSON 文件中，我们加载该 JSON 文件并将内容传递给`setCookie`函数。让我们看看文件中 cookie 的样子：

```js
{
  "name": "packt_privacy",
  "value": "true",
  "domain": ".packtpub.com",
  "path": "/",
  "expires": 1611427077,
  "httpOnly": false,
  "secure": false
}
```

结构相当简单直接。你不需要使用扩展并从 JSON 文件中加载 cookie。你可以调用 `setCookie` 函数，传递一个包含 `name`、`value`、`domain`、`path` 和 `expires` 属性的对象（后者是秒为单位的 Unix 时间），以及是否为 `httpOnly`，以及是否应该标记为 `secure`。

现在是时候看看我们如何处理使用 HTTP 头实现的授权了。你可能会发现一些网站使用 `authorization` HTTP 头来传递某种用户标识符。`authorization` 头看起来可能像这样：

```js
Authorization: <type> <credentials>
```

根据 MDN ([`www.hardkoded.com/ui-testing-with-puppeteer/authentication-schemes`](https://www.hardkoded.com/ui-testing-with-puppeteer/authentication-schemes))，你可以找到以下类型：`Basic`、`Bearer`、`Digest`、`HOBA`、`Mutual` 和 `AWS4-HMAC-SHA256`。如果这些名字听起来很吓人，请不要担心。你很可能只会看到 `Bearer` 类型。凭证是什么呢？嗯，这就是你在编写爬虫代码时需要找出的事情。你需要查看当你真正使用网站时发送了什么信息，并尝试模仿它。

在我们的例子中，我们将使用 `Basic`，因为这与之前看到的相同 HTTP 基本认证。当你使用认证弹出窗口登录时，浏览器将通过传递 `basic` 和 `username:password`（Base64 编码）来发送授权头。在我们的例子中，用户名是 `user`，密码是 `password`。因此，我们可以使用任何可用的 Base64 编码器，例如，[`www.base64encode.net/`](https://www.base64encode.net/)，并将 `user:password` 编码为 base64：`dXNlcjpwYXNzd29yZA==`。

我们可以通过两种方式注入这个头信息。第一种是使用 `setExtraHTTPHeaders` 函数。你可以在 `header-inject.js` 文件中看到这个代码：

```js
(async function() {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    await page.setExtraHTTPHeaders({
        authorization: 'basic dXNlcjpwYXNzd29yZA=='
    });
    await page.goto('https://ptsv2.com/t/ui-testing-puppeteer/post');
    await page.screenshot({ path : './authentication-header.png'});
    browser.close()
})();
```

`setExtraHTTPHeaders` 期望一个对象，其中属性名是头名称，值是头值。这里我们添加了 `authorization` 头，其值为 `'basic dXNlcjpwYXNzd29yZA=='`。就这样。Puppeteer 将将这个头添加到页面将发出的每一个请求中。

但如果我们试图抓取的网站需要授权头，而不是每个请求都需要，那会是什么情况？嗯，这会很棘手，但并不难。你可以参考 `header-inject2.js` 文件中的代码：

```js
const puppeteer = require('puppeteer');
(async function() {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    await page.setRequestInterception(true);
    page.on('request', r => {
        const overrides = {
            headers: r.headers()
        };
        if(r.url() == 'https://ptsv2.com/t/ui-testing-puppeteer/post')
            overrides.headers.authorization = 'basic dXNlcjpwYXNzd29yZA==';

        r.continue(overrides);
    });
    await page.goto('https://ptsv2.com/t/ui-testing-puppeteer/post');
    await page.screenshot({ path : './authentication-header.png'});
    browser.close()
})();
```

我们首先告诉 Puppeteer 我们想要拦截页面将发出的每一个请求。我们通过调用 `setRequestInterception` 函数，并将第一个参数设置为 `true` 来做到这一点。然后我们开始监听 `request` 事件。如果请求满足我们需要的条件，在这种情况下，如果它与我们的 URL 匹配，我们创建一个具有 `headers` 属性的 `overrides` 对象，然后调用请求对象的 `continue` 函数。我们不能覆盖头信息。`overrides` 对象还可以有 `url`、`method`（HTTP 方法）和 `postData` 属性。

请求对象还有一个名为 `abort` 的函数。使用这个函数，你可以取消那个请求。例如，你可以检查请求是否为图片，然后 `abort` 它。结果将是一个没有图片的网站。

重要提示

如果你调用 `setRequestInterception`，你需要实现一个 `request` 事件监听器。并且你需要对每个你监听的请求进行 `continue` 或 `abort`。

正如我在打开这个部分时提到的，这并不涵盖所有不同的身份验证和授权方案，但它将覆盖超过 90%的情况。现在是我们总结的时候了。

# 摘要

虽然这不是一本关于爬取的书，但我们在这里覆盖了很多内容。我希望第一章能给你一个关于爬取是什么的好印象，以及涵盖了你可以做什么和不应该做什么。我们还学习了如何创建自己的爬虫。我们在不到 100 行代码内创建了一个爬虫。我们向工具箱中添加了两个新工具：`puppeteer-cluster` 和 `puppeteer-extra`。我们在本章的最后学习了一点点关于身份验证和授权，几乎为你提供了开始爬取世界所需的一切。

如果在阅读本章之前你对爬取并不那么兴奋，我希望这能成为激发你开始创建自己的爬虫的火花。如果你已经了解爬取，我希望本章能为你提供更多作为专业人士爬取的工具。

我们下一章和最后一章将关于性能以及我们如何使用 Puppeteer 来衡量它。
