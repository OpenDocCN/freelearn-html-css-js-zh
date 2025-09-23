# 第二章。文件管理

在本章中，我们将介绍以下食谱：

+   复制文件

+   压缩文件

+   创建符号链接

+   合并文件

+   获取单个 URL

+   获取多个 URL

# 简介

在本章中，我们将专注于使用 Grunt 来处理我们的日常文件管理任务。很少有日子不涉及复制、压缩、下载或合并文件，借助 Grunt，我们可以自动化这些简单的任务。

# 复制文件

在我们追求项目自动化终极目标的过程中，我们很快就会想要自动化文件或目录从一个项目部分到另一个部分的复制。`contrib-copy(0.5.0)`插件为我们提供了这一功能，以及一些在复制文件过程中特别有用的其他选项。

## 准备工作

在此示例中，我们将使用我们在第一章中创建的基本项目结构，即第一章的*在项目中安装 Grunt*食谱。如果您还不熟悉其内容，请务必参考。

## 如何操作...

以下步骤将引导我们从一个目录复制单个文件到另一个目录。

1.  我们将按照第一章中*安装插件*食谱中提供的说明安装包含`contrib-copy`插件的包。

1.  让我们在我们的项目中设置一个示例文件，我们可以通过复制操作来尝试它。在`src`目录中创建一个名为`sample.txt`的文件，其内容如下：

    ```js
    Some sample data.
    ```

1.  现在，我们可以添加我们的`copy`任务配置，这将指导它将我们的示例文件从`src`目录复制到`dest`目录：

    ```js
    copy: {
      sample: {
        src: 'src/sample.txt',
        dest: 'dest/sample.txt'
      }
    }
    ```

    ### 小贴士

    注意，在复制操作中指定的目标目录如果尚不存在，将被自动创建。

1.  最后，我们可以运行`grunt copy`命令，它应该会提供输出，告知我们复制操作成功：

    ```js
    Running "copy:files" (copy) task
    Copied 1 files

    ```

1.  为了确认示例文件已被复制，我们现在可以使用我们最喜欢的编辑器打开`dest/sample.txt`文件，并确认其内容与原始文件的内容匹配。

## 更多...

`copy`任务提供了一些有用的配置选项，允许我们处理正在复制的文件的内容，复制源文件的权限，并设置目标文件的权限。

### 处理文件内容

`copy`任务提供了在复制文件时更改文件内容的选项。这个功能有许多有用的应用，例如渲染模板或替换与正则表达式匹配的文件部分。

在以下示例中，我们将简单地关注在复制的文件前添加一个字符串。我们将在此基础上构建我们在此食谱中早些时候的工作。

1.  我们将首先提供一个 `process` 选项的占位函数，该函数只是传递内容而不做任何修改。这将改变我们 `copy` 任务的配置，如下所示：

    ```js
    copy: {
      sample: {
        options: {
          process: function (content, path) {
     return content;
     }
        },
        src: 'src/sample.txt',
        dest: 'dest/sample.txt'
      }
    }
    ```

1.  现在，我们可以修改传递给 `process` 选项的函数，以便在正在复制的文件内容前添加一个字符串，使其看起来类似于以下内容：

    ```js
    process: function (content, path) {
      return 'Prepended string!\n' + content;
    }
    ```

1.  如果我们现在再次运行 `grunt copy` 命令，我们将看到文件正在被复制的确认信息，并且查看 `dest/sample.txt` 文件的内容，我们应该看到以下内容：

    ```js
    Prepended content
    Sample content.

    ```

### 复制源文件的权限

文件附加的权限通常非常重要。默认情况下，复制任务将忽略源文件的文件权限，但如果你希望它们应用于目标文件，则可以将 `mode` 选项设置为 `true`，如下例所示：

```js
copy: {
  sample: {
    options: {
 mode: true
    },
    src: 'src/sample.txt',
    dest: 'dest/sample.txt'
  }
}
```

### 设置目标文件的权限

如果你希望指定复制操作的目标文件应获得的权限，你可以通过将 `mode` 选项设置为所需的权限模式，使用传统的八进制格式的 Unix 权限来实现。

### 小贴士

你可以在 [`en.wikipedia.org/wiki/File_system_permissions`](http://en.wikipedia.org/wiki/File_system_permissions) 上阅读有关传统 Unix 权限约定的更多信息。

此外，还有一个方便的 Unix 文件权限计算器，可在 [`permissions-calculator.org/`](http://permissions-calculator.org/) 找到。

以下示例配置将使目标文件只对其所有者可读和可写：

```js
copy: {
  files: {
    src: 'src/sample.txt',
    dest: 'www/sample.txt',
    options: {
      mode: '0600'
    }
  }
}
```

# 文件压缩

在我们的本地系统以及整个万维网上，文件压缩是一种相当常见的做法。文件压缩最显著的应用是减少将通过网络传输的数据大小。

为了我们自己的目的，我们可能在某个时候想要自动化压缩一个我们希望可供下载或准备传输的文件或文件夹。`contrib-compress (0.10.0)` 插件为我们提供了这种功能，以及指定在过程中使用的存档类型和算法的选项。

## 准备工作

在本例中，我们将使用我们在 第一章 中 *在项目中安装 Grunt* 菜单创建的基本项目结构。如果你还不熟悉其内容，请务必参考。

## 如何操作...

以下步骤将指导我们使用 `compress` 任务的默认设置压缩单个样本文件。

1.  我们将按照 第一章 中 *安装插件* 菜单提供的说明，安装包含 `contrib-compress` 插件的包，*使用 Grunt 入门*。

1.  然后，我们可以创建一个示例文件来演示压缩功能。让我们下载一本文本格式的书籍作为示例，并将其保存为`book.txt`在我们的项目目录中。

    ### 小贴士

    简·奥斯汀的《傲慢与偏见》的文本版本可以从以下 URL 下载：

    [`www.gutenberg.org/cache/epub/1342/pg1342.txt`](http://www.gutenberg.org/cache/epub/1342/pg1342.txt)

1.  现在，我们可以为`compress`任务添加以下配置，这将把我们的示例文件压缩到`book.txt.gz`归档文件中：

    ```js
    compress: {
      sample: {
        src: 'book.txt.gz',
        dest: 'book.txt'
      }
    }
    ```

1.  为了尝试它，我们可以运行`grunt compress`命令，它应该产生类似于以下输出的结果：

    ```js
    Running "compress:sample" (compress) task
    Created book.txt.gz (314018 bytes)

    ```

1.  如果我们查看`book.txt.gz`文件，我们会看到它的大小大约是`book.txt`文件的一半。我们还可以进一步使用我们喜欢的归档提取器提取此文件，以确认它包含原始文件。

## 还有更多...

`compress`任务为我们提供了几个选项，允许我们归档一组文件，指定压缩模式，以及指定归档压缩级别。

### 归档一组文件

在我们的主要配方中，我们压缩了一个单独的文件，但将一组文件归档成一个单独的文件可能更为常见。为了做到这一点，我们修改`src`配置以指定应归档的文件，删除`dest`配置，然后使用`archive`选项提供一个归档文件的名称。

以下步骤继续我们在此配方中早期所做的操作，直到归档并将一组文件压缩成一个单独的文件。

1.  首先，我们将添加一些我们希望归档和压缩的更多文件。让我们将之前下载的书籍的名称从`book.txt`更改为`austen.txt`，并下载另外两本，分别重命名为`wells.txt`和`thoreau.txt`。

    ### 小贴士

    提到的两个示例书籍可以从以下 URL 按提到的顺序下载：

    [`www.gutenberg.org/cache/epub/36/pg36.txt`](http://www.gutenberg.org/cache/epub/36/pg36.txt)

    [`www.gutenberg.org/files/205/205-0.txt`](http://www.gutenberg.org/files/205/205-0.txt)

1.  现在，我们可以更改`compress`任务的配置，从仅压缩名为`book.txt`的文件，到包括所有三个下载的书籍在一个名为`books.zip`的归档文件中。我们通过更改其配置如下来实现：

    ```js
    compress: {
      sample: {
        options: {
     archive: 'books.zip'
     },
     src: ['austen.txt', 'wells.txt', 'thoreau.txt']
      }
    }
    ```

1.  让我们通过运行`grunt compress`命令来尝试它，它应该提供类似于以下输出的结果：

    ```js
    Running "compress:sample" (compress) task
    Created books.zip (786932 bytes)

    ```

1.  我们现在应该在项目目录中看到`books.zip`存档。为了确认它包含我们指定的所有文件，您可以使用您喜欢的归档工具来探索或提取它。

### 指定压缩模式

默认情况下，`compress`任务将根据在`archive`选项中指定的文件名的扩展名来确定其操作中使用的压缩模式。可以通过使用`mode`选项来改变这种行为。

以下示例将使用`zip`模式，尽管在`archive`选项中指定的文件扩展名是`tgz`：

```js
compress: {
  sample: {
    options: {
      mode: 'zip',
      archive: 'books.tgz'
    },
    files: {
      src: ['austen.txt', 'wells.txt', 'thoreau.txt']
    }
  }
}
```

### 小贴士

在出版时，`mode`选项支持的压缩模式有`gzip`、`deflate`、`deflateRaw`、`tar`、`tgz`和`zip`。

### 指定存档压缩级别

如果我们想在使用`zip`或`gzip`压缩模式时指定压缩率，我们可以使用`level`选项来做到这一点。该选项默认设置为`1`，但可以设置为从`1`到`9`的任何整数，后者表示更好的压缩效果，但性能会稍慢。

以下配置将把我们的书籍压缩到`9`级，这会花费一点时间，但可以将生成的存档大小减少大约 10%：

```js
compress: {
  sample: {
    options: {
      level: 9,
      archive: 'books.zip'
    },
    src: ['austen.txt', 'wells.txt', 'thoreau.txt']
  }
}
```

# 创建符号链接

在各种情况下创建文件和目录的引用可能非常有用，尤其是在我们想要分发文件的副本并保持它们更新，而无需在每次更改后手动复制它们时。

`contrib-symlink` `(0.3.0)`插件为我们提供了创建符号链接的功能。这是一个非常简单的插件，但它提供了所有标准的 Grunt 配置选项。

## 准备工作

在本例中，我们将使用在第一章中“在项目中安装 Grunt”配方中创建的基本项目结构，即*使用 Grunt 入门*。如果你还不熟悉其内容，请务必参考。

## 如何操作...

以下步骤将指导我们创建指向`www/img`目录内名为`assets/img/logo.png`的文件的符号链接。

1.  我们将按照第一章中“安装插件”配方中的说明来安装包含`contrib-symlink`插件的包。

1.  然后，我们可以添加以下配置，它将在`img/www`目录中创建符号链接：

    ```js
    symlink: {
      sample: {
        files: {
          'www/img/logo.png': 'assets/img/logo.png'
        }
      }
    }
    ```

1.  要尝试它，我们可以运行`grunt symlink`命令，它应该会通知我们符号链接的创建，输出类似于以下内容：

    ```js
    Running "symlink:files" (symlink) task
    >> Created 1 symbolic links.

    ```

1.  如果你现在查看`www/img`目录，你应该会看到一个名为`logo.png`的符号链接，它指向`assets/img/logo.png`文件。也许你还可以尝试更改图片的内容，以查看它们在符号链接中的反映。

# 文件连接

将文件连接起来的实践在所有类型的开发者中都很常见。无论是将分散的日志文件或许多较小的源文件合并到一个大的代码库中，文件端到端连接在日常工作中非常有用。

连接功能可以通过`contrib-concat` `(0.4.0)`插件提供。除了标准的 Grunt 配置外，它还提供了一组选项，以便根据我们的独特需求定制其行为。

## 准备中

在此示例中，我们将使用我们在第一章中“在项目中安装 Grunt”配方中创建的基本项目结构。如果您还不熟悉其内容，请务必参考它。

## 如何操作...

以下步骤将指导我们将三个 JavaScript 源文件连接到一个组合源文件中，这样我们只需要与我们的 Web 应用程序一起提供单个源文件。

1.  我们将按照第一章中“安装插件”配方提供的说明，安装包含`contrib-concat`插件的包，该配方位于*使用 Grunt 入门*。

1.  对于我们的示例，我们还需要三个 JavaScript 源文件。让我们称它们为`one.js`、`two.js`和`three.js`，并为每个文件提供以下内容，只需将`<filename>`替换为它们的名称：

    ```js
    var sample = 'Sample code from <filename>.';
    console.log(sample);
    ```

1.  现在，我们可以为`concat`任务添加以下配置，该配置将把我们的三个样本文件合并到`all.js`文件中：

    ```js
    concat: {
      sample: {
        files: {
          'all.js': ['one.js', 'two.js', 'three.js']
        }
      }
    }
    ```

1.  要尝试我们的任务，我们可以运行`grunt concat`命令，这将产生类似于以下内容的输出：

    ```js
    Running "concat:sample" (concat) task
    File all.js created

    ```

1.  现在，我们可以查看创建的`all.js`文件，其中将包含以下代码：

    ```js
    var sample = 'Sample code from one.js.';
    console.log(sample);
    var sample = 'Sample code from two.js.';
    console.log(sample);
    var sample = 'Sample code from three.js.';
    console.log(sample);
    ```

## 还有更多...

`concat`任务为我们提供了几个选项，允许我们使用自定义分隔符连接文件，在连接之前去除横幅，向连接结果添加横幅和页脚，以及在连接之前处理源文件的内容。

### 使用自定义分隔符连接文件

通常，在连接的文件之间有一个明显的分隔符会更受欢迎，这样在需要审查连接结果的内容时更容易区分它们。

默认情况下，`concat`任务将仅使用换行符分隔连接的文件，但可以通过`separator`选项提供自定义分隔符，如下例所示：

```js
concat: {
  sample: {
    options: {
      separator: '\n\n/* Next file */\n\n'
    },
    files: {
      'all.js': ['one.js', 'two.js', 'three.js']
    }
  }
}
```

使用前面的示例代码运行任务应该会产生一个内容类似于以下内容的`all.js`文件：

```js
var sample = 'Sample code from one.js.\n';
console.log(sample);

/* Next file */

var sample = 'Sample code from two.js.\n';
console.log(sample);

/* Next file */

var sample = 'Sample code from three.js.\n';
console.log(sample);
```

### 在连接之前去除横幅

在连接之前从 JavaScript 源文件中去除横幅的需求相当常见，因为它们通常占用相当多的额外存储空间，而且正如每个 Web 开发者都知道的，每一比特都很重要。

要自动从 JavaScript 源文件中删除横幅，我们可以按照以下示例使用 `stripBanners` 选项：

```js
concat: {
  sample: {
    options: {
      stripBanners: true
    },
    files: {
      'all.js': ['one.js', 'two.js', 'three.js']
    }
  }
}
```

### 向合并结果添加横幅和页脚

一旦生成了合并的结果，添加一个横幅或页脚以提供一些额外信息通常很有用。以下示例正是通过使用 `banner` 和 `footer` 选项来做到这一点的：

```js
concat: {
  sample: {
    options: {
      banner: '/* The combination of three files */\n',
 footer: '\n/* No more files to be combined */'
    },
    files: {
      'all.js': ['one.js', 'two.js', 'three.js']
    }
  }
}
```

使用前面的示例代码运行任务应该会生成一个包含以下内容的 `all.js` 文件：

```js
/* The combination of three files */
var sample = 'Sample code from one.js.\n';
console.log(sample);
var sample = 'Sample code from two.js.\n';
console.log(sample);
var sample = 'Sample code from three.js.\n';
console.log(sample);
/* No more files to be combined */
```

### 在合并之前处理源文件的内容

如果我们想修改合并的文件内容，`concat` 任务提供了在合并之前使用代码进行修改的选项。这可以通过向 `process` 选项提供一个函数来实现，该函数接收文件内容，修改它，并返回它。

以下示例利用此功能在合并的每个文件上方创建一个横幅，指示其文件名：

```js
concat: {
  sample: {
    options: {
      process: function (content, filename) {
 var banner = '/* ' + filename + ' */\n';
 return banner + content;
 }
    },
    files: {
      'all.js': ['one.js', 'two.js', 'three.js']
    }
  }
}
```

使用前面的示例代码运行任务应该会生成一个包含以下内容的 `all.js` 文件：

```js
/* one.js */
var sample = 'Sample code from one.js.\n';
console.log(sample);
/* two.js */
var sample = 'Sample code from two.js.\n';
console.log(sample);
/* three.js */
var sample = 'Sample code from three.js.\n';
console.log(sample);
```

# 获取单个 URL

如果您正在处理基于 Web 的项目，您可能需要在某个时候下载一些内容。下载文件是一个相当简单的操作，但自动化它可以从长远来看为您节省大量时间。

我们将使用相对流行的 `curl` `(2.0.2)` 插件从互联网下载资源。该插件提供了 `curl` 任务以下载单个文件。

## 准备工作

在本例中，我们将使用我们在 第一章 中创建的基本项目结构，即 *在项目中安装 Grunt* 菜谱，*使用 Grunt 入门*。如果您还不熟悉其内容，请务必参考。

## 如何操作...

以下步骤将指导我们使用 **OpenWeatherMap API** 下载伦敦市的当前天气并将其保存到 `london.json` 文件中。

1.  我们将按照 *安装插件* 菜谱中提供的说明安装包含 `curl` 插件的包，该菜谱位于 第一章，*使用 Grunt 入门*。

1.  现在，我们可以添加以下配置以将伦敦当前的天气信息下载到 `london.json` 文件中：

    ```js
    curl: {
          london: {
            src: 'http://api.openweathermap.org/'
              + 'data/2.5/weather?q=London',
            dest: 'london.json'
          }
        }
    ```

1.  为了测试我们的设置，我们可以运行 `grunt curl` 命令，它应该会提供类似于以下输出的结果：

    ```js
    Running "curl:london" (curl) task
    File "london.json" created.

    ```

1.  如果我们现在查看项目目录，我们应该看到下载的天气信息已保存到 `london.json` 文件中。

## 还有更多...

如果我们需要在下载内容时获得更多灵活性，`curl` 任务允许我们使用 `request` 包提供的 `request` HTTP 客户端实用工具来实现。可以在 `src` 配置中提供用于 `request` 实用工具的相同选项。

### 小贴士

你可以在以下网址了解更多关于 `request` HTTP 客户端实用程序的信息：

[`github.com/mikeal/request`](https://github.com/mikeal/request)

以下示例使用此功能为请求的查询字符串提供一个对象，而不是将其包含在 URL 字符串中：

```js
curl: {
  london: {
    src: {
      url: 'http://api.openweathermap.org/data/2.5/weather',
      qs: {q:'London'}
    },
    dest: 'london.json'
  }
}
```

# 获取多个 URL

如果你正在为你的项目自动化资源下载，你可能希望将多个资源下载到同一个目录中，并且很可能是从同一个网站下载。

我们将使用相对流行的 `curl` `(2.0.2)` 插件从互联网下载多个资源。它提供了一个专门的 `curl-dir` 任务来下载多个资源。

## 准备工作

在这个例子中，我们将使用我们在 第一章 中 *在项目中安装 Grunt* 菜单创建的基本项目结构。如果你还不熟悉其内容，请务必参考它。

## 如何操作...

以下步骤将指导我们将 Grunt 和 Node.js 的标志都下载到 `logos` 目录中。

1.  我们将按照 第一章 中 *安装插件* 菜单提供的说明来安装包含 `curl` 插件的包，*使用 Grunt 开始*。

1.  现在，我们可以添加以下 `curl-dir` 任务配置，它将下载两个标志到 `logos` 目录中：

    ```js
    'curl-dir': {
      logos: {
        src: [
          'http://nodejs.org/images/logo.png',
          'http://www.gruntjs.com/img/grunt-logo.png'
        ],
        dest: 'logos'
      }
    }
    ```

1.  要测试任务，我们可以使用 `grunt curl-dir` 命令来运行它，它应该提供类似于以下输出的结果：

    ```js
    Running "curl-dir:logo" (curl-dir) task
    Files "logos/logo.png", "logos/grunt-logo.png" created.

    ```

1.  现在我们应该在项目路径中有一个名为 `logos` 的目录，其中包含 Grunt 和 Node.js 的标志。

## 更多内容...

`curl-dir` 任务为我们提供了几个选项，允许我们下载类似的 URL，将下载的文件保存到修改后的文件名中，并使用特殊请求选项下载文件。

### 下载类似的 URL

同时，我们可能想要下载多个非常相似的 URL。在这种情况下，我们可以利用 `curl-dir` 任务提供的花括号扩展支持。

以下配置示例说明了我们如何下载伦敦、巴黎和东京的天气信息，而无需重复每个 URL 的起始部分：

```js
'curl-dir': {
  weather: {
    src: 'http://api.openweathermap.org/'
      + 'data/2.5/weather?q={London,Paris,Tokyo}',
    dest: 'weather'
  }
}
```

### 将下载的文件保存到修改后的文件名中

保存下载的资源到不同于其 URL 中包含的文件名也是相当常见的。`curl-dir` 任务提供的 `router` 配置允许我们指定一个函数，该函数接收我们下载的 URL 并返回资源应该保存到的文件名。

以下配置示例说明了我们如何下载三个城市的天气信息，并将每个结果保存到仅包含城市名称和 `.json` 文件扩展名的文件中：

```js
'curl-dir': {
  weather: {
    router: function (url) {
 var city = url.slice(url.indexOf('=') + 1, url.length);
 return city + '.json';
 },
    src: 'http://api.openweathermap.org/'
      + 'data/2.5/weather?q={London,Paris,Tokyo}',
    dest: 'weather'
  }
}
```

### 使用特殊请求选项下载文件

如果我们需要在下载内容请求中增加更多的灵活性，`curl-dir`任务允许我们通过使用`request`包提供的`request` HTTP 客户端工具来实现这一点。可以为`request`工具提供的相同选项在`src`配置中也可以提供。

以下配置示例通过将查询参数作为对象而不是附加到 URL 的字符串来利用此功能，同时也展示了如何使用之前讨论过的`router`配置来设置此类环境：

```js
'curl-dir': {
  weather: {
    router: function (src) {
      return src.qs.q + '.json';
    },
    src: [{
      url: 'http://api.openweathermap.org/data/2.5/weather',
      qs: {q:'London'}
    }, {
      url: 'http://api.openweathermap.org/data/2.5/weather',
      qs: {q:'Paris'}
    }],
    dest: 'weather'
  }
}
```
