# 前言

JavaScript 是一种快速发展的语言，每年都会添加新功能。本书旨在通过使用 JavaScript 构建各种应用程序来让您动手实践。这将帮助您在 JavaScript 上建立坚实的基础，从而有助于您适应其未来的新功能，以及学习其他现代框架和库。

# 本书涵盖内容

第一章，*构建 ToDo 列表*，从简单的 DOM 操作开始，使用 JavaScript 和事件监听器，这将让您对 JavaScript 如何与网站中的 HTML 进行交互有一个很好的理解。您将设置基本的开发环境并构建您的第一个 ToDo 列表应用程序。

第二章，*构建 Meme Creator*，帮助您构建一个有趣的应用程序 Meme Creator。通过这个，您将了解画布元素，使用 ES6 类，并介绍使用 CSS3 flexbox 进行布局。本章还向您介绍了 Webpack，并使用它设置自己的自动化开发环境。

第三章，*事件注册应用程序*，专注于开发具有适当表单验证的响应式事件注册表单，允许用户注册您即将举办的活动，并通过图表直观地显示注册数据。本章将帮助您了解执行 AJAX 请求的不同方法以及如何处理动态数据。

第四章，*使用 WebRTC 构建实时视频通话应用程序*，使用 WebRTC 在 JavaScript 中构建实时视频通话和聊天应用程序。本章重点介绍了在浏览器中使用 JavaScript 可用的强大 Web API。

第五章，*开发天气小部件*，帮助您使用 HTML5 自定义元素为应用程序构建天气小部件。您将了解 Web 组件及其在 Web 应用程序开发中的重要性。

第六章，*使用 React 构建博客*，讨论了由 Facebook 创建的用于在 JavaScript 中构建用户界面的库 React。然后，您将使用 React 和诸如`create-react-app`和`react-router`之类的工具构建博客。

第七章，*Redux*，将深入探讨使用 Redux 管理 React 组件之间的数据，从而使您的博客更易于维护和扩展，并提供更好的用户体验。

# 您需要为本书做好准备

为了在本书中构建项目时获得最佳体验，您将需要以下内容：

+   至少 4GB RAM 内存的 Windows 或 Linux 机器，或 Mac

+   iPhone 或 Android 移动设备

+   快速的互联网连接

# 本书适合谁

本书适用于具有 HTML、CSS 和 JavaScript 基础知识的 Web 开发人员，他们希望提高自己的技能并构建强大的 Web 应用程序。

具有 JavaScript 或任何其他编程语言的基础知识将会很有帮助。但是，如果您完全不了解 JavaScript 和编程，那么您可以阅读以下简单的教程之一，这将帮助您开始学习 JavaScript 的基础知识，然后您就可以立即阅读本书了：

+   Mozilla 开发者网络：[`developer.mozilla.org/en-US/docs/Learn/JavaScript/First_steps/What_is_JavaScript`](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/First_steps/What_is_JavaScript)

+   w3schools：[`www.w3schools.com/js/`](https://www.w3schools.com/js/)

# 惯例

在本书中，您会发现一些区分不同类型信息的文本样式。以下是一些示例以及它们的含义解释。文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名显示如下：“在我们的`index.html`文件中，我们的`<body>`元素被分成一个导航栏和包含网站内容的`div`。”

代码块设置如下：

```js
loadTasks() {
  let tasksHtml = this.tasks.reduce((html, task, index) => html +=  
  this.generateTaskHtml(task, index), '');
  document.getElementById('taskList').innerHTML = tasksHtml;
 }
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目会以粗体显示：

```js
function mapStateToProps() {
  return {
    // No states needed by App Component
  };
}
```

任何命令行输入或输出都以以下方式编写：

```js
npm install -g http-server
```

**新术语**和**重要单词**以粗体显示。

屏幕上看到的单词，例如菜单或对话框中的单词，会以这样的方式出现在文本中：“单击主页上的“阅读更多”按钮将立即带您到帖子详情页面。”

警告或重要提示会以这种方式出现。提示和技巧会以这种方式出现。
