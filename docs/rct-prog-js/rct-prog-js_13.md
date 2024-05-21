# 附录 A. Node.js Kick start

在 Web 开发中，有一个后端服务器是可取的。可用的服务器和语言列表很长。但一个引起了极大兴奋的服务器是 Node.js，它让你可以在前端和后端开发中使用相同的 JavaScript，并且提供了真正有趣的可能性。

本书的目的是介绍 Facebook 的前端用户界面框架 ReactJS。本附录的目的是提供足够的后端内容来运行一个经过身份验证的服务器，虽然有许多不错的选择，但 Node.js 可以在不需要处理新语言的情况下工作。本附录中所做的基本工作涵盖了与其他服务器和后端语言可能涉及的领域相当的范围。

在本附录中，我们将涵盖以下主题：

+   Node.js 如何从 INTERCAL 中汲取灵感

+   Node.js，就像 JavaScript 一样，存在着许多隐患

+   移植 Pragmatometer

但首先让我们看看 Node.js 和 INTERCAL。

# Node.js 和 INTERCAL

INTERCAL，正式名称为“没有可发音首字母缩写的编译语言”，是由普林斯顿大学的 Don Woods 和 Jim Lyon 于 1972 年首次宣布。作为一个旨在讽刺各种编程语言的趋势和时尚的语言的原型示例，它可能更为人所知的是一个旨在不易使用，而故意和不必要地难以使用的语言的原型示例。它的“Hello, world!”代码包含了完整的、重复性压力伤害的 16 行；它传奇般的 ROT-13 加密/解密器（对于 Perl 或 Unix shell 命令来说是一个简单的一行代码）在`alt.folklore.computers`上被描述为“四页完全无法理解的代码”。INTERCAL 最初是以 EBCDIC 的穿孔卡形式发布的，这是一种被称为加密标准的字符编码。

一个被讽刺的趋势是 Edgser Dikjstra 的“Go to 语句被认为是有害的”，这项工作不仅仅是开创性的，可以说是计算机科学史上最重要的文章。有人说，程序员是一个被告知“去地狱！”而不是被“地狱”所冒犯的人，而是被“去”所冒犯的人。一个 INTERCAL 变种（C-INTERCAL）认为`Go to`语句确实是有害的，他们想尽可能远离`Go to`语句——比 IF-THEN-ELSE 语句和 while 循环要远得多。他们提供了一个比 IF-THEN-ELSE 和 while 循环更激进的离开`Go to`语句的方法——`come from`语句。而`Go to`语句是说，“如果执行到达代码的这一点，就去到代码的那个区域”，`come from`反义词是说，“如果执行到达代码的那个区域，切换过去并在代码的这一点继续进行”。

可能提出的建议是：Node.js 的天才之处，在本章中我们提供了一个快速入门，是它竭尽全力基于`come from`语句或类似的东西进行流程控制。现在，Node.js 也是一个可用 JavaScript 编程的服务器，这一点也不可小觑，但它已经完全超越了所有其他可用 JavaScript 编程的服务器。它的天才不仅来自 JavaScript，还来自一个在你能够用`come from`作为流程控制的主要工具来解决问题时效果最佳的开发环境。通常更喜欢的术语是异步回调函数，而不是`come from`，但当你意识到 Node.js 作为一个`come from`编程的活生生的例子默认情况下是高性能的，并且在性能上超越了竞争对手一个数量级时，你会发现 Node.js 的工作效果最佳。

一个有 C 语言经验的人来到 Perl 可能会被告知，“除非你考虑到哈希表，否则你并没有真正考虑 Perl”，或者一个老派的 Java 程序员可能会被告知，“除非你考虑到闭包，否则你并没有真正考虑 JavaScript”。同样地，来自任何其他主流网络服务器的人可能会被告知，“除非你考虑到`come from`风格的异步回调函数，否则你并没有真正考虑 Node.js”。

### 提示

自匿名函数出现以来，它们在 Lisp 中就很出色，在 JavaScript 中也是一个很棒的特性。但在处理 Node.js 回调时，考虑使用非嵌套的命名函数作为匿名内部函数的替代。

从技术上讲，在 Node.js 中使用异步回调函数是可选的。然而，强烈建议除非你正在使用一个 Node.js 的学习工具，比如优秀的“学习 Node.js 为了大胜利！”（这个标题显然是对*Learn You a Haskell For Great Good*的一个优秀前辈的致敬），Node.js 的学习工具是在[`nodeschool.io`](http://nodeschool.io)上推广的那个，你应该记住 Knuth 的两条规则：

+   **规则 1（适用于所有程序员）**：不要优化

+   **规则 2（仅适用于高级程序员）**：稍后再优化

在 Node.js 的上下文中，这变成了：

+   **规则 1（适用于所有 Node.js 黑客）**：不要使用同步方法，而应该使用异步方法

+   **规则 2（仅适用于高级 Node.js 黑客）**：稍后再添加任何同步功能。

举一个同步实现的代码示例，一个非 Node.js 的人可能会看到的方式，我们可以读取并打印一个文件，比如`/etc/passwd`（在 Windows 上，应该使用不同的完整路径；你可以用记事本或你喜欢的编辑器创建并保存一个）：

```js
var fs = require('fs');
var contents = fs.readFileSync('/etc/passwd');
console.log(contents);
```

使用`come from`异步回调功能实现：

```js
var fs = require('fs'); 
fs.readFile('/etc/passwd', 'utf8', function(err, data) {
  if (err) {
    console.log('There was an error:');
    console.log(err);
    return;
  }
  console.log(data);
});
```

`console.log()`是否阻塞与否并不关心我们。

这是一个稍微复杂一点的 Node.js 的*Hello, world!*程序，或者可能是*Hello, world!*之后的程序，它在 Node.js 中就是这样的：

```js
console.log('Hello, world!');
```

让我们详细评论一下异步示例。

导入包的标准方法是调用`require()`，并将结果保存在你想要用来访问包的变量中。`fs`包是少数几个自带 Node.js 的包之一，但 Node.js 还附带了通过**Node Package Manager**（**npm**）获得的整个包宇宙，这是一个对于使用 Linux 包管理器的人来说可能很熟悉的包管理器。使用 npm，你可以搜索例如 Express.js 这样的包，这将在本章中简要介绍。Express.js 在 Node.js 社区中很受欢迎，与 Node.js 配合良好，有点像 Ruby 的 Rails 或 Python 的 Django。搜索 Express.js 可以像这样：

```js
npm search express

```

一旦确定了你想要的包名（或者你认为你想要的），你就可以安装它：

```js
npm install express

```

在前面的代码中，fs.readFile()函数调用是设置 come from 行为的。像其他异步调用一样，它有两个必需的参数：一个基本参数（可能是一个数组），它被传递给 fs.readFile()，以及一个回调函数。当调用这个函数时，程序不会阻塞，而是注册一个读取文件的请求，并使用指定的参数，然后 Node.js 将其留在原地，并处理其他请求。这是非常重要的。程序在等待时不会阻塞和无所事事，而是服务其他需求，文件操作完成后，程序会回到 come from 的位置，并执行提供的回调函数。很难让正确使用的 Node.js 阻塞，除非使用会占用 CPU 的东西，而当前的 CPU 速度很少会因为一个无害的请求而阻塞 CPU（有兴趣的人可以使用 Node.js 通过[`bitcoinjs.org/`](http://bitcoinjs.org/)等工具挖掘比特币，但可能很少有人担心他们的 Node.js 服务器的性能问题，会让它在一边挖矿）。有一个集群模块旨在利用多个核心，Node.js 默认情况下在一个单线程进程上运行在一个核心上。但是，如果你对你的用例是否足够极端需要像集群这样的东西来执行 Node 的默认性能结构有任何疑问，你可能还不需要集群。

Node 的工作方式（至少不使用集群）还有一个额外的优势，即避免并发问题，因为它是单线程的，事实上不是并发的。这是一件非常好的事情。并发是一个棘手、危险的问题。有高度熟练的程序员擅长处理并发，但总的来说，并发应该被视为有害的一种东西，一种经常让大多数普通程序员感到困惑的潘多拉魔盒。也许有理由认为使用不可变数据的纯函数语言在并发方面是一个单独的情况，但在这里我们将坚持认为 Node.js 默认情况下可以处理大量请求，而不需要开发人员应对棘手的并发问题是件好事。

给 fs.readFile()的第二个参数是可选的，fs.readFile()允许它是有点不寻常的。正常的异步调用看起来像 identifier(data, callback)。在这种情况下，第二个可选参数值得更仔细地研究。

这个参数是用来从字节数组中创建字符串的编码，给定的参数是正常的默认编码，'utf-8'，尽管在这种情况下有一点诱惑，可以回退到'ascii'。这是因为 Unix 的/etc/passwd 文件比 UTF-[anything]早几十年。但我们将成为良好的网民并使用'utf-8'。

在我们的情况下，使用 UTF-8 和 ASCII 编码的行为可能是相似的。实际上，如果/etc/passwd 文件像多年来许多/etc/passwd 文件一样只包含 ASCII 字符，可能仅支持 ASCII 字符，输出将是相同的。但是，如果不指定某种编码，它们中的任何一个都将是一个不同的东西。没有进一步的更改，回调将以字节而不是普通意义上的 JavaScript 字符串给出。在这里，我们可以看到 Node.js 的一些重要特点。

在浏览器中，有一个最快的 JavaScript 引擎之间的竞争，Node.js 采用了 Google Chrome 的 V8 引擎的一个版本（Node.js 的分支可能使用更新的版本），并在某些方面进行了扩展，以便能够作为通用的运行时环境，包括作为 Web 服务器。这包括添加了一些在客户端 Web 浏览器 JavaScript 中不存在的扩展。处理套接字作为服务器就是一个例子，这个功能以及 I/O 都是以前讨论过的异步模型的杰出发展。

另一个差距与二进制数据有关。在撰写本书时，标准浏览器 JavaScript 实际上并没有直接处理二进制数据的方法。虽然在下面的代码中可能有处理二进制数据的明显工作，但没有明确的方法来表示“我想要 128（八位）字节的交替 1 和 0，以 1 开头”：

```js
<img id="portrait" />
<script>
(document.getElementById('portrait').src =
'http://cjsh.name/images/portrait.png');
</script>
```

Node.js 扩展了 V8 的功能以处理适当的二进制数据，并且经过试验和验证的 JSON 与新的二进制友好的 BSON 相辅相成。处理二进制数据是低级的，可能对于其类似 C 的特性来说太低级了。例如，当 C 程序员完全从使用`malloc()`（“内存分配”）切换到使用`calloc()`（“清除内存分配”）时，就会提高生产力并减少挫败感。`malloc()`函数分配了一个原始的内存块，其中包含了来自内存先前占用者的任何残留物，如果你没有正确初始化内存的任何部分，就会导致奇怪和神奇的效果。

`calloc()`函数分配了一个原始的内存块，并用零覆盖了任何先前的内容。记住 Pete Hunt 的话，“我宁愿是可预测的，也不愿意是正确的？”停止直接使用`malloc()`而转而使用`calloc()`是 C 程序员可以选择可预测而不是正确的主要方式。然而，出于错误的优化考虑（不擦除分配的字节内存会快上几分之一秒），Node.js 只提供了 C `malloc()`的等价物，没有提供任何`calloc()`的等价物，据我所知。幸运的是，Node.js 的 JavaScript（或者说 C）是如此强大，以至于很容易移植`calloc()`功能。只需制作一个处理 Node.js 字节分配处理的包装器，然后使所有位都为零，并且在分配字节时专门使用这个包装器。

回到刚刚突出显示的代码，在 Node.js 的思维中，从文件或网络中读取的不是字符串，而是二进制字节。现在这些字节可能很容易通过给定的编码进行转换，如果你提供了想要的编码，`fs.readFile()`将给你一个合适的字符串，而不仅仅是字节。但让我们看一下与之前类似的一些代码。Node.js，像许多良好的环境一样，提供了一个**读取-求值-打印-循环**（**REPL**）来尝试一些东西（调用 node 可执行文件而不跟随任何参数将激活 REPL）。从 REPL：

```js
> fs.readFile('/etc/passwd', function(err, data){console.log(data)});
undefined
> <Buffer 23 23 0a 23 20 55 73 65 72 20 44 61 74 61 62 61 73 65 0a 23 20 0a 23 20 4e 6f 74 65 20 74 68 61 74 20 74 68 69 73 20 66 69 6c 65 20 69 73 20 63 6f 6e 73 ...>
```

文件中的各个字节用十六进制代码表示。

回到我们上一个代码示例，`function(err, data)`回调签名是编程合同的正常回调签名。回调应该最终被调用，可能非常快地被调用。这应该通过一个“真值”`err`来完成，如果是这种情况，回调应该选择性地采取步骤来响应错误反馈中包含的任何信息，并且在不进一步进行的情况下无条件返回，或者一个空的 err，在这种情况下，函数的前提条件得到满足，回调应该采取适当的行动来接收所请求的数据。前面的代码说明了这种模式：检查空的`err`，选择性地通过记录诊断信息对其进行操作，并且如果 err 为空，则打印文件内容。

# 警告 - Node.js 及其生态系统很热，热得足以严重伤害你！

当我是一名助教时，有一个不那么明显的建议是不要告诉学生某件事“很容易”。事后想来原因有些明显：如果你告诉别人某件事很容易，那么那些看不到解决方案的人可能会感到（更加）愚蠢，因为他们不仅不知道如何解决问题，而且他们无法理解的问题是一个很容易的问题！

有些问题不仅令从 Python/Django 转过来的人感到恼火，Python/Django 会在更改任何内容后立即重新加载源代码。而在 Node.js 中，默认行为是，如果你做了一次更改，旧版本将一直保持活动状态，直到永远或者直到你手动停止并重新启动服务器。这种不恰当的行为不仅令 Python 程序员感到恼火，也令原生的 Node.js 用户感到恼火，他们提供了各种解决方法。在 StackOverflow 上的问题“Node.js 中的文件自动重新加载”在我写这篇文章时，已经有超过 200 个赞和 19 个答案；一次编辑将用户引导到一个看护脚本，node-supervisor，主页在[`tinyurl.com/reactjs-node-supervisor`](http://tinyurl.com/reactjs-node-supervisor)。这个问题为新用户提供了一个很好的机会，让他们感到愚蠢，因为他们以为已经解决了问题，但旧的错误行为完全没有改变。而且很容易忘记重启服务器；我已经多次这样做了。我想传达的信息是，“不，你不是因为 Node.js 的这种行为而感到愚蠢；只是 Node.js 的设计者没有理由在这里提供适当的行为。尽量应对它，也许可以从 node-supervisor 或其他解决方案中得到一点帮助，但请不要走开时觉得自己很蠢。你不是有问题的人；问题在于 Node.js 的默认行为。”

这一部分经过一些辩论后被保留了下来，正是因为我不想给人留下“这很容易”的印象。在让事情正常运转的过程中，我反复割伤了手，我不想掩盖困难，也不想让你相信：让 Node.js 及其生态系统正常运行是一件简单的事情，如果对你来说不简单，那就是你不知道自己在做什么。如果你在使用 Node.js 时没有遇到令人讨厌的困难，那太好了。如果你遇到了，我希望你不要走开时感到“我很蠢。一定是我有问题。”如果你在处理 Node.js 时遇到了令人讨厌的意外，你并不蠢。不是你的问题！是 Node.js 及其生态系统的问题！

接下来，我们将探讨一个示例项目，一个远程等效的快速而简单的基于 localStorage 的持久性，这在第十一章中有所涉及，*用实例演示 JavaScript 中的函数式响应式编程 - 添加一个草稿本并把所有内容放在一起*。那是一个成功，但在过程中遇到了太多的困难。我有时会比较 Python 和 JavaScript；但也许值得花点时间看看为什么 JavaScript 的 Node.js 与 Python 的 Django 相比确实很讨厌。

我对 Django 的第一次体验，经过多年的经验后，感觉它是一个很棒的强大工具，但出于某种奇怪的原因，它不小心没有被纳入 Python 的标准库。事实上，这是有充分理由的，既不需要批评 Python 也不需要批评 Django：正如 Python 的终身独裁者所观察到的那样，当某样东西“死”了，而不是当它仍在成长时，你才把它放入标准库。Django 仍在成长，它仍在变得更好。因此，无论它有多好，Django 都不属于 Python 的标准库。

在许多情境中，有一个最少惊讶原则，一旦你开始熟悉 Python，它就不会经常给你带来不愉快的惊喜。Django 确实会带来一些惊喜，比如它的模板系统，在 ASP 和 JSP 时代是一个引人注目的提议。（现在它已经过了它的 15 分钟的荣耀，即使是 Python/Django 开发人员也开始用更强大的东西替换模板系统。ReactJS 基本上做了相反的选择是完全正确的。）但矛盾的是，Django 和 ReactJS 都提供了反映了火星技术的模板，如《新黑客词典》中定义的：

> *[TMRC]一种具有远见卓识的品质，使人能够忽略标准方法，提出完全意想不到的新算法。从一个离奇的角度攻击问题，以前没有人想到过，但事后看来是完全合理的。比较 grok，zen。

使用 Node.js 的感觉与使用 Python 甚至与使用 ReactJS 完全不同。它更令人沮丧，更困难，而且有更多不合理的事情。

这里有一个例子：在最初的研究时，我打算使用 passport.js 来卸载身份验证的脏活。我最初打算使用 Facebook 身份验证，但说明涉及在 Facebook 开发者网站上创建一些东西，并从 Facebook 应用程序中获取信息。即使在探索 Facebook 开发者网站并询问后，“passport.js 说要从 Facebook 开发者网站上的我的应用程序获取 XYZ 信息”，我完全没有得到及时的答复。

缩小我的野心，我决定使用 passport.js 最基本的适当身份验证——用户名和密码——直到我发现提供的用户名和密码支持几乎完全没有用。

它无用的原因是，正如**创建，读取，更新，销毁**（CRUD）提供了基本责任的列举——任何处理数据和记录的严肃和完整工具都需要涵盖的基础（无论是 SQL 数据库，任何一种 NoSQL 数据库，保存在编程环境中的 pickled 数据，编辑器或电子邮件客户端）——在主流帐户管理中有一组基本的基础需要涵盖，无论是用户名/密码还是任何新的更温和的让用户跟踪另一个登录和密码的替代方案。虽然个别网站可能选择退出某些功能，但提供帐户管理的 CRUD 的重复和基本功能包括以下内容：

+   允许用户创建新帐户

+   允许用户使用现有帐户登录

+   替换丢失的密码，而不会将未加密的密码通过电子邮件发送给用户

+   可能是网站管理成员的一组扩展功能，如帐户管理（如果需要），锁定和解锁帐户，以及帐户删除。

passport.js 功能中唯一涵盖的基础是使用已经存在的帐户登录，由我无法确定的某种方式创建，并成功或不成功地进行身份验证。也许在 CRUD 方面唯一更加病态不完整的事情是几十年前在 Byte 杂志的 4 月 1 日问题中提供的，当时有人宣传了一个非常划算的只读存储器。

现在我们可能会注意到，支持 100％的 CRUD 并不是唯一可能的方法。多年前，“写一次，读多次”（WORM）磁盘驱动器曾经引起了一些关注。虽然可能没有现代笔记本电脑配备真正的 WORM 驱动器，但 ClojureScript 包括了大量的工作来提供 WORM 数据。在这种情况下，WORM 意味着数据被设计为排除更新（尽管您可以很容易地制作修改后的副本），并且删除被保留到垃圾回收：在完全支持 CRUD 的情况下，ClojureScript 的 WORM 数据只允许无损地创建和读取数据。ClojureScript 反映了一个经过深思熟虑的决定，在这种情况下提供 WORM 数据会更容易实现完全的 CRUD 支持。这个决定现在值得明显的尊重。

现在，对于身份验证来说，缺乏等同于完全 CRUD 支持并不是世界末日，因为至少还有另一组人更适当地处理了“身份验证 CRUD”。Stormpath 为 Node、Python、Java 和 REST 提供了广告服务。他们的一名开发人员为我重写了身份验证的代码，以便我使用他们的系统。虽然这可能只是对可能涵盖他们产品的作者的友好表示，但 Stormpath 的包含甚至不是一个适当的集成挑战；它真的很简单。现在应该声明一下，Stormpath 不是开源的，而是一个带有免费定价的 SaaS。全功能、免费且“无需信用卡”的开发者层每月有 10 万次 API 调用的配额，他们估计用户登录大约使用三个 API 调用。他们肯定有盈利动机，但如果您的流量足够大，需要付费服务层，您不应该在乎您必须支付给他们的小事。该系统给人的整体印象有点年轻，人们正在解决剩下的问题，但确实有人礼貌地告诉他们有些不成熟的地方，问题很快就解决了。

另一个基本困难围绕着数据库。可以说 MongoDB 很重要，再加上 mongoose 的“从 Node.js 访问 MongoDB”包，值得学习曲线。在准备本章时，搜索排名靠前的教程证明了如何创建模式并保存其中的内容，但对如何有用地处理已经存在的所有必要模式的数据库留下了猜测。随后的工作发现了一个现有的 Stack Overflow 解决方案，似乎涵盖了具有已经存在的模式和数据库的数据库 CRUD。在放弃之前，我可能已经接近成功，但我打算从一开始就使用 mongoose/MongoDB 进行数据库工作，但我还没有达到熟练程度。

另一个看起来非常合适的数据库，而且可能还有挽回的余地，是 HTML5 键值存储的服务器端实现。这种方法的主要优势是大多数优秀的前端开发人员都足够了解的 API。而且，这也是大多数不那么优秀的前端开发人员都足够了解的 API。但是使用`node-localstorage`包时，虽然不提供`dictionary-syntax`访问（您可能想使用`localStorage.setItem(key, value)`或`localStorage.getItem(key)`，但不是`localStorage[key]`），但实现了完整的 localStorage 语义，包括默认的 5MB 配额。为什么？服务器端 JavaScript 开发人员需要保护自己吗？

对于客户端数据库功能来说，每个网站的 5MB 配额确实是一个慷慨而有用的呼吸空间，让开发者可以更好地使用它。你可以设置一个更低的配额，仍然可以为开发者提供比使用 cookie 管理更好的改进。5MB 的限制并不适合快速进行大数据客户端处理，但对于有资源的开发者来说，这是一个非常慷慨的允许，可以做很多事情。另一方面，5MB 对于最近购买的大多数磁盘来说并不是一个特别大的部分。这意味着如果你和一个网站对于磁盘空间的合理使用意见不一致，或者一个网站只是贪婪，这并不会让你花费太多，你也不会有硬盘被淹没的危险，除非你的硬盘已经太满了。也许我们最好的平衡是更少一点，或者更多一点，但总的来说，这是一个相当不错的解决方案，可以解决客户端环境中的内在紧张关系。

然而，可能要温和地指出，当你自己为服务器编写代码时，你不需要额外的保护来使你的数据库超过 5MB 的容量。大多数开发者既不需要也不希望工具像保姆一样保护他们免于存储超过 5MB 的服务器端数据。此外，这个 5MB 的配额，在客户端是一个黄金的平衡，但在 Node.js 服务器上却有点傻。

此外，对于多用户的数据库，可能会有点痛苦地指出，这不是每个用户账户 5MB，除非你为每个账户创建一个单独的数据库列表。这是 5MB 在所有用户账户之间共享。如果你爆发了，这可能会很痛苦！

文档说明配额是可定制的，但一周前给开发者的电子邮件询问如何更改配额没有得到回复，同样的问题也在 Stack Overflow 上提问了，也没有得到答复。我唯一找到的答案是在 GitHub 的 CoffeeScript 源代码中，它被列为构造函数的可选第二个整数参数。这很容易，你可以指定一个与磁盘或分区大小相等的配额。但除了移植一个没有意义的功能之外，工具的作者还完全没有遵循一个非常标准的约定，即将 0 解释为“无限制”，用整数来指定相关资源使用的最大限制。对于这个缺陷，最好的做法可能是指定配额为无限大：

```js
if (typeof localStorage === 'undefined' || localStorage === null)
  {      
  var LocalStorage = require('node-localstorage').LocalStorage;
  localStorage = new LocalStorage(__dirname + '/localStorage',
    Infinity);
  }
```

在我的研究中，这种类似业余的粗糙性不断出现。Express.js 比 Node.js 更高级，但就自毁的方式而言，Node.js 更接近于提供 C 的方式，而不是我最近使用的任何其他技术。C 是为那些更喜欢在开枪前装载自己的子弹的人准备的。

# 一个示例项目 - 为我们的 Pragmatometer 提供服务器

让我们朝着一个简单的项目努力。我们将创建一个通用的服务器后端，可以为本书最后几章中涉及的 Pragmatometer 项目提供修改，该项目通过在 HTML5 的本地存储中保存和恢复几个 JSON 字符串来处理持久性。我们将致力于开发一个可以提供静态内容的服务器，提供一个 API 来保存或恢复一个字符串和一个标识键，并处理基本的身份验证和账户管理。客户端编程应该比以前更有趣，基本上是通过将保存到本地存储更换为保存到我们的远程 Node.js 服务器。

我们将使用多种技术。最关注的将是在 Express.js 中工作，与例如 Stormpath 相比。Stormpath 似乎没有因发明基本新的、原创的或令人惊叹的东西，或者因身份验证机制的突破而获得赞誉。他们可能会因以一种使大量繁重工作减轻你的负担的方式解决了一个众所周知的问题而获得赞誉。添加 Stormpath 是小而不显眼的。大多数用户不会将其用作构建一些伟大工作的平台。因此，我们将重点关注 Express.js（以及让我们的客户端与 Express.js 通信），这是一个可以使用的平台。在框架的网站上，Express.js 被宣传为“Node.js 的快速、不受限制的 Web 框架”。他们基本上实现了他们吹嘘的东西。

我们需要构建一个服务器，但也需要修改第八章到第十一章中 Pragmatometer 项目的客户端。有`save()`和`restore()`函数，它们将被修改和扩展。

通过`npm install express`安装 Express.js。然后使用`express [项目的目录名]`创建一个 express 项目。你将会有一个完整的框架。你可以向`package.json`文件添加包，并运行`npm install`来填充你的本地副本。将会有一个公共或静态目录可以使用，并且`routes/index.js`以其他框架熟悉的方式处理路由。

# 客户端准备工作

第八章到第十一章中`js/`目录下的所有内容都移动到`public/javascripts`。更改的完整细节将发布在网站上。在这里，我们将`save()`和`restore()`函数从（客户端）特定于 localStorage 的功能改为保留 localStorage 以获得轻微的感知速度提升，但从远程服务器恢复和保存。在这种情况下，服务器是使用 Express.js 构建的 Node.js 服务器，但基本上可以是任何提供相同、简单和隐式 API 的服务器。

通常情况下，使用 ReactJS，对象的状态是在`setInitialState()`调用中设置的。理论上，我们可以通过加载同步等效的 Ajax 调用来保留相关的语义，但也可以填充一个存根，然后提供一个真正启动事情的回调。用于在从 Ajax 调用成功返回时填充对象状态的函数是`populate_state()`。

```js
  var populate_state = function(state, new_data) {
    for (field in new_data) {
      if (new_data.hasOwnProperty(field)) {
        state[field] = new_data[field];
      }
    }
  state.initialized = true;
  }
```

`restore()`函数略微复杂，因为它被编写成构建感知层的响应。它进行了一个 Ajax 调用，设置一个状态为初始化，并将`state.initialized`标记为`false`。它还从 JSON 中恢复（如果有保存的内容）。它检查 localStorage 是否可用，并且如果不可用，则优雅地降级，这可能是历史性的，因为 ReactJS 只声称与提供 localStorage 的浏览器（IE8 及更高版本）一起工作。尽管如此，它提供了一个我们如何进行优雅降级的例子。

```js
  var restore = function(identifier, default_value, state, callback) {
    populate_state(state, default_value);
    state.initialized = false;
    var complete = function(jqxhr) {
      if (jqxhr.responseText === 'undefined' || jqxhr.responseText.length &&
        jqxhr.responseText[0] === '<') {
        // We deliberately do nothing.
      } else {
        populate_state(state, JSON.parse(jqxhr.responseText));
      }
      callback();
      state.initialized = true;
    }
    jQuery.ajax('/restore', {
      'complete': complete,
      'data': {
        'identifier': identifier,
        'userid': userid
      },
      'method': 'POST'
    });
    if (Modernizr.localstorage) {
      if (localStorage[identifier] === null || localStorage[identifier]
        === undefined) {
        return default_value;
      } else {
        return JSON.parse(localStorage[identifier]);
      }
    } else {
      return default_value;
    }
  }
```

作为范围的限制，前面代码中`restore()`函数的实现，以及后面代码中的`save()`函数都没有处理在失败的 Ajax 调用（或调用）中的弹性。解决这个问题的一种方法是检查失败并保持重试，随着重试之间的延迟呈指数增长，以成为一个良好的网络公民，不会给网络增加持久的重负载。这种模式在高层次上大致上被 Gmail 遵循，在 TCP/IP 中也被内置。对于我们的实现，任何一个失败的 Ajax 调用可能没有传达的内容应该在键值存储中重新可用，除非有后续更新，这种情况下通常会保存两个更改。

`save()`函数稍微简单一些，但它代表了另一面：进行 Ajax 调用以保存/恢复，并在可用之前将其保存到和从 localStorage 中恢复：

```js
  var save = function(identifier, data) {
    if (Modernizr.localstorage) {
      localStorage[identifier] = JSON.stringify(data);
    }
    jQuery.ajax('/save', {
      'data': {
        'data': JSON.stringify(data),
        'identifier': identifier,
        'userid': userid
      },
      'method': 'POST'
    });
  }
```

当我们从 localStorage 中拉取东西时，我们试图阻止用户能够输入数据。这是因为在不可预测的竞争条件下，当来自 Ajax 调用的数据返回时，这些数据会被覆盖。换句话说，用户被阻止添加任何输入，直到从 Ajax 中恢复数据（即使值已经从 localStorage 中恢复）。这意味着，特别是提交按钮被禁用，目前，给`restore()`的回调函数的唯一应用是启用已被禁用的提交按钮。对于日历，`render()`方法有一个禁用的**提交**按钮（您可以更加纯粹并禁用所有输入字段，但禁用**提交**按钮足以防止用户数据被竞争条件覆盖）：

```js
    render: function() {
      var result = [this.render_basic_entry(
        this.state.entry_being_added)];
      if (this.state.entry_being_added &&
        this.state.entry_being_added.hasOwnProperty('repeats') &&
        this.state.entry_being_added.repeats) {
        result.push(this.render_entry_additionals(
          this.state.entry_being_added));
      }
      return (<div id="Calendar">
        <h1>Calendar</h1>
        {this.render_upcoming()}<form onSubmit={
        this.handle_submit}>{result}
        <input type="submit" value="Save" id="submit-calendar"
          disabled="disabled" /></form></div>);
    },
```

日历的`getInitialState`函数只安排了一个简单的数据存根，以同步方式放置。Ajax 调用返回后，它会给出一个更合适的值，并重新启用禁用的保存按钮，因为这里不再关注竞争条件：

```js
    getInitialState: function() {
      default_value = {
        entries: [],
        entry_being_added: this.new_entry()
        };
      restore('Calendar', default_value,
        default_value, function()
        {
        jQuery('#submit-calendar').prop('disabled', false);
        });
```

客户端还有一些细节，但它们并不特别困难。例如，我们添加一个注销链接（使用 CSS 定位到右上角），并且 JavaScript 行为（不调用通常的`preventDefault()`方法，因为我们不想阻止默认行为）擦除键值存储中的帐户数据：

```js
  jQuery('#logout').click(function() {
    if (Modernizr.localstorage) {
      localStorage.removeItem('Calendar');
      localStorage.removeItem('Todo');
      localStorage.removeItem('Scratch');
    }
  });
```

# 服务器端

当我们需要包时，我们应该将它们添加到我们的`package.json`文件中。一种做法是反向进行。执行 npm install XYZ，然后在“dependencies”下的`package.json`文件中添加一行，指定“XYZ”:“~1.2.3”，并记录安装的版本号。目前包括的依赖关系如下：

```js
{
  "name": "Pragmatometer",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "node ./bin/www"
  },
  "dependencies": {
    "body-parser": "~1.13.2",
    "connect-flash": "~0.1.1",
    "debug": "~2.2.0",
    "ejs": "~2.3.2",
    "express": "~4.12.4",
    "express-stormpath": "~1.0.5",
    "jade": "~1.9.2",
    "morgan": "~1.5.3",
    "serve-favicon": "~2.2.1",
    "stormpath": "~0.10.0"
  }
}
```

我们在[`stormpath.com/`](https://stormpath.com/)创建一个帐户，可能是一个免费的开发者帐户（除非您知道您需要更多），并在`app.js`中指定各种细节。此设置使用类似 HTML 的 EJS 而不是类似 Markdown 的 Jade 进行视图：

```js
// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.engine('html', require('ejs').renderFile);
app.set('view engine', 'ejs');

app.use(logger('dev'));
// uncomment after placing your favicon in /public
app.use(favicon(__dirname + '/public/images/favicon.ico'));
app.use('/public', express.static(path.join(__dirname, 'public')));

// Authentication middleware.
app.use(stormpath.init(app, {
  apiKeyId: '[Deleted]',
  apiKeySecret: '[Deleted]',
  application:
    'https://api.stormpath.com/v1/applications/[Deleted]',
  secretKey: '[Deleted]',
  sessionDuration: 365 * 24 * 60 * 60 * 1000
  }));

app.use('/users', users);
```

标记为`[Deleted]`的项目中除了一个都是从 Stormpath 的设置中获取的。有些人建议在制作自己的秘钥时要聪明一些；不要这样做！在 Mac、Unix、Linux 或 Cygwin（Cygwin 可以从[`cygwin.org`](http://cygwin.org)免费获取，并在 Windows 下运行）下，打开命令提示符并输入以下命令：

```js
python -c "import binascii; print binascii.hexlify(open('/dev/random').read(1024))"

```

这将为您提供一千字节的加密强大和随机数据，该数据已编码为易于复制和粘贴。

这里有一个关于卫生的注意事项：建议的做法是非常小心地处理您的秘钥，特别是不要将其包含在版本控制中。而是将其放入主目录下的点文件目录中，并设置权限，不让其他人对其进行任何操作。

可能，工作量最大的文件之一是`routes/index.js`。我们引入了几个依赖项，包括一个 body 解析器，它将能够从 Ajax 保存中获取数据，该数据以 POST JSON 的形式在请求的主体中：

```js
var body_parser = require('body-parser');
var json_parser = body_parser.json();
var express = require('express');
var stormpath = require('express-stormpath');
```

我们包括 localStorage，指定无限制作为我们的配额，然后为键中的字符提供一个清理器。这个特定的清理器保留字母数字字符，这对于应用程序的其余部分足以确保它不会产生键冲突。它还确保字符在排除冒号的白名单上。这使我们能够创建类似`username:component-name`的键，对冒号进行字符串分割，并始终在零号位置获取用户名和在第一个位置获取组件名称：

```js
var sanitize = function(raw) { 
  var workbench = []; 
  for(var index = 0; index < raw.length; ++index) {
    if ('ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_.'
      .indexOf(raw[index]) !== -1) {
      workbench.push(raw[index])
    }
  }
  return workbench.join('');
}
```

路由器的工作方式应该对以前在几乎任何上下文中看到过路由器的用户来说是熟悉的。虽然路由和非路由函数将被混合使用，但路由器是创建并连接到前两个路由的：

```js
var router = express.Router();

router.get('/', stormpath.loginRequired, function(request, response) {
  response.render('index.ejs');
});

router.post('/', stormpath.loginRequired, function(request, response) {
  response.render('index.ejs');
});
```

一旦包含了 Stormpath，包含的`stormpath.loginRequired`就是你需要的一切，以便让视图受到登录保护。我们继续定义两个非视图函数：用于`save()`和`restore()`特定用户的键的函数：

```js
var save = function(userid, identifier, value) {
  localStorage.setItem(sanitize(userid) + ':' + sanitize(identifier), value);
  return true;
}

var restore = function(userid, identifier) {
  var value = localStorage.getItem(sanitize(userid) + ':' + sanitize(identifier));
  if (value) {
    return value;
  } else {
    return 'undefined';
  }
}
```

我们添加了用于处理 POST Ajax 请求的路由。如果我们想要添加对 GET 或其他动词的支持，我们可以调用`router.get()`等方法：

```js
router.post('/restore', json_parser, function(request, response, next) {
  var result = restore(request.user.href, request.body.identifier);
  response.type('application/json');
  response.send(result);
});

router.post('/save', function(request, response, next) {
  var success_or_failure = save_identifier(request.user.href,
  request.query.identifier,
    request.query.data);
  response.type('application/json');
  response.send(String(success_or_failure));
  });
```

然后有一行样板代码我们保持不变：

```js
module.exports = router;
```

我们还使用 Express.js 的层次结构来存储静态数据；修改后的`index.ejs`从与之前的 js/不同的位置获取数据：

```js
    <script
      src="img/react-with-addons.js">
    </script>
    <script
      src="img/showdown.js">
    </script>
    <script src="img/ckeditor.js"></script>
    <script src="img/"></script>
    <script src="img/json2.js"></script>
    <script src="img/modernizr.js"></script>
    <script src="img/site.js"></script>
```

就是这样！我们在电子资源包中提供了详细信息。现在我们已经提供了一个带有账户管理的服务器端键值存储。

# 摘要

在考虑这个附录时，我考虑过的一个问题是“JavaScript 加上 Node.js 还是 Python 加上 Django？”本书的重点是 ReactJS 前端，后端的重点只是为了提供足够的支持前端。我自然地认为 Python 是如此简单，即使对新手来说也是如此，Django 也是如此简单（同样，即使对新手来说也是如此），即使引入一种新语言，基本的带有认证的键值存储应该是一个容易阅读和编写的附录。然而，作者当时认为我会选择 JavaScript 加上 Node.js 这条高路，这是每个人都想要的组合，自那时起，我一直在为他的决定付出代价，因为他没有提供 Python 加上 Django 的附录。

捆绑提供的代码当然是免费提供的，你可以从中获取任何不违反 Packt Publishing 许可的里程。但是，实现带有账户管理的键值存储的基本任务可能是本科生的家庭作业水平。从任何方面来看，它都不能展示出服务器所提供的惊人功能。

现在，Node.js 确实提供了令人惊叹的功能。然而，这些功能在这里没有被探索，因为目标是提供足够的“Node.js 加上 Express.js”来创建一个基于服务器的 Pragmatometer 项目的适应版本，该项目在第 8 到 11 章中进行了介绍。此外，鉴于对所有项目的热情和大量工作时间，可能有必要在撰写本书时对关于不成熟生态系统的任何评论进行严格的限制，可能需要 1 年、2 年或 3 年的时间。5 年后，可能真的有必要说，“2015 年的 Node.js 生态系统存在一些隐患。2020 年的 Node.js 生态系统有多个乐园。”

但是，像 passport.js 一样发布，通过`passport.authenticate('twitter')`、`passport.authenticate('google')`、`passport.authenticate('facebook')`等方式简单地实现动画效果，然后让用户长时间搜索和询问如何处理用户名密码认证以允许用户创建新账户，这是不会发生的。这是极其不合适的，而且在 Node.js 生态系统中发生了不止一次。在找到一个看起来提供了你需要的功能的 Node.js 工具的网站和激活“Hello, world!”级别的功能之间的过渡，成功率也许只有 50%。这代表了一个比我在整个 Web 历史上见过的任何事情都更大的鸿沟。

我可以看到人们会认为，不是因为我设计了一个带有多种状态的待办事项清单，而觉得我很聪明，而是因为我很实际，总的来说，这本书大大减少了读者在学习 ReactJS 时所需的工作量。然而，如果有人告诉我我很聪明，因为我想到了制作一个经过身份验证的键值存储，我会感到困惑，因为这在附录中有所涉及。这个成就并不是因为我设法让某些技术作为一个经过身份验证的键值存储工作——这与本科作业相当——而是因为它是在与 INTERCAL 连续存在的环境中完成的。

人们不断地将自己置于困境中，因为他们不断地将 JavaScript 作为一个整体来使用，而 JavaScript 能够成为一门受人尊敬的语言，关键在于道格拉斯·克罗克福德的一句话，本质上是这样说的：

> “JavaScript 作为一种语言有一些非常好的部分和一些非常糟糕的部分。这里是好的部分。只要忘记其他的东西存在。”

也许炙手可热的 Node.js 生态系统将会培养出自己的“道格拉斯·克罗克福德”，他会说：

Node.js 生态系统就像编码的西部荒野，但也有一些真正的宝藏。这是一张路线图。这些是几乎可以不惜任何代价避免的领域。这些领域是任何语言或环境中可以找到一些最丰富的矿藏的地方。

也许其他人可以把这些话当作一个挑战，跟随克罗克福德的脚步，为 Node.js 及其生态系统编写《好的部分》和/或《更好的部分》。我会买一本！
