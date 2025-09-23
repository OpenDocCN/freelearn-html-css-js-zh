# 附录 B. 服务器端预编译模板

在第二章中，我们学习了在应用程序中使用预编译模板的优势。此外，我们还看到了将模板存储在`index.html`文件中作为内联或作为单独的模板文件的一些选项。我们还看到了如何使用模板管理器预编译和缓存模板以避免每次编译的开销。然而，这个预编译过程仍然会在你启动应用程序时运行，这肯定会占用一定的时间。等等！这些模板不是静态资源吗？那么没有数据的模板编译版本也将是静态资源。对吧？那么为什么不保留一个包含所有预编译模板的单独文件，并在应用程序启动时立即使用它呢？如果你得到一个包含所有模板已经预编译和压缩的文件，它肯定会提高你的应用程序性能。这正是我们在这里要尝试的——我们将开发一个脚本在服务器端预编译模板，它将遍历所有模板文件并创建一个单独的模板管理器文件。在这里我们使用 Node.js，但你也可以使用任何服务器端技术来得到相同的结果。完整的代码示例在代码样本中给出。

为了预编译，我们需要一个支持预编译的模板引擎。在这里我们将使用 Underscore.js，但你也可以自由选择你想要的模板引擎来实现相同的结果。以下 Node.js 示例展示了如何实现这个功能：

```js
// load the file system node module
var fs = require('fs'),
  // load the underscore.js
  _ = require('../../../lib/underscore.js');

var templateDir = './templates/',
  template,
  tplName,

  // create a string which when evaluated will create the 
  // template object with cachedTemplates
  compiledTemplateStr = 'var Templates = {cachedTemplates : {}}; \n\n';

// Iterate through all the templates in templates directory
fs.readdirSync(templateDir).forEach(function (tplFile) {

  // Read the template and store the string in a variable
  template = fs.readFileSync(templateDir + tplFile, 'utf8');

  // Get the template name
  tplName = tplFile.substr(0, tplFile.lastIndexOf('.'));

  // Add template function's source to cachedTemplate
  compiledTemplateStr += 'Templates.cachedTemplates["' + tplName + '"] = ';
  compiledTemplateStr += _.template(template).source + '\n\n';
});

// Write all the compiled code in another file
fs.writeFile('compiled.js', compiledTemplateStr, 'utf8');
```

上述代码相当直观。我们创建了一个完整的 JavaScript 代码片段作为字符串，并将其返回到前端。以下是完成此操作的步骤：

1.  首先，我们浏览`templates`目录中的每个模板文件并检索其内容。

1.  我们已经定义了一个对象`Templates.cachedTemplates`，我们需要将每个模板文件的 内容存储在这个对象中，以模板文件名作为属性，模板字符串作为其值。

1.  Underscore 的`_.template()`方法通常返回一个函数。它还提供了一个名为`source`的属性，它提供了该特定函数的文本表示。以下行将给出函数的源代码：

    ```js
    _.template(template).source
    ```

1.  我们将所有的函数字符串逐个放入`Templates.cachedTemplates`中，一旦循环结束，我们就将整个内容写入另一个 JavaScript 文件。

现在假设客户端请求包含项目完整模板内容的`templates.js`文件。在服务器端，我们可以编写以下代码，将`compiled.js`文件的内容发送到浏览器：

```js
// While templates.js file is loaded, it will 
// send the compiled.js file's content
app.get('/templates.js', function (req, res) {
  res
    .type('application/javascript')
    .send(fs.readFileSync('compiled.js', 'utf8'));
});
```

因此，客户端对`template.js`文件的请求将显示类似于以下代码的内容：

```js
var Templates = {
  cachedTemplates: {}
};

Templates.cachedTemplates["userLogin"] = function (obj) {
  var __t, __p = '',
    __j = Array.prototype.join,
    print = function () {
      __p += __j.call(arguments, '');
    };
  with(obj || {}) {
    __p += '<ul>\r\n    <li>Username:\r\n        <input type="text" value="' +
      ((__t = (username)) == null ? '' : __t) +
      '" />\r\n    </li>\r\n    <li>Password:\r\n        <input type="password" value="' +
      ((__t = (password)) == null ? '' : __t) +
      '" />\r\n    </li>\r\n</ul>\r\n';
  }
  return __p;
}
```

最终输出是一个`TemplateManager`对象，其属性为模板的文件名，属性值为模板的编译版本。这样，你所有的模板文件都将被添加到`TemplateManager`对象中。然而，对于这段代码，你需要确保每个模板的文件名是不同的。否则，同名文件的模板将被另一个模板覆盖。

你不需要理解这个编译后的模板函数定义，因为这个函数将在库内部使用。请放心，一旦你用数据对象调用这个函数，你将得到正确的 HTML 输出：

```js
var user = new Backbone.Model({
  username: 'hello',
  password: 'world'
});

// Get the html
var html = Templates.cachedTemplates.userLogin(user.toJSON());
```

这种预编译 JavaScript 模板的解决方案非常有效，你可以在你的项目中自由地使用这个概念。我们已经成功地在多个项目中使用了这个概念。
