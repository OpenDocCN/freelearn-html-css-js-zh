# 第一章. Meteor 入门

即使编程速度较慢，Meteor 也是一个本质上快速的开发框架。本书的目的是提高你的开发速度并提高质量。提高开发速度的两个关键要素是编译器和模式。编译器为你的编程语言添加功能，而模式则增加了解决常见编程问题的速度。

本书将主要涵盖模式，但我们将使用本章快速入门编译器，并了解它们如何与 Meteor 相关——这是一个庞大但简单的主题。我们将查看的编译器如下：

+   CoffeeScript

+   Jade

+   笔记本

我们将回顾一些关于 Meteor 你应该具备的基本知识。这包括以下内容：

+   模板、辅助函数和事件

+   事件循环和合并框

+   必备的包

+   文件夹结构

# Meteor 的 CoffeeScript

CoffeeScript 是一个 JavaScript 编译器，它受到 Ruby、Python 和 Haskell 的启发，添加了“语法糖”，这使得 JavaScript 的编写更容易、更易读。CoffeeScript 简化了函数、对象、数组、逻辑语句、绑定、管理作用域等语法。所有 CoffeeScript 文件都保存为 `.coffee` 扩展名。我们将涵盖函数、对象、逻辑语句和绑定，因为这些是使用最频繁的功能。

## 对象和数组

CoffeeScript 去掉了花括号(`{}`)、分号(`;`)和逗号`,`。仅此一项就可以让你在键盘上避免重复不必要的按键。相反，CoffeeScript 强调正确使用**缩进**。缩进不仅会使你的代码更易读，而且将是代码正常工作的关键因素。实际上，你可能已经正确地使用了缩进！让我们看看一些例子：

```js
#COFFEESCRIPT
toolbox =
  hammer:true
  flashlight:false
```

### 提示

**下载示例代码**

你可以从[`www.packtpub.com`](http://www.packtpub.com)下载示例代码文件，这是你购买的所有 Packt 出版物的链接。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

在这里，我们创建了一个名为 `toolbox` 的对象，它包含两个键：`hammer` 和 `flashlight`。在 JavaScript 中的等效代码将是这样的：

```js
//JAVASCRIPT - OUTPUT
var toolbox = {
  hammer:true,
  flashlight:false
};
```

更简单！正如你所见，我们必须**缩进**来表示 `hammer` 和 `flashlight` 属性都是 `toolbox` 的一部分。在 CoffeeScript 中不允许使用 `var` 关键字，因为 CoffeeScript 会自动为你应用它。让我们看看我们如何创建一个数组：

```js
#COFFEESCRIPT
drill_bits = [
  "1/16 in"
  "5/64 in"
  "3/32 in"
  "7/64 in"
]

//JAVASCRIPT – OUTPUT
var drill_bits;
drill_bits = ["1/16 in","5/64 in","3/32 in","7/64 in"];
```

在这里，我们可以看到我们不需要任何逗号，但我们需要有括号来确定这是一个数组。

## 逻辑语句和运算符

CoffeeScript 在逻辑语句和函数中移除了很多括号(`()`)，这使得代码的逻辑在第一眼看起来更容易理解。让我们看看一个例子：

```js
#COFFEESCRIPT
rating = "excellent" if five_star_rating

//JAVASCRIPT – OUTPUT
var rating;

if(five_star_rating){
  rating = "excellent";
}
```

在这个例子中，我们可以清楚地看到 CoffeeScript 更易于阅读和编写。CoffeeScript 有效地替换了任何逻辑语句中的所有 **隐式括号**。

将 `&&`、`||` 和 `!==` 等运算符替换为单词，可以使代码更易读。以下是您将最常使用的运算符列表：

| CoffeeScript | JavaScript |
| --- | --- |
| `is` | `===` |
| `isnt` | `!==` |
| `not` | `!` |
| `and` | `&&` |
| `or` | `&#124;&#124;` |
| `true, yes, on` | `true` |
| `false, no, off` | `false` |
| `@, this` | `this` |

让我们看看一个稍微复杂一点的逻辑语句，并看看它是如何编译的：

```js
#COFFEESCRIPT
# Suppose that "this" is an object that represents a person and their physical properties

if @eye_color is "green"
  retina_scan = "passed"
else
  retina_scan = "failed"

//JAVASCRIPT - OUTPUT
if(this.eye_color === "green"){
  retina_scan = "passed";
} else {
  retina_scan = "failed";
}
```

注意 `this` 的上下文是如何传递给 `@` 符号而不需要点号的，这使得 `@eye_color` 等于 `this.eye_color`。

## 函数

JavaScript 函数是一段代码块，用于执行特定任务。JavaScript 有几种创建函数的方法，在 CoffeeScript 中得到了简化。它们看起来像这样：

```js
//JAVASCRIPT
//Save an anonymous function onto a variable
var hello_world = function(){
  console.log("Hello World!");
}

//Declare a function
function hello_world(){
  console.log("Hello World!");
}
```

CoffeeScript 使用 `->` 而不是 `function()` 关键字。以下示例输出一个 `hello_world` 函数：

```js
#COFFEESCRIPT
#Create a function
hello_world = ->
  console.log "Hello World!"

//JAVASCRIPT - OUTPUT
var hello_world;
hello_world = function(){
  return console.log("Hello World!");
}
```

再次，我们将使用制表符来指定函数的内容，这样就不需要花括号 (`{}`) 了。这意味着你必须确保整个函数的逻辑都缩进在其命名空间下。

我们的参数怎么办？我们可以使用 `(p1,p2) ->`，其中 `p1` 和 `p2` 是参数。让我们让我们的 `hello_world` 函数输出我们的名字：

```js
#COFFEESCRIPT
hello_world = (name) ->
  console.log "Hello #{name}"

//JAVSCRIPT – OUTPUT
var hello_world;
hello_world = function(name) {
  return console.log("Hello " + name);
}
```

在这个例子中，我们可以看到参数是如何放在括号内的。我们还在做 **字符串插值**。CoffeeScript 允许程序员通过使用 `#{}` 转义字符串来轻松地将逻辑添加到字符串中。注意，与 JavaScript 不同，你不需要在函数末尾返回任何内容，CoffeeScript 会自动返回最后一条命令的输出。

## 绑定

在 Meteor 中，我们经常会发现自己需要在嵌套函数和回调中使用 `this` 的属性。**函数绑定** 在这些情况下非常有用，并有助于避免在额外的变量中保存数据。函数绑定将函数内部的 `this` 对象的值设置为函数外部的 `this` 的值。让我们看一个例子：

```js
#COFFEESCRIPT
# Let's make the context of this equal to our toolbox object
# this =
#   hammer:true
#   flashlight:false

# Run a method with a callback
Meteor.call "use_hammer", ->
  console.log this
```

在这种情况下，`this` 对象将返回一个顶层对象，例如浏览器窗口。这毫无用处。现在让我们绑定 `this`：

```js
#COFFEESCRIPT
# Let's make the context of this equal to our toolbox object
# this =
#   hammer:true
#   flashlight:false

# Run a method with a callback
Meteor.call "use_hammer", =>
  console.log this
```

关键区别在于使用 `=>` 而不是预期的 `->` 来定义函数。使用 `=>` 将使回调的 `this` 对象等于执行函数的上下文。生成的编译脚本如下：

```js
//JAVASCRIPT
Meteor.call("use_hammer", (function(_this) {
  return function() {
    return Console.log(_this);
  };
})(this));
```

CoffeeScript 将提高你的编码质量和速度。尽管如此，*CoffeeScript 并非完美无瑕*。当你开始将函数与嵌套数组结合使用时，事情可能会变得复杂且难以阅读，尤其是在函数使用多个参数构建时。让我们看看一个看起来不像你预期的那样易于阅读的常见查询：

```js
#COFFEESCRIPT
People.update
  sibling:
    $in:["bob","bill"]
,
  limit:1
  ->
    console.log "success!"
```

这个集合查询传递了三个参数：`filter` 对象、`options` 对象和回调函数。为了区分前两个对象，我们不得不在函数同一级别放置一个逗号，然后缩进第二个参数。这很麻烦，但我们可以使用变量来使查询更易读：

```js
#COFFEESCRIPT
filter =
  sibling:
    $in:["bob","bill"]
options =
  limit:1
People.update filter, options, ->
  console.log "success!"
```

前往 [coffeescript.org](http://coffeescript.org)，通过点击“try coffeescript”链接来尝试使用该语言。

# Jade for Meteor

Jade 与 CoffeeScript 工作方式相似，但它用于 HTML 而不是其他。我建议您安装 `mquandalle:jade` 包。所有的 Jade 文件都保存为 `.jade` 扩展名。本节将涵盖 Jade 在 Meteor 中最常用的方面，例如 HTML 标签、组件和助手。

## HTML 标签

与 CoffeeScript 类似，Jade 是一种高度依赖缩进的编程语言。当您想向 HTML 标签添加子元素时，只需使用缩进。可以使用 CSS 选择器表示法（`'input#name.first'`）添加标签 ID 和类。这意味着类用点（`.`）表示，ID 用井号（`#`）表示。让我们看一个例子：

```js
//- JADE
div#container
  ul.list
    li(data-bind="clickable") Click me!

<!-- HTML – OUTPUT -->
<div id="container">
  <ul class="list">
    <li data-bind="clickable">Click me!</li>
  </ul>
</div>
```

如您所见，**特殊**属性如 `data-bind` 使用括号添加。符号如 `<`、`>` 和 **闭包**不再需要。在这个例子中，我们有一个 `div` 标签，其 `id` 属性为 `"container"`，一个 `ul` 标签，其 `class` 属性为 `list`，以及一个具有特殊属性 `data-bind` 的 `li` 标签。

您会发现自己在 `input` 标签上经常使用特殊属性来添加 `value`、`placeholder` 和其他属性。

## 模板和组件

Meteor 模板是 Jade 组件。在 Meteor 中，我们使用模板标签定义模板，并应用特殊的 `name` 属性来创建可重用的 HTML 块。在 Jade 中，当我们创建模板时，我们也创建了一个组件。这看起来如下：

```js
//- JADE
template(name="landing")
  h3 Hello World!

<!-- HTML – OUTPUT -->
<template name="landing">
  <h3>Hello World!</h3>
</template>
```

现在，我们可以在视图的任何地方使用这个模板作为 Jade 组件。要调用 Jade 组件，您只需在模板名称前加上一个加号。让我们看看一个例子，我们想在 `main_layout` 页面内放置一个 `landing` 页面：

```js
//- JADE
template(name="landing")
  h3 Hello World!

template(name="main_layout")
  +landing

<!-- HTML – OUTPUT -->
<template name="landing">
  <h3>Hello World!</h3>
</template>

<template name="main_layout">
  {{> landing}}
</template>
```

就这些！请注意，我们在模板名称前加上了加号 (`+`) 来调用它。这相当于在 SpaceBars（Meteor 的 Handlebars 版本）中使用 `{{> landing}}`。组件也可以有参数，这些参数可以在模板实例中使用。让我们通过一个例子来输出某人的名字：

```js
//- JADE
template(name="landing")
  h3 Hello {{name}}

template(name="main_layout")
  +landing(name="Mr Someone")

# COFFEESCRIPT
Template.landing.helpers
  "name": ->
    Template.instance().data.name

<!-- HTML – OUTPUT -->
<template name="landing">
  <h3>Hello {{name}}</h3>
</template>

<template name="main_layout">
  {{> landing name="Mr Someone"}}
</template>
```

向您的模板添加属性可以使您的模板像前面的例子那样灵活。尽管如此，您不太可能需要这样做，因为模板“吸收”来自其父上下文的数据。

## 助手

Meteor 中的辅助工具是函数，在渲染到视图之前返回数据。我们使用辅助工具进行迭代、逻辑语句和变量。两个基本辅助工具是 `each` 和 `if`，但添加 `raix:handlebar-helpers` 包将添加其他有用的辅助工具字典，以避免代码重复。让我们看看我们如何使用我们的辅助工具：

```js
//- JADE
template(name="list_of_things")
  each things
    if selected
      p.selected {{name}}
    else
      p {{name}}

<!-- HTML – OUTPUT -->
<template name="list_of_things">
  {{#each things}}
    {{#if selected}}
      <p class="selected">{{name}}</p>
    {{else}}
      <p>{{name}}</p>
    {{/if}}
  {{/each}}
</template>
```

在这个例子中，`each` 辅助工具正在遍历另一个名为 `things` 的辅助工具的返回值，如果 `selected` 辅助工具解析为 `true`，则我们将渲染 `p.selected` 并使用 `name` 变量。

重要的是要理解，所有不是 HTML 标签的内容都是辅助工具，如果你想在标签内使用辅助工具，你需要使用 `{{}}` 或 `#{}` 来表示这一点。

前往 [jade-lang.com](http://jade-lang.com) 和 `handlebars.js` 了解更具体的信息。有了这些信息，你应该能够做任何事情。

# Stylus for Meteor

Stylus 与 CoffeeScript 和 Jade 类似，但它用于 CSS。我建议您安装 `mquandalle:stylus`。这个包预装了有用的工具，如 `Jeet` 和 `Rupture`。所有 Stylus 文件都保存为 `.styl` 扩展名。关于 Stylus，我们只需要了解三件事：CSS 标签、变量和函数。

## CSS 标签

Stylus 是一种语言，它通过使用缩进来代替分号（`;`）和大括号（`{}`），来消除对分号和大括号的需求。让我们看看一个例子：

```js
// STYLUS
// Let's make a vertical positioning class
.vertical-align-middle
  //PART 1
  position:absolute
  width:100%
  height:100%
  display:table
  overflow-x:hidden

  .special
    background:black
```

我们可以在 `PART 1` 中看到，如何通过缩进属性来定义类的属性，使用 `.special` 选择具有 `special` 类的 HTML 标签，该标签是 `vertical-align-middle` 类的子类。让我们看看 `PART 1` 是如何编译的：

```js
/* CSS – OUTPUT PART 1 */
.vertical-align-middle {
  position:absolute;
  width:100%;
  height:100%;
  display:table;
  overflow-x:hidden;
}
.vertical-align-middle .special {
  background:black;
}
```

现在让我们添加一个更复杂的选择器：

```js
// STYLUS
// Let's make a vertical positioning class
.vertical-align-middle
  //PART 1
  ...

  //PART 2
  > *
    display:table-cell
    vertical-align:middle
```

`PART 2` 结合了特殊的 CSS2 选择器：特定父级（`>`）和所有元素（`*`）。在这个特定的顺序中，CSS2 选择器只选择“任何第一个兄弟元素”并应用规则。让我们看看 `PART 2` 是如何编译的：

```js
/* CSS – OUTPUT PART 2 */
.vertical-align-middle > * {
  display:table-cell;
  vertical-align:middle;
}
```

让我们在当前类中添加一个新的类，使对象对齐到顶部：

```js
// STYLUS
// Let's make a vertical positioning class
.vertical-align-middle
  //PART 1
  ...
  //PART 2
  ...
  //PART 3
  &.whole-page
    top:0
```

`PART 3` 使用连字符（`&`）来描述一个不是子元素而是与额外类连接的元素。让我们看看 `PART 3` 是如何编译的：

```js
/* CSS – OUTPUT PART 3 */
.vertical-align-middle.whole-page {
  top:0;
}
```

## 变量

与 CSS 不同，Stylus 支持变量。当我们想要对网站的外观进行重大更改时，这可以使许多事情变得可管理。假设我们有两个颜色想要在整个网站上使用，但我们知道这些颜色将会改变。让我们将它们定义为变量，这样我们就可以轻松地在以后修改它们：

```js
// STYLUS
primary-color = #ffffff
$secondary-color = #333333

.text-default
  color:primary-color
  background:$secondary-color

.text-inverted
  color:$secondary-color
  background:primary-color
```

简单吗？在这个例子中，`primary-color` 和 `$secondary-color` 都是变量。Stylus 可选支持使用货币符号（`$`）来表示变量。这可以使查找变量变得更加容易。

## 函数/混合

与 CSS 不同，Stylus 也支持函数。LESS、Stylus 和 **Sassy CSS**（**SCSS**）将函数称为 **混入**。函数会使你的 CSS 混合物更容易在整个项目中共享。我们将介绍 Stylus 中的两种混入类型：混入和透明混入。

混入是接受一组定义参数的函数。让我们看看我们如何编写一个混入：

```js
// STYLUS
animation(duration,delay,timing)
  -webkit-animation-duration:duration
  animation-duration:duration
  -webkit-animation-delay:delay
  animation-delay:delay
  -webkit-animation-timing-function:timing
  animation-timing-function:timing

button
  animation(500ms,0,ease-out)

/* CSS – OUTPUT */
button {
  -webkit-animation-duration:500ms;
  animation-duration:500ms;
  -webkit-animation-delay:0;
  animation-delay:0;
  -webkit-animation-timing-function:ease-out;
  animation-timing-function:ease-out;
}
```

在这个例子中，我们首先定义了 `animation` 混入，然后我们将混入应用到 `button` HTML 标签上。然而，有一种更简单、更有效的方法是通过透明混入来完成这项工作。

透明混入基本上会接受所有参数并将它们保存在一个 `arguments` 变量中，而无需你定义任何内容。让我们看看：

```js
// STYLUS
animation()
  -webkit-animation:arguments
  animation:arguments

button
  animation(pulse 3s ease infinite)

/* CSS – OUTPUT */
button {
  -webkit-animation:pulse 3s ease infinite;
  animation:pulse 3s ease infinite;
}
```

注意我们不需要在混入中定义每个参数，`arguments` 变量简单地传递了它能找到的所有参数。这对于保持代码的灵活性特别有用。

Stylus 实质上以升级 CSS 的方式，使得代码更容易管理，因此，最终为我们节省了大量开发时间。

前往 [stylus-lang.com](http://stylus-lang.com) 和 [learnboost.github.io/stylus](http://learnboost.github.io/stylus) 了解更多关于 Stylus 的信息。

# 模板、辅助函数和事件

既然我们对本书中将要使用的语言达成了一致，让我们快速回顾一下我们在开发过程中将使用的一些元素。

模板、辅助函数和事件用于构建应用程序的前端。有效地使用它们是我们设计后端的关键（我们将在 第二章，*发布和订阅模式*）中讨论）。

## 模板

Meteor 模板是特殊的 HTML 代码块，用于生成 Meteor 模板对象（`Template.<yourtemplate>`）。正是通过 Meteor 模板对象，我们将 HTML 代码与逻辑连接起来。使用过 MVC 框架的人会将这些模板称为视图。这是一个需要理解的关键概念。

打开你的终端并创建一个新的项目：

```js
meteor create basic_meteor

```

现在让我们添加我们的语言：

```js
meteor add coffeescript
meteor add mquandalle:jade
meteor add mquandalle:stylus

```

从 `/basic_meteor` 中删除三个可见文件（不要删除以点开头的任何文件），并创建 `/client/layout.jade`。这是每个 Meteor 项目以某种方式存在的东西。让我们开始编程：

```js
//- layout.jade
head
  title Meteor Basics
  meta(name="viewport" content="user-scalable=no, initial- scale=1.0, maximum-scale=1.0")
  meta(name="apple-mobile-web-app-capable" content="yes")
body
  +basic_template
```

```js
basic_template. Let's program this in a new file, /client/basic_template.jade:
```

```js
//- basic_template.jade
template(name="basic_template")
  h1 Hello World!
```

在幕后，Meteor 正在编译我们的 Jade 模板，并将它们全部放入一个大的文件中。在模板方面，你永远不必担心在加载 `layout.jade` 之前加载 `basic_template.jade`。

在整本书中，我们将使用 `meteorhacks:flow-router` 和 `meteorhacks:flow-layout` 来轻松导航到不同的模板。

## 创建辅助函数

我们已经讨论了 Jade 中的助手是什么，但在 Meteor 中如何创建助手呢？让我们回到我们的 `basic_meteor` 项目，并创建 `/client/basic_template.coffee`。重要的是要理解，Meteor 助手用于控制模板中的变量。那些使用过 MVC 框架的人可以将这个文件视为一个控制器。让我们编写我们的第一个助手：

```js
#basic_template.coffee
Template.basic_template.helpers
  "name": ->
    "Mr Someone"
```

注意，助手是在 Meteor 模板对象的 `helpers` 函数中定义的：`Template.<your_template>.helpers(<your_helpers>)`。助手主要是返回任何你想要它们返回的内容的函数，包括 Meteor 集合游标。现在让我们把这些放在一起：

```js
//- basic_template.jade
template(name="basic_template")
  h1 Hello {{name}}
```

这将在 `h1` HTML 标签内输出 `Hello Mr Someone`。让我们添加一个稍微复杂一些的助手：

```js
#basic_template.coffee
Template.basic_template.helpers
  "person": ->
    name:"Someone"
    prefix: "Mr"
    children: [
      {
        name:"Billy"
      }
      {
        name:"Nancy"
      }
    ]

//- basic_template.jade
template(name="basic_template")
  with person
    h1 Hello {{prefix}} {{name}}

    ul
      each children
        li {{name}}
```

在这个例子中，我们使用 `with` 来设置属于它的 HTML 标签的 **数据上下文**；这个数据上下文等同于 `person`。数据上下文指的是助手内部 `this` 的值。所以如果你设置一个对象作为数据上下文，`this` 将等同于那个对象。此外，我们使用 `each` 语句遍历 `children`，以便列出它们的名称。

## 事件

Meteor 利用常见的 JavaScript HTML 事件，如点击、更改和焦点。事件是任何发生在你正在监听 HTML 元素上的事情。假设我们想要能够通过点击它们来将一个人的名字更改为其中一个孩子的名字。我们通过模板的事件映射来完成这个操作。让我们看看一个例子，说明我们如何在不使用响应性或集合的情况下完成这个操作：

```js
#basic_template.coffee
Template.basic_template.events
  "click li": ->
    $("h1").text "Hello #{@name}"
```

简单！为了捕获模板事件，我们需要使用 `Template.<your_template>.events(<your_event_map>)` 函数。在这个特定的例子中，我们使用 jQuery 来替换文本。

`event map` 是一个对象，其中属性指定了一组要处理的事件。这些事件可以以下任何一种方式指定：

```js
# Runs any time you click anywhere
"click": ->

# Runs any time you click a li element
"click li": ->

#Runs any time you click a li element OR mouseover a li element
"click li, mouseover li": ->
```

事件的关键 `string` 由两部分组成：第一部分始终是事件类型（点击、悬停、更改等），而第二部分始终是 CSS 选择器。

# 事件循环和合并框

在深入研究 Meteor 之前，理解事件循环和合并框是什么，以及它们如何对你的代码产生不利影响是至关重要的。两者在编程方式上都是相对复杂的，所以我们将会关注理解一般概念。

## 事件循环

事件循环就像一个队列；它依次运行一系列函数。因为函数是顺序处理的，所以每个函数实际上阻止其他函数在完成之前被处理。

换句话说，事件循环的功能就像一个单行传送带，其中事物正在被检查。对于每次检查，生产线都会停止，什么也不移动。

Meteor 使用 Fibers – 一个 NodeJS 库 – 来解决这个问题。你将运行的大多数函数将在一个单独的纤维上运行。这意味着函数将在一个单独的处理传送带上运行。尽管如此，并非所有函数都是这样构建的，你需要确保你的服务器端函数不会阻止服务器。

那么，哪些函数可能会使服务器被阻止？`Meteor.methods()`、`Meteor.publish()` 以及任何不在服务器纤维内运行的函数。让我们看看我们如何解除每个函数的限制，以及何时应该这样做。

你知道在 `Meteor.methods()` 下定义的函数将需要很长时间来处理，应该始终在纤维上运行或在纤维上延迟耗时的代码。我们可以通过在方法内部调用 `@unblock()` 函数来快速解决这个问题。让我们看看一个例子：

```js
# METEOR METHODS
Meteor.methods
  #BLOCKING
  time_consuming: ->
    Meteor.setTimeout ->
        console.log "done"
      ,60000

  #NON-BLOCKING
  time_consuming_unblock: ->
    @unblock()
    Meteor.setTimeout ->
        console.log "done"
      ,60000
```

在这个例子中，当你运行 `Meteor.call("time_consuming")` 时，服务器将被阻止。当服务器被阻止时，其他访客将无法访问你的网站！相反，如果你运行 `Meteor.call("time_consuming_unblock")`，服务器将继续正常工作，但会消耗更多资源来这样做。

安装了 `meteorhacks:unblock` 包后，`Meteor.publish()` 也可以轻松解除限制。当我们要开始创建可能消耗大量资源的非常复杂的发布者时，这个包将特别有用。让我们看看一个例子：

```js
# METEOR PUBLISH
#BLOCKING
Meteor.publish "external_API_query", ->
  HTTP.get "http://connect.square.com/payments"

#NON-BLOCKING
Meteor.publish "external_API_query_unblocked", ->
  @unblock()
  HTTP.get "http://connect.square.com/payments"
```

在这个例子中，我们正在等待一个 HTTP 调用响应。如果我们订阅了 `external_API_query`，这肯定会阻止服务器，所以我们使用 `external_API_query_unblocked` 代替。

所有其他在服务器上运行且你知道会阻止服务器的函数，都应该在纤维上运行。Meteor 有一个特殊的功能帮助我们轻松实现这一点。它被称为 `Meteor.wrapAsync()`。让我们看看它是如何工作的：

```js
# METEOR UNBLOCKED FUNCTION
unblock_me = Meteor.wrapAsync ->
  Meteor.setTimeout ->
      console.log "done"
    ,60000
```

### 小贴士

记住事件循环非常重要，尤其是在我们将我们的网络应用程序连接到将导致服务器出现巨大延迟的外部服务时。

## 合并框

合并框是识别数据库中所有变化的算法。它基本上处理发布者、订阅者和反应性。合并框还使用 DDP 消息处理数据的初始加载。

重要的是要理解，我们可以通过在 `Meteor.publish()` 函数下可用的所有命令直接与合并框进行通信。我们能使我们的 `Meteor.publish` 函数更加优化，网站加载速度就会更快。

# 我们在线商店的开始

在整本书中，我们将开发一个电子商务网站，以帮助我们理解高级 Meteor 网络开发的核心概念。让我们从创建一个新的项目开始：

```js
meteor create online_shop

```

## 必备的包

[Atmospherejs.com](http://Atmospherejs.com) 一直是寻找包的“必去”网站。在这里，你可以找到社区免费提供的数千个包。有一些包我们绝对需要安装，以使我们的网站正常工作。

首先，我们安装语言：

```js
meteor add coffeescript
meteor add mquandalle:jade
meteor add mquandalle:stylus
meteor add less

```

接下来，我们将介绍路由和帮助我们的 SEO 和路由的函数：

```js
meteor add kadira:flow-router
meteor add kadira:blaze-layout
meteor add meteorhacks:fast-render
meteor add nimble:restivus
meteor add yasinuslu:blaze-meta
meteor add dfischer:prerenderio
meteor add wizonesolutions:canonical

```

### 注意

**警告**：不要运行 Meteor！除非你正确设置了 canonical，否则它可能会破坏你的项目。

我们还需要一些包来帮助我们管理发布者：

```js
meteor add lepozepo:publish-with-relations
meteor add tmeasday:publish-counts
meteor add meteorhacks:aggregate
meteor add http
meteor add meteorhacks:unblock

```

下一个包将扩展 Meteor 的功能：

```js
meteor add xorax:multiple-callbacks
meteor add aldeed:collection2
meteor add aldeed:autoform
meteor add fastclick
meteor add reactive-var
meteor add alanning:roles
meteor add accounts-password

```

```js
meteor add u2622:persistent-session
meteor add ongoworks:security

```

我们需要这些包来正确管理时间：

```js
meteor add momentjs:moment
meteor add mrt:moment-timezone

```

对于最后一组，我们将使用一些额外的包，这将使设计过程变得更快：

```js
meteor add kyleking:customizable-bootstrap-stylus
meteor add raix:handlebar-helpers
meteor add fortawesome:fontawesome
meteor add percolate:momentum

```

我们还需要移除一些包以增强安全性：

```js
meteor remove autopublish
meteor remove insecure

```

这些包将在本书的后续章节中详细介绍，但这些都是必需的。我们需要解释的第一个包是 `wizonesolutions:canonical` 包。这个包确保所有进入的流量都被路由到你的 `ROOT_URL`，因此当你希望所有流量都流向你的 SSL 网站时特别有用。在运行 Meteor 之前，我们需要做的第一件事是设置 canonical 以只在生产环境中运行。

创建 `/server/canonical.coffee` 文件，并添加以下代码：

```js
#/server/canonical.coffee
if process.env.NODE_ENV is "development" or process.env.ROOT_URL.indexOf("meteor.com") > -1
  Meteor.startup ->
    process.env.PACKAGE_CANONICAL_DISABLE = true
```

```js
PACKAGE_CANONICAL_DISABLE environment variable to make sure that canonical is inactive while you are developing.
```

环境变量是什么？这些变量是在部署的范围内定义的，并且确保在服务器上的构建完成之前项目知道这些信息。例如，使用哪个数据库、使用哪个域名以及其他设置信息通常可以在这类变量中找到。我们将在最后一章中介绍这些信息。

## 文件结构

在 Meteor 中，一个合适的文件结构非常重要。我们发现，最佳的工作方式是与功能顶级模块一起工作。这意味着每个文件夹都是一个微服务，因此可以独立运行。这为项目提供了很多模块化，并且其他人很容易理解你试图实现什么。在本节中，我们将介绍这个文件结构和 Meteor 的特殊文件夹。

让我们看看一个示例网络应用程序的文件夹结构：

```js
/online_shop
/online_shop/cart
/online_shop/cart/cart_route.coffee  #RUNS ON CLIENT AND SERVER
/online_shop/cart/client
/online_shop/cart/client/cart_view.jade  #RUNS ON CLIENT ONLY
/online_shop/cart/client/cart_controller.coffee
/online_shop/cart/client/cart.styl
/online_shop/cart/server
/online_shop/cart/server/cart_publisher.coffee  #RUNS ON SERVER ONLY
/online_shop/cart/server/cart_methods.coffee
```

在这个文件夹结构中，`cart` 是一个微服务，它由路由、视图、控制器和发布者组成。放在 `/client` 目录下的文件将被发布到客户端，并且只会在客户端运行。放在 `/server` 目录下的文件只会在服务器上运行和访问。如果一个文件放在这些目录之外，那么这个文件将在客户端和服务器上同时运行。预期的结构如下：

```js
/project_folder
/project_folder/_globals
  ./client/<global support function files>
  ./server/<global support function files>
  ./lib/collections/<collection>/<file>
  ./lib/collections/<collection>/server/<permissions file>

/project_folder/router
  ./client/layouts/<layout file>
  ./lib/<configuration file>
  ./lib/<middleware file>

/project_folder/<module>
  ./<route file>
  ./client/<template file>
  ./client/<logic file>
  ./client/<styles file>
  ./server/<publishers file>
  ./server/<methods file>
```

需要注意的是，`/lib` 目录将始终在所有其他代码之前运行。让我们将我们的规范文件放在 `/_globals/canonical/server` 目录下。

让我们创建我们的第一个模块：路由器。创建`/router/client/layout.jade`目录，在整个项目中我们只有一个布局。现在让我们编写我们的布局代码：

```js
//- LAYOUT.JADE
head
  title Online Shop
  meta(charset="utf-8")

  //- Allow saving to homescreen
  meta(name="apple-mobile-web-app-capable" content="yes")

  //- Do not try to detect phone numbers
  meta(name="format-detection" content="telephone=no")

  //- Make it mobile friendly
  meta(name="viewport" content="user-scalable=no, initial- scale=1.0, maximum-scale=1.0")

body

template(name="layout")
  +Template.dynamic(template=nav)
  +Template.dynamic(template=content)
```

在这里，我们介绍了`Template.dynamic`组件。该组件可以通过更改变量的值来动态渲染其他模板，该值是我们想要渲染的模板的名称。我们决定使用两个变量——`nav`和`content`——这些变量由路由器控制。所以，基本上，`content`变量将改变为不同的字符串，这些字符串等于我们模板的名称。

我们将在下一章中创建我们的`着陆`模块，以便学习不仅如何使用路由器，而且如何正确地订阅数据。

# 摘要

在本章中，我们已经解决了很多问题。现在我们可以更快地编程，因为我们有了像 CoffeeScript、Jade 和 Stylus 这样的工具来帮助我们。此外，我们还学会了如何使用模板、辅助工具和事件来处理我们的 Meteor 前端。理解事件循环和合并框使我们处理复杂、耗时操作时更加谨慎。最后，我们开始构建项目，并采用了一种将使开发更快的项目文件夹结构。

在下一章中，我们将介绍使 Meteor 应用程序可行的两个最重要的部分：Meteor 发布者和 Meteor 订阅者。使用这些模式，您将能够创建快速加载且不会对服务器造成过多压力的网站。
