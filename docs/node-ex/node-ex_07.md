# 第七章：发布内容

第六章，“添加友谊功能”，是关于添加友谊功能的。在社交网络中与其他用户建立联系的能力很重要。然而，更重要的是提供一个生成内容的接口。在本章中，我们将实现内容创建背后的逻辑。我们将涵盖以下主题：

+   发布和存储文本

+   显示用户的动态

+   发布文件

# 发布和存储文本

与前几章一样，我们有一个需要在应用程序的前端和后端部分都进行更改的功能。我们需要一个 HTML 表单，接受用户的文本，一个处理与后端通信的新模型，当然，还有 API 的更改。让我们从更新我们的主页开始。

## 添加一个发布文本消息的表单

我们有一个显示简单标题的主页。让我们使用它，并添加一个`<textarea>`标签来将内容发送到 API。在本章的后面，我们将使用同一个页面来显示用户的动态。让我们用以下标记替换孤独的`<h1>`标签：

```js
{{#if posting === true}}
  <form enctype="multipart/form-data" method="post">
    <h3>What is on your mind?</h3>
    {{#if error && error != ''}}
      <div class="error">{{{error}}}</div>
    {{/if}}
    {{#if success && success != ''}}
      <div class="success">{{{success}}}</div>
    {{/if}}
    <label for="text">Text</label>
    <textarea value="{{text}}"></textarea>
    <input type="file" name="file" />
    <input type="button" value="Post" on-click="post" />
  </form>
{{else}}
  <h1>Node.js by example</h1>
{{/if}}
```

我们仍然有标题，但只有当`posting`变量等于`false`时才显示。在接下来的部分中，我们将更新主页的控制器，我们将使用`posting`来保护内容的表单。在某些情况下，我们不希望使`<textarea>`可见。

请注意，我们有两个块来显示消息。如果在发布过程中出现错误，第一个块将可见，当一切顺利时，第二个块将可见。表单的其余部分是所需的用户界面——文本区域、输入文件字段和一个按钮。按钮会触发一个发布事件，我们将在控制器中捕获到。

## 介绍内容模型

我们肯定需要一个模型来管理与 API 的通信。让我们创建一个新的`models/Content.js`文件，并将以下代码放在那里：

```js
var ajax = require('../lib/Ajax');
var Base = require('./Base');

module.exports = Base.extend({
  data: {
    url: '/api/content'
  },
  create: function(content, callback) {
    var self = this;
    ajax.request({
      url: this.get('url'),
      method: 'POST',
      data: {
        text: content.text
      },
      json: true
    })
    .done(function(result) {
      callback(null, result);
    })
    .fail(function(xhr) {
      callback(JSON.parse(xhr.responseText));
    });
  }
});
```

该模块扩展了相同的`models/Base.js`类，它类似于我们系统中的其他模型。需要`lib/Ajax.js`模块，因为我们将进行 HTTP 请求。我们应该熟悉其余的代码。通过将文本作为参数传递给`create`函数，向`/api/content`发出`POST`请求。

当我们到达文件发布时，该模块将被更新。要创建仅基于文本的记录，这就足够了。

## 更新主页的控制器

现在我们有了一个合适的模型和形式，我们准备调整主页的控制器。如前所述，`posting`变量控制表单的可见性。它的值将默认设置为`true`，如果用户未登录，我们将把它改为`false`。每个 Ractive.js 组件都可以有一个`data`属性。它表示所有内部变量的初始状态：

```js
// controllers/Home.js
module.exports = Ractive.extend({
  template: require('../../tpl/home'),
  components: {
    navigation: require('../views/Navigation'),
    appfooter: require('../views/Footer')
  },
  data: {
    posting: true
  }
});
```

现在，让我们向`onrender`处理程序添加一些逻辑。这是我们组件的入口点。我们将首先检查当前用户是否已登录：

```js
onrender: function() {
  if(userModel.isLogged()) {
    // ...
  } else {
    this.set('posting', false);
  }
}
```

从第五章，“管理用户”中，我们知道`userModel`是一个全局对象，我们可以用它来检查当前用户的状态。如前所述，如果我们有一个未经授权的访问者，我们必须将`posting`设置为`false`。

下一个逻辑步骤是处理表单中的内容并向 API 提交请求。我们将使用新创建的`ContentModel`类，如下所示：

```js
var ContentModel = require('../models/Content');
var model = new ContentModel();
var self = this;
this.on('post', function() {
  model.create({
    text: this.get('text')
  }, function(error, result) {
    self.set('text', '');
    if(error) {
      self.set('error', error.error);
    } else {
      self.set('error', false);
      self.set('success', 'The post is saved successfully.<br />What about adding another one?');
    }
  });
});
```

一旦用户在表单中按下按钮，我们的组件就会触发一个`post`事件。然后我们将捕获事件并调用模型的`create`方法。给用户一个合适的响应很重要，所以我们用`self.set('text', '')`清除文本字段，并使用本地的`error`和`success`变量来指示请求的状态。

## 在数据库中存储内容

到目前为止，我们有一个 HTML 表单，它向 API 提交 HTTP 请求。在本节中，我们将更新我们的 API，以便我们可以在数据库中存储文本内容。我们模型的端点是`/api/content`。我们将添加一个新的路由，并通过允许只有授权用户访问来保护它：

```js
// backend/API.js
.add('api/content', function(req, res) {
  var user;
  if(req.session && req.session.user) {
    user = req.session.user;
  } else {
    error('You must be logged in in order to use this method.', res);
  }
})
```

我们将创建一个包含访客会话数据的`user`本地变量。发送到数据库的每个帖子都应该有一个所有者。因此，有一个快捷方式到用户的个人资料是很好的。

同样的`/api/content`目录也将用于获取帖子。同样，我们将使用`req.method`属性来查找请求的类型。如果是`GET`，我们需要从数据库中获取帖子并将它们发送到浏览器。如果是`POST`，我们需要创建一个新的条目。以下是将用户的文本发送到数据库的代码：

```js
switch(req.method) {
  case 'POST':
    processPOSTRequest(req, function(data) {
      if(!data.text || data.text === '') {
        error('Please add some text.', res);
      } else {
        getDatabaseConnection(function(db) {
          getCurrentUser(function(user) {
            var collection = db.collection('content');
            data.userId = user._id.toString();
            data.userName = user.firstName + ' ' + user.lastName;
            data.date = new Date();
            collection.insert(data, function(err, docs) {
              response({
                success: 'OK'
              }, res);
            });
          }, req, res);
        });
      }
    });
  break;
};
```

浏览器发送的数据作为`POST`变量传递。同样，我们需要`processPOSTRequest`的帮助来访问它。如果没有`.text`或者它是空的，API 将返回一个错误。如果一切正常并且文本消息可用，我们将继续建立数据库连接。我们还会获取当前用户的整个个人资料。我们的社交网络中的帖子将与以下附加属性一起保存：

+   `userId`：这代表了记录的创建者。我们将在生成动态时使用这个属性。

+   `userName`：我们不想为我们显示的每一篇帖子都调用`getCurrentUser`。因此，所有者的名称直接与文本一起存储。值得一提的是，在某些情况下，这样的调用是必要的。例如，在更改用户的名称时，将需要这些调用。

+   `date`：我们应该知道数据的创建日期。这对于数据的排序或过滤是有用的。

最后，我们调用`collection.insert`，这实际上将条目存储在数据库中。

在下一节中，我们将看到如何检索创建的内容并将其显示给用户。

# 显示用户的动态

现在，每个用户都能够在我们的数据库中存储消息。让我们继续通过在浏览器中显示记录来展示。我们将首先向获取帖子的 API 添加逻辑。这将很有趣，因为你不仅应该获取特定用户发送的消息，还应该获取他/她的朋友发送的消息。我们使用`POST`方法来创建内容。接下来的行将处理`GET`请求。

首先，我们将以以下方式获取用户的朋友的 ID：

```js
case 'GET':
  getCurrentUser(function(user) {
    if(!user.friends) {
      user.friends = [];
    }
    // ...
break;
```

在上一章中，我们实现了友谊功能，并直接在用户的个人资料中保留了用户的朋友的 ID。`friends`数组正是我们需要的，因为我们的社交网络中的帖子是通过它们的 ID 与用户的个人资料相关联的。

下一步是建立与数据库的连接，并仅查询与特定 ID 匹配的记录，如下所示：

```js
case 'GET':
  getCurrentUser(function(user) {
    if(!user.friends) {
      user.friends = [];
    }
    getDatabaseConnection(function(db) {
      var collection = db.collection('content');
      collection.find({ 
        $query: {
          userId: { $in: [user._id.toString()].concat(user.friends) }
        },
        $orderby: {
          date: -1
        }
      }).toArray(function(err, result) {
        result.forEach(function(value, index, arr) {
          arr[index].id = ObjectId(value.id);
          delete arr[index].userId;
        });
        response({
          posts: result
        }, res);
      });
    });
  }, req, res);
break;
```

我们将从`content`集合中读取记录。`find`方法接受一个具有`$query`和`$orderby`属性的对象。在第一个属性中，我们将放入我们的条件。在这种特殊情况下，我们想要获取所有属于`friends`数组的记录的 ID。为了创建这样的查询，我们需要`$in`运算符。它接受一个数组。除了用户的朋友的帖子，我们还需要显示用户的帖子。因此，我们将创建一个数组，其中包含一个项目——当前用户的 ID，并将其与`friends`连接起来，如下所示：

```js
[user._id.toString()].concat(user.friends)
```

成功查询后，`userId`属性将被删除，因为它不再需要。在`content`集合中，我们保留消息的文本和所有者的名称。最后，记录将附加到`posts`属性上发送。

通过在前面的代码中添加的内容，我们的后端返回了当前用户和他们的朋友发布的帖子。我们所要做的就是更新我们主页的控制器并使用 API 的方法。在监听`post`事件的代码之后，我们添加以下代码：

```js
var getPosts = function() {
  model.fetch(function(err, result) {
    if(!err) {
      self.set('posts', result.posts);
    }
  });
};
getPosts();
```

调用`fetch`方法触发对模型端点`/api/content`的 API 的`GET`请求。这个过程被包装在一个函数中，因为当创建新帖子时会发生相同的操作。正如我们已经知道的，如果`model.create`成功，就会触发回调。我们将在那里添加`getPosts()`，这样用户就可以在动态中看到他/她的最新帖子：

```js
// frontend/js/controllers/Home.js
model.create(formData, function(error, result) {
  self.set('text', '');
  if(error) {
    self.set('error', error.error);
  } else {
    self.set('error', false);
    self.set('success', 'The post is saved  successfully.<br />What about adding another one?');
    getPosts();
  }
});
```

`getPosts`函数产生的结果是存储在名为`posts`的本地变量中的对象列表。同样的变量可以在 Ractive.js 模板中访问。我们需要遍历数组中的项目，并在屏幕上显示信息，如下所示：

```js
// frontend/tpl/home.html
<header>
  <navigation></navigation>
</header>
<div class="hero">
  {{#if posting === true}}
    <form enctype="multipart/form-data" method="post">
      ...
    </form>
    {{#each posts:index}}
      <div class="content-item">
        <h2>{{posts[index].userName}}</h2>
        {{posts[index].text}}
      </div>
    {{/each}}
  {{else}}
    <h1>Node.js by example</h1>
  {{/if}}
</div>
<appfooter />
```

在表单之后，我们使用`each`操作符来显示帖子的作者和文本。

在这一点上，我们网络中的用户将能够创建和浏览以文本块形式的消息。在下一节中，我们将扩展到目前为止编写的功能，并使上传图像与文本一起成为可能。

# 发布文件

我们正在构建一个单页面应用程序。这类应用程序的特点之一是所有操作都在不重新加载页面的情况下进行。上传文件而不改变页面一直是棘手的。过去，我们使用涉及隐藏 iframe 或小型 Flash 应用程序的解决方案。幸运的是，当 HTML5 出现时，它引入了**FormData**接口。

流行的 Ajax 是由`XMLHttpRequest`对象实现的。2005 年，Jesse James Garrett 创造了“Ajax”这个术语，我们开始使用它在 JavaScript 中进行 HTTP 请求。以以下方式执行`GET`或`POST`请求变得很容易：

```js
var http = new XMLHttpRequest();
var url = "/api/content";
var params = "text=message&author=name";
http.open("POST", url, true);

http.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
http.setRequestHeader("Content-length", params.length);
http.setRequestHeader("Connection", "close");

http.onreadystatechange = function() {
  if(http.readyState == 4 && http.status === 200) {
    alert(http.responseText);
  }
}

http.send(params);
```

前面的代码生成了一个正确的`POST`请求，甚至设置了正确的标头。问题在于参数被表示为字符串。形成这样的字符串需要额外的工作。发送文件也很困难。这可能是相当具有挑战性的。

FormData 接口解决了这个问题。我们创建一个对象，它是表示表单字段及其值的键/值对集合。然后，我们将这个对象传递给`XMLHTTPRequest`类的`send`方法：

```js
var formData = new FormData();
var fileInput = document.querySelector('input[type="file"]');
var url = '/api/content';

formData.append("username", "John Black");
formData.append("id", 123456);
formData.append("userfile", fileInput.files[0]);

var request = new XMLHttpRequest();
request.open("POST", url);
request.send(formData);
```

我们所要做的就是使用`append`方法并指定`file`类型的`input` DOM 元素。其余工作由浏览器完成。

为了提供上传文件的功能，我们需要添加文件选择的 UI 元素。以下是`home.html`模板中表单的样子：

```js
<form enctype="multipart/form-data" method="post">
  <h3>What is on your mind?</h3>
  {{#if error && error != ''}}
    <div class="error">{{error}}</div>
  {{/if}}
  {{#if success && success != ''}}
    <div class="success">{{{success}}}</div>
  {{/if}}
  <label for="text">Text</label>
  <textarea value="{{text}}"></textarea>
  <input type="file" name="file" />
  <input type="button" value="Post" on-click="post" />
</form>
```

相同的代码，但是有一个新的`input`元素，类型等于`file`。到目前为止，我们的控制器中发送`POST`请求的实现并没有使用`FormData`接口。让我们改变这一点，并更新`controllers/Home.js`文件：

```js
this.on('post', function() {
  var files = this.find('input[type="file"]').files;
  var formData = new FormData();
  if(files.length > 0) {
    var file = files[0];
    if(file.type.match('image.*')) {
      formData.append('files', file, file.name);
    }
  }
  formData.append('text', this.get('text'));
  model.create(formData, function(error, result) {
    self.set('text', '');
    if(error) {
      self.set('error', error.error);
    } else {
      self.set('error', false);
      self.set('success', 'The post is saved  successfully.<br />What about adding another one?');
      getPosts();
    }
  });
});
```

代码已经改变。因此，代码创建了一个新的`FormData`对象，并使用`append`方法收集新帖子所需的信息。我们确保用户选择的文件被附加。默认情况下，HTML 输入只提供选择一个文件。但是，我们可以添加`multiple`属性，浏览器将允许我们选择多个文件。值得一提的是，我们过滤所选文件，并且只使用图像。

经过最新的更改，我们模型的`create`方法接受`FormData`对象而不是普通的 JavaScript 对象。因此，我们也必须更新模型：

```js
// models/Content.js
create: function(formData, callback) {
  var self = this;
  ajax.request({
    url: this.get('url'),
    method: 'POST',
    formData: formData,
    json: true
  })
  .done(function(result) {
    callback(null, result);
  })
  .fail(function(xhr) {
    callback(JSON.parse(xhr.responseText));
  });
}
```

`data`属性被`formData`属性替换。现在我们知道前端将选定的文件发送到 API。但是，我们没有处理`multipart/form-data`类型的`POST`数据的代码。通过`POST`请求发送的文件的处理并不简单，`processPOSTRequest`在这种情况下无法完成任务。

Node.js 拥有一个庞大的社区，有成千上万的模块可用。`formidable`模块是我们要使用的。它有一个相当简单的 API，并且处理包含文件的请求。文件上传过程中，`formidable`会将文件保存在服务器硬盘的特定位置。然后，我们会收到资源的路径。最后，我们必须决定如何处理它。

在`backend/API.js`文件中，应用流程分为`GET`和`POST`请求。我们将更新`POST`情况的一个重要部分。以下行包含了`formidable`的初始化：

```js
case 'POST':
  var formidable = require('formidable');
  var uploadDir = __dirname + '/../static/uploads/';
  var form = new formidable.IncomingForm();
  form.multiples = true;
  form.parse(req, function(err, data, files) {
    // ...
  });
break;
```

正如我们之前提到的，该模块将上传的文件保存在硬盘上的临时文件夹中。`uploadDir`变量包含了用户图片的更合适的位置。传递给`formidable`的`parse`函数的回调在`data`参数中接收普通文本字段，并在`files`中上传图像。

为了避免嵌套 JavaScript 回调的长链条，我们将一些逻辑提取到函数定义中。例如，将文件从`temporary`移动到`static`文件夹可以按以下方式执行：

```js
var processFiles = function(userId, callback) {
  if(files.files) {
    var fileName = userId + '_' + files.files.name;
    var filePath = uploadDir + fileName;
    fs.rename(files.files.path, filePath, function() {
      callback(fileName);
    });
  } else {
    callback();
  }
};
```

我们不想混合不同用户的文件。因此，我们将使用用户的 ID 并创建他/她自己的文件夹。还有一些其他问题可能需要我们处理。例如，我们可以为每个文件创建子文件夹，以防止已上传资源的覆盖。然而，为了尽可能保持代码简单，我们将在这里停止。

以下是将帖子保存到数据库的完整代码：

```js
case 'POST':
  var uploadDir = __dirname + '/../static/uploads/';
  var formidable = require('formidable');
  var form = new formidable.IncomingForm();
  form.multiples = true;
  form.parse(req, function(err, data, files) {
    if(!data.text || data.text === '') {
      error('Please add some text.', res);
    } else {
      var processFiles = function(userId, callback) {
        if(files.files) {
          var fileName = userId + '_' + files.files.name;
          var filePath = uploadDir + fileName;
          fs.rename(files.files.path, filePath, function(err) {
            if(err) throw err;
            callback(fileName);
          });
        } else {
          callback();
        }
      };
      var done = function() {
        response({
          success: 'OK'
        }, res);
      }
      getDatabaseConnection(function(db) {
        getCurrentUser(function(user) {
          var collection = db.collection('content');
          data.userId = user._id.toString();
          data.userName = user.firstName + ' ' + user.lastName;
          data.date = new Date();
          processFiles(user._id, function(file) {
            if(file) {
              data.file = file;
            }
            collection.insert(data, done);
          });
        }, req, res);
      });
    }
  });
break;
```

我们仍然需要与数据库建立连接并获取当前用户的个人资料。这里的不同之处在于，我们向存储在 MongoDB 中的对象附加了一个新的`file`属性。

最后，我们必须更新主页的模板，以便显示上传的文件：

```js
{{#each posts:index}}
  <div class="content-item">
    <h2>{{posts[index].userName}}</h2>
    {{posts[index].text}}
    {{#if posts[index].file}}
    <img src="img/{{posts[index].file}}" />
    {{/if}}
  </div>
{{/each}}
```

现在，`each`循环检查是否有文件与帖子文本一起传输。如果有，它会显示一个显示图像的`img`标签。通过这最后的添加，我们社交网络的用户将能够创建由文本和图片组成的内容。

# 总结

在本章中，我们为我们的应用程序做了一些非常重要的事情。通过扩展我们的后端 API，我们实现了内容的创建和传递。前端也进行了一些更改。

在下一章中，我们将继续添加新功能。我们将使创建品牌页面和活动成为可能。
