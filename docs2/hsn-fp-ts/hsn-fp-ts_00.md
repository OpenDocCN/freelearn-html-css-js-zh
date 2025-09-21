# 前言

函数式编程是一种将计算视为数学函数评估的编程范式，并避免改变状态和可变数据。函数式编程范式的起源可以追溯到 20 世纪 30 年代，当时阿隆佐·丘奇介绍了 Lambda 演算。Lambda 演算提供了一个描述函数及其评估的理论框架，是一种数学抽象，而不是编程语言。然而，Lambda 演算是大多数函数式编程语言的基础。

在 20 世纪 50 年代末，约翰·麦卡锡开发了 Lisp，这是最早的函数式编程语言之一。Lisp 引入了许多函数式编程范式特性，并成为其他流行函数式编程语言（如 Scheme 和 Clojure）的主要影响。 

1973 年，罗宾·米尔纳在爱丁堡大学创建了 ML。ML 最终发展成了几种替代语言，其中最常见的是 OCaml 和标准 ML。1977 年，约翰·巴克斯以允许“程序代数”并遵循组合性原则的方式定义了函数程序。1985 年，研究软件有限公司发布了 Miranda，对惰性函数式编程语言的兴趣增长。几年后，存在十多种非严格、纯函数式编程语言。1987 年，在俄勒冈州波特兰举行的函数式编程语言和计算机架构会议上，形成了强烈的共识，即应成立一个委员会来定义此类语言的开放标准；Haskell 应运而生。

20 世纪 70 年代和 80 年代是函数式编程显著进步的年份。然而，在 90 年代和 21 世纪初，函数式编程在与面向对象编程语言（如 Java 和 C#）的竞争中失去了市场份额。

在 2010 年代，JavaScript 的采用呈指数增长，成为最受欢迎的编程语言。Scheme 编程语言是 JavaScript 的主要影响因素之一，因此 JavaScript 实现了许多函数式编程特性，例如支持高阶函数。JavaScript 成为许多年轻开发者接触函数式编程的第一步。然而，由于 JavaScript 是一种多范式编程语言，许多人忽略了其函数式编程能力。然而，在最近几年，随着受函数式编程原则高度影响的技术（如 React、RxJS 和 Redux）的出现，JavaScript 社区对函数式编程的兴趣显著增加。

随着 JavaScript 的普及，JavaScript 应用程序的复杂性也呈指数级增长。Web 用户界面变得更加复杂，JavaScript 开始被用于多种替代场景，例如后端应用程序。随后，TypeScript 编程语言被引入作为一种工具，它允许我们管理新的复杂级别。

TypeScript 旨在通过向 JavaScript 添加静态类型系统来减少系统的复杂性。静态类型系统可以在编译时检测错误，这是一种有益的代码文档形式。在函数式编程中，静态类型系统可以非常有用。大多数面向对象的编程语言，如 Java 和 C#，正在缓慢地采用函数式编程特性，而复杂的静态类型系统通常与纯函数式编程语言（如 Haskell）相关联。

本书不会鼓励你停止使用面向对象编程。相反，我们将尝试将函数式编程和面向对象编程范式视为解决同一问题的两种不同解决方案：管理复杂性：

"面向对象编程通过封装移动部分使代码易于理解。函数式编程通过最小化移动部分使代码易于理解。"

– *迈克尔·费瑟斯*

随着云计算的采用持续增长，分布式系统的普及也在增加，因此，预计在接下来的十年中，函数式编程的受欢迎程度也将上升，因为它特别适合并发系统和分布式系统。函数式编程鼓励实现无状态组件，这些组件可以轻松扩展。由于分布式系统的复杂性通常很高，这仅仅是函数式编程如何作为对抗复杂性的武器的另一个例子。

将 TypeScript 与面向对象编程和函数式编程的原则和技术相结合，可以为我们提供更丰富的工具集，以对抗系统中的复杂性。本书将为你提供关于广泛的功能性编程原则、模式和技术的知识，这应该有助于你成为一个更全能的软件工程师，并为你应对现代 Web 应用程序中日益增长的复杂性做好准备。

# 本书面向对象

如果你是一名开发者，目标是首次学习函数式编程并提高你应用程序的质量，那么这本书就是为你准备的。不需要函数式编程的先验知识。然而，为了充分利用这本书，建议你具备 JavaScript 和 TypeScript 的基本理解。

# 本书涵盖内容

第一章，*函数式编程基础*，介绍了主要的函数式编程术语，如纯函数。

第二章，*精通函数*，深入探讨了函数式编程应用中的主要构建块——函数。本章还探讨了 TypeScript 中大多数与函数相关的特性。我们将学习如何在许多不同的场景下与函数一起工作，以及如何在处理函数时利用 TypeScript 的类型系统特性。

第三章，*精通异步编程*，深入探讨了 JavaScript 和 TypeScript 中的主要异步编程 API，包括回调、承诺、生成器和异步函数。这些 API 在函数式编程中相关，因为它们可以用来实现惰性求值。

第四章，*运行时 – 事件循环和 this 操作符*，是两章中第一章，专注于探索与许多函数式编程技术相关的运行时概念。例如，如果我们理解了事件循环，我们就能更好地理解递归。

第五章，*运行时 – 闭包和原型*，是第二章节，专注于探索与许多函数式编程技术相关的运行时概念。例如，理解闭包可以帮助我们理解一些高阶函数是如何工作的。

第六章，*函数式编程技术*，详细探讨了主要的函数式编程技术和模式。我们将探讨部分函数应用、函数组合和柯里化等概念。本章还探讨了其他许多函数式编程技术和模式，如无参数风格。

第七章，*范畴论*，探讨了范畴论。你将了解什么是代数数据类型以及它们之间的关系。然后，你将学习如何实现一些主要代数数据类型，包括函子和单子。

第八章，*不可变性、光学和惰性*，探讨了三个重要的函数式编程技术。你将了解什么是惰性求值，它的好处以及如何实现它。你还将了解不可变数据结构、它们的优点以及如何实现它们。最后，你将了解函数式光学以及它们如何帮助不可变数据结构。

第九章，*函数式响应式编程*，探讨了函数式响应式编程范式。我们将了解什么是可观察的，以及它们如何被用来简化我们的代码。我们还将学习如何使用 RxJS，这是 JavaScript 生态系统中最领先的响应式编程库。

*第十章, 实际应用中的函数式编程*，探讨了几个可用于生产环境的函数式编程库，如 Ramda 和 Funfix，以创建实际的函数式编程应用程序。

附录 A, *函数式编程学习路线图*，这是为 Fantasyland 学习机构和 LambdaConf 会议开发的。它用于跟踪我们对函数式编程的知识水平。

附录 B, *TypeScript 函数式编程库目录*，在这个附录中，你可以找到一个与 TypeScript 兼容的函数式编程库列表。

# 为了充分利用这本书

阅读这本书不需要任何额外的材料。不需要函数式编程的先验知识。然而，建议对 JavaScript 和 TypeScript 有基本了解，以便充分利用这本书。

建议按顺序阅读章节。然而，如果你是函数式编程的新手，但已经对函数、异步编程和运行时有了高级知识，你可能可以跳过第二到五章。

如果你有一些 JavaScript 经验，但 TypeScript 对你来说是新的，你可以参考[`www.typescriptlang.org/docs/handbook/basic-types.html`](http://www.typescriptlang.org/docs/handbook/basic-types.html)的 TypeScript 手册。如果 TypeScript 是你的第一门静态类型编程语言，这个资源可能特别有用。或者，你也可以参考由*Remo H. Jansen*和*Packt Publishing*出版的书籍*Learning TypeScript 2.x, Second Edition*。

如果你需要安装 Node.js 的帮助，可以参考[`nodejs.org/en/download/package-manager`](https://nodejs.org/en/download/package-manager/)的官方文档。如果你需要安装 TypeScript 的帮助，可以参考[`www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html`](http://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)的官方文档。

# 下载示例代码文件

你可以从[www.packt.com](http://www.packt.com)的账户下载这本书的示例代码文件。如果你在其他地方购买了这本书，你可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择 SUPPORT 标签。

1.  点击代码下载与勘误。

1.  在搜索框中输入书名，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本解压或提取文件夹：

+   WinRAR/7-Zip（适用于 Windows）

+   Zipeg/iZip/UnRarX（适用于 Mac）

+   7-Zip/PeaZip（适用于 Linux）

书籍的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Hands-On-Functional-Programming-with-Typescript`](https://github.com/PacktPublishing/Hands-On-Functional-Programming-with-Typescript)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的书籍和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 下载彩色图像

我们还提供了一个包含本书中使用的截图/图表的彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/9781788831437_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781788831437_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“将下载的`WebStorm-10*.dmg`磁盘映像文件作为系统中的另一个磁盘挂载。”

代码块设置如下：

```js
function find<T>(arr: T[], filter: (i: T) => boolean) {
    return arr.filter(filter);
}

find(heroes, (h) => h.name === "Spiderman");
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```js
const valueOfThis = { name: "Anakin", surname: "Skywalker" };
const greet = person.greet.bind(valueOfThis);
greet.call(valueOfThis, "Mos espa", "Tatooine");
greet.apply(valueOfThis, ["Mos espa", "Tatooine"]);
// Hi, my name is Remo Jansen. I'm from Mos espa Tatooine.
```

任何命令行输入或输出都按照以下方式编写：

```js
npm install ramda @types/ramda
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并通过`customercare@packtpub.com`给我们发送邮件。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packt.com`与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下您的评价。一旦您阅读并使用过这本书，为何不在购买它的网站上留下评价呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，而我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问 [packt.com](http://www.packt.com/).
