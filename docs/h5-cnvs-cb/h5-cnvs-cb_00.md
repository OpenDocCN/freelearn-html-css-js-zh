# 前言

HTML5 画布正在改变网络上的图形和可视化。由 JavaScript 驱动，HTML5 Canvas API 使 Web 开发人员能够在浏览器中创建可视化和动画，而无需 Flash。尽管 HTML5 Canvas 迅速成为在线图形和交互的标准，但许多开发人员未能充分利用这一强大技术所提供的所有功能。

《HTML5 Canvas Cookbook》首先介绍了 HTML5 Canvas API 的基础知识，然后提供了处理 API 不直接支持的高级技术，如动画和画布交互的方法。最后，它提供了一些最常见的 HTML5 画布应用的详细模板，包括数据可视化、游戏开发和 3D 建模。它将使您熟悉有趣的主题，如分形、动画、物理学、颜色模型和矩阵数学。

通过本书的学习，您将对 HTML5 Canvas API 有扎实的理解，并掌握了创建任何类型的 HTML5 画布应用的技术，仅受想象力的限制。

# 本书内容

第一章，“开始使用路径和文本”，首先介绍了子路径绘制的基础知识，然后通过探索绘制锯齿和螺旋的算法来深入研究更高级的路径绘制技术。接下来，本章深入探讨了文本绘制，最后探索了分形。

第二章，“形状绘制和合成”，首先介绍了形状绘制的基础知识，并向您展示如何使用颜色填充、渐变填充和图案。接下来，本章深入研究了透明度和合成操作，然后提供了绘制更复杂形状的方法，如云、齿轮、花朵、纸牌花色，甚至是一个完整的矢量飞机，包括图层和阴影。

第三章，“使用图像和视频”，介绍了图像和视频处理的基础知识，向您展示如何复制和粘贴画布的部分，并涵盖了不同类型的像素操作。本章还向您展示了如何将图像转换为数据 URL，将画布绘制保存为图像，并使用数据 URL 加载画布。最后，本章以一个可以用于动态聚焦和模糊图像的像素操作的像素化图像焦点算法结束。

第四章，“掌握变换”，探索了画布变换的可能性，包括平移、缩放、旋转、镜像变换和自由形式变换。此外，本章还详细探讨了画布状态堆栈。

第五章，“使用动画使画布栩栩如生”，首先构建一个`Animation`类来处理动画阶段，并向您展示如何创建线性运动、二次运动和振荡运动。接下来，它涵盖了一些更复杂的动画，如肥皂泡的振荡、摆动的钟摆和旋转的机械齿轮。最后，本章以创建自己的粒子物理模拟器的方法结束，并提供了在画布内创建数百个微生物以测试性能的方法。

第六章，与画布交互：将事件侦听器附加到形状和区域，首先构建了一个扩展画布 API 的`Events`类，提供了一种在画布上附加事件侦听器到形状和区域的方法。接下来，该章节涵盖了获取画布鼠标坐标的技术，检测区域事件，检测图像事件，检测移动触摸事件和拖放。该章节最后提供了一个创建图像放大器的方法和另一个创建绘图应用程序的方法。

第七章，创建图表和图表，提供了生产就绪的图表类，包括饼图、条形图、方程图和折线图。

第八章，用游戏开发拯救世界，通过展示如何创建一个名为 Canvas Hero 的整个横向卷轴游戏，让您开始使用画布游戏开发。该章节向您展示如何创建精灵表，创建关卡和边界地图，创建处理英雄、坏人、关卡和英雄生命的类，还向您展示如何使用 MVC（模型视图控制器）设计模式构建游戏引擎。

第九章，介绍 WebGL，首先构建了一个 WebGL 包装类，以简化 WebGL API。该章节通过展示如何创建一个 3D 平面和一个旋转的立方体来介绍 WebGL，还向您展示如何向模型添加纹理和光照。该章节最后展示了如何创建一个完整的 3D 世界，您可以在其中进行第一人称探索。

附录 A，附录 B 和附录 C 讨论了其他特殊主题，如画布支持检测、安全性、画布与 CSS3 过渡和动画，以及移动设备上画布应用的性能。

# 您需要什么

要开始使用 HTML5 画布，您只需要一个现代浏览器，如 Google Chrome，Firefox，Safari，Opera 或 IE9，以及一个简单的文本编辑器，如记事本。

# 这本书适合谁

本书面向熟悉 HTML 和 JavaScript 的 Web 开发人员。它适用于初学者和有一定 JavaScript 工作知识的 HTML5 开发人员。

# HTML5 画布是什么？

Canvas 最初是由苹果于 2004 年创建的，用于实现 Dashboard 小部件并在 Safari 浏览器中提供图形支持，后来被 Firefox、Opera 和 Google Chrome 采用。如今，画布是下一代 Web 技术的新 HTML5 规范的一部分。

HTML5 画布是一个 HTML 标签，您可以将其嵌入到 HTML 文档中，用于使用 JavaScript 绘制图形。由于 HTML5 画布是一个位图，绘制到画布上的每个像素都会覆盖其下的像素。

这是本书所有 2D HTML5 画布配方的基本模板：

```js
<!DOCTYPE HTML>
<html>
    <head>
        <script>
            window.onload = function(){
                var canvas = document.getElementById("myCanvas");
                var context = canvas.getContext("2d");

                // draw stuff here
            };
        </script>
    </head>
    <body>
        <canvas id="myCanvas" width="578" height="200">
        </canvas>
    </body>
</html>
```

请注意，画布元素嵌入在 HTML 文档的主体内，并且使用`id`、`width`和`height`进行定义。JavaScript 使用`id`引用画布标签，`width`和`height`用于定义绘图区域的大小。一旦使用`document.getElementById()`访问了画布标签，我们就可以定义一个 2D 上下文：

```js
var context = canvas.getContext("2d");
```

尽管本书大部分内容涵盖了 2D 上下文，但最后一章使用了 3D 上下文来使用 WebGL 渲染 3D 图形。

# 约定

在本书中，您会发现一些区分不同信息类型的文本样式。以下是一些这些样式的示例，以及它们的含义解释。

文本中的代码词显示如下：“定义`Events`构造函数。”

代码块设置如下：

```js
var Events = function(canvasId){
    this.canvas = document.getElementById(canvasId);
    this.context = this.canvas.getContext("2d");
    this.stage = undefined;
    this.listening = false;
};
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目会以粗体显示：

```js
var Events = function(canvasId){
 this.canvas = document.getElementById(canvasId);
 this.context = this.canvas.getContext("2d");
    this.stage = undefined;
    this.listening = false;
};
```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词，在菜单或对话框中出现在文本中，就像这样：“它在原点处写出文本**Hello Logo!**”

### 注意

警告或重要说明会出现在这样的框中。

### 提示

提示和技巧会以这种方式出现。
