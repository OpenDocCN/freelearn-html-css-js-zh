# 第十四章：最终自动化和流程优化

在这最后一章中，我们将讨论在创建 CSS 代码过程中重复流程的最终自动化。有很多流程可以自动化，但是意识到它是否可以做到以及使用的工具的知识是必不可少的。在本章中，我们将专注于工具以及如何在 Gulp 任务运行器中实现自动化。

在本章中，我们将涵盖以下主题：

+   视网膜和移动设备上的图像

+   如何识别未使用的 CSS

+   如何压缩代码

+   如何从页面列表中制作截图以便快速概览

+   如何使用 Jade 模板的基础知识并将其编译到 Gulp 中

# Gulp

在本书的开头，我介绍了 Gulp 作为 SASS 的入门。但是仅仅使用 Gulp 来编译 SASS 可能是浪费时间的。在本章中，我们将向 Gulp 添加更多任务，这些任务可以作为前端开发人员使用，并将帮助你优化你的代码。

# Jade 作为你的模板引擎

在大型项目的情况下编写 HTML 文件可能会有问题。页面的可重复元素的维护，比如主导航页脚边栏，在需要处理 10 个文件时可能会成为一个问题。每次你想要更改页脚中的内容，你都需要更新 10 个文件。当一个项目有 50 个模板时，情况会变得更加复杂。你可以开始使用 PHP 或任何包含重复代码部分的文件的语言，或者使用其中一个模板语言。有多种模板系统。以下是一些知名和时髦的模板系统：

+   手柄

+   HAML

+   Jade

+   Slim

让我们专注于 Jade。为什么？因为有以下功能：

+   混合支持

+   主模板

+   文件的部分化

+   缩进的语法（类似于 SASS）

## 安装和使用 Jade

Jade 是通过 node 包管理器安装的。你可以用以下命令安装它：

```css
npm install jade --global
```

如果你想编译一些文件，只需要调用 HTML 文件如下：

```css
jade filename.html
```

更多信息，我建议你查看 Jade 模板系统的官方文档[`jade-lang.com/`](http://jade-lang.com/)。

## Jade 的基础知识

理论介绍很重要，但让我们试着将代码的这部分描述成 Jade：

```css
<nav>
   <ul>
       <li><a href="#">Home</a></li>
       <li><a href="#">About</a></li>
       <li><a href="#">Contact</a></li>
   </ul>
</nav>
```

在 Jade 中，它会是这样的：

```css
nav
   ul
       li
           a(href="#") Home
       li
           a(href="#") About
       li
           a(href="#") Contact
```

你可以看到，你不需要考虑标准的 HTML 问题“我的标签关闭了吗？”缩进会跟踪标签的开启和关闭。你想要附加到标签中的每个文本都会出现在标签描述（名称和属性）后的空格后。让我们看一下代码的这部分：

```css
a(href="#") Home
```

这部分代码将被编译成：

```css
<a href="#">Home</a>
```

如你所见，在 Jade 中，属性（`href`）出现在元素名称（`a`）之后，用括号括起来。让我们看一下我们将要翻译成 Jade 的 HTML 代码的下一部分：

```css
<head>
    <meta charset="utf-8">
    <title>Page title</title>
    <link rel="stylesheet" href="css/main.css" media="screen" title="no title" charset="utf-8">
</head>
```

这部分代码将在所有页面上重复出现，因为它包含了我们 HTML 的`head`标签。在 Jade 中，它会是这样的：

```css
head
   meta(charset="utf-8")
   title Page title
   link(rel="stylesheet", href="css/main.css", media="screen", title="no title", charset="utf-8")
```

在这里你可以看到如何向 HTML 元素添加更多属性。在`link`元素中，括号中的每个属性都用逗号分隔。

代码的下一部分与带有类和 ID 的 DOM 元素有关：

```css
<main id="main">
   <article class="main--article">
       <a href="#">
           <img src="img/error_log.png" alt=""/>
           <span class="comments"></span>
       </a>
       <h3>Lorem ipsum dolor sit amet, consectetur adipisicing elit</h3>
       <p>
           sed do eiusmod tempor incididunt ut labore et dolore
       </p>
       <a href="#" class="readmore">Read more</a>
   </article>
</main>
```

在 Jade 中，代码看起来是这样的：

```css
main#main
   article.main--article
       a(href="#")
           img(src="img/error_log.png", alt="Error log")
           .comments
       h3 Lorem ipsum dolor sit amet
       p sed do eiusmod tempor incididunt ut labore et dolore 
       a(href="#").readmore Read more
```

你可以看到，你不需要描述这部分：

```css
<main id="main">
```

这是写成这样的：

```css
main(id="main")
```

在 Jade 中有一个简短的形式：

```css
main#main
```

类似的情况也适用于类：

```css
<article class="main--article">
```

你也可以使用一个简短的形式：

```css
article.main--article
```

这种简短的方法使 Jade 易于理解，因为它基于 CSS 中使用的选择器。

## Jade 中的混合

Jade 中的混合很有用，特别是当网页上有一些可重复的元素时。例如，可能是一些带有`href`的小元素`a`：

```css
mixin link(href, name)
   a(href= href)=name
```

现在我们需要做的就是将它添加到你的模板中：

```css
+link("url", "Your link")
```

在你编译的文件中，你会看到：

```css
<a href="url">Your link</a>
```

## 在 Jade 中包含和扩展功能

如前所述，我们可以将代码的部分保存在单独的文件中。这样做的最简单方法是使用`include`方法。假设我们已经在文件`navigation.jade`中定义了主要的`nav`，并且我们希望在我们的模板中添加其内容。代码如下：

文件名是：`navigation.jade`

```css
nav
   ul
       li
           a(href="#") Home
       li
           a(href="#") About
       li
           a(href="#") Contact
```

文件名是：`template.jade`

```css
doctype html
html
   head
       meta(charset="utf-8")
       title Page title
       link(rel="stylesheet", href="css/main.css", media="screen", title="no title", charset="utf-8")

   body
       include _navigation.jade
```

当您编译`template.jade`时，您将得到：

```css
<!DOCTYPE html>
<html>
<head>
   <meta charset="utf-8">
   <title>Page title</title>
   <link rel="stylesheet" href="css/main.css" media="screen" title="no title" charset="utf-8">
</head>
<body>
<nav>
   <ul>
       <li><a href="#">Home</a></li>
       <li><a href="#">About</a></li>
       <li><a href="#">Contact</a></li>
   </ul>
</nav>
</body>
</html>
```

这是一个很好的时机来使用可以扩展的主布局。这可以通过代码操作来完成。第一次操作必须在主模板中进行 - 定义一个将在我们的 HTML 文件中交换的块。第二次需要在将代表最终 HTML 文件的文件中完成 - 指定将扩展的主模板。代码如下：

文件名是：`master.jade`

```css
doctype html
html
   head
       meta(charset="utf-8")
       title Page title
       link(rel="stylesheet", href="css/main.css", media="screen", title="no title", charset="utf-8")

   body
       include _navigation.jade
       block content
```

文件名是：`index.jade`

```css
extends master

block content
   h1 Content
```

编译后的文档：

```css
<!DOCTYPE html>
<html>
<head>
   <meta charset="utf-8">
   <title>Page title</title>
   <link rel="stylesheet" href="css/main.css" media="screen" title="no title" charset="utf-8">
</head>
<body>
<nav>
   <ul>
       <li><a href="#">Home</a></li>
       <li><a href="#">About</a></li>
       <li><a href="#">Contact</a></li>
   </ul>
</nav>
<h1>Content</h1>
</body>
</html>
```

## 在 gulp.js 中使用 Jade

要在`gulpfile.js`中创建或添加 Jade 任务，您需要使用`npm`安装特定的包：`gulp-jade`。要这样做，请使用以下命令：

```css
npm install --save gulp-jade
```

然后，您需要在`gulpfile.js`中定义一个新任务，并为模板添加一个监视器，该模板将存储在`src/jade`目录中。以下是来自本书第一章的扩展`gulpfile.js`的清单：

```css
var gulp = require('gulp'),
   sass = require('gulp-sass'),
   jade = require('gulp-jade');

gulp.task('sass', function () {
   return gulp.src('src/css/main.sass')
       .pipe(sass().on('error', sass.logError))
       .pipe(gulp.dest('dist/css/main.css'));
});

gulp.task('jade', function() {
   gulp.src('src/jade/*.jade')
       .pipe(jade())
       .pipe(gulp.dest('dist/'));
});

gulp.task('default', function () {
   gulp.watch('src/sass/*.sass', ['sass']);
   gulp.watch('src/jade/*.jade', ['jade']);
});
```

它会表现如何？每当您更改`src/jade`文件夹中的任何文件时，编译后的文件将会出现在`dist`文件夹中。当然，如果您愿意，这种结构可以更改；这只是使用示例。随意更改它！

# UnCSS

有多少次你面对过这样的情况：HTML 中没有使用某些类/选择器，但在 CSS 代码中有描述？每当您的项目更改或重新设计时，这种情况都会发生。例如，您的任务是删除某些部分并在 HTML 代码中添加几行。因此，您将添加一些 CSS 代码，然后删除其中的一些。但您确定 CSS 代码不包含未使用的 CSS 代码部分吗？UnCSS 将帮助您完成此任务。要安装它，您需要执行此命令：

```css
npm install -g uncss
```

让我们来看看在`npm`命令中使用的标志：

| 标志 | 描述 |
| --- | --- |
| `-g` | 全局安装 |
| `--save` | 本地安装这些包将出现在`package.json`的`dependencies`部分。这些包是在生产中运行应用程序所需的。 |
| `--save-dev` | 本地安装这些包将出现在`package.json`的`devDependencies`部分。这些包是用于开发和测试过程的。 |

## 在 Gulp 中集成 UnCSS

首先，我们需要通过`npm`安装`gulp-uncss`：

```css
npm install --save gulp-uncss
```

现在，我们需要在`gulpfile.js`中添加新的任务。我们需要在项目中创建一个测试阶段，该阶段将存储在`test`目录中。您需要这些新任务来基于`uncss`进行处理：

```css
gulp.task('clean-css-test', function () {
   return gulp.src('test/css/main.css', {read: false})
       .pipe(rimraf({force: true}));
});

gulp.task('jade-test', function() {
   return gulp.src('src/jade/templates/*.jade')
       .pipe(jade())
       .on('error', gutil.log)
       .pipe(gulp.dest('test/'));
});

gulp.task('sass-test',['clean-css-test'], function () {
   return gulp.src('src/sass/main.sass')
       .pipe(sass().on('error', sass.logError))
       .pipe(gulp.dest('test/css/'));
});

gulp.task('uncss',['jade-test', 'sass-test'], function () {
   return gulp.src('test/css/main.css')
       .pipe(uncss({
           html: ['test/**/*.html']
       }))
       .pipe(gulp.dest('./test/uncss'));
});
```

要运行`uncss`任务，您需要使用以下命令：

```css
gulp uncss
```

此命令将执行以下任务：

+   将 Jade 文件编译到`test`文件夹中

+   从`test`文件夹中删除旧的 CSS 文件

+   将 SASS 文件编译到`test`文件夹中

+   运行`uncss`任务，并将文档保存为`test/uncss`文件夹中仅使用的 CSS 部分

现在我们需要实时测试。我们将准备一个简短的测试环境。

以下是文件的结构：

```css
├── jade
│   ├── master_templates
│   │   └── main.jade
│   ├── partials
│   │   ├── footer.jade
│   │   └── navigation.jade
│   └── templates
│       ├── about.jade
│       ├── contact.jade
│       └── index.jade
└── sass
    └── main.sass
```

代码如下：

文件名是：`main.jade`

```css
doctype html
html
   head
       meta(charset="utf-8")
       title Page title
       link(rel="stylesheet", href="css/main.css", media="screen", title="no title", charset="utf-8")

   body
       include ../partials/navigation
       block content
       include ../partials/footer
```

文件名是：`navigation.jade`

```css
nav
   ul
       li
           a(href="#") Home
       li
           a(href="#") About
       li
           a(href="#") Contact
```

文件名是：`footer.jade`

```css
footer
   p Copyright fedojo.com
```

文件名是：`index.jade`

```css
extends ../master_templates/main

block content
   .main
       p Test of INDEX page
```

文件名是：`about.jade`

```css
extends ../master_templates/main

block content
   .main
       h1 Test of ABOUT page
```

文件名是：`contact.jade`

```css
extends ../master_templates/main

block content
   .main
       h1 Test of CONTACT page
```

文件名是：`main.sass`

```css
body
background: #fff

p
color: #000

.header
background: #000
color: #fff

.footer
background: #000
color: #fff

header
background: #000
color: #fff

footer
background: #000
color: #fff
```

现在，让我们检查这个过程对我们是否有利。这是从 SASS 编译的文件：

```css
body {
 background: #fff; 
}

p {
 color: #000; 
}

.header {
 background: #000;
 color: #fff;
 }

.footer {
 background: #000;
 color: #fff; 
}

header {
 background: #000;
 color: #fff; 
}

footer {
 background: #000;
 color: #fff; 
}
```

此文件由`uncss`检查，它查看了所有模板（`index.html`，`about.html`和`contact.html`）：

```css
body {
 background: #fff;
}

p {
 color: #000;
}

footer {
 background: #000;
 color: #fff;
}
```

我们的新命令是用 Gulp 构建的，删除了所有不必要的 CSS 声明。

# 压缩 CSS

缩小文件的过程主要应该用于生产代码。在开发过程中，对缩小的文件进行操作会很困难，因此我们只需要为生产代码进行缩小。可以通过添加适当的标志（`--compressed`）来启用 SASS 或 Compass 编译中的缩小。我们还将使用一个外部工具，在 `uncss` 过程之后对代码进行缩小。现在我们需要安装 `gulp-clean-css`：

```css
npm install --save gulp-clean-css
```

现在，缩小 `uncss` 过程的结果。我们将创建一个 `prod` 目录，其中存储项目的最终版本。现在让我们导入 `gulp-clean-css`：

```css
cleanCSS = require('gulp-clean-css')
```

让我们在 `gulpfile.js` 中创建所需的部分：

```css
gulp.task('clean-css-production', function () {
   return gulp.src('prod/css/main.css', {read: false})
       .pipe(rimraf({force: true}));
});

gulp.task('sass-production',['clean-css-production'], function () {
   return gulp.src('src/sass/main.sass')
       .pipe(sass().on('error', sass.logError))
       .pipe(uncss({
           html: ['prod/**/*.html']
       }))
       .pipe(cleanCSS())
       .pipe(gulp.dest('prod/css/'));
});

gulp.task('jade-production', function() {
   return gulp.src('src/jade/templates/*.jade')
       .pipe(jade())
       .pipe(gulp.dest('prod/'));
});

gulp.task('production',['jade-production', 'sass-production']);
```

# 最终的自动化工具

现在我们需要将之前创建的所有任务汇总到一个文件中。`gulp` 项目的核心是两个文件：`package.json`，它汇总了所有项目依赖，以及 `gulpfile`，你可以在其中存储所有任务。以下是任务：

文件名是：`package.json`

```css
{
 "name": "automatizer",
 "version": "1.0.0",
 "description": "CSS automatizer",
 "main": "gulpfile.js",
 "author": "Piotr Sikora",
 "license": "ISC",
 "dependencies": {
   "gulp": "latest",
   "gulp-clean-css": "latest",
   "gulp-jade": "latest",
   "gulp-rimraf": "latest",
   "gulp-sass": "latest",
   "gulp-uncss": "latest",
   "gulp-util": "latest",
   "rimraf": "latest"
 }
}
```

文件名是：`gulpfile.json`

```css
var gulp = require('gulp'),
   sass = require('gulp-sass'),
   jade = require('gulp-jade'),
   gutil = require('gulp-util'),
   uncss = require('gulp-uncss'),
   rimraf = require('gulp-rimraf'),
   cleanCSS = require('gulp-clean-css');

gulp.task('clean-css-dist', function () {
   return gulp.src('dist/css/main.css', {read: false})
       .pipe(rimraf({force: true}));
});

gulp.task('clean-css-test', function () {
   return gulp.src('test/css/main.css', {read: false})
       .pipe(rimraf({force: true}));
});

gulp.task('sass',['clean-css-dist'], function () {
   return gulp.src('src/sass/main.sass')
       .pipe(sass().on('error', sass.logError))
       .pipe(gulp.dest('dist/css/'));
});

gulp.task('jade', function() {
   return gulp.src('src/jade/templates/*.jade')
       .pipe(jade())
       .pipe(gulp.dest('dist/'));
});

gulp.task('jade-test', function() {
   return gulp.src('src/jade/templates/*.jade')
       .pipe(jade())
       .on('error', gutil.log)
       .pipe(gulp.dest('test/'));
});

gulp.task('sass-test',['clean-css-test'], function () {
   return gulp.src('src/sass/main.sass')
       .pipe(sass().on('error', sass.logError))
       .pipe(gulp.dest('test/css/'));
});

gulp.task('uncss',['jade-test', 'sass-test'], function () {
   return gulp.src('test/css/main.css')
       .pipe(uncss({
           html: ['test/**/*.html']
       }))
       .pipe(gulp.dest('test/uncss'));
});

gulp.task('clean-css-production', function () {
   return gulp.src('prod/css/main.css', {read: false})
       .pipe(rimraf({force: true}));
});

gulp.task('sass-production',['clean-css-production'], function () {
   return gulp.src('src/sass/main.sass')
       .pipe(sass().on('error', sass.logError))
       .pipe(uncss({
           html: ['prod/**/*.html']
       }))
       .pipe(cleanCSS())
       .pipe(gulp.dest('prod/css/'));
});

gulp.task('jade-production', function() {
   return gulp.src('src/jade/templates/*.jade')
       .pipe(jade())
       .pipe(gulp.dest('prod/'));
});

gulp.task('production',['jade-production', 'sass-production']);

gulp.task('default', function () {
   gulp.watch('src/sass/*.sass', ['sass']);
   gulp.watch('src/jade/*.jade', ['jade']);
});
```

# 总结

在本章中，我们讨论了 Jade 模板系统的基础知识。我们看到了如何将其添加到前端开发人员的工作流程中。基于模板系统，你现在可以将 UnCSS 包含到你的流程中，并从 CSS 文件中删除不必要的代码。然后我们对最终结果进行了缩小，并创建了生产代码。

你可以将这个自动化工具视为项目的起点，并根据自己的项目进行调整。你也可以添加新功能并不断完善它。
