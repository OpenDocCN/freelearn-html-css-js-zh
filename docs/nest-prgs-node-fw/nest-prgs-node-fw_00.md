# 前言

# 什么是 Nest.js？

有很多可用的 Web 框架，随着 Node.js 的出现，发布的框架更是层出不穷。随着 Web 技术的变化和发展，JavaScript 框架很快就会进入和退出流行。Nest.js 对许多开发人员来说是一个很好的起点，因为它使用一种非常类似于迄今为止最常用的 Web 语言 JavaScript 的语言。许多开发人员是使用诸如 Java 或 C/C++之类的语言来学习编程的，这两种语言都是严格的语言，因此使用 JavaScript 可能会有点尴尬，并且由于缺乏类型安全性，容易出错。Nest.js 使用 TypeScript，这是一个很好的折衷方案。它是一种语言，提供了 JavaScript 的简单性和强大性，同时又具有您可能习惯的其他语言的类型安全性。Nest.js 中的类型安全性仅在编译时可用，因为 Nest.js 服务器被编译为运行 JavaScript 的 Node.js Express 服务器。然而，这仍然是一个重大优势，因为它允许您在运行时之前更好地设计无错误的程序。

Node.js 在 NPM（Node Package Manager）中拥有丰富的软件包生态系统。拥有超过 35 万个软件包，它是世界上最大的软件包注册表。使用 Express 的 Nest.js 在开发 Nest 应用程序时，您可以访问每一个这些软件包。许多软件包甚至为其软件包提供了类型定义，允许 IDE 读取软件包并提供建议/自动填充代码，这在跨 JavaScript 代码与 TypeScript 代码交叉时可能是不可能的。Node.js 最大的好处之一是可以从中提取模块的庞大存储库，而不必编写自己的模块。Nest.js 已经将其中一些模块包括在 Nest 平台的一部分中，比如`@nestjs/mongoose`，它使用 NPM 库`mongoose`。在 2009 年之前，JavaScript 主要是一种前端语言，但在 2009 年 Node.js 发布之后，它推动了许多 JavaScript 和 TypeScript 项目的开发，如：Angular、React 等。Angular 对 Nest.js 的开发产生了很大的启发，因为两者都使用了允许可重用的模块/组件系统。如果您不熟悉 Angular，它是一个基于 TypeScript 的前端框架，可用于跨平台开发响应式 Web 应用程序和原生应用程序，并且它的功能与 Nest 非常相似。两者在一起也非常搭配，Nest 提供了运行通用服务器的能力，以提供预渲染的 Angular 网页，以加快网站交付时间，使用了上面提到的服务器端渲染（SSR）。

# 关于示例

本书将引用一个托管在 GitHub 上的 Nest.js 项目的示例（https://github.com/backstopmedia/nest-book-example）。在整本书中，代码片段和章节将引用代码的部分，以便您可以看到您所学习的内容的一个工作示例。示例 Git 存储库可以在命令提示符中克隆。

```js
git clone https://github.com/backstopmedia/nest-book-example.git

```

这将在您的计算机上创建项目的本地副本，您可以通过使用 Docker 构建项目在本地运行：

```js
docker-compose up

```

一旦您的 Docker 容器在本地主机：3000 端口上运行起来，您将希望在做任何其他事情之前运行迁移。要做到这一点，请运行：

```js
docker ps

```

获取正在运行的 Docker 容器的 ID：

```js
docker exec -it [ID] npm run migrate up

```

这将运行数据库迁移，以便您的 Nest.js 应用程序可以使用正确的模式读取和写入数据库。

如果您不想使用 Docker，或者无法使用 Docker，您可以使用您选择的软件包管理器（如`npm`或`yarn`）构建项目：

```js
npm install

```

或

```js
yarn

```

这将在您的`node_modules`文件夹中安装依赖项。然后运行：

```js
npm start:dev

```

或以下内容启动您的 Nest.js 服务器：

```js
yarn start:dev

```

这些将运行`nodemon`，如果有任何更改，将导致您的 Nest.js 应用程序重新启动，使您无需停止、重建和重新启动应用程序。

# 关于作者

+   格雷格·马戈兰（Greg Magolan）是 Rangle.io 的高级架构师、全栈工程师和 Angular 顾问。他在 Agilent Technologies、Electronic Arts、Avigilon、Energy Transfer Partners、FunnelEnvy、Yodel 和 ACM Facility Safety 等公司工作了 15 年以上，开发企业软件解决方案。

+   杰伊·贝尔（Jay Bell）是 Trellis 的首席技术官。他是一名资深的 Angular 开发人员，使用 Nest.js 在生产中开发领先行业的软件，帮助加拿大的非营利组织和慈善机构。他是一位连续创业者，曾在许多行业开发软件，从利用无人机帮助打击森林大火到构建移动应用程序。

+   大卫·吉哈罗（David Guijarro）是 Car2go Group GmbH 的前端开发人员。他在 JavaScript 生态系统内有丰富的工作经验，成功建立并领导了多元文化、多功能团队。

+   阿德里安·德佩雷蒂（Adrien de Peretti）是一名全栈 JavaScript 开发人员。他对新技术充满热情，不断寻找新挑战，特别对人工智能和机器人领域感兴趣。当他不在电脑前时，阿德里安喜欢在大自然中玩各种运动。

+   帕特里克·豪斯利（Patrick Housley）是 VML 的首席技术专家。他是一名拥有超过六年技术行业经验的 IT 专业人士，能够分析涉及多种技术的复杂问题，并提供详细的解决方案和解释。他具有强大的前端开发技能，有领导开发团队进行维护和新项目开发的经验。
