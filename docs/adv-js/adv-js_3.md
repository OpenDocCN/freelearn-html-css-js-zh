# 第三章：DOM 操作和事件处理

## 学习目标

在本章结束时，您将能够做到以下几点：

+   解释 DOM 遍历和操作

+   创建事件对象和浏览器事件

+   组织事件传播和冒泡

+   高效地委托事件

+   利用 jQuery 处理事件和 DOM 操作

本章将涵盖处理文档节点、事件对象以及链式、导航和处理事件的过程。

## 介绍

在第一章中，我们涵盖了 ES6 中发布的许多新的强大功能。我们讨论了 JavaScript 的发展，并突出了 ES6 中的关键新增功能。我们讨论了作用域规则、变量声明、箭头函数、模板文字、增强对象属性、解构赋值、类和模块、转译以及迭代器和生成器。

在第二章中，我们涵盖了 JavaScript 的异步编程范式。我们讨论了 JavaScript 事件循环、回调、承诺和 async/await 语法。本章使我们能够应用*第一章，介绍 ECMAScript 6*中的材料，并编写强大的异步程序。

在本章中，我们将学习**文档对象模型（DOM）**和**JavaScript 事件对象**。在第一个主题中，我们将定义文档对象模型并解释 DOM 链式、导航和操作。然后，我们将解释 JavaScript 事件对象，并展示如何与处理 DOM 事件进行交互。在本章中，我们将涵盖 jQuery，并使用它来遍历 DOM 和处理事件。

## DOM 链式、导航和操作

**文档对象模型（DOM）**是 HTML 文档的接口。DOM 以一种程序可以更改文档结构、样式和内容的方式来表示网页。DOM 是网页的面向对象表示。

DOM 有两个标准：**万维网联盟（W3C）**标准和**Web 超文本应用技术工作组（WHATWG）**标准。WHATWG 是为了应对 W3C 标准的缓慢发展而开发的。这两个标准都将 HTML 元素定义为可以被 JavaScript 代码访问的对象，并为所有 HTML 元素定义了属性、访问器方法和事件。DOM 对象方法是您可以在 HTML 元素上执行的操作，DOM 对象属性是您可以获取或设置的值。DOM 标准提供了一种让 JavaScript 以编程方式添加、获取、更改或删除 HTML 元素的方法。

#### 注意

W3C DOM 标准和 WHATWG DOM 标准由大多数现代浏览器（Chrome、Firefox 和 Edge）实现，并且许多浏览器扩展了这些标准。在与 DOM 进行交互时，我们必须确保我们使用的所有函数与我们的用户可能使用的浏览器兼容。

网页的 DOM 构造为对象树，称为**节点**。树的顶部对象是**文档节点**。**文档**是作为网页内容、DOM 树的入口点的接口。页面中的 HTML 元素被添加到文档下的 DOM 树中。它们被称为**元素节点**。

DOM 树中的元素与其周围元素有三种类型的关系：**父级**、**同级**和**子级**。元素的父元素是包含它的元素。元素的同级节点是同样包含在父元素中的元素。元素的子节点是它包含的元素。以下是一个示例 DOM 树的图示：

![图 3.1：DOM 树结构](img/Figure_3.1.jpg)

###### 图 3.1：DOM 树结构

在前面的图表中，我们可以看到全局父级是**文档对象**。文档对象有一个子节点，即`<html>`元素。`<html>`元素的父节点是文档，它有两个子节点，即`<head>`和`<body>`元素。`<head>`和`<body>`元素是彼此的兄弟节点，因为它们都有相同的父节点。

### 练习 20：从 DOM 树结构构建 HTML 文档

这里的目标是创建一个名为"My title"的网页，显示标题"My header"和链接"My link"。参考前面的图表以获取 DOM 树结构。

要从 DOM 树结构构建 HTML 文档，请执行以下步骤：

1.  创建一个 HTML 文件。

1.  在文件中添加一个`<html>`标签。

1.  在`<html>`标签内添加一个`<head>`标签。

1.  在`<head>`标签后添加一个`<title>`标签。

1.  在`<title>`标签中添加文本**My title**。

1.  在`<head>`标签下方添加一个`<body>`标签。

1.  在`<body>`标签下添加`<a>`和`<h1>`元素。

1.  为`<a>`标签添加`href`属性，并将其内部文本设置为**My link**。

1.  在`<h1>`标签中添加文本**My header**。

1.  关闭`body`和`html`标签并获取输出。

**代码**

##### index.js

```php
<html>
  <head>
    <title>My title</title>
  </head>
  <body>
    <a href>My link</a>
    <h1>My header</h1>
  </body>
</html>
```

https://bit.ly/2FiLgcE

###### 片段 3.1：演示 DOM 树的简单网站

结果

![图 3.2：我的标题链接输出](img/Figure_3.2.jpg)

###### 图 3.2：我的标题链接输出

您已成功从 DOM 树结构构建了 HTML 文档。

### DOM 导航

现在我们了解了 DOM 的基本结构，我们准备在我们的应用程序中开始与它进行接口。在我们可以用 JavaScript 修改 DOM 之前，我们必须导航 DOM 树以找到我们想要修改的特定元素节点。我们可以通过两种方式之一找到特定节点：**通过标识符找到**或**导航 DOM 树**。最快的查找方法是通过标识符查找元素。DOM 元素可以通过以下四种方式之一查找：

+   ID

+   标签名

+   类

+   CSS 查询选择器

### 查找 DOM 节点

通过`document.getElementById( id )`方法可以通过 ID 获取元素。该方法接受一个表示要查找的元素 ID 的参数 id，并返回一个元素对象。返回的对象将是描述指定 ID 的 DOM 节点的元素对象。如果没有匹配提供的 ID 的元素，则该函数将返回 null。以下是`getElementById`函数的示例：

```php
<div id="exampleDiv">Some text here</div>
<script>
  const elem = document.getElementById( 'exampleDiv' );
</script>
```

###### 片段 3.2：通过 ID 获取元素

通过`document.getElementsByTagName( name )`方法可以通过标签名获取元素。该函数接受一个表示要搜索的 HTML 标签名的参数。`getElementsByTagName`返回一个匹配给定标签名的元素的实时`HTMLCollection`。返回的列表是实时的，这意味着它会自动更新与 DOM 树。不需要多次使用相同的元素和参数调用该函数。以下是`getElementsByTagName`的示例：

```php
<div id="exampleDiv1">Some text here</div>
<div id="exampleDiv2">Some text here</div>
<div id="exampleDiv3">Some text here</div>
<script>
  const elems = document.getElementsByTagName( 'div' );
</script>
```

###### 片段 3.3：按标签名获取元素

#### 注意

`HTMLCollection`是表示元素节点集合（类似数组的对象）的接口。它可以被迭代，并提供用于从列表中选择的方法和属性。

要通过类名获取元素，我们使用`document.getElementsByClassName( name )`方法。该函数接受一个表示要搜索的 HTML 类名的参数，并返回一个匹配给定类名的元素的实时`HTMLCollection`。以下是`getElementsByClassName`的示例：

```php
<div class="example">Some text here</div>
<img class="example"></img>
<style class="example"></style>
<script>
  const elems = document.getElementsByClassName( 'example' );
</script>
```

###### 片段 3.4：按类名获取元素

`querySelector()`和`querySelectorAll()`这两个函数用于通过 CSS 查询选择器获取 HTML 元素。它们都接受一个表示 CSS 选择器字符串的单个字符串参数。`querySelector`将返回一个单个元素。`querySelectorAll`将返回与查询匹配的元素的静态（非实时）`NodeList`。可以通过创建包含每个选择器的逗号分隔字符串将多个查询选择器传递给函数。如果将多个选择器传递给查询选择器函数，函数将匹配并返回满足任何选择器要求的元素。`querySelector`和`querySelectorAll`的功能如下片段所示：

```php
<div id="id1">Some text here</div>
<img class="class"></img>
<script>
  const elem = document.querySelector( 'img.class' );
  const elems = document.querySelectorAll( 'img.class, #id1' );
</script>
```

###### 片段 3.5：使用 CSS 选择器获取元素

#### 注意

`NodeList`类似于`HTMLCollection`。它是一个类似数组的 HTML 节点集合，可以进行迭代。

之前介绍的每个方法及其函数语法如下表所示：

![图 3.3：方法和语法](img/Figure_3.3.jpg)

###### 图 3.3：方法和语法

`getElementsByTagName`、`getElementsByClassName`、`querySelector`和`querySelectorAll`函数不仅限于文档对象；它们也可以在元素节点上调用。如果它们在元素节点上调用，函数返回的结果元素集合将仅限于函数调用的元素的子节点。以下示例显示了这一点。

**示例**：我们获取具有 id`div1`的 div 元素对象，并将其保存在`elem`变量中。然后我们使用`getElementsByTagName`来获取其他 div 元素。该函数在保存在`elem`中的元素对象上调用，因此搜索范围仅限于`div1`的子节点。`getElementsByTagName`将返回一个包含 divs`div2`和`div3`的`HTMLCollection`，因为它们是`div1`的后代：

```php
<div id="div1">
  <div id="div2">
    <div> id="div3"> Some text here </div>
  </div>
</div>
<div> Some other text here </div>
<script>
  const elem = document.getElementById( 'div1' );
  const elems = elem.getElementsByTagName( 'div' );
</script>
```

###### 片段 3.6：返回 HTMLCollection

查找 DOM 元素的第二种方法是通过导航 DOM 树来查找元素关系。一旦找到要处理的 DOM 元素，我们可以使用多个属性来获取该元素的子节点、父节点和兄弟节点。我们可以通过使用`parentNode`、`childNodes`、`firstChild`、`lastChild`、`previousSibling`和`nextSibling`属性从一个节点到另一个节点遍历 DOM 树。

`parentNode`属性返回节点的父节点。父节点是 DOM 树中的一个节点，该节点是其后代。父节点始终存在，除非在文档节点上调用`parentNode`。由于文档节点位于 DOM 树的顶部，它没有父节点，调用`parentNode`将返回 null。可以使用`parentNode`属性遍历 DOM 树。以下示例显示了`parentNode`的用法：

```php
<div id="div1">
  <div id="div2">
    <div id="div3"> Some text here </div>
  </div>
</div>
<script>
  const div3 = document.getElementById( 'div3' );
  const div2 = div3.parentNode;
  const div1 = div2.parentNode;
</script>
```

###### 片段 3.7：父节点

`nextSibling`和`previousSibling`属性用于获取 DOM 树中节点的兄弟节点。`previousSibling`将返回 DOM 树中的前一个兄弟节点（添加到当前节点之前的父节点的兄弟节点），`nextSibling`将返回 DOM 树中的下一个兄弟节点（添加到当前节点之后的父节点的兄弟节点）。在绘制 DOM 树时，通常将节点的前一个兄弟节点显示在左侧，下一个兄弟节点显示在右侧。可以使用`nextSibling`和`previousSibling`函数横向遍历 DOM 树。以下示例显示了这些属性：

```php
<div id="div0">
  <div id="div1"> Some text here </div>
  <div id="div2"> Some text here </div>
  <div id="div3"> Some text here </div>
</div>
<script>
  const div2 = document.getElementById( 'div2' );
  const sibling1 = div2.previousSibling; //div1
  const sibling2 = div2.nextSibling; // div3
</script>
```

###### 片段 3.8：遍历兄弟节点

最后三个属性用于导航到节点的子节点；它们是`childNodes`，`firstChild`和`lastChild`。`childNodes`属性返回元素的子节点的实时`NodeList`。`firstChild`和`lastChild`属性分别返回子`NodeList`中的第一个或最后一个节点。以下片段显示了这些属性的使用：

```php
<div id="div0">
  <div id="div1"> Some text here </div>
  <div id="div2"> Some text here </div>
  <div id="div3"> Some text here </div>
</div>
<script>
  const div0 = document.getElementById( 'div0' );
  const child1 = div0.firstChild; //div1
  const child2 = div0.childNodes[1]; // div2
  const child3 = div0.lastChild; // div3
</script>
```

###### 片段 3.9：遍历兄弟节点

### 遍历 DOM

DOM 树导航属性总结如下表：

![图 3.4：DOM 树导航属性](img/Figure_3.4.jpg)

###### 图 3.4：DOM 树导航属性

### DOM 操作

当您编写应用程序或网页时，您拥有的最强大的工具之一是以某种方式操纵文档结构。这是通过 DOM 操作函数来完成的，用于控制 HTML 并为应用程序或页面设置样式。能够在用户使用应用程序或网站时操纵 HTML 文档，使我们能够动态更改页面的部分而无需完全重新加载内容。例如，当您在手机上使用消息应用时，应用的代码正在操纵您正在查看的文档。每次发送消息时，它都会更新文档以附加构成消息的元素和样式。我们可以操纵 DOM 的三种基本方式。我们可以添加元素或节点，删除元素或节点，以及更新元素或节点。

向 DOM 树添加新元素是交互应用程序的必备功能。在您使用的大多数 Web 应用程序中都有许多示例。谷歌的 Gmail 和微软的 Skype 在您使用应用程序时都会主动向 DOM 添加元素。向 DOM 添加新元素有两个步骤。首先，我们必须为要添加的元素创建一个节点，然后我们必须将新节点添加到 DOM 树中。

要创建新元素或节点，我们可以使用`document.createElement()`、`Node.cloneNode()`和`document.createTextNode()`函数。`CreateElement`是在全局文档对象上调用的，并接受两个参数。第一个是`tagName`。**tagName**是一个字符串，指定要创建的元素类型。如果我们想要创建一个新的 div 元素，我们将通过`tagName`传递`div`字符串。第二个参数是一个可选参数，称为 options。Options 是一个包含单个属性的`ElementCreationObject`，名为'is'。此属性允许我们指定要添加的元素是否是自定义元素。我们将不使用此属性，但知道它的用途很重要。`CreateElement`返回一个新创建的 Element 对象。`document.createElement()`的语法和用法如下片段所示：

```php
<script>
  const newElem = document.createElement( 'div' );
</script>
```

###### 片段 3.10：使用 document.createElement

新的元素节点也可以使用`cloneNode`函数创建。`cloneNode`是在 DOM 节点对象上调用的，并复制调用它的节点。它接受一个名为`deep`的布尔值作为参数，并返回要克隆的节点的副本。如果`deep`设置为`false`，`cloneNode`将进行浅克隆，只克隆调用它的节点。如果`deep`设置为`true`，`cloneNode`将进行深度复制，并复制节点及其所有子节点（节点的完整 DOM 树）。克隆节点会复制其所有属性及其值。这包括在 HTML 中内联添加的事件监听器，但不包括通过`addEventListener`用 JavaScript 添加的监听器，或者通过元素属性分配的监听器。

以下是`cloneNode`的示例：

```php
<div id="div1">
  <div id="div2"> Text </div>
</div>
<script>
  const div1 = document.getElementById( 'div1' );
  const div1Clone = div1.cloneNode( false );
  const div1Div2Clone = div1.cloneNode( true )
</script>
```

###### 片段 3.11：克隆节点

在前面的例子中，我们创建了一个包含两个 div 的文档，`div1`和`div2`。`div2`嵌套在`div1`中。在前面的代码中，我们通过 id 选择了`div1`，并通过浅`nodeClone`将其克隆到`div1Clone`中。然后我们进行了深度`nodeClone`，并将`div1`及其嵌套的子元素`div2`克隆到`div1Div2Clone`中。

#### 注意

`cloneNode`可能会导致文档中出现重复的元素 id。如果复制具有 id 的节点，则应更新该节点的 id 属性为唯一值。

DOM 的规范最近已经更新。在 DOM4 规范中，`cloneNode` 的 `deep` 是一个可选参数。如果省略，该方法将默认将值设置为 true，使用深克隆作为默认行为。要创建浅克隆，必须将 `deep` 设置为 false。在最新的 DOM 规范中，此行为已更改。`deep` 仍然是一个可选参数；但是，默认值为 false。我们建议始终提供 `deep` 参数以实现向后和向前兼容性。

`CreateTextNode` 用于创建仅包含文本的节点。当用文本填充页面时，会使用仅包含文本的 DOM 节点。我们使用 `createTextNode` 将新文本放入像 div 这样的元素中。`CreateTextNode` 接受一个参数，一个名为 `data` 的字符串，并返回一个文本节点。`createTextNode` 的示例如下所示：

```php
<script>
  const textNode = document.createTextNode( 'Text goes here' );
</script>
```

###### 片段 3.12：创建文本节点

现在我们知道如何创建新的 DOM 节点，我们必须将新节点添加到 DOM 树中，以便在应用程序中看到更改。我们可以使用两个函数添加新节点：`Node.appendChild()` 和 `Node.insertBefore()`。这两个函数都是在 DOM 节点对象上调用的。

`Node.appendChild` 将节点添加到其调用的节点的子节点列表的末尾。`Node.appendChild` 接受一个参数 `aChild`，并返回附加的子节点。`aChild` 参数是我们要附加到父节点的子节点列表的节点。如果 `appendChild` 传入的是已经存在于 DOM 树中的节点，该节点将从当前位置移动到 DOM 中的新位置，作为指定父节点的子节点。如果 `appendChild` 传入的是 `DocumentFragment`，则 `DocumentFragment` 的整个内容将移动到父节点的子节点列表中，并返回一个空的 Document Fragment。`appendChild` 的语法和用法如下所示：

```php
<div id="div1"></div>
<script>
  const div1 = document.getElementById( 'div1' ); 
  const aChild = document.createElement( 'div' );
  parent.appendChild( aChild );
</script>
```

###### 片段 3.13：使用 appendChild 插入节点

#### 注意

`DocumentFragment` 只是一个没有父节点的 DOM 树。

在前面的示例中，我们创建了一个带有 div `div1` 的 HTML 文档。然后我们创建了一个新的 div `div2`，然后使用 `appendChild` 函数将其附加到 `div1` 的子列表中。

节点还可以使用 `Node.insertBefore()` 函数插入到 DOM 中。`insertBefore` 函数将节点插入到其调用的节点的子节点列表中，位于指定的参考节点之前。`insertBefore` 函数接受两个参数，`newNode` 和 `referenceNode`，并返回插入的节点。`newNode` 参数表示我们要插入的节点。`referenceNode` 参数是父节点的子节点列表中的一个节点或值 `null`。如果 `referenceNode` 是父节点子列表中的一个节点，`newNode` 将插入到该节点之前，但如果 `referenceNode` 是值 `null`，`newNode` 将插入到父节点的子节点列表的末尾。与 `Node.appendChild()` 类似，如果函数给定要插入的节点已经在 DOM 树中，该节点将从其在 DOM 树中的旧位置中移除，并作为父节点的子节点放置在新位置。`InsertBefore` 还可以插入整个 `DocumentFragment`。如果 `newNode` 是 `DocumentFragment`，函数将返回一个空的 `DocumentFragment`。

`appendChild` 的示例如下所示：

```php
<div id="div1">
  <div id="div2"></div>
</div>
<script>
  const div1 = document.getElementById( 'div1' );
  const div2 = document.getElementById( 'div2' );
  const div3 = document.createElement( 'div' );
  const div4 = document.createElement( 'div' );
  div1.insertBefore( div3, div2 );
  div1.insertBefore( div4, null );
</script>
```

###### 片段 3.14：使用 insertBefore 插入节点

在前面的示例中，我们创建了一个带有嵌套子 div `div2` 的 div `div1`。在脚本中，我们通过元素 id 获取了 `div1` 和 `div2`。然后我们创建了两个新的 div，`div3` 和 `div4`。我们将 `div3` 插入到 `div1` 的子列表中。我们将 `div2` 作为参考节点传递，因此 `div3` 被插入到 `div1` 的子列表中 `div2` 的前面。然后我们将 `div4` 插入到 `div1` 的子列表中。我们将 null 作为参考节点传递。这会导致 `div4` 被追加到 `div1` 的子列表的末尾。

#### 注意

`referenceNode`参数不是可选参数。你必须明确传入一个节点或值 null。不同的浏览器和浏览器版本对无效值的解释不同，应用功能可能会受到影响。

操作 DOM 的另一个关键功能是能够从 DOM 树中删除 DOM 节点。这个功能可以在 Gmail 和 Facebook 中看到。当你在 Gmail 中删除一封邮件或删除 Facebook 的帖子时，与该邮件或帖子相关的 DOM 元素将从 DOM 树中删除。DOM 节点的移除是通过`Node.removeChild()`函数完成的。`RemoveChild`从其被调用的父节点中移除指定的子节点。它接受一个参数 child，并返回被移除的子 DOM 节点。child 参数必须是父节点的子节点列表中的一个子节点。如果子元素不是父节点的子节点，将抛出异常。

下面的片段展示了`removeChild`功能的示例：

```php
<div id="div1">
  <div id="div2"></div>
</div>
<script>
  const div1 = document.getElementById( 'div1' );
  const div2 = document.getElementById( 'div2' );
  div1.removeChild( div2 );
</script>
```

###### 片段 3.15：从 DOM 中删除节点

在前面的示例中，我们创建了一个 div，`div1`，带有一个嵌套的子 div，`div2`。在脚本中，我们通过元素 id 获取了两个 div，然后从`div1`的子节点列表中移除了`div2`。

现在我们可以向 DOM 添加和删除节点，修改已经存在的节点将非常有用。节点可以通过以下方式进行更新：

+   替换节点

+   更改内部 HTML

+   更改属性

+   更改类

+   更改样式

### 更新 DOM 中的节点

修改 DOM 节点的第一种方法是完全用新的 DOM 节点替换它。DOM 节点可以使用`Node.replaceChild()`函数替换任何一个子节点。`ReplaceChild`替换父节点的一个子节点，并用一个新指定的节点调用它。它接受两个参数，`newChild`和`oldChild`，并返回被替换的节点（`oldChild`）。`oldChild`参数是将被替换的父节点子节点列表中的节点，`newChild`参数是将替换`oldChild`的节点。

下面的片段展示了这个示例：

```php
<div id="div1">
  <div id="div2"></div>
</div>
<div id="div3"></div>
<script>
  const div1 = document.getElementById( 'div1' );
  const div2 = document.getElementById( 'div2' );
  const div3 = document.getElementById( 'div3' );
  div1.replaceChild( div3, div2 );
</script>
```

###### 片段 3.16：替换 DOM 中的节点

在前面的示例中，我们创建了两个 div，`div1`和`div2`。`Div1`创建了一个嵌套的子 div，`div2`。在脚本中，我们通过元素 id 获取每个 div。然后我们用`div3`替换了`div1`的子元素`div2`。

操作 DOM 节点的第二种方法是通过更改节点的内部 HTML。节点的`innerHTML`属性可用于获取或设置元素中包含的 HTML 或 XML 标记。该属性可用于更改元素子元素中的当前 HTML 代码。它可以用于更新或覆盖 DOM 树中元素下方的任何内容。要将 HTML 插入节点，将`innerHTML`参数设置为包含要添加的 HTML 元素的字符串。传递到参数中的字符串将被解析为 HTML，并创建新的 DOM 节点；然后将它们作为子节点添加到引用该属性的父节点中。下面的片段展示了`innerHTML`属性的示例：

```php
<div id="div1"></div>
<script>
  const div1 = document.getElementById( 'div1' );
  div1.innerHTML = '<p>Paragraph1</p><p>Paragraph2</p>';
</script>
```

###### 片段 3.17：替换节点的 innerHTML

#### 注意

设置`innerHTML`的值会完全覆盖旧值。DOM 节点将被移除，并用从 HTML 字符串解析出的新节点替换。

出于安全原因，`innerHTML`不会解析和执行 HTML 字符串中`<script>`标签内包含的脚本。然而，还有其他方法可以通过`innerHTML`属性执行 JavaScript。你永远不应该使用`innerHTML`来追加你无法控制的字符串数据。

操作元素节点的第三种方法是通过更改节点的属性。元素节点属性可以通过三个函数进行交互：`Element.getAttribute()`、`Element.setAttribute()`和`Element.removeAttribute()`。这三个函数都必须在元素节点上调用。

#### 注意

应用于元素节点的一些属性具有特殊的含义。在添加或移除属性时要小心。HTML 属性列表如下所示：[`developer.mozilla.org/en-US/docs/Web/HTML/Attributes`](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes)。

`getAttribute`函数接受一个参数，即属性的名称，并返回指定属性的值。如果属性不存在，函数将返回 null 或空字符串("")。现代 DOM 规范规定该函数应返回 null 的值，大多数浏览器遵循这一规范，但一些浏览器仍遵循旧的 DOM3 规范，该规范规定正确的返回值应为空字符串。重要的是要处理这两种情况。

`setAttribute`函数用于设置或更新指定属性的值。它接受两个参数，**name**和**value**，并不返回任何值。`name`参数是要设置的属性的名称。`value`参数是要设置的属性的字符串值。如果传入的值不是字符串，它将在设置之前转换为字符串。由于值被转换为字符串，将属性设置为对象或 null 将不会得到预期的值。属性将被设置为传入值的字符串化版本。

`removeAttribute`函数从节点中移除指定的属性。它接受一个参数`attrName`，并不返回任何值。`attrName`参数是要移除的属性的名称。您可以使用`removeAttribute`来代替尝试使用`setAttribute`将属性的值设置为 null。下面的片段中展示了`getAttribute`、`setAttribute`和`removeAttribute`的示例：

```php
<div id="div1"></div>
<script>
  const div1 = document.getElementById( 'div1' );
  div1.setAttribute( 'testName', 'testValue' );
  div1.getAttribute( 'testName' );
  div1.removeAttribute( 'testName' );
</script>
```

###### 片段 3.18：获取、设置和移除属性

在前面的例子中，我们创建了一个名为`div1`的 div。然后我们通过其 id 获取该 div，添加`testName`属性，并将其值设置为`testValue`。然后我们获取`testName`的值并将其移除。

操作节点的第四种方式是通过更改其类信息。元素类信息用于关联类似的 HTML 元素以进行样式和分组。可以通过两种方式访问元素的类，即`className`属性或`classList`属性。`className`属性返回一个包含所有元素类信息的字符串。该属性可用于获取或设置类值。`classList`属性返回一个实时的`DOMTokenList`对象。这个对象只是当前类信息的实时列表，具有特殊的方法用于获取和更新类信息。

### 更新 DOM 中的节点

`classList`对象有六个辅助函数。它们在下表中详细说明：

![图 3.5：辅助函数](img/Figure_3.5.jpg)

###### 图 3.5：辅助函数

以下是这些辅助函数在下面的片段中使用：

```php
<div id="div1" class="testClass"></div>
<script>
  const classes = document.getElementById( 'div1' ).classList;
  classes.add( 'class1', 'class2' ); // adds class1 and class2
  classes.remove( 'testClass' ); // removes testClass
  classes.item( 1 ); // gets class at index 1: class2
  classes.toggle( 'class2' ); // removes class2 because it exists
  classes.contains( 'class2' ); // checks for class2: false
  classes.replace( 'class1', 'class0' ) // replaces class1 with class3  
</script>
```

###### 片段 3.19：使用 classList 对象

我们通常修改节点的第五种和最后一种方式是通过样式对象。样式对象反映了节点的 CSS 样式，每个元素节点都有一个样式对象。样式对象可以通过`Element.style`获得。样式对象包含了可以分配给对象的每个 CSS 样式的属性。这个对象是只读的，所以不应该直接通过覆盖样式对象来设置元素样式。相反，我们应该改变样式对象的各个属性：

```php
<div id="div1" style="color:blue">Hello World!</div>
<script>
  const style = document.getElementById( 'div1' ).style;
  style[ 'color' ]; // Returns blue
  style[ 'background-color' ] = 'red'; // Sets background-color to red
</script>
```

###### 片段 3.20：使用 classList 对象

#### 注意

可以在[`www.w3schools.com/jsref/dom_obj_style.asp`](https://www.w3schools.com/jsref/dom_obj_style.asp)上找到所有可用样式属性的列表。

DOM 操作是网页的最重要部分之一。DOM 可以通过查找、添加、删除和更新树中的节点来进行操作。我们可以通过唯一的 id、类或 CSS 查询选择器等多种方式找到 DOM 节点。一旦找到了 DOM 节点，我们可以通过移动到该元素的子节点、同级节点或父节点来遍历 DOM 树。要向 DOM 树中添加新元素，我们必须首先创建一个新的元素节点，然后将该元素附加到 DOM 中的某个位置。要删除一个元素，我们只需获取要删除的元素的节点，然后调用节点的删除函数。要更新一个节点，我们可以更改其属性、属性或直接替换节点。DOM 操作允许我们构建动态网页，这一点非常重要。

**结论**

从 HTML 代码构建的 Web 文档由文档对象模型（DOM）表示。DOM 是从节点构建的类似树的结构。每个节点对应 HTML 源代码中的一个元素。作为程序员，我们可以与 DOM 交互，动态更新网页。我们可以通过查找、创建、删除和更新元素节点与 DOM 进行交互。结合所有这些概念，我们可以创建可以根据用户交互更新视图的动态网页。几乎每个网站都可以看到这种功能，包括亚马逊、Facebook 和 Google。

### 练习 21：DOM 操作

您的团队正在构建一个电子邮件网站。该网站需要从 JSON 文件中加载用户的电子邮件数据，并动态填充加载的电子邮件数据的表格。电子邮件在示例代码文件中提供。电子邮件表应显示**发件人**、**收件人**和**主题**字段，并为每封电子邮件创建一行。使用电子邮件对象通过本章学习的 DOM 操作来构建 DOM 中的电子邮件表。

使用 DOM 操作技术构建 JavaScript 的电子邮件列表，执行以下步骤：

1.  打开名为**exercise**的文件，路径为**/exercises/exercise21/exercise.html**。

1.  在文件底部的`script`标签中，编写 JavaScript 代码（在*Code*下，本练习的末尾）。

1.  创建一个新的表元素(`<table>`)并将其保存到一个名为`table`的变量中。

1.  使用大括号(`{}`)创建一个新的作用域块。

创建一个数组来保存表头类型**To**、**From**和**Subject**。将数组保存到变量`headers`中。

创建一个表行元素(`<tr>`)并将其保存在变量`row`中。使用`forEach`函数循环遍历`headers`数组。

1.  在`forEach`的回调函数中，执行以下操作：

创建表头元素(`<th>`)并将其保存到`header`变量中。使用`appendChild()`，将一个新的文本节点附加到`header`。文本节点应包含`header`名称。

将存储在`header`中的表头元素作为子元素附加到存储在`row`中的表行中。

1.  将存储在`row`中的表行作为子元素附加到存储在`table`中的表中。输出如下图所示：![图 3.6：步骤 4 输出](img/Figure_3.6.jpg)

###### 图 3.6：步骤 4 输出

1.  使用大括号(`{}`)创建一个新的作用域块。

1.  使用`forEach`循环遍历数据数组`data`，并执行以下操作：

创建一个新的表行元素(`<tr>`)并将其保存在`row`变量中。创建另一个新的表数据元素(`<td>`)并将其保存在`to`变量中。

接下来，创建另外两个表数据元素(`<td>`和`<td>`)，并将它们保存为变量（`subject`和`from）。

将一个文本节点附加到存储在`to`中的表数据元素，该文本节点包含`forEach`循环的数据对象的`to`值。将另一个文本节点附加到存储在`from`中的表数据元素，该文本节点包含`forEach`循环的数据对象的`from`值。

将一个文本节点附加到存储在`subject`中的表数据元素，该文本节点包含`forEach`循环的数据对象的`subject`值。

将存储在`to`中的元素附加到存储在`row`中的表行。将存储在`from`中的元素附加到存储在`row`中的表行。

将存储在`subject`中的元素附加到存储在`row`中的表行。将存储在`row`中的行附加到存储在`table`中的表。

1.  获取`emailTableHolder` DOM 节点，并将存储在`table`变量中的表作为子节点附加。

1.  在 Web 浏览器中加载 HTML 文件以查看结果。

![图 3.7：最终输出](img/Figure_3.7.jpg)

###### 图 3.7：最终输出

**代码**

##### solution.html

```php
const table = document.createElement( 'table' );
const row = document.createElement( 'tr' );
[ 'to', 'from', 'subject' ].forEach( h => { 
  const header = document.createElement( 'th' );
  header.appendChild( document.createTextNode( h ) );
  row.appendChild( header );
} );
table.appendChild( row );
data.forEach( email => {
  const row = document.createElement( 'tr' );
  /* code omitted for brevity */
  table.appendChild( row );
} );
document.getElementById( 'emailTableHolder' ).appendChild( table );
```

###### Snippet 3.21: 使用 DOM 操作创建电子邮件列表

https://bit.ly/2FmvdK1

**结果**

您已成功分析了 DOM 操作技术。

## DOM 事件和事件对象

DOM 事件是功能性和响应式 Web 应用程序的基础。事件在任何具有任何形式用户交互的网站中使用。Facebook、Google 和 Skype 等网站都大量使用事件。事件是告诉程序员有关 DOM 节点发生了某事的信号。几乎可以出于任何原因触发事件。我们可以使用 JavaScript 来监听事件，并在事件发生时运行函数。

### DOM 事件

**DOM 事件**是由 DOM 节点发送的通知，以通知程序员 DOM 节点发生了某事。这可以是用户单击元素、在键盘上按键或视频播放结束等任何事情。可以触发许多事件。可以为触发的每个事件附加事件侦听器。事件侦听器是等待事件触发然后调用事件处理程序的接口。事件处理程序是响应事件运行的代码。事件处理程序是我们作为程序员分配给事件的 JavaScript 函数。这称为注册事件处理程序。

#### 注意

可以在此处找到完整的事件列表：[`developer.mozilla.org/en-US/docs/Web/Events`](https://developer.mozilla.org/en-US/docs/Web/Events)。

添加事件处理程序的最佳方法是使用`addEventListener`函数。`addEventListener`函数设置指定的事件处理程序在指定类型的事件触发时被调用。该函数接受三个参数，**type**，**listener**，以及**options**或**useCapture**。第一个参数 type 是要监听的区分大小写的事件类型。第二个参数 listener 是可以接收通知的对象，通常是 JavaScript 函数。选项和**useCapture**参数是可选的，您只能提供其中之一。选项参数指定具有**capture**、**once**和**passive**属性的选项对象。在选项参数中，名为'capture'的属性是一个布尔值，指示事件将在推送到 DOM 树之前分派给事件处理程序。名为'once'的属性是一个布尔值，指示事件处理程序在调用一次后是否应该被移除。名为'passive'的属性是一个布尔值，指示事件处理程序永远不会调用`preventDefault`函数（在处理事件子主题中讨论）。useCapture 参数的功能与`options.capture`属性相同。

### 事件侦听器

**事件侦听器**可以附加到任何 DOM 节点。要附加事件侦听器，我们必须选择需要监听事件的节点，然后我们可以在该节点上调用`addEventListener`函数。如下面的代码片段所示：

```php
<button id="button1">Click me!</button>
<script>
  const button1 = document.getElementById( 'button1' );
  button1.addEventListener( 'click', () => {
    console.log( 'Clicked' );
  }, false );
</script>
```

###### Snippet 3.22: 获取、设置和移除属性

在前面的示例中，我们创建了一个 ID 为`button1`的按钮。在脚本中，我们选择了该按钮并添加了一个事件侦听器。事件侦听器监听点击事件。当发生点击事件时，它调用处理程序函数，该函数记录到控制台。

#### 注意

您可能会在 HTML 代码中看到内联事件处理程序，例如，`<button onclick="alert('Hello!')">Press me</button>`。您不应该这样做。最佳做法是保持 JavaScript 和 HTML 分开。当您混合 HTML 和 JavaScript 时，代码很快就会变得难以管理、低效，并且更难解析和解释。

在以后的时间，如果我们决定不再需要事件监听器，我们可以使用`removeEventListener`函数将其移除。`removeEventListener`函数从指定的事件类型中移除指定的处理程序函数。它接受与`addEventListener`相同的参数。要正确地移除事件监听器，`removeEventListener`必须与添加的监听器匹配。`removeEventListener`会查找具有相同类型、监听器函数和捕获选项的监听器。如果找到匹配项，则移除事件监听器。以下是`removeEventListener`的示例：

```php
<button id="button1">Click me!</button>
<script>
  const button1 = document.getElementById( 'button1' );
  function eventHandler() { console.log( 'clicked!' }
  button1.addEventListener( 'click', eventHandler, true );
  button1.removeEventListener( 'click', eventHandler, true );
</script>
```

###### 片段 3.23：获取、设置和删除属性

在前面的示例中，我们创建了一个带有 id `button1`的按钮。在脚本中，我们通过添加单击事件的事件监听器来获取该按钮。然后，我们移除相同的监听器，提供与`addEventListener`函数提供的完全相同的参数，以便`removeEventListener`可以正确匹配我们要移除的监听器。

### 事件对象和处理事件

每个事件处理程序函数都接受一个参数。这是事件对象。您经常会看到此参数被定义为`event`、`evt`或简单地`e`。它会自动传递给事件处理程序，以提供有关事件的信息。事件处理程序可以利用事件对象中的信息来操作 DOM，并允许用户与页面交互：

```php
<div id="div1">Click me!</div>
<script>
  const div1 = document.getElementById( 'div1' );
  button1.addEventListener( 'click', ( e ) => {
    e.target.style.backgroundColor = 'red';
  }, false);
</script>
```

###### 片段 3.24：使用事件处理程序操作 DOM

可以通过调用事件类的新实例（`new Event()`）来创建事件对象的新实例。构造函数接受两个参数：**type**和**options**。类型是事件的类型，选项是一个可选对象，包含以下字段：**bubbles**、**cancelable**和**composed**。这三个字段也都是可选的。bubbles 属性指示事件是否应该冒泡。`cancelable`属性指示事件是否可以被取消。composed 属性指示事件是否应该触发阴影根之外的监听器。这三个默认值都为 false。

事件对象具有许多有用的属性和函数。这些属性可以被利用来获取有关事件的附加信息。例如，我们可以使用`Event.target`属性来获取最初触发事件的 DOM 节点，或者我们可以使用`Event.type`来查看事件的名称。当您希望为多个元素使用相同的处理程序时，`Event.target`非常有用。我们可以重用处理程序，只需使用`Event.target`来检查哪个元素触发了事件，而不是为每个事件创建一个新的处理程序函数。

当从 DOM 元素触发事件时，它会通知附加到 DOM 节点的事件监听器。然后，事件会传播或冒泡，直到达到树的顶部为止。这种效果称为事件传播或事件冒泡。它允许我们通过减少页面中所需的事件监听器数量来使我们的代码更加高效。如果我们有一个具有许多子元素的元素，它们都需要相同的用户交互，我们可以将单个事件监听器添加到父元素，并捕获从子节点冒泡上来的任何事件。这称为事件委托。我们委托事件处理给父节点，而不是将监听器附加到每个子节点。

### 事件传播

事件传播可以通过`stopPropagation`函数进行控制。这个函数是事件对象中的许多函数之一。`StopPropagation`不带任何参数。当调用它时，它会阻止当前事件的进一步传播。这意味着它完全捕获了事件，并阻止它向上冒泡到任何其他父节点。停止事件传播在使用委派时或者在子节点和父节点上有监听同一事件但执行不同任务时非常有用。

### 触发事件

标准 DOM 事件由浏览器自动触发。JavaScript 为我们提供了两个非常强大的工具，允许我们更多地控制页面中事件的触发。第一个工具是通过 JavaScript 触发事件。第二个是自定义事件。

在本章的前面部分，我们学到可以创建事件对象的新实例。如果我们不能触发事件并使 DOM 树知道发生了什么，单独的事件就不是很有用。DOM 节点有一个成员函数`dispatchEvent()`，允许我们触发或分发事件对象的实例。`DispatchEvent()`应该在您希望从事件节点触发的 DOM 节点上调用。它接受一个参数并返回一个布尔值。这个参数是将在目标 DOM 节点上触发的事件对象。如果事件是可取消的并且处理事件的一个事件处理程序被调用`Event.preventDefault()`，`DispatchEvent()`的布尔返回值将为 false。否则，`dispatchEvent()`将返回 true。以下是`dispatchEvent()`的示例：

```php
const event = new MouseEvent( 'click' , { 
  bubbles:true,
  cancelable: true
} );
const element = document.getElementById( 'button' );
const canceled = element.dispatchEvent(event);
```

###### 片段 3.26：触发事件

如果事件的类型没有正确指定，`dispatchEvent`方法将抛出`UNSPECIFIED_EVENT_TYPE_ERR`错误。这意味着如果事件的类型为 null 或空字符串，或者在调用`dispatchEvent()`之前未初始化事件，则会抛出运行时错误。

重要的是要注意，使用`dispatchEvent()`触发的事件不会通过事件循环异步调用。由 DOM 节点触发的正常事件会通过事件循环异步调用事件处理程序。当使用`dispatchEvent()`时，事件处理程序会同步调用。所有适用的事件处理程序都会在代码继续执行`dispatchEvent`调用后执行并返回。如果有许多事件处理程序或者其中一个事件处理程序做了大量同步工作，其他事件可能会被阻塞。

#### 注意

一些浏览器实现了`fireEvent()`函数，用于在 DOM 节点上触发事件。这个函数是一个非标准函数，在大多数浏览器上不起作用。不要在生产代码中使用这种方法。

### 练习 22：处理您的第一个事件

要设置事件侦听器并捕获触发的事件，请执行以下步骤：

1.  创建一个带有`body`标签的 HTML 文件。

1.  在`body`标签内，创建一个文本为“点击我！”的按钮，并将其 id 设置为`button1`。

1.  在按钮后添加一个`script`标签。

1.  在`script`标签中，通过 id 选择按钮并将其保存到`button1`变量中。

1.  为存储在`button1`中的元素添加`click`事件的事件侦听器。

#### 注意

回调应调用警报函数并用“点击！”字符串警报浏览器。

**代码**

##### index.html

```php
<html>
<body>
 <button id="button1">Click me!</button>
 <script>
   const button1 = document.getElementById( 'button1' );
   button1.addEventListener( 'click', ( e ) => {
     alert('clicked!');
   }, false );
 </script>
</body>
</html>
```

###### 片段 3.25：DOM 事件处理

https://bit.ly/2M0Bcp5

**结果**

###### ![图 3.8：步骤 2 点击我！按钮](img/Figure_3.8.png)

###### 图 3.8：步骤 2 点击我！按钮

![图 3.9：输出视图](img/Figure_3.9.jpg)

###### 图 3.9：输出视图

您已成功设置了事件侦听器并捕获了触发的事件。

### 自定义事件

JavaScript 还允许创建自定义事件。自定义事件是一种触发事件和监听具有自定义类型的事件的方式。事件的类型可以是任何非空字符串。创建自定义事件的最基本方式是使用事件类型作为自定义事件名称初始化事件对象的新实例。这是通过以下语法完成的：`const event = new Event( 'myCustomEvent' )`。像这样创建事件不允许向事件添加任何自定义信息或属性。要创建带有附加信息的自定义事件，我们可以使用`CustomEvent`类。`CustomEvent`类构造函数接受两个参数。第一个参数是表示我们要创建的自定义事件的类型名称的字符串。第二个参数是表示自定义事件初始化选项的对象。它接受与传递给事件类初始化器的选项相同的字段，另外还有一个名为`detail`的字段。详细字段默认为 null，是与事件相关的与事件关联的值。我们想要传递给自定义事件的任何信息都可以通过详细参数传递。此参数中的数据也传递给所有监听自定义事件的处理程序。

#### 注意

事件构造函数适用于所有现代浏览器，除了 Internet Explorer。为了与 IE 完全兼容，必须使用稍后讨论的`createEvent()`和`initEvent()`方法，或者使用`polyfill`来模拟`CustomEvent`类。

为了最大限度地提高代码浏览器兼容性，我们还必须讨论用于创建自定义事件的`initEvent()`和`createEvent()`方法。这些方法已被弃用并从 Web 标准中删除。但是，一些浏览器仍然支持这些功能。要在旧版浏览器中创建自定义事件，必须首先使用`var event = document.createEvent( 'Event' )`创建事件（在旧版浏览器中必须使用`var`而不是`const`），然后使用`event.initEvent()`初始化新事件。`CreateEvent()`接受一个参数，类型。这是将要创建的事件对象的类型。此类型必须是标准 JavaScript 事件类型之一，例如`Event`、`MouseEvent`等。`InitEvent()`接受三个参数。第一个参数是表示事件类型名称的字符串。例如，点击事件的类型是**click**。第二个参数是表示事件冒泡行为的布尔值。第三个参数是表示事件可取消行为的布尔值。这两种行为在本主题的*事件对象和处理事件*部分中进行了讨论。

为了捕获和处理自定义事件，我们可以使用标准的事件监听器行为。我们所需要做的就是使用`addEventListener()`附加一个监听自定义事件类型的事件监听器。例如，如果我们创建了一个事件类型为`myEvent`的`CustomEvent`，我们只需要添加一个事件监听器来监听这个类型，使用`addEventListener( 'myEvent', e => {} )`。每当类型为`myEvent`的事件被触发时，添加的事件监听器回调函数将被调用。

当调用事件监听器回调时，回调中的事件参数将具有一个额外的字段`detail`。此字段将包含通过自定义事件选项对象的`detail`字段传递给自定义事件的信息。与自定义事件相关的任何信息都应通过`detail`对象传递。详细对象的示例如下所示：

```php
const element = document.getElementById( 'button' );
element.addEventListener( 'myClick', e => {
  console.log( e.detail );
} );
const event = new CustomEvent( 'myClick' , { detail: 'Hello!' } );
const canceled = element.dispatchEvent( event );
```

###### 片段 3.27：在详细信息中触发自定义事件

### 练习 23：处理和委托事件

您正在构建一个购物清单页面，以帮助忙碌的购物者管理购物清单，而无需纸和笔。我们的购物清单应用程序将是一个带有表、文本输入和添加行按钮的页面。添加行按钮将在购物清单表中添加新行。添加的行包含购物清单项目（来自文本输入的文本）和一个删除按钮。删除按钮将从购物清单表中删除该行。

以下步骤将构建应用程序：

1.  在**exercises/exercise23/exercise.html**中打开起始文件。

1.  在 HTML `body`中的`userInteractionHolder` `div`中，添加一个文本输入和一个按钮。

给按钮添加 id`addButton`。

向具有 id`shoppingList`的`div`添加一个表元素。

向在上一步创建的表（`id="shoppingList"`）中添加一行。

向表添加两个标题项，一个带有文本`Item`，另一个带有文本`Remove`。

![图 3.10：第 2 步后的输出](img/Figure_3.10.jpg)

###### 图 3.10：第 2 步后的输出

1.  在`script`标签中，通过其 id 选择按钮，并添加一个点击监听器，调用`_addRow`函数。创建`_addRow`函数，具有以下功能：

接受一个参数`e`，即事件对象。使用 DOM 遍历，使用事件目标上的`previousSibling`属性获取文本输入。将文本输入元素节点保存到变量`inputBox`中。

将文本框中的值保存到`value`变量中。

通过将其设置为空字符串（""）来清除文本区域的值。

创建一个表行元素，并将其保存在`row`变量中。

使用 DOM 操作和链接将表数据元素附加到表行。将文本节点附加到表数据元素。

#### 注意

文本节点应包含存储在`value`中的值。

1.  使用 DOM 操作和链接将表数据附加到表行。

1.  将按钮附加到表数据元素。

1.  将文本`remove`附加到按钮上。

1.  返回到按钮元素。

1.  为按钮添加监听器，并让它调用`_removeRow`函数。

1.  选择`shopingList`表，并将行附加到其中。

1.  创建`_removeRow`函数，具有以下功能：

接受一个参数`e`，其中将包含事件对象。

使用 DOM 遍历，获取发生按钮点击的行元素，并使用`parentNode`属性。记录行元素。

使用 DOM 遍历和链接，获取包含行的表，然后从表中删除行：

![图 3.11：最终输出](img/Figure_3.11.jpg)

###### 图 3.11：最终输出

**代码**

##### solution.html

```php
<body>
<h1>Shopping List</h1>
<div id="userInteractionHolder">
  <!-- add a text input and an add button -->
</div>
 <div id="shoppingListHolder">
  <!-- add a table with a row for column headers -->
</div>
<script>
  /* add event listener to add button */
  function _addRow( e ) { /* get input data and add row with it */ }
  function _removeRow( e ) { /* get row from event and it */ }
</script>
```

###### 片段 3.28：使用 DOM 操作和事件处理构建购物清单应用程序

https://bit.ly/2D1c3rC

**结果**

您已成功应用事件处理概念来构建一个有用的 Web 应用程序。

## jQuery

jQuery 是一个轻量级的 JavaScript 库，旨在简化 DOM 交互。它是在 Web 开发中使用最广泛的库之一。jQuery 旨在简化对 DOM 的调用，并使代码更加流畅。在这个主题中，我们将概述 jQuery 是什么，如何在项目中安装 jQuery，jQuery 基础知识，使用 jQuery 进行 DOM 操作以及使用 jQuery 处理事件。

jQuery 是一个旨在使 DOM 遍历、操作、事件处理、动画和 AJAX 请求更简单使用，并使使用这些元素的代码更加流畅的库。jQuery 是一个广泛的 JavaScript 库。对 JavaScript 的深入理解对于发挥 jQuery 的所有功能至关重要。

jQuery 提供了一个易于使用的 API，具有广泛的跨浏览器兼容性。jQuery 实现了他们所谓的“当前”浏览器支持。这只是意味着 JQuery 将在浏览器的当前发布版本和上一个发布版本（v23.x 和 22.x，但不是 v21.x）上运行并得到支持。代码可能会在旧的浏览器版本上成功运行，但对于出现在旧的浏览器版本中的任何错误，JQuery 的错误修复将不会被推送。jQuery 浏览器兼容性还延伸到 Android 和 IOS 设备上的原生移动浏览器。

#### 注意

完整的文档可以在官方 JQuery 网页上找到：[` jquery.com/`](https://jquery.com/)。

安装 jQuery 的第一种方法是直接下载源 JavaScript 文件。这些文件可以在[`code.jquery.com`](http://code.jquery.com)找到。JavaScript 文件可以直接添加到项目的文件结构中。由于文件大小较小，应在生产代码中使用缩小版本。

#### 注意

**代码缩小**是从源代码中删除不必要字符而不改变其功能的过程。缩小是为了减小代码文件的大小。这对 JavaScript、HTML 和 CSS 文件很重要，因为它减少了发送和加载网页所需的资源。

安装 JQuery 的第二种方法是使用包管理器。用于此目的的最流行的命令行包管理器是 NPM、Yarn 和 Bower。本书的最后一章将更详细地讨论 NPM。要使用这些 CLI（命令行界面）包管理器之一安装 jQuery，首先安装和配置相关的包管理器。要使用 NPM 安装，运行以下命令：`npm install jquery`。这将把 jQuery 文件放在`node_modules/jquery/dist/`文件夹下的`node_modules`文件夹中。要使用 Yarn 安装，使用以下命令：`yarn add jquery`。要使用 Bower 安装，使用以下命令：`bower install jquery`。使用 Bower 安装将把文件放在`bower_components/jquery/dist/`文件夹下的`bower_components`文件夹中。

一旦安装了 JQuery，我们就可以开始将 jQuery 加载到我们的项目中。这只需要在 HTML 文件中添加一个脚本标签即可。在主 HTML 文件中，只需添加一个带有 jQuery 库文件路径的脚本标签（<script src="path/to/jquey.js"></script>）。JQuery 现在已安装并准备好在项目中使用！

### jQuery 基础

JQuery 是围绕选择和处理 DOM 节点构建的库。默认情况下，所有 JQuery 操作都可以在库名称`jQuery`和快捷变量`$`下使用。我们将通过引用快捷变量来调用所有 JQuery 函数。

在创建或选择 DOM 节点时，jQuery 始终返回一个 JQuery 对象的实例。JQuery 对象是一个类似数组的集合，其中包含零索引序列的 DOM 元素、一些熟悉的数组函数和属性，以及所有内置的 JQuery 方法。关于 JQuery 对象有两点很重要。首先，JQuery 对象不是活动对象。JQuery 对象的内容不会随着 DOM 树的更改而更新。如果 DOM 已更改，可以通过重新运行相同的 JQuery 选择器来更新 JQuery 对象。其次，JQuery 对象也不相等。使用相同查询构建的两个 JQuery 对象之间的相等比较将不会是真值。要比较 JQuery 对象，必须检查集合中包含的元素。

#### 注意

零索引意味着对象具有可以用来引用项目序列中的项目的数字属性（0、1、2、…、n）。

JQuery 对象不是数组。JQuery 对象上可能不存在内置数组属性和函数。

### jQuery 选择器

JQuery 的核心功能围绕选择和操作 DOM 元素展开。这是通过 jQuery 核心选择器来实现的。要选择 DOM 元素，我们调用 jQuery 选择器函数`jQuery( selector )`，或者简写为`$( selector )`。传递给 jQuery 函数的选择器几乎可以是任何有效的 CSS 选择器、回调函数或 HTML 字符串。如果传递给 JQuery 选择器的是 CSS 选择器，将返回一个匹配元素的集合，这些元素将在一个 JQuery 对象中返回。如果传递给选择器的是 HTML 字符串，将从提供的 HTML 字符串创建一个节点集合。如果传递给选择器函数的是回调函数，当 DOM 加载完成时将运行回调。jQuery 还可以接受一个 DOM 节点，并从中创建一个 JQuery 对象。如果将 DOM 节点传递给 jQuery 选择器函数，该节点将自动被选择并返回到一个 JQuery 集合中。下面的片段展示了 JQuery 选择器函数的一个示例：

```php
const divs = $( "div" ); // JQuery select all divs
const div1 = document.getElementById( 'div1' ); // DOM select a div
const jqueryDiv1 = $( div1 ); // Create a JQuery object from div
```

###### 片段 3.29：选择 DOM 节点

大多数 jQuery 函数都是在一组 DOM 节点（`$()`）上操作的；然而，jQuery 也提供了一组不是这样的函数。这些函数直接通过$变量引用。这两者之间的区别对于新的 jQuery 用户来说可能会令人困惑。记住这个区别最简单的方法是注意到`$`命名空间中的函数通常是实用方法，不适用于选择。有些情况下，选择器方法和核心实用方法具有相同的名称，例如`$.each()`和`$().each()`。在阅读 jQuery 文档和学习新函数时，一定要确保你正在探索正确的函数。

在创建基本 DOM 结构之前，HTML 页面的 DOM 不能安全地进行操作。JQuery 提供了一种安全等待 DOM 准备就绪的方法。这是通过`ready()` JQuery 对象函数来实现的。这个函数应该在包含 HTML 文档的 jQuery 对象上调用（`$( document ).ready()`）。`ready()`函数接受一个参数，一个回调函数。一旦 DOM 准备就绪，这个函数就会运行。操作 DOM 的代码应该放在这个回调函数中。

在 JavaScript 中使用多个库时，命名空间冲突总是一个问题。jQuery 及其所有插件和功能都包含在`jQuery`命名空间中。因此，jQuery 和任何其他库之间不应该有冲突。然而，有一个例外，jQuery 默认使用`$`作为 jQuery 命名空间的快捷方式。如果你使用另一个使用`$`变量的库，可能会与 jQuery 发生冲突。为了避免这种情况，你可以将 jQuery 置于无冲突模式。要做到这一点，调用 jQuery 命名空间上的`noConflict()`函数（`jQuery.noConflict()`）。这将打开无冲突模式，并允许你为 jQuery 库分配一个新的快捷变量名。变量名可以是任何你喜欢的，从`$`到`mySuperAwesomeJQuery`。启用无冲突模式并更改 jQuery 快捷变量名的完整示例如下片段所示：

```php
<script src="jquery.js"></script>
<script>
  // Set the jQuery alias to $j instead of $
  const $j = jQuery.noConflict();
</script>
```

###### 片段 3.30：启用无冲突模式

### jQuery DOM 操作

JQuery 是围绕 DOM 操作构建的。在这里，我们将介绍 JQuery DOM 操作的基础知识。我们将从选择元素开始，然后转向遍历和操作 DOM，最后以链式操作结束。

### 选择元素

DOM 操作的第一步始终是选择要处理的 DOM 节点。JQuery 最基本的概念是“选择一些元素并对其进行操作”。jQuery 通过选择器函数`$()`非常容易地选择元素。jQuery 支持大多数 CSS3 选择器来选择节点。选择元素的最简单方法是通过 id、类名、属性和 CSS 来选择元素。

通过将 CSS 元素 id 选择器传递给 jQuery 选择器函数来选择元素：`$( '#elementId' )`。这将返回一个包含匹配该 id 的元素的 JQuery 对象。通过类名选择与通过 id 选择的方式相同。将 CSS 类名选择器传递给 jQuery 选择器函数：`$( '.className' )`。这将返回一个包含所有匹配该类名的元素的 jQuery 对象。通过属性选择元素是通过将属性 CSS 选择器传递给 jQuery 选择器函数来完成的：`$( "div[attribute-name='example']" )`。这将返回一个包含所有匹配指定元素类型和属性名称/值的元素的 JQuery 对象。jQuery 还支持更复杂的选择器。您可以传递复合 CSS 选择器、逗号分隔的选择器列表和伪选择器，如`:visible`。这些选择器都返回包含匹配元素的 JQuery 对象。

#### 注意

如果 jQuery 选择器没有匹配任何节点，它仍然会返回一个 JQuery 对象。JQuery 对象的集合中将没有节点，并且对象的长度属性将等于零。如果要检查选择器是否找到节点，必须检查长度属性，而不是 JQuery 对象的真实性。

一旦您选择了一些节点，就可以使用 JQuery 对象函数来过滤和细化选择。一些非常有用的简单函数包括`has()`、`not()`、`filter()`、`first()`和`eq()`。所有这些函数都接受一个选择器，并返回一个带有过滤节点集的 JQuery 对象。`has()`函数将列表过滤为包含其后代与提供给`has()`的 CSS 选择器匹配的元素。`not()`函数将 JQuery 对象的节点过滤为仅包含不匹配提供的 CSS 选择器的节点。`filter()`函数将节点过滤为仅显示与提供的 CSS 选择器匹配的节点。`first()`返回 JQuery 对象内部节点列表中的第一个节点。`eq()`函数返回一个包含该索引处节点的 JQuery 对象。这些方法的完整深入文档以及其他过滤方法可以在 JQuery 网站上找到。

### 遍历 DOM

一旦使用 jQuery 选择器选择了节点，我们可以遍历 DOM 以查找更多元素。DOM 节点可以沿着三个方向遍历：到父节点、到子节点和到兄弟节点。

遍历父节点有很多种方式，但最简单的方式之一是在 JQuery 对象上调用四个函数中的一个。第一种遍历父节点的方式是调用`parent()`函数。这个函数简单地返回一个包含原始节点的父节点的 JQuery 对象。第二个函数是`parents()`函数。这个函数接受一个 CSS 选择器，并返回一个包含匹配节点的 JQuery 对象。`parents()`遍历 DOM 树，选择与提供的查询条件匹配的任何父节点，直到树的顶部。如果没有给出条件，它将选择所有父节点。第三个父遍历函数是`parentsUntil()`函数。它也接受一个 CSS 选择器，并返回一个 JQuery 对象。这个函数遍历父树，选择元素，直到它达到与提供的选择器匹配的元素。与提供的选择器匹配的节点不包括在新的 JQuery 对象中。最后一个方法是`closest()`方法。这个函数接受一个 CSS 选择器，并返回一个包含与提供的选择器匹配的第一个父节点的 JQuery 对象。

#### 注意

`closest()`总是从包含它被调用的 JQuery 对象中的节点开始搜索。如果传递给`closest()`的选择器与该节点匹配，它将始终返回自身。

遍历子节点可以通过两种简单的方式轻松完成：`children()`和`find()`。`children()`函数接受一个 CSS 选择器，并返回一个 JQuery 对象，该对象是调用它的节点的直接后代，并且匹配选择器。`find()`函数接受一个 CSS 选择器，并返回 DOM 树中任何匹配提供的 CSS 选择器的后代节点的 JQuery 对象，包括嵌套的子节点。

遍历兄弟节点可以通过`next()`、`prev()`和`siblings()`函数以最简单的方式完成。`next()`获取下一个兄弟节点，`prev()`获取上一个兄弟节点。这两个函数都返回 JQuery 对象中的新节点。`siblings()`接受一个 CSS 选择器，并选择匹配提供的选择器的元素的兄弟节点（前一个和后一个）。`prev()`和`next()`也有类似的函数：`prevAll()`、`prevUntil()`、`nextAll()`和`nextUntil()`。正如你所期望的那样，`All`函数选择所有之前或之后的节点。`Until`函数选择节点，直到但不包括与提供的 CSS 选择器匹配的节点。

### 修改 DOM

现在我们可以选择 DOM 节点了，我们需要学习如何修改和创建它们。要创建一个节点，我们可以简单地将 HTML 字符串传递给选择器函数。JQuery 将解析 HTML 字符串并创建字符串中的节点。这样做的方式是：`$('<div>')`。HTML 字符串将被解析为 div 元素，并返回一个包含该元素的 JQuery 对象。

要向 DOM 添加元素，我们可以使用`append()`、`before()`和`after()`函数。`append()`函数接受一个 JQuery 对象，并将其附加到调用`append()`函数的 JQuery 对象的子节点中。然后返回一个包含调用`append()`函数的节点的 JQuery 对象。`before()`和`after()`函数以类似的方式工作。它们都接受一个 JQuery 对象，并在调用它们的 JQuery 对象中的节点之前或之后插入它。

要删除 DOM 节点，我们可以使用`remove()`和`detach()`函数。`remove()`永久删除与函数传入的 CSS 选择器匹配的节点。`remove()`返回一个包含已删除节点的 JQuery 对象。所有事件监听器和相关数据都将从节点中删除。如果它们返回到 DOM 中，监听器和数据将需要重新设置。`detach()`删除节点但保留事件和数据。与`remove()`一样，它返回一个包含已分离节点的 JQuery 对象。如果打算最终将节点返回到页面上，则应使用`detach()`。

使用 JQuery 修改节点非常简单。一旦我们选择了节点，遍历了树，然后将选择过滤到单个节点，我们就可以调用 JQuery 对象函数来修改属性和 CSS 等内容。要修改属性，我们可以使用`attr()`函数。`attr()`接受两个值。第一个是要修改的属性的名称。第二个值设置属性等于什么。要修改元素的 CSS，我们可以使用`css()`函数。此函数接受两个参数。第一个参数是要修改的 CSS 属性。第二个参数值设置 CSS 属性等于什么。这两个函数也可以用作`get`函数。如果省略第二个值，`attr()`和`css()`函数将返回属性或 CSS 属性的值，而不是设置它。

### 链接

大多数 jQuery 对象函数返回 jQuery 对象。这使我们能够链接调用，并且不需要用分号和换行符分隔每个函数调用。在链接 jQuery 函数时，jQuery 会跟踪对选择器和 JQuery 对象中的节点的更改。我们可以使用`end()`函数将当前选择恢复到其原始选择。以下是一个示例：

```php
$( "#myList" )
  .find( ".boosted" ) // Finds descendents with the .boosted class
  .eq( 3 ) // Select the third index of the <li> filtered list
    .css( 'background-color', 'red' ) // Set css
    .end() // Restore selection to .boosted items in #myList
  .eq( 0 )
    .attr( 'age', 23 );
```

###### 片段 3.31：链接和.end()

### jQuery 事件

如前面在*DOM 事件*部分讨论的那样，任何响应灵敏和功能的网页都必须依赖事件。jQuery 还提供了一个简单的接口来添加事件处理程序和处理事件。

**注册处理程序**

使用 jQuery 注册事件非常简单。jQuery 对象提供了许多注册事件的方法。注册事件的最简单方法是使用`on()`函数。`On()`可以使用两组不同的数据进行调用。

设置事件监听器的第一种方法是使用`on()`调用四个参数：**events**，**selector**，**data**和**handler**。Events 是一个以空格分隔的事件类型和可选的命名空间字符串（`click hover scroll.myPlugin`）。将为提供的每个事件创建一个监听器。第二个参数是选择器。这是可选的。如果提供了 CSS 选择器字符串，则事件监听器也将添加到与选择器匹配的所选元素的所有后代元素。第三个参数是数据。这可以是任何内容，也是可选的。如果提供了数据，那么在触发事件时它将被传递到事件对象的数据字段中。最后一个参数是处理程序函数。这是在事件触发时将被调用的函数。

设置事件监听器的第二种方法是使用三个参数调用`on()`：**events**，**selector**和**data**。与第一种方法类似，事件指定将创建监听器的事件。但是，在这种情况下，事件是一个对象。键是事件名称，将为其设置监听器的值，而值是事件触发时将被调用的函数。与第一种方法一样，选择器和数据参数是可选的。它们的功能与第一种方法相同。

要删除事件监听器，我们可以使用`off()`方法。使用 off 删除事件监听器的最简单方法是提供要删除监听器的事件的名称。与`on()`一样，我们可以通过空格分隔的字符串或对象提供事件类型。

### 触发事件

jQuery 提供了一种从 JavaScript 中触发事件的简单方法：`trigger()`函数。`trigger()`函数应该用于触发事件，并接受事件类型和无限数量的额外参数。事件类型是将被触发的事件类型。额外的参数将传递给事件处理程序函数，并在事件对象之后作为参数传递。

### 自定义事件

jQuery 中的自定义事件非常简单。与 Vanilla JavaScript 中的自定义事件不同，在 jQuery 中，要为自定义事件设置事件处理程序，我们只需要使用`on()`函数创建一个监听器，事件类型为自定义事件。要触发事件，我们只需要使用`trigger()`来触发它，事件类型为自定义事件。

### 活动 3：实现 jQuery

您想制作一个控制家庭智能 LED 照明系统的网络应用程序。您有三个 LED，可以单独打开或关闭，或者全部一起切换。您必须构建一个简单的 HTML 和 jQuery 界面，显示灯的开启状态，并具有控制灯的按钮。

要使用 JQuery 构建一个功能应用程序，请执行以下步骤：

1.  使用命令行上的`npm run init`设置一个 Node.js 项目并安装 jQuery。

1.  创建一个加载 jQuery 脚本的 HTML 文件。

1.  在 HTML 文件中，添加三个起始为白色的 div。

1.  在 div 上方添加一个切换按钮，并在每个 div 后面添加一个按钮。

1.  为每个按钮设置点击事件的事件监听器。

1.  切换按钮应更改所有 div 的颜色。其他按钮应更改相关`div`的颜色。

在颜色变化时，将颜色在黑色和白色之间切换

**代码**

**结果**

![图 3.12：步骤 4 输出后](img/Figure_3.12.jpg)

###### 图 3.12：步骤 4 输出后

![图 3.13：步骤 6 输出后](img/Figure_3.13.jpg)

](image/Figure_3.13.jpg)

###### 图 3.13：步骤 6 输出后

您已成功利用 jQuery 构建了一个功能应用程序。

#### 注意

本活动的解决方案可以在第 285 页找到。

## 总结

网络开发围绕着文档对象模型和事件对象展开。JavaScript 被设计成能够快速高效地与 DOM 和 DOM 事件进行交互，为我们提供强大而丰富的互动网页。在本章的第一个主题中，我们讨论了 DOM 树，并讨论了导航和操作 DOM 的方法。在本章的第二个主题中，我们讨论了 JavaScript 事件对象，展示了如何与 DOM 事件交互，并演示了如何设置处理程序来捕获事件。在本章的最后一个主题中，我们讨论了 jQuery 模块。我们讨论了 jQuery 对象和 jQuery 选择器，并展示了如何使用 jQuery 进行 DOM 操作和事件处理。通过学习本章的内容，您应该已经准备好开始编写自己的强大而丰富的互动网页。

在下一章中，您将分析测试的好处，并建立代码测试环境。
