# 附录 A. 向 Rails 应用程序添加自定义样式

最后，我们到达了这本书的最后一章，我们将讨论如何向由 Bootstrap 框架支持的 Rails 应用程序添加自定义样式。在整个书中，我们探讨了各种 Bootstrap CSS 和 JavaScript 组件，它们可以直接使用。

在这一章中，我们将看到如何扩展 Bootstrap 框架并添加我们自己的样式。Bootstrap 框架中仍然缺少许多重要组件。我们将检查一些免费提供的流行 Bootstrap 插件。

我们将要涵盖的主题包括：

+   将 Bootstrap-sass 添加到 Rails 应用程序

+   通过变量自定义 Bootstrap

# 将 Bootstrap-sass 添加到 Rails 应用程序

在第二章的*在 Rails 项目中安装 Bootstrap*部分中，*介绍 Bootstrap 3*，我们看到了如何通过三种不同的方式将 Bootstrap 包含到我们的 Rails 应用程序中：

+   CDN 方法

+   Bootstrap-sass gem

+   通过下载 Bootstrap 文件

为了加快速度，我们选择了 CDN 方法。嗯，在这一章中，我们将通过 Bootstrap-sass gem 使用 Bootstrap。这将使我们能够完全自定义 Bootstrap 的默认样式。所以，让我们继续，并在我们的应用程序中安装`Bootstrap-sass gem`：

1.  前往`application`文件夹，并使用文本编辑器编辑`Gemfile`文件。将以下两行代码添加到文件末尾：

    ```js
    gem 'bootstrap-sass', '~> 3.3.1'
    gem 'autoprefixer-rails'
    ```

1.  上述两行将安装`bootstrap-sass`和`autoprefixer-rails gems`到您的应用程序中。`autoprefixer-rails`是自动在 CSS `stylesheets`中添加浏览器供应商前缀所必需的。

1.  让我们捆绑应用程序，以便上述 gem 实际上被下载并安装到我们的应用程序中。

    ```js
    bundle install

    ```

1.  一旦上述命令执行完成，导航到`app` | `assets` | `stylesheets`文件夹。将`application.css`文件重命名为`application.css.scss`文件。接下来，从文件中移除我们之前包含的 CDN 链接。

1.  现在，我们需要在`application.css.scss`文件中包含通过 gem 下载的 Bootstrap 文件。为此，包含以下 2 行：

    ```js
    @import "bootstrap-sprockets";
    @import "bootstrap";
    ```

    `bootstrap-sprockets`值是正确链接字体文件与 Bootstrap 的 CSS 文件所必需的。

是时候使用最近安装的 gem 链接 Bootstrap 的 JavaScript 文件了：

1.  首先，我们需要从`application.html.erb`文件中移除硬编码的 Bootstrap 的 JavaScript CDN 链接，该文件位于`layouts`文件夹中，通过导航到`app` | `views` | `layouts`文件夹。从该文件中移除以下行：

    ```js
    <script src="img/bootstrap.min.js"></script>
    ```

1.  接下来，通过导航到`app` | `assets` | `javascript`文件夹，前往 JavaScript 文件夹并编辑`application.js`文件。在 jQuery 行后立即添加以下行：

    ```js
    //= require bootstrap-sprockets
    ```

1.  最后，我们完成了。如果您在浏览器中重新打开应用程序，您会看到一切正常，就像之前一样。

# 通过变量定制 Bootstrap

大多数可见的 Bootstrap 样式都可以通过使用预定义的 Bootstrap 变量简单地覆盖。在继续之前，你应该了解 Bootstrap 最初只与 LESS 兼容。他们后来将其移植到了 Sass 版本。

### 注意

LESS 和 Sass 是帮助我们组织和编写可扩展 CSS 样式的 CSS 预处理器。在语法上，它们非常相似，只是在附加功能上有所不同，一个有而另一个没有。

因此，LESS 版本中存在的所有变量在 Sass 版本中也保持不变。虽然 Bootstrap 没有提供专门列出 Sass 中变量列表的页面，但你可以在他们的官方网站上找到 LESS 版本中变量的列表（[`getbootstrap.com/customize/#less-variables`](http://getbootstrap.com/customize/#less-variables)）。让我们继续并更改一些默认的 Bootstrap 样式。

在我们的应用中，我们在多个地方使用了 `.btn-success`。因此，让我们更改其中的一些 CSS 样式。重新打开 `application.css.scss` 文件，在 Bootstrap 的导入行之前添加以下行：

```js
$btn-success-color: #333;
$btn-success-bg: #AEDBAE;
$btn-success-border: darken($btn-success-bg, 5%);
```

我们可以通过 `$btn-success-color`、`$btn-success-bg` 和 `$btn-success-border` 这些 Bootstrap Sass 变量完全更改 `.btn-success` 类的样式。在上面的代码中，我已经将按钮的文本颜色更改为 `#333`。我还将背景颜色变亮为新的十六进制颜色，并最终使用 Sass 中的深色颜色函数更改了边框颜色。

你可以通过查看所有可用的变量并根据需要进行定制。你还可以通过在 `application.css.scss` 文件中添加以下行来包含可用的 Bootstrap 主题：

```js
@import "bootstrap/theme";
```

`bootstrap/theme` 是从 Bootstrap 主题定制而来的官方 Bootstrap 默认样式。它包含了一些酷炫的样式，你应该尝试使用它。

# 摘要

Bootstrap 定制化可以帮助你创建外观不同的网站。如果你是 Rails 应用开发团队中的设计师，这是你必须掌握的领域。在本章中，我们看到了如何在 Rails 应用中包含 Bootstrap-sass 钩子。我们还了解了通过预定义变量覆盖默认 Bootstrap 样式的步骤。希望这对你有所帮助。

最后，如果你在使用 Bootstrap 的 Rails 应用中还有相关问题，请给我发条推文 `@fazlerocks`，我会很乐意帮助你！
