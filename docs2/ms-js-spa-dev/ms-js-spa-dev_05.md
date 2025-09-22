# 第五章. 一切都关于视图

用户界面层，或**视图**，是任何应用程序中最明显的组件。无论底层发生什么，无论是 REST、Websockets、MQTT 还是 SOAP，视图都是所有内容汇聚以提供完整、交互式应用程序体验的地方。就像服务器端一样，视图有其自己的复杂性和从开发角度出发的众多架构选择。现在，我们将探讨一些这些选择以及可以在全面视图层中使用的不同设计模式。

在本章中，我们将介绍以下内容：

+   不同的 JavaScript 模板引擎之间的差异

+   预编译 JavaScript 模板的优势

+   如何优化您的应用程序布局

# JavaScript 模板引擎

在应用程序的前端维护视图对于保持其服务器端无关性大有裨益。即使您在底层使用 MVC 框架来为应用程序提供 REST 端点，保持前端视图模板和逻辑也将确保您可以在未来更容易地更换 MVC 后端，而不会显著改变应用程序的逻辑和架构结构。JavaScript 模板引擎提供了一种有效的方法，可以在前端完全管理视图模板。

有许多开源 JavaScript 模板引擎可用。接下来，我们将介绍一些更受欢迎的模板引擎的基础知识：

+   Underscore.js

+   Mustache.js

+   Handlebars.js

+   Pure.js

+   Pug

+   EJS

# Underscore.js

**Underscore.js**库因其有用的 JavaScript 函数式编程辅助工具和实用方法而闻名。其中一种实用方法是`_.template()`。此方法用于将带有表达式的字符串编译成函数，这些函数用动态值替换那些表达式。

Underscore.js 模板的语法分隔符类似于**ERB**或**嵌入式 Ruby**模板语法，在开标签后跟一个**等于**符号：

```js
<p>Hello, <%= name %>.</p> 

```

在 HTML 中使用的 Underscore.js 模板表达式看起来像前面的示例。变量名将动态传递给编译后的函数，用于此模板。

`_.template()`方法还可以用于在模板中解析和执行任意 JavaScript 代码。Underscore.js 模板中的 JavaScript 代码使用 ERB 风格标签进行分隔，但开标签后不跟一个等于符号：

```js
<ul> 
    <% _.each(items, function(item) { %> 
        <li><%= item.property %></li> 
    <% } %> 
</ul> 

```

正如您在这个示例中可以看到的，模板中的 ERB 标签使脚本能够访问全局`_`对象，并允许它使用库的`_.each()`方法遍历该上下文中包含的给定对象或数组，甚至可以从该上下文向上遍历作用域链。脚本对`_`对象的访问表明，任何附加到`window`命名空间的全局变量都对脚本可用。

给模板赋予执行任意 JavaScript 代码的能力，在社区中引发了广泛的讨论，普遍的看法是这种做法是不被提倡的。这主要是因为从其他网络脚本语言，如 PHP 中吸取的教训。

将动态业务逻辑的代码与 HTML 直接混合在模板中，可能会导致代码库难以维护和调试，其他开发人员和未来几代人也会遇到困难。这种类型的代码也违反了 MVC 和 MVW 架构模式的原则。不言而喻，开发者编写代码时，选择在模板中包含多少或多少业务逻辑取决于他们自己，但许多 JavaScript 模板引擎的创造者认为，打开这扇门并不是一个选择。因此，*无逻辑*模板的概念应运而生。

你可以在`underscorejs.org`了解更多关于 Underscore.js 的信息。

# Mustache.js

**Mustache.js**是流行的**Mustache 模板系统**在 JavaScript 模板中的实现。Mustache 自诩为一种*无逻辑*的模板语法。这个概念背后的想法并不是要使模板完全没有逻辑，而是更多地劝阻在模板中包含大量业务逻辑的做法。

Mustache 的名字来源于使用双大括号作为模板的默认分隔符标签，这些大括号形状类似于胡须。Mustache 模板与 Underscore.js 模板的主要区别在于，Mustache 不允许在标签的另一种形式中放置任意 JavaScript 代码；它只允许使用表达式。

在其最简单的形式中，Mustache 模板将 JavaScript 对象中的值直接映射到相应的模板表达式，这些表达式由对象值的键表示。以这里显示的示例对象为例：

```js
{ 
    "name": { 
        "first": "Udis", 
        "last": "Petroyka" 
    }, 
    "age": "82" 
} 

```

这个对象中的值可以用以下方式表示在 Mustache 模板中：

```js
<p>{{name.first}} {{name.last}}</p> 
<p>Age: {{age}}</p> 

```

在这个例子中，你可以看到，即使嵌套对象值也可以通过使用 JavaScript 点表示法来访问，如`{{name.first}}`和`{{name.last}}`所示：

```js
<p>Udis Petroyka</p> 
<p>Age: 82</p> 

```

## 部分

Mustache 模板还包括渲染*部分*或*文本块*的能力。这涉及到使用包含开标签和闭标签语法的替代表达式语法。一个部分如何渲染取决于被调用的键的值。

### 布尔值

给定一个布尔值，一个部分将根据该布尔值是`true`还是`false`来渲染或不渲染：

```js
{ 
    "name": { 
        "first": "Jarmond", 
        "last": "Dittlemore" 
    }, 
    "email_subscriber": false 
} 

```

部分的分隔符语法由开有大括号、后跟井号`#`符号和用于开始部分的属性名，以及闭有大括号、后跟斜杠`/`符号和用于结束部分的属性名组成。这种语法与 HTML 的打开和关闭标签类似：

```js
<p>{{name.first}} {{name.last}}</p> 
{{#email_subscriber}} 
    <p>Content here will not be shown for this user.</p> 
{{/email_subscriber}} 

```

在这个例子中，`email_subscriber`属性被设置为 false，因此模板将渲染以下 HTML：

```js
<p>Jarmond Dittlemore</p> 

```

从本质上讲，使用具有布尔值的部分等同于一个`if`条件语句。这种用法确实包括逻辑，尽管它是最基本的形式。这样，*无逻辑*这个术语被证明并不像最初感知的那样严格。

### 列表

此外，部分可以用来遍历作为给定对象键值的列表项。在部分内部，上下文或变量作用域将转移到正在迭代的键上。以下是一个父键及其对应值列表的例子：

```js
{ 
    "people": [ 
        { "firstName": "Peebo", "lastName": "Sanderson" }, 
        { "firstName": "Udis", "lastName": "Petroyka" }, 
        { "firstName": "Jarmond", "lastName": "Dittlemore" }, 
        { "firstName": "Chappy", "lastName": "Scrumdinger" } 
    ] 
} 

```

给定一个包含人和他们名字的列表，可以使用一个部分来将每个人的名字渲染为 HTML 无序列表：

```js
<ul> 
{{#people}} 
    <li>{{firstName}} {{lastName}}</li> 
{{/people}} 
</ul> 

```

给定前面的示例对象，此模板代码将渲染以下 HTML：

```js
<ul> 
    <li>Peebo Sanderson</li> 
    <li>Udis Petroyka</li> 
    <li>Jarmond Dittlemore</li> 
    <li>Chappy Scrumdinger</li> 
</ul> 

```

### Lambda

对象属性值也可以从*lambda*或函数中返回，这些函数作为数据传递回当前部分的上下文：

```js
{ 
    "people": [ 
        { "firstName": "Peebo", "lastName": "Sanderson" }, 
        { "firstName": "Udis", "lastName": "Petroyka" }, 
        { "firstName": "Jarmond", "lastName": "Dittlemore" }, 
        { "firstName": "Chappy", "lastName": "Scrumdinger" } 
    ], 
    "name": function() { 
        return this.firstName + ' ' + this.lastName; 
    } 
} 

```

在列表的情况下，lambda 将基于当前列表项的上下文返回一个值，用于迭代：

```js
<ul> 
{{#people}} 
    <li>{{name}}</li> 
{{/people}} 
</ul> 

```

以这种方式，前面的模板将产生与上一个示例相同的输出：

```js
<ul> 
    <li>Peebo Sanderson</li> 
    <li>Udis Petroyka</li> 
    <li>Jarmond Dittlemore</li> 
    <li>Chappy Scrumdinger</li> 
</ul> 

```

## 倒置部分

Mustache 模板中的倒置部分是在该部分的键值是`false`或*假值*时才渲染的，例如`null`、`undefined`、`0`或一个空列表`[]`。以下是一个对象的例子：

```js
{ 
    "name": { 
        "first": "Peebo", 
        "last": "Sanderson" 
    }, 
    "email_subscriber": false 
} 

```

倒置部分以一个反引号`^`符号开头，而不是用于标准部分的井号`#`符号。给定前面的示例对象，以下模板语法可以用来渲染`false`属性值的 HTML：

```js
<p>{{name.first}} {{name.last}}</p> 
{{^email_subscriber}} 
    <p>I am not an email subscriber.</p> 
{{/email_subscriber}} 

```

根据对象中的`false`属性值，此模板将渲染以下 HTML：

```js
<p>Peebo Sanderson</p> 
<p>I am not an email subscriber.</p> 

```

## 注释

Mustache 模板还允许您在模板中包含注释。使用 Mustache 语法而不是 HTML 注释的优点是，它们不会被渲染到 HTML 输出中，就像标准 HTML 注释那样：

```js
<p>Udis likes to comment{{! hi, this comment won't be rendered }}</p> 
<!- This is a standard HTML comment -> 

```

Mustache 注释由一个*感叹号*或感叹号紧跟在开括号之后表示。前面的模板代码将渲染以下内容：

```js
<p>Udis likes to comment</p> 
<!- This is a standard HTML comment -> 

```

如所示，Mustache 模板注释不是渲染的 HTML 的一部分，但标准的 HTML 注释是。使用 Mustache 模板注释的优势在于渲染的 HTML 的负载大小，您希望将其保持尽可能小，并且可能没有很多情况您实际上想要在动态 HTML 中渲染注释。这允许您在模板代码中为其他开发者提供有用的注释，而不会给应用程序的前端带来负担。

## 部分

Mustache 模板中最有用的功能之一是能够在编译的模板中包含*部分*，或在运行时渲染的单独模板。从概念上讲，这个功能与服务器端模板语言的*包含*类似。

部分的语法在开有大括号 `>` 符号后跟部分名称。一个常见的文件命名约定是在未编译的部分文件名前加上下划线 `_`。考虑以下两个文件：

`user.hbs`

```js
<h3>{{name.first}} {{name.last}}</h3> 
{{> user-details}} 

```

`_user-details.hbs`

```js
<ul> 
    {{^email_subscriber}} 
    <li>I am not an email subscriber</li> 
    {{/email_subscriber}} 
    <li>Age: {{age}}</li> 
    <li>Profession: {{profession}}</li> 
</ul> 

```

在 user.hbs 的第二行指示了包含部分文件的调用。这将解析与 user.hbs 相同上下文中的 _user-details.hbs。在标准的编译器设置中，部分文件名中的下划线会被排除在键名之外，模板将存储在部分命名空间内：

```js
{ 
    "name": { 
        "first": "Jarmond", 
        "last": "Dittlemore" 
    }, 
    "email_subscriber": false, 
    "age": 24, 
    "profession": "Student" 
} 

```

给定前面的示例对象，模板完全渲染的 HTML 将看起来像以下内容：

```js
<h3>Jarmond Dittlemore</h3> 
<ul> 
    <li>I am not an email subscriber</li> 
    <li>Age: 24</li> 
    <li>Profession: Student</li> 
</ul> 

```

如示例所示，对象中的键名直接用于与父模板相同上下文的部分。

## 设置替代定界符

Mustache 模板的一个较不寻常的特性是能够在模板内部的标准 Mustache 定界符标签中设置*替代*定界符。这是通过在开有大括号定界符后跟一个等号 `=`，插入新的开定界符，然后是新的闭定界符，以及一个等号后跟标准闭定界符标签来完成的：

```js
{{=<% %>=}} 

```

如果这段代码放在 Mustache 模板的任何地方，那么下面的定界符标签将使用新的语法：

```js
{ 
    "name": { 
        "first": "Chappy", 
        "last": "Scrumdinger" 
    }, 
    "email_subscriber": false, 
    "age": 96, 
    "profession": "Oilman" 
} 

```

给定前面的对象，可以使用该数据结合替代定界符标签来构建模板：

```js
<p>Standard tags: {{name.first}} {{name.last}}</p> 
{{=<% %>=}} 
<p>New tags - Age: <%age%></p> 
<%={{ }}=%> 
<p>Standard tags again - Profession: {{profession}}</p> 

```

在这个例子中，标准标签使用了一次，然后使用集合定界符功能将标签更改为使用 ERB 风格的定界符，然后标签再次更改为原始的标准定界符：

```js
<p>Standard tags: Chappy Scrumdinger</p> 
<p>New tags - Age: 96</p> 
<p>Standard tags again - Profession: Oilman</p> 

```

生成的 HTML 将看起来像前面的代码，在一个模板中使用两组完全不同的定界符。

你可以在 [github.com/janl/mustache.js](http://github.com/janl/mustache.js) 上了解更多关于 Mustache.js 的信息，或者了解原始的 Mustache 模板在 [mustache.github.io](http://mustache.github.io)。

# Handlebars.js

Handlebars.js 模板也被认为是*无逻辑*的，并且主要基于 Mustache 模板，但提供了一些额外的功能。它们还排除了 Mustache 模板中一些创建者认为不实用的功能。

Handlebars 是 JavaScript 社区中较突出的模板引擎之一。它被包括 Backbone.js、Ember.js 和流行的 Meteor.js 框架在内的几个主要开源 JavaScript 框架所使用。它使用自己的反应式 Handlebars 模板引擎，称为 Spacebars。由于其流行，我们将在下面更深入地介绍 Handlebars。

## 显式路径查找与递归路径查找

Handlebars 模板与 Mustache 模板区分开来的一个特性是，Handlebars 不支持递归路径查找，正如 Mustache 模板所做的那样。这涉及到 Handlebars 中的“部分”或“块”。当你处于对象子属性上下文时，Handlebars 不会自动查找作用域链中的表达式引用。相反，你必须显式定义你要查找的变量的路径。这使得 Handlebars 模板中的作用域更有意义和可理解：

```js
{ 
    "name": { 
        "first": "Peebo", 
        "last": "Sanderson" 
    }, 
    "email_subscriber": false, 
    "age": 54, 
    "profession": "Singer" 
} 

```

给定此对象，以下模板语法将适用于 Mustache 模板：

```js
<!-- Mustache template --> 
<p>{{name.first}} {{name.last}}</p> 
{{#profession}} 
<p>Profession: {{profession}}</p> 
{{/profession}} 

```

此模板将渲染 `#profession` 块作用域内 `profession` 键的值，因为 Mustache 支持递归路径查找。换句话说，嵌套上下文始终可以访问其上方的父上下文中的变量。然而，Handlebars 默认情况下并不是这样：

```js
<!-- Handlebars template --> 
<p>{{name.first}} {{name.last}}</p> 
{{#profession}} 
<p>Profession: {{this}}</p> 
{{/profession}} 

```

如此示例所示，`this` 关键字用于引用当前块上下文设置的变量。如果直接引用 `profession` 变量，这将在 Handlebars 中引发错误。

```js
<!-- Handlebars template --> 
<p>{{name.first}} {{name.last}}</p> 
{{#profession}} 
<p>Profession: {{../profession}}</p> 
{{/profession}} 

```

此外，Handlebars 可以使用前面代码中显示的 `../` 语法通过**显式路径**引用查找作用域链中的变量。这种语法模仿了命令行界面中递归文件路径查找的方式。在这个例子中，`../profession` 引用简单地查找当前块上下文设置的变量：

```js
<p>Peebo Sanderson</p> 
<p>Profession: Singer</p> 

```

Handlebars 默认不支持递归路径查找的原因是为了提高速度。通过将路径查找限制在当前块上下文中，Handlebars 模板可以更快地渲染。提供了一个编译时 `compat` 标志来覆盖此功能并允许递归路径查找，但 Handlebars 的创建者建议不要这样做，并指出这样做会有性能成本。

## 辅助函数

Handlebars 模板不支持 Mustache 模板中定义的 lambda 表达式，而是使用辅助函数来增加功能。Handlebars 中的辅助函数是一种将视图逻辑抽象化的方式，这些逻辑在其他情况下可能直接在模板中完成，尤其是在使用限制较少的模板引擎（如 Underscore.js）时。相反，你可以编写一个以常规 JavaScript 函数形式存在的辅助函数，将其注册到 Handlebars 命名空间中，并在模板中使用它作为一个单独的表达式或块表达式：

```js

{ 
    "name": { 
        "first": "Udis", 
        "last": "Petroyka" 
    }, 
    "age": "82" 
} 

```

给定此示例对象，可以编写一个辅助函数来根据对象属性返回用户的完整姓名：

```js
Handlebars.registerHelper('fullName', function(name) { 
    return name.first + ' ' + name.last; 
}); 

```

如此所示，`Handlebars` 对象提供了一个 `registerHelper` 方法，它允许你通过定义第一个参数为名称，第二个参数为 lambda 表达式来定义一个辅助函数。lambda 表达式的参数可以直接从模板上下文中在辅助函数被调用的点提供；在这种情况下，作为一个表达式：

```js
<p>Hi, my name is {{fullName name}}.</p> 

```

如前例所示，辅助函数的语法使用辅助函数的名称紧随 Handlebars 开启标签之后，后跟传递给辅助函数的任何参数；在这种情况下，是 `name` 参数：

```js
<p>Hi, my name is Udis Petroyka.</p> 

```

模板随后将作为 HTML 渲染，显示从辅助函数返回的完整名称，这是通过从模板上下文中传递所需的对象属性来实现的。

### 辅助函数作为块表达式

Handlebars 模板使用块表达式语法来调用辅助函数。然而，块表达式的上下文完全取决于辅助函数的编写方式。Handlebars 提供了几个内置的块辅助函数。

#### `#if` 块辅助函数

Handlebars 提供了一个简单的 `#if` 块辅助函数，用于根据布尔值或真值与假值解析来渲染内容或不渲染内容。这意味着像 `0`、`null`、`undefined` 和空列表 `[]` 这样的值将被解析为假。

考虑以下对象：

```js
{ 
    "name": { 
        "first": "Jarmond", 
        "last": "Dittlemore" 
    }, 
    "email_subscriber": false 
} 

```

而不是使用标准的 Mustache 风格部分实现，这里的 `#if` 辅助函数可以用于布尔值：

```js
<p>{{name.first}} {{name.last}}</p> 
{{#if email_subscriber}} 
    <p>I am an email subscriber.</p> 
{{/if}} 

```

此模板不会渲染 `#if` 块内的部分，因为 `email_subscriber` 是 `false`。内置的 `#if` 辅助函数还提供了在 `#if` 块内包含 `{{else}}` 部分的选项，如果传递的变量评估为 `false`，则将渲染该部分：

```js
<p>{{name.first}} {{name.last}}</p> 
{{#if email_subscriber}} 
    <p>I am an email subscriber.</p> 
{{else}} 
    <p>I am not an email subscriber.</p> 
{{/if}} 

```

给定示例对象，此模板将渲染以下内容：

```js
<p>Jarmond Dittlemore</p> 
<p>I am not an email subscriber.</p> 

```

Handlebars 中的 `#if` 辅助函数与 Mustache 模板中的部分之间的另一个区别是，`#if` 辅助函数内的上下文不会改变，而部分内的上下文会改变为被调用的对象属性。

#### `#unless` 块辅助函数

Handlebars 中的 `#unless` 块辅助函数类似于 Mustache 模板中的反转部分功能，也可以被认为是 Handlebars `#if` 辅助函数的反面。如果传递给 `#unless` 辅助函数的值是假的，则该块将被渲染：

```js
{ 
    "name": { 
        "first": "Chappy", 
        "last": "Scrumdinger" 
    }, 
    "email_subscriber": false 

} 

```

考虑一个类似于先前 `#if` 示例的模板，并基于前面的对象：

```js
<p>{{name.first}} {{name.last}}</p> 
{{#unless email_subscriber}} 
   <p>I am not an email subscriber.</p> 
{{/if}} 

```

此模板将渲染 `#unless` 块内的内容，因为 `email_subscriber` 的值为 `false`：

```js
<p>Chappy Scrumdinger</p> 
<p>I am not an email subscriber.</p> 

```

#### `#each` 块辅助函数

`#each` 块辅助函数用于遍历列表和对象。在其最基本的形式中，它就像在列表上下文中使用 Mustache 部分，但它具有额外的功能，使其功能更强大：

```js
{ 
    "people": [ 
        { "firstName": "Peebo", "lastName": "Sanderson" }, 
        { "firstName": "Udis", "lastName": "Petroyka" }, 
        { "firstName": "Jarmond", "lastName": "Dittlemore" }, 
        { "firstName": "Chappy", "lastName": "Scrumdinger" } 
    ] 
} 

```

在列表的 `#each` 上下文中，可以使用 `this` 关键字来引用列表中的当前值：

```js
<ul> 
{{#each people}} 
    <li>{{this.firstName}} {{this.lastName}}</li> 
{{/each}} 
</ul> 

```

这与 Mustache 模板的迭代 lambda 示例类似，但在此情况下不需要 lambda 属性值来访问迭代的对象属性。

由于在此示例中，每个迭代的范围都限制在当前正在迭代的对象上，因此前面的模板也可以更简单地写成以下形式：

```js
<ul> 
{{#each people}} 
    <li>{{firstName}} {{lastName}}</li> 
{{/each}} 
</ul> 

```

如你所见，由于每个迭代的上下文都设置为该对象，因此不需要`this`关键字来访问每个对象的属性。

#### `#with`块辅助函数

`#with`块辅助函数的工作方式与 Mustache 模板中的标准部分非常相似，通过将当前块的上下文限制为传入的父键：

```js
{ 
    "name": { 
        "first": "Peebo", 
        "last": "Sanderson" 
    }, 
    "email_subscriber": false, 
    "age": 54, 
    "profession": "Singer" 
} 

```

给定这个示例对象，可以使用`#with`辅助函数来限制一个块到`name`键的上下文中来构建一个模板：

```js
<h1>User Information</h1> 
<dl> 
    <dt>Name</dt> 
    {{#with name}} 
    <dd>{{first}} {{last}}</dd> 
    {{/with}} 
    <dt>Age</dt> 
    <dd>{{age}}</dd> 
    <dt>Profession</dt> 
   <dd>{{profession}}</dd> 
</dl> 

```

此模板将渲染以下 HTML：

```js
<h1>User Information</h1> 
<dl> 
    <dt>Name</dt> 
   <dd>Peebo Sanderson</dd> 
   <dt>Age</dt> 
    <dd>54</dd> 
    <dt>Profession</dt> 
   <dd>Singer</dd> 
</dl> 

```

# Handlebars 与 Mustache 模板的其他差异

Handlebars.js 中的许多功能都是为了让模板在浏览器中渲染得更快而设计的，这些功能使 Handlebars.js 与 Mustache.js 区分开来。Handlebars 中允许这一点的其中一个主要功能是预编译模板，正如我们在第二章中讨论的，“模型-视图-任意”。

## 预编译模板

预编译模板将它们转换为在渲染之前通常在应用程序中编译的 JavaScript 函数。使用此功能通过跳过该步骤来提高应用程序的速度，并且它还减少了浏览器对应用程序的负载，因为 JavaScript 编译器不需要包含在前端资产负载中。

## 没有替代分隔符

Handlebars 的创建者还决定，在模板内设置替代分隔符的能力并非必要。如果你没有预先编译你的模板，这将进一步减少应用程序的资产负载。

通常，除了个人偏好之外，你想要更改模板分隔符样式的唯一原因是为了避免与其他模板语言的冲突，例如，使用相同分隔符的服务器端模板语言。如果你要在服务器端模板中的 JavaScript 块内包含 Handlebars 模板，这个问题就会出现。然而，如果你预先编译你的模板或者通过将它们保存在自己的外部 JavaScript 文件中从服务器端模板中抽象它们，那么这个问题可以完全避免，并且不需要设置替代分隔符。

你可以在`handlbarsjs.com`了解更多关于 Handlebars.js 的信息。

# Pure.js

Pure.js 是一个 JavaScript 模板引擎，它将无逻辑模板的概念推向了比 Mustache 和 Handlebars 更极端的程度。Pure.js 不使用在渲染之前必须插值的特殊模板表达式语法。相反，它仅使用纯 HTML 标签和 CSS 选择器，结合 JSON 数据，在 DOM 中渲染值。因此，Pure.js 使用完全无逻辑的视图，因为没有模板标记可以包含任何逻辑。

## 标记

使用纯 HTML，一个简单的 Pure.js 模板可以构建如下：

```js
<p class="my-template"> 
    Hello, my name is <span></span>. 
</p> 

```

空的`<span>`元素是你可以为特定模板添加数据的地方，但你可以使用任何 HTML 标签。

```js
var data = { 
    name: 'Udis Petroyka' 
}; 

var directive = { 
    'span': 'name' 
}; 

```

在这个例子中，我们在 `data` 变量中提供了模板的数据，然后提供了一个所谓的 `directive`，它告诉模板引擎如何映射这些数据：

```js
$p('.my-template').render(data, directive); 

```

Pure.js 在全局范围内提供了一个 `$p` 对象，该对象提供了与模板交互的方法。在这种情况下，我们调用 `render()` 方法，并将 `data` 和 `directive` 作为参数传递：

```js
<p class="my-template"> 
    Hello, my name is <span>Udis Petroyka</span>. 
</p> 

```

这将是这个简单示例的渲染结果。您可以在 `beebole.com/pure/.` 上了解更多关于 Pure.js 的信息。

# Pug

Pug，正式名称为 Jade，是一个在 Node.js 社区中非常突出的 JavaScript 模板引擎。它主要受到 **HTML 抽象标记语言**（**Haml**）的影响，最初设计用来通过使用比原始 HTML 更简洁、更简洁的语法来简化 ERB 模板的编写。因此，Pug 需要编译其表达式，以及标记语言本身。

Pug 与 YAML 类似，其层次结构通过缩进来表示，定界符使用空格。这意味着不需要关闭元素标签：

```js
doctype html 
html(lang="en") 
  head 
    title= pageTitle 
body 
  h1 This is a heading 
  if thisVariableIsTrue 
    p This paragraph will show. 
  else 
    p This paragraph will show instead. 

```

如此例所示，Pug 可以用作 HTML 的简单缩写语法。它还可以包含带有变量的简单条件，所有这些都遵循相同的流畅语法。通过在标签名后包含括号并定义属性，可以添加 HTML 元素属性，例如示例中的 `html(lang="en")`。用变量填充的元素通过在标签名后放置等号并跟随 JavaScript 键名来表示，如示例中的 `title= pageTitle` 所示：

```js
{ 
    pageTitle: 'This is a dynamic page title', 
    thisVariableIsTrue: true 
} 

```

使用这个示例 JavaScript 对象，前面的模板将渲染以下 HTML：

```js
<!DOCTYPE html> 
<html lang="en"> 
  <head> 
    <title>This is a dynamic page title</title> 
  </head> 
  <body> 
    <h1>This is a heading</h1> 
    <p>This paragraph will show.</p> 
  </body> 
</html> 

```

内联变量还可以使用另一种语法，允许访问顶层属性和嵌套属性。考虑以下对象：

```js
{ 
    "name": { 
        "first": "Jarmond", 
        "last": "Dittlemore" 
    }, 
    "email_subscriber": false, 
    "age": 24, 
    "profession": "Student" 
} 

```

一个 Pug 模板可以编写来访问这个对象中的所有变量，如下所示：

```js
h1#title Hello, my name is #{name.first} #{name.last}. 
if email_subscriber 
  p I am an email subscriber. 
else 
  p i am not an email subscriber. 
h2.age Age 
p= 24 
h2.profession Profession 
p I am a #{profession}. 

```

在这个模板中，您可以看到使用了内联变量语法 `#{}`，以及一个条件和用 `=` 语法填充变量的元素。

您还会注意到，`h1` 标签后面紧跟着一个 `#` 符号和单词 `title`，而 `h2` 标签后面跟着 `.className`。这展示了 Pug 的另一个特性，它允许使用标准的 CSS 选择器语法来包含 ID 和类。从这个模板渲染的 HTML 将如下所示：

```js
<h1 id="title">Hello, my name is Jarmond Dittlemore.</h1> 
<p>I am not an email subscriber.</p> 
<h2 class="age">Age</h2> 
<p>24</p> 
<h2 class="profession">Profession</h2> 
<p>I am a Student.</p> 

```

这个例子展示了使用 Pug 编写比标准 HTML 结合另一种模板语法更简洁的写作方式，这可能是它变得如此受欢迎的原因。您可以在 [pug-lang.com](http://pug-lang.com) 上了解更多关于 Pug 的信息。

# 嵌入式 JavaScript (EJS)

**EJS** 是一个类似于 Underscore.js 的 JavaScript 模板引擎，它也使用 ERB `<% %>` 样式的定界符。或者，它还允许使用 `[% %]` 样式的标签作为定界符。

就像 Underscore.js 一样，EJS 允许在使用标准 `<% %>` ERB 风格语法时解析任意 JavaScript，并允许使用等于号 `=` 后跟开标签 `<%= %>` 来评估表达式：

```js
<ul> 
<% for (var i = 0; i < people.length; i++) { %> 
    <li><%= firstName %> <%= lastName %></li> 
<% } %> 
</ul> 

```

此模板可用于遍历一个对象列表，其中键名作为具有不同值的变量进行评估：

```js
{ 
    "people": [ 
        { "firstName": "Peebo", "lastName": "Sanderson" }, 
        { "firstName": "Udis", "lastName": "Petroyka" }, 
        { "firstName": "Jarmond", "lastName": "Dittlemore" }, 
        { "firstName": "Chappy", "lastName": "Scrumdinger" } 
    ] 
} 

```

使用示例模板遍历此对象将渲染以下 HTML：

```js
<ul> 
    <li>Peebo Sanderson</li> 
    <li>Udis Petroyka</li> 
    <li>Jarmond Dittlemore</li> 
    <li>Chappy Scrumdinger</li> 
</ul> 

```

## 同步模板加载

在典型的 EJS 用例中，每个模板都存储在一个具有专有 `.ejs` 扩展名的文件中。通过创建一个新的 `EJS` 对象，提供模板文件的路径，并使用要插入的数据调用渲染方法，从 JavaScript 代码编译模板。

假设 EJS 模板保存在您项目中的一个文件中，文件名为 `templates/people.ejs`。然后可以编写以下 JavaScript 代码来将其渲染为示例中显示的 HTML：

```js

var data = { 
    "people": [ 
        { "firstName": "Peebo", "lastName": "Sanderson" }, 
        { "firstName": "Udis", "lastName": "Petroyka" }, 
        { "firstName": "Jarmond", "lastName": "Dittlemore" }, 
        { "firstName": "Chappy", "lastName": "Scrumdinger" } 
    ] 
}; 
var html = new EJS({url: 'templates/people.ejs'}).render(data); 

```

全局 `EJS` 对象是一个构造函数，您可以通过创建一个新实例来解析模板，并对其调用方法进行渲染。请注意，由于文件路径在 JavaScript 中被引用，必须进行 *同步* 调用来最初加载模板以进行解析。这保持了您应用程序初始页面加载的低延迟，但可能会根据加载的模板的复杂性和应用程序运行的服务器速度导致交互时的响应时间更长。

一旦在您的 JavaScript 代码中创建了渲染后的 HTML，您只需将其插入到应用程序的 DOM 中：

```js
var html = new EJS({url: 'templates/people.ejs'}).render(data); 
document.body.innerHTML = html; 

```

## 异步数据加载

EJS 的一个独特功能是能够使用从外部源异步加载数据来渲染模板。使用前面的示例，假设 JSON 数据在一个名为 `people.json` 的外部文件中：

```js
new EJS({url: 'templates/people.ejs'}).update('element_id', 'people.json'); 

```

在此示例中，调用的是 `.update()` 方法而不是 `.render()`。对象实例也没有被分配给变量，因为 DOM 插入由 `.update()` 方法以及通过传递 DOM 节点 ID 来处理。为了使此方法工作，不能使用其他 CSS 选择器来将 HTML 注入 DOM；只有 ID 才有效。

## 缓存

EJS 默认在模板首次同步加载后缓存模板。这提供了一个优势，即多次使用的模板在第一次请求之后将始终加载得更快，而未使用的模板不会占用任何内存。这种方法与在应用程序初始页面加载时将所有模板加载到内存中的预编译方法形成鲜明对比。这两种方法都有优缺点，因此必须仔细选择最适合您特定应用程序的最佳方法。

可以通过在传递给任何模板实例化的选项对象中包含一个 `cache` 键来关闭任何模板的缓存：

```js
var html = new EJS({url: 'templates/people.ejs', cache: false}).render(data); 

```

## 视图辅助函数

EJS 包含一些与 Handlebars 模板中辅助函数概念类似的视图辅助函数。它们允许使用更短的语法来表示一些常见的 HTML 元素。以下我们将展示几个示例。

### link_to 视图辅助函数

`link_to` 视图辅助函数提供了一个简单的模板语法，以确保 HTML 超链接：

```js
<p>Here is a <%= link_to('link', '/link/path') %> to a path.</p> 

```

`link_to` 视图辅助函数的第一个参数是链接的显示文本，第二个参数是要传递给链接的 `href` 属性的路径。注意，视图辅助函数的定界符使用的是开表达式定界符语法。以下示例将渲染以下 HTML：

```js
<p>Here is a <a href="/link/path">link</a> to a path.</p> 

```

### img_tag 视图辅助函数

此 `img_tag` 视图辅助函数提供了一个在渲染的 HTML 中包含图像的简单语法：

```js
<p><%= img_tag('/path/to/image.png', 'Description text') %></p> 

```

`link_to` 视图辅助函数的第一个参数是图像的路径，第二个参数是图像的 `alt` 属性的文本。以下示例将渲染以下 HTML：

```js
<p><img src="img/image.png" alt="Description text"></p> 

```

### form_tag 视图辅助函数

`form_tag` 视图辅助函数提供了一个构建 HTML 表单的语法，可以与其他视图辅助函数结合使用，以创建输入元素：

```js
<%= form_tag('/path/to/action', {method: 'post', multipart: true}) %> 
    <%= input_field_tag('user_input', 'value text here') %> 
    <%= submit_tag('Submit') %> 
<%= form_tag_end() %> 

```

在此示例中，使用了四个视图辅助函数来构建表单。`form_tag` 视图辅助函数创建表单的主体，提供表单操作作为第一个参数，并使用大括号中的 JavaScript 对象语法在第二个参数中提供其他表单属性。`input_field_tag` 视图辅助函数用于创建标准输入文本字段，输入名称作为第一个参数，可选地，输入值作为第二个参数。`submit_tag` 视图辅助函数创建一个带有按钮文本的表单提交输入，作为第一个参数传递。最后，使用 `form_tag_end` 视图辅助函数关闭表单的主体。此示例将渲染以下 HTML：

```js
<form action="/path/to/action" method="post" enctype="multipart/form-data"> 
    <input type="text" id="user_input" name="user_input" value="value text here"> 
    <input type="submit" value="Submit"> 
</form> 

```

EJS 还包括许多其他视图辅助函数，用于常见的 HTML 元素，使用 `_tag` 后缀。

## 部分模板

EJS 有自己的部分实现，它使用同步模板加载技术，在模板定界符标签内工作。要使用此功能，需要在父模板中直接调用部分模板文件。考虑以下两个模板文件：

templates/parent.ejs

```js
<p>This is the parent template.</p> 
<%= this.partial({url: 'templates/partial.ejs %> 

```

templates/partial.ejs

```js
<p>This is the partial template.</p> 

```

注意，调用部分模板时，使用表达式定界符和 `this.partial` 方法的调用引用了父模板中的 URL。要在另一个模板中加载部分模板，只需从你的 JavaScript 代码中初始化父模板即可：

```js
var html = new EJS({url: 'templates/parent.ejs'}).render(); 
document.body.innerHTML = html; 

```

最终渲染的 HTML 将如下所示：

```js
<p>This is the parent template.</p> 
<p>This is the partial template.</p> 

```

这些示例提供了 EJS 的简要概述，但稍后我们将更深入地使用这个模板引擎。有关 EJS 模板的更多信息，请访问 `embeddedjs.com`。

# 优化你的应用程序布局

构建一个 JavaScript 单页应用（SPA）通常涉及许多抽象层，包括自定义应用程序代码、第三方库、前端框架、任务运行器、转译器等等。所有这些最终可能导致前端应用程序需要下载大量的 JavaScript 代码，因此应始终采取措施尽可能最小化这种影响。

让我们回到我们迄今为止一直在工作的 Node.js 示例应用程序。在 第二章 中，我们为应用程序编写了 index.html 布局页面，其中包含了以下脚本标签，用于第三方库和编译后的模板：

```js
<script src="img/jquery.min.js"></script> 
<script src="img/handlebars.runtime.min.js"></script> 
<script src="img/payload.js"></script> 
<script src="img/templates.js"></script> 

```

与一个完整规模的应用程序可能包含的 JavaScript 文件数量相比，这实际上是一个最小示例。

## UglifyJS 和 grunt-contrib-uglify

用于最小化和连接 JavaScript 文件的常见工具是 **UglifyJS**。我们可以利用这个工具在命令行上，并通过 Grunt 任务运行器和 **grunt-contrib-uglify** 任务来自动化它： 

```js
$ npm install grunt-contrib-uglify --save-dev

```

一旦安装，打开 Gruntfile.js 并立即在 watch 任务上方添加以下任务：

```js
uglify: { 
    options: { 
        preserveComments: false 
    }, 
    main: { 
        files: { 
            'js/all.min.js': [ 
              'bower_components/jquery/dist/jquery.js', 
              'bower_components/handlebars/handlebars.runtime.js', 
              'bower_components/payloadjs/payload.js', 
              'js/src/templates.js' 
            ] 
        } 
    } 
} 

```

这设置了 uglify 任务，以移除所有注释（`preserveComments` 选项设置为 false），打乱或缩短变量和函数名称，并将指示的 JavaScript 文件列表连接到单个目标文件名 `all.min.js`。使用此设置，UglifyJS 将根据输入文件创建尽可能小的 JavaScript 下载大小。

接下来，确保在 Gruntfile.js 的底部加载新的 uglify 任务，与其他任务一起：

```js
grunt.loadNpmTasks('grunt-contrib-uglify'); 

```

现在，你只需在命令行上运行任务：

```js
$ grunt uglify

```

运行任务后，你应该看到类似以下内容的输出：

```js
Running "uglify:main" (uglify) task
File all.min.js created: 322.28 kB → 108.6 kB
>> 1 file created.
Done, without errors.

```

你会注意到 CLI 输出显示了 JavaScript 文件的原始大小，以及第二行中最终输出的大小；在这个例子中，显示 322.28 kB → 108.6 kB。在这种情况下，它被压缩到原始大小的一半以下。

现在，你可以更改你的 index.html 布局文件，使其仅调用一个 JavaScript 文件：

```js
<!doctype html> 
<html> 
    <head> 
        <title>My Application</title> 
    </head> 
    <body> 
        <div id="app"></div> 
        <script src="img/all.min.js"></script> 
   </body> 
</html> 

```

将 `<script>` 标签放置在页面底部也确保了在 JavaScript 完全下载之前，任何位于其上方的内容都将被加载并可见给用户。这是通过防止用户在看到任何内容之前出现延迟来优化 SPAs 的另一种常见做法。

## grunt-contrib-handlebars

如果您在应用程序中使用 Handlebars 模板，则 `grunt-contrib-handlebars` 任务可用于从命令行和通过监视任务轻松预编译它们。在 第二章 中，我们创建了项目根目录下的示例 `user.handlebars` 文件，在 第四章 中，我们创建了 `users.handlebars`。现在，在 `js/templates` 目录下创建一个新的目录，并将文件移到那里。接下来，将文件重命名为 `user.hbs` 和 `users.hbs` 以便简洁。`.hbs` 扩展名也广泛接受为 Handlebars 文件：

```js
/ 
js/ 
   templates/ 
                   user.hbs 
                   users.hbs 

```

接下来，安装 grunt-contrib-handlebars 插件：

```js
$ npm install grunt-contrib-handlebars --save-dev

```

安装完成后，将以下任务配置添加到 `Gruntfile.js` 文件中，位于 `uglify` 任务配置之上：

```js
handlebars: {
     options: {
         namespace: 'Handlebars.templates',
         processName: function(file) {
             return file.replace(/js\/templates\/|\.hbs/g, '');
         },
         partialRegex: /.*/,
         partialsPathRegex: /\/partials\//
     },
     files: {
         src: 'js/templates/**/*.hbs',
         dest: 'js/src/templates.js'     }
 } 

```

Handlebars 的 Grunt 插件为您做出的假设比 Handlebars 命令行工具默认的假设要少，因此此配置为您做了几件事情。

### 选项配置

首先，`options` 对象传递了四个参数。`namespace` 选项简单地告诉编译器使用哪个全局命名空间来存储编译后的 Handlebars 模板函数。`Handlebars.templates` 是使用命令行工具的默认命名空间，因此我们将采用这个。

`processName` 参数传递一个函数，该函数接受一个 Handlebars 文件作为参数，并使用它来创建 `Handlebars.templates` 命名空间中该模板的键名。在这种情况下，我们使用正则表达式来获取路径和文件名，并移除除了文件名前缀之外的所有内容，因此 `user.hbs` 的编译模板函数，例如，将在 `Handlebars.templates.user` 下可用。

`partialRegex` 选项接受一个正则表达式，用于识别部分文件名的模式。默认情况下，这是一个以下划线 `_` 为前缀的文件，但在此情况下，我们将使用目录来存储部分，因此 `partialRegex` 选项设置为 `.*`，这意味着它将识别给定路径上的任何文件作为部分。

`partialsPathRegex` 选项接受一个正则表达式，用于识别部分目录的路径。我们将其设置为 `/\/partials\//`，这将评估为主模板路径下传递的 `/partials` 目录。结合 `partialRegex` 选项，这告诉编译器解析 `/partials` 目录中的每个文件作为部分，并将编译后的模板函数添加到 `Handlebars.partials` 命名空间中。

### 文件配置

传递给 Handlebars Grunt 任务的文件配置对象用于告诉编译器使用什么文件模式来查找编译模板，以及定义编译模板的输出文件名：

```js
files: { 
    src: 'js/templates/**/*.hbs', 
    dest: 'js/src/templates.js' 
} 

```

在这个例子中，我们已经定义了模板 `src` 目录位于根路径下的 `js/templates/`，并且解析该目录及其所有子目录下扩展名为 `.hbs` 的所有文件。递归目录查找由 `/**/*.hbs` 语法表示。

`dest` 键告诉编译器使用所有模板的最终编译输出创建 `js/src/templates.js` 文件。

### 运行 Grunt Handlebars 任务

为了运行 handlebars 任务，我们首先需要在 Gruntfile.js 的底部加载该插件：

```js
grunt.loadNpmTasks('grunt-contrib-handlebars'); 

```

接下来，从命令行运行 grunt handlebars 命令：

```js
$ grunt handlebars

```

运行任务后，您应该会看到类似以下内容的输出：

```js
Running "handlebars:files" (handlebars) task
>> 1 file created.
Done, without errors.

```

现在，如果您查看 `js/src/` 目录，应该会看到在那里创建了一个 `templates.js` 文件，它与我们在 第一章 中创建的 `app.js` 文件相邻，即 *使用 NPM、Bower 和 Grunt 进行组织*。现在我们在这里存储 `templates.js` 文件，请继续删除根目录中的原始 `templates.js` 文件，并编辑 Grunt uglify 任务中的 `files` 对象，使其如下所示：

```js
files: { 
    'js/all.min.js': [ 
        'bower_components/jquery/dist/jquery.js', 
        'bower_components/handlebars/handlebars.runtime.js', 
        'bower_components/payloadjs/payload.js', 
        'js/src/templates.js' 
    ] 
} 

```

现在我们已经将新的 `templates.js` 文件添加到 uglify 任务中，以便将其包含在完整的压缩应用程序 JavaScript 中。

## 监视更改

现在您正在加载压缩的 JavaScript 文件，您可能希望添加一个监视任务，在您开发时创建该文件，这样您就不必不断从 CLI 运行命令。

对于这个例子，让我们假设我们想要检测 `js/src` 目录中任何文件的更改，我们在那里积极工作。编辑 `Gruntfile.js` 中的 `watch` 任务配置，并在该任务的 `jshint` 目标下方直接添加以下内容：

```js
uglify: { 
    files: ['js/src/*.js'], 
    tasks: ['uglify:main'] 
} 

```

这告诉 Grunt 在检测到匹配模式的文件更改时运行 `uglify` 任务的 `main` 目标。此外，将 `jshint` 监视任务上面的 `uglify` 监视任务更改为以下内容：

```js
jshint: { 
    files: ['js/src/*.js', '!js/src/templates.js'], 
    tasks: ['jshint'] 
} 

```

这告诉监视任务在运行 jshint 任务时忽略对 `templates.js` 的更改。我们想忽略此文件，因为它已编译并且不会通过 JSHint 测试。

将相同的文件忽略路径添加到文件顶部附近的 *main* `jshint` 任务 `files` 配置中：

```js
files: { 
    src: [ 
        'Gruntfile.js', 
        'js/src/*.js', 
        '!js/src/templates.js' 
    ] 
} 

```

这将防止 JSHint 在运行 `jshint` 任务时将 `templates.js` 与其定义的规则进行比较。

我们还需要一个监视 Handlebars 模板文件更改的 `watch` 任务。在 `watch` 任务中，在 `uglify` 目标下方添加以下配置：

```js
handlebars: { 
    files: ['js/templates/**/*.hbs'], 
    tasks: ['handlebars'] 
} 

```

这将监视 Handlebars 模板和部分文件的任何更改，并相应地运行 `handlebars` 任务。这样做将生成 `templates.js` 文件，然后触发 uglify 监视任务运行，并将完整的应用程序 JavaScript 编译为 `all.min.js`。

接下来，从命令行运行 Grunt `watch` 命令：

```js
$ grunt watch
Running "watch" task
Waiting...

```

现在打开`user.hbs`，将标记修改为以下示例所示。注意，`{{name.first}}`和`{{name.last}}`表达式已更新为我们之前在 MongoDB 中创建的属性，参见*第四章，最佳实践 - 与应用程序的后端交互*：

```js
<h3>{{first_name}} {{last_name}}</h3> 
<p>{{title}}</> 

```

保存文件后，检查运行`watch`任务的控制台。你应该会看到以下类似的输出：

```js
>> File "js/templates/user.hbs" changed.
Running "handlebars:files" (handlebars) task
>> 1 file created.
Done, without errors.
Completed in 0.979s - Waiting...
>> File "js/src/templates.js" changed.
Running "uglify:main" (uglify) task
File all.min.js created: 322.28 kB → 108.6 kB
>> 1 file created.
Done, without errors.

```

对`user.hbs`的更改触发了两个任务的连锁反应，并且你的应用程序 JavaScript 被编译成最新版本。如果你打开编译后的`templates.js`文件，你会看到已经创建了一个用户及其属性，并关联了模板函数：

```js
this["Handlebars"]["templates"]["user"] = Handlebars.template({...}}); 
this["Handlebars"]["templates"]["users"] = Handlebars.template({...}}); 

```

接下来，当`handlebars`任务仍在运行时，将`user.hbs`文件移动到`js/templates/partials`目录。这将再次触发监视任务。当它完成后，再次打开`templates.js`，你会注意到`Handlebars.templates.user`属性不再定义。相反，使用`.registerPartial()`函数调用来代替：

```js
Handlebars.registerPartial("user", Handlebars.template({...}); 

```

这将在父模板中使用 Handlebars 部分语法包含`user.hbs`部分时调用`user.hbs`部分。现在打开`users.hbs`，将其修改为使用`user.hbs`部分：

```js
{{#each data}} 
    {{> user}} 
{{/each}} 

```

这将遍历提供的用户数据。在第四章，*最佳实践 - 与应用程序的后端交互*中，我们留下了只有一个条目的测试数据库，所以现在让我们添加另一个条目以使此示例更具说明性。

在另一个控制台会话中，使用 Express 运行你的本地 Node.js 服务器：

```js
$ node server.js
App now listening on port 8080

```

现在在浏览器中转到 localhost:8080，并使用 POST 请求表单添加另一个条目到数据库：

```js
First name: Peebo 
Last name: Sanderson 
Title: Singer 

```

添加额外记录后，点击 GET 请求表单下的“加载用户数据”链接。你应该会看到以下类似的输出：

```js
Philip Klauzinski 
Sr. UI Engineer 

Peebo Sanderson 
Singer 

```

此内容是通过在`users.hbs`中循环遍历 MongoDB 中的用户数据并填充`user.hbs`部分中的表达式来渲染的。

## 整合所有内容

使用单个、压缩的 JavaScript 文件作为你的应用程序代码，预编译你的 JavaScript 模板，并在应用程序布局页面的底部加载 JavaScript，这些都是优化你的 SPA 下载时间的好做法。将所有 JavaScript 包含在一个文件中而不是多个文件中，与压缩 JavaScript 一样重要，因为它减少了客户端为加载你的 SPA 而必须发出的 HTTP 请求的数量。同样的做法也应该用于 CSS，并且可以使用 Grunt 和插件如`grunt-contrib-cssmin`和`grunt-postcss`来实现。

# 摘要

你现在应该对一些流行的 JavaScript 模板引擎之间的区别有了很好的理解，包括如何使用它们进行基本视图，以及它们的一些优缺点。你还应该了解使用预编译模板和浏览器中编译的模板之间的区别。此外，你已经学习了在布局文件中使用的优化方法，以最小化应用程序的下载大小，包括压缩、合并成一个文件，以及在文档底部包含 JavaScript。在下一章中，我们将通过分解数据绑定技术来进一步深入到视图层。
