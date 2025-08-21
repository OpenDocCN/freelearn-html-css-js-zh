# 第六章：为您的游戏添加功能

这一章与前几章略有不同，因为本章没有与之相关的游戏。我们之所以不使用本章的概念构建游戏，是因为所涵盖的概念要么对于单独的一章来说过于复杂（例如，整本书都致力于 WebGL 的主题），要么它们并不是游戏中特别好的匹配。此外，本章末尾提到的一些功能在浏览器支持方面仍然很少（如果有的话），API 的稳定性可能也不太可靠。因此，我们将简单解释每个 API，提供有意义的示例，并希望这种肤浅的介绍足以让您对每个 API 所涉及的前景感到兴奋。

本章的第一部分将涵盖*四个*非常令人兴奋和强大的 HTML5 API，它们是浏览器平台的重要补充。首先，我们将介绍**WebGL**，它将**OpenGL ES**的强大功能带入浏览器，实现了硬件加速的 3D 图形渲染，而无需任何插件。接下来，我们将讨论如何使用 Web 套接字实现类似线程的体验，视频 API 实现原生视频播放和操作，以及地理位置信息，它允许 JavaScript 确定用户的物理位置（地理位置）。

最后，我们将通过查看 HTML5 演变中的最新功能来结束本章。这些功能将 CSS 提升到一个新的水平，使其不再仅仅是一个基于矩形的渲染引擎。我们将学习的第一个新功能是 CSS 着色器，它允许我们指定每个像素的渲染方式。这是使用 GLSL 着色器完成的，正如我们在 WebGL 讨论中将看到的那样，它们是我们编写并在 GPU 上运行的独立程序，以尽可能低的层次控制渲染方式。通过自定义着色器，我们可以做的远远超出简单的预设 CSS 变换。

本章后半部分涵盖的其他新的 CSS 功能是 CSS 列和 CSS 区域和排除。CSS 列使得动态调整容器显示多少列文本变得非常容易。例如，如果我们希望一块文本以 3 个等宽或等高列显示，通常需要设置三个不同的容器，然后将每个容器浮动到左侧。使用列，我们可以简单地将所有文本存储在单个容器中，然后使用 CSS 生成列。最后，CSS 区域和排除使得在复杂图案内或周围呈现文本成为可能，而不是传统的矩形形状。您肯定见过杂志这样做，其中一块文本围绕着汽车轮廓或其他物体的轮廓。过去，使用纯文本（而不是使用图像）实现这种效果在 HTML 中几乎没有尝试，因为这需要极其复杂的操作。现在只需要几行 CSS 代码。

# 高级 HTML5 API

尽管以下 API 和功能在复杂性和学习曲线陡度上有很大差异，但我们的目标是至少对每个主题进行彻底介绍。为了更深入地了解和实践每个主题，建议您在这里提供的介绍中补充其他来源。

由于 HTML5 规范和功能的部分尚未完全成熟，一些 API 可能在所有浏览器中都不完全支持，即使是最新的现代浏览器也是如此。由于本章将涵盖 HTML5 的绝对最新功能（在撰写时），有可能一些浏览器可能不适合本章涵盖的示例。因此，建议您使用最先进的网络浏览器的最新版本。不仅如此，您还必须确保检查您的浏览器可用的任何实验性功能和/或安全标志。以下代码片段是专门针对谷歌 Chrome 编写的，因为它支持所有描述的功能。我们将注意到任何特定的配置设置，以确保功能正常工作，但随着新的 Web 浏览器更新的部署，这些可能需要或不需要。

# WebGL

也许没有其他 HTML5 功能对游戏开发人员来说像 WebGL 那样令人兴奋。这个新的 JavaScript API 允许我们渲染高性能、硬件加速的 2D 和 3D 图形。该 API 是 OpenGL ES 2.0 的一种变体，并利用 HTML5 画布元素来弥合浏览器和用户计算机中的图形处理单元之间的差距。

虽然 3D 编程是一个值得一本书的话题，但以下概述足以让我们开始学习最重要的概念，并且将允许我们开始使用浏览器平台进行 3D 游戏开发。对于那些寻找 OpenGL ES 2 的良好学习资源的人，可以看看*Munshi，Ginsburg 和 Shreiner 的 OpenGL ES 2.0 编程指南*。

### 注意

由于 WebGL 在很大程度上基于 OpenGL ES 2.0，您可能会想要从 OpenGL 书籍和其他来源寻找关于它的参考和补充材料。请记住，OpenGL 版本 1.5 及更早版本与 OpenGL 2.0（以及由此产生的 WebGL 的 OpenGL ES 2.0）有很大不同，可能不是一个完整的学习来源，尽管它可能是一个不错的起点。

这两个版本之间的主要区别是渲染管线。在早期版本中，API 使用了一个固定的管线，重活由幕后完成。新版本暴露了一个完全可编程的管线，我们需要提供自己的**着色器**程序来将我们的模型渲染到屏幕上。

## 你好，世界！

在进一步探讨 WebGL 和 3D 编程的理论方面之前，让我们快速看一下最简单的可能的 WebGL 应用程序，在这里我们将简单地渲染一个黄色三角形在绿色背景上。您会注意到这需要相当多的代码行。请记住，WebGL 解决的问题并不是一个微不足道的问题。WebGL 的目的是渲染最复杂的三维交互场景，而不是简单的静态二维形状，正如下面的例子所示。

为了避免大段的代码片段，我们将把示例分解成几个单独的部分。每个部分将按照它们执行的顺序呈现。

我们需要做的第一件事是设置我们的示例将运行的页面。这里有两个组件，两个着色器程序（关于着色器程序是什么的更多信息将在后面介绍）和`WebGLRenderingContext`对象的初始化。

```js
<body>

  <script type="glsl-shader/x-fragment" id="glsl-frag-simple">
    precision mediump float;

    void main(void) {
      gl_FragColor = vec4(1.0, 1.0, 0.3, 1.0);
    }
  </script>

  <script type="glsl-shader/x-vertex" id="glsl-vert-simple">
    attribute vec3 aVertPos;

    uniform mat4 uMVMat;
    uniform mat4 uPMat;

    void main(void) {
      gl_Position = uPMat * uMVMat * vec4(aVertPos, 1.0);
    }
  </script>

  <script>
    (function main() {
      var canvas = document.createElement("canvas");
      canvas.width = 700;
      canvas.height = 400;
      document.body.appendChild(canvas);

      var gl = null;
      try {
        gl = canvas.getContext("experimental-webgl") ||
          canvas.getContext("webgl");
        gl.viewportWidth = canvas.width;
        gl.viewportHeight = canvas.height;
      } catch (e) {}

      if (!gl) {
        document.body.innerHTML =
          "<h1>This browser doesn't support WebGl</h1>";
      }

      var shaderFrag = document.getElementById
        ("glsl-frag-simple").textContent;
      var shaderVert = document.getElementById
      ("glsl-frag-simple").textContent;
    })();
  </script>
</body>
```

`glsl-shader/x-vertex`和`glsl-shader/x-fragment`类型的`script`标签利用了 HTML 如何渲染未知标签。当浏览器解析一个带有它不理解的`type`属性的`script`标签（即一个虚构的类型，比如`glsl-shader/x-vertex`）时，它会简单地忽略标签的所有内容。由于我们想要在 HTML 文件中定义着色器程序的内容，但又不希望该文本显示在 HTML 文件中，这种小技巧非常方便。这样我们就可以定义这些脚本，访问它们，而不用担心浏览器不知道如何处理那种特定的语言。

如前所述，在 WebGL 中，我们需要向 GPU 提供所谓的着色器程序，这是用一种称为**GLSL**（OpenGL 着色语言）的语言编写的实际编译程序，它为 GPU 提供了渲染我们的模型所需的指令。变量`shaderFrag`和`shaderVert`保存了每个着色器程序的源代码的引用，这些源代码本身包含在我们自定义的`script`标签中。

接下来，我们创建一个常规的 HTML5 画布元素，将其注入到 DOM 中，并创建一个`gl`对象。注意 WebGL 和 2D 画布之间的相似之处。当然，在这一点之后，这两个 API 一个来自火星，一个来自金星，但在那之前，它们的初始化是相同的。我们不是从画布对象请求 2D 渲染上下文对象，而是简单地请求 WebGL 渲染上下文。由于大多数浏览器（包括谷歌 Chrome）在 WebGL 方面仍处于实验阶段，因此在请求上下文时，我们必须使用实验前缀提供`webgl`字符串。分隔两个`getContext`调用的布尔`OR`运算符表示我们正在从实验前缀请求上下文，或者不使用前缀。浏览器支持的调用将成功。

从这一点开始，对 WebGL 的每个 API 调用都是通过这个`gl`对象完成的。如果返回`WebGLRenderingContext`对象的对画布的调用失败，我们就无法对 WebGL 进行任何调用，最好是停止执行。否则，我们可以继续进行我们的程序，传递这个对象，以便我们可以与 WebGL 交互。

```js
function getShader(gl, code, type) {
  // Step 1: Create a specific type of shader
  var shader = gl.createShader(type);

  // Step 2: Link source code to program
  gl.shaderSource(shader, code);

  // Step 3: Compile source code
  gl.compileShader(shader);

  return shader;
}

function getShaderProgram(gl, shaderFrag, shaderVert) {

  // Step 1: Create a shader program
  var program = gl.createProgram();

  // Step 2: Attach both shaders into the program
  gl.attachShader(program, shaderFrag);
  gl.attachShader(program, shaderVert);

  // Step 3: Link the program
  gl.linkProgram(program);

  return program;
}

(function main() {
  // ...

  var shaderFrag = getShader(gl,
    document.getElementById("glsl-frag-simple").textContent,
    gl.FRAGMENT_SHADER);

  var shaderVert = getShader(gl,
    document.getElementById("glsl-vert-simple").textContent,
    gl.VERTEX_SHADER);

  var shader = getShaderProgram(gl, shaderFrag, shaderVert);

  // Specify which shader program is to be used
  gl.useProgram(shader);

  // Allocate space in GPU for variables
  shader.attribVertPos = gl.getAttribLocation(shader, "aVertPos");
  gl.enableVertexAttribArray(shader.attribVertPos);

  shader.pMatrixUniform = gl.getUniformLocation
    (shader, "uPMatrix");
  shader.mvMatrixUniform = gl.getUniformLocation
    (shader, "uMVMatrix");
})();
```

这个过程的下一步是创建顶点和片段着色器，然后将它们组合成一个单一的着色器程序。顶点着色器的整个工作是指定最终渲染模型中顶点的位置，片段着色器的工作是指定两个或多个顶点之间每个像素的颜色。由于任何渲染都需要这两个着色器，WebGL 将它们合并成一个单一的着色器程序。

着色器程序成功编译后，它将被发送到 GPU，其中处理片段和顶点。我们可以通过在发送到 GPU 之前在着色器程序中指定的指针位置来将输入发送到我们的着色器中。这一步是通过在`gl`对象（`WebGLRenderingContext`对象）上调用`get*Location`方法来完成的。一旦我们有了对这些位置的引用，我们可以稍后为它们分配一个值。

请注意，我们的着色器脚本声明了`vec4`和`mat4`类型的变量。在诸如 C 或 C++之类的强类型语言中，变量可以具有`int`（整数）、`float`（浮点数）、`bool`（布尔值）或`char`（字符）类型。在 GLSL 中，有一些新的数据类型是该语言的本机类型，这些类型在图形编程中特别有用。这些类型是向量和矩阵。我们可以使用数据类型`vec2`创建一个具有两个分量的向量，或者使用`vec4`创建一个具有四个分量的向量。同样，我们可以通过调用`mat3`创建一个 3 x 3 矩阵，它实质上创建了一个具有三个`vec3`元素的类似数组的结构。

```js
function initTriangleBuffer(gl) {
  // Step 1: Create a buffer
  var buffer = gl.createBuffer();

  // Step 2: Bind the buffer with WebGL
  gl.bindBuffer(gl.ARRAY_BUFFER, buffer);

  // Step 3: Specify 3D model vertices
  var vertices = [
    0.0,   0.1, 0.0,
    -1.0, -1.0, 0.0,
    1.0,  -1.0, 0.0
  ];

  // Step 4: Fill the buffer with the data from the model
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices),
    gl.STATIC_DRAW);

  // Step 5: Create some variables with information about the
    vertex buffer
  // to simplify calculations later on

  // Each vertex has an X, Y, Z component
  buffer.itemSize = 3;

  // There are 3 unique vertices
  buffer.numItems = parseInt(vertices.length / buffer.itemSize);

  return buffer;
}

(function main() {
  // ...

  var triangleVertBuf = initTriangleBuffer(gl);
})();
```

在我们放置了一个着色器程序之后，这个程序将告诉显卡如何为我们绘制的点进行绘制，接下来我们需要一些点来绘制。因此，下一步创建了一个我们稍后将要绘制的点的缓冲区。如果您还记得第四章，*使用 HTML5 捕捉蛇*，在那里我们介绍了新的类型化数组，那么这对您来说将是熟悉的。WebGL 存储顶点数据的方式是使用这些类型化数组，但更具体地说，是 32 位浮点数组。

在这种情况下，我们只绘制一个三角形，计算和跟踪所有点是一个微不足道的任务。然而，3D 模型通常不是手工绘制的。在使用某种 3D 建模软件绘制复杂模型之后，我们将导出代表模型的几百到几千个单独顶点。在这种情况下，我们需要计算模型有多少个顶点，并且最好将这些数据存储在某个地方。由于 JavaScript 允许我们动态地向对象添加属性，我们利用这一点将这两个计算存储在缓冲对象本身上。

最后，让我们实际将我们的三角形绘制到屏幕上。当然，如果我们还没有写足够的样板代码，让我们谈谈 3D 编程的一个主要组成部分，并写一点额外的代码来允许我们最终渲染我们的模型。

不要深入讨论 3D 坐标空间和转换矩阵的话题，将 3D 形状渲染到 2D 屏幕（例如您的计算机显示器）的一个关键方面是，我们需要执行一些线性代数来将表示我们模型的点从 3D 空间转换为简单的 2D 空间（考虑 x 和 y 坐标）。这是通过创建一对矩阵结构并执行一些矩阵乘法来完成的。然后，我们只需要将我们 3D 模型中的每个点（在这个例子中是我们的三角形缓冲区）乘以一个称为**MVP 矩阵**的矩阵（这是由三个单独的矩阵组成的矩阵，即模型、视图和投影矩阵）。这个矩阵是通过乘以单独的矩阵构建的，每个矩阵代表从 3D 到 2D 的转换过程中的一步。

如果您以前上过任何线性代数课程，您会知道矩阵相乘并不像乘以两个数字那么简单。您还会注意到，在 JavaScript 中表示矩阵也不像定义一个整数类型的变量那么微不足道。为了简化和解决这个问题，我们可以使用 JavaScript 中提供的许多矩阵实用程序库之一。在这个例子中，我们将使用一个非常强大的名为**GL-Matrix**的库，这是由 Brandon Jones 和 Colin MacKenzie IV 创建的开源库。

```js
<script src="img/glmatrix.js"></script>
…

function drawScene(gl, entityBuf, shader) {
  // Step 1: Create the Model, View and Projection matrices
  var mvMat = mat4.create();
  var pMat = mat4.create();

  // Step 2: Initialize matrices
  mat4.perspective(45, gl.viewportWidth / gl.viewportHeight, 0.1,
    100.0, pMat);
  mat4.identity(mvMat);
  mat4.translate(mvMat, [0.0, 0.5, -3.0]);

  // Step 3: Set up the rendering viewport
  gl.viewport(0, 0, gl.viewportWidth, gl.viewportHeight);
  gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

  // Step 4: Send buffers to GPU
  gl.bindBuffer(gl.ARRAY_BUFFER, entityBuf);
  gl.vertexAttribPointer(shader.attribVertPos,
    entityBuf.itemSize, gl.FLOAT, false, 0, 0);
  gl.uniformMatrix4fv(shader.pMatrixUniform, false, pMat);
  gl.uniformMatrix4fv(shader.mvMatrixUniform, false, mvMat);

  // Step 5: Get this over with, and render the triangle already!
  gl.drawArrays(gl.TRIANGLES, 0, entityBuf.numItems);
}

(function main() {
  // ...

  // Clear the WebGL canvas context to some background color
  gl.clearColor(0.2, 0.8, 0.2, 1.0);
  gl.enable(gl.DEPTH_TEST);

  // WebGL: Please draw this triangle on the gl object,
    using this shader...
  drawScene(gl, triangleVertBuf, shader);
})();
```

关于前面的代码有几点值得注意。首先，您会注意到这是一个只绘制一次的单帧。如果我们决定在场景中进行动画（在真正的游戏中肯定会这样做），我们需要在请求动画帧循环中运行`drawScene`函数。这个循环将涉及到所有显示的步骤，包括生成我们的 MVP 矩阵的所有模型的矩阵数学。是的，这是要在更复杂的场景上多次每秒执行的大量计算。

其次，观察我们的模型视图投影矩阵的使用。我们首先将它们创建为 4x4 矩阵，然后实例化每一个。投影矩阵的作用就是这样——将 3D 点投影到 2D 空间（画布渲染上下文），根据需要拉伸点以保持画布指定的纵横比。在 WebGL 中，渲染上下文的坐标系在两个轴（垂直和水平轴）上从零到一。投影矩阵使得可能将点映射到超出该有限范围的点。

模型和视图矩阵使我们能够将点建模为相对于对象中心（其自己的坐标系）到世界坐标系的点。例如，假设我们正在建模一个机器人。假设机器人的头部位于点（0, 0, 0）的中心。从那个点开始，机器人的手臂可能分别位于相对于机器人头部的点（-5, 1, 0）和（5, 1, 0）。但是机器人在世界上的位置究竟在哪里？如果我们在这个场景中有另一个机器人，它们相对于彼此的位置是如何的？通过模型和视图矩阵，我们可以将它们都放在同一个全局坐标系上。在我们的例子中，我们将三角形移动到点（0, 0, -0.5, -3.0），这是一个接近世界坐标系原点的点。

最后，我们将我们的矩阵绑定到显卡上，在那里我们通过调用`WebGLRenderingContext`对象中定义的绘制函数来渲染我们的场景。如果您仔细观察`drawScene`函数的末尾，我们会向`shader`对象发送一些值。查看我们之前编写的两个着色器程序（使用 GLSL），我们指定了三个变量，这些变量作为程序的输入。细心的学生会问这些变量来自哪里（这些变量在顶点着色器中定义，命名为`aVertPos`、`uMVMat`和`uPMat`，这些是 GLSL 语言中定义的特殊数据类型）。它们来自我们的 JavaScript 代码，并通过调用`gl.vertexAttribPointer`和`gl.uniformMatrix4fv`将它们传递到 GPU 中的着色器程序。

大约 150 行代码后，我们有一个黄色三角形在绿色背景上渲染，如下面的截图所示。再次提醒您，WebGL 绝不是一个简单的编程接口，也不是用于可以使用更简单工具完成的简单绘图的首选工具，比如画布元素的 2DRenderingContext、SVG，甚至只是一个简单的图片编辑软件。

尽管 WebGL 需要大量样板代码来渲染一个非常简单的形状，如下面的截图所示，但渲染和动画复杂场景并不比这复杂多少。设置渲染上下文、创建着色器程序和加载缓冲区所需的基本步骤，在创建极其复杂的场景时也是一样的。

![Hello, World!](img/6029OT_07_01.jpg)

总之，尽管 WebGL 对于刚接触 HTML5 甚至游戏开发的开发人员来说可能是一个难题，但其基本原理是相当简单的。对于那些希望加深对 WebGL（或 3D 图形编程）理解的人，建议您学习三维编程和线性代数的相关主题，以及 WebGL 独有的原则。作为奖励，可以继续熟悉 GLSL 着色语言，因为这是 WebGL 的核心所在。

# Web 套接字

如果你曾经考虑过在 HTML5 中创建高性能的多人游戏，那么新的 Web 套接字 API 正是你一直在寻找的东西。如果你以前没有做过太多套接字编程，那么你一直缺少的就是这个：不是每次需要请求资源时都要建立与服务器的连接，而是套接字只需创建一次连接，然后客户端和服务器可以在同一连接上来回通信。换句话说，想象一下给某人打电话，说“你好”，然后在对方回答“你好”后挂断电话。然后，你再次给那个人打电话，等待他们接听电话，一旦你们都准备好了，你就问对方在电话那头怎么样。收到答案后，你再次挂断电话。这种情况持续了整个对话的时间，你每次只问一个问题（或者一次只做一个陈述），大部分时间都是你们两个在等待电话响起并连接电话。

现在，通过套接字编程，上述情景就像是打一个电话，然后在不挂断电话的情况下进行整个对话。你只有在对话最终结束，你和对方说再见，并同意挂断电话时才会挂断电话。在这种情况下，问题和答案之间几乎没有延迟，只有声音从一个电话传到另一个电话所涉及的固有延迟。

在 HTML5 中，套接字 API 分为两部分，即服务器部分和客户端部分。套接字的服务器端是我们在本书中不会过多讨论的，考虑到所涉及的性质。客户端接口是我们将大部分讨论的地方，尽管你会高兴地知道，Web 套接字和 Web 工作者的 JavaScript 接口几乎是相同的。

```js
// Step 1: Open connection
var con = new WebSocket
  ("ws://localhost:8888/packt/sockets/multiplayer-game-server");

// Step 2: Register callbacks
con.addEventListener("open", doOnOpen);
con.addEventListener("error", doOnError);
con.addEventListener("message", doOnMessage);
con.addEventListener("close", doOnClose);

function doOnOpen(event) {
  var msg = document.createElement("p");
  msg.textContent = "Socket connected to " + event.srcElement.URL;
  document.body.appendChild(msg);
}

function doOnError(event) {
  var msg = document.createElement("p");
  msg.textContent = "Error: " + event;
  document.body.appendChild(msg);
} 
function doOnMessage(event) {
  var response = JSON.parse(event.data);

  var msg = document.createElement("p");
  msg.textContent = "Message received: " + response.message;
  document.body.appendChild(msg);
}

function doOnClose(event) {
  var msg = document.createElement("p");
  msg.textContent = "Socket connection closed at " +
    event.timeStamp;
  document.body.appendChild(msg);
}

// Step 3: Send a message to the server
con.send("Hello!");
```

从前面的代码片段中可以看出，Web 套接字接口和 Web 工作者接口之间没有太多的区别。也许最显著的区别是我们可以通过哪个接口向服务器发送消息。Web 工作者使用`postMessage`函数，而 Web 套接字使用`send`函数。传统的事件处理函数与工作者的工作方式完全相同。套接字有四个与之关联的事件，分别是`onOpen`，`onClose`，`onError`和`onMessage`。前两个事件`onOpen`和`onClose`在服务器成功验证请求并升级与浏览器的连接时以及服务器以某种方式关闭与特定套接字的连接时被调用。`onError`事件在服务器应用程序发生错误时触发。最后，当服务器向客户端推送消息时，JavaScript 套接字的句柄通过`onMessage`回调函数被警告。传递给函数的事件对象与 Web 工作者`onMessage`事件对象类似，具有一个`data`属性，其中包含实际发送的数据，以及一个`timestamp`属性，指示消息发送的时间。

## 连接

了解 Web 应用程序如何通过 Web 套接字连接到后端服务器对于学习套接字 API 的工作原理至关重要。首先要记住的是，连接浏览器与服务器的协议与通常的 HTTP 连接不同。浏览器保持与服务器的连接方式是通过使用新的`WebSocket`协议，这是通过以下几个步骤完成的。`WebSocket`协议基于传统的 TCP，并使用 HTTP 来升级浏览器和后端服务器之间的连接，如下面的屏幕截图所示：

![连接](img/6029OT_07_02.jpg)

当我们在 JavaScript 中创建`WebSocket`类的实例时，浏览器会尝试与服务器建立持久的套接字连接。首先发生的事情是浏览器向`WebSocket`构造函数中指定的 URI 发送 HTTP 请求。此请求包含一个升级标头，指定它希望将连接升级到使用`WebSocket`协议。然后服务器和浏览器执行典型的握手，对于本书的目的，不会详细解释。如果您有兴趣实现自己的后端服务器应用程序来处理这个低级握手，可以参考在线官方 Web 套接字文档。

简而言之，客户端将此 HTTP 请求发送到服务器，包括一个包含密钥的标头，这是一个简单的文本字符串。然后服务器对该字符串进行哈希和编码，并发送回一个 HTTP 响应，浏览器验证并接受协议升级是否一切正常。如果这个握手成功，浏览器将实例化`WebSocket`对象，然后我们可以使用它通过相同的连接与服务器通信。

## 服务器端代码

Web 套接字的典型用例是多人游戏，其中两个或更多玩家要么相互对战，要么共享同一游戏，但来自不同的地点。这样的游戏可以通过两个玩家从不同的计算机连接到服务器，然后服务器接收来自两个玩家的输入并发送计算出的输出来实现。然后，每个玩家的客户端应用程序只需渲染从服务器接收到的数据。例如，玩家 A 按下键盘上的一个键，使由玩家 A 控制的角色跳跃。这些数据被发送到服务器，服务器会跟踪角色的位置以及是否可以跳跃等。服务器根据从玩家 A 接收到的输入计算要执行的操作（在这个例子中，服务器确定玩家 A 的角色现在正在执行跳跃），然后将玩家 A 的角色的更新状态发送给玩家 A 和玩家 B。他们的应用程序然后只需渲染玩家 A 的角色在空中。当然，每个玩家的游戏本地实例也会根据本地玩家的操作渲染其计算出的状态，以提供即时反馈。但是，游戏的服务器端实例有能力使来自任一玩家的输入导致的任何游戏状态无效。这样，两个玩家都可以体验非常流畅、响应迅速的多人游戏体验，同时保持游戏的完整性。

现在，根据服务器端代码实现的具体语言，这可能是一个微不足道的任务，也可能是一个真正的噩梦。总的来说，这个服务器端代码需要跟踪连接到它的所有套接字。显然，应用程序的复杂性将与游戏的目标相关。然而，就 Web 套接字 API 而言，主要的重点是使用`send`接口函数将数据传递回客户端，并通过`onMessage`函数检查输入。

## 客户端代码

正如我们在前面的代码片段中看到的，使用 JavaScript 的`WebSocket`对象非常简单。但是需要记住的两件事是，对`WebSocket.send`的每次调用都是异步的，并且传递给`WebSocket.send`的任何数据必须是（或将被转换为）`DOMString`。这意味着如果我们向服务器发送对象、函数或其他任何内容，服务器将以 UTF-16 编码的字符串形式接收。如果我们向服务器发送 JSON 字符串，那么我们只需要解析数据并访问具体内容。但是，如果我们只是发送一个实际的对象，比如一个字面的 JSON 对象，服务器将收到以下代码片段中的内容：

```js
// Client code
var con = new WebSocket
  ("ws://localhost:8888/packt/sockets/multiplayer-game-server");
// …

con.send({name: "Rodrigo"});

// Server code
String input = get_input_from_socket();
input.toString() == "[object Object]";
```

因此，通过 Web 套接字发送对象时，JavaScript 不会尝试对对象进行编码，而是简单地调用对象的`toString`函数，并将其输出发送到套接字。

# 视频

能够直接在浏览器内播放视频而无需担心插件是一种愉快的体验。不仅如此，由于视频元素实际上是 DOM 的一个本机部分，这意味着我们也可以像处理所有其他 DOM 元素一样处理它。换句话说，我们可以对视频元素应用 CSS 样式，浏览器会很乐意为我们解决问题。例如，假设我们想要创建视频在闪亮表面上播放的效果，其中视频在垂直方向反射，反射渐隐，融入背景，如下面的截图所示：

![Video](img/6029OT_07_03.jpg)

由于浏览器负责渲染视频，以及对其管理的所有元素应用 CSS 样式和效果，我们不必担心渲染带有特殊效果的视频所涉及的逻辑。但是请记住，我们在视频上添加的 CSS 越多，浏览器就需要越多的工作来使视频看起来符合我们的要求，这可能会迅速影响性能。但是，如果我们在视频中添加的只是一些简单的细节，那么大多数现代 Web 浏览器都不会在渲染时出现问题。

```js
<style>
video {
  -webkit-box-reflect: below 1px;
  -webkit-transition: all 1.5s;
}

video {
  -webkit-filter: contrast(250%);
}

div {
  position: relative;
}

div img {
  position: absolute;
  left: 0;
  top: 221px;
  width: 400px;
  height: 220px;
}
</style>

<div>
  <video controls width="400" height="220"
    poster="bunny-poster.png">
    <!-- Video courtesy of http://www.bigbuckbunny.org -->
    <source src="img/bunny.ogg" type="video/ogg" />
    <source src="img/bunny.mp4" type="video/mp4" />
    <source src="img/bunny.webm" type="video/webm" />
  </video>
  <img src="img/semi-transparent-mask.png" />
</div>
```

与新的 HTML5 音频元素类似，我们可以更多或更少地使用标签的两种方式。一种方法是简单地创建 HTML 节点，指定与`audio`标签相同的属性，指定一个或多个`source`节点，然后结束。或者，我们可以使用可用的 JavaScript API，并以编程方式操纵视频文件的播放。

```js
// Step 1: Create the video object
var video = document.createElement("video");
video.width = 400;
video.height = 220;
video.controls = true;
video.poster = "bunny-poster.png";

// Step 2: Add one or more sources
var sources = [
  {src: "bunny.ogg", type: "video/ogg"},
  {src: "bunny.mp4", type: "video/mp4"},
  {src: "bunny.webm", type: "webm"}
];

for (var i in sources) {
  var source = document.createElement("source");
  source.src = sources[i].src;
  source.type = sources[i].type;

  video.appendChild(source);
}

// Step 3: Make video player visible
document.body.appendChild(video);
```

我们还可以忽略默认控件，并通过利用引用视频元素的 JavaScript 对象上可用的属性来自行管理播放、暂停、调整音量等操作。以下是我们可以在视频对象上调用的属性和函数列表。

## 属性

+   `autoplay`（布尔值）

+   `currentTime`（浮点数—以秒为单位）

+   `paused`（布尔值）

+   `controls`（布尔值）

+   `muted`（布尔值）

+   `width`（整数）

+   `height`（整数）

+   `videoWidth`（整数—只读）

+   `videoHeight`（整数—只读）

+   `poster`（字符串—图像 URI）

+   `duration`（整数—只读）

+   `loop`（布尔值）

+   `currentSrc`（字符串）

+   `preload`（布尔值）

+   `seeking`（布尔值）

+   `playbackRange`（整数）

+   `ended`（布尔值）

+   `volume`（整数—介于 0 和 100 之间，不包括 0 和 100）

## 事件

| `loadstart` | 用户代理开始查找媒体数据，作为资源选择算法的一部分。 |
| --- | --- |
| `progress` | 用户代理正在获取媒体数据。 |
| `suspend` | 用户代理有意不获取媒体数据。 |
| `abort` | 用户代理在完全下载之前停止获取媒体数据，但不是由于错误。 |
| `error` | 在获取媒体数据时发生错误。 |
| `emptied` | 其网络状态先前不处于`NETWORK_EMPTY`状态的媒体元素刚刚切换到该状态（要么是因为在加载过程中发生了致命错误，即将报告，要么是因为在资源选择算法已经运行时调用了`load()`方法）。 |
| `stalled` | 用户代理正在尝试获取媒体数据，但数据出乎意料地没有出现。 |
| `loadedmetadata` | 用户代理刚刚确定了媒体资源的持续时间和尺寸，文本轨道已准备就绪。 |
| `loadeddata` | 用户代理可以首次在当前播放位置渲染媒体数据。 |
| `canplay` | 用户代理可以恢复播放媒体数据，但估计如果现在开始播放，媒体资源无法以当前播放速率一直播放到结束，而无需停止进行进一步的内容缓冲。 |
| `canplaythrough` | 用户代理估计，如果现在开始播放，媒体资源可以以当前播放速率一直播放到结束，而无需停止进行进一步的缓冲。 |
| `playing` | 经过暂停或由于缺乏媒体数据而延迟后，播放已准备好开始。 |
| `waiting` | 播放已经停止，因为下一帧尚未准备好，但用户代理预计该帧将及时准备好。 |
| `seeking` | 寻找的 IDL 属性已更改为 true。 |
| `seeked` | 寻找的 IDL 属性已更改为 false。 |
| `ended` | 播放已停止，因为媒体资源的结束已经到达。 |
| `durationchange` | 持续时间属性刚刚被更新。 |
| `timeupdate` | 当前播放位置因正常播放或特别有趣的方式（例如不连续地）而发生了变化。 |
| `play` | 元素不再暂停。在`play()`方法返回后触发，或者`autoplay`属性导致播放开始时触发。 |
| `pause` | 元素已暂停。在`pause()`方法返回后触发。 |
| `ratechange` | 默认的`Playback Rate`或`playback Rate`属性刚刚被更新。 |
| `volumechange` | `volume`属性或`muted`属性已更改。在相关属性的 setter 返回后触发。 |

### 注

有关事件的更多信息，请访问 W3C 候选推荐媒体事件[`www.w3.org/TR/html5/embedded-content-0.html#mediaevents`](http://www.w3.org/TR/html5/embedded-content-0.html#mediaevents)

你应该对新的 HTML5 视频元素感到兴奋的另一个原因是，视频的每一帧都可以直接渲染到画布 2D 渲染上下文中，就像单独的一帧是一个独立的图像一样。这样，我们就能够在浏览器上进行视频处理。不幸的是，我们无法导出由我们的 JavaScript 应用程序创建的视频的`video.toDataURL`等价物。

```js
var ctx = null;
var ctxOff = null;

var poster = new Image();
poster.src = "bunny-poster.jpg";
poster.addEventListener("click", initVideo);
document.body.appendChild(poster);

// Step 1: When the video plays, call our custom drawing function
video.autoplay = false;
video.loop = false;

// Step 2: Add one or more sources
var sources = [
  {src: "bunny.ogg", type: "video/ogg"},
  {src: "bunny.mp4", type: "video/mp4"},
  {src: "bunny.webm", type: "webm"}
];

for (var i in sources) {
  var source = document.createElement("source");
  source.src = sources[i].src;
  source.type = sources[i].type;

  video.appendChild(source);
}

// Step 3: Initialize the video
function initVideo() {
  video.addEventListener("play", initCanvas);
  video.play();
}

// Step 4: Only initialize our canvases once
function initCanvas() {
  // Step 1: Initialize canvas, if needed
  if (ctx == null) {
    var canvas = document.createElement("canvas");
    var canvasOff = document.createElement("canvas");

    canvas.width = canvasOff.width = video.videoWidth;
    canvas.height = canvasOff.height = video.videoHeight;

    ctx = canvas.getContext("2d");
    ctxOff = canvasOff.getContext("2d");

    // Make the canvas - not video player – visible
    poster.parentNode.removeChild(poster);
    document.body.appendChild(canvas);
  }

  renderOnCanvas();
}

function renderOnCanvas() {
  // Draw frame to canvas if video is still playing
  if (!video.paused && !video.ended) {

    // Draw original frame to offscreen canvas
    ctxOff.drawImage(video, 0, 0, canvas.width, canvas.height);

    // Manipulate frames offscreen
    var frame = getVideoFrame();

    // Draw new frame to visible video player
    ctx.putImageData(frame, 0, 0);
    requestAnimationFrame(renderOnCanvas);
  }
}

function getVideoFrame() {
  var img = ctxOff.getImageData
    (0, 0, canvas.width, canvas.height);

  // Invert the color of every pixel in the canvas context
  for (var i = 0, len = img.data.length; i < len; i += 4) {
    img.data[i] = 255 - img.data[i];
    img.data[i + 1] = 255 - img.data[i + 1];
    img.data[i + 2] = 255 - img.data[i + 2];
  }

  return img;
}
```

这个想法是在屏幕外播放视频，这意味着实际的视频播放器从未附加到 DOM。视频仍在播放，但浏览器从不需要将每一帧闪电般地显示在屏幕上（它只在内存中播放）。当每一帧播放时，我们将该帧绘制到画布上下文中（就像我们对图像做的那样），从画布上下文中获取像素，操纵像素数据，然后最终将其重新绘制到画布上。

由于视频只不过是一个接着一个播放的帧序列，给人以动画的错觉，我们可以从 HTML5 视频中提取每一帧，并像处理其他图像一样使用它与画布 API。由于没有办法绘制到视频元素，我们只需将视频播放器中的每一帧绘制到一个普通的画布对象中，就能达到相同的效果——但是像素经过精心设计。以下截图展示了这种技术的结果：

![Events](img/6029OT_07_04.jpg)

实现这一结果的一种方法是创建两个画布元素。如果我们只绘制到同一个画布上（绘制视频帧，然后处理该帧，然后绘制下一帧，依此类推），定制帧将只在屏幕上显示一小部分时间。只有在我们迅速绘制下一个传入帧之前才会可见。反过来，这个下一个帧只会在我们循环遍历该帧的像素数据并重新绘制该帧时才会可见。你明白了，结果会很混乱，一点也不是我们想要的。

因此，我们使用两个画布上下文。一个上下文负责仅显示我们正在处理的像素（也称为处理后的像素），另一个上下文对用户永远不可见，其目的是保存每一帧从视频中直接传来的像素。这样，我们每次迭代只在主画布上绘制一次，而在这个画布上显示的只有处理后的像素。原始像素（也称为内存中播放的原始视频的像素）将继续以尽可能快的速度流到离屏画布上下文。

# 地理位置

尽管 3D 图形很棒，基于套接字的多人游戏也很棒，但这两种技术都不一定是新的。另一方面，地理位置是一种较新的现象。有了它，我们能够使用 JavaScript 来确定用户的物理位置（地理位置）。拥有这样的工具使我们能够开发出令人惊叹的、高度创新的游戏概念。

现在，每当有一个新功能出现，承诺能够准确追踪用户的物理位置，大多数人（除了开发人员）都会对此感到至少有点害怕。毕竟，如果玩一个非常黑暗的生存恐怖游戏，知道其他玩家可以准确看到你的住址，那将是多么可怕。幸运的是，整个地理位置 API 都是基于用户选择的，这意味着用户会被提示应用程序尝试捕获用户的位置，只有当用户接受应用程序的请求时，浏览器才允许应用程序继续捕获用户的 GPS 位置。

如下截图所示，当尝试使用地理位置 API 时，浏览器会以某种方式向用户发出警报，并请求继续。如果用户决定不与应用程序共享他/她的位置，浏览器将不会与应用程序共享位置。

![地理位置](img/6029OT_07_05.jpg)

尽管每个浏览器在请求步骤上的实现略有不同，特别是关于如何向用户图形化传达此通知和请求的方式，但应用程序无法强制或秘密收集此信息。

```js
function getGeo(position) {
  var geo = document.createElement("ul");
  var lat = document.createElement("li");
  var lon = document.createElement("li");

  lat.textContent = "Latitude: " + position.coords.latitude;
  lon.textContent = "Longitude: " + position.coords.longitude;

  geo.appendChild(lat);
  geo.appendChild(lon);
  document.body.appendChild(geo);
}

function doOnPermissionDenied(message) {
  var p = document.createElement("p");

  p.textContent = "Permission Denied Error: " + message;
  document.body.appendChild(p);
}

function doOnPositionUnavailable(message) {
  var p = document.createElement("p");

  p.textContent = "Position Unavailable Error: " + message;
  document.body.appendChild(p);
}

function doOnTimeout(message) {
  var p = document.createElement("p");

  p.textContent = "Operation Timeout Error: " + message;
  document.body.appendChild(p);
}

function doNoGeo(positionError) {
  switch (positionError.code) {
    case positionError.PERMISSION_DENIED:
      doOnPermissionDenied(positionError.message);
      break;

    case positionError.POSITION_UNAVAILABLE:
      doOnPositionUnavailable(positionError.message);
      break;

    case positionError.TIMEOUT:
      doOnTimeout(positionError.message);
      break;
  }
}

// Ask the user if you may use Geolocation
navigator.geolocation.getCurrentPosition(getGeo, doNoGeo);
```

API 的第一部分涉及请求用户允许获取他/她的位置。这是通过在全局 navigator 对象的`geolocation`属性上调用`getCurrentPosition`函数来完成的。该函数接受两个参数，即一个回调函数，如果用户允许浏览器共享用户的位置，则调用该函数，以及一个回调函数，如果用户拒绝应用程序的请求，则调用该函数。

如果用户接受了应用程序的请求来共享地理位置，回调函数将被调用，并传入一个`Geoposition`对象。该对象有*九*个我们可以使用的属性：

+   `timestamp`: 回调函数被调用时

+   `coords`: 一个`Coordinates`类的实例

+   `accuracy`: GPS 坐标的准确度（以米为单位）

+   `altitude`: 以米为单位

+   `altitudeAccuracy`: 海拔的准确度（以米为单位）

+   `heading`: 以顺时针方向的度数

+   `latitude`: 作为双精度

+   `longitude`: 作为双精度

+   `speed`: 以米/秒为单位

位置对象中只有三个属性是必须存在的。这些是`纬度`和`经度`值，以及`精度`属性。如果使用的硬件支持，所有其他值都是可选的并且可用。还要记住，这个功能在移动设备上同样可用，因此用户的位置在应用程序使用过程中可能会有所变化。幸运的是，一旦用户同意与应用程序共享他或她的位置，任何后续调用获取当前位置的操作都将立即成功。当然，用户也可以从浏览器中清除对特定域的权限，因此任何后续获取位置的调用可能会失败（如果用户已经完全禁用了该功能），或者导致新的权限请求（如果用户只是清除了浏览器上的权限缓存）。

从下面的屏幕截图中可以看出，当页面使用地理位置时，谷歌浏览器在地址栏上显示不同的图标，以通知用户。通过点击这个特殊的图标，用户可以重置权限，或者在更长时间的基础上阻止或允许应用程序。

![地理位置](img/6029OT_07_06.jpg)

## 一个谷歌地图示例

如今，地理位置最常见的用例可能涉及将位置呈现到地图上。幸运的是，谷歌提供了一个出色的免费 API，我们可以利用它来实现这一目的。通过这个地图服务，我们可以捕获用户的地理位置，然后在地图上渲染一个标记，就在用户所在的位置（或者在用户所在位置的精度距离内的某个地方）。虽然谷歌地图 API 相当强大，但我们只会简单介绍如何获取用户的位置，然后在地图上呈现该坐标点的一个相当琐碎的例子。

地图 API 的基本思想很简单：创建一个地图对象，将其呈现在某个 HTML 容器对象内，指定地图的中心位置（以便我们知道地图中用户立即可见的一般区域），并在地图上添加标记。标记对象至少需要两个属性，即对地图对象的引用和 GPS 坐标点。在我们的示例中，我们将把地图的中心放在用户的 GPS 坐标上，并在同一位置放置一个标记。

```js
// Step 1: Request permission to get the user's location
function initGeo() {
  navigator.geolocation.getCurrentPosition(renderToMap, doNoGeo);
}

// Step 2: Render the user's location on a map
function renderToMap(position) {
  var container = document.createElement("div");
  container.id = "myContaier";
  container.style.width = window.innerWidth + "px";
  container.style.height = window.innerHeight + "px";

  document.body.appendChild(container);

  // Define some point based on a GPS coordinate
  var coords = new google.maps.LatLng(
    position.coords.latitude,
    position.coords.longitude);

  // Specify how we want the map to look
  var options = {
    zoom: 16,
    center: coords,
    mapTypeControl: false,
    mapTypeId: google.maps.MapTypeId.ROADMAP
  };

  // Create a map, and inject it into the DOM element referenced
  var map = new google.maps.Map(container, options);

  // Create a marker and associate it with our map
  var marker = new google.maps.Marker({
    position: coords,
    map: map,
    title: "Where's me?"
  });
}
```

虽然前面的例子可能不是你见过的最激动人心的软件，但它很好地说明了两个重要的观点。首先，地理位置 API 很强大，但也可能是所有其他 HTML5 API 中最容易使用的，因为它提供了所有功能和你需要知道的一切。其次，前面的片段展示了 Web 平台是多么开放，以及我们可以通过利用他人的工作来实现多少潜力。

运行前面的代码将导致一个非常漂亮的地图覆盖整个屏幕，地图的中心点是用户当前的位置，如下面的屏幕截图所示。请记住，谷歌地图只是许多免费 API 中的一个例子，我们可以与地理位置等强大的 HTML5 功能一起使用。

![一个谷歌地图示例](img/6029OT_07_07.jpg)

# 即将推出的 CSS 功能

我最喜欢的关于开放网络的事情之一是它也是一个活跃的网络。随着新的想法的出现和新的需求的显现，新功能被引入到规范中只是时间问题。CSS 就是一个完美的例子，最近规范中添加了一些新功能。最重要的是，大多数浏览器供应商都非常积极地将这些新功能引入到他们的浏览器中。

在接下来的部分中，我们将介绍 CSS 的三个新功能，即 CSS 着色器、CSS 列和 CSS 区域和排除。为了让您了解这些功能的开发活跃程度，我们将讨论第一个功能**CSS 着色器**，它最近更名为 CSS 自定义滤镜。谈论一个快速发展的开发生命周期。

## 在最前沿编程

尽管本书中的大部分内容都是新的和最先进的，但到目前为止，讨论的大多数 HTML5 功能和 API 都是相当稳定的。我的意思是，几乎任何主要的浏览器都应该能够处理这些功能而不会出现任何问题。然而，以下 CSS 功能刚刚出炉。更具体地说，这三个功能仍在烘烤中，配方正在不断完善，直到达到更稳定的水平。

有了这个说法，这一部分可能需要您使用绝对最新的浏览器，使用最新的可能版本，甚至可能需要您深入您选择的浏览器的设置部分，以便设置任何高级标志，以便这些新的实验性功能能够工作。本章的所有代码示例都是为 Google Chrome Canary（夜间构建）编写和测试的。在我写这篇文章时，安装 Google Chrome Canary 后，必须手动启用以下标志：

+   启用`实验性 WebKit 功能`

+   启用`CSS 着色器`

您可能不需要启用`WebGL`标志，因为这个特定的标志已经默认启用了一段时间，但是如果该标志被禁用，您可以以相同的方式使其可用。要查看可以在 Google Chrome 上设置的所有可用标志，只需在浏览器的地址栏中输入以下命令（通常在那里输入网站的 URL）：`chrome://flags`。

在标志页中，您将看到一个标志列表，以及每个标志的描述。查找与`实验性 WebKit 功能`和`CSS 着色器`相关的两个标志，并确保它们已启用。如下截图所示，要注意的是，粗心地设置和取消标志可能会影响 Google Chrome 的行为和性能。确保更改最少的标志，以避免使浏览器的工作不够理想，并确保跟踪您更改的任何标志，以便在发生任何不良情况时可以恢复更改。

在最前沿编程

关于使用这些绝对最新的实验性 API 进行开发的最后一点说明是，由于实验性 API 的性质，不同浏览器之间可能存在特定的语法和功能，以及显著的性能差异。由于并非所有浏览器同时开始采用新的 API，因此很大一部分用户无法查看您的最新和最棒的代码，直到 API 变得足够稳定——有时需要的时间比我们希望的长。

## CSS 着色器

目前，这是 CSS 中添加的绝对最新功能。CSS 着色器背后的最初想法是允许设计师使用 GLSL 着色器来渲染任意 HTML 元素。现在，我们不仅可以指定元素的背景颜色、边框样式、框阴影等，还可以处理元素的每个像素是如何渲染的。

最近，这个功能已经合并到现有的 CSS 滤镜规范中，该规范规定了一些预先制作的滤镜，我们可以应用到一个元素上。例如，我们可以将模糊滤镜应用到图像元素上，让浏览器在从服务器传送到 Web 应用程序时动态处理图像。然而，我们现在不仅仅依赖于浏览器决定使用哪些滤镜，而是可以自己制作滤镜，并让 CSS 渲染引擎使用它们。因此，这个新的 CSS API 的当前名称（无论如何）是**自定义 CSS 滤镜**。

使用 CSS 滤镜非常容易。毕竟，它们只是一个常规的 CSS 属性。截至目前，我们可以应用*九种*不同的滤镜，不仅适用于图像，还适用于任何可以接收 CSS 样式的东西。如果将滤镜添加到具有一个或多个子节点的元素中，正如 CSS 的性质一样，滤镜效果将传播到任何和所有子元素，除非其中一个或多个指定了自己的滤镜，或者故意指定不应该对其和其子元素应用任何滤镜。

CSS 滤镜的当前列表如下：

+   `blur`：应用高斯模糊

+   `brightness`：通过应用更多或更少的白色颜色来增加元素的亮度

+   `contrast`：调整元素的对比度

+   `drop-shadow`：对元素应用阴影效果

+   `grayscale`：将元素的颜色转换为灰度

+   `hue-rotate`：根据颜色圆对元素应用色相旋转

+   `invert`：反转元素的颜色

+   `opacity`：对元素应用透明度

+   `saturate`：增加元素的饱和度

+   `sepia`：将元素的颜色转换为棕褐色

请记住，尽管这些滤镜实际上只是 CSS 属性，但实际上它们是浏览器在 CSS 查询匹配的元素上执行的单独函数。因此，每个滤镜函数都需要一个或多个参数，在幕后，这些参数是传递给预定义的着色器程序的变量。

```js
<style>
div {
  margin: 10px;
  padding: 0;
  border: 1px solid #ddd;
  background: #fafafa;
  width: 400px;

  transition: all 3.3s;
  filter: invert(1);
}

div:hover {
  -webkit-filter: invert(0) blur(3px) contrast(150%);
}

h2 {
  margin: 0;
  padding: 10px;
  font-size: 4.75em;
  color: #aaa;
  text-shadow: 0 -1px 0 #555, 0 1px 0 #fff;
}
</style>

<div>
  <h2>CSS Filters</h2>
  <img src="img/strawberry.jpg" width="400" height="350" />
</div>
```

在下面的屏幕截图中，左侧的图像是一个常规的 HTML 元素，带有一个标题和一个图像。在右侧，我们应用了一个 CSS 滤镜，反转了颜色。整个效果是用一行代码实现的。

![CSS 着色器](img/6029OT_07_14.jpg)

请注意，我们可以通过简单地将其他滤镜列为 CSS 属性的值来将多个滤镜应用于同一元素。此外，请记住，即使只需一行代码就可以将这些令人兴奋的滤镜之一添加到我们的应用程序中，每个使用的滤镜都意味着浏览器需要在其已经在做的所有工作之上进行更多的工作。因此，我们使用这些滤镜越多，我们就可以预期性能相应地下降。

### 使用自定义滤镜

为了在渲染我们的应用程序时输入自己的过滤函数，我们需要创建执行我们想要的操作的着色器程序。值得庆幸的是，这些着色器程序是用我们在 WebGL 中使用的相同的着色语言编写的。如果你认为学习 JavaScript、CSS 和 HTML 已经是很多工作了，我很抱歉地说，但是请继续将 GLSL 添加到你必须掌握的语言列表中（或者找到已经掌握它的人），以充分利用 HTML5 革命。

要指定用于 CSS 滤镜的自定义着色器，我们只需将自定义函数作为 filter 属性的值调用，传入我们的顶点和片段着色器，然后是顶点着色器要使用的任何可能的变量。片段着色器使用的外部变量是从顶点着色器传入的，因此我们无法直接从 CSS 中传入任何内容。

```js
div {
  margin: 10px;
  padding: 0;
  border: 1px solid #ddd;
  background: #fafafa;
  width: 400px;

  filter: custom(url(simple-vert-shader.glsl)
    mix(url(simple-frag-shader.glsl) normal source-atop,
    16 32,
    lightPosition 0.0 0.0 1.0;
}
```

前面的滤镜定义有三个部分。首先，我们调用`custom`表示我们将使用自己的着色器。我们传递给这个函数的第一个参数是顶点着色器。这个文件的扩展名并不重要，因为文件的内容将被编译并发送到 GPU。很多时候，你会看到其他开发人员为他们的着色器使用文件扩展名，比如`.glsl`或`.vs`和`.fs`（分别用于顶点着色器和片段着色器）。请注意，片段着色器通过`mix()`函数发送，而不是直接通过`url()`函数发送，这与顶点着色器的情况不同。最后，我们指定将构成元素内容网格的行数和列数。构成这个网格的顶点是浏览器自动创建的。最后，与我们自定义滤镜一起传递的最后一组参数是顶点着色器使用的 uniform 值（附带它们的名称）。

由于 GLSL 本身超出了本书的范围，我们将避免对这些自定义着色器进行彻底的示例。相反，我们将看一个象征性的例子，它将使用虚拟着色器。如果没有正确的背景知识和图形编程、着色器编程和其他 3D 图形主题的经验，解释自定义着色器程序将是相当具有挑战性的。

以下着色器程序从 CSS 中获取三个输入，即表示图像中每个像素应用的红色、绿色和蓝色的量的值，介绍 OpenGL 着色语言（GLSL）的快速简要入门课程，我只想说：uniform 就像是一个全局变量，我们可以传递给顶点着色器。顶点着色器每个顶点调用一次，并确定每个顶点的位置。为了将值发送到片段着色器，顶点着色器可以使用 varying 变量。如果我们在顶点着色器中定义了一个带有`varying`关键字的任何类型的变量，这意味着分配给它的任何值将可供片段着色器使用，前提是片段着色器还定义了相同名称和类型的 varying 变量。因此，如果我们希望从 CSS 直接将一个值传递到片段着色器，我们可以简单地将值发送到顶点着色器，然后使用`varying`将该值传递到片段着色器。片段着色器每个像素调用一次，并确定要应用于该像素的颜色。

```js
// ----------------------------------------------------
// Vertex shader: simple-vert-shader.glsl
// ----------------------------------------------------
precision mediump float;

// Built-in attribute
attribute vec4 a_position;

// Built-in uniform
uniform mat4 u_projectionMatrix;

// Values sent in from CSS
uniform float red;
uniform float green;
uniform float blue;

// Send values to fragment shader
varying float v_r;
varying float v_g;
varying float v_b;

void main() {

  v_r = red;
  v_g = green;
  v_b = blue;

  // Set the position of each vertex
  gl_Position = u_projectionMatrix * a_position;
}
```

前面的顶点着色器所做的只有两件事：将我们的值从 CSS 传递到片段着色器，并设置内容网格上每个顶点的顶点位置。

```js
// ----------------------------------------------------
// Vertex shader: simple-vert-shader.glsl
// ----------------------------------------------------
precision mediump float;

// Input from vertex shader
varying float v_r;
varying float v_g;
varying float v_b;

void main() {

  // Set the color of each fragment
  css_ColorMatrix = mat4(v_r, 0.0, 0.0, 0.0,
    0.0, v_g, 0.0, 0.0,
    0.0, 0.0, v_b, 0.0,
    0.0, 0.0, 0.0, 1.0);
}
```

有了这个着色器程序，我们只需要在 HTML 文件中调用它。我们需要注意的三个参数是红色、绿色和蓝色的 uniform 值。无论我们为这三个颜色通道发送什么值，它都会反映在我们应用这个滤镜的任何元素的渲染上。

```js
<style>
div {
  margin: 10px;
  padding: 0;
  border: 1px solid #ddd;
  background: #fafafa;
  width: 400px;

  /**
   * We can leverage CSS transitions to make our simple
   * shaders seem even more impressive
   */
  transition: filter 1.0s;

  filter: custom(url(simple-vert-shader.glsl)
    mix(url(simple-frag-shader.glsl)
    normal source-atop),
    16 32,
    red 1.0, green 0.0, blue 0.0);
}

div:hover {
  filter: custom(url(simple-vert-shader.glsl)
    mix(url(simple-frag-shader.glsl)
    normal source-atop),
    16 32,
    red 1.0, green 1.0, blue 0.0);
}

h2 {
  margin: 0;
  padding: 10px;
  font-size: 4.75em;
  color: #aaa;
  text-shadow: 0 -1px 0 #555, 0 1px 0 #fff;
}
</style>

<div>
  <h2>CSS Filters</h2>
  <img src="img/strawberry.jpg" width="400" height="350" />
</div>
```

有了这个设置，我们的`div`元素将默认以一种特定的方式呈现。在这种情况下，我们只在 DOM 节点内的每个像素上打开红色通道。然而，当我们悬停在元素上时，我们应用相同的着色器，但颜色完全不同。这次我们让每个像素看起来更加黄色。借助 CSS 过渡，我们可以平滑地过渡这两种状态，产生一个简单而非常舒适的效果。当然，您对 GLSL 了解得越多，您就可以使这些自定义着色器变得更加花哨和强大。而且作为额外的奖励，我们不必担心在 WebGL 中使用着色器所涉及的所有设置工作。浏览器提供的默认抽象非常有用，使得自定义着色器非常可重用，因为使用我们的着色器的人只需要跟踪几个 CSS 属性。最重要的是，由于着色器程序在这个 CSS 级别上至少是纯文本文件，我们可以通过检查其源代码来了解其他人的着色器是如何工作的。通过使用我们的自定义着色器，我们可以轻松地控制哪些颜色通道在单个像素级别上打开或关闭，如下面的屏幕截图所示。这种像素级别的操作不仅限于图像，而是在我们将滤镜应用于的每个 DOM 元素的每个像素上执行。文字、图像、容器等。

![使用自定义滤镜](img/6029OT_07_15.jpg)

然而，请注意，由于这项技术是最新的，几乎没有工具可以帮助我们开发、调试和维护 GLSL 着色器。您很快会注意到，当在您的着色器中发现错误时，您将只会看到一个未经过滤的 HTML 文档。例如，如果您的着色器程序无法编译，浏览器将不会告诉您发生了什么，或者在哪里，甚至可能为什么。因此，编写自定义 CSS 滤镜可能是目前网页开发中最具挑战性的方面，因为浏览器尚未在这个过程中提供很有用的帮助。

## CSS 列

如果您至少使用互联网几周，或者至少看过几十个不同的网站，您肯定会注意到 HTML 文档的矩形特性。虽然可以使用 HTML、JavaScript 和 CSS 的组合来创建非常健壮的设计，但网页设计师已经等待了很长时间，以寻找一个简单的解决方案来创建多列设计。

通过新的 CSS 列功能，我们可以创建一个常规的文本块，然后告诉 CSS 引擎将该块显示为两列或更多列。其他所有事情都由浏览器非常高效地处理。例如，假设我们希望将一个文本块显示为四个等宽的列，每列之间间隔 20 像素。这可以通过两行直观的代码实现（可能需要供应商前缀，但在这个例子中被故意忽略）。

```js
<style>
div {
  column-count: 4;
  column-gap: 20px;
</style>

<div>
  <p>Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna aliquam erat volutpat. Ut wisi enim ad minim veniam, quis nostrud exerci tation ullamcorper suscipit lobortis nisl ut aliquip ex ea commodo consequat. Duis autem vel eum iriure dolor in hendrerit in vulputate velit esse molestie consequat, vel illum dolore eu feugiat nulla facilisis at vero eros et accumsan et iusto odio dignissim qui blandit praesent luptatum zzril delenit augue duis dolore te feugait nulla facilisi.</p>

  <p>Nam liber tempor cum soluta nobis eleifend option congue nihil imperdiet doming id quod mazim placerat facer possim assum. Typi non habent claritatem insitam; est usus legentis in iis qui facit eorum claritatem. Investigationes demonstraverunt lectores legere me lius quod ii legunt saepius.</p>

  <p>Claritas est etiam processus dynamicus, qui sequitur mutationem consuetudium lectorum. Mirum est notare quam littera gothica, quam nunc putamus parum claram, anteposuerit litterarum formas humanitatis per seacula quarta decima et quinta decima. Eodem modo typi, qui nunc nobis videntur parum clari, fiant sollemnes in futurum.</p>
</div>
```

通过上述设置，浏览器知道我们希望将文本渲染成四列，每列之间间隔 20 像素。请注意，从来没有提到过每列的宽度。在这种情况下，浏览器计算出`div`容器内的可用空间，减去列间隙所需的总宽度（两列之间的空间，不包括列与容器之间的空间），然后将剩余宽度分成总列数。这样，当我们调整浏览器窗口大小时，列将自动调整大小，其他所有内容将保持其尺寸。

在我们指定列间距宽度之后，浏览器可以根据列的可用空间确定每一列的宽度（如果我们指定了固定数量的列），或者确定要显示的列数（如果我们为每一列指定了宽度），如下面的屏幕截图所示。通常情况下，指定列宽和列数是没有意义的。

![CSS 列](img/6029OT_07_09.jpg)

或者，我们可以简单地告诉浏览器我们希望每列有多宽，以及两列之间有多少间隙。在这种情况下，浏览器会做相反的事情。它会计算剩余的可用空间来呈现列，然后在给定我们指定的宽度约束的情况下，尽可能多地呈现列。

```js
<style>
div {
  column-width: 200px;
  column-gap: 20px;
</style>
```

### 列规则

与围绕在盒子周围的边框的概念类似，如 border: 1px solid #333，CSS 列带有规则的概念。简单地说，列规则是在两列之间垂直绘制的单个边框。规则可以像边框一样进行样式设置，并且在两列之间正确渲染，利用列间隙提供的空间。如果列规则的可用空间大于列间隙提供的空间，间隙将被正确渲染，规则将被忽略。

```js
<style>
div {
  column-count: 3;
  column-gap: 20px;
  column-rule-width: 1px;
  column-rule-style: dashed;
  column-rule-color: rgb(255, 10, 10);
</style>
```

同样，类似于边框属性，我们可以指定与列规则相关的每个属性，或者按照与边框相同的顺序简写定义（宽度、样式和颜色）。边框样式的有效值包括以下内容：

+   `none`: 无边框

+   `dotted`: 边框是一系列点

+   `dashed`: 边框是一系列短线段

+   `solid`: 边框是单一线段

+   `double`: 边框是两条实线。两条线和它们之间的空间之和等于'border-width'的值

+   `groove`: 边框看起来像是雕刻在画布上

+   `ridge`: 与'groove'相反：边框看起来像是从画布中出来的

### 注意

有关表格边框样式的更多信息，您可以访问[`www.w3.org/TR/CSS2/tables.html#table-border-styles`](http://www.w3.org/TR/CSS2/tables.html#table-border-styles)

### 列断

有时，我们可能希望对内容在哪里断开成新的列有一些控制。例如，如果我们有几个文本块，每个文本块前面都有某种标题。如果列的最后一行是一个孤立的标题，用来介绍下一节，那看起来可能不太好。列断属性给了我们这种能力，我们可以在元素之前或之后指定列断。

通过指定列应该在何处断开成下一列，我们可以更好地控制每列的呈现和填充，如下截图所示：

![列断](img/6029OT_07_10.jpg)

在 CSS 中用于控制分页的相同属性也用于控制列的断开。我们可以使用三个属性来控制列断，即`break-before`、`break-after`和`break-inside`。前两个属性相当直观——我们可以使用 break before 或 after 来指示特定元素之前或之后的行为，例如总是断开列、永不断开，或者在应该正常插入的地方插入列断。另一方面，break inside 指定多行文本内部的行为，而不仅仅是在其开始或结束处。

```js
<style>
div {
  -webkit-column-count: 3;
  -webkit-column-gap: 20px;
  -webkit-column-rule: 1px solid #fff;
  padding: 20px;
  margin: 10px;
  background: #eee;
}

div p {
  margin: 0 0 10px;
 -webkit-column-break-inside: auto;
}

div h2 {
  margin: 0 0 10px;
  color: #55c;
  text-shadow: 0 1px 0 #fff;
 -webkit-column-break-before: always;
}
</style>

<div>
  <h2>Lorem Ipsum</h2>
  <p>Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna aliquam erat volutpat. Ut wisi enim ad minim veniam, quis nostrud exerci tation ullamcorper suscipit lobortis nisl ut aliquip ex ea commodo consequat. Duis autem vel eum iriure dolor in hendrerit in vulputate velit esse molestie consequat, vel illum dolore eu feugiat nulla facilisis at vero eros et accumsan et iusto odio dignissim qui blandit praesent luptatum zzril delenit augue duis dolore te feugait nulla facilisi.</p>

  <h2>Nam Liber Tempor</h2>
  <p>Nam liber tempor cum soluta nobis eleifend option congue nihil imperdiet doming id quod mazim placerat facer possim assum. Typi non habent claritatem insitam; est usus legentis in iis qui facit eorum claritatem. Investigationes demonstraverunt lectores legere me lius quod ii legunt saepius.</p>

  <h2>Claritas est etiam</h2>
  <p>Claritas est etiam processus dynamicus, qui sequitur mutationem consuetudium lectorum. Mirum est notare quam littera gothica, quam nunc putamus parum claram, anteposuerit litterarum formas humanitatis per seacula quarta decima et quinta decima. Eodem modo typi, qui nunc nobis videntur parum clari, fiant sollemnes in futurum.</p>
</div>
```

请注意，列断属性应用于`h2`标签，现在它成为控制每列断的元素。当然，如果我们在这个文本块中指定的列数比标题标签多，那么段落标签中的文本显然会分成新的列。这种行为也可以被控制，尽管在这种情况下，我们只是将`column-break-inside`属性设置为`auto`，明确表示我们希望每个段落标签的内容如果需要的话可以溢出到其他列中。

### CSS 区域和排除

CSS 的两个新的与文本相关的特性是区域和排除。区域的行为与列有些相似，因为我们指定了特定文本块的呈现和流动方式。区域和列之间的主要区别在于，列被限制为等宽的隐含矩形，而区域指定了一个单独的内容源，并定义了该内容的流动位置。例如，我们可以告诉 CSS 将来自给定源的文本呈现到三个独立的`div`元素中，以及一个任意的 SVG 多边形。这些元素中的每一个都不需要以任何特定的方式相关联 - 一个可以是绝对定位的，一个可以被转换，等等。然后文本将从一个元素流向下一个元素，按照每个元素在 HTML 文件中定义的顺序。另一方面，排除则完全相反。它不是定义文本流入的区域，而是描述文本应该绕过的区域或形状。

这两个分开但又密切相关的 API 的整个原因是推动我们可以将 Web 应用程序的视觉设计推向何方。直到现在，实现这种效果的唯一方法是通过外部软件，希望有一个非常特定的插件，允许在浏览器内执行这样的软件或技术。现在浏览器已经变得更加成熟，我们可以直接从样式表中实现这些类似杂志的效果。

#### 区域

区域的工作方式与列有些相似，但基本上是不同的。总的来说，区域所做的就是指定一个内容源，然后将 CSS 表达式分配为该内容的目的地。内容从指定为源的元素移动，并流入所有分配为目的地的元素。如果一个或多个元素由于内容不足而没有接收到任何内容，这些元素将表现得就像一个普通的*空*元素一样。除了将元素标识为目的地的 CSS 属性之外，该元素与任何其他常规 HTML 元素没有任何不同。

```js
<style>
h2, p {
  margin: 0 0 10px;
}

#src {
  flow-into: mydiv;
}

.container {
  flow-from: mydiv;

  border: 1px solid #c00;
  padding: 0.5em;
  margin: 0.5em;
}

.col1, .col2, .col3 {
  float: left;
  width: 50%;
}

#one {
  height: 250px;
}

#two, #three {
  height: 111px;
}

.col3 {
  clear: both;
  width: 100%;
}
</style>

<div id="src">
  <h2>Lorem Ipsum</h2>
  <p>Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna aliquam erat volutpat. Ut wisi enim ad minim veniam, quis nostrud exerci tation ullamcorper suscipit lobortis nisl ut aliquip ex ea commodo consequat. Duis autem vel eum iriure dolor in hendrerit in vulputate velit esse molestie consequat, vel illum dolore eu feugiat nulla facilisis at vero eros et accumsan et iusto odio dignissim qui blandit praesent luptatum zzril delenit augue duis dolore te feugait nulla facilisi.</p>

  <h2>Nam Liber Tempor</h2>
  <p>Nam liber tempor cum soluta nobis eleifend option congue nihil imperdiet doming id quod mazim placerat facer possim assum. Typi non habent claritatem insitam; est usus legentis in iis qui facit eorum claritatem. Investigationes demonstraverunt lectores legere me lius quod ii legunt saepius.</p>

  <h2>Claritas est etiam</h2>
  <p>Claritas est etiam processus dynamicus, qui sequitur mutationem consuetudium lectorum. Mirum est notare quam littera gothica, quam nunc putamus parum claram, anteposuerit litterarum formas humanitatis per seacula quarta decima et quinta decima. Eodem modo typi, qui nunc nobis videntur parum clari, fiant sollemnes in futurum.</p>
</div>

<div class="col1">
  <div class="container" id="one"></div>
</div>
<div class="col2">
  <div class="container" id="two"></div>
  <div class="container" id="three"></div>
</div>
<div class="col3">
  <div class="container" id="four"></div>
</div>
```

在这里，我们将具有`id`属性为`src`的元素的内容分配为内容提供者，可以这么说。这是通过分配新的 CSS 属性`flow-into`来完成的，该属性分配了一个字符串，我们可以用它来标识这个特定的区域内容源。这意味着该元素的内容不会在 DOM 中呈现，而是会分布在所有具有`flow-from` CSS 属性的元素中，其值与具有`flow-into`属性的元素使用的关键字匹配。

```js
#src {
  flow-into: description-text;
}

div.description {
  flow-from: description-text;
}
```

一旦定义了区域源，并创建了区域链，浏览器就会负责将内容分发到所有区域中。每个区域都可以有独特的样式，也可以是一个独特的元素。例如，可以定义一个区域源并创建两个目标。一个目标可以是标准的`div`元素，另一个可以是 SVG 形状。CSS 区域还可以与排除相结合，我们将在下一节讨论。

如下截图所示，四个元素被样式化并浮动，同时一个区域源负责填充这些区域。在区域调整大小的情况下，由于浏览器窗口本身被调整大小，用户代理会负责刷新内容，流入新调整大小的区域。

![区域](img/6029OT_07_13.jpg)

### 排除

排除的工作方式与我们通常使文本围绕图像或任何其他内联元素流动的方式非常相似。主要区别在于，我们可以进一步指定一些 CSS 细节，告诉文本如何流动。

```js
<style>
img {
  width: 300px;
  height: 60px;
  display: inline-block;
  float: left;
}
</style>

<div>
  <img src="img/lipsum-logo.png" />
  <h2>Lorem Ipsum</h2>
  <p>Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna aliquam erat volutpat. Ut wisi enim ad minim veniam, quis nostrud exerci tation ullamcorper suscipit lobortis nisl ut aliquip ex ea commodo consequat. Duis autem vel eum iriure dolor in hendrerit in vulputate velit esse molestie consequat, vel illum dolore eu feugiat nulla facilisis at vero eros et accumsan et iusto odio dignissim qui blandit praesent luptatum zzril delenit augue duis dolore te feugait nulla facilisi.</p>
</div>
```

这个琐碎的片段只是告诉`div`元素内的其余内容围绕图像的右侧流动。即使我们在那个图像的位置放置了一个 SVG 对象，而这个对象是一个指向右侧的三角形形状的多边形，文本也会围绕该对象进行换行，将其视为矩形。

然而，通过 CSS 排除的魔力，我们可以向图像标签或 SVG 对象添加属性，以改变其外部形状的解释方式。默认情况下，由于任何 HTML 元素都有 x 和 y 位置，以及`width`和`height`属性，每个元素都被视为一个矩形。使用形状属性会改变这一点。

```js
<style>
h2, p {
  margin: 0 0 10px;
}

svg {
  float: left;
  width: 300px;
  height: 400px;
 shape-outside: polygon(0 0, 100% 50%, 0 100%);
}

svg polygon {
  fill: #c33;
}
</style>

<div>
  <svg >
<polygon points="0, 0, 300, 200, 0, 400"></polygon></svg>

  <h2>Lorem Ipsum</h2>
  <p>Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna aliquam erat volutpat. Ut wisi enim ad minim veniam, quis nostrud exerci tation ullamcorper suscipit lobortis nisl ut aliquip ex ea commodo consequat. Duis autem vel eum iriure dolor in hendrerit in vulputate velit esse molestie consequat, vel illum dolore eu feugiat nulla facilisis at vero eros et accumsan et iusto odio dignissim qui blandit praesent luptatum zzril delenit augue duis dolore te feugait nulla facilisi.</p>
</div>
```

关于 CSS 排除的一个棘手之处是它只是定义了文本流动的形状或路径，而不一定是要呈现的形状或路径。换句话说，前面代码示例中突出显示的两行代码是完全独立的。这两个多边形定义之所以如此相似，只是为了视觉效果。如果我们在文本块中使用了图像、`div`或任何其他 HTML 元素，CSS 的`shape-outside`属性仍然会导致文本以相同的方式围绕该元素流动，无论该元素具有什么物理形状。仅仅添加 CSS 的`shape`属性到一个元素并不会改变它自己的视觉属性。

运行前面的代码示例会产生类似以下截图的输出。再次记住，文本遵循的路径与显示的元素形状之间的关系，即不允许文本进入的形状，纯粹是巧合和有意为之。如果我们不是一个 SVG 多边形，而是一个图像元素，文本仍然会遵循那个箭头形状，但是矩形图像会浮在遵循与图像边界相交路径的任何文本上方。严格来说，排除只涉及文本在给定文本块内的流动方式。文本沿着路径的任何东西是否被呈现，取决于设计师，这是排除之外的一个单独问题，如下图所示：

![排除](img/6029OT_07_11.jpg)

如果最终目标只是简单地定义文本要遵循的路径，就像前面的例子一样，我们不需要使用 SVG 或任何特定的 HTML 元素。只要有一个元素存在，并为该元素分配基本的浮动属性，排除就足够工作了。记住，排除的唯一重要部分是形状属性。

```js
<style>
.shape {
  display: inline-block;
  float: left;
  width: 300px;
  height: 400px;
  shape-outside: polygon(0 0, 100% 50%, 0 100%);
}
</style>

<div>
  <span class="shape"> </span>

  <h2>Lorem Ipsum</h2>
  <p>Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna aliquam erat volutpat. Ut wisi enim ad minim veniam, quis nostrud exerci tation ullamcorper suscipit lobortis nisl ut aliquip ex ea commodo consequat. Duis autem vel eum iriure dolor in hendrerit in vulputate velit esse molestie consequat, vel illum dolore eu feugiat nulla facilisis at vero eros et accumsan et iusto odio dignissim qui blandit praesent luptatum zzril delenit augue duis dolore te feugait nulla facilisi.</p>
</div>
```

或者，我们可以使用`shape-outside`的伴随属性，即`shape-inside`。直观地，这个属性定义了与其对应属性相反的作用。`shape-outside`属性告诉浏览器文本需要围绕（外部）的地方，而`shape-inside`属性告诉浏览器文本必须留在其中的区域。两个属性的所有属性值都是相同的。两个属性之间唯一的区别在于，在`shape-outside`中，文本被放置在占位元素的外部。而在`shape-inside`中，任何要在定义的形状内部引导的文本都被放置为形状元素的后代节点。

```js
<style>
.shape {
  display: block;
  width: 300px;
  height: 400px;
  shape-inside: polygon(0 0, 100% 50%, 0 100%);
}
</style>

<div>
  <h2>Lorem Ipsum</h2>
  <span class="shape">
    <p>Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna aliquam erat volutpat. Ut wisi enim ad minim veniam, quis nostrud exerci tation ullamcorper suscipit lobortis nisl ut aliquip ex ea commodo consequat. Duis autem vel eum iriure dolor in hendrerit in vulputate velit esse molestie consequat, vel illum dolore eu feugiat nulla facilisis at vero eros et accumsan et iusto odio dignissim qui blandit praesent luptatum zzril delenit augue duis dolore te feugait nulla facilisi.</p>
  </span>
</div>
```

与`shape-outside`相比，`shape-inside`属性将其自身的内容包含在内部，而`shape-outside`则只是一个其兄弟元素必须围绕的块，如下图所示：

![排除](img/6029OT_07_12.jpg)

最后，为了预料到这两个属性可能引发的问题，是的，我们很可能结合定义`shape-outside`属性的排除和定义`shape-inside`属性的排除。请注意，`shape-inside`排除只是一个块级元素，就像任何其他元素一样。在没有任何 CSS 指令的 HTML 文件的源代码中，`shape-inside`排除将无法与普通文本块区分开。因此，我们很可能将`shape-inside`排除的元素用作`shape-outside`排除。同一个元素可以具有两个 CSS 属性，因为它们的效果是互斥的。元素内的任何文本将与`shape-inside`排除声明绑定，而元素周围的任何内容将与`shape-outside`属性的效果相关联。

```js
<style>
h2, p {
  margin: 0 0 10px;
}

#wrap {
  width: 50%;
  height: 100%;
  float: left;

  shape-inside: polygon(0 0, 100% 50%, 0 100%);
  shape-outside: polygon(0 0, 100% 50%, 0 100%);
}
</style>

<div>
  <h2>Lorem Ipsum</h2>

  <div id="wrap">
    <p>Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna aliquam erat volutpat. Ut wisi enim ad minim veniam, quis nostrud exerci tation ullamcorper suscipit lobortis nisl ut aliquip ex ea commodo consequat. Duis autem vel eum iriure dolor in hendrerit in vulputate velit esse molestie consequat, vel illum dolore eu feugiat nulla facilisis at vero eros et accumsan et iusto odio dignissim qui blandit praesent luptatum zzril delenit augue duis dolore te feugait nulla facilisi.</p>
  </div>

  <h2>Nam Liber Tempor</h2>
  <p>Nam liber tempor cum soluta nobis eleifend option congue nihil imperdiet doming id quod mazim placerat facer possim assum. Typi non habent claritatem insitam; est usus legentis in iis qui facit eorum claritatem. Investigationes demonstraverunt lectores legere me lius quod ii legunt saepius.</p>

  <h2>Claritas est etiam</h2>
  <p>Claritas est etiam processus dynamicus, qui sequitur mutationem consuetudium lectorum. Mirum est notare quam littera gothica, quam nunc putamus parum claram, anteposuerit litterarum formas humanitatis per seacula quarta decima et quinta decima. Eodem modo typi, qui nunc nobis videntur parum clari, fiant sollemnes in futurum.</p>
</div>
```

### 定义形状

方便的是，形状属性的可能值与基本 SVG 形状相同。四种可用的形状是矩形、椭圆、圆和多边形。点值可以表示为长度值或百分比值。每种形状的语法非常一致，形式为`<shape>([value]{?})`。例如：

+   `rectangle(x, y, width, height)`: 定义一个尖锐的矩形，形状的左上角位于点 x，y 处

+   `rectangle(x, y, width, height, round-x, round-y)`: 定义一个矩形，并可以选择圆角

+   `ellipse(x, y, radius-x, radius-y)`: 定义一个以点 x，y 为中心的椭圆

+   `circle(x, y, radius)`: 定义一个给定半径的圆，以点 x，y 为中心

+   `polygon(p1-x p1-y, p2-x p2-y, (…))`: 给定三个或更多对 x，y 位置，定义一个多边形

# 总结

本章介绍了一些更复杂和尖端的 HTML5 API。主要亮点是新的 3D 渲染和图形编程 API—WebGL。我们还研究了 HTML5 的新视频播放能力，以及在浏览器上本地播放视频的每一帧的操作能力。最后，我们涉足了最新和最伟大的 CSS 改进和增加。这涉及到 CSS 着色器、列和区域以及排除等 API。

在下一章中，我们将通过深入研究使用 HTML5 进行移动网络开发来结束我们对 HTML5 游戏开发这个迷人世界的探索。我们将学习移动游戏开发与传统桌面应用程序开发的不同之处。我们还将学习两个 HTML5 API 来帮助我们。我们将构建一个完全适合移动设备的 2D 太空射击游戏来说明这些概念。
