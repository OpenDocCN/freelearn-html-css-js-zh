# 前言

欢迎来到《使用 HTML5 开发多人游戏》。本书将教你如何开发支持多个玩家在同一游戏世界中互动的游戏，并如何执行网络编程操作以实现这样的系统。它涵盖了诸如 WebSockets 和 JavaScript 中的客户端和服务器端游戏编程，延迟减少技术以及处理来自多个用户的服务器查询等主题。我们将通过从头到尾开发两款实际的多人游戏来实现这一目标，并在此过程中还将教授 HTML5 游戏开发的各种主题。本书的目的是教会你如何使用 HTML5 为多个玩家创建游戏世界，他们希望通过互联网进行竞争或互动。

# 本书涵盖内容

第一章, *开始多人游戏编程*，介绍了网络编程，重点是设计多人游戏。它通过引导你创建一个实时的井字棋游戏，来说明多人游戏开发的基本概念。

第二章, *设置环境*，描述了 JavaScript 开发领域的最新技术，包括通过 Node.js 在服务器端使用 JavaScript。它还描述了当前的技术，以管理 JavaScript 的开发周期和资源管理工具，如 Npm、Bower、Grunt 等。

第三章, *实时喂养蛇*，将现有的单人 Snake 游戏改造成具有使用先前描述的工具在同一游戏世界中与多个玩家一起玩的能力。还描述和演示了大厅、房间、匹配和处理用户查询的概念，为 Snake 游戏增加了功能。本章介绍了当今行业中最强大和广泛使用的 WebSocket 抽象——socket.io。

第四章, *减少网络延迟*，教授了减少网络延迟的技术，以创建流畅的游戏体验。其中最常见的技术之一——客户端预测，被演示并应用到了前一章描述的 Snake 游戏中。游戏服务器代码也被更新，以提高性能，引入了第二个更新循环。

第五章, *利用前沿技术*，描述了在网络平台上进行游戏开发所发现的令人兴奋的机会。它解释了 WebRTC、HTML5 的游戏手柄、全屏模式和媒体捕获 API。其他承诺和实验性技术和 API 也在此处描述。

第六章, *添加安全和公平游戏*，涵盖了与网络游戏相关的常见缺陷和安全漏洞。在这里，描述和演示了常见的技术，使你能够开发提供无作弊游戏体验的游戏。

# 本书所需内容

要使用本书，你需要安装 Node.js 和 Npm，现代的网络浏览器（如 Google Chrome 5.0，Firefox 3.5，Safari 5.0 或 Internet Explorer 9.0 及更高版本），以及文本编辑器或集成开发环境（IDE）。你还需要基本到中级的 JavaScript 知识，以及一些先前的游戏编程经验，最好是 JavaScript 和 HTML5。

# 本书的受众

本书的目标读者是能制作基本单人游戏的 HTML5 游戏开发人员，他们现在想尽快学习如何在他们的 HTML5 游戏中快速加入多人游戏功能。

# 约定

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些示例以及它们的含义解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名显示如下：“第一个将以`action`的值为键，第二个将以`data`的键为值。”

代码块设置如下：

```js
wss.on('connection', function connection(ws) {
    board.on(Board.events.PLAYER_CONNECTED, function(player) {
        wss.clients.forEach(function(client) {
            board.players.forEach(function(player) {
                client.send(makeMessage(events.outgoing.JOIN_GAME, player));
```

当我们希望引起您对代码块的特定部分的注意时，相关的行或项目将以粗体显示：

```js
validator.isEmail('foo@bar.com'); //=> true
validator.isBase64(inStr);
validator.isHexColor(inStr);
validator.isJSON(inStr);
```

任何命令行输入或输出都以以下方式编写：

```js
npm install socket.io --save
npm install socket.io-client –save

```

### 注意

警告或重要说明显示在这样的框中。

### 提示

提示和技巧显示如下。
