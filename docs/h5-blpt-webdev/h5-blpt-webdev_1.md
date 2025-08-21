# 第一章：在我们开始之前

当您开始一个新项目时，您会有多么高兴？我也是！新项目文件夹的气味非常令人兴奋。不幸的是，它很快就变成了一堆文件夹，子文件夹和匆忙编写的标记，然后你就知道了，这是发布日，你恐惧地意识到你的页面缺少一些基本的元数据（嗯，那些网站图标！），一些部分在某些浏览器中无法阅读——什么？打印时也需要好看？

HTML5 Boilerplate 的诞生是因为从头开始并错过了重要部分而感到沮丧。拥有一个清单并不像从一个已经带有您的清单所需文件的项目开始那么有用。

HTML5 Boilerplate 为您提供了最好的工具，让您可以开始下一个 Web 开发项目。

# HTML5 Boilerplate 的特点

在我们深入了解 HTML5 Boilerplate 的内部之前，让我们看看它的一些特点，这些特点将帮助您在下一个项目中使用。HTML5 Boilerplate 可以从`html5boilerplate.com`下载，并且根据 MIT 许可证可用于任何免费或商业产品。源代码可以在 Github 的 URL 上找到，即`github.com/h5bp/html5-boilerplate/`。

## 跨浏览器兼容性

HTML5 Boilerplate 带有一组文件，可以轻松进行跨浏览器开发。

### 文档类型

跨浏览器兼容性最重要的原因是使用不正确的文档类型声明。通过使用 HTML5 文档类型声明，您可以确保浏览器以标准模式呈现您的网站。

### 注意

如果您对文档类型有兴趣，我在`nimbupani.com/the-truth-about-doctypes.html`上详细介绍了它。

### Normalize.css

浏览器会在您未指定属性的元素上应用它们的默认样式。问题是，每个浏览器应用的样式是不同的。但是，`Normalize.css`确保这些默认样式在所有浏览器中保持一致。

### 注意

Nicolas Gallagher 在`necolas.github.com/normalize.css/`上详细介绍了`Normalize.css`背后的动机。

### 清除浮动

Clearfix 一直是清除浮动的一种流行方式。在 HTML5 Boilerplate 中，这已经简化为使用微清除解决方案，这是一组更小的选择器，可以实现相同的目标，并经过测试和验证，可以在 Opera 9 及更高版本，Safari 4 及更高版本，IE6 及更高版本，Firefox 3.5 及更高版本以及 Chrome 上使用。

### 注意

微清除解决方案的发明者 Nicolas Gallagher 在`nicolasgallagher.com/micro-clearfix-hack/`上更详细地介绍了使用的声明背后的选择。

### 搜索框样式

当您将输入元素的类型设置为搜索时，所有 WebKit 浏览器（如 Safari，Chrome，Mobile Safari，以及即将推出的）都会添加难以样式化的 UI chrome。HTML5 Boilerplate 带有一组样式，可以使搜索框在所有浏览器中的外观和感觉保持一致，同时也很容易进行样式设置。

### 条件类

`index.html`页面带有一组类，可以在 HTML 元素上轻松调整样式，以适应低于 9 的 IE 版本。

### Modernizr

Modernizr 是一个 JavaScript 库，用于测试 HTML5 技术的存在，并根据加载您网站的浏览器中的存在或不存在输出一组类。例如，如果浏览器不支持边框半径，Modernizr 会输出类`no-borderradius`，而在支持边框半径的浏览器上，它将输出类`borderradius`。Boilerplate 中包含了 Modernizr 的自定义构建。

### 注意

从[`modernizr.com/docs/`](http://modernizr.com/docs/)和[`www.slideshare.net/michaelenslow/its-a-mod-world-a-practical-guide-to-rocking-modernizr`](http://www.slideshare.net/michaelenslow/its-a-mod-world-a-practical-guide-to-rocking-modernizr)的幻灯片中了解有关使用 Modernizr 的更多信息。

### 没有 console.log 错误

在现代浏览器中工作时，通常会使用`console.log`函数来调试 JavaScript 代码。有多少次你忘记在生产中删除或注释掉它们，结果发现它们在不支持该函数使用的 Internet Explorer 或其他浏览器中抛出错误？你可以安全地使用`plugin.js`文件中包含的`log`函数，只在支持它的浏览器中记录语句。

### 辅助类

曾经需要隐藏文本来显示图片吗？如何为使用屏幕阅读器的人提供额外的文本或在所有浏览器中隐藏？HTML5 Boilerplate 提供了这两种情况的类，经过实地测试，可以在各种情况和所有浏览器中使用。

## 性能优化

`.htaccess`文件包含了最佳的缓存默认设置，这使得当页面由 Apache Web 服务器提供时，页面加载速度显著加快。还有其他 Web 服务器的配置文件可提供类似的功能。

## 渐进增强

HTML 元素有一个`no-js`类，可以用于为不支持 JavaScript 的浏览器提供替代样式。使用 Modernizr 时，当在支持 JavaScript 的浏览器中使用时，这个类名会被替换为`js`。

## 可访问的焦点样式

所有浏览器在点击链接时都会提供默认的焦点样式。HTML5 Boilerplate 确保这些样式仅在使用键盘导航时元素处于焦点状态时应用。

## 打印样式

一个良好的默认打印样式是我们在创建网页时经常忽略的东西。然而，HTML5 Boilerplate 已经为您提供了最佳性能的默认打印样式。

# 开始使用的工具

你可以使用你喜欢的编辑器开始使用 Boilerplate。如果你使用 Git 作为你的版本控制系统，我们还包括一个`.gitignore`文件，它会自动忽略文件，如`.DS_STORE`或其他不必要的文件，不会被标记为版本。可以用来处理 HTML5 Boilerplate 的一些编辑器如下：

+   Aptana Studio：HTML5 Boilerplate 可以直接在 Aptana Studio 中使用。选择一个 Web 项目，然后选择 Boilerplate 开始使用。Robert Gravelle 在[www.htmlgoodies.com/html5/tutorials/aptana-studio-3-guided-tour-and-tutorial-create-a-web-project-using-the-html-5-boilerplate-framework.html](http://www.htmlgoodies.com/html5/tutorials/aptana-studio-3-guided-tour-and-tutorial-create-a-web-project-using-the-html-5-boilerplate-framework.html)上有一篇文章，解释了如何在 Aptana Studio 项目中使用 HTML5 Boilerplate。

+   Visual Studio：在 Visual Studio 2010 中有两个可用的模板。一个是用于 Web 表单的，可以从`h5bpwebapptemplate.codeplex.com/`下载，另一个可以从[www.jondavis.net/techblog/post/2011/04/24/HTML5-Boilerplate-Visual-Studio-2010-Template.aspx](http://www.jondavis.net/techblog/post/2011/04/24/HTML5-Boilerplate-Visual-Studio-2010-Template.aspx)下载。

+   TextMate：这个 URL 托管了 HTML5 Boilerplate 的标记和样式的 TextMate 捆绑包，[www.dontcom.com/post/1546820479/html5-boilerplate-textmate-template-bundles](http://www.dontcom.com/post/1546820479/html5-boilerplate-textmate-template-bundles)。

## 注意

这些工具并非由 HTML5 Boilerplate 项目官方维护，因此可能已经过时。最好使用下一节中概述的流程。

# 获取文件的位置

有三种获取 HTML5 Boilerplate 的方法，如下：

+   从网站上：项目的最新稳定版本可在`html5boilerplate.com`上获得。

+   来自 Initializr：Jonathan Verecchia 在`initializr.com`上托管了一个更广泛的模块选择。这里的所有模块都来自该网站上可用的稳定版本。

+   从 Github 主页：HTML5 Boilerplate 托管在 Github 上。最新文件可从项目的 github 页面`github.com/h5bp/html5-boilerplate`获取。您可以放心地在启动新项目时使用这些文件，并且在从 Github 下载时保证获得这些文件的最新版本。

由于您刚刚开始使用 HTML5 Boilerplate，我强烈建议您从 Github 下载文件，甚至更好地通过 Git 这样做，这样当 Github 上的主文件更新时，您可以轻松更新它们。

### 注意

如果您对 Git 不熟悉，Roger Dudler 在 rogerdudler.github.com/git-guide/上维护了一个很好的入门介绍；如果您对版本控制的概念不熟悉，可以在`hoth.entp.com/output/git_for_designers.html`上找到一个很好的解释。

# H5BP 文件概述

HTML5 Boilerplate 的不同文件和文件夹解释如下：

+   `index.html`：这是我们建议您使用的所有 HTML 页面的标记。

+   `main.css`：样式位于名为`main.css`的单个样式表中，位于`css`文件夹中。

+   `normalize.css`：此文件位于单独位置，以便您可以立即使用最新更新的`normalize.css`版本。在生产中，理想情况下，您应将`main.css`和`normalize.css`合并为单个文件，以确保网络请求的最小数量，从而使您的页面加载更快。

+   `doc`：此文件夹包含了理解 HTML5 Boilerplate 文件所需的所有文档。

+   `img`：此文件夹应包含您将用于创建网站的所有图像。一开始是空的，但您应该在这里包含您使用的所有图像。

+   `js`：这是所有脚本的父文件夹。HTML5 Boilerplate 附带了一组脚本，使您更容易入门。此文件夹包含以下文件和文件夹：

+   `vendor`：此文件夹包含所有脚本库。您可以获取最新的压缩和未压缩版本的 jQuery 以及现代化的自定义版本。您将使用的任何其他库理想情况下都应放在此文件夹中。

+   `plugins.js`：您将使用的所有 jQuery 插件都应内联在此文件中。如果您使用了 jQuery 轮播插件，您将把代码复制到`plugins.js`中。

+   `main.js`：这将是您调用在页面上运行的脚本的文件。以 jQuery 轮播插件为例，我们将从此文件中调用插件在我们的页面上运行。

+   `404.html`：如果您有一个找不到的页面，那么可以提供此页面。确保它包含所有可用的信息，并且与网站中的其他页面具有相同的外观和感觉。

+   `humans.txt`：这是一个很棒的倡议，允许您指明谁在网站上工作（在 humanstxt.org 上阅读更多关于此倡议的信息）。我们强烈建议您使用此功能来指示您的工作，并告知任何好奇的人，这是谁的工作。

+   `crossdomain.xml`：如果您希望将 Flash 文件托管在其他地方以访问位于您的网站将托管的域上的资产，则此文件非常有用。您可以使用另一个域中的 Flash 音频播放器来使用托管在您的网站上的文件。在这种情况下，您需要仔细选择您的跨域策略（我们将在第五章中详细介绍此文件，*自定义服务器*）。

+   `robots.txt`：搜索引擎使用此文件来了解哪些文件要索引，哪些不要索引。

+   `.htaccess`：这是特定于您的网站的 Apache 服务器配置文件。默认情况下包含了大量最佳实践。

+   `favicion.ico`：大多数浏览器在您收藏网站上的页面或在标签上的页面标题旁边时使用网站图标。通过使用一个独特的可识别的图标，您将能够使您的网站脱颖而出，并且易于导航到。

+   `apple-touch-icon-*.png`：iOS 和 Android 设备允许将网站添加到手机主屏幕的书签。它们都使用这些触摸图标来表示您的网站。Boilerplate 附带了一组图标，以识别您需要创建图标的所有尺寸和格式。

+   `readme.md`：readme 包含了所有的许可信息，以及关于使用这些文件的功能和获取更多信息的列表。

# 寻求帮助

现在我们已经知道这些文件是什么，以及从哪里获取它们，重要的是您要熟悉如何寻求帮助，最重要的是在哪里寻求帮助。请记住，大多数 HTML5 Boilerplate 项目的维护者都是在业余时间工作。您在明确表达需要帮助的内容方面花费的时间越多，他们就能越快、越好地帮助您。以下是如何寻求帮助：

+   隔离问题：确切的问题是什么？使用 dabblet.com、codepen.io、jsfiddle.net 或 jsbin.com 创建一个最少标记、样式和脚本的测试案例来重现问题。大多数情况下，这样做本身就会让您找到问题所在。

+   如果您能够重现这个问题并将其隔离为 HTML5 Boilerplate 功能引起的问题，请转到`github.com/h5bp/html5boilerplate.com/issues`，使用搜索字段检查是否已经报告过。如果没有，请创建一个新问题，并附上测试案例的链接。

+   如果这个问题不是 HTML5 Boilerplate 的结果，而是您无法确定的交互作用，请转到`stackoverflow.com/questions/tagged/html5boilerplate`，创建一个链接到隔离测试案例的问题。确保将问题标记为 html5boilerplate 或 h5bp，这样维护者中的一个就可以迅速回答。

+   如果问题小到可以在 Twitter 上提问，请在[`twitter.com/h5bp`](https://twitter.com/h5bp)上发推文，附上测试案例的链接以及您需要帮助的具体部分。

### 注意

Lea Verou 在`coding.smashingmagazine.com/2011/09/07/help-the-community-report-browser-bugs/`上写了一篇关于提交浏览器错误报告的好文章，同样适用于寻求任何开源网页开发项目的帮助。

# 总结

在这一章中，我们已经了解了为什么 HTML5 Boilerplate 是网页开发者的绝佳工具箱。此外，我们已经了解了哪些功能对您的网页开发项目最有用，以及 HTML5 Boilerplate 中的每个文件都做了什么。我们还花了一些时间看看从哪里获取 HTML5 Boilerplate 的文件以及如何寻求帮助。在下一章中，我们将开始使用 HTML5 Boilerplate 进行一个示例项目。
