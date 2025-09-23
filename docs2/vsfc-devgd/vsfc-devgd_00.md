# 前言

*Visualforce 开发者指南* 是一本实用的、动手操作的口袋指南，它提供了清晰简单的指导，用于在 Force.com 平台和移动应用程序上开发 Visualforce 页面。它还包含一个单一、连续的真实世界示例，并附有代码示例。本书以简单的方式解释了 Visualforce 的概念和技术方面。

*Visualforce 开发者指南* 涵盖了主要主题，从为 Force.com 平台开发 Visualforce 开始，继续到快速且无痛苦地为移动应用程序开发 Visualforce 页面。

Force.com 平台可以自动生成用户界面（标准页面），但在某些情况下，你可能需要构建一个更定制的 UI。Visualforce 允许开发者构建复杂和定制的用户界面，这些界面可以原生地托管在 Force.com 平台上。Visualforce 是一个包含类似于 HTML 的基于标签的标记语言的框架。

# 本书涵盖的内容

第一章, *Visualforce 入门*，解释了 MVC 模型和 Visualforce 架构。我们将介绍 Visualforce 页面。我们将了解 MVC 架构并讨论 Visualforce 页面。进一步地，我们将了解 Visualforce 页面的架构。我们将定义 Visualforce 页面的优势，并了解 Visualforce 开发工具。

第二章，*控制器和扩展*，介绍了可用于 Visualforce 页面的控制器和扩展类型。我们将了解控制器类型，并查看一些示例。

第三章, *Visualforce 和标准 Web 开发技术*，解释了如何使用 CSS、JavaScript、jQuery 和 HTML5 等标准 Web 开发技术来开发 Visualforce 页面。本章还包括了 CSS、JavaScript 和 jQuery 静态资源的用法。

第四章, *Visualforce 自定义组件*，概述了 Visualforce 自定义组件，并进一步解释了如何创建自定义组件以及自定义组件的用法。

第五章, *动态 Visualforce 绑定*，解释了与标准对象和自定义对象使用动态引用的用法，如何引用 Apex Maps 和 Lists，以及如何处理字段集。

第六章, *Visualforce 图表*，讨论了 Visualforce 图表，这是一个组件集合，它提供了一种简单直观的方式来在 Visualforce 页面和自定义组件中创建图表。

第七章，*移动端 Visualforce*，介绍了如何通过使用 Visualforce Mobile 将建立在 Force.com 平台上的应用程序扩展到移动设备。

第八章，*Visualforce 开发最佳实践*，解释了 Visualforce 和 Apex 开发的最佳实践。我们将探讨如何提高用户体验，并检查一些 Visualforce 开发的编码标准。

附录，*Apex 和 Visualforce 开发安全提示*，提供了一些 Visualforce 和 Apex 开发的安全提示。我们将探讨一些用于扫描代码以检查安全和质量的工具，并学习一些安全漏洞。

# 您需要这本书的物品

Force.com 平台的要求如下：

+   互联网/网站的基本知识

+   如果您有一个免费的 Developer Force 账户（如果没有，请访问：[`events.developerforce.com/signup`](https://events.developerforce.com/signup)）

+   Visualforce 的基本知识

# 这本书的适用对象

Visualforce 开发者指南不是 Visualforce 开发的完整参考或圣经。这是一本节省时间的口袋指南，包括 Visualforce 开发中最需要和常用的技术方面。因此，这本书适合对 Visualforce 有基本知识的 Force.com 开发者。

# 习惯用法

在这本书中，您会发现许多不同风格的文本，用于区分不同类型的信息。以下是一些这些风格的示例及其含义的解释。

文本中的代码词如下所示："我们可以通过使用`$Resource`全局变量而不是硬编码文档 ID，在页面标记中通过名称引用静态资源。"

代码块设置如下：

```js
<apex:page controller="TransientExampleController">
    Non Transient Date: {!t1} <br/>
    Transient Date    : {!t2} <br/>
    <apex:form >
        <apex:commandLink value="Refresh"/>
    </apex:form>
</apex:page>
```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的词，例如在菜单或对话框中，在文本中如下所示：

"组件参考"是探索不同组件及其用途的最佳场所。

### 注意

警告或重要注意事项如下所示。

### 小贴士

小技巧和窍门如下所示。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者的反馈对我们开发您真正能从中获得最大收益的标题非常重要。

要向我们发送一般反馈，只需发送一封电子邮件到`<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从 [`www.packtpub.com`](http://www.packtpub.com) 下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误清单提交表单**链接，并输入您的错误清单详情。一旦您的错误清单得到验证，您的提交将被接受，错误清单将被上传到我们的网站，或添加到该标题的“错误清单”部分。任何现有的错误清单都可以通过从 [`www.packtpub.com/support`](http://www.packtpub.com/support) 选择您的标题来查看。

## 盗版

在互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
