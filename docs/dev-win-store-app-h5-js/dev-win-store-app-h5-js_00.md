# 序言

*使用 HTML5 和 JavaScript 开发 Windows Store 应用* 是一本实践性强的指南，涵盖了 Windows Store 应用的基本重要特性以及示例代码，向您展示如何开发这些特性，同时学习 HTML5 和 CSS3 中的新特性，使您能够充分利用您的网页开发技能。

# 本书内容覆盖范围

第一章, *HTML5 结构*, 介绍了新 HTML5 规范中的语义元素、媒体元素、表单元素和自定义数据属性。

第二章, *使用 CSS3 进行样式设计*, 介绍了 CSS3 在开发使用 JavaScript 的 Windows Store 应用时会频繁用到的增强和特性。本章涵盖了以下主题：CSS3 选择器、网格和弹性盒布局、动画和转换、以及媒体查询。

第三章, *Windows 应用的 JavaScript*, 介绍了 JavaScript 的 Windows 库及其特性，并突出显示了用于开发应用的命名空间和控件。

第四章, *使用 JavaScript 开发应用*, 介绍了开始使用 JavaScript 开发 Windows 8 应用所需的工具和提供的模板。

第五章, *将数据绑定到应用*, 描述了如何在应用中实现数据绑定。

第六章, *使应用响应式*, 描述了如何使应用响应式，以便它能处理不同屏幕尺寸和视图状态的变化，并响应缩放。

第七章, *使用磁贴和通知使应用活跃*, 描述了应用磁贴和通知的概念，以及如何为应用创建一个简单的通知。

第八章, *用户登录*, 描述了 Live Connect API 以及如何将应用与该 API 集成以实现用户认证、登录和检索用户资料信息。

第九章, *添加菜单和命令*, 描述了应用栏、它如何工作以及它在应用中的位置。此外，我们还将学习如何声明应用栏并向其添加控件。

第十章, *打包和发布*, 介绍了我们将如何了解商店并学习如何使应用经历所有阶段最终完成发布。同时，我们还将了解如何在 Visual Studio 中与商店进行交互。

第十一章，*使用 XAML 开发应用*，描述了其他可供开发者使用的平台和编程语言。我们还将涵盖使用 XAML/C#创建应用程序的基本知识。

# 本书需要你具备的知识

为了实施本书中将要学习的内容并开始开发 Windows Store 应用，你首先需要 Windows 8。此外，你还需要以下开发工具和工具包：

+   微软 Visual Studio Express 2012 用于 Windows 8 是构建 Windows 应用的工具。它包括 Windows 8 SDK、Visual Studio 的 Blend 和项目模板。

+   Windows App Certification Kit

+   Live SDK

# 本书适合谁

这本书适合所有想开始为 Windows 8 创建应用的开发者。此外，它针对想介绍 standards-based web technology with HTML5 和 CSS3 的进步的开发者。另外，这本书针对想利用他们在 Web 开发中的现有技能和代码资产，并将其转向为 Windows Store 构建 JavaScript 应用的 Web 开发者。简而言之，这本书适合所有想学习 Windows Store 应用开发基础的人。

# 约定

在这本书中，你会发现有几种不同信息类型的文本样式。以下是这些样式的一些示例及其含义的解释。

文本中的代码词汇如下所示：“`createGrouped`方法在列表上创建一个分组投影，并接受三个函数参数。”

代码块如下所示：

```js
// Get the group key that an item belongs to.
  function getGroupKey(dataItem) {
  return dataItem.name.toUpperCase().charAt(0);   
}

// Get a title for a group
  function getGroupData(dataItem) {
  return {
    title: dataItem.name.toUpperCase().charAt(0);
  }; 
}
```

**新术语**和**重要词汇**以粗体显示。例如，你在屏幕上看到的、菜单或对话框中的单词，会在文本中以这种方式出现：“你将能够为应用程序 UI 设置选项；这些选项之一是支持的旋转。”

### 注意

警告或重要说明以这种方式出现在盒子里。

### 技巧

技巧和小窍门像这样出现。

# 读者反馈

读者对我们的书籍的反馈总是受欢迎的。告诉我们你对这本书的看法——你喜欢或可能不喜欢的地方。读者反馈对我们开发您真正能从中获得最大收益的标题非常重要。

要发送给我们一般性反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在消息主题中提及书籍标题。

如果你在某个主题上有专业知识，并且你对编写或贡献书籍感兴趣，请查看我们网站上的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然你已经成为 Packt 书籍的骄傲拥有者，我们有很多东西可以帮助你充分利用你的购买。

## 勘误

尽管我们已经竭尽全力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现任何错误——可能是文本或代码中的错误——我们非常感谢您能向我们报告。这样做可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何错误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详情。一旦您的错误得到验证，您的提交将被接受，并且错误将被上传到我们的网站或添加到该标题下的错误列表中。您可以通过[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题查看现有的错误。

## 版权侵犯

互联网上版权材料的侵犯是一个持续存在的问题，涵盖所有媒体。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上以任何形式发现我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求解决方案。

如果您发现任何可疑的版权侵犯材料，请通过`<copyright@packtpub.com>`联系我们并提供链接。

我们非常感谢您在保护我们的作者和我们提供有价值内容的能力方面所提供的帮助。

## 问题反馈

如果您在阅读本书的过程中遇到任何问题，可以通过`<questions@packtpub.com>`联系我们，我们会尽最大努力为您解决问题。
