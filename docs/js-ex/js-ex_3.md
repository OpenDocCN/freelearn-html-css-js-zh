# 第三章：活动注册应用程序

希望您在创建表情并与朋友分享时玩得很开心！您在上一个项目中成功使用 HTML5 画布构建了一个表情创作器。您还使用了 flexbox 来设计页面布局，并学习了有关 ES6 模块的一些知识。

上一章最重要的部分是我们使用 Webpack 创建的开发环境。它让我们可以使用`HotModuleReplacement`更快地开发应用程序，创建具有单个文件资产和减小代码大小的优化生产构建，并且还可以隐藏原始源代码，同时我们可以使用源映射来调试原始代码。

现在我们有了模块支持，我们可以使用它来创建模块化函数，这将允许我们编写可重用的代码，可以在项目的不同部分之间使用，也可以在不同的项目中使用。在本章中，您将构建一个活动注册应用程序，同时学习以下概念：

+   编写 ES6 模块

+   使用 JavaScript 进行表单验证

+   使用动态数据（从服务器加载的数据）

+   使用 fetch 进行 AJAX 请求

+   使用 Promises 处理异步函数

+   使用 Chart.js 创建图表

# 活动 - JS 聚会

以下是我们项目的情景：

您正在本地组织一个 JavaScript 聚会。您邀请了来自学校、大学和办公室的对 JavaScript 感兴趣的人。您需要为与会者创建一个注册活动的网站。该网站应具有以下功能：

+   帮助用户注册活动的表单

+   显示对活动感兴趣的用户数量的统计数据页面

+   关于页面，包括活动详情和活动位置的 Google 地图嵌入

此外，大多数人将使用手机注册活动。因此，应用程序应完全响应。

这是应用程序在手机上的样子：

![](img/00020.jpeg)

# 初始项目设置

要开始项目，请在 VSCode 中打开第三章的起始文件。创建一个`.env`文件，并使用`.env.example`文件中的值。为每个环境变量分配以下值：

+   `NODE_ENV=dev`：在生成构建时应设置为`production`。

+   `SERVER_URL=http://localhost:3000`：我们很快将在此 URL 上运行服务器。

+   `GMAP_KEY`：我们将在此项目中使用 Google Maps API。您需要生成自己的唯一 API 密钥以使用 Google Maps。请参阅：[`developers.google.com/maps/documentation/javascript/get-api-key`](https://developers.google.com/maps/documentation/javascript/get-api-key) 生成您的 API 密钥，并将密钥添加到此环境变量中。

在第二章中，*构建表情创作器*，我提到当模块与 Webpack 捆绑在一起时，您无法在 HTML 中访问 JavaScript 变量。在第一章中，*构建待办事项列表*，我们使用 HTML 属性调用 JavaScript 函数。这看起来可能很有用，但它也会向用户（我指的是访问您页面的其他开发人员）公开我们的对象结构。用户可以通过检查 Chrome DevTools 来清楚地了解`ToDoClass`类的结构。在构建大型应用程序时应该防止这种情况发生。因此，Webpack 不允许变量存在于全局范围内。

一些插件需要全局范围内存在变量或对象（比如我们将要使用的 Google Maps API）。为此，Webpack 提供了一个选项，可以将一些选定的对象作为库暴露到全局范围内（在 HTML 内）。查看起始文件中的`webpack.config.js`文件。在`output`部分，我已经添加了`library: 'bundle'`，这意味着如果我们向任何函数、变量或对象添加`export`关键字，它们将在全局范围内的`bundle`对象中可访问。我们将看到如何在向我们的应用程序添加 Google Maps 时使用它。

现在我们已经准备好环境变量，打开项目根文件夹中的终端并运行`npm install`来安装所有依赖项。一旦依赖项安装完成，在终端中输入`npm run watch`来启动 Webpack 开发服务器。您现在可以在控制台中由 Webpack 打印的本地主机 URL（`http://localhost:8080/`）上查看页面。查看所有页面。

# 向页面添加样式

目前，页面是响应式的，因为它是使用 Bootstrap 构建的。然而，我们仍然需要对表单进行一些样式更改。在桌面屏幕上，它目前非常大。此外，我们需要将标题对齐到页面中央。让我们为`index.html`页面添加样式。

将表单及其标题居中对齐到页面中央，在`styles.css`文件（`src/css/styles.css`）中添加以下代码（确保 Webpack 开发服务器正在运行）：

```js
.form-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
```

由于 Webpack 中启用了`HotModuleReplacement`，样式将立即反映在页面上（不再重新加载！）。现在，给标题添加一些边距，并为表单设置最小宽度：

```js
.title {
  margin: 20px;
}
.form-group {
  min-width: 500px;
}
```

现在表单的最小宽度将为`500px`。然而，我们面临另一个问题！由于表单将始终为`500px`，在移动设备上（移动用户是我们的主要受众）将超出屏幕。我们需要使用媒体查询来解决这个问题。媒体查询允许我们根据页面所在的媒介类型添加 CSS。在我们的情况下，我们需要在移动设备上更改`min-width`。要查询移动设备，请在先前的样式下方添加以下样式：

```js
@media only screen and (max-width: 736px) {
  .form-group {
    min-width: 90vw;
  }
}
```

这将检查设备宽度是否小于`736px`（通常，移动设备属于此类别），然后添加`90vw`的`min-width`。`vw`代表视口宽度。`90vw`表示视口宽度的大小的 90%（这里，视口是屏幕）。

有关使用媒体查询的更多信息，请访问 w3schools 页面：[`www.w3schools.com/css/css_rwd_mediaqueries.asp`](https://www.w3schools.com/css/css_rwd_mediaqueries.asp)。

我在`index.html`和`status.html`页面上使用了加载指示器图像。要指定图像的大小而不破坏其原始宽高比，使用`max-width`和`max-height`如下：

```js
.loading-indicator {
  max-height: 50px;
  max-width: 50px;
}
```

查看状态页面。加载指示器的大小将被减小。我们已经为我们的应用程序添加了必要的样式。现在，是时候使用 JavaScript 使其工作了。

# 使用 JavaScript 验证和提交表单

HTML 表单是 Web 应用程序中最重要的部分，用户输入会被记录下来。在我们的 JS Meetup 应用程序中，我们使用 Bootstrap 构建了一个漂亮的表单。让我们使用`index.html`文件来探索表单包含的内容。表单包含四个必填字段：

+   姓名

+   电子邮件地址

+   电话号码

+   年龄

它还包含三个可选字段（其中两个的值已经预先选择）：

+   用户的职业

+   他在 JavaScript 方面的经验水平

+   对他对这次活动期望学到的内容进行评论

由于职业和经验水平选项已预先选择了默认值，因此它们不会被标记为用户必填。但是，在验证期间，我们需要将它们视为必填字段。只有评论字段是可选的。

这是我们的表单应该如何工作的：

+   用户填写所有表单细节并点击提交

+   表单详细信息将被验证，如果缺少任何必填字段，它将用红色边框突出显示这些字段

+   如果表单值有效，它将继续将表单提交到服务器

+   提交表单后，用户将收到通知表单已成功提交，并且表单条目将被清除

JavaScript 最初用作在 HTML 中进行表单验证的语言。随着时间的推移，它已经发展成为一个完整的 Web 应用程序开发语言。使用 JavaScript 构建的 Web 应用程序会向服务器发出许多请求，以向用户提供动态数据。这些网络请求始终是异步的，需要正确处理。

# HTML 表单

在我们实现表单验证逻辑之前，让我们先了解表单的正常工作方式。单击当前表单中的提交。您应该会看到一个空白页面，并显示消息“无法 POST /register”。这是 Webpack 开发服务器的消息，表示没有为`/register`配置`POST`方法的路由。这是因为在`index.html`中，表单是使用以下属性创建的：

```js
<form action="/register" method="post" id="registrationForm">
```

这意味着当单击提交按钮发送数据到`/register`页面时，使用`POST`方法。在进行网络请求时，`GET`和`POST`是两种常用的 HTTP 方法或动词。`GET`方法不能有请求正文，因此所有数据都通过 URL 作为查询参数传输。但是，`POST`方法可以有请求正文，其中数据可以作为表单数据或 JSON 对象发送。

有不同的 HTTP 方法用于与服务器通信。查看以下 REST API 教程页面，了解有关 HTTP 方法的更多信息：[`www.restapitutorial.com/lessons/httpmethods.html`](http://www.restapitutorial.com/lessons/httpmethods.html)。

当前，表单以`POST`方法使用表单数据发送数据。在您的`index.html`文件中，将表单方法属性更改为`get`并重新加载页面（Webpack 开发服务器不会自动重新加载 HTML 文件的更改）。现在，单击提交。您应该看到类似的空白页面，但是现在表单详细信息正在发送到 URL 本身。现在 URL 将如下所示：

```js
http://localhost:8080/register?username=&email=&phone=&age=&profession=school&experience=1&comment=
```

所有字段都为空，除了职业和经验，因为它们是预先选择的。表单值添加在路由`/register`的末尾，后跟一个`?`符号，指定下一个文本是查询参数，表单值使用`&`符号分隔。由于`GET`请求会将数据发送到 URL 本身，因此不适合发送机密数据，例如登录详细信息或我们将在此表单中发送的用户详细信息。因此，选择`POST`方法进行表单提交。在您的`index.html`文件中将方法更改为 post。

让我们看看如何检查使用`POST`请求发送的数据。打开 Chrome DevTools 并选择网络选项卡。现在在表单中输入一些详细信息，然后单击提交。您应该在网络请求列表中看到一个名为`register`的新条目。如果单击它，它将打开一个新面板，其中包含请求详细信息。请求数据将出现在表单数据部分的标头选项卡中。请参考以下屏幕截图：

![](img/00021.jpeg)

Chrome DevTools 具有许多用于处理网络请求的工具。我们只使用它来检查我们发送的数据。但是您还可以做更多的事情。根据上图，您可以在标头选项卡的表单数据部分中看到我在表单中输入的表单值。

访问以下 Google 开发者页面：[`developers.google.com/web/tools/chrome-devtools/`](https://developers.google.com/web/tools/chrome-devtools/) 以了解更多关于使用 Chrome DevTools 的信息。

现在你对提交表单的工作原理有了一个很好的了解。我们在`/register`路由中没有创建任何页面，并且通过将表单重定向到单独的页面进行提交不再是一个好的用户体验（我们处于**单页应用程序**（**SPA**）的时代）。考虑到这一点，我创建了一个小的 Node.js 服务器应用程序，可以接收表单请求。我们将禁用默认的表单提交操作，并将使用 JavaScript 作为 AJAX 请求提交表单。

# 在 JavaScript 中读取表单数据

是时候编码了！使用`npm run watch`命令保持 Webpack 开发服务器运行（`NODE_ENV`变量应为`dev`）。在 VSCode 中打开项目文件夹，并从`src/js/`目录中打开`home.js`文件。我已经在`index.html`文件中添加了对`dist/home.js`的引用。我还将在`home.js`中添加代码来导入`general.js`文件。现在，在导入语句下面添加以下代码：

```js
class Home {
  constructor() {

  }

}

window.addEventListener("load", () => {
 new Home();
});
```

这将创建一个新的`Home`类，并在页面加载完成时创建一个新的实例。我们不需要将实例对象分配给任何变量，因为我们不会像在 ToDo 列表应用程序中那样在 HTML 文件中使用它。一切都将从 JavaScript 本身处理。

我们的第一步是创建对表单中所有输入字段和表单本身的引用。这包括表单本身和当前在页面中使用`.hidden` Bootstrap 类隐藏的加载指示器。将以下代码添加到类的构造函数中：

```js
 this.$form = document.querySelector('#registrationForm');
 this.$username = document.querySelector('#username');
 this.$email = document.querySelector('#email');
 this.$phone = document.querySelector('#phone');
 this.$age = document.querySelector('#age');
 this.$profession = document.querySelector('#profession');
 this.$experience = document.querySelector('#experience');
 this.$comment = document.querySelector('#comment');
 this.$submit = document.querySelector('#submit');
 this.$loadingIndicator = document.querySelector('#loadingIndicator');
```

就像我在构建 Meme Creator 时提到的，最好将对 DOM 元素的引用存储在以`$`符号为前缀的变量中。现在，我们可以轻松地从其他变量中识别具有对 DOM 元素的引用的变量。这纯粹是为了开发效率，不是你需要遵循的严格规则。在前面的代码中，对于体验单选按钮，只存储了第一个单选按钮的引用。这是为了重置单选按钮；要读取所选单选按钮的值，需要使用不同的方法。

现在我们可以在`Home`类中访问所有的 DOM 元素。触发整个表单验证过程的事件是表单提交时发生的。表单提交事件发生在`<form>`元素内部带有属性`type="submit"`的 DOM 元素被点击时。在我们的情况下，`<button>`元素包含这个属性，并且被引用为`$submit`变量。尽管`$submit`触发了提交事件，但事件属于整个表单，也就是`$form`变量。因此，我们需要在我们的类中为`this.$form`添加一个事件监听器。

我们只会有一个事件监听器。因此，在声明前面的变量之后，只需将以下代码添加到构造函数中：

```js
this.$form.addEventListener('submit', event => {
  this.onFormSubmit(event);
});
```

这将为表单附加一个事件监听器，并在表单提交时调用类的`onFormSubmit()`方法，以表单提交事件作为其参数。因此，让我们在`Home`类中创建`onFormSubmit()`方法：

```js
onFormSubmit(event) {
  event.preventDefault();
}
```

`event.preventDefault()`将阻止默认事件动作发生。在我们的情况下，它将阻止表单的提交。在 Chrome 中打开页面（`http://localhost:8080/`）并尝试点击提交。如果没有任何动作发生，那太好了！我们的 JavaScript 代码正在阻止表单提交。

我们可以使用这个函数来启动表单验证。表单验证的第一步是读取表单中所有输入元素的值。在`Home`类中创建一个新的方法`getFormValues()`，它将以 JSON 对象的形式返回表单字段的值：

```js
getFormValues() {
  return {
    username: this.$username.value,
    email: this.$email.value,
    phone: this.$phone.value,
    age: this.$age.value,
    profession: this.$profession.value,
    experience: parseInt(document.querySelector('input[name="experience"]:checked').value),
    comment: this.$comment.value,
  };
}
```

看到我如何使用`document.querySelector()`来读取选中的单选按钮的值了吗？该函数本身就是不言自明的。我添加了`parseInt()`，因为该值将作为字符串返回，并且需要转换为 Int 以进行验证。在`onFormSubmit()`方法中创建一个变量来存储表单中所有字段的值。您的`onFormSubmit()`方法现在将如下所示：

```js
onFormSubmit(event) {
  event.preventDefault();
  const formValues = this.getFormValues();
}
```

尝试使用`console.log(formValues)`在 Chrome DevTools 控制台中打印`formValues`变量。您应该看到一个 JSON 对象中的所有字段及其相应的值。现在我们有了所需的值，下一步是验证数据。

在我们的 JS Meetup 应用程序中，我们只有一个表单。但在更大的应用程序中，您可能会在应用程序的不同部分中有多个表单执行相同的操作。但是，由于设计目的，表单将具有不同的 HTML 类和 ID，但表单值将保持不变。在这种情况下，验证逻辑可以在整个应用程序中重复使用。这是构建您的第一个可重用 JavaScript 模块的绝佳机会。

# 表单验证模块

通过使用 Webpack，我们现在有能力创建单独的模块并在 JavaScript 中导入它们。但是，我们需要某种方法来组织我们创建的模块。随着应用程序的规模增长，您可能会有数十甚至数百个模块。以便能够轻松识别它们的方式来组织它们将极大地帮助您的团队，因为他们将能够在需要时轻松找到模块，而不是重新创建具有相同功能的模块。

在我们的应用程序中，让我们在`src/js/`目录中创建一个名为`services`的新文件夹。该目录将包含所有可重用的模块。现在，在`services`目录中，创建另一个名为`formValidation`的目录，在其中我们将创建`validateRegistrationForm.js`文件。您的项目`src/js/`目录现在将如下所示：

```js
.
├── about.js
├── general.js
├── home.js
├── services
│   └── formValidation
│       └── validateRegistrationForm.js
└── status.js
```

现在，想象自己是一个第一次看到这段代码的不同开发人员。在`js`目录中，有另一个名为`services`的目录。在其中，`formValidation`作为一个服务可用。您现在知道有一个用于表单验证的服务。如果您查看此目录，它将具有`validateRegistrationForm.js`文件，该文件仅凭其文件名就告诉您此模块的目的。

如果您想为登录表单创建一个验证模块（只是一个想象的场景），只需在`formValidation`目录中创建另一个名为`validateLoginForm.js`的文件。这样，您的代码将易于维护，并通过最大程度地重用所有模块来扩展。

不要担心文件名太长！可维护的代码更重要，但如果文件名太长，它会更容易理解该文件的目的。但是，如果您在团队中工作，请遵守团队使用的 lint 工具的规则。

是时候构建模块了！在您刚刚创建的`validateRegistrationForm.js`文件中，添加以下代码：

```js
export default function validateRegistrationForm(formValues) {
}
```

使用模块文件和其默认导出项相同的名称将使导入语句看起来更容易理解。当您将此模块导入到您的`home.js`文件中时，您将看到这一点。前面的函数将接受`formValues`（我们从上一节中的表单中读取的）JSON 对象作为参数。

在编写此函数之前，我们需要为每个输入字段设置验证逻辑为单独的函数。当输入满足验证条件时，这些函数将返回 true。让我们从验证用户名开始。在`validateRegistrationForm()`下面，创建一个名为`validateUserName()`的新函数，如下所示：

```js
function validateUserName(name) {
  return name.length > 3 ? true: false;
}
```

我们使用此函数来检查用户名是否至少为`3`个字符长。我们使用条件运算符，如果长度大于`3`则返回`true`，如果长度小于`3`则返回`false`。

我们之前在 ToDo 列表应用程序中使用了条件运算符`()?:`。如果您仍然对这个运算符有困难，可以访问以下 MDN 页面进行了解：[`developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Conditional_Operator`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)。

我们可以使这个函数更加简洁：

```js
function validateUserName(name) {
  return name.length > 3;
}
```

这样，JavaScript 将自动评估长度是否大于三，并根据结果分配 true 或 false。现在，要验证电子邮件地址，我们需要使用正则表达式。我们曾经使用正则表达式来更改 Meme Creator 应用程序中图像的 MIME 类型。这一次，我们将研究正则表达式的工作原理。

# 在 JavaScript 中使用正则表达式

正则表达式（RegExp）基本上是一个模式的定义（例如一系列字符、数字等），可以在其他文本中进行搜索。例如，假设您需要找到段落中以字母*a*开头的所有单词。然后，在 JavaScript 中，您将模式定义为：

```js
const pattern = /^a+/
```

正则表达式总是在`/ /`内定义。在前面的代码片段中，我们有以下内容：

+   `^`表示在开头

+   `+`表示至少有一个

这个正则表达式将匹配以字母*a*开头的字符串。您可以在以下网址测试这些语句：[`jsfiddle.net/`](https://jsfiddle.net/)。要使用这个正则表达式验证一个字符串，请执行以下操作：

```js
pattern.test('alpha') // this will return true
pattern.test('beta') // this will return false
```

要验证电子邮件地址，请使用以下函数，其中包含一个用于验证电子邮件地址的正则表达式：

```js
function validateEmail(email) {
  const emailRegex = /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\ [\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
  return emailRegex.test(email);
}
```

不要被正则表达式所压倒，它是互联网上常见的东西。每当您需要常见格式的正则表达式，比如电子邮件地址或电话号码，您都可以在互联网上找到它们。要验证手机号码，请执行以下操作：

```js
function validatePhone(phone) {
  const phoneRegex = /^\(?([0-9]{3})\)?[-. ]?([0-9]{3})[-. ]?([0-9]{4})$/;
  return phoneRegex.test(phone);
}
```

这将验证电话号码是否符合`XXX-XXX-XXXX`的格式（此格式在表单的占位符中给出）。

如果您的要求非常具体，您将不得不编写自己的正则表达式。那时，请参考以下页面：[`developer.mozilla.org/en/docs/Web/JavaScript/Guide/Regular_Expressions`](https://developer.mozilla.org/en/docs/Web/JavaScript/Guide/Regular_Expressions)。

电子邮件地址在表单中默认验证，因为电子邮件输入字段的类型属性设置为电子邮件。但是，有必要在 JavaScript 中验证它，因为并非所有浏览器都可能支持此属性，而且 HTML 可以很容易地从 Chrome DevTools 进行编辑。其他字段也是一样。

要验证年龄，假设用户应该在 10-25 岁的年龄组中：

```js
function validateAge(age) {
  return age >= 10 && age <= 25;
}
```

要验证职业，职业的接受值为`school`、`college`、`trainee`和`employee`。它们是`index.html`文件中职业选择字段的`<option>`元素的值。要验证`profession`，请执行以下操作：

```js
function validateProfession(profession) {
  const acceptedValues = ['school','college','trainee','employee'];
  return acceptedValues.indexOf(profession) > -1;
}
```

JavaScript 数组有一个名为`indexOf()`的方法。它接受一个数组元素作为参数，并返回该元素在数组中的索引。但是，如果数组中不存在该元素，则返回`-1`。我们可以使用这个函数来检查职业的值是否是接受的值之一，方法是找到它在数组中的索引，并检查索引是否大于`-1`。

最后，要验证经验，经验单选按钮的值为 1、2 和 3。因此，经验应该是 0-4 之间的数字：

```js
function validateExperience(experience) {
  return experience > 0 && experience < 4;
}
```

由于评论字段是可选的，我们不需要为该字段编写验证逻辑。现在，在我们最初创建的`validateRegistrationForm()`函数中，添加以下代码：

```js
export default function validateRegistrationForm(formValues) {

  const result = {
    username: validateUserName(formValues.username),
    email: validateEmail(formValues.email),
    phone: validatePhone(formValues.phone),
    age: validateAge(formValues.age),
    profession: validateProfession(formValues.profession),
    experience: validateExperience(formValues.experience),
  };

}
```

现在，结果对象包含每个表单输入的验证状态（`true`/`false`）。检查整个表单是否有效。只有当结果对象的所有属性都为`true`时，表单才有效。要检查结果对象的所有属性是否都为`true`，我们需要使用`for`/`in`循环。

`for`/`in`循环遍历对象的属性。由于`result`对象的所有属性都需要为`true`，因此创建一个初始值为`true`的变量`isValid`。现在，遍历`result`对象的所有属性，并将值与`isValid`变量进行逻辑与（`&&`）操作：

```js
let field, isValid = true;
for(field in result) {
  isValid = isValid && result[field];
}
```

通常，您可以使用点符号（`.`）访问对象的属性。但是，由于我们使用了`for`/`in`循环，属性名称存储在变量`field`中。在这种情况下，如果`field`包含值`age`，我们需要使用方括号表示法`result[field]`来访问属性；这相当于点表示法中的`result.age`。

只有当结果对象的所有属性都为`true`时，`isValid`变量才为`true`。这样，我们既有表单的验证状态，又有各个字段的状态。`validateRegistrationForm()`函数将作为另一个对象的属性返回`isValid`变量和`result`对象：

```js
export default function validateRegistrationForm(formValues) {
  ...
  ...
  return { isValid, result };
}
```

我们在这里使用了 ES6 的对象字面量属性值简写特性。我们的表单验证模块已经准备好了！我们可以将这个模块导入到我们的`home.js`文件中，并在事件注册应用程序中使用它。

在你的`home.js`文件中，在`Home`类之前，添加以下行：

```js
import validateRegistrationForm from './services/formValidation/validateRegistrationForm';
```

然后，在`Home`类的`onFormSubmit()`方法中，添加以下代码：

```js
onFormSubmit(event) {
  event.preventDefault();

  const formValues = this.getFormValues();
  const formStatus = validateRegistrationForm(formValues);

  if(formStatus.isValid) {
    this.clearErrors();
    this.submitForm(formValues);
  } else {
    this.clearErrors();
    this.highlightErrors(formStatus.result);
  }
}
```

上述代码执行以下操作：

+   它调用我们之前创建的`validateRegistrationForm()`模块，并将`formValues`作为其参数，并将返回的值存储在`formStatus`对象中。

+   首先，它检查整个表单是否有效，使用`formStatus.isValid`的值。

+   如果为`true`，则调用`clearErrors()`方法清除 UI（我们的 HTML 表单）中的所有错误高亮，并调用另一个方法`submitForm()`提交表单。

+   如果为`false`（表单无效），则调用`clearErrors()`方法清除表单，然后使用`formStatus.result`调用`highlightErrors()`方法，该方法作为参数包含各个字段的验证详细信息，以突出显示具有错误的字段。

我们需要在`Home`类中创建在上述代码中调用的方法，因为它们是`Home`类的方法。`clearErrors()`和`highlightErrors()`方法的工作很简单。`clearErrors`只是从输入字段的父`<div>`中移除`.has-error`类。而`highlightError`如果输入字段未通过验证（字段的结果为`false`），则将`.has-error`类添加到父`<div>`中。

`clearErrors()`方法的代码如下：

```js
clearErrors() {
  this.$username.parentElement.classList.remove('has-error');
  this.$phone.parentElement.classList.remove('has-error');
  this.$email.parentElement.classList.remove('has-error');
  this.$age.parentElement.classList.remove('has-error');
  this.$profession.parentElement.classList.remove('has-error');
  this.$experience.parentElement.classList.remove('has-error');
}
```

`highlightErrors()`方法的代码如下：

```js
highlightErrors(result) {
  if(!result.username) {
    this.$username.parentElement.classList.add('has-error');
  }
  if(!result.phone) {
    this.$phone.parentElement.classList.add('has-error');
  }
  if(!result.email) {
    this.$email.parentElement.classList.add('has-error');
  }
  if(!result.age) {
    this.$age.parentElement.classList.add('has-error');
  }
  if(!result.profession) {
    this.$profession.parentElement.classList.add('has-error');
  }
  if(!result.experience) {
    this.$experience.parentElement.classList.add('has-error');
  }
}
```

目前，将`submitForm()`方法留空：

```js
submitForm(formValues) {
}
```

在浏览器中打开表单（希望您保持 Webpack 开发服务器运行）。尝试在输入字段中输入一些值，然后单击提交。如果输入了有效的输入值，它不应执行任何操作。如果输入了无效的输入条目（根据我们的验证逻辑），则输入字段将以红色边框突出显示，因为我们向字段的父元素添加了`.has-error` Bootstrap 类。如果您更正了具有有效值的字段，然后再次单击提交，错误应该消失，因为我们使用了`clearErrors()`方法来清除所有旧的错误高亮。

# 使用 AJAX 提交表单

现在我们进入表单部分的第二部分，提交表单。我们已经禁用了表单的默认提交行为，现在需要实现一个用于提交逻辑的 AJAX 表单。

AJAX 是**异步 JavaScript 和 XML**（**AJAX**）的缩写。它不是一个编程工具，而是一个概念，通过它你可以发出网络请求，从服务器获取数据，并更新网站的某些部分，而无需重新加载整个页面。

异步 JavaScript 和 XML 这个名字可能听起来有点困惑，但最初 XML 被广泛用于与服务器交换数据。我们也可以使用 JSON/普通文本与服务器交换数据。

为了将表单提交到服务器，我创建了一个小的 Node.js 服务器（使用 express 框架构建），假装保存你的表单详情并返回一个成功消息。服务器在代码文件的`Chapter03`文件夹中。要启动服务器，只需在服务器目录中运行`npm install`，然后运行`npm start`命令。这将在`http://localhost:3000/`URL 上启动服务器。如果你在浏览器中打开这个 URL，你会看到一个空白页面，上面显示着消息 Cannot GET /;这意味着服务器正常运行。

服务器有两个 API 端点，我们需要与其中一个通信以发送用户的详情。这就是注册 API 端点的工作方式：

```js
Route: /registration,
Method: POST,
Body: the form data in JSON format
{
  "username":"Test User",
  "email":"mail@test.com",
  "phone":"123-456-7890",
  "age":"16",
  "profession":"school",
  "experience":"1",
  "comment":"Some comment from user"
} If registration is success:
status code: 200
response: { "message": "Test User is Registered Successfully" }
```

在真实的 JavaScript 应用中，你将不得不处理很多像这样的网络请求。大部分用户操作都会触发需要服务器处理的 API 调用。在我们的场景中，我们需要调用前面的 API 来注册用户。

让我们来规划一下 API 调用应该如何工作：

+   正如其名称所示，这个事件将是异步的。我们需要使用 ES6 的一个新概念，叫做 Promises，来处理这个 API 调用。

+   在下一节中，我们将有另一个 API 调用。最好将 API 调用创建为类似模块验证模块的形式。

+   我们必须验证服务器是否成功注册了用户。

+   由于整个 API 调用会花费一些时间，我们应该在过程中向用户显示一个加载指示器。

+   最后，如果注册成功，我们应该立即通知用户并清空表单。

# 在 JavaScript 中进行网络请求

JavaScript 有`XMLHttpRequest`用于进行 AJAX 网络请求。ES6 引入了一个叫做 fetch 的新规范，它通过 Promises 支持使得处理网络请求更加现代和高效。除了这两种方法，jQuery 还有`$.ajax()`方法，广泛用于进行网络请求。`Axios.js`是另一个广泛用于进行网络请求的`npm`包。

我们将在我们的应用中使用 fetch 进行网络请求。

Fetch 在 Internet Explorer 中不起作用，需要使用 polyfills。查看：[`caniuse.com/`](https://caniuse.com/)来了解任何你想使用的新的`HTML/CSS/Javascript`组件的浏览器兼容性。

# 什么是 Promise？

到现在为止，你可能会想知道我所说的 Promise 是什么？嗯，Promise，顾名思义，是 JavaScript 做出的一个承诺，即异步函数将在某个时刻完成执行。

在上一章中，我们遇到了一个异步事件：使用`FileReader`读取文件内容。这就是`FileReader`的工作方式：

+   它开始读取文件。由于读取是一个异步事件，其他 JavaScript 代码在读取仍在进行时会继续执行。

你可能会想，*如果我需要在事件完成后执行一些代码怎么办？*这就是`FileReader`处理的方式：

+   一旦读取完成，`FileReader`会触发一个`load`事件。

+   它还有一个`onload()`方法来监听`load`事件，当`load`事件被触发时，`onload()`方法将开始执行。

+   因此，我们需要将我们需要的代码放在`onload()`方法中，它只会在`FileReader`完成读取文件内容后执行。

这可能看起来是处理异步事件的更简单方式，但想象一下如果有多个需要依次发生的异步事件！你将不得不触发多少事件，需要跟踪多少事件监听器？这将导致非常难以理解的代码。此外，JavaScript 中的事件监听器是昂贵的资源（它们消耗大量内存），必须尽量减少。

回调函数经常用于处理异步事件。但是，如果有很多异步函数依次发生，您的代码将看起来像这样：

```js
asyncOne('one', () => {
  ...
  asyncTwo('two', () => {
    ...
    asyncThree('three', () => {
      ...
      asyncFour('four', () => {
      });
    });
  });
});
```

在编写了很多回调之后，您的闭合括号将被排列成金字塔形。这被称为回调地狱。回调地狱很混乱，构建应用程序时应该避免。因此，回调在这里没有用处。

进入 Promises，一种处理异步事件的新方法。这是 JavaScript `Promise`的工作方式：

```js
new Promise((resolve, reject) => {
  // Some asynchronous logic
  resolve(5);
});
```

`Promise`构造函数创建一个具有两个参数的函数，resolve 和 reject，它们都是函数。然后，`Promise`只有在调用 resolve 或 reject 时才会返回值。当异步代码成功执行时，调用 resolve，当发生错误时调用 reject。在这里，`Promise`在异步逻辑执行时返回一个值`5`。

假设您有一个名为`theAsyncCode()`的函数，它执行一些异步操作。您还有另一个函数`onlyAfterAsync()`，它需要严格在`theAsyncCode()`之后运行，并使用`theAsyncCode()`返回的值。

以下是如何使用 Promises 处理这两个函数：

```js
function theAsyncCode() {
  return new Promise((resolve, reject) => {
    console.log('The Async Code executed!');
    resolve(5);
  });
}
```

首先，`theAsyncCode()`应该返回一个`Promise`而不是一个值。您的异步代码应该写在那个`Promise`里。然后，您编写`onlyAfterAsync()`函数：

```js
function onlyAfterAsync(result) {
  console.log('Now onlyAfterAsync is executing...');
  console.log(`Final result of execution - ${result}`);
}
```

要依次执行前面的函数，我们需要使用`Promise.then().catch()`语句将它们链接起来。在这里，`Promise`由`theAsyncCode()`函数返回。因此，代码应该是：

```js
theAsyncCode()
.then(result => onlyAfterAsync(result))
.catch(error => console.error(error))
```

当`theAsyncCode()`执行`resolve(5)`时，`then`方法会自动以解析值作为其参数调用。现在我们可以在`then`方法中执行`onlyAfterAsync()`方法。如果`theAsyncCode()`执行的是`reject('an error')`而不是`resolve(5)`，它将触发`catch`方法而不是`then`。

如果您有另一个函数`theAsyncCode2()`，它使用`theAsyncCode()`返回的数据，那么它应该在`onlyAfterAsync()`函数之前执行：

```js
function theAsyncCode2(data) {
  return new Promise((resolve, reject) => {
    console.log('The Async Code 2 executed');
    resolve(data);
  });
}
```

您只需要更新您的`.then().catch()`链，如下所示：

```js
theAsyncCode()
.then(data => theAsyncCode2(data))
.then(result => onlyAfterAsync(result))
.catch(error => console.error(error));
```

这样，所有三个函数将依次执行。如果`theAsyncCode()`或`theAsyncCode2()`中的任何一个返回`reject()`，那么将调用`catch`语句。

如果我们只需要使用链中前一个函数的解析值作为参数调用函数，我们可以进一步简化链：

```js
theAsyncCode()
.then(theAsyncCode2)
.then(onlyAfterAsync)
.catch(console.error);
```

这将得到相同的结果。我在[`jsfiddle.net/jjq60Ly6/4/`](https://jsfiddle.net/jjq60Ly6/4/)上设置了一个小的 JS fiddle，您可以在那里体验 Promises 的工作。访问 JS fiddle，打开 Chrome DevTools 控制台，然后单击 JS fiddle 页面左上角的 Run。您应该看到按顺序从三个函数中打印出`console.log`语句。随意编辑 fiddle 并尝试使用 Promises 进行实验。

在完成本章后不久，ES8 被宣布，确认了`async`函数是 JavaScript 语言的一部分。ES8 的`async`和`await`关键字提供了一种更简单的方式来解决 Promise，而不是 ES6 中使用的`.then().catch()`链。要学习使用`async`函数，请访问以下 MDN 页面：[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)。

# 创建 API 调用模块

我们将使用 POST API 调用来注册我们的用户。但是，在应用程序的状态部分，我们需要使用`GET`请求来显示对活动感兴趣的人的统计数据。因此，我们将构建一个通用的 API 调用模块。

要创建 API 调用模块，在`services`目录内，创建另一个名为`api`的目录，并在其中创建`apiCall.js`。您的`services`目录的结构应如下所示：

```js
.
├── api
│   └── apiCall.js
└── formValidation
    └── validateRegistrationForm.js
```

在`apiCall.js`文件中创建以下函数：

```js
export default function apiCall(route, body = {}, method='GET') {
}
```

在前面的函数中，路由是一个必需的参数，而`body`和`method`有其默认值。这意味着它们是可选的。如果您只使用一个参数调用该函数，则另外两个参数将使用它们的默认值：

```js
apiCall('/registration) // values of body = {} and method = 'GET' 
```

如果您使用所有三个参数调用该函数，它将像普通函数一样工作：

```js
apiCall('/registration', {'a': 5}, 'POST'); // values of body = {'a': 5} and method = 'POST'
```

默认参数仅在 ES6 中引入。我们使用默认参数是因为`GET`请求不需要`body`属性。它只将数据作为查询参数发送到 URL 中。

我们已经在默认表单的提交部分看到了`GET`和`POST`请求的工作原理。让我们构建一个`apiCall`函数，可以执行`GET`和`POST`请求：

在`apiCall`函数内，创建一个名为`request`的新`Promise`对象：

```js
export default function apiCall(route, body = {}, method='GET') {

  const request = new Promise((resolve, reject) => {
    // Code for fetch will be written here
  });

}
```

fetch API 接受两个参数作为输入，并返回`Promise`，当网络请求完成时解析。第一个参数是请求 URL，第二个参数包含有关请求的信息的对象，如`headers`，`cors`，`method`，`body`等。

# 构建请求详细信息

将以下代码写入请求`Promise`内。首先，因为我们正在处理 JSON 数据，我们需要创建一个带有内容类型`application/json`的标头。我们可以使用`Headers`构造函数来实现这一目的：

```js
const headers = new Headers({
  'Content-Type': 'application/json',
});
```

现在，使用之前创建的`headers`和参数中的`method`变量，我们创建`requestDetails`对象：

```js
const requestDetails = {
  method,
  mode: 'cors',
  headers,
};
```

请注意，我已在`requestDetails`中包含了`mode: 'cors'`。**跨域资源共享**（**CORS**）允许服务器安全地进行跨域数据传输。假设您有一个运行在`www.mysite.org`上的网站。您需要向在`www.anothersite.org`上运行的另一个服务器发出 API 调用（网络请求）。

然后，这是一个跨域请求。要进行跨域请求，`www.anothersite.org`上的服务器必须设置`Access-Control-Allow-Origin`标头以允许`www.mysite.org`。否则，浏览器将阻止跨域请求，以防止未经授权访问另一个服务器。来自`www.mysite.org`的请求还应在其请求详细信息中包含`mode: 'cors'`。

在我们的事件注册应用程序中，Webpack 开发服务器正在`http://localhost:8080/`上运行，而 Node.js 服务器正在`http://localhost:3000/`上运行。因此，这是一个跨域请求。我已经启用了`Access-Control-Allow-Origin`并设置了`Access-Control-Allow-Headers`，以便它不会对`apiCall`函数造成任何问题。

有关 CORS 请求的详细信息可以在以下 MDN 页面找到：[`developer.mozilla.org/en/docs/Web/HTTP/Access_control_CORS`](https://developer.mozilla.org/en/docs/Web/HTTP/Access_control_CORS)。

我们的`requestDetails`对象还应包括请求的`body`。但是，`body`应仅包括在`POST`请求中。因此，可以在`requestDetails`对象声明下面编写，如下所示：

```js
if(method !== 'GET') requestDetails.body = JSON.stringify(body);
```

这将为`POST`请求添加`body`属性。要进行 fetch 请求，我们需要构建请求 URL。我们已经设置了环境变量`SERVER_URL=http://localhost:3000`，Webpack 将其转换为全局变量`SERVER_URL`，可在 JavaScript 代码的任何地方访问。路由传递给`apiCall()`函数的参数。fetch 请求可以构建如下：

```js
function handleErrors(response) {
  if(response.ok) {
    return response.json();
  } else {
    throw Error(response.statusText);
  }
}

fetch(`${SERVER_URL}/${route}`, requestDetails)
  .then(response => handleErrors(response))
  .then(data => resolve(data))
  .catch(err => reject(err));
```

`handleErrors` 函数的作用是什么？它将检查服务器返回的响应是否成功（`response.ok`）。如果是，它将解码响应并返回它（`response.json()`）。否则，它将抛出一个错误。

我们可以使用我们之前讨论的方法进一步简化 Promise 链：

```js
fetch(`${SERVER_URL}/${route}`, requestDetails)
  .then(handleErrors)
  .then(resolve)
  .catch(reject);
```

Fetch 有一个小问题。它无法自行处理超时。想象一下服务器遇到问题，无法返回请求。在这种情况下，fetch 将永远不会解决。为了避免这种情况，我们需要做一些变通。在 `request` Promise 之后，创建另一个名为 `timeout` 的 `Promise`：

```js
const request = new Promise((resolve, reject) => {
....
});

const timeout = new Promise((request, reject) => {
  setTimeout(reject, timeoutDuration, `Request timed out!`);
});
```

在 `apiCall.js` 文件的 `apicall()` 函数之外创建一个名为 `timeoutDuration` 的常量，如下所示：

```js
const  timeoutDuration = 5000;
```

将此常量放在文件顶部，以便我们可以在将来轻松更改超时持续时间（更易于代码维护）。`timeout` 是一个简单的 Promise，它在 5 秒后自动拒绝（来自 `timeoutDuration` 常量）。我已经创建了服务器，以便在 3 秒后响应。

现在，JavaScript 有一种很酷的方法来解决多个 Promises，即 `Promise.race()` 方法。正如其名字所示，这将使两个 Promises 同时运行，并接受首先解决/拒绝的那个的值。这样，如果服务器在 3 秒内没有响应，5 秒后就会发生超时，`apiCall` 将被拒绝并显示超时！为此，在 `apiCall()` 函数中的 `request` 和 `timeout` Promises 之后添加以下代码：

```js
return new Promise((resolve, reject) => {
  Promise.race([request, timeout])
    .then(resolve)
    .catch(reject);
});
```

`apiCall()` 函数作为一个整体返回一个 Promise，该 Promise 是 `request` 或 `timeout` Promise 的解决值（取决于它们中哪一个更快执行）。就是这样！我们的 `apiCall` 模块现在已经准备好在我们的事件注册应用程序中使用。

如果您觉得 `apiCall` 函数难以理解和跟踪，请再次阅读 `Chapter03` 完整代码文件中的 `apiCall.js` 文件，以便更简单地解释。要详细了解 Promise 并带有更多示例，请阅读以下 Google Developers 页面：[`developers.google.com/web/fundamentals/getting-started/primers/promises`](https://developers.google.com/web/fundamentals/getting-started/primers/promises) 和 MDN 页面：[`developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise)。

# 其他网络请求方法

点击这些链接了解 JavaScript 中进行网络请求的其他插件/API：

+   jQuery，`$.ajax()` 方法：[`api.jquery.com/jquery.ajax/`](http://api.jquery.com/jquery.ajax/)

+   `XMLHttpRequest`: [`developer.mozilla.org/en/docs/Web/API/XMLHttpRequest`](https://developer.mozilla.org/en/docs/Web/API/XMLHttpRequest)

+   Axios.js: [`github.com/mzabriskie/axios`](https://github.com/mzabriskie/axios)

要使 fetch 在 Internet Explorer 中工作，请阅读以下页面，了解如何为 fetch 添加 `polyfill`：[`github.com/github/fetch/`](https://github.com/github/fetch/)。

# 回到表单

开始提交的第一步是隐藏提交按钮并用加载指示器替换它。这样，用户就不会意外地点击两次提交。此外，加载指示器还表示后台正在进行某个过程。在 `home.js` 文件的 `submitForm()` 方法中，添加以下代码：

```js
submitForm(formValues) {
  this.$submit.classList.add('hidden');
  this.$loadingIndicator.classList.remove('hidden');
}
```

这将隐藏提交按钮并显示加载指示器。要进行 `apiCall`，我们需要导入 `apiCall` 函数并通知用户请求已完成。我在 `package.json` 文件中添加了一个名为 `toastr` 的包。当您运行 `npm install` 命令时，它应该已经安装。

在 `home.js` 文件的顶部，添加以下导入语句：

```js
import apiCall from './services/api/apiCall';
import toastr from 'toastr';
import '../../node_modules/toastr/toastr.less';
```

这将导入`toastr`及其样式文件（`toastr.less`），以及最近创建的`apiCall`模块。现在，在`submitForm()`方法中，添加以下代码：

```js
apiCall('registration', formValues, 'POST')
  .then(response => {
    this.$submit.classList.remove('hidden');
    this.$loadingIndicator.classList.add('hidden');
    toastr.success(response.message);
    this.resetForm(); // For clearing the form
  })
  .catch(() => {
    this.$submit.classList.remove('hidden');
    this.$loadingIndicator.classList.add('hidden');
    toastr.error('Error!');
  });
```

由于`apiCall()`返回一个 Promise，我们在这里使用`Promise.then().catch()`链。当注册成功时，`toastr`将在页面的右上角显示一个成功的提示，其中包含服务器发送的消息。如果出现问题，它将简单地显示一个错误提示。此外，我们需要使用`this.resetForm()`方法清除表单。在`Home`类中添加`resetForm()`方法，代码如下：

```js
resetForm() {
  this.$username.value = '';
  this.$email.value = '';
  this.$phone.value = '';
  this.$age.value = '';
  this.$profession.value = 'school';
  this.$experience.checked = true;
  this.$comment.value = '';
}
```

在 Chrome 中返回到活动注册页面，尝试提交表单。如果所有值都有效，它应该成功提交表单并显示成功的提示消息，表单值将被重置为初始值。在现实世界的应用中，服务器将向用户发送确认邮件。然而，服务器端编码超出了本书的范围。但我想在下一章中稍微解释一下这个。

尝试关闭 Node.js 服务器并提交表单。它应该会抛出一个错误。在学习 JavaScript 的一些高级概念的同时，您已经成功完成了构建您的活动注册表单。现在，让我们继续进行我们应用程序的第二页——状态页面，我们需要显示一个注册用户统计图表。

# 使用 Chart.js 向网站添加图表

我们刚刚为用户创建了一个不错的注册表单。现在是时候处理我们活动注册应用程序的第二部分了。状态页面显示了一个图表，显示了对活动感兴趣的人数，根据经验、职业和年龄。如果现在打开状态页面，它应该显示一个数据加载中...的消息和加载指示器图像。但我已经在`status.html`文件中构建了所有这个页面所需的组件。它们都使用 Bootstrap 的`.hidden`类当前隐藏。

让我们看看`status.html`文件中有什么。尝试从以下每个部分中删除`.hidden`类，看看它们在 Web 应用程序中的外观。

首先是加载指示器部分，它目前显示在页面上：

```js
<div id="loadingIndicator">
  <p>Data loading...</p>
  <image src="./src/assets/images/loading.gif" class="loading-indicator"></image>
</div>
```

接下来是一个包含错误消息的部分，当 API 调用失败时显示：

```js
<div id="loadingError" class="hidden">
  <h3>Unable to load data...Try refreshing the page.</h3>
</div>
```

在前面的部分之后，我们有一个选项卡部分，它将为用户提供在不同图表之间切换的选项。代码如下所示：

```js
<ul class="nav nav-tabs hidden" id="tabArea">
  <li role="presentation" class="active"><a href="" id="experienceTab">Experience</a></li>
  <li role="presentation"><a href="" id="professionTab">Profession</a></li>
  <li role="presentation"><a href="" id="ageTab">Age</a></li>
</ul>
```

选项卡只是一个带有`.nav`和`.nav-tabs`类的无序列表，由 Bootstrap 样式为选项卡。选项卡部分是带有`.active`类的列表项，用于突出显示所选的选项卡部分（`role="presentation"`用于辅助选项）。在列表项内，有一个空的`href`属性的锚标签。

最后，我们有我们的图表区域，有三个画布元素，用于显示前面选项卡中提到的三个不同类别的图表：

```js
<div class="chart-area hidden" id="chartArea">
  <canvas id="experienceChart"></canvas>
  <canvas id="professionChart"></canvas>
  <canvas id="ageChart"></canvas>
</div>
```

正如我们在上一章中看到的，画布元素最适合在网页上显示图形，因为编辑 DOM 元素是一项昂贵的操作。Chart.js 使用画布元素来显示给定数据的图表。让我们制定状态页面应该如何工作的策略：

+   在从服务器获取统计数据的 API 调用时，应该显示加载指示器

+   如果数据成功检索，则加载指示器应该被隐藏，选项卡部分和图表区域应该变得可见

+   只有与所选选项卡对应的画布应该可见；其他画布元素应该被隐藏

+   应该使用 Chart.js 插件向画布添加饼图

+   如果数据检索失败，则所有部分应该被隐藏，错误部分应该被显示

好了！让我们开始工作。打开我已经在`status.html`中添加为参考的`status.js`文件。创建一个`Status`类，并在其构造函数中引用所有所需的 DOM 元素，如下所示：

```js
class Status {
  constructor() {
    this.$experienceTab = document.querySelector('#experienceTab');
    this.$professionTab = document.querySelector('#professionTab');
    this.$ageTab = document.querySelector('#ageTab');

    this.$ageCanvas = document.querySelector('#ageChart');
    this.$professionCanvas = document.querySelector('#professionChart');
    this.$experienceCanvas = document.querySelector('#experienceChart');

    this.$loadingIndicator = document.querySelector('#loadingIndicator');
    this.$tabArea = document.querySelector('#tabArea');
    this.$chartArea = document.querySelector('#chartArea');

    this.$errorMessage = document.querySelector('#loadingError');

    this.statisticData; // variable to store data from the server
 }

}
```

我还创建了一个类变量`statisticData`，用于存储从 API 调用中检索到的数据。此外，在页面加载时添加创建类的实例的代码：

```js
window.addEventListener("load", () => {
  new Status();
});
```

我们状态页面的第一步是向服务器发出网络请求，以获取所需的数据。我已在 Node.js 服务器中创建了以下 API 端点：

```js
Route: /statistics,
Method: GET,  Server Response on Success:
status code: 200
response: {"experience":[35,40,25],"profession":[30,40,20,10],"age":[30,60,10]}
```

服务器将以适合与 Chart.js 一起使用的格式返回包含基于其经验、职业和年龄感兴趣的人数的数据。让我们使用之前构建的`apiCall`模块来进行网络请求。在您的`status.js`文件中，首先在`Status`类上面添加以下导入语句：

```js
import apiCall from './services/api/apiCall';
```

之后，在`Status`类中添加以下方法：

```js
loadData() {
  apiCall('statistics')
    .then(response => {
      this.statisticData = response;

      this.$loadingIndicator.classList.add('hidden');
      this.$tabArea.classList.remove('hidden');
      this.$chartArea.classList.remove('hidden');
    })
    .catch(() => {
      this.$loadingIndicator.classList.add('hidden');
      this.$errorMessage.classList.remove('hidden');
    });
}
```

这次，我们可以只使用一个参数调用`apiCall()`函数，因为我们正在进行`GET`请求，并且我们已经将`apiCall()`函数的默认参数定义为`body = {}`和`method = 'GET'`。这样，我们在进行`GET`请求时就不必指定 body 和 method 参数。在您的构造函数中，添加`this.loadData()`方法，这样当页面加载时它将自动进行网络请求：

```js
constructor() {
  ...
  this.loadData();
}
```

现在，在 Chrome 中查看网页。三秒后，应该显示选项卡。目前，单击选项卡只会重新加载页面。我们将在创建图表后处理这个问题。

# 将图表添加到画布元素

我们的类变量`statisticData`中有所需的数据，应该用它来渲染图表。我已经在`package.json`文件中添加了 Chart.js 作为项目依赖项，当您执行`npm install`命令时，它应该已经安装。让我们通过在`status.js`文件顶部添加以下代码来将 Chart.js 导入我们的项目中：

```js
import Chart from 'chart.js';
```

不一定要在文件顶部添加`import`语句。但是，在顶部添加`import`语句可以清晰地看到当前文件中模块的所有依赖关系。

Chart.js 提供了一个构造函数，我们可以使用它来创建一个新的图表。`Chart`构造函数的语法如下：

```js
new Chart($canvas, {type: 'pie', data});
```

`Chart`构造函数的第一个参数应该是对 canvas 元素的引用，第二个参数是具有两个属性的 JSON 对象：

+   `type`属性应该包含我们在项目中需要使用的图表类型。我们需要在项目中使用饼图。

+   `data`属性应该包含作为基于图表类型的格式的对象所需的数据集。在我们的情况下，对于饼图，所需的格式在 Chart.js 文档的以下页面上指定：[`www.chartjs.org/docs/latest/charts/doughnut.html`](http://www.chartjs.org/docs/latest/charts/doughnut.html)。

数据对象将具有以下格式：

```js
{
  datasets: [{
    data: [],
    backgroundColor: [],
    borderColor: [],
  }],
  labels: []
}
```

数据对象具有以下属性：

+   一个`datasets`属性，其中包含另一个对象的数组，该对象具有`data`、`backgroundColor`和`borderColor`作为数组

+   `labels`属性是一个标签数组，顺序与数据数组相同

创建的图表将自动占据其父元素提供的整个空间。在`Status`类内部创建以下函数，将`Chart`加载到状态页面中：

您可以根据经验创建一个图表，如下所示：

```js
loadExperience() {
  const data = {
    datasets: [{
      data: this.statisticData.experience,
      backgroundColor:[
        'rgba(255, 99, 132, 0.6)',
        'rgba(54, 162, 235, 0.6)',
        'rgba(255, 206, 86, 0.6)',
      ],
      borderColor: [
        'white',
        'white',
        'white',
      ]
    }],
    labels: [
      'Beginner',
      'Intermediate',
      'Advanced'
    ]
  };
  new Chart(this.$experienceCanvas,{
    type: 'pie',
    data,
  });
}
```

您可以根据职业创建一个图表，如下所示：

```js
loadProfession() {
  const data = {
    datasets: [{
      data: this.statisticData.profession,
      backgroundColor:[
        'rgba(255, 99, 132, 0.6)',
        'rgba(54, 162, 235, 0.6)',
        'rgba(255, 206, 86, 0.6)',
        'rgba(75, 192, 192, 0.6)',
      ],
      borderColor: [
        'white',
        'white',
        'white',
        'white',
      ]
    }],
    labels: [
      'School Students',
      'College Students',
      'Trainees',
      'Employees'
    ]
  };
  new Chart(this.$professionCanvas,{
    type: 'pie',
    data,
  });
}
```

您可以根据年龄创建一个图表，如下所示：

```js
loadAge() {
  const data = {
    datasets: [{
      data: this.statisticData.age,
      backgroundColor:[
        'rgba(255, 99, 132, 0.6)',
        'rgba(54, 162, 235, 0.6)',
        'rgba(255, 206, 86, 0.6)',
      ],
      borderColor: [
        'white',
        'white',
        'white',
      ]
    }],
    labels: [
      '10-15 years',
      '15-20 years',
      '20-25 years'
    ]
  };
  new Chart(this.$ageCanvas,{
    type: 'pie',
    data,
  });
}
```

这些函数应在数据加载到`statisticData`变量中时调用。因此，在 API 调用成功后调用它们的最佳位置是在`loadData()`方法中添加以下代码，如下所示：

```js
loadData() {
  apiCall('statistics')
    .then(response => {
      ...
      this.loadAge();
      this.loadExperience();
      this.loadProfession();
     })
...
}
```

现在，在 Chrome 中打开状态页面。您应该看到页面上呈现了三个图表。图表已经占据了其父元素的整个宽度。要减小它们的大小，请在您的`styles.css`文件中添加以下样式：

```js
.chart-area {
  margin: 25px;
  max-width: 600px;
}
```

这将减小图表的尺寸。Chart.js 最好的部分是它默认是响应式的。尝试在 Chrome 的响应式设计模式下调整页面大小。当页面的高度和宽度改变时，你应该看到图表被重新调整大小。我们现在在我们的状态页面上添加了三个图表。

对于我们的最后一步，我们需要选项卡来切换图表的外观，以便一次只有一个图表可见。

# 设置选项卡部分

选项卡应该工作，以便在任何给定时间只有一个图表可见。此外，所选选项卡应使用 `.active` 类标记为活动状态。这个问题的一个简单解决方案是隐藏所有图表，从所有选项卡项目中移除 `.active`，然后只向点击的选项卡项目添加 `.active` 并显示所需的图表。这样，我们可以轻松获得所需的选项卡功能。

首先，在 `Status` 类中创建一个方法来清除选定的选项卡并隐藏所有图表：

```js
hideCharts() {
  this.$experienceTab.parentElement.classList.remove('active');
  this.$professionTab.parentElement.classList.remove('active');
  this.$ageTab.parentElement.classList.remove('active');
  this.$ageCanvas.classList.add('hidden');
  this.$professionCanvas.classList.add('hidden');
  this.$experienceCanvas.classList.add('hidden');
}
```

创建一个方法来为点击的选项卡项目添加事件监听器：

```js
addEventListeners() {
  this.$experienceTab.addEventListener('click', this.loadExperience.bind(this));
  this.$professionTab.addEventListener('click', this.loadProfession.bind(this));
  this.$ageTab.addEventListener('click', this.loadAge.bind(this));
}
```

还要在 `constructor` 中使用 `this.addEventListeners();` 调用前面的方法，以便在页面加载时附加事件监听器。

每当我们点击选项卡项目中的一个时，它将调用相应的加载图表函数。比如我们点击了 Experience 选项卡。这将使用 `event` 作为参数调用 `loadExperience()` 方法。但是我们可能希望在 API 调用后调用此函数以加载图表，而不带有事件参数。为了使 `loadExperience()` 在两种情况下都能工作，修改该方法如下：

```js
loadExperience(event = null) {
  if(event) event.preventDefault();
  this.hideCharts();
  this.$experienceCanvas.classList.remove('hidden');
  this.$experienceTab.parentElement.classList.add('active');

  const data = {...}
  ...
}
```

在前面的函数中：

+   事件参数被定义为默认值 `null`。如果使用事件参数调用 `loadExperience()`（当用户点击选项卡时），`if(event)` 条件将通过，`event.preventDefault()` 将停止锚标签的默认点击操作。这将防止页面重新加载。

+   如果从 `apiCall` 的 promise 链中调用 `this.loadExperience()`，它将不具有 `event` 参数，事件的值默认为 `null`。`if(event)` 条件将失败（因为 `null` 是一个假值），`event.preventDefault()` 将不会被执行。这将防止异常，因为在这种情况下 `event` 未定义。

+   之后，调用 `this.hideCharts()`，这将隐藏所有图表并从所有选项卡中移除 `.active`。

+   接下来的两行将从经验图表的画布中移除 `.hidden` 并向 Experience 选项卡添加 `.active` 类。

在 `apiCall` 函数的 `then` 链中，移除 `this.loadAge()` 和 `this.loadProfession()`，这样只有经验图表会首先加载（因为它是第一个选项卡）。

如果你在 Google Chrome 中打开并点击 Experience 选项卡，它应该重新渲染图表而不刷新页面。这是因为我们在 `loadExperience()` 方法中添加了 `event.preventDefault()` 来阻止默认操作，并使用 Chart.js 在点击选项卡时渲染图表。

通过在 `loadAge()` 和 `loadProfession()` 中使用相同的逻辑，我们现在可以轻松使选项卡按预期工作。在你的 `loadAge()` 方法中添加以下事件处理代码：

```js
loadAge(event = null) {
  if(event) event.preventDefault();
  this.hideCharts();
  this.$ageCanvas.classList.remove('hidden');
  this.$ageTab.parentElement.classList.add('active');

  const data = {...}
  ...
}
```

同样，在 `loadProfession()` 方法中添加以下代码：

```js
loadProfession(event = null) {
  if(event) event.preventDefault();
  this.hideCharts();
  this.$professionCanvas.classList.remove('hidden');
  this.$professionTab.parentElement.classList.add('active');

  const data = {...}
  ...
}
```

打开 Chrome。点击选项卡以检查它们是否都正常工作。如果是，你已成功完成了状态页面！Chart.js 默认是响应式的；因此，如果你调整页面大小，它将自动调整饼图的大小。现在，还有最后一页，你需要添加谷歌地图来显示事件位置。在普通的 JavaScript 中，添加谷歌地图很简单。但是，在我们的情况下，因为我们使用 Webpack 来捆绑我们的 JavaScript 代码，我们需要在正常流程中添加一个小步骤（谷歌地图需要在 HTML 中访问 JavaScript 变量！）。

Chart.js 有八种类型的图表。请尝试访问：[`www.chartjs.org/`](http://www.chartjs.org/)，如果你正在寻找更高级的图表和图形库，请查看`D3.js`（**数据驱动文档**）：[`d3js.org/`](https://d3js.org/)。

# 在网页中添加谷歌地图

在 VSCode 或文本编辑器中打开`about.html`文件。它将有两个`<p>`标签，你可以在其中添加有关活动的一些信息。之后，将会有一个 ID 为`#map`的`<div>`元素，它应该显示活动在地图中的位置。

我之前已经要求你生成一个 API 密钥来使用谷歌地图。如果你还没有生成，请从以下网址获取：[`developers.google.com/maps/documentation/javascript/get-api-key`](https://developers.google.com/maps/documentation/javascript/get-api-key)，并将其添加到你的`.env`文件的`GMAP_KEY`变量中。根据谷歌地图的文档，要在网页上添加一个带有标记的地图，你必须在页面上包含以下脚本：

```js
<script  async  defer src="https://maps.googleapis.com/maps/api/js?key=API_KEY&callback=**initMap**">
```

在这里，`<script>`标签的`async`和`defer`属性将异步加载脚本，并确保它仅在文档加载后执行。

要了解有关`async`和`defer`的工作原理的更多信息，请参考以下 w3schools 页面。有关 Async: [`www.w3schools.com/tags/att_script_async.asp`](https://www.w3schools.com/tags/att_script_async.asp)，有关 Defer: [`www.w3schools.com/tags/att_script_defer.asp`](https://www.w3schools.com/tags/att_script_defer.asp)。

让我们来看看`src`属性。在这里，有一个 URL，后面跟着两个查询参数，key 和 callback。Key 是你需要包含你的谷歌地图 API 密钥的地方，callback 应该是一个需要在脚本加载完成后执行的函数（脚本是异步加载的）。挑战在于脚本需要包含在我们的 JavaScript 变量不可访问的 HTML 中（我们现在是 Webpack 用户！）。

但是，正如我之前解释的，在`webpack.config.js`文件中，我已经添加了`output.library`属性，它将通过将它们的作用域从`const`或`let`更改为`var`，将使用`export`关键字标记的对象、函数或变量暴露给 HTML（但它们不能直接通过它们的名称访问）。我给出的`output.library`的值是`bundle`。因此，使用`export`关键字标记的东西将作为`bundle`对象的属性可用。

在 Chrome 中打开事件注册应用程序，并打开 Chrome DevTools 控制台。如果你在控制台中输入`bundle`，你会发现它打印出一个空对象。这是因为我们还没有从*Webpack 的入口文件*中进行任何导出（我们在`apiCall.js`和`registrationForm.js`中进行了一些导出，但这些文件不在`webpack.config.js`的入口属性中）。因此，目前我们只有一个空的 bundle 对象。

让我们想一种成功将谷歌地图脚本包含在我们的 Web 应用程序中的方法：

+   API 密钥当前在我们的 JavaScript 代码中作为全局变量`GMAP_KEY`可用。因此，最好是在页面加载完成后从 JavaScript 创建脚本元素并将其附加到 HTML 中。这样，我们就不必导出 API 密钥。

+   对于回调函数，我们将创建一个 JavaScript 函数并导出它。

在 VSCode 中打开`about.js`文件并添加以下代码：

```js
export function initMap() {
}

window.addEventListener("load", () => {
  const $script = document.createElement('script');
  $script.src = `https://maps.googleapis.com/maps/api/js?key=${GMAP_KEY}&callback=bundle.initMap`;
  document.querySelector('body').appendChild($script);
});
```

上述代码执行以下操作：

+   当页面加载完成时，它将创建一个新的脚本元素`document.createElement('script')`并将其存储在`$script`常量对象中。

+   现在，我们将`src`属性添加到`$script`对象中，并将值设置为所需的脚本 URL。请注意，我已经在密钥中包含了`GMAP_KEY`变量，并将`bundle.initMap`作为回调函数（因为我们在`about.js`中导出了`initMap`）。

+   最后，它将把脚本作为子元素附加到 body 元素。这将使 Google Maps 脚本按预期工作。

+   我们这里不需要`async`或`defer`，因为只有在页面加载完成后才加载脚本。

在你的 Chrome DevTools 控制台上，当你在 about 页面上时，尝试再次输入`bundle`。这一次，你应该看到一个打印出`initMap`作为其属性之一的对象。

在我们的 ToDo List 应用中，我们通过直接在模板字符串中编写 HTML 代码来创建 HTML 元素。这对于构建大量 HTML 元素非常有效。然而，对于较小的元素，最好使用`document.createElement()`方法，因为当该元素有很多需要动态值的属性时，这样做会使代码更易读和易懂。

# 添加带有标记的 Google 地图

我们已经成功在页面上包含了 Google Maps 脚本。当 Google Maps 脚本加载完成时，它将调用我们在`about.js`文件中声明的`initMap`函数。现在，我们将使用该函数来创建一个指向 JS Meetup 活动位置的地图标记。

添加 Google 地图标记和更多功能的过程在 Google 地图文档中有很好的解释，可在以下链接找到：[`developers.google.com/maps/documentation/javascript/adding-a-google-map`](https://developers.google.com/maps/documentation/javascript/adding-a-google-map)。

我们之前包含的 Google Maps 脚本为我们提供了一些构造函数，可以创建`map`、`Marker`和`infowindow`。要添加一个带有`marker`的简单 Google 地图，请在`initMap()`函数内添加以下代码：

```js
export function initMap() {
  const map = new google.maps.Map(document.getElementById('map'), {
    zoom: 13,
    center: {lat: 59.325, lng: 18.070}
  });

  const marker = new google.maps.Marker({
    map,
    draggable: true,
    animation: google.maps.Animation.DROP,
    position: {lat: 59.325, lng: 18.070}
  });

  marker.addListener('click', () => {
    infowindow.open(map,marker);
  });

  const infowindow = new google.maps.InfoWindow({
    content: `<h3>Event Location</h3><p>Event Address with all the contact details</p>`
  });

  infowindow.open(map,marker);
}
```

用你的活动地点的纬度和经度替换上述代码中的`lat`和`lng`值，并将`infowindow`对象的内容更改为活动地点的地址和联系方式。现在，在 Google Chrome 上打开`about.html`页面；你应该看到地图上有一个标记指向你的活动地点。信息窗口将默认打开。

恭喜！你已经成功构建了你的 Event Registration 应用！但是，在我们开始邀请人们参加活动之前，你的应用还有一件事情需要做。

# 生成生产构建

你可能已经注意到了关于 Meme Creator 和 Event Registration 应用的一些问题。这些应用首先加载纯 HTML；之后加载样式。这使得应用在一段时间内看起来很普通。在 ToDo List 应用中不存在这个问题，因为我们首先加载了 CSS。在 Meme Creator 应用中，有一个名为*为不同环境优化 Webpack 构建*的可选部分。现在可能是阅读它的好时机。如果你还没有阅读过，请回去，阅读一下那部分内容，然后回来生成生产构建。

到目前为止，我们的应用一直在开发环境中运行。记得吗？在`.env`文件中，我告诉你要设置`NODE_ENV=dev`。这是因为，当你按照我创建的`webpack.config.js`文件设置`NODE_ENV=production`时，Webpack 将进入生产模式。`npm run watch`命令用于运行 Webpack 开发服务器，为我们提供一个开发服务器。在你的`package.json`文件中，应该有另一个名为`webpack`的命令。这个命令用于生成生产构建。

这个项目中包含的`webpack.config.js`文件有很多插件，用于优化代码，使应用加载时间更快。只有当`NODE_ENV`为 production 时，`npm run watch`才能正常工作，因为有很多插件用于进行生产优化。要为你的 Event Registration 应用生成生产构建，请按照以下步骤进行：

1.  将`.env`文件中`NODE_ENV`变量的值更改为`production`。

1.  在终端中从项目根文件夹运行以下命令`npm run webpack`。

命令执行需要一段时间，但一旦完成，你应该在项目的`/dist`文件夹中看到许多文件。那里会有 JS 文件、CSS 文件和包含生成的 CSS 和 JS 文件的`.map`文件的源映射信息。JS 文件将被压缩和精简，以便加载和执行时间非常快。还会有一个包含 Bootstrap 使用的字体的字体目录。

到目前为止，我们只在 HTML 中包含了 JS 文件，因为它也包含了 CSS 代码。然而，这就是为什么页面在开始加载时显示空白的 HTML 而没有 CSS 的原因。CSS 文件应该在`<body>`元素之前包含，这样它将首先加载，页面样式在加载时将是统一的（看看我们在第一章中如何包含 CSS 文件，*构建一个待办事项列表应用*）。对于生产构建，我们需要删除对旧 JS 文件的引用，并包含新生成的 CSS 和 JS 文件。

在你的`dist/`目录中，会有一个`manifest.json`文件，其中包含了 Webpack 每个入口生成的文件列表。`manifest.json`应该看起来像这样：

```js
{
  "status": [
    "16f9901e75ba0ce6ed9c.status.js",
    "16f9901e75ba0ce6ed9c.status.css",
    "16f9901e75ba0ce6ed9c.status.js.map",
    "16f9901e75ba0ce6ed9c.status.css.map"
  ],
  "home": [
    "756fc66292dc44426e28.home.js",
    "756fc66292dc44426e28.home.css",
    "756fc66292dc44426e28.home.js.map",
    "756fc66292dc44426e28.home.css.map"
  ],
  "about": [
    "1b4af260a87818dfb51f.about.js",
    "1b4af260a87818dfb51f.about.css",
    "1b4af260a87818dfb51f.about.js.map",
    "1b4af260a87818dfb51f.about.css.map"
  ]
}
```

前缀数字只是哈希值，它们可能对你来说是不同的；不用担心。现在，为每个 HTML 文件包含 CSS 和 JS 文件。例如，取`status.html`文件，并在前面的`manifest.json`文件的 status 属性中添加 CSS 和 JS 文件，如下所示：

```js
...
<head>
  ...
  <link rel="stylesheet" href="dist/16f9901e75ba0ce6ed9c.status.css">
</head>
<body>
  ...
  <script src="dist/16f9901e75ba0ce6ed9c.status.js"></script>
</body>
...
```

对其他 HTML 文件重复相同的过程，然后你的生产构建就准备好了！现在不能使用 Webpack 开发服务器，所以你可以使用`http-server`工具打开网页，或者直接用 Chrome 打开 HTML 文件（我建议使用`http-server`）。这一次，在页面加载时，你不会看到没有样式的 HTML 页面，因为 CSS 会在 body 元素之前加载。

# 发布代码

现在你已经学会了如何生成生产构建，如果你想把这段代码发送给其他人呢？比如 DevOps 团队或服务器管理员。在这种情况下，如果你正在使用版本控制，将`dist/`目录、`node_modules/`目录和`.env`文件添加到你的忽略列表中。发送代码时不包括这两个目录和`.env`文件。其他人应该能够使用`.env.example`文件找出要使用的环境变量，创建`.env`文件，并使用`npm install`和`npm run webpack`命令生成`node_modules/`和`dist/`目录。

对于所有其他步骤，将过程整齐地记录在项目根目录的`README.md`文件中，并将其与其他文件一起发送。

共享`.env`文件应该避免的主要原因是环境变量可能包含敏感信息，不应以明文形式在版本控制中传输或存储。

你现在已经学会了如何为使用 Webpack 构建的应用生成生产构建。现在，Meme Creator 应用还没有生产构建！我会让你使用本章中使用的`webpack.config.js`文件作为参考。所以，继续为你的 Meme Creator 创建一个生产构建。

# 摘要

干得好！你刚刚构建了一个非常有用的活动注册应用。在这个过程中，你学到了一些 JavaScript 的高级概念，比如构建可重用代码的 ES6 模块，使用 fetch 进行异步 AJAX 调用，并使用 Promises 处理异步代码。你还使用 Chart.js 库构建图表来直观显示数据，最后使用 Webpack 创建了一个生产就绪的构建。

学会了所有这些概念，你不再是 JavaScript 的初学者；你可以自豪地称自己为专家！但是，除了这些概念，现代 JavaScript 还有很多其他内容。正如我之前告诉过你的，JavaScript 不再仅仅是用于浏览器表单验证的脚本语言。在下一章中，我们将使用 JavaScript 构建一个点对点视频通话应用程序。
