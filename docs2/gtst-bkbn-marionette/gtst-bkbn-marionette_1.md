# 第一章. 从 Backbone Marionette 开始

这本实用指南提供了使用 `Marionette.js` 编写可扩展应用程序的基本步骤。随着你通过初始示例的进展，你将了解框架组件如何协同工作以创建复合应用程序。但在我们深入示例之前，以下是你在本介绍章节中可以找到的一些内容：

+   `Marionette.js` 的描述和特性

+   `Marionette.js` 在 Backbone 应用程序中的作用

+   框架的好处

+   架构和可扩展性的概述

+   安装和文档说明

# 介绍 Marionette.js

`Backbone.js` 的复合应用程序库是 `Backbone.marionette`，也称为 `Marionette.js`。它为我们提供了核心结构，并简化了你的 JavaScript 应用程序需要可扩展性的许多模式和惯例。

## Backbone 需要 Marionette.js

`Backbone.js` 是一个越来越受欢迎的框架，用于构建单页应用以及中小型应用。它提供了一套优秀的构建块来组织你的前端开发，并构建支持移动设备的应用程序。然而，它将大部分的应用设计、架构和可扩展性留给了开发者。尽管如此，`Marionette.js` 通过填补 `Backbone.js` 自身未提供的空白，并提供了你可以利用的约定来构建自己的自定义对象。简单来说，`Marionette.js` 在开发 Backbone 应用程序时使你的生活变得更简单。

## Marionette.js 的关键好处

添加了许多用于创建现实世界应用程序的关键模式和工具，`Marionette.js` 在 Backbone 中找到了自己的位置。以下是一些你可以在这个框架中找到的好处：

+   结构、组织和模式

+   复合应用程序架构

+   带有事件聚合器的事件驱动架构

+   可以减少视图的样板代码

+   企业消息模式的影响

+   模块化选项

+   渐进式使用；因为它不是一个全有或全无的框架，这意味着你可以只使用你需要的组件

+   在视觉区域内部支持嵌套视图和布局

+   视图、区域和布局中的内置内存管理和僵尸清除

`Marionette.js` 提供了大量构建任何模块大小应用程序所需的应用基础设施组件。

`Marionette.js` 提供了一系列对象，这些对象有助于创建几乎任何大小和复杂性的良好结构化应用程序。它通过提供一组常见的设计和实现模式来实现这一目标，这些模式是创建者 Derick Bailey 在开发模块化 Backbone 应用程序时使用的。

## 构建大型应用程序

在规划应用程序的架构时，你通常会尽可能提前思考，试图实现一个解耦的架构，将功能分解成独立的模块，并避免在这些模块之间进行直接调用。因此，你可以添加和删除模块而不影响其行为。

|   | "构建大型应用程序的秘诀永远不是构建大型应用程序。将你的应用程序分解成小块。然后，将这些可测试的、易于管理的块组装成你的大型应用程序" |   |
| --- | --- | --- |
|   | -- Justin Meyer，JavaScriptMVC 的作者 |

考虑到相互关联的组件难以更改和扩展而不影响彼此。`Marionette.js`通过使用模块定义提供了非常增量且模块化的方法。它依赖于事件聚合器在模块之间发送消息以协调来自系统其他部分的功能，而不在许多其他对象类型中添加对它们的直接引用，这些对象类型有助于简化应用程序的设计。

## 逐步使用

这是`Marionette.js`的创造者用来创建框架的基本概念之一。`Marionette.js`通过提供应用程序对象和模块架构，促进了增量且模块化的方法，以跨子应用程序、功能和文件扩展应用程序。所有这些都是为了独立运行并与 Backbone 的核心组件协同工作，以满足应用程序的具体需求。

|   | "关键是从一开始就承认你不知道这一切会如何发展。当你接受自己不知道一切时，你开始设计具有防御性的系统...你应该花大部分时间思考接口而不是实现。" |   |
| --- | --- | --- |
|   | -- Nicholas Zakas，《高性能 JavaScript 网站》的作者 |

`Marionette.js`的主要优点之一是你不需要使用它的所有组件。jQuery-jQuery UI 的比较也适用于这里。你无论如何使用 jQuery 日历的事实迫使你必须使用整个 UI 库。同样，这也适用于 Marionette，因为你只使用对你应用程序有意义的其中一个组件，这并不强制你使用 Marionette 的其他组件。

# 安装 Marionette.js

我们将介绍如何下载和设置开发环境，以便你可以通过一些简单的步骤开始使用`Marionette.js`。如果你已经熟悉安装`Marionette.js`，你可能想跳过本章剩余的部分。请随意跳转到第二章，*我们的第一个应用程序*。

## 文本编辑器

在构建大型且可扩展的应用程序时，你将大部分时间花在代码编辑器上。我们建议你使用适合你的编辑器。Notepad++或 Sublime Text 无疑是很好的选择。Sublime Text 已经有很多片段和包可以帮助你在开发中。

## 网络浏览器

与复杂的客户端应用程序一起工作需要一套良好的开发者工具。为了编写本指南，我们将主要使用 Google Chrome 和 Mozilla Firefox，但所有代码和示例都应在所有现代浏览器（IE9+、Opera 和 Safari）中工作。

我们将使用 [jsfiddle.net](http://jsfiddle.net) 来展示运行示例。使用这个网站非常简单，大多数情况下，你只需要运行代码就能看到它的实际效果。

## 前置条件

在撰写本文时，`Marionette.js`的当前稳定版本是 v1.3.0，它依赖于以下库：

+   `JSON2.js`

+   `jQuery` (v1.7, v1.8, and v1.9)

+   `Underscore.js` (v1.4.4)

+   `Backbone.js` (v1.0.0)

+   `backbone.wreqr.js`

+   `backbone.babysitter.js`

我们想指出，对 Backbone 和 Underscore 有基本了解将帮助你从这本书中获得最大收益，并理解 Marionette 相对于纯 Backbone 开发的优点。

## 获取 Marionette.js

获取`Marionette.js`最新构建的最佳方式是访问项目网站，[`marionettejs.com/`](http://marionettejs.com/)。

他们有一个**预包装**选项。`.zip`文件包含了你开始使用`Marionette.js`（包括 Backbone、jQuery 和其他前置条件）所需的全部文件。你也可以下载不包含所有依赖的`Marionette.js`文件，如果这些库的 CDN 版本可用，你也可以只使用这些库的 CDN 版本。

## 文档

你可以从[`github.com/Marionette.jsjs/Marionette.js/tree/master/docs`](https://github.com/Marionette.jsjs/Marionette.js/tree/master/docs)下载`Marionette.js`每个部分的文档。文档分为多个文件。带注释的源代码可以在[`marionettejs.com/docs/backbone.marionette.html`](http://marionettejs.com/docs/backbone.marionette.html)找到。你可以在其维基页面上找到文章、博客文章、演示文稿、常见问题解答等，[`github.com/marionettejs/backbone.marionette/wiki`](https://github.com/marionettejs/backbone.marionette/wiki)。

Marionette 的创造者 Derick Bailey 创建了一个示例应用程序，可以用作使用`Marionette.js`构建 Backbone 应用程序的参考。该应用程序的名称是`BBCloneMail`，它是一个展示复合应用程序的绝佳例子。你可以在[`bbclonemail.heroku.com`](http://bbclonemail.heroku.com)上找到`BBCloneMail`，源代码在[`github.com/derickbailey/bbclonemail`](http://github.com/derickbailey/bbclonemail)。

# 摘要

在本章中，我们探讨了`Marionette.js`为构建可扩展应用程序提供的核心概念和优点。我们还提供了查找、下载和安装进行开发所需的基本工具的链接。在下一章中，你将介绍构成`Marionette.js`应用程序的组件或构建块。
