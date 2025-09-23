# 附录 C. 使用 AMD 和 Require.js 组织模板

**异步模块定义**（**AMD**）是一个用于定义模块和异步加载模块依赖的 JavaScript API。这是一个相对较新但非常稳健的概念，许多开发者现在都在采用它。在第七章 *组织 Backbone 应用程序 – 结构、优化和部署* 中，我们详细介绍了使用 Require.js 的 AMD。如果您需要更多关于这个库的细节，我们建议您访问[`requirejs.org/`](http://requirejs.org/)以获取完整的概述，然后回到这一节。

通常，Require.js 将每个文件的内容视为 JavaScript。因此，如果我们不将模板文件作为 JavaScript 文件加载，我们就不能以相同的方式加载它们。幸运的是，对于模板，有一个`text`插件允许我们加载基于文本的依赖项。我们使用此文件加载的任何文件都将被视为文本文件，我们接收到的内容将是一个字符串；它可以很容易地与您的模板方法一起使用。要使用此插件，只需在文件路径前加上`text!`，文件内容将以纯文本形式检索；为此，请遵循以下示例：

```js
// AMD Module definition with dependencies
define([
    'backbone',
    'underscore',

    // text plugin that gets any file content as text
    'text!../templates/userLogin.html'
  ],
  function (Backbone, _, userLoginTpl) {
    'use strict';

    var UserLogin = Backbone.View.extend({
      // Compile the template string that we received 
      template: _.template(userLoginTpl),
      render: function () {
        this.$el.html(this.template({
          username: 'johndoe',
          password: 'john'
        }));

        return this;
      }
    });

    return UserLogin;
  });
```

使用此机制的好处是，您可以通过创建单独的模板文件来组织您的模板，并且它们将自动包含在您的模块中。由于这涉及到异步加载，文件将通过 AJAX 请求下载，这是我们之前决定为不好的做法。然而，Require.js 附带了一个`r.js`优化工具，它可以构建模块并可以通过内联这些模板与相应的模块来节省这些额外的 AJAX 请求。

# 使用 requirejs-tpl 插件进行预编译

使用 AMD，我们简化了模板组织过程，但最终结果仍然是一个未编译的模板字符串。在第二章 *与视图一起工作* 中，我们看到了模板编译如何每次都影响应用程序性能，我们也分析了预编译模板的好处。如果我们有一种可以加载这些模板文件并提供已编译的模板字符串的方法，那会很有用吗？幸运的是，对于 Require.js 有多个`tpl`插件可用于自动化模板编译，您可以直接在模块定义中使用这些插件。让我们看看由 ZeeAgency 开发的类似插件([`github.com/ZeeAgency/requirejs-tpl`](https://github.com/ZeeAgency/requirejs-tpl))。依赖项加载与`text`插件完全相同，您只需使用`tpl!`插件前缀而不是`text!`：

```js
 define(['tpl!your-template-path.tpl'], function (template) { 
  return template ({
    your: 'data'
  });
});
```

现在，`r.js`提供了优化和打包的预编译模板。`tpl!`插件肯定比`text!`插件更方便和有用。

使用 Require.js 进行模板组织是维护模板的最佳方式之一；现在很多 JavaScript 开发者都在选择这种方式。如果你正在使用 AMD 为你的 Backbone 应用程序开发，那就毫不犹豫地选择它吧。
