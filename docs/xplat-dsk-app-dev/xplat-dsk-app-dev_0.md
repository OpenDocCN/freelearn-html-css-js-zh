# 前言

HTML5 桌面应用程序开发正在蓬勃发展，如果考虑到 JavaScript 现在是网络上最流行的编程语言，这一点也就不足为奇了。HTML5 的特性与 Node.js 和运行时 API 的结合非常丰富，更不用说 GitHub 上提供的无数 Node.js 模块了。此外，HTML5 桌面应用程序可以在不修改代码的情况下分发到不同的平台（Windows、macOS 和 Linux）。

本书的目标是帮助读者发现令人兴奋的机会，解锁 Node.js 驱动的运行时（NW.js 和 Electron）给 JavaScript 开发人员，并且惊讶地发现在这个领域追赶编程细节是多么容易。

# 本书需要什么

要构建和运行本书中的示例，您需要使用 Linux 或 macOS；您还需要 npm/Node.js。在撰写本文时，作者使用以下软件测试了示例：

+   npm v.5.2.0

+   节点 v.8.1.1

+   Ubuntu 16.04 LTS、Windows 10 和 macOS Sierra 10.12

# 本书适合谁

本书适用于任何对使用 HTML5 创建桌面应用程序感兴趣的开发人员。前两章需要基本的网络技能（HTML、CSS 和 JavaScript）和 Node.js 的基础知识。本书的这一部分包括了 npm 的速成课程，本书将使用它来构建和运行示例，前提是您有使用命令行的经验（Linux、macOS 或 Windows）。接下来的四章需要对 React 有一定的经验。最后两章，有基本的 TypeScript 知识会有所帮助。

# 约定

在本书中，您会发现一些区分不同信息类型的文本样式。以下是一些样式的示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下："嗯，我们可以更改语言环境并触发事件。那么如何使用模块呢？

在 `FileList` 视图中，我们有一个 `formatTime` 静态方法，用于格式化传入的 `timeString` 以便打印。我们可以根据当前选择的 `locale` 进行格式化。

代码块设置如下：

```js
{ 
 "name": "file-explorer", 
 "version": "1.0.0", 
 "description": "", 
 "main": "main.js", 
 "scripts": { 
 "test": "echo "Error: no test specified" && exit 1" 
 }, 
 "keywords": [], 
 "author": "", 
 "license": "ISC" 
} 

```

任何命令行输入或输出都以以下方式编写：

```js
sudo npm install nw --global

```

新术语和重要单词以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的单词，会以这样的方式出现在文本中："显示项目"菜单包含"文件夹"、"复制"、"粘贴"和"删除"。

警告或重要说明会以这样的方式出现在框中。

提示和技巧会以这样的方式出现。
