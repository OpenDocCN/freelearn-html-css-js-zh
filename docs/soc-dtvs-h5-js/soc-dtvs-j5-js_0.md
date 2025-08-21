# 序言

世界是一个有趣的地方，对此毫无疑问。我们通过感官体验世界，这些感官收集数据供我们的大脑处理。世界经常是混乱的，我们需要深入思考才能理解事物。为了简化这个过程，数据可以转换成更容易理解的其他形式。

本书是关于将数据塑造成更易理解的形式。它是关于利用我们这个时代最丰富的数据源——社交网络，并将它们的大量数据转换成可理解的形式。为此，我们使用了最新的 HTML 和 JavaScript。

# 本书涵盖内容

第一章，*数据可视化*，向我们介绍了一个充满不断增长数据集的世界。它还讨论了如何使用可视化作为我们的独木舟来驾驭这条数据之河。

第二章，*JavaScript 和 HTML5 用于可视化*，探讨了 HTML 和 JavaScript 中的新特性，这些特性为数据可视化提供了机会。它讨论了画布和可缩放矢量图形。

第三章，*OAuth*，探讨了常常令人困惑的 OAuth 技术，并展示了如何将其用于将权限委托给我们的应用程序。这是因为社交网站上的大量数据是私有的，我们可能需要这些数据。

第四章，*JavaScript 用于可视化*，介绍了 Raphaël.js 和 d3.js，这两个都是非常出色的 JavaScript 库，可以减少手动构建可视化的痛苦，否则这是一个耗时且容易出错的任务。

第五章，*Twitter*，探讨了如何从 Twitter 检索数据并使用它来构建可视化。

第六章，*Stack Overflow*，探讨了如何检索流行的 Stack Overflow 的数据 API，它提供了一些令人着迷的可视化机会，可以用来创建交互式图表。

第七章，*Facebook*，探讨了 Facebook JavaScript API 以及如何使用它来检索数据作为我们下一个可视化的基础。在我心中，原始的、仍然是世界上最大的社交媒体网络就是 Facebook。

第八章，*Google+*，探讨了谷歌最新的社交媒体尝试以及如何检索数据来创建力导向图。

# 本书所需准备

使用这本书中的示例和代码所需的工具非常少。你需要安装 node.js ([`node.org`](http://node.org))，这在 第五章 *Twitter* 中有所介绍。你想要下载 d3.js ([`d3js.org`](http://d3js.org))，jQuery ([`jquery.com`](http://jquery.com))，和 Raphael.js ([`raphaeljs.com/`](http://raphaeljs.com/))。所有的演示可以在任何现代的网页浏览器中查看。代码已经针对 Chrome 进行测试，但应该在 FireFox、Opera 上也能工作，甚至包括 Internet Explorer。

# 这本书适合谁

这本书适合所有对数据感到兴奋并希望与他人分享这种兴奋的人。任何对可以从社交网络提取的数据感兴趣的人也会发现这本书很有趣。读者应该具备 JavaScript 和 HTML 的基本工作知识。jQuery 在整本书中都被多次使用，所以读者最好熟悉这个库的基本知识。对 node.js 有一些了解会有帮助，但不是必需的。

# 约定

在这本书中，你会发现有许多种文本样式来区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词汇如下所示：“一旦权限被分配，Facebook 会将用户重定向回你的 `redirect_uri`，允许你利用令牌来查询 Facebook API。”

代码块如下所示：

```js
OAuth.initialize('<Your Public key>');
OAuth.redirect('facebook', "callback/url");
```

当我们希望吸引你的注意力到代码块的某个特定部分时，相关的行或项目会被设置为粗体：

```js
function onSignInCallback(authResult) {
      gapi.client.load('plus','v1', function(){
        if (authResult['access_token']) {
          $('#gConnect').hide();
                           retrieveFriends();
        } else if (authResult['error']) {
          console.log('There was an error: ' + authResult['error']);
          $('#gConnect').show();
        }
        console.log('authResult', authResult);
      });
    }
```

**新术语**和**重要词汇**以粗体显示。例如，您在屏幕上、菜单或对话框中看到的词汇，在文本中如下所示：“这可以通过点击 **API Access** 选项卡中的 **Create an OAuth 2.0 client ID…** 来完成。”

### 注意

警告或重要说明以这样的盒子形式出现。

### 技巧

技巧和窍门就像这样出现。

# 读者反馈

我们对读者的反馈总是开放的。让我们知道你对这本书的看法——你喜欢什么或者可能不喜欢什么。读者反馈对我们开发您真正能从中获得最多收益的标题非常重要。

要发送给我们一般性反馈，只需发送一封电子邮件到 `<feedback@packtpub.com>`，并通过消息主题提及书名。

如果你在某个话题上有专业知识，并且有兴趣撰写或贡献一本书，请查看我们在 [www.packtpub.com/authors](http://www.packtpub.com/authors) 上的作者指南。

# 客户支持

现在你拥有了一本 Packt 出版的书籍，我们有许多资源可以帮助你充分利用你的购买。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户上下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了此书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 错误

虽然我们已经竭尽全力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——我们将非常感谢您能向我们报告。这样做可以节省其他读者的挫折感，并帮助我们改进本书的后续版本。如果您发现任何错误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误提交****表单**链接，并输入您的错误详情。一旦您的错误得到验证，您的提交将被接受，错误将会上传到我们的网站，或添加到该标题下的现有错误列表中。您可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看任何现有的错误。

## 盗版

互联网上的版权材料盗版是一个持续存在的问题，涵盖所有媒体。 Packt 对此非常重视，如果您在互联网上以任何形式发现我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

如果您发现有疑似侵犯版权的材料，请通过`<copyright@packtpub.com>`联系我们。

我们感谢您在保护我们的作者和为我们提供有价值内容方面所提供的帮助。

## 问题

如果您在阅读本书的过程中遇到任何问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽最大努力解决问题。
