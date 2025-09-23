# 前言

这本快速入门指南是一本涵盖与 TypeScript 相关所有主题的书籍。本书将引导读者了解使用 TypeScript 的所有功能和好处。通过这本指南，你将提升你的类型化技能，这将使你的 JavaScript 知识得到应用。TypeScript 是 JavaScript 的超集。本书不会教你如何进行 Web 编程，而是说明如何在已知的并喜爱的语言之上使用 TypeScript。

# 本书面向对象

本书面向那些了解 JavaScript 但从未接触过 TypeScript 的开发者。

# 本书涵盖内容

第一章，*TypeScript 入门*，涵盖了如何将 TypeScript 引入新项目或现有 JavaScript 项目。我们将看到如何在当前的构建设置中设置你的环境。TypeScript 被各种构建工具支持，如 Grunt、Gulp、Webpack，或者简单地使用 CLI。我们还深入探讨了所有可用选项中的基本选项，以帮助你开始使用 TypeScript。

第二章，*关于使用原始类型进行类型注册*，阐述了变量的作用域、未定义变量和 null 变量之间的微妙区别，以及如何使变量成为可选或必需的。在本章结束时，读者将处于所有变量都正确声明并带有 TypeScript 准确支持的类型的情境中。原始类型和非原始类型之间的区别将不再是难题。使用枚举或符号将变得自然，每当在系统中引入新的领域对象时，创建新类型将变成一种习惯。

第三章，*利用对象释放类型的力量*，说明了对象、Object、对象字面量和使用构造函数创建的对象之间的区别。本章还介绍了类型联合的概念，它将允许单个值有无限类型的组合。此外，还引入了交集的概念，它允许我们以不同的方式操作类型。在本章结束时，读者将能够创建包含复杂结构的对象的复杂组合。我们将剖析如何创建具有强类型索引签名的字典，理解类型如何通过映射带来好处，以及如何使用正确的对象来尽可能精确地定义具有广泛影响的对象。

第四章，*在面向对象中转换您的代码*，解释了面向对象编程有其自己的术语集，TypeScript 依赖于其中许多。在本章中，我们将通过示例讨论 TypeScript 支持的面向对象编程的所有概念。我们将看到类是什么以及如何将类实例化为对象。同样，我们将看到如何使用 TypeScript 将构造函数强类型化，但也将看到如何使用简写语法直接从构造函数分配类的字段。封装的可见性原则、如何实现接口以及如何将抽象引入类都得到了涵盖。

第五章，*使用不同模式作用域变量*，涵盖了最基本的概念的微妙之处，即变量。从原始类型到对象的精确类型对于访问特定成员至关重要。在运行时和设计时精确作用域类型对于两个环境之间的一致性和获得关于可能或不可能的反馈至关重要。不同类型变量之间的配置多样性需要本章涵盖的许多不同模式。

第六章，*通过泛型重用代码*，基于前几章介绍的思想。本章通过增强类型并使其泛型化来构建这些思想。基本主题，如定义泛型类和接口，是存在的。通过本章，我们将进入更高级的主题，如泛型约束。本章的目标是使您的代码更加泛型，以增加类、函数和结构的可重用性，从而减少代码复制的负担。

第七章，*掌握定义类型的艺术*，涵盖了如何从我们未直接工作的库中创建类型，而是在 TypeScript 项目中导入库。主要区别在于，当使用项目外部的代码时，您不会直接使用 TypeScript 代码，而是使用它们的定义。原因是那些库中提供的是 JavaScript 代码，而不是 TypeScript 代码。我们将看到如何掌握为不提供它们的代码创建定义文件的艺术，让您能够在强大的环境中继续工作。

# 为了充分利用这本书

这本书是为想要开始使用 TypeScript 构建应用的 JavaScript 开发者准备的。不需要对 TypeScript 有任何先前的知识。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在 [www.packtpub.com](http://www.packtpub.com/support) 登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Typescript-3.0-Quick-Start-Guide`](https://github.com/PacktPublishing/Typescript-3.0-Quick-Start-Guide)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包可供下载，网址为**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。查看它们！

# 下载彩色图像

我们还提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/TypeScript3QuickStartGuide_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/TypeScript3QuickStartGuide_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“将下载的`WebStorm-10*.dmg`磁盘映像文件作为系统中的另一个磁盘挂载。”

代码块设置如下：

```js
let a:number = 2;
a = "two"; // Doesn't compile
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```js
function switchFunction(num: number) {
   let b: string = "functionb";

   switch (num) {
       case 1:
         let b: string = "case 1";
       break;
   }
}
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要说明看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请通过`feedback@packtpub.com`发送电子邮件，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`发送电子邮件。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将非常感激您能向我们报告。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，我们将非常感激您能提供位置地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/)。
