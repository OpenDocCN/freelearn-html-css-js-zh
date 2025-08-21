# 第八章：使用 Keystone CMS 创建博客

在本章中，我们讨论了完全使用 Node.js 制作的 CMS，称为**Keystone**的用法。

**KeystoneJS**自述为一个创建数据库驱动网站的开源平台。它已经有一个构建 Web 应用程序和强大博客的核心引擎，但它远不止于此。使用 Keystone.js 框架可以构建任何东西。

Keystone CMS 的主要吸引力之一是它使用 Express 框架和 Mongoose ODM，这两个工具我们在本书中已经使用过。

由于它是一个非常新的框架，它只有一个简单的默认主题，使用了 Bootstrap 框架，但是 Keystone 团队计划包括自定义新主题的选项，这将在不久的将来实现。

Keystone 使用了模型视图模板模式，非常类似于模型视图演示等模式。

在这一章中，我们将看到如何使用框架的所有功能构建一个新主题，以及如何通过新功能扩展它。

在这一章中，我们将涵盖以下主题：

+   安装 KeystoneJS

+   KeystoneJS 的结构和特性

+   如何使用简单样式定制

+   处理主题以及如何创建新主题

+   扩展核心功能以创建模型和视图。

# 我们正在构建什么

对于本章，我们将以一个简单的博客作为基础。我们将看到如何扩展它并创建可以通过控制面板管理的新页面，并且我们将得到一个与以下图像非常相似的结果：

![我们正在构建什么](img/image_08_001.jpg)

Keystone 博客主题首页

# 安装 Keystone 框架

与之前的章节一样，我们将使用官方的 Keystone.js yeoman 生成器。

### 提示

您可以在此链接找到有关 KeystoneJS 的更多信息：[`keystonejs.com/`](http://keystonejs.com/)。

让我们安装生成器。打开您的终端/Shell 并输入以下命令：

```js
npm install keystone -g

```

# 创建脚手架应用程序

现在是时候创建一个新文件夹并开始开发我们的博客应用程序了：

1.  创建一个名为 chapter-08 的文件夹。

1.  在 chapter-08 文件夹中打开您的终端/Shell，并输入以下命令：

```js
yo keystone

```

在此命令之后，keystone.js 将触发一系列关于应用程序基本配置的问题；您必须回答这些问题，如下截图所示：

![创建脚手架应用程序](img/image_08_002.jpg)

Keystone 生成器的提示问题

1.  在所有生成器任务结束后，我们可以在终端窗口上看到以下输出：

```js
Your KeystoneJS project is ready to go!
 For help getting started, visit http://keystonejs.com/guide
 We've included a test Mandrill API Key, which will simulate
        email 
sending but not actually send emails. Please replace
        it with your own 
when you are ready.
 We've included a demo Cloudinary Account, which is reset daily. 
 Please configure your own account or use the Local Image field
        instead 
before sending your site live.
 To start your new website, run "npm start".

```

请注意，在启动应用程序之前，我们需要纠正两个小错误。在撰写本文时，生成器存在此故障；但是，当书籍发布时，这个问题应该已经被修复。如果没有，这是解决此问题的方法。

## 修复 lint 错误和 admin 对象名称

1.  在项目根目录中打开 gulpfile.js 并删除有关 lint 任务的行：

```js
      watch:lint

```

1.  修复管理用户名，打开根文件夹中的 Keystone.js 文件并替换以下代码：

```js
      keystone.set('nav', { 
        posts: ['posts', 'post-categories'], 
        galleries: 'galleries', 
        enquiries: 'enquiries', 
        userAdmins: 'user-admins' 
      }); 

```

就这些了，我们已经有了我们的博客。让我们来检查一下结果。

# 运行 Keystone 博客

1.  打开终端/Shell 并输入以下命令：

```js
gulp 

```

1.  转到 http://localhost:3000/；您应该看到以下结果：![运行 Keystone 博客](img/image_08_003.jpg)

Keystone 主页

如前所述，界面非常简单。它可以查看生成器生成的默认信息，包括有关用户和密码的信息。

1.  点击右上角的**登录**链接，并使用上一个截图中的用户名和密码填写登录表单。结果将是控制面板，如下图所示：![运行 Keystone 博客](img/image_08_004.jpg)

Keystone 控制面板

每个链接都有一个表单，用于插入博客的数据，但现在不用担心这个；在本章后面，我们将看到如何使用管理面板。

正如我们在之前的图片中所看到的，布局非常简单。然而，这个框架的亮点不是它的视觉外观，而是它的核心引擎构建强大应用程序的能力。

### 提示

您可以在官方网站[`keystonejs.com/`](http://keystonejs.com/)上了解更多关于 Keystone 的信息。

# Keystone 引擎的解剖

在我们直接进入代码之前，我们将了解 Keystone 的目录结构是如何工作的。

启动应用程序后，我们将得到以下结果：

![Keystone 引擎的解剖](img/image_08_005.jpg)

Keystone 目录结构

这里是每个目录/文件夹的描述：

| **文件夹名称** | **文件夹路径** | **描述** |
| --- | --- | --- |
| 模型 | /models/ | 应用程序数据库模型。 |
| 公共 | /public/ | 图像、JavaScript、样式表和字体。 |
| 路由 | /routes//routes/views | 视图控制器（在 Restful API 上，我们可以使用一个名为 API 的文件夹）。 |
| 模板 | /templates//templates/emails//templates/layouts//templates/mixins//templates/views | 应用程序视图模板。 |
| 更新 | /updates/ | 迁移脚本和数据库填充。 |

此外，我们在根文件夹中有以下文件：

+   .editorconfig：设置编辑器的缩进

+   .env：设置 Cloudnary Cloud 凭据

+   .gitignore：Git 源控制的忽略文件

+   gulpfile.js：应用程序任务

+   keystone.js：引导应用程序

+   package.json：项目配置和 NPM 模块

+   procfile：**Heroku**部署的配置

在接下来的行中，我们将深入了解每个部分的功能。

### 提示

路由文件夹中有一些文件，我们现在不会解释，但不用担心；我们将在下一个主题中看到这些文件。

# 更改默认的 bootstrap 主题

我们将展示两种自定义博客的方法：一种是表面的，只改变样式表，另一种是更深入的，改变整个页面的标记。

对于样式表的更改，我们正在使用[`bootswatch.com/`](http://bootswatch.com/)免费的 Bootstrap 主题。

bootstrap 框架非常灵活；我们将使用一个名为 superhero 的主题。

1.  转到[`bootswatch.com/superhero/_variables.scss`](http://bootswatch.com/superhero/_variables.scss) URL。

1.  复制页面内容。

1.  在 public/styles/boostrap/bootstrap 中，创建一个名为 _theme_variables.scss 的新文件，并粘贴从 Bootswatch 页面复制的代码。

1.  打开 public/styles/bootstrap/_bootstrap.scss 并替换以下行：

```js
      // Core variables and mixins 
      @import "bootstrap/_theme_variables"; 
      @import "bootstrap/mixins";

```

现在我们将重复*步骤 1*和*2*，但现在使用不同的 URL。

1.  转到[`bootswatch.com/superhero/_bootswatch.scss`](http://bootswatch.com/superhero/_bootswatch.scss) URL。

1.  复制页面内容。

1.  在 public/styles/bootstrap 中创建一个名为 _bootswatch.scss 的文件，并粘贴内容。

1.  打开 public/styles/bootstrap/_bootstrap.scss 并替换以下突出显示的行：

```js
      // Bootswatch overhide classes
      @import "bootswatch";

```

1.  完成。现在我们有了一个与 keystone.js 采用的标准布局不同的布局，让我们看看结果。打开您的终端/Shell 并输入以下命令：

```js
 gulp 

```

1.  转到 URL：http://localhost:3000/，您应该会看到以下结果：![更改默认的 bootstrap 主题](img/image_08_006.jpg)

Keystone 主屏幕

通过这个小改变，我们已经可以看到所取得的结果。然而，这是一个非常表面的定制，因为我们没有改变任何 HTML 标记文件。

在之前的图片中，我们可以看到我们只是改变了页面的颜色，因为它保持了标记不变，只使用了一个 bootstrap 主题。

在下一个示例中，我们将看到如何修改应用程序的整个结构。

# 修改 KeystoneJS 核心模板路径

现在让我们对模板目录进行一些重构。

1.  在模板中，创建一个名为 default 的文件夹。

1.  将模板文件夹中的所有文件移动到新的 default 文件夹中。

1.  复制默认文件夹中的所有内容，并将它们粘贴到一个名为 newBlog 的新文件夹中。

结果将是以下截图，但我们需要更改 keystone.js 文件以配置新文件夹：

![修改 KeystoneJS 核心模板路径](img/image_08_007.jpg)

模板文件夹结构

1.  从根文件夹打开 keystone.js 文件并更新以下行：

```js
      'views': 'templates/themes/newBlog/views', 
      'emails': 'templates/themes/newBlog/emails', 

```

完成。我们已经创建了一个文件夹来保存所有我们的主题。

## 构建我们自己的主题

现在我们将更改主题标记。这意味着我们将编辑 newBlog 主题内的所有 HTML 文件。我们使用[`github.com/BlackrockDigital/startbootstrap-clean-blog`](https://github.com/BlackrockDigital/startbootstrap-clean-blog)提供的免费模板作为参考和来源。我们的目标是拥有类似以下截图的布局：

![构建我们自己的主题](img/image_08_008.jpg)

Keystone 主屏幕

1.  打开模板/主题/newBlog/layouts/default.swig 并将以下代码添加到<head>标记中：

```js
      {# Custom Fonts #} 
      <link href="http://maxcdn.bootstrapcdn.com/font-awesome/4.1.0
       /css/font-awesome.min.css" rel="stylesheet" type="text/css"> 
      <link href='http://fonts.googleapis.com
       /css?family=Lora:400,700,400italic,700italic'
       rel='stylesheet' type='text/css'> 
      <link href='http://fonts.googleapis.com      
       /css?family=Open+Sans:300italic,400italic,600italic,
       700italic,800italic,400,300,600,700,800' rel='stylesheet'
       type='text/css'> 

```

1.  删除{# HEADER #}和{# JAVASCRIPT #}注释之间的所有行。

### 提示

请注意，此操作将删除 default.swig 文件底部的 body 标记后的所有内容和 JavaScript 链接。

1.  现在将以下代码行放在{# HEADER #}和{# JAVASCRIPT #}注释之间：

```js
      <div id="header"> 
      {# Customise your sites navigation by changing the 
       navLinks Array in ./routes/middleware.js 
        ... or completely change this header to suit your design. #} 

      <!-- Navigation --> 
      <nav class="navbar navbar-default navbar-custom
       navbar-fixed-top"> 
        <div class="container-fluid"> 
          <!-- Brand and toggle get grouped for better mobile
           display --> 
          <div class="navbar-header page-scroll"> 
            <button type="button" class="navbar-toggle"
             data-toggle="collapse" data-target="#bs-example-navbar-
             collapse-1"> 
              <span class="sr-only">Toggle navigation</span> 
              <span class="icon-bar"></span> 
              <span class="icon-bar"></span> 
              <span class="icon-bar"></span> 
            </button> 
            <a class="navbar-brand" href="/">newBlog</a> 
          </div> 
          <!-- Collect the nav links, forms, and other content
            for toggling --> 
          <div class="collapse navbar-collapse" id="bs-example
           -navbar-collapse-1"> 
            <ul class="nav navbar-nav navbar-left"> 
              {%- for link in navLinks -%} 
                {%- set linkClass = '' -%} 
                {%- if link.key == section -%} 
                  {%- set linkClass = ' class="active"' -%} 
              {%- endif %} 
              <li{{ linkClass | safe }}> 
                <a href="{{ link.href }}">{{ link.label }}</a> 
              </li> 
              {%- endfor %} 
              </ul> 
                <ul class="nav navbar-nav navbar-right"> 
                  {% if user -%} 
                    {%- if user.canAccessKeystone -%} 
                      <li><a href="/keystone">Open Keystone</a>
                      </li> 
                    {%- endif -%} 
                      <li><a href="/keystone/signout">Sign Out</a>
                      </li> 
                    {%- else -%} 
                      <li><a href="/keystone/signin">Sign In</a>
                      </li> 
                    {%- endif %} 
                </ul> 
          </div> 
          <!-- /.navbar-collapse --> 
          </div> 
          <!-- /.container --> 
        </nav> 
        <!-- Page Header --> 
        <header class="intro-header"> 
        <div class="container"> 
          <div class="row"> 
            <div class="col-lg-8 col-lg-offset-2 col-md-10 col-
             md-offset-1"> 
            <div class="site-heading"> 
              <h1>Node.js 6 Blueprints</h1> 
              <hr class="small"> 
              <span class="subheading">A Clean Blog using 
               KeystoneJS</span> 
            </div> 
          </div> 
        </div> 
      </div> 
      </header> 
      </div> 

      {# BODY #} 
      <div id="body"> 
      {# NOTE: There is no .container wrapping class around body
        blocks to allow more flexibility in design. 
      Remember to include it in your templates when you override
        the intro and content blocks! #} 

      {# The Intro block appears above flash messages (used for
       temporary information display) #} 
      {%- block intro -%}{%- endblock -%} 

      {# Flash messages allow you to display once-off status messages
       to users, e.g. form 
      validation errors, success messages, etc. #} 
      {{ FlashMessages.renderMessages(messages) }} 

      {# The content block should contain the body of your templates
       content #} 
      {%- block content -%}{%- endblock -%} 
      </div> 

```

1.  打开模板/主题/newBlog/views/blog.swig 并用以下代码替换代码：

```js
      {% extends "../layouts/default.swig" %} 

      {% macro showPost(post) %} 
      <div class="post" data-ks-editable="editable(user, { list:
        'Post', id: post.id })"> 
        <div class="post-preview"> 
          {% if post.image.exists %} 
            <img src="img/{{ post._.image.fit(400,300) }}" class="img
              text-center" width="100%" height="260px"> 
          {% endif %} 
          <a href="/blog/post/{{ post.slug }}"> 
            <h2 class="post-title"> 
              {{ post.title }} 
            </h2> 
            <h3 class="post-subtitle"> 
              {{ post.content.brief | safe }} 
            </h3> 
          </a> 
          <p class="post-meta">Posted by <a href="#"> 
            {% if post.author %} {{ post.author.name.first }}
            {% endif %} 
          </a>
            {% if post.publishedDate %} 
          on
            {{ post._.publishedDate.format("MMMM Do, YYYY") }} 
            {% endif %} 
            {% if post.categories and post.categories.length %} 
          in 
          {% for cat in post.categories %} 
          <a href="/blog/{{ cat.key }}">{{ cat.name }}</a> 
            {% if loop.index < post.categories.length - 1 %},
            {% endif %} 
          {% endfor %} 
          {% endif %} 
        </p> 
        {% if post.content.extended %} 
        <a class="read-more" href="/blog/post/{{ post.slug }}">
          Read more...</a> 
        {% endif %} 
      </div> 
      <hr> 
      </div> 
      {% endmacro %} 

      {% block intro %} 
        <div class="container"> 
        {% set title = "Blog" %} 
          {% if data.category %} 
            {% set title = data.category.name %} 
          {% endif %} 
          <h1>{{ title }}</h1> 
        </div> 
      {% endblock %} 

      {% block content %} 
      <div class="container"> 
        <div class="row"> 
          <div class="col-sm-8 col-md-9"> 
            {% if filters.category and not data.category %} 
              <h3 class="text-muted">Invalid Category.</h3> 
            {% else %} 
            {% if data.posts.results.length %} 
              {% if data.posts.totalPages > 1 %} 
                <h4 class="text-weight-normal">Showing 
                  <strong>{{ data.posts.first }}</strong> 
                  to 
                  <strong>{{ data.posts.last }}</strong> 
                  of 
                  <strong>{{ data.posts.total }}</strong> 
                  posts. 
                </h4> 
             {% else %} 
            <h4 class="text-weight-normal">Showing 
              {{ utils.plural(data.posts.results.length, "*
               post") }}
            </h4> 
          {% endif %} 
          <div class="blog"> 
            {% for post in data.posts.results %} 
              {{ showPost(post) }}
            {% endfor %} 
          </div> 
          {% if data.posts.totalPages > 1 %} 
          <ul class="pagination"> 
            {% if data.posts.previous %} 
            <li> 
              <a href="?page={{ data.posts.previous }}"> 
                <span class="glyphicon glyphicon-chevron-left">
                </span> 
              </a> 
            </li> 
            {% else %} 
            <li class="disabled"> 
              <a href="?page=1"> 
                <span class="glyphicon glyphicon-chevron-left">
                </span> 
              </a> 
            </li> 
            {% endif %} 
            {% for p in data.posts.pages %} 
              <li class="{% if data.posts.currentPage == p %}
                active{% endif %}"> 
              <a href="?page={% if p == "..." %}{% if i %}
                {{data.posts.totalPages }}{% else %}1{% endif %}
                {% else %}{{ p }}{% endif %}">{{ p }}
              </a> 
              </li> 
            {% endfor %} 
            {% if data.posts.next %} 
            <li> 
              <a href="?page={{ data.posts.next }}"> 
                <span class="glyphicon glyphicon-chevron-right">
                </span> 
              </a> 
            </li> 
            {% else %} 
            <li class="disabled"> 
              <a href="?page={{ data.posts.totalPages }}"> 
                <span class="glyphicon glyphicon-chevron-right">
                </span> 
              </a> 
            </li> 
            {% endif %} 
          </ul> 
          {% endif %} 
          {% else %} 
            {% if data.category %} 
              <h3 class="text-muted">There are no posts in the
                category {{ data.category.name }}.
              </h3> 
            {% else %} 
              <h3 class="text-muted">There are no posts yet.</h3> 
            {% endif %} 
          {% endif %} 
          {% endif %} 
        </div> 
        {% if data.categories.length %} 
          <div class="col-sm-4 col-md-3"> 
            <h2>Categories</h2> 
              <div class="list-group" style="margin-top: 70px;"> 
                <a href="/blog" class="{% if not data.category %}
                  active{% endif %} list-group-item">All Categories
                </a> 
               {% for cat in data.categories %} 
               <a href="/blog/{{ cat.key }}" class="{% if
                 data.category and data.category.id == cat.id %}
                 active{% endif %} list-group-item">{{ cat.name }}
               </a> 
               {% endfor %} 
              </div> 
          </div> 
        {% endif %} 
      </div> 
      </div> 
      {% endblock %} 

```

1.  打开模板/主题/newBlog/views/contact.swig 并用以下代码替换代码：

```js
      {% extends "../layouts/default.swig" %} 

      {% block intro %} 
        <div class="container"> 
          <h1>Contact Us</h1> 
        </div> 
      {% endblock %} 

      {% block content %} 
        <div class="container"> 
         {% if enquirySubmitted %} 
           <h3>Thanks for getting in touch.</h3> 
        {% else %} 
          <div class="row control-group"> 
            <div class="col-lg-8 col-lg-offset-2 col-md-10 col-md-
              offset-1"> 
              <form method="post"> 
                <input type="hidden" name="action" value="contact"> 
                  {% set className = "" %} 
                  {% if validationErrors.name %} 
                    {% set className = "has-error" %} 
                  {% endif %} 
                <div class="form-group {{ className }} col-xs-12
                  floating-label-form-group controls"> 
                  <label>Name</label> 
                  <input type="text" name="name.full" value="{{
                   formData['name.full'] | default('') }}" class=
                   "form-control" placeholder="Name"> 
                </div> 
                {% set className = "" %} 
                {% if validationErrors.email %} 
                  {% set className = "has-error" %} 
                {% endif %} 
                <div class="form-group {{ className }} col-xs-12
                  floating-label-form-group controls"> 
                  <label>Email</label> 
                  <input type="email" name="email" value="{{ 
                  formData.email | default('') }}" class=
                  "form-control" placeholder="E-mail"> 
                </div> 
                  <div class="form-group col-xs-12 floating-label-
                    form-group controls"> 
                    <label>Phone</label> 
                    <input type="text" name="phone" value="{{ 
                      formData.phone | default('') }}" placeholder=
                      "Phone Number (Optional)" class="form-control"> 
                  </div> 
                  {% set className = "" %} 
                  {% if validationErrors.enquiryType %} 
                    {% set className = "has-error" %} 
                  {% endif %} 
                  <div class="form-group {{ className }} col-xs-12
                   floating-label-form-group controls"> 
                    <span class="title-label text-muted">
                     What are you contacting us about?
                    </span> 
                    <br> 
                    <select name="enquiryType" class="form-control"> 
                      <option value="">(select one)</option> 
                      {% for type in enquiryTypes %} 
                        {% set selected = "" %} 
                        {% if formData.enquiryType === type.value %} 
                          {% set selected = " selected" %} 
                        {% endif %} 
                      <option value="{{ type.value }}"{{ selected }}>
                        {{ type.label }}</option> 
                      {% endfor %} 
                    </select> 
                  </div> 
                  {% set className = "" %} 
                  {% if validationErrors.message %} 
                    {% set className = "has-error" %} 
                  {% endif %} 
                  <div class="form-group {{ className }} col-xs-12
                    floating-label-form-group controls"> 
                     <label>Message</label> 
                     <textarea rows="5" class="form-control"
                       placeholder="Message" name="message">
                     </textarea>
                     {{ formData.message }} 
                   </div> 
                   <br> 
                   <div class="row"> 
                     <div class="form-group col-xs-12"> 
                       <button type="submit" class="btn
                         btn-default">Send</button> 
                     </div> 
                   </div> 
                 </form> 
               </div> 
             </div> 
          {% endif %} 
          </div> 
      {% endblock %} 

```

1.  打开模板/主题/newBlog/views/gallery.swig 并用以下代码替换代码：

```js
      {% extends "../layouts/default.swig" %} 

      {% block intro %} 
      <div class="container"> 
        <h1>Gallery</h1> 
      </div> 
      {% endblock %} 

      {% block content %} 
        <div class="container"> 
        {% if galleries.length %} 
          {% for gallery in galleries %} 
            <h2>{{ gallery.name }} 
            {% if gallery.publishedDate %} 
              <span class="pull-right text-muted">{{ 
                gallery._.publishedDate.format("Do MMM YYYY") }}
              </span> 
        {% endif %} 
            </h2> 
            <div class="row"> 
            {% if gallery.heroImage.exists %} 
              <div class="gallery-image"> 
                <img src="img/{{ gallery._.heroImage.limit(0.73,200) }}">
              </div> 
              <br> 
              <hr> 
                <div class="row"> 
                  <div class='list-group gallery'> 
                  {% for image in gallery.images %} 
                  <div class='col-sm-6 col-xs-6 col-md-4 col-lg-4'> 
                    <a class="thumbnail fancybox" rel="ligthbox"
                      href="{{ image.limit(640,480) }}"> 
                    <img class="img-responsive" alt="" src="{{ 
                      image.limit(300,320) }}" /> 
                    </a> 
                  </div> 
                  {% endfor %} 
                </div> 
              </div> 
            {% else %} 
            <div class="row"> 
              <div class='list-group gallery'> 
                {% for image in gallery.images %} 
                <div class='col-sm-6 col-xs-6 col-md-4 col-lg-4'> 
                  <a class="thumbnail fancybox" rel="ligthbox"
                    href="{{ image.limit(640,480) }}"> 
                  <img class="img-responsive" alt="" src="{{ 
                    image.limit(300,320) }}" /> 
                  </a> 
                </div> 
              {% endfor %} 
            </div> 
          </div> 
        {% endif %} 
      </div> 
      {% endfor %} 
      {% else %} 
        <h3 class="text-muted">There are no image galleries yet.</h3> 
      {% endif %} 
      </div> 
      {% endblock %} 

```

1.  打开模板/主题/newBlog/views/index.swig 并用以下代码替换代码：

```js
      {% extends "../layouts/default.swig" %} 

      {% block content %} 
        <div class="container"> 
          <div class="row"> 
            <div class="col-lg-8 col-lg-offset-2 col-md-10 col-md-
              offset-1"> 
            {% for post in data.posts %} 
            <div class="post-preview"> 
              <a href="/blog/post/{{ post.slug }}"> 
              <h2 class="post-title"> 
                {{ post.title }} 
              </h2> 
              <h3 class="post-subtitle"> 
                {{ post.content.brief | safe }} 
              </h3> 
              </a> 
              <p class="post-meta">Posted by <span class=
                 "text-primary"> 
                {% if post.author %} {{ post.author.name.first }}
                {% endif %} 
                </span> {% if post.publishedDate %} 
                  on 
                {{ post._.publishedDate.format("MMMM Do, YYYY") }} 
                {% endif %}</p> 
              </div> 
              <hr> 
            {% endfor %} 
            <!-- Pager --> 
            {% if data.posts %} 
            <ul class="pager"> 
              <li class="next"> 
                <a href="/blog">Older Posts &rarr;</a> 
              </li> 
            </ul> 
            {% endif %} 
          </div> 
        </div> 
      </div> 

      {% endblock %} 

```

请注意，在 index.swig 中，我们添加了一些代码行以在索引页面上显示帖子列表，因此我们需要更改 index.js 控制器。

1.  打开 routes/views/index.js 并添加以下代码行：

```js
      var keystone = require('keystone'); 

      exports = module.exports = function (req, res) { 

        var view = new keystone.View(req, res); 
        var locals = res.locals; 

          // locals.section is used to set the currently selected 
          // item in the header navigation. 
          locals.section = 'home'; 

          // Add code to show posts on index 
          locals.data = { 
            posts: [] 
          }; 
          view.on('init', function(next) { 
            var q = keystone.list('Post').model.find() 
            .where('state', 'published') 
            .sort('-publishedDate') 
            .populate('author') 
            .limit('4'); 

          q.exec(function(err, results) { 
            locals.data.posts = results; 
            next(err); 
          }); 
        }); 

        // Render the view 
        view.render('index'); 
      };

```

1.  打开模板/主题/newBlog/views/post.swig 并用以下代码替换代码：

```js
      {% extends "../layouts/default.swig" %} 

      {% block content %} 
      <article> 
        <div class="container"> 
          <a href="/blog">&larr; back to the blog</a> 
          <div class="row"> 
            <div class="col-lg-8 col-lg-offset-2 col-md-10 col-md-
              offset-1"> 
              {% if not data.post %} 
                <h2>Invalid Post.</h2> 
              {% else %} 
                <h1>{{ data.post.title }}</h1> 
                  {% if data.post.publishedDate %} 
                  on 
              {{ data.post._.publishedDate.format("MMMM Do, YYYY") }} 
              {% endif %} 
              {% if data.post.categories and
                data.post.categories.length %} 
              in 
              {% for cat in data.post.categories %} 
                <a href="/blog/{{ cat.key }}">{{ cat.name }}</a> 
              {% if loop.index < data.post.categories.length - 1 %},
              {% endif %}
            {% endfor %} 
          {% endif %} 
          {% if data.post.author %} 
            by {{ data.post.author.name.first }} 
          {% endif %} 
          <div class="post"> 
            {% if data.post.image.exists %} 
              <div class="image-wrap"> 
                <img src="img/{{ data.post._.image.fit(750,450) }}"
                  class="img-responsive"> 
               </div> 
             {% endif %} 
             {{ data.post.content.full | raw }} 
               </div> 
             {% endif %} 
           </div> 
         </div> 
       </div> 
       </article> 
       <hr> 
       {% endblock %} 

```

通过这一段代码，我们已经完成了 HTML 标记的更改。现在我们需要应用新的样式表。

## 更改样式表

由于我们选择了 SASS 来处理 keystone.js 设置中的样式表，我们已经拥有了使用**SASS**功能的一切。

打开 public/styles/site/_variables.scss 并替换以下代码行：

```js
    // Override Bootstrap variables in this file, e.g.
     $font-size-base: 14px;
    // Theme Variables
    $brand-primary: #0085A1;
    $gray-dark: lighten(black, 25%);
    $gray: lighten(black, 50%);
    $white-faded: fade(white, 80%);
    $gray-light: #eee;

```

请记住，我们使用 http://blackrockdigital.github.io/startbootstrap-clean-blog/index.html 作为参考，我们只挑选了一些代码块。请注意，模板使用的是 LESS 而不是**SASS**，但在这里我们重新编写所有代码以适应 SASS 语法。

由于空间原因，我们没有在此示例中放置整个样式表。您可以从 Packt Publishing 网站([www.packtpub.com](http://www.packtpub.com))或直接从 GitHub 书库下载示例代码。

重要的是要注意，我们为示例博客创建了相同的样式表，但我们将**LESS**语法转换为**SASS**。

1.  打开 public/styles/site/_layout.scss 并使用代码。

1.  在 public/styles/site/中创建一个名为 _mixins.scss 的新文件，并添加以下代码行：

```js
      // Mixins 
      @mixin transition-all() { 
        -webkit-transition: all 0.5s; 
        -moz-transition: all 0.5s; 
        transition: all 0.5s; 
      } 
      @mixin background-cover() { 
        -webkit-background-size: cover; 
        -moz-background-size: cover; 
        background-size: cover; 
        -o-background-size: cover; 
      } 
      @mixin serif() { 
        font-family: 'Lora', 'Times New Roman', serif; 
      } 
      @mixin sans-serif () { 
        font-family: 'Open Sans', 'Helvetica Neue', Helvetica, Arial,
          sans-serif; 
      } 

```

现在我们只需要编辑 public/styles/site.scss 以包含新的 mixin 文件。

1.  打开 public/styles/site.scss 并添加以下代码行：

```js
      // Bootstrap 
      // Bootstrap can be removed entirely by deleting this line. 
      @import "bootstrap/bootstrap"; 
      // The easiest way to customise Bootstrap variables while 
      // being able to easily override the source files with new 
      // versions is to override the ones you want in another file. 
      // 
       // You can also add your own custom variables to this file for 
         // use in your site stylesheets. 
      @import "site/variables"; 
      // Add mixins 
      @import "site/mixins"; 
      // Site Styles 
      // =========== 
      // Add your own site style includes here 
      @import "site/layout"; 

```

1.  将样本图像 header-bg-1290x1140.jpg 从 sample-images 文件夹添加到 public/images/文件夹中（您可以从 Packt Publishing 网站或 GitHub 官方书页下载所有示例文件）。

# 添加画廊脚本

正如我们所看到的，默认的 Keystone.js 主题非常简单，只使用了 Bootstrap 框架。现在我们将使用一个名为 Fancybox 的 jQuery 插件来应用新的样式在我们的画廊中。

### 提示

您可以在官方网站[`fancybox.net/`](http://fancybox.net/)上获取有关**Fancybox**的更多信息。

1.  打开模板/主题/newBlog/layouts/default.swig 并在<head>标记内添加以下突出显示的代码：

```js
      {# Customise the stylesheet for your site by editing
       /public/styles/site.sass #} 
      <link href="/styles/site.css" rel="stylesheet"> 
      <!-- fancyBox --> 
      <link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs
 /fancybox/2.1.5/jquery.fancybox.min.css" media="screen"> 
      {# This file provides the default styling for the KeystoneJS 
        Content Editor #} 
      {%- if user and user.canAccessKeystone -%} 
        <link href="/keystone/styles/content/editor.min.css" 
          rel="stylesheet"> 
      {%- endif -%}

```

1.  现在让我们将以下代码行添加到模板/主题/newBlog/layouts/default.swig 底部的脚本中：

```js
      {# Add scripts that are globally required by your site here. #} 
      <script src="//cdnjs.cloudflare.com/ajax/libs/fancybox/2.1.5
        /jquery.fancybox.min.js"></script> 
      <script> 
      $(document).ready(function(){ 
        // Gallery 
      $(".fancybox").fancybox({ 
          openEffect: "elastic", 
          closeEffect: "elastic" 
      }); 
      // Floating label headings for the contact form 
        $("body").on("input propertychange", ".floating-label-
          form-group", function(e) { 
          $(this).toggleClass("floating-label-form-group-with-value",
            !!$(e.target).val()); 
          }).on("focus", ".floating-label-form-group", function() { 
            $(this).addClass("floating-label-form-group-with-focus"); 
            }).on("blur", ".floating-label-form-group", function() { 
              $(this).removeClass("floating-label-form-group-
                with-focus"); 
              }); 
      }); 
      </script> 

      {# Include template-specific javascript files by extending 
        the js block #} 
     {%- block js -%}{%- endblock -%}

```

由于我们已经在项目中使用了 jQuery，因为 Bootstrap 依赖于它，所以我们不需要再次插入它。

1.  打开您的终端/Shell 并输入以下命令：

```js
gulp 

```

1.  转到 http://localhost:3000/gallery，您可以看到以下结果：![添加 Gallery 脚本](img/image_08_009.jpg)

模板图库

请注意，我们已经将示例内容包含到我们的博客中，但不用担心；在本章的后面，我们将看到如何包含内容。

# 扩展 keystone.js 核心

现在我们几乎准备好了新主题。

我们现在将看到如何扩展核心 keystone.js 并在我们的博客上添加另一页，如上一个截图所示，我们有一个**关于**菜单项，所以让我们创建它：

1.  在 models/folder 中创建一个名为 About.js 的新文件，并添加以下代码行：

```js
      var keystone = require('keystone'); 
      var Types = keystone.Field.Types; 

      /** 
       * About Model 
       * ========== 
      */ 

      var About = new keystone.List('About', { 
        // Using map to show title instead ObjectID on Admin Interface 
        map: { name: 'title' }, 
        autokey: { path: 'slug', from: 'title', unique: true }, 
      }); 

      About.add({ 
        title: { type: String, initial: true, default: '',
          required: true }, description: { type: Types.Textarea } 
      }); 

      About.register();

```

1.  将新模块添加到管理导航中，打开根文件夹中的 keystone.js，并添加以下突出显示的代码行：

```js
      // Configure the navigation bar in Keystone's Admin UI
      keystone.set('nav', { 
        posts: ['posts', 'post-categories'], 
        galleries: 'galleries', 
        enquiries: 'enquiries', 
        userAdmins: 'user-admins', 
        abouts: 'abouts' 
      });

```

请注意，左侧的单词将显示在导航栏上作为关于菜单项，右侧的单词是 about.js 集合。

1.  让我们自定义列显示。在 About.js 文件的 register()函数之前添加以下代码行：

```js
 About.defaultColumns = 'title, description|60%'; 

```

1.  要将路由添加到关于页面，打开 routes/index.js 并添加以下突出显示的代码行：

```js
      // Setup Route Bindings 
      exports = module.exports = function (app) { 
         // Views 
         app.get('/', routes.views.index); 
         app.get('/about', routes.views.about); 
         app.get('/blog/:category?', routes.views.blog); 
         app.get('/blog/post/:post', routes.views.post); 
         app.get('/gallery', routes.views.gallery); 
         app.all('/contact', routes.views.contact); 

        // NOTE: To protect a route so that only admins can see it,
        use the requireUser middleware: 
        // app.get('/protected', middleware.requireUser, 
        routes.views.protected); 
      }; 

```

现在让我们为 routes.views.blog 函数创建控制器。

1.  在 routes/views/文件夹中创建一个名为 about.js 的新文件，并添加以下代码：

```js
      var keystone = require('keystone'); 
      exports = module.exports = function (req, res) { 
        var view = new keystone.View(req, res); 
        var locals = res.locals; 

         // locals.section is used to set the currently selected 
         // item in the header navigation. 
         locals.section = 'about'; 
         // Add code to show posts on index 
         locals.data = { 
           abouts: [] 
         }; 
         view.on('init', function(next) { 
           var q = keystone.list('About').model.find() 
               .limit('1'); 
             q.exec(function(err, results) { 
               locals.data.abouts = results; 
                 next(err); 
           }); 
         }); 
         // Render the view 
         view.render('about'); 
         };

```

1.  在 routes/middleware.js 上添加路由，如下突出显示的代码：

```js
      exports.initLocals = function (req, res, next) { 
        res.locals.navLinks = [ 
          { label: 'Home', key: 'home', href: '/' }, 
          { label: 'About', key: 'about', href: '/about' }, 
          { label: 'Blog', key: 'blog', href: '/blog' }, 
          { label: 'Gallery', key: 'gallery', href: '/gallery' }, 
          { label: 'Contact', key: 'contact', href: '/contact' }, 
        ]; 
        res.locals.user = req.user; 
          next(); 
      }; 

```

在这个例子中，我们看到如何通过使用内置函数来扩展框架的功能。

### 提示

您可以在此链接中阅读有关**Keystone API**的更多信息：[`github.com/keystonejs/keystone/wiki/Keystone-API`](https://github.com/keystonejs/keystone/wiki/Keystone-API)。

因此，所有这些步骤的最终结果将如下截图所示：

![扩展 keystone.js 核心](img/image_08_010.jpg)

带有关于菜单项的 Keystone 控制面板

请注意，我们可以在上一个截图中看到**关于**菜单。

# 使用控制面板插入内容

经过所有这些步骤，我们成功为我们的博客创建了一个完全定制的布局；现在我们将使用书籍源代码下载中的 sample-images 文件夹中的可用图像输入内容：

1.  转到 http://localhost:3000/keystone，使用用户：john@doe.com 和密码：123456 访问控制面板。

1.  转到 http://localhost:3000/keystone/post-categories，单击**帖子类别**链接。

1.  单击**创建帖子类别**按钮，将**旧车**标题插入输入字段，并单击**创建**按钮。

1.  对于书籍示例，我们将只使用一个类别，但在实际应用中，您可以创建任意多个。

1.  转到 http://localhost:3000/keystone/posts，单击**创建帖子**按钮，并按照以下截图中显示的内容添加内容：![使用控制面板插入内容](img/image_08_011.jpg)

创建帖子屏幕上的示例内容

1.  对于第二个帖子条目，重复*步骤 4*的相同过程，并将标题更改为**不带图像的示例帖子示例 II**。

1.  对于第三个帖子条目，重复*步骤 4*的相同过程，并将标题更改为**带图像的示例帖子示例**，单击**上传图像**按钮，并使用 sample-images 文件夹中的文件 sample-blog-image.png。

### 提示

请注意，您可以随时从 Packt Publishing 网站或直接从 GitHub 书库下载书籍源代码和图像样本。

在*步骤 6*结束时，我们的控制面板将如下截图所示：

![使用控制面板插入内容](img/image_08_012.jpg)

帖子控制面板

正如我们所看到的，Keystone.js 具有非常简单和易于使用的界面。我们可以扩展框架的所有功能，以创建令人难以置信的东西。

我们的帖子页面如下：

![使用控制面板插入内容](img/image_08_013.jpg)

博客页面截图

# 总结

在本章中，我们讨论了关于 Keystone 框架的一些非常重要的概念，以便使用数据库创建应用程序和网站。

我们看到了如何通过使用内部 Keystone API 来创建新的模型、视图和模板来扩展框架。

此外，我们展示了使用样式表来自定义 CMS 的两种不同方式，以及如何完全改变页面结构以及如何插入新功能，比如**Fancybox**插件到图片库中。

在下一章中，我们将看到如何使用命令行界面（CLI）来进行 JSLint、Concat、Minify 和其他任务，只使用 Node Package Manager（NPM）来构建和部署应用程序。
