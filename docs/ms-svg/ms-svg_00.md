# 前言

这本书适用于希望在项目中添加可伸缩、设备独立的动画、图像和可视化效果的 Web 开发人员和设计师。可伸缩矢量图形是一种由万维网联盟（W3C）于 1998 年推出的图像文件格式。多年来，它一直因浏览器兼容性差和不友好的 API 而不受重视。在过去几年里，它已成为现代 Web 开发工具包的重要组成部分。SVG 为现代 Web 提供了许多重要功能。例如，在多种设备分辨率的世界中，它提供了一条简单的路径，可以实现高质量的图像缩放，而无需为图像生成多个分辨率，也无需跳过复杂的标记模式。此外，作为基于 XML 的标记，它还允许轻松访问常见的 JavaScript 模式，用于创建高度交互式的界面。

本书将教会你如何使用 SVG 作为静态图像、在 CSS 中、作为 HTML 文档中的元素以及作为动画或可视化的一部分进行编写的基础知识。

# 这本书适合谁

这本书适用于对可伸缩矢量图形感兴趣的 Web 开发人员。它是从前端 Web 开发人员的角度编写的，但任何有 JavaScript、CSS 和基于 XML 的语法经验的人都应该能够理解本书。

不需要有 SVG 的先验经验。

# 本书涵盖的内容

第一章《介绍可伸缩矢量图形》介绍了 SVG 的基础知识，并将向你展示一些使用该格式的基本示例。

第二章《开始使用 SVG 创作》详细介绍了创作 SVG 的基本概念。

第三章《深入挖掘 SVG 创作》介绍了更高级的 SVG 创作概念，包括变换、裁剪和遮罩，以及将 SVG 元素导入文档。

第四章《在 HTML 中使用 SVG》进一步介绍了在 HTML 文档中使用 SVG 元素和 SVG 图像的细节。

第五章《使用 SVG 和 CSS》介绍了在 CSS 中使用 SVG 图像，取代 PNG 和 Gif 在现代 Web 开发工具包中的使用。本章还介绍了使用 CSS 修改 SVG 元素的多种方法。

第六章《JavaScript 和 SVG》通过介绍常见的文档对象模型方法，教会读者基本的 JavaScript SVG 应用程序接口，这些方法允许开发人员访问和操作 SVG 属性。

第七章《常见的 JavaScript 库和 SVG》教授了如何从常见的库和框架（包括 jQuery、AngularJS、Angular 和 ReactJS）中与 SVG 进行交互的基础知识。

第八章《SVG 动画和可视化》介绍了使用 SVG 进行可视化和动画的示例。

第九章《辅助库 Snap.svg 和 SVG.js》介绍了两个当前帮助处理常见 SVG 任务的库：Snap.svg 和 SVG.js。

第十章《使用 D3.js 工作》介绍了 D3 的基本用法，并通过一些简单的示例来激发你对这个强大库的兴趣。

第十一章《优化 SVG 的工具》专注于优化 SVG 的不同工具。

# 为了充分利用本书。

本书假设你具有 HTML、XML、CSS 和 JavaScript 的知识。了解 Node.js 和基于 npm 的开发也会有所帮助。

在开始之前，确保您已安装了 Node.js 会很有帮助。您还需要一个文本编辑器。本书中的示例是使用 Visual Studio Code 编写的，但任何文本编辑器都可以。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便文件直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)上登录或注册。

1.  选择 SUPPORT 选项卡。

1.  点击代码下载和勘误。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的解压缩软件解压缩文件夹：

+   WinRAR/7-Zip 适用于 Windows

+   Zipeg/iZip/UnRarX 适用于 Mac

+   7-Zip/PeaZip 适用于 Linux

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Mastering-SVG`](https://github.com/PacktPublishing/Mastering-SVG)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有其他代码包来自我们丰富的书籍和视频目录，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/9781788626743_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781788626743_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。例如："将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。"

代码块设置如下：

```xml
<svg  width="350" height="150" viewBox="0 0 350 150" version="1.1">
    <circle cx="100" cy="75" r="50" fill="rgba(255,0,0,.5)"/>
    <circle cx="100" cy="75" r="50" fill="rgba(255,0,0,.5)" 
     transform="translate(10)" />
    <circle cx="100" cy="75" r="50" fill="rgba(255,0,0,.5)" 
     transform="translate(75,0)" />
</svg>
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目会以粗体显示：

```xml
types {
    image/svg+xml svg svgz;
}
```

任何命令行输入或输出都以以下方式编写：

```xml
 $ npx create-react-app react-svg
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会在文本中显示为这样。例如："在本文档中，我们创建了一个风格化的字母 R。"

警告或重要说明会显示为这样。

提示和技巧会显示为这样。
