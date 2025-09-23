# 第三章。模板引擎

在本章中，我们将介绍以下菜谱：

+   渲染 Jade 模板

+   在 Jade 模板中使用数据

+   在 Jade 模板中使用自定义过滤器

+   编译 Jade 模板

+   编译 Handlebars 模板

+   编译 Underscore 模板

+   在 Handlebars 模板中使用部分模板

+   在 AMD 模块中包装 Jade 模板

+   在 AMD 模块中包装 Handlebars 模板

+   在 AMD 模块中包装 Underscore 模板

+   在 CommonJS 模块中包装 Handlebars 模板

+   在编译前修改 Jade 模板

+   在编译前修改 Handlebars 模板

+   在编译前修改 Underscore 模板

# 简介

通过组合逻辑和数据生成内容提出了一系列独特的问题，模板引擎专门设计来解决这些问题。在本章中，我们将主要关注使用各种模板引擎生成 HTML，尽管其中一些设计成允许生成几乎任何类型的可读文件格式。

尽管许多功能丰富的 JavaScript 社区中的 Web 应用程序框架都提供了模板的按需编译、渲染和缓存，但使用 Grunt 做到这一点提供了构建高度优化和专用解决方案所需的灵活性。模板可以直接渲染到 HTML 或编译为 JST，可以直接使用而无需任何额外处理。这些模板的产物也可以打包，以便在几乎任何环境中使用。

在撰写本文时，有许多优秀的模板引擎可供选择，其中大多数都通过插件支持 Grunt，但本章我们将重点关注 Grunt 社区最广泛使用的那些。

# 渲染 Jade 模板

Jade 模板引擎允许我们使用其最小化且熟悉的语法轻松构建和维护 HTML 模板。在本菜谱中，我们将使用 `contrib-jade (0.12.0)` 插件来编译一个渲染简单 HTML 页面的模板。

## 准备工作

在本例中，我们将使用我们在 第一章 中创建的基本项目结构，即 *在项目中安装 Grunt* 菜谱，*开始使用 Grunt*。如果您还不熟悉其内容，请务必参考。

## 如何做到这一点...

以下步骤将引导我们创建一个简单的 Jade 模板并将其渲染为 HTML 文件。

1.  我们将首先按照 第一章 中提供的 *安装插件* 菜谱说明安装包含 `contrib-jade` 插件的包，*开始使用 Grunt*。

1.  让我们在项目目录中创建一个简单的 Jade 模板，命名为 `index.jade`，并添加以下内容：

    ```js
    doctype html
    html
      head
        title Index
      body
        h1 Index
    ```

1.  现在，我们可以在配置中添加以下 `jade` 任务，它将编译我们刚刚创建的 `index.jade` 文件到项目目录中的 `index.html` 文件：

    ```js
    jade: {
      sample: {
        options: {
          pretty: true
        },
        src: 'index.jade',
        dest: 'index.html'
      }
    }
    ```

    ### 小贴士

    在这个例子中，我们将 `pretty` 选项设置为 `true`，以便从模板生成的 HTML 更易于阅读。当项目仍在开发阶段时，这通常是首选的，但在生产环境中通常不使用此选项，因为启用此选项会增加生成文件的大小。

1.  最后，我们可以使用 `grunt jade` 命令运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "jade:sample" (jade) task
    File index.html created.

    ```

1.  如果我们现在查看我们的项目目录，我们应该看到包含以下内容的新的 `index.html` 文件：

    ```js
    <!DOCTYPE html>
    <html>
      <head>
        <title>Index</title>
      </head>
      <body>
        <h1>Index</h1>
      </body>
    </html>
    ```

# 在 Jade 模板中使用数据

一旦我们有了 Jade 模板，我们就可以使用它以各种数据渲染相同的页面结构。在这个食谱中，我们将使用 `contrib-jade (0.12.0)` 插件与 `data` 选项结合使用，以发送应在模板渲染中使用的数据。

## 准备工作

在这个例子中，我们将使用我们在本章中 *渲染 Jade 模板* 食谱中创建的基本项目结构。如果您还不熟悉其内容，请务必参考它。

## 如何操作...

以下步骤将指导我们在渲染模板时提供数据，并修改我们的 `index.jade` 模板以使用提供的数据。

1.  首先，我们将修改我们的 `index.jade` 模板，以便使用其上下文提供的变量：

    ```js
    doctype html
    html
      head
     title= title
      body
     h1= title
     p= body

    ```

1.  现在，我们可以为 `title` 和 `body` 变量提供值，这些值将由模板使用。这是通过在配置中添加的 `data` 选项中指定它们来完成的：

    ```js
    jade: {
      sample: {
        options: {
          pretty: true,
     data: {
     title: 'Most Amazing Site',
     body: 'Disappointing mundane content.'
     }
        },
        src: 'index.jade',
        dest: 'index.html'
      }
    }
    ```

1.  最后，我们可以使用 `grunt jade` 命令运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "jade:sample" (jade) task
    File index.html created.

    ```

1.  如果我们现在查看运行任务生成的 `index.html` 文件，我们会发现模板中指示的变量已被 `data` 选项中指定的值所替换：

    ```js
    <!DOCTYPE html>
    <html>
     <head>
     <title>Most Amazing Site</title>
     </head>
     <body>
     <h1>Most Amazing Site</h1>
     <p>Disappointing mundane content.</p>
     </body>
    </html>

    ```

## 还有更多...

通常来说，将模板中使用的数据硬编码在 Grunt 配置中并不是一个好主意，而是应该从外部数据源导入。

以下步骤将指导我们创建外部数据源并修改我们的配置，以便在渲染模板时使用的数据从中加载。

1.  首先，我们在项目目录中创建一个名为 `data.json` 的文件，其中包含我们希望在模板渲染中使用的所有数据：

    ```js
    {
      "title": "Most Amazing Site",
      "body": "Disappointing mundane content."
    }
    ```

1.  然后，我们可以修改我们的 `jade` 任务配置，通过使用 `grunt.file.readJSON` 方法从外部源导入数据：

    ```js
    jade: {
      sample: {
        options: {
          pretty: true,
          data: grunt.file.readJSON('data.json')
        },
        src: 'index.jade',
        dest: 'index.html'
      }
    }
    ```

1.  如果我们现在使用 `grunt jade` 命令运行任务，我们应该得到与主食谱中完全相同的结果。

# 在 Jade 模板中使用自定义过滤器

Jade 模板引擎提供的过滤器使我们能够指定在处理模板中的特定块时应使用的方法。这通常用于以 Jade 语言本身之外的其他格式编写的块内容。例如，Jade 库提供的`coffee`和`markdown`过滤器可以将**CoffeeScript**代码渲染成 JavaScript，将**Markdown**内容渲染成 HTML。

在这个菜谱中，我们将使用`contrib-jade (0.12.0)`插件及其`filters`选项，使我们能够在模板中访问一个名为`link`的自定义过滤器。此过滤器将使用一个简单的算法来查找 URL 并将它们包裹在锚标签中，从而将它们转换为链接。

## 准备工作

在这个例子中，我们将使用我们在本章*渲染 Jade 模板*菜谱中创建的基本项目结构。如果您还不熟悉其内容，请务必参考它。

## 如何操作...

以下步骤将指导我们添加`link`自定义过滤器并修改我们的`index.jade`模板以使用它。

1.  首先，我们将通过使用之前配置的`jade`任务中的`filters`选项来添加`link`自定义过滤器到我们的配置中：

    ```js
    jade: {
      sample: {
        options: {
          pretty: true,
          filters: {
     link: function (block) {
     var url_re = /(http:\/\/[^\s]*)/;
     var link_replace = '<a href="$1">$1</a>';
     return block.replace(url_re, link_replace);
     }
     }
        },
        src: 'index.jade',
        dest: 'index.html'
      }
    }
    ```

1.  现在，我们可以修改我们的`index.jade`模板的内容，以使用我们的`link`自定义过滤器：

    ```js
    doctype html
    html
      head
        title Index
      body
        h1 Index
        p
     :link
     Be sure to check out
     http://gruntjs.com/
     for more information.

    ```

1.  最后，我们可以使用`grunt jade`命令运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "jade:sample" (jade) task
    File index.html created.

    ```

1.  如果我们现在查看由运行任务生成的`index.html`文件，我们会发现它已经用锚标签包围了我们提供给`link`过滤器的块内的链接：

    ```js
    <!DOCTYPE html>
    <html>
      <head>
        <title>Index</title>
      </head>
      <body>
        <h1>Index</h1>
        <p>
          Be sure to check out
          <a href="http://gruntjs.com/">http://gruntjs.com/</a>
          for more information.
        </p>
      </body>
    </html>
    ```

# 编译 Jade 模板

当构建一个需要在尽可能短的时间内在前端渲染 HTML 模板的 Web 应用程序时，编译模板变得至关重要。Jade 模板引擎允许我们将模板编译成用于前端的**JavaScript 模板**（**JSTs**）。

在这个菜谱中，我们将使用`contrib-jade (0.12.0)`插件来编译一个渲染简约博客的模板。

## 准备工作

在这个例子中，我们将使用我们在第一章中*在项目中安装 Grunt*菜谱中创建的基本项目结构。如果您还不熟悉其内容，请务必参考它。

## 如何操作...

以下步骤将指导我们创建一个简单的 Jade 模板并将其编译成包含在其他文件中的 JST。

1.  我们将按照第一章中*安装插件*菜谱中提供的说明来安装包含`contrib-jade`插件的包，该菜谱位于*开始使用 Grunt*。

1.  让我们在项目目录中创建一个名为`blog.jade`的简单 Jade 模板文件，并给它以下内容：

    ```js
    .blog
      h1= title
      each post in posts
        .post
          h2= post.title
          p= post.body
    ```

1.  现在，我们可以将以下 `jade` 任务添加到我们的配置中，它将 `blog.jade` 模板编译成包含在我们项目目录的 `templates.js` 文件中的 JST：

    ```js
    jade: {
      blog: {
        options: {
          client: true
        },
        src: 'blog.jade',
        dest: 'templates.js'
      }
    }
    ```

1.  最后，我们可以使用 `grunt jade` 命令来运行任务，它应该会产生类似于以下内容的输出：

    ```js
    Running "jade:blog" (jade) task
    File templates.js created.

    ```

1.  我们现在应该在项目目录中有一个名为 `templates.js` 的新文件，其中包含我们编译模板的 JST 代码。

## 它是如何工作的...

以下步骤展示了编译模板的使用示例，并展示了一个渲染结果的示例。

1.  首先，您需要将 Jade 运行时库包含到您希望使用编译模板的应用程序或页面中。

    ### 小贴士

    在撰写本文时，Jade 运行时库位于官方 Jade 仓库的根目录，可以通过以下网址下载：

    [`raw.githubusercontent.com/jadejs/jade/1.11.0/runtime.js`](https://raw.githubusercontent.com/jadejs/jade/1.11.0/runtime.js)

1.  然后，您需要将 `jade` 任务生成的 `templates.js` 文件包含到应用程序或页面中，以便使包含模板的 `JST` 全局变量可用。

1.  以下示例代码将渲染编译后的模板，使用一些样本数据并将结果存储在 `result` 变量中：

    ```js
    var result = JST'blog';
    ```

1.  `result` 变量现在应该包含以下 HTML：

    ```js
    <div class="blog">
      <h1>My Awesome Blog</h1>
        <h2>First Post</h2>
        <p>My very first post.</p>
        <h2>Second Post</h2>
        <p>This is getting old.</p>  
    </div>
    ```

## 还有更多...

`jade` 任务在基本模板编译的基础上提供了一些有用的选项，允许我们指定编译模板的命名空间，指示模板名称应该如何从文件名中派生，以及带有调试支持的模板编译。

### 指定编译模板的命名空间

默认情况下，编译后的模板存储在 `JST` 命名空间中，但可以通过使用 `namespace` 选项将其更改为任何我们喜欢的名称。在以下示例中，我们配置任务将模板存储在 `Templates` 命名空间中：

```js
jade: {
  blog: {
    options: {
      client: true,
      namespace: 'Templates'
    },
    src: 'blog.jade',
    dest: 'templates.js'
  }
}
```

### 指示模板名称应该如何从文件名中派生

可以使用 `processName` 选项来指示模板在命名空间中存储的名称应该如何从文件名中派生。`jade` 任务的默认行为是使用文件扩展名之前的所有内容。在以下示例中，我们指示整个文件名应该全部大写：

```js
jade: {
  blog: {
    options: {
      client: true,
      processName: function (filename) {
 return filename.toUpperCase();
 }
    },
    src: 'blog.jade',
    dest: 'templates.js'
  }
}
```

### 带有调试支持的模板编译

Jade 模板引擎为编译后的模板提供了额外的调试支持，可以通过 `compileDebug` 选项启用，如下例所示：

```js
jade: {
  blog: {
    options: {
      client: true,
      compileDebug: true
    },
    src: 'blog.jade',
    dest: 'templates.js'
  }
}
```

# 编译 Handlebars 模板

Handlebars 模板引擎简化了任何标记或其他可读文件的构建和维护。它熟悉的语法、可扩展性和无逻辑的方法提供了一个全面的模板构建体验。该引擎主要关注将模板编译成在 Web 应用程序前端广泛使用的 JST。

在这个菜谱中，我们将使用 `contrib-handlebars (0.8.0)` 插件来编译一个渲染简约博客的模板。

## 准备工作

在这个例子中，我们将使用我们在 第一章 中创建的基本项目结构，即 在项目中安装 Grunt 菜谱。如果您还不熟悉其内容，请务必参考。

## 如何操作...

以下步骤将引导我们创建一个简单的 Handlebars 模板，并设置 Grunt 以将其编译到另一个文件中包含的 JST。

1.  我们将按照 第一章 中提供的 *安装插件* 菜谱中的说明安装包含 `contrib-handlebars` 插件的包。

1.  让我们在项目目录中创建一个简单的 Handlebars 模板文件，命名为 `blog.hbs`，并给它以下内容：

    ```js
    <div class="blog">
      <h1>{{title}}</h1>
      {{#posts}}
        <h2>{{title}}</h2>
        <p>{{body}}</p>
      {{/posts}}
    </div>
    ```

1.  现在，我们可以在配置中添加以下 `handlebars` 任务，该任务将 `post.hbs` 模板编译到 `templates.js` 文件中的 JST 内：

    ```js
    handlebars: {
      blog: {
        src: 'blog.hbs',
        dest: 'templates.js'
      }
    }
    ```

1.  最后，我们可以使用 `grunt handlebars` 命令运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "handlebars:blog" (handlebars) task
    >> 1 file created.

    ```

1.  现在我们应该在项目目录中有一个名为 `templates.js` 的新文件，其中包含编译模板的 JST 代码。

## 它是如何工作的...

以下步骤展示了编译模板的使用方法，并显示了渲染结果的示例。

1.  首先，您需要将 Handlebars 运行时库包含到您想要使用编译模板的应用程序或页面中。

    ### 小贴士

    在撰写本文时，Handlebars 运行时库可以从以下 URL 下载：

    [`builds.handlebarsjs.com.s3.amazonaws.com/handlebars-v1.3.0.js`](http://builds.handlebarsjs.com.s3.amazonaws.com/handlebars-v1.3.0.js)

1.  然后，您需要将由 `handlebars` 任务生成的 `templates.js` 文件包含到应用程序或页面中，以创建包含模板的 `JST` 全局变量。

1.  以下示例代码将使用一些样本数据渲染编译后的模板，并将结果存储在 `result` 变量中：

    ```js
    var result = JST'blog.hbs';
    ```

1.  `result` 变量现在应该包含以下 HTML：

    ```js
    <div class="blog">
      <h1>My Awesome Blog</h1>
        <h2>First Post</h2>
        <p>My very first post.</p>
        <h2>Second Post</h2>
        <p>This is getting old.</p>  
    </div>
    ```

## 还有更多...

`handlebars` 任务在基本模板编译的同时提供了一些有用的选项，允许我们指定编译模板的命名空间，并指示模板名称应该如何从它们的文件名中派生。

### 指定编译模板的命名空间

默认情况下，编译后的模板存储在 `JST` 命名空间中，但可以通过使用 `namespace` 选项将其更改为任何我们喜欢的名称。在以下示例中，我们配置任务以将模板存储在 `Templates` 命名空间中：

```js
handlebars: {
  blog: {
    options: {
      namespace: 'Templates'
    },
    src: 'blog.hbs',
    dest: 'templates.js'
  }
}
```

### 指示模板名称应该如何从文件名中派生

`processName` 选项可用于指示模板在命名空间中存储的名称应如何从其文件名派生。`handlebars` 任务的默认行为是使用整个文件名。在以下示例中，我们指示它仍然使用文件名，但以大写字母形式：

```js
handlebars: {
  blog: {
    options: {
      processName: function (filename) {
 return filename.toUpperCase();
 }
    },
    src: 'blog.hbs',
    dest: 'templates.js'
  }
}
```

# 编译 Underscore 模板

除了提供的许多实用工具外，Underscore 库还包括一个简单、快速且灵活的模板引擎。这些模板通常编译为 JST 以在 Web 应用程序的前端使用。

在这个菜谱中，我们将使用 `contrib-jst (0.6.0)` 插件来编译一个渲染简约博客的模板。

## 准备工作

在这个示例中，我们将使用我们在 第一章 中 *在项目中安装 Grunt* 菜谱中创建的基本项目结构。如果您还不熟悉其内容，请务必参考它。

## 如何操作...

以下步骤将引导我们创建一个简单的 Underscore 模板并将其编译到另一个文件中的 JST 中。

1.  我们将按照 第一章 中 *安装插件* 菜谱提供的说明，安装包含 `contrib-jst` 插件的包。

1.  让我们在项目目录中创建一个简单的 Underscore 模板文件，命名为 `blog.html`，并给它以下内容：

    ```js
    <div class="blog">
      <h1><%= title %></h1>
      <% _.each(posts, function (post) { %>
        <div class="post">
          <h2><%= post.title %></h2>
          <p><%= post.body %></p>
        </div>
      <% }) %>
    </div>
    ```

1.  现在，我们可以将以下 `jst` 任务添加到我们的配置中，该任务将 `blog.html` 模板编译到我们项目目录中的 `templates.js` 文件中：

    ```js
    jst: {
      blog: {
        src: 'blog.html',
        dest: 'templates.js'
      }
    }
    ```

1.  最后，我们可以使用 `grunt jst` 命令运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "jst:blog" (handlebars) task
    >> 1 file created.

    ```

1.  现在，我们应该在我们的项目目录中有一个名为 `templates.js` 的新文件，其中包含我们编译模板的 JST 代码。

## 它是如何工作的...

以下步骤演示了编译模板的使用，并显示了一个渲染结果的示例。

1.  首先，您需要在希望使用编译模板的应用程序或页面中包含 Underscore 运行时库。

    ### 提示

    在撰写本文时，可以从以下 URL 下载 Underscore 运行时库：

    [`underscorejs.org/underscore-min.js`](http://underscorejs.org/underscore-min.js)

1.  然后，您需要将 `jst` 任务生成的 `templates.js` 文件包含到应用程序或页面中，以便使包含模板的 `JST` 全局变量可用。

1.  以下示例代码将渲染编译后的模板，使用一些示例数据并将结果存储在 `result` 变量中：

    ```js
    var result = JST'blog.html';
    ```

1.  `result` 变量现在应包含以下 HTML：

    ```js
    <div class="blog">
      <h1>My Awesome Blog</h1>
        <h2>First Post</h2>
        <p>My very first post.</p>
        <h2>Second Post</h2>
        <p>This is getting old.</p>  
    </div>
    ```

## 还有更多...

`jst` 任务在基本模板编译的同时提供了一些有用的选项，允许我们指定编译模板的命名空间并指示模板名称应如何从其文件名派生。

### 指定编译模板的命名空间

默认情况下，编译后的模板存储在 `JST` 命名空间中，但可以通过使用 `namespace` 选项将其更改为我们喜欢的任何内容。在以下示例中，我们配置任务将模板存储在 `Templates` 命名空间中：

```js
jst: {
  blog: {
    options: {
      namespace: 'Templates'
    },
    src: 'blog.html',
    dest: 'templates.js'
  }
}
```

### 指示模板名称应如何从文件名派生

可以使用 `processName` 选项来指示模板在命名空间中存储时应使用的名称应如何从其文件名派生。`jst` 任务的默认行为是使用整个文件名。在以下示例中，我们指示它仍然使用文件名，但使用大写字母：

```js
jst: {
  blog: {
    options: {
      processName: function (filename) {
 return filename.toUpperCase();
 }
    },
    src: 'blog.html',
    dest: 'templates.js'
  }
}
```

# 在 Handlebars 模板中使用部分

Handlebars 模板引擎提供的部分系统允许我们在不同的上下文中重用较小的模板代码片段。每当你在模板的不同位置看到重复的模式时，这可能是一个使用部分的好机会。

在本菜谱中，我们将使用 `contrib-handlebars (0.8.0)` 插件及其部分模板加载功能，在博客的两个不同部分中渲染具有类似结构的帖子。

## 准备工作

在本例中，我们将使用本章中 *编译 Handlebars 模板* 菜单中创建的基本项目结构。如果你还不熟悉其内容，请务必参考它。

## 如何操作...

以下步骤将指导我们创建一个帖子部分模板并修改我们的博客模板，以便渲染新的和旧的章节。

1.  默认情况下，`handlebars` 任务将以下划线开头的模板识别为部分模板并加载。让我们在我们的项目目录中创建一个名为 `_post.hbs` 的文件，它将包含我们的部分帖子模板：

    ```js
    <div class="post">
      <h3>{{title}}</h3>
      <p>{{body}}</p>
    </div>
    ```

1.  现在，我们将通过将新部分模板的文件名添加到任务应包含的模板列表中来修改任务的配置：

    ```js
    handlebars: {
      sample: {
        src: ['blog.hbs', '_post.hbs'],
        dest: 'templates.js'
      }
    }
    ```

1.  一旦设置好部分模板以便包含，我们就可以修改我们的 `blog.hbs` 模板以使用它。让我们修改它以渲染新的和旧的章节：

    ```js
    <div class="blog">
      <h1>{{title}}</h1>
      <h2>New Posts</h2>
     {{#newPosts}}
     {{>post}}
     {{/newPosts}}
     <h2>Old Posts</h2>
     {{#oldPosts}}
     {{>post}}
     {{/oldPosts}}
    </div>
    ```

1.  最后，我们可以使用 `grunt handlebars` 命令运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "handlebars:blog" (handlebars) task
    >> 1 file created.

    ```

1.  现在，我们应该在我们的项目目录中有一个修改后的 `templates.js` 文件版本，其中包含我们最新博客模板的 JST 代码。

## 它是如何工作的...

以下步骤演示了编译模板的使用并显示了一个渲染结果的示例。

1.  首先，你需要在希望使用编译模板的应用程序或页面中包含 Handlebars 运行时库。

    ### 小贴士

    在撰写本文时，Handlebars 运行时库可以从以下 URL 下载：

    [`builds.handlebarsjs.com.s3.amazonaws.com/handlebars-v1.3.0.js`](http://builds.handlebarsjs.com.s3.amazonaws.com/handlebars-v1.3.0.js)

1.  然后，你需要将`handlebars`任务生成的`templates.js`文件包含在应用程序或页面中，以便使包含模板的`JST`全局变量可用。

1.  以下示例代码将渲染编译后的模板，使用一些示例数据并将结果存储在`result`变量中：

    ```js
    var result = JST'blog.hbs';
    ```

1.  `result`变量现在应包含以下 HTML：

    ```js
    <div class="blog">
      <h1>My Awesome Blog</h1>
      <h2>New Posts</h2>
      <div class="post">
        <h3>Brand Spanking</h3>
        <p>Still has that new post smell.</p>
      </div>
      <div class="post">
        <h3>Shiny Post</h3>
        <p>Just got off the floor.</p>
      </div>
      <h2>Old Posts</h2>
      <div class="post">
        <h3>Old News</h3>
        <p>Not that relevant anymore.</p>
      </div>
      <div class="post">
        <h3>Way Back When</h3>
        <p>My very first post.</p>
      </div>
    </div>
    ```

## 还有更多...

`handlebars`任务提供了一些有用的选项，与部分模板的加载结合使用，允许我们在命名空间中使部分模板可用，指示部分模板应该如何识别，以及指示部分模板名称应该如何推导。

### 在命名空间中使部分模板可用

如果我们想在代码中直接使用部分模板，而不仅仅是其他模板中，我们可以通过使用`partialsUseNamespace`选项，就像以下示例中那样，在命名空间中使它们与其他模板一起可用：

```js
handlebars: {
  blog: {
    options: {
      partialsUseNamespace: true
    },
    src: ['blog.hbs', '_post.hbs'],
    dest: 'templates.js'
  }
}
```

### 指示如何识别部分模板

我们可能会达到想要更改部分模板识别方式的地步。这可以通过组合使用`partialRegex`和`partialsPathRegex`选项来完成。以下示例表明，所有在`partials`目录中找到的模板都应该被加载为部分模板：

```js
handlebars: {
  blog: {
    options: {
      partialRegex: /.*/,
 partialsPathRegex: /partials\//
    },
    src: ['blog.hbs', 'partials/post.hbs'],
    dest: 'templates.js'
  }
}
```

### 指示如何推导部分模板的名称

默认情况下，`handlebars`任务会从部分文件名中移除前导下划线和文件扩展名，以确定其名称。以下示例使用`processPartialName`选项来指示在使用之前应将整个文件名转换为大写：

```js
handlebars: {
  blog: {
    options: {
      processPartialName: function (filename) {
 return filename.toUpperCase();
 }
    },
    src: ['blog.hbs', '_post.hbs'],
    dest: 'templates.js'
  }
}
```

# 将 Jade 模板包裹在 AMD 模块中

在这个食谱中，我们将使用`contrib-jade (0.12.0)`插件及其`amd`选项来将我们的编译后的博客模板包裹在一个**异步模块定义**（**AMD**）模块中。

## 准备工作

在这个例子中，我们将使用本章中“编译 Jade 模板”食谱中创建的基本项目结构。如果你还不熟悉其内容，请务必参考它。

## 如何做到这一点...

以下步骤将引导我们修改配置，以便它将编译后的模板包裹在一个 AMD 模块中。

1.  首先，我们将通过添加`amd`选项来修改配置，以指示编译后的模板应该被包裹在一个 AMD 模块中：

    ```js
    jade: {
      blog: {
        options: {
          amd: true,
          client: true
        },
        src: 'blog.jade',
        dest: 'templates.js'
      }
    }
    ```

1.  使用`namespace`选项，我们现在也可以指定不再希望编译后的模板包含在`JST`命名空间中，因为它们现在将包含在自己的模块中：

    ```js
    jade: {
      blog: {
        options: {
          amd: true,
          client: true,
          namespace: false
        },
        src: 'blog.jade',
        dest: 'templates.js'
      }
    }
    ```

1.  现在，我们还可以将模板编译为不同的文件名，因为每个模板现在将代表其自己的文件：

    ```js
    jade: {
      blog: {
        options: {
          amd: true,
          client: true,
          namespace: false
        },
        src: 'blog.jade',
        dest: 'blog.js'
      }
    }
    ```

1.  最后，我们可以使用 `grunt jade` 命令来运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "jade:blog" (jade) task
    File blog.js created.

    ```

1.  现在应该在项目目录中有一个名为 `blog.js` 的文件，该文件包含了一个用 AMD 模块包裹的编译后的模板。

## 它是如何工作的...

以下步骤展示了编译后包裹的模板的使用方法，并显示了渲染结果的示例。

1.  首先，你需要确保你现有的 AMD 框架能够访问名为 `jade` 的 Jade 运行时库。这可能需要修改 AMD 框架的配置。

    ### 小贴士

    在撰写本文时，Jade 运行时库可在官方 Jade 仓库的根目录下找到，并且可以从以下 URL 下载：

    [`raw.githubusercontent.com/jadejs/jade/1.11.0/runtime.js`](https://raw.githubusercontent.com/jadejs/jade/1.11.0/runtime.js)

1.  然后，你需要确保由 `jade` 任务生成的 `blog.js` 文件能够被 AMD 框架访问，无论你更喜欢哪个名称或路径。

1.  以下示例代码将获取编译后的模板作为依赖项，使用一些示例数据渲染编译后的模板，然后将结果存储在 `result` 变量中：

    ```js
    define(['templates/blog'], function(blog) {
      var result = blog({
        title: 'My Awesome Blog',
        posts: [{
          title: 'First Post',
          body: 'My very first post.'
        }, {
          title: 'Second Post',
          body: 'This is getting old.'
        }]
      });
    });
    ```

1.  `result` 变量现在应该包含以下 HTML：

    ```js
    <div class="blog">
      <h1>My Awesome Blog</h1>
        <h2>First Post</h2>
        <p>My very first post.</p>
        <h2>Second Post</h2>
        <p>This is getting old.</p>  
    </div>
    ```

# 在 AMD 模块中包裹 Handlebars 模板

在这个菜谱中，我们将使用 `contrib-handlebars (0.8.0)` 插件及其 `amd` 选项来将编译后的博客模板包裹在 AMD 模块中。

## 准备工作

在这个例子中，我们将使用本章中 *编译 Handlebars 模板* 菜谱中创建的基本项目结构。如果你还不熟悉其内容，请务必参考它。

## 如何做到...

以下步骤将引导我们修改配置，以便将编译后的模板包裹在 AMD 模块中。

1.  首先，我们将通过添加 `amd` 选项来修改配置，以指示编译后的模板应该被包裹在 AMD 模块中：

    ```js
    handlebars: {
      blog: {
        options: {
          amd: true
        },
        src: 'blog.hbs',
        dest: 'templates.js'
      }
    }
    ```

1.  使用 `namespace` 选项，我们现在也可以指定我们不再希望编译后的模板包含在 `JST` 命名空间中，因为它们现在将包含在自己的模块中：

    ```js
    handlebars: {
      blog: {
        options: {
          amd: true,
          namespace: false
        },
        src: 'blog.hbs',
        dest: 'templates.js'
      }
    }
    ```

1.  我们现在也可以将模板编译到不同的文件名中，因为每个模板现在都将由其自己的文件表示：

    ```js
    handlebars: {
      blog: {
        options: {
          amd: true,
          namespace: false
        },
        src: 'blog.hbs',
        dest: 'blog.js'
      }
    }
    ```

1.  最后，我们可以使用 `grunt handlebars` 命令来运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "handlebars:blog" (handlebars) task
    >> 1 file created.

    ```

1.  现在应该在项目目录中有一个名为 `blog.js` 的文件，该文件包含了一个用 AMD 模块包裹的编译后的模板。

## 它是如何工作的...

以下步骤展示了编译后包裹的模板的使用方法，并显示了渲染结果的示例。

1.  首先，你需要确保你现有的 AMD 框架能够访问名为 `handlebars` 的 Handlebars 运行时库。这可能需要修改 AMD 框架的配置。

    ### 小贴士

    在撰写本文时，Handlebars 运行时库可以从以下网址下载：

    [`builds.handlebarsjs.com.s3.amazonaws.com/handlebars-v1.3.0.js`](http://builds.handlebarsjs.com.s3.amazonaws.com/handlebars-v1.3.0.js)

    在本提示中提到的运行时库的情况下，您将不得不使用垫片（shim），因为该库没有为与 AMD 模块一起使用而准备。您可以通过以下网址了解更多关于使用 RequireJS 框架进行垫片的信息：

    [`requirejs.org/docs/api.html#config-shim`](http://requirejs.org/docs/api.html#config-shim)

1.  然后，您需要确保由 `handlebars` 任务生成的 `blog.js` 文件在您选择的任何名称或路径下都可以被 AMD 框架访问。

1.  以下示例代码将获取编译模板作为依赖项，使用一些示例数据渲染编译模板，然后将结果存储在 `result` 变量中：

    ```js
    define(['templates/blog'], function(blog) {
      var result = blog({
        title: 'My Awesome Blog',
        posts: [{
          title: 'First Post',
          body: 'My very first post.'
        }, {
          title: 'Second Post',
          body: 'This is getting old.'
        }]
      });
    });
    ```

1.  `result` 变量现在应包含以下 HTML：

    ```js
    <div class="blog">
      <h1>My Awesome Blog</h1>
        <h2>First Post</h2>
        <p>My very first post.</p>
        <h2>Second Post</h2>
        <p>This is getting old.</p>  
    </div>
    ```

# 在 AMD 模块中包装 Underscore 模板

在本配方中，我们将使用 `contrib-jst (0.6.0)` 插件及其 `amd` 选项来将编译后的博客模板包装在一个 AMD 模块中。

## 准备工作

在本例中，我们将使用本章“编译 Underscore 模板”配方中创建的基本项目结构。如果您还不熟悉其内容，请务必参考。

## 如何操作...

以下步骤将引导我们修改配置，以便将我们的编译模板包装在 AMD 模块中。

1.  首先，我们将通过添加 `amd` 选项来修改配置，以指示编译模板应包装在 AMD 模块中：

    ```js
    jst: {
      blog: {
        options: {
          amd: true
        },
        src: 'blog.html',
        dest: 'templates.js'
      }
    }
    ```

1.  使用 `namespace` 选项，我们现在还可以指定我们不再希望编译模板包含在 `JST` 命名空间中，因为它们现在将包含在其自己的模块中：

    ```js
    jst: {
      blog: {
        options: {
          amd: true,
          namespace: false
        },
        src: 'blog.html',
        dest: 'templates.js'
      }
    }
    ```

1.  现在，我们还可以将模板编译为不同的文件名，因为每个模板现在都将位于其自己的文件中：

    ```js
    jst: {
      blog: {
        options: {
          amd: true,
          namespace: false
        },
        src: 'blog.html',
        dest: 'blog.js'
      }
    }
    ```

1.  最后，我们可以使用 `grunt jst` 命令运行任务，这应该会产生类似于以下内容的输出：

    ```js
    Running "jst:blog" (jst) task
    File blog.js created.

    ```

1.  现在应在项目目录中有一个名为 `blog.js` 的文件，其中包含包装在 AMD 模块中的编译模板。

## 工作原理...

以下步骤演示了编译和包装模板的使用，并显示了一个渲染结果的示例。

1.  首先，您需要确保您放置的 AMD 框架在启动时加载 Underscore 库，因为此插件创建的包装器不将其作为依赖项引用。

    ### 提示

    在撰写本文时，Underscore 运行时库可以从以下网址下载：

    [`underscorejs.org/underscore-min.js`](http://underscorejs.org/underscore-min.js)

    您可以在以下网址了解更多关于在 RequireJS 框架中指定启动依赖项的信息：

    [`requirejs.org/docs/api.html#config-deps`](http://requirejs.org/docs/api.html#config-deps)

1.  然后，你需要确保由 `jst` 任务生成的 `blog.js` 文件可以在你喜欢的任何名称或路径下被 AMD 框架访问。

1.  以下示例代码将获取编译后的模板作为依赖项，使用一些示例数据渲染编译后的模板，然后将结果存储在 `result` 变量中：

    ```js
    define(['templates/blog'], function(blog) {
      var result = blog({
        title: 'My Awesome Blog',
        posts: [{
          title: 'First Post',
          body: 'My very first post.'
        }, {
          title: 'Second Post',
          body: 'This is getting old.'
        }]
      });
    });
    ```

1.  `result` 变量现在应包含以下 HTML：

    ```js
    <div class="blog">
      <h1>My Awesome Blog</h1>
        <h2>First Post</h2>
        <p>My very first post.</p>
        <h2>Second Post</h2>
        <p>This is getting old.</p>  
    </div>
    ```

# 在 CommonJS 模块中包装 Handlebars 模板

在这个菜谱中，我们将使用 `contrib-handlebars (0.8.0)` 插件将编译后的博客模板包装在一个 **CommonJS** 模块中。

## 准备工作

在这个例子中，我们将使用本章中 *编译 Handlebars 模板* 菜谱中创建的基本项目结构。如果你还不熟悉其内容，请务必参考它。

## 如何操作...

以下步骤将引导我们修改配置，以便将编译后的模板包装在 CommonJS 模块中。

1.  首先，我们将通过添加 `amd` 选项来修改配置，表示编译后的模板应该被包装在 CommonJS 模块中。

    ```js
    handlebars: {
      blog: {
        options: {
          commonjs: true
        },
        src: 'blog.hbs',
        dest: 'templates.js'
      }
    }
    ```

1.  使用 `namespace` 选项，我们现在也可以指定不再希望编译后的模板包含在 `JST` 命名空间中，因为它们现在将包含在一个模块中：

    ```js
    handlebars: {
      blog: {
        options: {
          commonjs: true,
          namespace: false
        },
        src: 'blog.hbs',
        dest: 'templates.js'
      }
    }
    ```

1.  最后，我们可以使用 `grunt handlebars` 命令运行任务，它应该产生类似以下输出：

    ```js
    Running "handlebars:blog" (handlebars) task
    >> 1 file created.

    ```

1.  现在项目目录中应该有一个 `templates.js` 文件，其中包含包装在 CommonJS 模块中的编译后的模板。

## 他是如何工作的...

以下步骤展示了编译和包装模板的使用方法，并显示了渲染结果的示例。

1.  首先，我们必须确保 Handlebars 库已安装在本地的 CommonJS 支持环境中。以 Node.js 为例，模块必须使用 `npm install handlebars` 命令安装。

1.  接下来，我们将通过使用类似以下代码的方式，使代码片段中的模板可用，我们将导入模板：

    ```js
    var Handlebars = require('handlebars');
    ```

1.  然后，我们需要将模板导入到我们的代码中，并通过使用类似以下代码的方式将其传递给 Handlebars 库：

    ```js
    var templates = require('./templates')(Handlebars);
    ```

    ### 小贴士

    这段代码假设生成的 `templates.js` 文件与当前正在工作的代码文件位于同一路径上。

1.  现在我们已经导入了模板，我们可以使用它们。以下示例渲染了 `blog.hbs` 模板，并将结果存储在 `result` 变量中：

    ```js
    var result = templates'blog.hbs';
    ```

1.  `result` 变量现在应包含以下 HTML：

    ```js
    <div class="blog">
      <h1>My Awesome Blog</h1>
        <h2>First Post</h2>
        <p>My very first post.</p>
        <h2>Second Post</h2>
        <p>This is getting old.</p>
    </div>
    ```

# 在编译前修改 Jade 模板

有时候，你可能想在编译或渲染模板之前修改模板的内容。这通常是在你想要从模板中移除过多的空白空间，或者如果你想移除或替换你无法控制的模板部分时所需的。

在这个菜谱中，我们将利用 `contrib-jade (0.12.0)` 插件及其 `jade` 任务提供的 `processContent` 选项，在将模板渲染为 HTML 之前从我们的模板中移除空白字符。

## 准备工作

在这个示例中，我们将使用本章中 *编译 Jade 模板* 菜谱中创建的基本项目结构。如果您还不熟悉其内容，请务必参考它。

## 如何操作...

以下步骤将引导我们修改配置，以便在将模板渲染为 HTML 之前从模板中移除过多的空白字符。

1.  首先，我们可以向 `processContent` 选项提供一个空函数，该函数接收模板的内容并将其传递而不进行修改：

    ```js
    jade: {
      sample: {
        options: {
     processContent: function (content) {
     return content;
     }
     },
        src: 'index.jade',
        dest: 'index.html'
      }
    }
    ```

1.  然后，我们可以修改提供给 `processContent` 的函数的代码，以便从模板的内容中移除所有尾随空白字符：

    ```js
    processContent: function (content) {
      content = content.replace(/[\x20\t]+$/mg, '');
     content = content.replace(/[\r\n]*$/, '\n');
      return content;
    }
    ```

1.  如果我们现在使用 `grunt jade` 命令运行任务，我们将看到以下类似的输出：

    ```js
    Running "jade:sample" (jade) task
    File index.html created.

    ```

1.  在 `index.html` 文件中显示的渲染结果现在不应显示任何原始模板中存在的尾随空白字符的迹象。

# 在编译 Handlebars 模板之前进行修改

虽然这种情况并不常见，但可能会有这样的时候，您希望在编译或渲染模板之前修改模板的内容。这通常是在您希望从模板中移除过多的空白字符，或者如果您想移除或替换您无法控制的模板的某些部分时所需的。

在这个菜谱中，我们将利用 `contrib-handlebars (0.8.0)` 插件及其 `handlebars` 任务提供的 `processContent` 选项，在编译模板之前从我们的模板中移除空白字符。

## 准备工作

在这个示例中，我们将使用本章中 *编译 Handlebars 模板* 菜谱中创建的基本项目结构。如果您还不熟悉其内容，请务必参考它。

## 如何操作...

以下步骤将引导我们修改配置，以便在编译模板之前从模板中移除过多的空白字符。

1.  首先，我们可以向 `processContent` 选项提供一个空函数，该函数接收模板的内容并将其传递而不进行修改：

    ```js
    handlebars: {
      blog: {
        options: {
     processContent: function (content) {
     return content;
     }
     },
        src: 'blog.hbs',
        dest: 'templates.js'
      }
    }
    ```

1.  然后，我们可以修改提供给 `processContent` 的函数的代码，以便从模板的内容中移除所有前导和尾随空白字符：

    ```js
    processContent: function (content) {
      content = content.replace(/^[\x20\t]+/mg, '');
     content = content.replace(/[\x20\t]+$/mg, '');
     content = content.replace(/^[\r\n]+/, '');
     content = content.replace(/[\r\n]*$/, '\n');
      return content;
    }
    ```

1.  如果我们现在使用 `grunt handlebars` 命令运行任务，我们应该看到以下类似的输出：

    ```js
    Running "handlebars:blog" (handlebars) task
    >> 1 file created.

    ```

1.  在 `templates.js` 文件中显示的编译结果现在不应显示任何原始模板中存在的尾随空白字符的迹象。

# 在编译 Underscore 模板之前进行修改

虽然这种情况并不常见，但可能会有这样的时候，你希望在编译或渲染模板之前修改模板的内容。这通常是在你希望从模板中移除过多空白字符，或者如果你想移除或替换你无法控制的模板的某些部分时所需的。

在这个菜谱中，我们将利用`contrib-jst (0.6.0)`插件及其`jst`任务提供的`processContent`选项，在编译模板之前从模板中移除空白字符。

## 准备工作

在这个示例中，我们将使用本章“编译 Underscore 模板”菜谱中创建的基本项目结构。如果你还不熟悉其内容，请务必参考。

## 如何做到这一点...

以下步骤将指导我们修改配置，以便在编译模板之前移除模板中的过多空白字符。

1.  首先，我们可以向`processContent`选项提供一个空函数，该函数接收模板的内容并将其传递而不做任何修改：

    ```js
    jst: {
      blog: {
        options: {
     processContent: function (content) {
     return content;
     }
     },
        src: 'blog.html',
        dest: 'templates.js'
      }
    }
    ```

1.  然后，我们可以修改提供给`processContent`的函数代码，以便从模板内容中移除所有前导和尾随空白字符：

    ```js
    processContent: function (content) {
      content = content.replace(/^[\x20\t]+/mg, '');
     content = content.replace(/[\x20\t]+$/mg, '');
     content = content.replace(/^[\r\n]+/, '');
     content = content.replace(/[\r\n]*$/, '\n');
      return content;
    }
    ```

1.  如果我们现在使用`grunt jst`命令运行任务，我们应该看到以下类似的输出：

    ```js
    Running "jst:blog" (jst) task
    File templates.js created.

    ```

1.  现在在`templates.js`文件中编译的结果应该不再显示原始模板中存在的任何尾随空白字符的迹象。
