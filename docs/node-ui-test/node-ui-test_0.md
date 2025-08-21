# 前言

自动化用户界面测试一直是编程的圣杯。现在，使用 Zombie.js 和 Mocha，您可以快速创建和运行测试，使得即使是最小的更改也可以轻松测试。增强您对代码的信心，并在开发过程中最小化使用真实浏览器的次数。

*使用 Node.js 进行 UI 测试*是一本关于如何自动测试您的 Web 应用程序，使其坚如磐石且无 bug 的快速而全面的指南。您将学习如何模拟复杂的用户行为并验证应用程序的正确行为。

您将在 Node.js 中创建一个使用复杂用户交互和 AJAX 的 Web 应用程序；在本书结束时，您将能够从命令行完全测试它。然后，您将开始使用 Mocha 作为框架和 Zombie.js 作为无头浏览器为该应用程序创建用户界面测试。

您还将逐模块创建完整的测试套件，测试简单和复杂的用户交互。

# 本书涵盖内容

第一章 *开始使用 Zombie.js*，帮助您了解 Zombie.js 的工作原理以及可以使用它测试哪些类型的应用程序。

第二章 *创建简单的 Web 应用*，解释了如何使用 Node.js、CouchDB 和 Flatiron.js 创建一个简单的 Web 应用。

第三章 *安装 Zombie.js 和 Mocha*，教您如何使用 Zombie.js 和 Mocha 为 Web 应用程序创建测试环境的基本结构。

第四章 *理解 Mocha*，帮助您了解如何使用 Mocha 创建和运行异步测试。

第五章 *操作 Zombie 浏览器*，解释了如何使用 Zombie.js 创建一个模拟浏览器，可以加载 HTML 文档并对其执行操作。

第六章 *测试交互*，解释了如何在文档中触发事件以及如何测试文档操作的结果。

第七章 *调试*，教会您如何使用 Zombie 浏览器对象和其他一些技术来检查应用程序的内部状态。

第八章 *测试 AJAX*，不包含在本书中，但可以通过以下链接免费下载：

[`www.packtpub.com/sites/default/files/downloads/0526_8_testingajax.pdf`](http://www.packtpub.com/sites/default/files/downloads/0526_8_testingajax.pdf)

# 本书所需内容

要使用本书，您需要一台运行现代主流操作系统（如 Windows、Mac 或 Linux）的个人电脑。

# 本书适合谁

本书适用于使用并在一定程度上了解 JavaScript 的程序员，尤其是具有事件驱动编程经验的人。例如，如果您曾在网页上使用 JavaScript 设置事件回调和进行 AJAX 调用，您将会更容易上手。另外，一些使用 Node.js 的经验也会减轻学习曲线，但不是绝对要求。

# 约定

在本书中，您将找到一些文本样式，用于区分不同类型的信息。以下是一些这些样式的示例，以及它们的含义解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄均显示如下：“要从 Node 中访问 CouchDB 数据库，您将使用一个名为`nano`的库。”

代码块设置如下：

```js
browser.visit('http://localhost:8080/form', function() {
  browser
    .fill('Name', 'Pedro Teixeira')
    .select('Born', '1975')
    .check('Agree with terms and conditions')
    .pressButton('Submit', function() {
      assert.equal(browser.location.pathname, '/success');
      assert.equal(browser.text('#message'),
        'Thank you for submitting this form!');
    });
});
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```js
  "scripts": {
 "test": "mocha test/users.js",
    "start": "node app.js"
  },...
```

任何命令行输入或输出均按以下格式编写：

```js
$ npm install
...
mocha@1.4.2 node_modules/mocha
...

zombie@1.4.1 node_modules/zombie
...
```

**新术语**和**重要单词**会以粗体显示。屏幕上看到的单词，比如菜单或对话框中的单词，会以这样的方式出现在文本中："点击**下一步**按钮会将您移动到下一个屏幕"。

### 注意

警告或重要提示会以这样的方式出现在一个框中。

### 提示

提示和技巧会以这样的方式出现。
