# 前言

JavaScript 正逐渐成为世界上最流行的语言之一。然而，它作为一种玩具语言的历史意味着开发人员往往忽视良好的设计。设计模式是一个很好的工具，可以提出一些经过验证的解决方案。

# 本书内容

本书分为两个主要部分，每个部分包含多个章节。本书的第一部分，我们将称之为*第一部分*，涵盖了 GoF 书中的经典设计模式。

第一章，*Designing for Fun and Profit*，介绍了设计模式是什么，以及为什么我们有兴趣使用设计模式。我们还将谈谈 JavaScript 的一些历史，以便让您了解历史背景。

第二章，*Organizing Code*，探讨了如何创建用于组织代码的经典结构，如命名空间、模块和类，因为 JavaScript 缺乏这些构造作为一等公民。

第三章，*Creational Patterns*，涵盖了《四人组模式》书中概述的创建模式。我们将讨论这些模式如何适用于 JavaScript，而不是《四人组》书写作时流行的语言。

第四章，*Structural Patterns*，研究了创建模式。我们将检查《四人组模式》书中的结构模式。

第五章，*Behavioral Patterns*，讨论了行为模式。这些是《四人组模式》书中我们将研究的最后一组模式。这些模式控制了将类连接在一起的不同方式。

*第二部分*着眼于 GoF 书中未涵盖或特定于 JavaScript 的模式。

第六章，*Functional Programming*，涵盖了函数式编程语言中的一些模式。我们将看看如何在 JavaScript 中使用这些模式来改进代码。

第七章，*Reactive Programming*，探讨了 JavaScript 中回调模型编程所涉及的问题。它提出了响应式编程，一种基于流的事件方法，作为可能的解决方案。

第八章，*Application Patterns*，研究了创建单页面应用程序的各种模式。我们将提供清晰度，并探讨如何使用使用每种现有模式的库，以及创建我们自己的轻量级框架。

第九章，*Web Patterns*，探讨了一些适用于 Web 应用程序的模式。我们还将研究一些关于将代码部署到远程运行时（如浏览器）的模式。

第十章，*Messaging Patterns*，涵盖了消息传递是一种强大的通信技术，可在应用程序内部甚至之间进行通信。在本章中，我们将研究一些关于消息传递的常见结构，并讨论为什么消息传递如此有用。

第十一章，*Microservices*，涵盖了微服务，这是一种以惊人的速度增长的方法。本章探讨了这种编程方法背后的思想，并建议在使用这种方法构建时要牢记的一些模式。

第十二章，*Patterns for Testing*，讨论了构建软件的困难，以及构建优质软件的双重困难。本章提供了一些模式，可以使测试过程变得更容易一些。

第十三章 , *高级模式* , 解释了一些模式，比如面向方面的编程在 JavaScript 中很少被应用。我们将探讨这些模式如何在 JavaScript 中应用以及是否应该应用它们。

第十四章 , *ECMAScript-2015/2016 今日解决方案* , 涵盖了一些工具，允许您在今天使用未来版本 JavaScript 的功能。我们将研究微软的 TypeScript 以及 Traceur。

# 本书所需内容

本书不需要专门的软件。JavaScript 可以在所有现代浏览器上运行。有用于驱动各种工具的独立 JavaScript 引擎，如 C++（V8）和 Java（Rhino），这些工具包括 Node.js，CouchDB，甚至 Elasticsearch。这些模式可以应用于这些技术中的任何一个。

# 本书适合对象

本书非常适合希望在 JavaScript 中获得面向对象编程专业知识和 ES-2015 新功能以提高网页开发技能并构建专业质量网页应用的开发者。

# 约定

在本书中，您会发现一些区分不同信息种类的文本样式。以下是一些这些样式的示例和它们的含义解释。

文本中的代码词，数据库表名，文件夹名，文件名，文件扩展名，路径名，虚拟 URL，用户输入和 Twitter 句柄会以以下形式显示: "您会注意到我们明确定义了 `name` 字段。"

代码块设置如下:

```js
let Castle = function(name){
 this.name = name;
}
Castle.prototype.build = function(){ console.log(this.name);}
let instance1 = new Castle("Winterfell");
instance1.build();
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目会以粗体显示:

```js
let Castle = function(name){
 this.name = name;
}
 **Castle.prototype.build = function(){ console.log(this.name);}** 

let instance1 = new Castle("Winterfell");
instance1.build();
```

任何命令行输入或输出都以以下形式书写:

```js
ls -1| cut -d \. -f 2 -s | sort |uniq
```

**新术语**和**重要词汇**以粗体显示。屏幕上看到的词语，例如菜单或对话框中的词语，会以这样的形式出现在文本中: "要访问它们，有一个菜单项，位于**工具** | **Chrome 中的开发者工具** | **工具** | **Firefox 中的 Web 开发者** 下。"

### 注意

警告或重要提示会以这样的形式出现在一个框中。

### 提示

提示和技巧会出现在这样的形式中。
