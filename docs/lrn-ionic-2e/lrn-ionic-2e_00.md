# 前言

本书解释了如何使用 Ionic 轻松构建混合移动应用。无论是与 REST API 集成的简单应用，还是涉及原生功能的复杂应用，Ionic 都提供了简单的 API 来处理它们。

凭借对网页开发和 TypeScript 的基本知识，以及对 Angular 的良好了解，一个人可以轻松地将百万美元的创意转化为一款只需几行代码的应用。

在本书中，我们将探讨如何实现这一点。

# 本书涵盖的内容

第一章，*Angular - 入门*，向您介绍全新 Angular 的强大功能。我们将了解 TypeScript 的基础知识和理解 Angular 所需的概念。我们将学习 Angular 模块、组件和服务，并通过构建一个应用来结束本章。

第二章，*欢迎使用 Ionic*，介绍了名为 Cordova 的混合移动框架。它展示了 Ionic 如何融入混合移动应用开发的大局。本章还介绍了使用 Ionic 进行应用开发所需的软件。

第三章，*Ionic 组件和导航*，带您了解 Ionic 的各种组件，从页眉到导航栏。我们还将学习使用 Ionic Framework 在页面之间进行导航。

第四章，*Ionic 装饰器和服务*，探讨了我们用于初始化各种 ES6 类的装饰器。我们还将学习平台服务、配置服务以及其他一些内容，以更好地理解 Ionic。

第五章，*Ionic 和 SCSS*，讨论了如何利用内置的 SCSS 支持为 Ionic 应用设置主题。

第六章，*Ionic Native*，展示了 Ionic 应用如何使用 Ionic Native 与设备功能如相机和电池进行接口交互。

第七章，*构建 Riderr 应用*，展示了本书如何构建一个端到端的应用，该应用可以使用本书迄今为止所学的知识与设备 API 和 REST API 进行接口交互。我们将构建的应用将是 Uber API 的前端。使用这个应用，用户可以预订 Uber 车辆。

第八章，*Ionic 2 迁移指南*，展示了如何将使用 Ionic Framework v1 构建的 Ionic 应用迁移到 Ionic 2，并且相同的方法也适用于 Ionic 3。

第九章，*测试 Ionic 2 应用*，将带您了解测试 Ionic 应用的各种方法。我们将学习单元测试、端到端测试、monkey 测试以及使用 AWS Device Farm 进行设备测试。

第十章，*发布 Ionic 应用*，展示了如何使用 Ionic CLI 和 PhoneGap Build 生成 Cordova 和 Ionic 构建的应用的安装程序。

第十一章，*Ionic 3*，讨论了升级到 Angular 4 和 Ionic 3 的内容。我们还将了解 Ionic 3 的一些新功能。

附录，展示了如何有效地使用 Ionic CLI 和 Ionic 云服务来构建、部署和管理您的 Ionic 应用。

# 本书所需的内容

要开始构建 Ionic 应用，您需要对 Web 技术、TypeScript 和 Angular 有基本的了解。对移动应用开发、设备原生功能和 Cordova 的良好了解是可选的。

您需要安装 Node.js、Cordova CLI 和 Ionic CLI 才能使用 Ionic 框架。如果您想要使用设备功能，如相机或蓝牙，您需要在您的设备上设置移动操作系统。

本书旨在帮助那些想要学习如何使用 Ionic 构建移动混合应用程序的人。它也非常适合想要使用 Ionic 应用程序主题、集成 REST API，并了解更多关于设备功能（如相机、蓝牙）的人。

对 Angular 的先前了解对于成功完成本书至关重要。

# 约定

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下：“TypeScript 文件保存为`.ts`扩展名。”

代码块设置如下：

```html
x = 20; 
// after a few meaningful minutes  
x = 'nah! It's not a number any more';

```

任何命令行输入或输出都以以下方式编写：

```html
npm install -g @angular/cli

```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的单词，会在文本中出现，就像这样：“我们将编写三种方法，一种用于获取随机 gif，一种用于获取最新趋势，一种用于使用关键字搜索 Gif API。”

警告或重要说明以以下方式显示。

提示和技巧会以这种方式出现。
