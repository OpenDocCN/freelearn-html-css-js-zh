# 1

# 准备工作

Node.js 是一种服务器端 JavaScript 运行时环境，已成为创建网络应用中最受欢迎的平台之一。Node.js 享有庞大的包和框架库，提供了各种可想象的功能和功能，这些功能和功能建立在 Node.js 提供的丰富 API 上，用于处理底层任务。

本书与其他大多数 Node.js 书籍不同，因为它解释了 Node.js API 和流行网络应用包之间的关系。每一章都解释了如何使用 Node.js API 实现网络开发所需的核心功能，然后在替换自定义实现之前，使用流行的、经过良好测试的开源包。

理解 Node.js API 可以让你深入了解网络应用功能是如何真正工作的，而使用流行的包可以让你在不编写自定义代码的情况下构建这些功能。当问题出现时，任何项目都会出现，你对 Node.js API 的了解将为你提供一个坚实的基础，以揭示特定包的工作方式，并找出出了什么问题。

本书的第一部分概述了 Node.js 开发中使用的工具，并为最重要的语言和平台特性提供了入门指南，包括 JavaScript 如何实现并发，以及这与使用 Node.js 处理 HTTP 请求的关系。

本书第二部分重点介绍网络应用的关键构建块：生成 HTML 内容、处理表单、使用数据库和验证用户。

本书的第三部分和最后一部分演示了如何将前面章节中描述的功能结合起来，创建一个简单但真实的网络商店应用程序。

这是我在许多书中使用过的 SportsStore 示例，它包括产品目录、购物车、结账流程和管理功能等常见特性。

# 你需要了解什么？

要跟随本书中的示例，你应该熟悉 HTML 和 CSS，并了解 JavaScript 开发的基础知识。我为本书中有用的 JavaScript 功能提供了一个入门指南，但这不是一个完整的语言教程。

# 你如何设置你的开发环境？

Node.js 开发所需工具在 *第二章* 中已设置。后续章节需要额外的工具和包，并为每个工具和包提供完整说明。

# 有很多示例吗？

有 *大量* 的示例。最佳的学习方式是通过示例，我已经将尽可能多的示例打包到这本书中。为了最大化本书中的示例数量，我采用了一个简单的约定来避免重复列出相同的代码或内容。当我创建一个文件时，我会展示其全部内容，就像我在 *列表 1.1* 中所做的那样。我在列表的标题中包含文件名及其文件夹，并以粗体显示我所做的更改。

列表 1.1：在 primer 文件夹中的 index.ts 文件中断言一个未知值

```js
function getUKCapital() : string {
    return "London";
}
function writeCity(f: () => string)  {
    console.log(`City: ${f()}`)
}
writeCity(getUKCapital);
writeCity(() => "Paris");
**let myCity = "Rome";**
**writeCity(() => myCity);** 
```

这是从**第三章**中列出的一部分，展示了可以在`primer`文件夹中找到的名为`index.ts`的文件的内容。不要担心列表的内容或文件的目的；只需意识到这种类型的列表包含文件的完整内容，而你需要为遵循示例所做的更改以加粗形式显示。

一些代码文件变得很长，而我将要描述的功能只需要进行小的更改。而不是列出完整的文件，我使用省略号（连续的三个点）来表示部分列表，这仅显示了文件的一部分，如*列表 1.2*所示。

列表 1.2\. 在 src/server/data 文件夹中的 sql_repository.Ts 文件中包含用户输入

```js
...
getResultsByName($name: string, $limit: number): Promise<Result[]> {
    **return this.executeQuery****(`**
**SELECT Results.*, name, age, years, nextage FROM Results**
 **INNER JOIN People ON personId = People.id**
**INNER JOIN Calculations ON calculationId = Calculations.id**
**WHERE name = "${$name}"`, {});**
}
... 
```

这是从**第十二章**中列出的一部分，它展示了对一个较大文件的一部分所做的更改。当你看到部分列表时，你会知道文件的其余部分不需要更改，只有被加粗标记的部分是不同的。

在某些情况下，文件的不同部分需要更改，这使得很难以部分列表的形式展示。在这种情况下，我省略了部分文件内容，如*列表 1.3*所示。

列表 1.3\. 在 src/server/data 文件夹中的 orm_helpers.ts 文件中定义模型关系

```js
import { DataTypes, Sequelize } from "sequelize";
import { Calculation, Person, ResultModel } from "./orm_models";
const primaryKey = {
    id: {
        type: DataTypes.INTEGER,
        autoIncrement: true,
        primaryKey: true
    }       
};
export const initializeModels = (sequelize: Sequelize) => {
    // ...statements omitted for brevity...
}
**export const defineRelationships = () => {**
**ResultModel.belongsTo(Person, { foreignKey: "personId" });**
**ResultModel.belongsTo(Calculation, { foreignKey: "calculationId"});**
**}** 
```

在这个列表中，更改仍然被加粗标记，而列表中省略的文件部分不受此示例的影响。

# 你在哪里可以找到示例代码？

你可以从[`github.com/PacktPublishing/Mastering-Node.js-Web-Development`](https://github.com/PacktPublishing/Mastering-Node.js-Web-Development)下载本书所有章节的示例项目。我们还有其他来自我们丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 如果你跟随示例有困难怎么办？

首先要做的是回到章节的开始，重新开始。大多数问题都是由跳过步骤或没有完全应用列表中显示的更改造成的。请密切关注代码列表中的加粗强调，它突出了所需的更改。

然后，检查包含在书 GitHub 仓库中的勘误表/更正列表。技术书籍很复杂，尽管我尽了最大努力以及编辑们的努力，错误是不可避免的。检查勘误表以获取已知错误列表和解决它们的说明。

如果你仍然有问题，那么从书的 GitHub 仓库中下载你正在阅读的章节的项目([`github.com/PacktPublishing/Mastering-Node.js-Web-Development`](https://github.com/PacktPublishing/Mastering-Node.js-Web-Development))，并将其与你的项目进行比较。我是通过逐章工作来创建 GitHub 仓库中的代码的，所以你应该在你的项目中拥有相同文件和相同内容的文件。

如果您仍然无法使示例工作，那么您可以通过 adam@adam-freeman.com 联系我寻求帮助。请在您的邮件中清楚地说明您正在阅读哪本书，以及哪个章节/示例引起了问题。页码或代码列表总是有帮助的。请记住，我收到很多邮件，我可能不会立即回复。

# 如果您在书中发现错误怎么办？

您可以通过电子邮件在 adam@adam-freeman.com 报告错误，或者访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，点击 **Submit Errata** 并填写表格，尽管我要求您首先检查这本书的勘误表/更正列表，您可以在书的 GitHub 仓库中找到它，网址为 [`github.com/PacktPublishing/Mastering-Node.js-Web-Development`](https://github.com/PacktPublishing/Mastering-Node.js-Web-Development)，以防它已经被报告。

我将可能让读者困惑的错误，特别是示例代码中的问题，添加到 GitHub 仓库中的勘误表/更正文件中，并对第一个报告错误的第一位读者表示感激。我还发布了一个不太严重的问题列表，这通常意味着代码周围的文本错误，不太可能引起混淆。

# 如何联系作者？

您可以通过 adam@adam-freeman.com 发送邮件给我。自从我在书中开始发布电子邮件地址以来，已经有几年时间了。我并不完全确定这是一个好主意，但我很高兴我这么做了。我收到了来自世界各地的邮件，来自各行各业工作或学习的人们，而且——至少大部分情况下——这些邮件都是积极的、礼貌的，并且收到它们是一种乐趣。

我尽量迅速回复，但我收到很多邮件，有时我会积压，尤其是在我埋头写作新书的时候。我总是试图帮助那些在书中遇到示例问题的读者，尽管我在联系我之前要求您遵循本章前面描述的步骤。

虽然我欢迎读者的邮件，但有一些常见问题，答案总是“不”。我恐怕我不会为您的初创公司编写代码，帮助您完成大学作业，参与您的发展团队的设计争议，或者教您如何编程。

# 如果您真的喜欢这本书怎么办？

请通过 adam@adam-freeman.com 发送邮件并告知我。收到快乐读者的来信总是令人高兴，我感激您花时间发送这些邮件。写这些书可能会有困难，而这些邮件为坚持这项有时可能感觉不可能的活动提供了至关重要的动力。或者，您也可以将反馈发送到 feedback@packtpub.com。

# 如果这本书让您生气了怎么办？

您仍然可以通过 adam@adam-freeman.com 发送电子邮件，我仍然会尽力帮助您。请记住，只有当您解释了问题以及您希望我做什么时，我才能帮助您。您应该理解，有时，唯一的结局是接受我不是您的作者，并且只有当您归还此书并选择另一本书时，我们才能达成和解。我会仔细思考让您不高兴的原因，但经过 25 年的书籍写作，我已经接受并非每个人都喜欢阅读我喜欢写的书籍。或者，您也可以通过 feedback@packtpub.com 发送反馈。

现在您已经了解了本书的结构，下一章将探讨用于 Node.js 开发的工具。

# 下载此书的免费 PDF 副本

感谢您购买此书！

您喜欢在路上阅读，但无法携带您的印刷书籍到处走吗？

您选择的设备是否与您的电子书购买不兼容？

不必担心，现在每购买一本 Packt 书籍，您都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何设备上阅读。直接从您最喜欢的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

优惠远不止于此，您还可以获得独家折扣、时事通讯和每天收件箱中的优质免费内容。

按照以下简单步骤获取这些好处：

1.  扫描下面的二维码或访问以下链接：

![](img/B21959_Free_PDF_QR.png)

[`packt.link/free-ebook/9781804615072`](https://packt.link/free-ebook/9781804615072)

1.  提交您的购买证明。

1.  就这些！我们将直接将您的免费 PDF 和其他福利发送到您的电子邮件。

# 分享您的想法

一旦您阅读了《精通 Node.js Web 开发》，我们很乐意听听您的想法！请[点击此处直接访问此书的亚马逊评论页面](https://www.packtpub.com/)并分享您的反馈。

您的评论对我们和科技社区非常重要，并将帮助我们确保我们提供高质量的内容。
