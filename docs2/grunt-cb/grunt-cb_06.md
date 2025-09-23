# 第六章 部署准备

在本章中，我们将介绍以下食谱：

+   压缩 HTML

+   压缩 CSS

+   优化图像

+   检查 JavaScript 代码

+   压缩 JavaScript 代码

+   设置 RequireJS

# 简介

一旦我们的网络应用程序构建完成并且稳定性得到保证，我们就可以开始为将其部署到目标市场做准备。这主要涉及优化构成应用程序的资产。在这个背景下，优化主要指的是某种形式的压缩，其中一些可能会带来性能提升。

专注于压缩主要是由于资产越小，它从托管位置传输到用户网络浏览器就越快。这导致用户体验大大改善，有时对于应用程序的功能也是必不可少的。

# 压缩 HTML

在这个食谱中，我们使用 `contrib-htmlmin (0.3.0)` 插件通过压缩来减小一些 **HTML** 文档的大小。

## 准备工作

在这个例子中，我们将使用我们在 第一章 中 *在项目中安装 Grunt* 食谱中创建的基本项目结构。如果您还不熟悉其内容，请务必参考。

## 如何操作...

以下步骤将引导我们创建一个示例 HTML 文档并配置一个压缩它的任务：

1.  我们将按照 第一章 中 *安装插件* 食谱中提供的说明，安装包含 `contrib-htmlmin` 插件的包。

1.  接下来，我们将在 `src` 目录中创建一个名为 `index.html` 的简单 HTML 文档，我们希望对其进行压缩，并在其中添加以下内容：

    ```js
    <html> 
      <head> 
        <title>Test Page</title> 
      </head> 
      <body> 
        <!-- This is a comment! --> 
        <h1>This is a test page.</h1> 
      </body> 
    </html>
    ```

1.  现在，我们将向我们的配置中添加以下 `htmlmin` 任务，该任务表示我们希望从 `src/index.html` 文件中移除空白和注释，并且希望将结果保存到 `dist/index.html` 文件中：

    ```js
    htmlmin: { 
      dist: { 
        src: 'src/index.html', 
        dest: 'dist/index.html', 
        options: { 
          removeComments: true, 
          collapseWhitespace: true
        }
      }
    }
    ```

    ### 小贴士

    这里使用 `removeComments` 和 `collapseWhitespace` 选项作为示例，因为使用默认的 `htmlmin` 任务将不会有任何效果。其他压缩选项可以在以下网址找到：

    [`github.com/kangax/html-minifier#options-quick-reference`](https://github.com/kangax/html-minifier#options-quick-reference)

1.  我们现在可以使用 `grunt htmlmin` 命令运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "htmlmin:dist" (htmlmin) task 
    Minified dist/index.html 147 B → 92 B

    ```

1.  如果我们现在查看 `dist/index.html` 文件，我们会看到所有空白和注释已经被移除：

    ```js
    <html>
      <head>
        <title>Test Page</title>
      </head>
      <body>
        <h1>This is a test page.</h1>
      </body>
    </html>
    ```

# 压缩 CSS

在这个食谱中，我们将使用 `contrib-cssmin (0.10.0)` 插件通过压缩来减小一些 **CSS** 文档的大小。

## 准备工作

在这个示例中，我们将使用我们在第一章中“在项目中安装 Grunt”配方中创建的基本项目结构。如果您还不熟悉其内容，请务必参考它。

## 如何做到这一点...

以下步骤将指导我们创建一个示例 CSS 文档并配置一个将其压缩的任务。

1.  我们将按照第一章中“安装插件”配方提供的说明，安装包含`contrib-cssmin`插件的包，该配方位于[《使用 Grunt 入门》](https://wiki.example.org/getting_started_with_grunt)。

1.  然后，我们将在`src`目录中创建一个名为`style.css`的简单 CSS 文档，我们希望将其压缩，并提供以下内容：

    ```js
    body { 
      /* Average body style */
      background-color: #ffffff; 
      color: #000000; /*! Black (Special) */
    }
    ```

1.  现在，我们将添加以下`cssmin`任务到我们的配置中，该任务表示我们希望将`src/style.css`文件压缩，并将结果保存到`dist/style.min.css`文件中：

    ```js
    cssmin: { 
      dist: { 
        src: 'src/style.css', 
        dest: 'dist/style.min.css' 
      } 
    }
    ```

1.  我们现在可以使用`grunt cssmin`命令运行此任务，它应该产生以下输出：

    ```js
    Running "cssmin:dist" (cssmin) task 
    File dist/style.css created: 55 B → 38 B

    ```

1.  如果我们查看生成的`dist/style.min.css`文件，我们将看到它包含原始`src/style.css`文件的压缩内容：

    ```js
    body{background-color:#fff;color:#000;/*! Black (Special) */}
    ```

## 还有更多...

`cssmin`任务为我们提供了几个有用的选项，可以与它的基本压缩功能一起使用。我们将探讨添加标题、移除特殊注释和报告 gzip 结果。

### 添加标题

如果我们希望在生成的 CSS 文件中自动包含有关压缩结果的一些信息，我们可以在标题中这样做。可以通过向`banner`选项提供所需的标题内容来将标题添加到结果之前，如下面的示例所示：

```js
cssmin: { 
  dist: { 
    src: 'src/style.css', 
    dest: 'dist/style.min.css', 
    options: { 
      banner: '/* Minified version of style.css */' 
    } 
  } 
}
```

### 移除特殊注释

应该在压缩过程中保留的注释称为特殊注释，可以使用"`/*! comment */`"标记来表示。默认情况下，`cssmin`任务将保留所有特殊注释，但我们可以通过使用`keepSpecialComments`选项来改变这种行为。

`keepSpecialComments`选项可以设置为`*`、`1`或`0`的值。`*`值是默认值，表示应保留所有特殊注释，`1`表示仅保留找到的第一个注释，而`0`表示不应保留任何注释。以下配置将确保从我们的压缩结果中移除所有注释：

```js
cssmin: { 
  dist: { 
    src: 'src/style.css', 
    dest: 'dist/style.min.css', 
    options: { 
      keepSpecialComments: 0 
    } 
  } 
}
```

### 报告 gzip 结果

报告功能有助于查看`cssmin`任务如何压缩我们的 CSS 文件。默认情况下，将显示目标文件的大小和压缩结果的大小，但如果我们还想看到结果的 gzip 大小，可以将`report`选项设置为`gzip`，如下面的示例所示：

```js
cssmin: {
  dist: {
    src: 'src/main.css',
    dest: 'dist/main.css',
    options: {
      report: 'gzip'
    }
  }
}
```

# 优化图像

在这个菜谱中，我们将使用`contrib-imagemin (0.9.4)`插件通过尽可能压缩图片来减小图片大小，同时不牺牲其质量。此插件还提供了一个自己的插件框架，将在本菜谱的末尾进行讨论。

## 准备工作

在本例中，我们将使用我们在第一章中创建的基本项目结构，即“在项目中安装 Grunt”菜谱中的内容，即*使用 Grunt 入门*。如果您还不熟悉其内容，请务必参考。

## 如何操作...

以下步骤将引导我们配置一个将压缩项目中的图片的任务。

1.  我们将首先按照第一章中“安装插件”菜谱中的说明安装包含`contrib-imagemin`插件的包。

1.  接下来，我们可以确保在`src`目录中有一个名为`image.jpg`的图片，我们希望对其进行优化。

    ### 小贴士

    在本例的其余部分，我们将使用与本菜谱一起提供的示例代码中的示例图片。

1.  现在，我们将添加以下`imagemin`任务到我们的配置中，并指出我们希望对`src/image.jpg`文件进行优化，并将结果保存到`dist/image.jpg`文件中：

    ```js
    imagemin: {
      dist: {
        src: 'src/image.jpg',
        dest: 'dist/image.jpg'
      }
    }
    ```

1.  然后，我们可以使用`grunt imagemin`命令来运行任务，这应该会产生以下输出：

    ```js
    Running "imagemin:dist" (imagemin) task
    Minified 1 image (saved 13.36 kB)

    ```

1.  如果我们现在查看`dist/image.jpg`文件，我们将看到其大小已经减小，而质量没有受到影响。

## 更多内容...

`imagemin`任务为我们提供了几个选项，允许我们调整其优化功能。我们将探讨如何调整**PNG**压缩级别，禁用渐进式**JPEG**生成，禁用交错**GIF**生成，指定要使用的**SVGO**插件，以及使用`imagemin`插件框架。

### 调整 PNG 压缩级别

通过多次运行压缩算法，可以增加 PNG 图片的压缩。默认情况下，压缩算法运行 16 次。可以通过向`optimizationLevel`选项提供一个从`0`到`7`的数字来更改此数字。`0`值表示压缩实际上被禁用，而`7`表示算法应该运行 240 次。在以下配置中，我们将压缩级别设置为最大值：

```js
imagemin: {
  dist: {
    src: 'src/image.png',
    dest: 'dist/image.png',
    options: {
      optimizationLevel: 7
    }
  }
}
```

### 禁用渐进式 JPEG 生成

渐进式 JPEG 是通过多次压缩来压缩的，这使得它们的一个低质量版本可以快速变得可见，并在接收其余图像时提高质量。这在通过较慢的连接显示图像时特别有帮助。

默认情况下，`imagemin` 插件会以渐进式格式生成 JPEG 图像，但可以通过将 `progressive` 选项设置为 `false` 来禁用此行为，如下面的示例所示：

```js
imagemin: {
  dist: {
    src: 'src/image.jpg',
    dest: 'dist/image.jpg',
    options: {
      progressive: false
    }
  }
}
```

### 禁用交错 GIF 生成

交错 GIF 与渐进式 JPEG 相当，因为它允许在完全下载之前以较低分辨率显示包含的图像，并在接收其余图像时提高质量。

默认情况下，`imagemin` 插件会以交错格式生成 GIF 图像，但可以通过将 `interlaced` 选项设置为 `false` 来禁用此行为，如下面的示例所示：

```js
imagemin: {
  dist: {
    src: 'src/image.gif',
    dest: 'dist/image.gif',
    options: {
      interlaced: false
    }
  }
}
```

### 指定要使用的 SVGO 插件

在优化 SVG 图像时，默认使用 SVGO 库。这允许我们指定使用 SVGO 库提供的各种插件，每个插件在目标文件上执行特定功能。

### 小贴士

请参考以下网址以获取有关如何使用 svgo 插件选项和 SVGO 库的更详细说明：

[`github.com/sindresorhus/grunt-svgmin#available-optionsplugins`](https://github.com/sindresorhus/grunt-svgmin#available-optionsplugins)

库中的大多数插件默认启用，但如果我们想特别指出哪些应该使用，我们可以通过使用 `svgoPlugins` 选项来实现。在这里，我们可以提供一个对象数组，其中每个对象包含一个属性，表示要影响的插件名称，后跟一个 `true` 或 `false` 值，以指示是否激活。以下配置禁用了三个默认插件：

```js
 imagemin: {
  dist: {
    src: 'src/image.svg',
    dest: 'dist/image.svg',
    options: {
      svgoPlugins: [
        {removeViewBox:false},
 {removeUselessStrokeAndFill:false},
 {removeEmptyAttrs:false}
      ]
    }
  }
}
```

### 使用 `imagemin` 插件框架

为了支持各种图像优化项目，`imagemin` 插件拥有自己的插件框架，允许开发者轻松创建一个扩展，以便使用他们所需的工具。

### 小贴士

您可以在以下网址获取 `imagemin` 插件框架的可用插件模块列表：

[`www.npmjs.com/browse/keyword/imageminplugin`](https://www.npmjs.com/browse/keyword/imageminplugin)

以下步骤将指导我们安装并使用 `mozjpeg` 插件来压缩项目中的图像。这些步骤从主配方开始。

1.  我们将首先使用 `npm install imagemin-mozjpeg` 命令安装 `imagemin-mozjpeg` 包，这将产生以下输出：

    ```js
    imagemin-mozjpeg@4.0.0 node_modules/imagemin-mozjpeg

    ```

1.  包安装完成后，我们需要将其导入到配置文件中，以便在任务配置中使用它。我们通过在 `Gruntfile.js` 文件的顶部添加以下行来实现：

    ```js
    var mozjpeg = require('imagemin-mozjpeg');
    ```

1.  插件安装并导入后，我们现在可以更改 `imagemin` 任务的配置，通过添加 `use` 选项并提供初始化的插件：

    ```js
    imagemin: {
      dist: {
        src: 'src/image.jpg',
        dest: 'dist/image.jpg',
        options: {
          use: [mozjpeg()]
        }
      }
    }
    ```

1.  最后，我们可以通过运行 `grunt imagemin` 命令来测试我们的设置。这应该会产生类似于以下输出的结果：

    ```js
    Running "imagemin:dist" (imagemin) task
    Minified 1 image (saved 9.88 kB)

    ```

# 检查 JavaScript 代码

在这个菜谱中，我们将使用`contrib-jshint (0.11.1)`插件来检测 JavaScript 代码中的错误和潜在问题。它也常用于在团队或项目中强制执行代码约定。从其名称可以推断出，它基本上是 JSHint 工具的 Grunt 适配器。

## 准备工作

在这个例子中，我们将使用我们在第一章中*在项目中安装 Grunt*菜谱中创建的基本项目结构。如果你还不熟悉其内容，请务必参考它。

## 如何做...

以下步骤将引导我们创建一个样本 JavaScript 文件，并配置一个任务，使用 JSHint 工具扫描和分析它。

1.  我们将按照第一章中*安装插件*菜谱提供的说明安装包含`contrib-jshint`插件的包，该菜谱在*使用 Grunt 入门*中。

1.  接下来，我们将在`src`目录中创建一个名为`main.js`的样本 JavaScript 文件，并在其中添加以下内容：

    ```js
    sample = 'abc';
    console.log(sample);
    ```

1.  我们的样本文件准备好了，现在我们可以将以下`jshint`任务添加到我们的配置中。我们将配置这个任务以针对样本文件，并添加一个基本选项，这是我们在这个例子中需要的：

    ```js
    jshint: {
      main: {
        options: {
          undef: true
        },
        src: ['src/main.js']
      }
    }
    ```

    ### 小贴士

    `undef`选项是专门用于此示例的标准 JSHint 选项，并且对于此插件的功能不是必需的。指定此选项表示我们希望对未明确定义就使用的变量引发错误。

1.  我们现在可以使用`grunt jshint`命令来运行任务，这将生成输出，告诉我们样本文件中存在的问题：

    ```js
    Running "jshint:main" (jshint) task

     src/main.js
     1 |sample = 'abc';
     ^ 'sample' is not defined.
     2 |console.log(sample);
     ^ 'console' is not defined.
     2 |console.log(sample);
     ^ 'sample' is not defined.

    >> 3 errors in 1 file

    ```

## 还有更多...

`jshint`任务为我们提供了几个选项，允许我们更改其一般行为，以及它如何分析目标代码。我们将探讨如何指定标准 JSHint 选项，指定全局定义的变量，将报告的输出发送到文件，以及在 JSHint 错误发生时防止任务失败。

### 指定标准 JSHint 选项

`contrib-jshint`插件提供了一种简单的方法，将任务选项对象中的所有标准 JSHint 选项传递给底层的 JSHint 工具。

### 小贴士

JSHint 工具提供的所有选项的列表可以在以下 URL 找到：

[`jshint.com/docs/options/`](http://jshint.com/docs/options/)

以下示例将`curly`选项添加到我们在主菜谱中创建的任务中，以强制在适当的地方使用花括号：

```js
jshint: {
  main: {
    options: {
      undef: true,
      curly: true
    },
    src: ['src/main.js']
  }
}
```

### 指定全局定义的变量

利用全局定义的变量在处理 JavaScript 时相当常见，这就是`globals`选项派上用场的地方。使用此选项，我们可以定义一组将在目标代码中使用的全局值，这样当 JSHint 遇到它们时就不会引发错误。

在以下示例中，我们指出应将 `console` 变量视为全局变量，并在遇到时不会引发错误：

```js
jshint: {
  main: {
    options: {
      undef: true,
      globals: {
 console: true
 }
    },
    src: ['src/main.js']
  }
}
```

### 将报告输出发送到文件

如果我们希望将 JSHint 分析的结果输出存储起来，我们可以通过指定一个文件路径来使用 `reporterOutput` 选项，如下面的示例所示：

```js
jshint: {
  main: {
    options: {
      undef: true,
      reporterOutput: 'report.dat'
    },
    src: ['src/main.js']
  }
}
```

### 防止任务因 JSHint 错误而失败

对于 `jshint` 任务，默认行为是在任何目标文件中遇到 JSHint 错误时退出正在运行的 Grunt 进程。如果你希望即使在出现错误的情况下也能继续监视文件变化，这种行为尤其不受欢迎。

在以下示例中，我们指出当遇到错误时，我们希望保持进程运行，通过将 `force` 选项设置为 `true` 值来实现：

```js
jshint: {
  main: {
    options: {
      undef: true,
      force: true
    },
    src: ['src/main.js']
  }
}
```

# 混淆 JavaScript 代码

在本食谱中，我们将使用 `contrib-uglify (0.8.0)` 插件来压缩和混淆包含 JavaScript 代码的一些文件。

在很大程度上，压缩过程只是删除源代码文件中的所有不必要的字符并缩短变量名。这有可能显著减小文件大小，略微提高性能，并使你公开可用的代码的内部工作原理更加神秘。

## 准备工作

在本例中，我们将使用在 第一章 中创建的基本项目结构，即 第一章 的 *在项目中安装 Grunt* 食谱，*使用 Grunt 入门*。如果你还不熟悉其内容，请务必参考。

## 如何操作...

以下步骤将引导我们创建一个示例 JavaScript 文件并配置一个将对其进行混淆的任务。

1.  我们将首先按照 第一章 中提供的 *安装插件* 食谱中的说明安装包含 `contrib-uglify` 插件的包，*使用 Grunt 入门*。

1.  然后，我们可以在 `src` 目录中创建一个名为 `main.js` 的示例 JavaScript 文件，我们希望对其进行压缩，并为其提供以下内容：

    ```js
    var main = function () {
      var one = 'Hello' + ' ';
      var two = 'World';

      var result = one + two;

      console.log(result);
    };
    ```

1.  在我们的示例文件准备就绪后，我们现在可以向配置中添加以下 `uglify` 任务，指定示例文件为目标并提供一个输出文件目的地：

    ```js
    uglify: {
      main: {
        src: 'src/main.js',
        dest: 'dist/main.js'
      }
    }
    ```

1.  现在，我们可以使用 `grunt uglify` 命令运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "uglify:main" (uglify) task
    >> 1 file created.

    ```

1.  如果我们现在查看生成的 `dist/main.js` 文件，我们应该看到它包含原始 `src/main.js` 文件的压缩内容。

## 还有更多...

`uglify` 任务为我们提供了几个选项，允许我们更改其一般行为并查看它如何混淆目标代码。我们将探讨指定标准 UglifyJS 选项、生成源映射以及将生成的代码包装在封装器中。

### 指定标准 UglifyJS 选项

基础的 UglifyJS 工具可以为它的每个独立功能部分提供一组选项。这些部分是扭曲器、压缩器和美化器。`contrib-plugin`插件允许使用`mangle`、`compress`和`beautify`选项将这些选项传递给这些部分。

### 小贴士

对于扭曲器、压缩器和美化器各部分的可用选项，可以在以下每个 URL 中找到（按提到的顺序列出）：

[`github.com/mishoo/UglifyJS2#mangler-options`](https://github.com/mishoo/UglifyJS2#mangler-options)

[`github.com/mishoo/UglifyJS2#compressor-options`](https://github.com/mishoo/UglifyJS2#compressor-options)

[`github.com/mishoo/UglifyJS2#beautifier-options`](https://github.com/mishoo/UglifyJS2#beautifier-options)

以下示例修改了主要菜谱的配置，为这些部分中的每一个提供一个单独的选项：

```js
uglify: {
  main: {
    src: 'src/main.js',
    dest: 'dist/main.js',
    options: {
 mangle: {
 toplevel: true
 },
 compress: {
 evaluate: false
 },
 beautify: {
 semicolons: false
 }
    }
  }
}
```

### 生成源映射

随着代码被扭曲和压缩，它对人类来说变得几乎无法阅读，因此，调试几乎变得不可能。正因为如此，我们才有了在压缩代码时生成源映射的选项。

以下示例使用`sourceMap`选项来表示我们希望生成与我们的扭曲代码一起的源映射：

```js
uglify: {
  main: {
    src: 'src/main.js',
    dest: 'dist/main.js',
    options: {
      sourceMap: true
    }
  }
}
```

运行修改后的任务现在除了包含我们扭曲源代码的`dist/main.js`文件外，还会在扭曲文件所在的同一目录中生成一个名为`main.js.map`的源映射文件。

### 将生成的代码包装在封装器中

当构建自己的 JavaScript 代码模块时，通常有一个好主意是将它们包装在一个包装函数中，以确保你不会将不会在模块本身之外使用的变量污染全局作用域。

为了这个目的，我们可以使用`wrap`选项来表示我们希望将生成的扭曲代码包装在一个包装函数中，如下面的示例所示：

```js
uglify: {
  main: {
    src: 'src/main.js',
    dest: 'dist/main.js',
    options: {
 wrap: true
    }
  }
}
```

如果我们现在查看结果`dist/main.js`文件，我们应该看到原始文件的所有扭曲内容现在都包含在一个包装函数中。

# 设置 RequireJS

在这个菜谱中，我们将使用`contrib-requirejs (0.4.4)`插件将我们的 Web 应用程序的模块化源代码打包到一个文件中。

在很大程度上，这个插件只是为**RequireJS**工具提供了一个包装器。RequireJS 提供了一个框架来模块化 JavaScript 源代码，并有序地消费这些模块。它还允许将整个应用程序打包到一个文件中，并只导入所需的模块，同时保持模块结构完整。

### 小贴士

要查看本配方中创建的捆绑应用程序的实际效果，请参阅为此配方提供的示例代码。它包括一个基本开发服务器设置，按照第一章中“设置基本 Web 服务器”配方进行设置，以及所需的库和一个消耗生成的应用程序捆绑文件的示例`index.html`文件。

## 准备工作

在本例中，我们将使用我们在第一章中“在项目中安装 Grunt”配方中创建的基本项目结构。如果您还不熟悉其内容，请务必参考它。

## 如何做到这一点...

以下步骤将指导我们创建一些用于示例应用程序的文件，并设置一个将它们捆绑成一个文件的任务。

1.  我们将首先按照第一章中“安装插件”配方提供的说明安装包含`contrib-requirejs`插件的包。

1.  首先，我们需要一个包含我们的 RequireJS 配置的文件。让我们在`src`目录下创建一个名为`config.js`的文件，并在其中添加以下内容：

    ```js
    require.config({
      baseUrl: 'app'
    });
    ```

1.  其次，我们将创建一个我们希望在应用程序中使用的示例模块。让我们在`src/app`目录下创建一个名为`sample.j`s 的文件，并在其中添加以下内容：

    ```js
    define(function (require) {
      return function () {
        console.log('Sample Module');
      }
    });
    ```

1.  最后，我们需要一个包含我们应用程序主入口点的文件，并使用我们的示例模块。让我们在`src/app`目录下创建一个名为`main.js`的文件，并在其中添加以下内容：

    ```js
    require(['sample'], function (sample) {
      sample();
    });
    ```

1.  现在我们已经拥有了构建示例应用程序所需的所有必要文件，我们可以设置一个`requirejs`任务，将它们捆绑成一个文件：

    ```js
    requirejs: {
      app: {
        options: {
          mainConfigFile: 'src/config.js',
          name: 'main',
          out: 'www/js/app.js'
        }
      }
    }
    ```

    ### 小贴士

    `mainConfigFile`选项指出将确定 RequireJS 行为的配置文件。

    `name`选项表示包含应用程序入口点的模块的名称。在本例中，我们的应用程序入口点包含在`app/main.js`文件中，而`app`是我们在`src/config.js`文件中的应用程序的基本目录。这把`app/main.js`文件名转换成了`main`模块名称。

    `out`选项用于指示应接收捆绑应用程序结果的文件。

1.  我们现在可以使用`grunt requirejs`命令运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "requirejs:app" (requirejs) task

    ```

1.  现在，我们应该在`www/js`目录下有一个名为`app.js`的文件，其中包含我们的整个示例应用程序。

## 更多内容...

`requirejs`任务为我们提供了由 RequireJS 工具提供的所有底层选项。我们将探讨如何使用这些公开的选项并生成源映射。

### 使用 RequireJS 优化器选项

RequireJS 优化器是一个非常复杂的工具，因此提供了大量的选项来调整其行为。`contrib-requirejs` 插件允许我们通过仅指定插件本身的选项来轻松设置这些选项。

### 小贴士

所有可用的 RequireJS 构建系统配置选项可以在以下 URL 的示例配置文件中找到：

[`github.com/jrburke/r.js/blob/master/build/example.build.js`](https://github.com/jrburke/r.js/blob/master/build/example.build.js)

以下示例表明，应该使用 `optimize` 选项来使用 **UglifyJS2** 优化器而不是默认的 **UglifyJS** 优化器：

```js
requirejs: {
  app: {
    options: {
      mainConfigFile: 'src/config.js',
      name: 'main',
      out: 'www/js/app.js',
      optimize: 'uglify2'
    }
  }
}
```

### 生成源映射

当源代码捆绑成一个文件时，调试会变得有些困难，因为你现在必须浏览大量的代码才能到达你真正感兴趣的地方。

通过将生成的捆绑文件与其来源的模块化结构相关联，源映射可以帮助我们解决这个问题。简单来说，有了源映射，即使我们实际上使用的是捆绑文件，我们的调试器也会显示我们之前分开的文件。

以下示例使用了 `generateSourceMap` 选项来表明我们希望生成与结果文件一起的源映射：

```js
requirejs: {
  app: {
    options: {
      mainConfigFile: 'src/config.js',
      name: 'main',
      out: 'www/js/app.js',
      optimize: 'uglify2',
      preserveLicenseComments: false,
      generateSourceMaps: true
    }
  }
}
```

### 小贴士

为了使用 `generateSourceMap` 选项，我们必须通过将 `optimize` 选项设置为 `uglify2` 来表明应该使用 UglifyJS2 进行优化，并且通过将 `preserveLicenseComments` 选项设置为 `false` 来表明不应保留许可证注释。
