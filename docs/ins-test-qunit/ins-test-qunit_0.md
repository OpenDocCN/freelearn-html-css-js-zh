# 前言

JavaScript 大约 20 年前首次发布。多年来，它并没有被广大观众认真对待。今天，它是一种成熟（也是最流行）的编程语言。无论为 Web 做些什么，以一种或另一种方式，都依赖于 JavaScript。此外，随着 HTML5 相关技术的发展，开发重点正在进一步转向富互联网应用程序架构。这意味着基于 JavaScript 的复杂可扩展解决方案将进一步扩展。服务器端 JavaScript 正在迅速赢得其动力，从而为 JavaScript 的自动化测试提供了便利。

QUnit 是一个非常易于使用的 JavaScript 测试框架，同时它也非常强大和灵活，是一个对于任何规模项目的单元测试和功能测试的良好选择。

本书提供了一个逐步掌握 QUnit 的指南。它展示了如何设置和使用 QUnit，如何使用 QUnit 进行跨浏览器测试，以及如何在持续集成工具的配合下从 QUnit 中获益。

本书的目标是帮助读者在短时间内充分利用 QUnit。

# 本书涵盖的内容

设置 QUnit（简单）章节解释了如何搭建一个测试环境以及如何熟悉 QUnit 框架。

测试断言（简单）章节描述了一个关于 QUnit 断言方法以及流行的 QUnit 断言插件的教程。

编写自定义断言插件（高级）章节解释了如何利用 QUnit 回调来编写自定义断言插件。

测试异常（中等）章节描述了 JavaScript 中的自定义异常以及如何断言异常行为。

测试异步调用（中等）章节描述了如何对异步回调进行断言以及在单元测试中模拟 XHR。

组织测试用例（简单）章节探讨了 QUnit 的功能，以逻辑方式组织测试。

使用共享设置（中等）章节描述了一种在测试组之间共享任务的方法，并在测试执行之间保持测试环境的清洁。

测试用户行为（中等）章节解释了通过模拟终端用户行为并对预期行为进行断言，对一个简单的计算器应用程序进行功能测试。

使用 QUnit 在控制台运行（高级）章节解释了如何使用无头浏览器 PhantomJS 在命令行运行 QUnit 测试。它还解释了如何将 QUnit 测试任务分配给 Grunt，并使用 Travis CI 自动化测试。

跨浏览器分布式测试（高级）章节解释了如何使用命令行工具 Bunyip 自动化客户端跨浏览器测试。

构建 Web 项目（高级）章节解释了如何将 QUnit 测试任务添加到项目构建脚本中。

QUnit 和 CI - 设置 Jenkins（高级）章节解释了如何设置 Jenkins 持续集成服务器来自动化 QUnit 测试。

# 本书适合的对象

这本书对于任何正在使用 JavaScript 并寻求一个强大但易于使用的测试框架的开发者来说都会很有帮助。

读者不需要了解任何特定的框架，但至少需要了解 JavaScript 和 HTML 的基本原理。

如果读者有任何自动化测试背景，这本书将更加有益。

# 约定

在这本书中，你会发现有多种文本样式，用以区分不同类型的信息。以下是这些样式的几个示例及其含义的解释。

文本中的代码词汇如下所示显示：“在项目工作目录的根目录下将`build.xml`设置为构建脚本。”

代码块如下所示设置：

```js
<?xml version="1.0"?>
<!DOCTYPE project>
<project name="tree"  basedir="." default="build">
<target name="build" description="runs QUnit tests using PhantomJS">
  <!-- Clean up output directory -->
  <delete dir="./build/qunit/"/>  
  <mkdir dir="./build/qunit/"/>  
  <!-- QUnit Javascript Unit Tests -->
  <echo message="Executing QUnit Javascript Unit Tests..."/>

</target>
</project>
```

当我们希望吸引您注意代码块中的某个特定部分时，相关行或项目以粗体显示：

```js
utils.subscribe( global, "load", function() {
             var calc =  document.getElementById("calc"),
                 num1 = document.getElementById("num1"),
                 num2 = document.getElementById("num2"),
                 sum = document.getElementById("sum");

 QUnit.test( "Test that calc sums up typed in numbers by click", function( assert ){
 num1.value = "5";
 num2.value = "7";
 utils.trigger( calc, "click" );
 assert.strictEqual( sum.value, "12" );
 });
 });

```

任何命令行输入或输出如下所示书写：

```js
phantomjs runner.js test-suite.html

```

**新术语**和**重要词汇**以粗体显示。例如，您在屏幕上、菜单或对话框中看到的词汇，在文本中像这样出现：“进入 Jenkins 仪表板页面，然后点击**添加新作业**链接。”

### 注意

警告或重要说明出现在像这样的盒子中。

### 技巧

技巧和小贴士像这样出现。

# 读者反馈

来自读者的反馈总是受欢迎的。让我们知道您对这本书的看法——您喜欢或不喜欢的部分。读者反馈对我们来说非常重要，以开发您真正能从中获得最大收益的标题。

要想给我们发送一般性反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在消息主题中提到书名。

如果您在某个主题上有专业知识，并且有兴趣撰写或贡献一本书，请查看我们在[www.packtpub.com/authors](http://www.packtpub.com/authors)上的作者指南。

# 客户支持

既然您已经成为 Packt 书籍的骄傲拥有者，我们有很多东西可以帮助您充分利用您的购买。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)账户上下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了此书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册以便将文件直接通过电子邮件发送给您。

## 错误

尽管我们已经尽一切努力确保内容的准确性，但错误确实会发生。如果您在我们的某本书中发现了一个错误——可能是文本或代码中的错误——我们将非常感谢您能向我们报告。这样做，您可以节省其他读者不必要的挫折，并帮助我们改进本书的后续版本。如果您发现任何错误，请通过访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，选择您的书籍，点击**错误提交****表单**链接，并输入您错误的详细信息。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站，或添加到该标题的错误部分现有的错误列表中。

## 盗版

互联网上的版权材料盗版是一个持续存在的问题，涉及所有媒体。 Packt 非常重视我们版权和许可的保护。如果您在互联网上发现我们作品的任何非法副本，无论以何种形式，请立即提供位置地址或网站名称，以便我们可以寻求解决方案。

如有怀疑的侵权材料，请通过`<copyright@packtpub.com>`联系我们。

感谢您在保护我们的作者和我们为您提供有价值内容的能力方面所提供的帮助。

## 问题

如果您在阅读本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们会尽力解决。
