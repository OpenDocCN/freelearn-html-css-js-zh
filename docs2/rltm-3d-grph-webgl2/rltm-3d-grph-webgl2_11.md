# 第十一章：WebGL 2 突出特点

在本书中，我们使用 WebGL 2 覆盖了计算机图形学的基础，这是所有现代浏览器都内置的基于网络的 3D 图形 API。我们了解到 WebGL 1 基于 OpenGL ES 2.0，而 WebGL 2 基于 OpenGL ES 3.0，这保证了 WebGL 1 中作为 *可选* 扩展提供的许多特性，以及许多其他强大的方法。尽管我们使用 WebGL 2 学习了广泛的计算机图形学主题，但几乎所有这些知识和技能都可以转移到其他图形 API 上。因此，让我们花一点时间来介绍 WebGL 2 相比 WebGL 1 提供的关键特性，以及从 WebGL 1 迁移到 WebGL 2 的策略。

在本章中，我们将涵盖以下内容：

+   对 WebGL 2 API 的更深入探讨

+   WebGL 2 核心规范的新增内容

+   从 WebGL 1 迁移 3D 应用到 WebGL 2 的策略

# WebGL 2 的新特性是什么？

截至 2016 年 1 月 27 日，WebGL 2 在 Firefox 和 Chrome 中默认可用。这意味着只要您使用以下浏览器之一，您将自动获得对 WebGL 2 的访问权限，而无需任何额外的依赖项：

+   Firefox 51 或更高版本

+   Google Chrome 56 或更高版本

+   Chrome for Android 64 或更高版本

WebGL 2 支持

要获取支持 WebGL 2 的浏览器列表的最新版本，请通过以下链接访问 Khronos Group 网站：[`www.khronos.org/WebGL/wiki/Getting_a_WebGL_Implementation`](http://www.khronos.org/webgl/wiki/Getting_a_WebGL_Implementation)。或者，您也可以访问知名的 **CanIUse.com** 资源：[`caniuse.com/#search=WebGL 2`](https://caniuse.com/#search=webgl2)。

如 第一章 所述，*入门指南*，WebGL 1 基于 OpenGL ES 2.0；因此，它不暴露查询计时器、计算着色器、统一缓冲区等特性。话虽如此，随着 WebGL 2（基于 OpenGL ES 3.0），我们能够访问更多 GPU 特性，如实例化和多个渲染目标。鉴于 WebGL 2 相比 WebGL 1 是一个相当大的升级，让我们突出其一些重要特性。

# 顶点数组对象

如 第二章 中所述，*渲染*，我们可以通过使用 `OES_vertex_array_object` 扩展在 WebGL 1 中实现 **顶点数组对象**。话虽如此，它们在 WebGL 2 中默认可用。这是一个重要的特性，应该始终使用，因为它可以显著减少渲染时间。当不使用顶点数组对象时，所有属性数据都在全局 WebGL 状态中，这意味着调用如 `gl.vertexAttribPointer`、`gl.enableVertexAttribArray` 和 `gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, buffer)` 等函数会操作全局状态。这会导致性能损失，因为在任何绘制调用之前，我们需要设置所有顶点属性并设置用于索引数据的 `ELEMENT_ARRAY_BUFFER`。另一方面，使用顶点数组对象时，我们会在应用程序初始化期间设置所有属性，并在渲染期间简单地绑定数据，从而获得更好的性能。

这与 DirectX 中的 `IDirect3DVertexDeclaration9`/`ID3D11InputLayout` 接口非常相似。

| WebGL 1 带扩展 | WebGL 2 |
| --- | --- |
| `createVertexArrayOES` | `createVertexArray` |
| `deleteVertexArrayOES` | `deleteVertexArray` |
| `isVertexArrayOES` | `isVertexArray` |
| `bindVertexArrayOES` | `bindVertexArray` |

例如：

```js
// Create a VAO instance
var vertexArray = gl.createVertexArray();

// Bind the VAO
gl.bindVertexArray(vertexArray);

// Set vertex array states

// Set with GLSL layout qualifier
const vertexPositionLocation = 0;
// Enable the attribute
gl.enableVertexAttribArray(vertexPositionLocation);
// Bind Buffer 
gl.bindBuffer(gl.ARRAY_BUFFER, vertexPositionBuffer);
// ...
// Configure instructions for VAO
gl.vertexAttribPointer(vertexPositionLocation, 2, gl.FLOAT, false, 0, 0);

// Clean
gl.bindVertexArray(null);
gl.bindBuffer(gl.ARRAY_BUFFER, null);

// ...

// Render
gl.bindVertexArray(vertexArray);
gl.drawArrays(gl.TRIANGLES, 0, 6);
```

# 更广泛的纹理格式范围

虽然 WebGL 1 有一组有限的纹理格式，但 WebGL 2 提供了更大的一组，其中一些列在这里：

| `RGBA32I` | `RG8` | `RGB16UI` |
| --- | --- | --- |
| `RGBA32UI` | `RG8I` | `RGB8_SNORM` |
| `RGBA16I` | `RG8UI` | `RGB8I` |
| `RGBA16UI` | `R32I` | `RGB8UI` |
| `RGBA8` | `R32UI` | `SRGB8` |
| `RGBA8I` | `R16I` | `R11F_G11F_B10F` |
| `RGBA8UI` | `R16UI` | `RGB9_E5` |
| `SRGB8_ALPHA8` | `R8` | `RG32F` |
| `RGB10_A2` | `R8I` | `RG16F` |
| `RGB10_A2UI` | `R8UI` | `RG8_SNORM` |
| `RGBA4` | `RGBA32F` | `R32F` |
| `RGB5_A1` | `RGBA16F` | `R16F` |
| `RGB8` | `RGBA8_SNORM` | `R8_SNORM` |
| `RGB565` | `RGB32F` | `DEPTH_COMPONENT32F` |
| `RG32I` | `RGB32I` | `DEPTH_COMPONENT24` |
| `RG32UI` | `RGB32UI` | `DEPTH_COMPONENT16` |
| `RG16I` | `RGB16F` |  |
| `RG16UI` | `RGB16I` |  |

# 3D 文本纹理

**3D 纹理** 是一种纹理，其中每个米普级别包含单个三维图像。3D 纹理本质上只是一个堆叠的 2D 纹理，可以在着色器中使用 `x`、`y` 和 `z` 坐标进行采样。这种功能使我们能够在单个对象中拥有多个 2D 纹理，以便着色器可以无缝选择每个对象使用的图像。

这对于可视化体数据（如医学扫描）、3D 效果（如烟雾）、存储查找表等非常有用。

# 纹理数组

**纹理数组**，类似于 3D 纹理，是一个减少复杂性、提高代码可维护性以及增加可使用纹理数量的优秀特性。通过确保纹理数组中所有纹理切片的大小相同，着色器可以访问许多纹理，同时占用更小的空间。

# 实例渲染

在 WebGL 2 中，**实例化**或**实例渲染**默认可用**。实例渲染是一种连续多次执行相同的绘制命令的方法，每次产生略微不同的结果。这对于使用非常少的 API 调用渲染大量几何体来说是一个非常有效的方法。

实例化是某些类型几何体性能的极大提升，特别是那些实例数量多但顶点数量不多的对象。好的例子是草地和毛发。实例化避免了每个对象单独 API 调用的开销，同时通过避免为每个单独的实例存储几何数据来最小化内存成本。

这里有一个快速示例：

```js
gl.drawArraysInstanced(gl.TRIANGLES, 0, 3, 2);
```

# 非 2 的幂次纹理支持

正如我们在第七章中看到的，**纹理**和*mipmaps*是一种强大的特性，其中预计算的、优化的图像序列，每个图像都是同一图像的逐级降低分辨率的表示，允许进行更优化的渲染。虽然在 WebGL 1 中，mipmap 中每个图像或级别的宽度和高度都是前一级别的一半的 2 的幂，但在 WebGL 2 中，这个限制被移除了。也就是说，**非 2 的幂次纹理**与 2 的幂次纹理的工作方式相同。

# 片段深度

在 WebGL 2 中，我们可以手动设置自己的自定义值到深度缓冲区（z 缓冲区）。这个特性允许你在片段着色器中操作片段的深度。这可能会很昂贵，因为它迫使 GPU 绕过大量的正常片段丢弃行为，但也可以实现一些在没有极其高多边形几何体的情况下难以实现的效果。

# 着色器中的纹理大小

在 WebGL 2 中，你可以使用`textureSize`在 ESSL 着色器中查找任何纹理的大小。在 WebGL 1 中，你需要创建一个 uniform 并将数据手动传递到着色器中。

例如：

```js
vec2 size = textureSize(sampler, lod);
```

# 同步对象

在 WebGL 1 中，从 JavaScript 到 GPU 再到屏幕的路径对开发者来说相当不透明。也就是说，你发送绘制命令，在未来的某个未定义的时刻，结果会显示在屏幕上。在 WebGL 2 中，**同步对象**允许开发者获得更多关于 GPU 何时完成工作的洞察。使用`gl.fenceSync`，你可以在 GPU 命令流中的某个位置放置一个标记，然后稍后调用`gl.clientWaitSync`来暂停 JavaScript 执行，直到 GPU 完成所有命令直到栅栏。显然，对于想要快速渲染的应用程序来说，阻塞执行是不理想的，但这对获取准确的基准测试非常有帮助。这也可能在未来用于在工作者之间同步。

# 直接纹理查找

通常方便将大量数据存储在纹理中。这在 WebGL 1 中是可能的，但你只能使用纹理坐标在 `0.0` 到 `1.0` 的范围内访问纹理。在 WebGL 2 中，访问这类数据要容易得多，因为你可以轻松地使用像素/纹理坐标从纹理中查找值。

例如：

```js
vec4 values = texelFetch(sampler, ivec2Position, lod);
```

# 灵活的着色器循环

在 WebGL 1 中，着色器中的循环必须使用一个常量整数表达式。然而，由于 WebGL 2 基于 OpenGL ES 3.0，这个限制不再存在。

# 着色器矩阵函数

由于 WebGL 2 的着色语言比 WebGL 1 的功能更丰富，我们现在有更多矩阵数学运算可供使用。例如，如果需要矩阵的 `inverse` 或 `transpose`，我们需要将其作为统一变量传递。然而，在 WebGL 2 中，`inverse` 和 `transpose` 等函数是直接内置于着色器中的。

# 常见压缩纹理

在 WebGL 1 中，存在各种硬件依赖的压缩纹理格式。例如，`S3TC` 和 `PVTC` 格式分别仅适用于桌面和 iOS。然而，在 WebGL 2 中，以下格式由于硬件无关性而更加灵活：

+   `COMPRESSED_R11_EAC RED`

+   `COMPRESSED_SIGNED_R11_EAC RED`

+   `COMPRESSED_RG11_EAC RG`

+   `COMPRESSED_SIGNED_RG11_EAC RG`

+   `COMPRESSED_RGB8_ETC2 RGB`

+   `COMPRESSED_SRGB8_ETC2 RGB`

+   `COMPRESSED_RGB8_PUNCHTHROUGH_ALPHA1_ETC2 RGBA`

+   `COMPRESSED_SRGB8_PUNCHTHROUGH_ALPHA1_ETC2 RGBA`

+   `COMPRESSED_RGBA8_ETC2_EAC RGBA`

+   `COMPRESSED_SRGB8_ALPHA8_ETC2_EAC`

# 统一缓冲对象

设置着色程序统一变量是几乎任何 WebGL/OpenGL 绘制循环的重要组成部分。这可能会使你的绘制调用相当频繁，因为它们会进行数百或数千次的 `gl.uniform` 调用。

在 WebGL 1 中，如果我们有需要更新的 `n` 个统一变量，那么就需要使用 `n` 次适当的统一变量方法调用——这可能会相当慢。然而，在 WebGL 2 中，我们可以使用 **统一缓冲对象**，这允许我们从单个缓冲区中指定大量统一变量。这大大提高了性能，因为我们可以在 WebGL 之外使用 JavaScript 类型的数组来操作缓冲区中的统一变量，并通过单次调用更新一组统一变量。此外，统一缓冲区可以同时绑定到多个程序，因此可以一次性更新全局数据（如投影或视图矩阵），所有使用它们的程序将自动看到更改后的值。

**异构统一缓冲对象** 需要注意的是，在给定的应用程序中，你可以利用一系列不同的统一缓冲对象来满足你的应用需求。

# 整数纹理和属性

当在 WebGL 1 中，纹理和属性无论其原始类型如何，都表示为浮点值，而在 WebGL 2 中，纹理和属性提供整数表示。

# 变换反馈

WebGL 2 提供的一个强大技术是顶点着色器可以将它们的输出写回缓冲区。这在需要利用 GPU 的计算能力执行复杂计算并能在我们的应用程序中读取它们的情况下非常有用。

# 样本对象

虽然 WebGL 1 中所有纹理参数都是 *每个纹理*，但在 WebGL 2 中，我们可以选择使用 **样本对象**。通过使用样本，我们可以将所有纹理参数移动到样本中，允许单个纹理以不同的方式采样。

在 WebGL 1 中，纹理图像数据和采样信息（告诉 GPU 如何读取图像数据）都存储在纹理对象中。当我们想要从同一个纹理中两次以不同的方法（比如，线性过滤与最近邻过滤）读取时，可能会很痛苦，因为我们需要有两个纹理对象。通过使用样本对象，我们可以将这些两个概念分开。我们可以有一个纹理对象和两个不同的样本对象。这将导致我们的引擎组织纹理的方式发生变化。

这里有一个例子：

```js
const samplerA = gl.createSampler();
gl.samplerParameteri(samplerA, gl.TEXTURE_MIN_FILTER, gl.NEAREST_MIPMAP_NEAREST);
gl.samplerParameteri(samplerA, gl.TEXTURE_MAG_FILTER, gl.NEAREST);
gl.samplerParameteri(samplerA, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
gl.samplerParameteri(samplerA, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);

const samplerB = gl.createSampler();
gl.samplerParameteri(samplerB, gl.TEXTURE_MIN_FILTER, gl.LINEAR_MIPMAP_LINEAR);
gl.samplerParameteri(samplerB, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
gl.samplerParameteri(samplerB, gl.TEXTURE_WRAP_S, gl.MIRRORED_REPEAT);
gl.samplerParameteri(samplerB, gl.TEXTURE_WRAP_T, gl.MIRRORED_REPEAT);

// ...

gl.activeTexture(gl.TEXTURE0);
gl.bindTexture(gl.TEXTURE_2D, texture);
gl.bindSampler(0, samplerA);

gl.activeTexture(gl.TEXTURE1);
gl.bindTexture(gl.TEXTURE_2D, texture);
gl.bindSampler(1, samplerB);
```

# 深度纹理

WebGL 1 的一个主要缺点是缺乏对 **深度纹理** 的支持。在 WebGL 2 中，它们默认可用。

# 标准导数

虽然 WebGL 1 中你需要计算法线并将其传递给着色器，但在 WebGL 2 中，你可以通过使用默认可用的更大数学运算集在着色器内计算它们。

# UNSIGNED_INT 索引

在 WebGL 2 中，索引几何体没有实际的大小限制，因为我们可以使用 32 位 `int` 作为索引。

# 混合方程 MIN / MAX

在 WebGL 中，你可以通过这些附加函数轻松地获取两个颜色在混合时的 `MIN` 或 `MAX`。

# 多重渲染目标（MRT）

在 WebGL 2 中，你可以从着色器一次绘制到多个缓冲区。这对于各种延迟渲染技术来说非常强大。这对许多开发者来说是一个“大事件”，因为它使得许多已经成为现代实时 3D 核心部分的现代延迟渲染技术对 WebGL 变得实用。

# 顶点着色器中的纹理访问

虽然 WebGL 1 中在顶点着色器内访问纹理是可能的，但你可能需要计算你可以访问多少个纹理，这可能是零。在 WebGL 2 中，纹理访问更加流畅，并且纹理访问计数至少需要是 `16`。

# 多样本渲染缓冲区

虽然 WebGL 1 中我们只能使用 GPU 内置的多样本系统来抗锯齿我们的 `canvas`，但在 WebGL 2 中，支持我们自己的自定义多样本处理。

# 查询对象

**查询对象**为开发者提供了另一种更明确的方式来窥视 GPU 的内部工作原理。查询封装了一组 GL 命令，以便 GPU 异步报告某些统计信息。例如，遮挡查询是这样进行的：在一系列绘制调用周围执行`gl.ANY_SAMPLES_PASSED`查询将允许你检测是否有任何几何体通过了深度测试。如果没有，你知道该对象是不可见的，并且可以选择不在未来的帧中绘制该几何体，直到发生某些事件（对象移动、相机移动等）表明该几何体可能再次变得可见。

应该注意的是，这些查询是异步的，这意味着查询的结果可能在查询最初发出后的许多帧之后才准备好！这使得它们使用起来很棘手，但在适当的情况下可能值得。

这里有一个例子：

```js
gl.beginQuery(gl.ANY_SAMPLES_PASSED, query);
gl.drawArraysInstanced(gl.TRIANGLES, 0, 3, 2);
gl.endQuery(gl.ANY_SAMPLES_PASSED);

//...

(function tick() {
  if (!gl.getQueryParameter(query, gl.QUERY_RESULT_AVAILABLE)) {
    // A query's result is never available in the same frame
    // the query was issued.  Try in the next frame.
    requestAnimationFrame(tick);
    return;
  }

  var samplesPassed = gl.getQueryParameter(query, gl.QUERY_RESULT);
  gl.deleteQuery(query);
})();
```

# 纹理 LOD

**纹理 LOD**参数用于确定从哪个 mipmap 中获取。这允许进行 mipmap 流式传输，即只加载当前所需的 mipmap 级别。这对于 WebGL 环境非常有用，因为纹理是通过网络下载的。

```js
gl.texParameterf(gl.TEXTURE_2D, gl.TEXTURE_MIN_LOD, 0.0);
gl.texParameterf(gl.TEXTURE_2D, gl.TEXTURE_MAX_LOD, 10.0);
```

# 着色器纹理 LOD

**着色器纹理 LOD**偏差控制使得基于物理渲染的哑光环境效果中的 mipmap 级别控制更加简单。现在作为 WebGL 2 核心的一部分，`lodBias`可以作为可选参数传递给纹理。

# 常用的浮点纹理

在 WebGL 1 中，浮点纹理是可选的*，*但在 WebGL 2 中，它们默认可用。

# 迁移到 WebGL 2

正如我们之前所描述的，WebGL 2 几乎与 WebGL 1 完全向后兼容。

向后兼容性

所有向后兼容性的例外情况都记录在以下链接中：[`www.khronos.org/registry/WebGL/specs/latest/2.0/#BACKWARDS_INCOMPATIBILITY`](https://www.khronos.org/registry/webgl/specs/latest/2.0/#BACKWARDS_INCOMPATIBILITY)。

话虽如此，让我们来了解一下将 WebGL 1 应用程序迁移到 WebGL 2 的一些关键组件。

# 获取上下文

在 WebGL 1 中，你会用类似以下的方式获取 WebGL 上下文：

```js
const names = ['WebGL', 'experimental-WebGL', 'webkit-3d', 'moz-WebGL'];

for (let i = 0; i < names.length; ++i) {
  try {
    const context = canvas.getContext(names[i]);
    // work with context
  } catch (e) {
    console.log('Error attaining WebGL context', e);
  }
}
```

在 WebGL 2 中，你只需用一行代码获取上下文，如下所示：

```js
const context = canvas.getContext('WebGL 2');

```

# 扩展

在 WebGL 中，许多可选扩展对于更高级的功能是*必需的*，但在 WebGL 2 中，你可以移除其中大部分扩展，因为它们默认可用。以下是一些包括在内：

+   深度纹理：[`www.khronos.org/registry/WebGL/extensions/WebGL_depth_texture`](https://www.khronos.org/registry/webgl/extensions/WEBGL_depth_texture/)

+   浮点纹理：

    +   [`www.khronos.org/registry/WebGL/extensions/OES_texture_float`](https://www.khronos.org/registry/webgl/extensions/OES_texture_float)

    +   [`www.khronos.org/registry/WebGL/extensions/OES_texture_float_linear`](https://www.khronos.org/registry/webgl/extensions/OES_texture_float_linear)

+   顶点数组对象：[`www.khronos.org/registry/WebGL/extensions/OES_vertex_array_object`](https://www.khronos.org/registry/webgl/extensions/OES_vertex_array_object)

+   标准导数：[`www.khronos.org/registry/WebGL/extensions/OES_standard_derivatives`](https://www.khronos.org/registry/webgl/extensions/OES_standard_derivatives)

+   实例绘制：[`www.khronos.org/registry/WebGL/extensions/ANGLE_instanced_arrays`](https://www.khronos.org/registry/webgl/extensions/ANGLE_instanced_arrays)

+   `UNSIGNED_INT`索引：[`www.khronos.org/registry/WebGL/extensions/OES_element_index_uint`](https://www.khronos.org/registry/webgl/extensions/OES_element_index_uint)

+   设置`gl_FragDepth`：[`www.khronos.org/registry/WebGL/extensions/EXT_frag_depth`](https://www.khronos.org/registry/webgl/extensions/EXT_frag_depth)

+   混合方程`MIN`/`MAX`：[`www.khronos.org/registry/WebGL/extensions/EXT_blend_minmax`](https://www.khronos.org/registry/webgl/extensions/EXT_blend_minmax)

+   直接纹理 LOD 访问：[`www.khronos.org/registry/WebGL/extensions/EXT_shader_texture_lod`](https://www.khronos.org/registry/webgl/extensions/EXT_shader_texture_lod)

+   多个绘制缓冲区：[`www.khronos.org/registry/WebGL/extensions/WebGL_draw_buffers`](https://www.khronos.org/registry/webgl/extensions/WEBGL_draw_buffers)

+   顶点着色器中的纹理访问

# 着色器更新

虽然 WebGL 2 的着色语言，基于 GLSL 300，与 WebGL 1 的着色语言向后兼容，但我们需要进行一些更改以确保我们的着色器能够编译。现在让我们来探讨这些更改。

# 着色器定义

在 WebGL 2 的着色器中，我们必须在所有着色器前添加以下代码行：`#version 300 es`。需要注意的是，这必须是着色器的第一行，否则着色器将无法编译。

# 属性定义

由于属性作为输入提供给着色器，在 GLSL 300 ES 中，`attribute`修饰符被移除。例如，使用 WebGL 的 GLSL 100，你可能会有以下代码：

```js
attribute vec3 aVertexNormal;
attribute vec4 aVertexPosition;
```

在 GLSL 300 ES 中，这将如下所示：

```js
in vec3 aVertexNormal;
in vec4 aVertexPosition;
```

# 变量定义

在 GLSL 100 中，变量通常在顶点着色器和片段着色器中定义，但在 GLSL 300 ES 中，`varying`修饰符已被移除。也就是说，根据值是作为*输入*提供还是作为*输出*返回，变量修饰符会更新为相应的`in`和`out`修饰符。例如，考虑以下 GLSL 100 中的代码：

```js
// inside of the vertex shader
varying vec2 vTexcoord;
varying vec3 vNormal;

// inside of the fragment shader
varying vec2 vTexcoord;
varying vec3 vNormal;
```

这将变为以下 GLSL 300 ES 中的代码：

```js
// inside of the vertex shader
out vec2 vTexcoord;
out vec3 vNormal;

// inside of the fragment shader
in vec2 vTexcoord;
in vec3 vNormal;
```

# 不再使用`gl_FragColor`

在 GLSL 100 中，你最终会通过在片段着色器中设置`gl_FragColor`来渲染像素的颜色，但在 GLSL 300 ES 中，你只需从你的片段着色器中暴露一个值。例如，考虑以下 GLSL 100 中的代码：

```js
void main(void) {
  gl_FragColor = vec4(1.0, 0.2, 0.3, 1.0);
}
```

这将通过设置一个定义的输出变量来更新，如下所示：

```js
out vec4 fragColor;

void main(void) {
  fragColor = vec4(1.0, 0.2, 0.3, 1.0);
}
```

重要的是要注意，尽管我们声明了一个名为 `fragColor` 的变量，但由于歧义，你可以选择任何不以 `gl_` 前缀开始的名称。在这本书中，我们定义了这个自定义变量为 `fragColor`。

# 自动纹理类型检测

在 GLSL 100 中，你会通过使用适当的方法，如 `texture2D`，从纹理中获取颜色，但在 GLSL 300 ES 中，着色器会根据使用的采样器类型自动检测类型。例如，考虑以下 GLSL 100 中的内容：

```js
uniform sampler2D uSome2DTexture;
uniform samplerCube uSomeCubeTexture;

void main(void) {
  vec4 color1 = texture2D(uSome2DTexture, ...);
  vec4 color2 = textureCube(uSomeCubeTexture, ...);
}
```

这将在 GLSL 300 ES 中更新为以下内容：

```js
uniform sampler2D uSome2DTexture;
uniform samplerCube uSomeCubeTexture;

void main(void) {
  vec4 color1 = texture(uSome2DTexture, ...);
  vec4 color2 = texture(uSomeCubeTexture, ...);
}
```

# 非 2 的幂纹理支持

如第七章中所示，在 WebGL 1 中，对于不符合 *2 的幂* 限制的纹理，不存在米柏（mipmap）。然而，在 WebGL 2 中，非 2 的幂纹理与 2 的幂纹理工作方式完全相同。

# 浮点帧缓冲区附加

在 WebGL 1 中，需要一种奇怪的技巧来检查是否支持渲染到浮点纹理，而在 WebGL 2 中，这涉及到通过标准方法进行简单的检查。

# 顶点数组对象

虽然使用顶点数组对象不是 *必需的* 要求，但在迁移过程中使用它是一个高度 *推荐* 的特性。通过使用顶点数组对象，你可以改善你代码的整体结构以及你应用程序的性能。

# 摘要

让我们总结一下本章所学的内容：

+   我们涵盖了 WebGL 2 规范中仅有的许多核心方法。

+   我们了解了一些 WebGL 1 和 WebGL 2 之间的关键区别。

+   我们讨论了将 WebGL 1 应用程序转换为 WebGL 2 的迁移策略。

我们几乎完成了！你能相信吗？接下来，在最后一章，*前方之旅*，我们将通过列出概念、资源和其他有用的信息来结束这本书，这些信息既鼓舞人心又赋权，帮助你继续沿着掌握实时计算机图形学的道路前进。
