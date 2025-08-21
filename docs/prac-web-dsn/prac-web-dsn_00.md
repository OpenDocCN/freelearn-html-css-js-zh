# 前言

自从我开始在这个领域工作以来，我仍然对网络的发展感到惊讶。我一直喜欢互联网是一个快速发展的技术——技术、设计、流程以及一切都变化如此迅速。

《实用网页设计》是一本全面的网页设计师实践指南。每一章都经过彻底修订，提供易于理解和简单使用的信息、技巧和方法。

本书的第一部分是关于网页设计的基础知识。它关注其历史、发展，以及主要组成部分。我们将以逐步的设计工作流程和响应式设计与自适应设计的比较结束本书。

本书的第二部分将教你如何从头开始构建和实施你的网站，介绍 Bootstrap 框架、客户端渲染以及设计工作流程中最好的工具。

# 本书的受众

《实用网页设计》教读者网页设计的基础知识，以及如何从头开始构建具有交互和动态内容的响应式网站。这是任何想学习网页设计和前端开发的人的完美书籍。适合没有经验的人，也适合有一些经验并愿意提高的人。

# 充分利用本书

要充分利用本书，最好有一些设计经验，但并非必需。你可以在完全不了解的情况下完成这门课程。

此外，你需要一台运行 Windows 或 OS X 的计算机；你最喜欢的互联网浏览器的最新版本（Chrome、Firefox 或 Safari）；以及一个代码编辑器，在本书中，我们将使用 Atom。

# 下载示例代码文件

你可以从[www.packtpub.com](http://www.packtpub.com)的账户中下载本书的示例代码文件。如果你在其他地方购买了本书，你可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，文件将直接发送到你的邮箱。

你可以按照以下步骤下载代码文件：

1.  登录或注册[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”选项卡。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

文件下载后，请确保使用以下最新版本解压或提取文件夹：

+   WinRAR/7-Zip 适用于 Windows

+   Zipeg/iZip/UnRarX 适用于 Mac

+   7-Zip/PeaZip 适用于 Linux

本书的代码包也托管在 GitHub 上：[`github.com/PacktPublishing/Practical-Web-Design`](https://github.com/PacktPublishing/Practical-Web-Design)。如果代码有更新，将在现有的 GitHub 存储库上更新。

我们还提供了来自我们丰富书籍和视频目录的其他代码包，可以在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。去看看吧！

# 下载彩色图片

我们还提供了一份 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。你可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/PracticalWebDesign_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/PracticalWebDesign_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。这是一个例子：“让我们创建这个文件夹并将其命名为`Racing Club Website`。”

代码块设置如下：

```html
<html> <!--This is our HTML main tag-->
 <head> <!--This is our head tag where we put our title and script and all infos relative to our page.-->
  <title>My Page Title</title>
 </head>
 <body> <!--This is where all our content will go-->
  <h1>John Doe</h1>

 </body>
</html>
```

当我们希望引起你对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```html
.content {
  background-color: red;
  width: 75%;
}
.sidebar {
  background-color: green;
  width: 25%;
}
```

**粗体**：表示一个新术语，一个重要词，或者你在屏幕上看到的词。例如，菜单或对话框中的单词会以这种方式出现在文本中。这是一个例子：“然后点击右上角的三个点，然后点击“显示设备框架”。

警告或重要说明会显示为这样。

提示和技巧会显示为这样。
