# 前言

WebGL 是一种强大的网络技术，它将硬件加速的 3D 图形带到了浏览器中，而无需用户安装额外的软件。鉴于 WebGL 基于 OpenGL 并将 3D 图形编程引入网络开发，即使是经验丰富的网络开发者也可能感到陌生。另一方面，对于在传统计算机图形方面有经验的人来说，使用 JavaScript 构建 3D 应用程序需要一些适应。一种常见的观点是，JavaScript 的速度不如其他在计算机图形中使用的传统语言快；尽管在比较 CPU 密集型算法时这是一个担忧，但在比较 GPU 密集型工作时则不是问题。这正是 WebGL 发挥作用的地方！WebGL 提供的强大功能，加上浏览器无处不在的普及性和易用性，使这项技术在推动网络沉浸式体验的未来方面处于独特且吸引人的位置。

本书包含许多示例，展示了尽管 WebGL 外观不友好，但它仍然易于学习。每一章都针对 3D 图形编程的一个重要方面，并提供了其实施的不同替代方案。这些主题总是与练习相关联，允许读者将这些概念付诸实践。

*《使用 WebGL 2 的实时 3D 图形》* 提供了学习 WebGL 2 的清晰路线图。虽然 WebGL1 基于 OpenGL ES 2.0 规范，但 WebGL 2 是从 OpenGL ES 3.0 演变而来，这保证了许多 WebGL1 扩展和新特性的可用性。每一章都从本章的学习目标总结开始，接着详细描述每个主题。本书提供了丰富示例、最新的对一系列基本 WebGL 主题的介绍，包括绘制、颜色、纹理、变换、帧缓冲区、光照、表面、几何形状等。每一章都包含了大量有用且实用的示例，展示了这些主题在 WebGL 场景中的实现。随着每一章的阅读，你的 3D 图形编程技能将得到提升。本书将成为你值得信赖的伴侣，其中包含了开发引人入胜的 3D 网络应用程序所需的 JavaScript 和 WebGL 2 的信息。

# 本书面向对象

本书是为对在网络上构建 3D 应用程序感兴趣的开发者所写。对 JavaScript 和线性代数有基本的了解是理想的，但不是强制性的。不期望有先前的 WebGL 知识。

# 本书涵盖内容

第一章，*入门*，介绍了 HTML5 `canvas` 元素，并描述了如何为它获取 WebGL 2 上下文。之后，它讨论了 WebGL 应用程序的基本结构。虚拟汽车展厅应用程序被展示为 WebGL 功能的演示。此应用程序还展示了 WebGL 应用程序的不同组件。

第二章，*渲染*，介绍了 WebGL API 用于定义、处理和渲染对象。本章还演示了如何使用 AJAX 和 JSON 进行异步几何加载。

第三章，*灯光*，介绍了 ESSL，WebGL 的着色语言。本章展示了如何使用 ESSL 着色器实现 WebGL 场景的照明策略。着色和反射光照模型背后的理论被涵盖，并通过各种示例付诸实践。

第四章，*相机*，说明了在 WebGL 中使用矩阵代数创建和操作相机的方法。这里还描述了在 WebGL 场景中使用的透视矩阵和法线矩阵。本章还展示了如何将这些矩阵传递给 ESSL 着色器，以便它们可以应用于每个顶点。本章包含几个示例，展示了如何在 WebGL 中设置相机。

第五章，*动画*，扩展了矩阵的使用，以在场景元素上执行几何变换（移动、旋转、缩放）。本章讨论了矩阵栈的概念。展示了如何使用矩阵栈为场景中的每个对象维护独立的变换。本章还描述了使用矩阵栈和 JavaScript 计时器的几种动画技术。每种技术都提供了实际演示。

第六章，*颜色、深度测试和 Alpha 混合*，详细介绍了在 ESSL 着色器中使用颜色的方法。本章展示了如何在 WebGL 场景中定义和操作多个光源。它还解释了深度测试和 Alpha 混合的概念，并展示了如何使用这些功能创建半透明对象。本章包含几个实际练习，将这些概念付诸实践。

第七章，*纹理*，展示了如何在 WebGL 场景中创建、管理和映射纹理。本章介绍了纹理坐标和纹理映射的概念。本章讨论了通过实际示例展示的不同映射技术。本章还展示了如何使用多个纹理和立方体贴图。

第八章，*拾取*，描述了拾取的简单实现，这是描述用户与场景中对象选择和交互的技术术语。本章中描述的方法计算鼠标点击坐标，并确定用户是否点击了在 `canvas` 中渲染的任何对象。解决方案的架构通过几个回调钩子展示，这些钩子可以用于实现特定逻辑的应用。还给出了几个拾取的示例。

第九章，*整合一切*，将本书中讨论的概念联系起来。本章回顾了演示的架构，并重新审视和扩展了在第一章*入门*中概述的虚拟汽车展厅应用。以虚拟汽车展厅为案例研究，本章展示了如何将 Blender 模型导入 WebGL 场景，以及如何创建支持 Blender 中使用的材质的 ESSL 着色器。

第十章，*高级技术*，提供了一些高级技术的示例，包括后处理效果、点精灵、法线贴图和光线追踪。每种技术都附有实际示例。阅读本章后，你将能够独立掌握更多高级技术。

第十一章，*WebGL 2 亮点*，概述了 WebGL 2 规范的一些主要特性和更新。本章还提供了将基于 WebGL1 的应用程序转换为 WebGL 2 的迁移策略。

第十二章，*未来之旅*，通过关于技术、概念、工具和社区的推荐，总结了*使用 WebGL 2 的实时 3D 图形*。读者可以利用这些推荐在掌握实时 3D 图形的旅程中受益。

# 为了充分利用本书

你需要一个支持 WebGL 2 的浏览器。除了 Microsoft Internet Explorer 之外，所有主要的浏览器供应商都支持 WebGL 2：

+   Firefox 51 或更高版本

+   Google Chrome 56 或更高版本

+   Chrome for Android 64 或更高版本

可以在这里找到支持 WebGL 的浏览器更新列表：[`www.khronos.org/webgl/wiki/Getting_a_WebGL_Implementation`](http://www.khronos.org/webgl/wiki/Getting_a_WebGL_Implementation)。

你还需要一个能够识别和突出显示 JavaScript 语法的源代码编辑器。

你还需要一个 Web 服务器，如 Apache、Lighttpd 或 Python，以加载远程资源。

# 下载示例代码文件

你可以从[www.packt.com](http://www.packt.com)的账户下载本书的示例代码文件。如果你在其他地方购买了这本书，你可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择**支持**标签。

1.  点击**代码下载与勘误**。

1.  在**搜索**框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书中的代码包也托管在 GitHub 上，地址为**[`github.com/PacktPublishing/Real-Time-3D-Graphics-with-WebGL-2`](https://github.com/PacktPublishing/Real-Time-3D-Graphics-with-WebGL-2)**。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的图书和视频目录的代码包可供选择，这些代码包可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**找到。查看它们吧！

# 本地运行示例

如果你没有网络服务器，我们建议你从以下选项中安装一个轻量级网络服务器：

+   **Serve**：[`github.com/zeit/serve`](https://github.com/zeit/serve)

+   **Lighttpd**：[`www.lighttpd.net`](http://www.lighttpd.net/)

+   **Python 服务器**：[`developer.mozilla.org/en-US/docs/Learn/Common_questions/set_up_a_local_testing_server`](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/set_up_a_local_testing_server)

话虽如此，为了在你的机器上本地运行示例，请确保从`examples`目录的根目录运行你的服务器，因为`common`目录是跨章节的共享依赖项。

# 下载彩色图像

我们还提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。你可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/9781788629690_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781788629690_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入等。以下是一个示例：“在你的编辑器中打开`ch01_01_demo.html`文件。”

代码块按以下方式设置：

```js
<html>
<head>
 <title>Real-Time 3D Graphics with WebGL 2</title>

 <style type="text/css">
 canvas {
 border: 5px dotted blue;
 }
 </style>
</head>
<body>

 <canvas id="webgl-canvas" width="800" height="600">
 Your browser does not support the HTML5 canvas element.
 </canvas>

</body>
</html>
```

当我们希望将你的注意力引到代码块的一个特定部分时，相关的行或项目将以粗体显示：

```js
<canvas id="webgl-canvas" width="800" height="600">
  Your browser does not support the HTML5 canvas element.
</canvas>
```

任何命令行输入或输出都按以下方式编写：

```js
$ mkdir webgl-demo
$ cd webgl-demo
```

**粗体**：表示新术语、重要单词或你在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要注意事项如下所示。

小贴士和技巧如下所示。

# 部分

在本书中，你会发现一些频繁出现的标题（行动时间、刚刚发生了什么？和尝试一下）。

为了清楚地说明如何完成一个程序或任务，我们按照以下方式使用这些部分：

# 行动时间

1.  动作 1

1.  动作 2

1.  动作 3

指令通常需要一些额外的解释以确保它们有意义，因此它们后面跟着这些部分：

***刚刚发生了什么？***

本节解释了你刚刚完成的任务或指令的工作原理。

# 尝试一下

这些是实际挑战，它们为你提供了如何使用所学内容进行实验的想法。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对这本书的任何方面有疑问，请在邮件主题中提及书名，并给我们发送邮件至`customercare@packtpub.com`。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将非常感激您能向我们报告。请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packt.com`与我们联系，并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用过这本书，为何不在购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 公司可以了解您对我们产品的看法，并且我们的作者可以看到他们对书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问[packt.com](http://www.packt.com/)。
