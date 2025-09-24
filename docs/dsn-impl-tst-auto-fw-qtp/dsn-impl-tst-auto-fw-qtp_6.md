# 第六章. Web 应用程序中的基于 DOM 和 XPath 的框架

对于 Web 应用程序自动化，QTP 允许我们使用**文档对象模型**（**DOM**）和执行 JavaScript。QTP 还使用**XPath**查找元素。在本章中，我们将学习关于 DOM、基本 JavaScript 和 XPath 术语。我们还将通过示例学习如何使用 XPath 查找元素并在创建脚本中使用它们。QTP 允许在脚本中执行 JavaScript 代码。JavaScript 可以使用 HTML DOM 查找元素、更改 HTML 内容、属性、样式以及删除 HTML 元素。

# 文档对象模型

文档对象模型（DOM）允许将 HTML 文档表示为树结构，同时也允许动态访问和更新 HTML 文档的内容、结构和样式。

## HTML DOM

HTML DOM 定义了 HTML 元素的对象和属性以及访问它们的方法。用简单的话说，HTML DOM 允许以标准方式添加、检索、更改或删除 HTML 元素。在 HTML DOM 中，一切都被视为节点：

+   整个文档是一个名为节点文档的节点

+   每个 HTML 元素都是一个元素节点

+   元素内的文本是一个文本节点

+   节点中的每个属性都是一个属性节点

+   注释是一个注释节点

### 节点关系 - 父节点、子节点和兄弟节点

HTML 是一种标记语言，它定义了标签；这些标签以以下方式相互关联：

+   每个文档有一个根元素，并且没有父元素。

+   一个节点可以有多个子节点，但只有一个父节点。

+   具有相同父节点的节点被称为兄弟节点。

看以下 HTML 代码片段以了解关系：

```js
<html>
  <head>
    <title> DOM Example </title>
  </head>
  <body>
    <h1>Chapter 1</h1>
    <p>QTP</p>
  </body>
</html>
```

前面脚本中标签之间的关系如下所示：

+   `<html>`标签是根节点，它没有父节点

+   `<html>`节点是`<head>`和`<body>`标签的父节点，换句话说，`<html>`节点有两个子节点：`<head>`和`<body>`

+   QTP 文本节点的父节点是`<h1>`节点

+   `<head>`节点有一个子节点：`<title>`节点

+   `DOM Example`是一个文本节点，其父节点是`<title>`节点

+   `<h1>`和`<p>`节点是`<body>`的子节点，并且彼此是兄弟节点

+   `<head>`元素是`<html>`元素的第一个子节点，而`<body>`元素是`<html>`元素的最后一个子节点

+   `<h1>`元素是`<body>`元素的第一个子节点，而`<p>`元素是`<body>`元素的最后一个子节点

## JavaScript 和 DOM

DOM 允许 JavaScript 操作所有 HTML 元素、HTML 属性和 CSS 样式。它还允许响应页面中发生的事件。DOM 允许 JavaScript 执行以下操作：

+   查找 HTML 元素

+   修改 HTML 元素的 CSS（层叠样式表）

+   修改 HTML 元素的文本内容（`innerHTML`）

+   响应 HTML DOM 事件

+   添加或删除 HTML 元素

### 查找 HTML 元素

DOM 允许以下方式查找 HTML 元素：

+   通过 ID

+   通过名称

+   通过标签名

+   通过类名

### 通过 ID 查找 HTML 元素

`getElementById`方法允许通过 ID 检索元素，例如：

```js
var id = document.getElementById("id1");
```

此方法将返回一个对象。如果找不到元素，ID 将包含一个 null 值。

### 通过标签名查找 HTML 元素

`getElementsByTagName`方法允许通过标签名检索元素，例如：

```js
var y = document.getElementsByTagName("p");
```

### 通过名称查找 HTML 元素

使用`getElementsByTagName`方法通过标签名检索元素，如下例所示。`getElementsByName`方法允许通过标签名检索元素，例如：

```js
var y = document.getElementsByName("userName");
```

### 通过 className 查找 HTML 元素

如下例所示，使用`getElementsByClassName`方法通过类名检索元素。`getElementsByClassName`允许通过类名检索元素，例如：

```js
var z  = document.getElementsByClassName(names);
```

### 注意

使用`getElementsByClassName`方法查找元素在 IE 5、6、7 和 8 中不起作用。

### 更改 HTML 内容

更改 HTML 元素的内容非常简单，因为它可以通过更改 HTML 元素的`innerHTML`属性来实现。使用以下语法：

```js
document.getElementById(elementid).innerHTML= new value
```

以下示例更改了 HTML 元素的内容：

```js
<html>
  <body>
    <h1 id="headerid">Heading</h1>
    <script>
      varelement=document.getElementById("headerid");
      element.innerHTML="New Heading";
    </script>
  </body>
</html>
```

### 更改 HTML 属性

如下例所示，使用`getElementById`方法检索 HTML 元素，并更改具有新值的属性：

```js
document.getElementById(elementid).attribute = NewAttributevalue
```

以下示例更改了具有新值的属性：

```js
<html>
  <body>
    <img id="logosmall" alt ="Logo">
    <script>
      document.getElementById("logosmall").alt="This is logo";
    </script>
  </body>
</html>
```

### 更改 HTML 样式

要更改 HTML 元素的样式，请使用以下代码：

```js
document.getElementById(elementid).style.property= new style
```

以下示例更改了`<p>`元素的样式：

```js
<html>
  <body>
    <p id="pid2">Color!</p>
    <script>
      document.getElementById("pid2").style.color="red";
    </script>
    <p>The paragraph above is changed to red.</p>
  </body>
</html>
```

此示例在用户点击按钮时更改具有`id="pid1"`的 HTML 元素的样式：

```js
<!DOCTYPE html>
<html>
  <body>
    <h1 id="pid1">My Heading 1</h1>
    <button type="button" onclick="document.getElementById('pid1').style.color='red'">ClickMe!</button>
  </body>
</html>
```

### 响应事件

DOM 允许在事件发生时执行 JavaScript 代码或函数；例如，当用户将鼠标悬停在 HTML 元素上时：

```js
object.onmouseover=function(){SomeJavaScriptCode};
```

以下是一些 HTML 事件的示例：

在以下示例中，从事件处理器中调用了一个函数：事件处理器允许我们调用 JavaScript 函数，如下所示：

```js
<html>
  <head>
    <script>
      function changetext(id) {id.innerHTML="Text is changed!"; }
    </script>
  </head>
  <body>
    <h1 onclick="changetext(this)">Click on this text!</h1>
  </body>
</html>
```

### 创建新的 HTML 元素

首先创建元素节点，然后将其附加到现有元素：

```js
<div id="div1">
  <p id="pid1">This is a paraOne.</p>
  <p id="pid2">This is paraTwo.</p>
</div>
<script>
var para=document.createElement("p");
var node=document.createTextNode("This is new Para.");
para.appendChild(node);
var element=document.getElementById("div1");
element.appendChild(para);
</script>
```

以下代码可以解释如下：

+   它创建了一个新的`<p>`元素

+   要向`<p>`元素添加文本，首先创建一个文本节点

+   将文本节点附加到`<p>`元素

+   它找到现有元素

+   它将新元素附加到现有元素上

### 移除现有 HTML 元素

在移除子元素之前，了解元素的父元素，然后移除它，例如：

```js
<div id="divid1">
  <p id="pid1">This is a paragraph.</p>
  <p id="pid2">This is another paragraph.</p>
</div>
<script>
//Find the element with id="divid1":
var parent=document.getElementById("divid1");
//Find the <p> element with id="pid1":
var child=document.getElementById("pid1");
//Remove the child from the parent:
parent.removeChild(child);
</script>
```

以下 HTML 文档包含一个具有两个子节点（两个`<p>`元素）的`<div>`元素：

```js
<div id="div1">
  Child: 1 <p id="pid1"> This is a paragraph. </p>
  Child: 2 <p id="pid2"> This is another paragraph. </p>
</div>
```

DOM 要求知道元素及其父元素都需要被移除。它不允许在没有父元素引用的情况下移除子元素。解决方案使用子对象的`parentNode`属性来获取父元素，然后使用`removeChild`来移除它：

```js
var childelement = document.getElementById("pid1");
childelement.parentNode.removeChild(childelement);
```

## DOM 和 QTP

QuickTest 对 Web 对象的属性允许我们获取 DOM 对象的引用，并可以在 DOM 对象上执行任何操作。例如，以下代码显示页面上的对象允许通过名称 `username` 获取元素，即下一步的文本框将值设置为 `ashish`。

```js
Set Obj = 
  Browser("Tours").Page("Tours").Object.getElementsByName("userName)
'Get the length of the objects
Print obj.length
'
obj(0).value="ashish"
```

以下代码片段显示了可以使用 Web 对象的 `Object` 属性执行的各种操作：

```js
'Retrieve all the link elements in a web page
Set links = Browser ("Mercury Tours").Page("Mercury Tours").Object.links
 'Length property provide total number of links 
For i =0 to links.length -1
  Print links(i).toString() 'Print the value of the links
Next
Get the web edit object and set the focus on it
Set MyWebEdit = Browser("Tours").Page("Tours").WebEdit("username").Object
MyWebEdit.focus
'Retrieve the html element by its name 
 obj = Browser("Tours").Page("Tours").Object.getElementsByName("userName")
 'set the value to the retrieved object 
 obj.value ="ashish"
'Retrieve the total number of the images in the web pages
set  images =  Browser("Mercury Tours").Page("Mercury Tours").Object.images
print  images.length
'Clicking on the Image
obj = Browser("Tours").Page("Mercury Tours").Object.getElementsByName("login")
obj.click
'Selecting the value from the drop down using selectedIndex method
 Browser("Flight").Page("Flight).WebList("pass.0.meal").Object.selectedIndex =1
'Click on the check box
Browser("Flight").Page("Flight").WebCheckBox("ticketLess").Object.click
```

### 触发事件

QTP 允许在 Web 对象上触发事件：

```js
Browser("The Fishing Website fishing").Page("The Fishing Website fishing").Link("Link").FireEvent "onmouseover"
```

以下示例使用 `FireEvent` 方法触发表单上的 `onpropertychange` 事件：

```js
Browser("New Page").Page("New Page").WebElement("html tag:=Form").FireEvent "onpropertychange"
```

QTP 允许执行 JavaScript 代码。有两个 JavaScript 函数允许我们与网页交互。我们可以检索对象并在它们上执行操作，或者我们可以从页面上的元素检索属性：

`RunScript` 执行作为此函数参数传递的 JavaScript 代码。

以下示例显示了 `RunScript` 方法如何调用 `ImgCount` 方法，该方法返回页面中的图像数量：

```js
length = Browser("Mercury Tours").Page("Mercury Tours").RunScript("ImgCount();  function  ImageCount() {var list = document.getElementsByTagName('img'); return  list.length;}")
print "The total number of images on the page is " & length
```

`RunScriptsFormFile` 使用 JavaScript 文件的完整路径来执行它。位置可以是绝对或相对文件系统路径，或者是一个质量中心路径。

以下是一个示例 JavaScript 文件（`logo.js`）：

```js
var myNode = document.getElementById("lga");
myNode.innerHTML = '';
```

使用以下代码中的 `logo.js` 文件：

```js
Browser("Browser").Page("page").RunScriptFromFile "c:\logo.js"
'Check that the Web page behaves correctly
If Browser("Browser").Page("page").Image("Image").Exist Then
    Reporter.ReportEvent micFail, "Failed to remove logo"
End If
```

上述示例使用 `RunScriptFromFile` 方法从网页中删除一个 DOM 元素，并检查在删除 DOM 元素后页面是否仍然表现正确。

# 使用 XPath

**XPath** 允许在 HTML 文档中导航和查找元素和属性。XPath 使用路径表达式在 HTML 文档中导航。QTP 允许使用 XPath 创建对象描述，例如：

```js
xpath:=//input[@type='image' and contains(@name,'findFlights')
```

在下一节中，我们将学习各种 XPath 术语和方法，以使用 XPath 查找对象。

## XPath 术语

XPath 使用各种术语来定义元素及其在 HTML 元素之间的关系，以下表格显示了这些术语：

| 原子值 | 原子值是没有子节点或父节点的节点 |
| --- | --- |
| 祖先 | 一个节点的父节点、父节点的父节点，依此类推 |
| 后代 | 一个节点的子节点、子节点的子节点，依此类推 |
| 父节点 | 每个元素和属性都有一个父节点 |
| 子节点 | 元素节点可以有零个、一个或多个子节点 |
| 同级 | 具有相同父节点的节点 |

## 选择节点

路径表达式允许在文档中选择节点。以下表格显示了常用的路径表达式：

| 符号 | 含义 |
| --- | --- |
| `/`(斜杠) | 选择相对于根节点的元素 |
| `//`(双斜杠) | 从当前节点选择文档中的节点，无论其位置如何都匹配选择 |
| `.`(点) | 表示当前节点 |
| `..` | 表示当前节点的父节点 |
| `@` | 表示一个属性 |
| `nodename` | 选择所有名为 "nodename" 的节点 |

斜杠 (`/`) 在开头使用时定义绝对路径；例如，`/html/head/title` 返回 `title` 标签。如果在中部使用，它定义祖先和后代关系；例如，`//div/table` 返回包含表格的 `div`。

双斜杠 (`//`) 用于在任意位置查找节点；例如，`//table` 返回所有表格。如果在中部使用，它定义后代关系；例如，`/html//title` 返回 `title` 标签，它是 `html` 标签的后代。

参考以下表格，查看更多示例及其含义：

| 表达式 | 含义 |
| --- | --- |
| `//a` | 查找所有锚点标签 |
| `//a//img` | 列出链接内的图像 |
| `//img/@alt` | 显示所有 `alt` 标签 |
| `//a/@href` | 显示每个链接的 `href` 属性 |
| `//a[@*]` | 带有任意属性的锚点标签 |
| `//title/text()` 或 `/html/head/title/text()` | 获取页面的标题 |
| `//img[@alt]` | 列出具有 `alt` 标签的图像 |
| `//img[not(@alt)]` | 列出没有 `alt` 标签的图像 |
| `//*[@id='mainContent']` | 获取具有特定 CSS ID 的元素 |
| `//div [not(@id="div1")]` | 从 XPath 创建数组元素 |
| `//p/..` | 选择 `p`（段落）子节点的父元素 |
| `XXX[@att]` | 选择 `XXX` 的所有具有名为 `att` 的属性子元素 |
| `./@*` 例如，`//script/./@*` | 查找当前元素的 所有属性值 |

## 断言

断言被嵌入在方括号中，用于查找特定节点或包含特定值的节点：

+   `//p[@align]`: 这允许找到所有具有对齐属性值为中心的标签

+   `//img[@alt]`: 这允许找到所有包含 `alt` 标签的 `img`（图像）标签

+   `//table[@border]`: 这允许找到所有包含 `border` 属性的 `table` 标签

+   `//table[@border >1]`: 这找到边框值大于 1 的表格

使用完整路径检索表格行：

```js
//body/div/table/tbody/tr[1]
```

获取 `//body/div/table/..`（`table` 标签的父节点）的名称

```js
//body/div/table/..[name()]
```

| 路径表达式 | 结果 |
| --- | --- |
| `//div/p[1]` | 选择 `div` 元素的第一个段落元素 |
| `//div/p [last()]` | 选择 `div` 元素的最后一个段落元素 |
| `//div/p[last()-1]` | 选择 `div` 元素的第二个最后一个段落元素 |
| `//div/p[position()<3]` | 选择 `div` 元素的第一个两个段落元素 |
| `//script[@language]` | 选择所有具有名为 `language` 的属性的脚本元素 |
| `//script[@language='javascript']` | 选择所有具有名为 `language` 的属性且值为 JavaScript 的脚本元素 |
| `//div/p[text()>45.00]` | 选择 `div` 元素的具有文本元素值大于 45.00 的所有段落元素 |

## 选择未知节点

除了在 XPath 中选择特定节点外，XPath 允许我们使用 `*`、`@` 和 `node()` 函数选择 HTML 元素的组。

+   `*` 代表元素节点

+   `@` 代表属性

+   `node()` 代表任何节点

之前提到的元素允许选择未知节点；例如：

+   `/div/*` 选择 `div` 元素的所有子节点

+   `//*` 选择文档中的所有元素

+   `//script[@*]` 选择包含属性的标题元素

## 选择多个路径

在 XPath 表达式中使用 **联合** | **运算符** 允许选择多个路径，如下表所示：

| 路径表达式 | 操作 |
| --- | --- |
| `//div | /p | //div/span` | 选择 `div` 元素的所有段落和 `span` 元素 |
| `//p | //span` | 选择文档中所有的 `p`（段落）和 `span` 元素 |

## XPath 中的轴

轴允许定义相对于其当前节点的节点。以下是在 XPath 中的轴列表：

+   自身

+   父节点

+   属性

+   子节点

+   祖先和祖先或自身

+   后代和后代或自身

+   后续和后一个兄弟

+   前一个和前一个兄弟

+   命名空间

## 使用位置路径定位元素

元素的位置可以用绝对和相对路径两种方式表示。

每一步都是针对当前节点中的节点进行评估。语法看起来如下：

```js
axisname::nodetest[predicate]
```

轴定义了所选节点与当前节点之间的树形关系。

节点测试允许我们在轴中识别节点。

零个或多个谓词允许进一步细化所选节点集。

看一下以下示例：

```js
//input[ancestor-or-self::*[@name='userName']]
```

在前面的示例中，`ancestor-or-self` 是一个轴，`::*` 是节点测试，`[@name='userName']` 是谓词，允许我们搜索所有包含在 `input` 类型元素 `//input[ansector-or-self::*[@name='userName']]` 中的 `ancestor-or-self` 元素。

轴允许查找元素的后代；例如：

`//div[descendant::p]` 允许我们找到所有具有后代为段落的 `div` 元素。

`descendant::p` 轴允许查找元素的后代和祖先，例如，查找按钮的所有祖先或查找所有后代为 `table` 的 `div`。

```js
//input[Ancestor::div[@align='center']]
//div[descendant::table]
```

XPath 函数的使用：

+   `//div/text()`：使用 `text` 函数从 `div` 标签中检索文本

+   `//div/node()`：使用 `node` 函数获取 `div` 标签下的所有节点

要获取后代，XPath 函数 `start-with` 查找以 `//a[starts-with(@href,'http://ads')]` 开头的值。

查找包含 `style` 属性的所有标签 `//*[@style]`。

## XPath 中的运算符

XPath 表达式使用运算符来构建要评估的条件。以下为 XPath 运算符列表：

| 运算符类别 | 运算符 | 描述 |
| --- | --- | --- |
| 数学 | +-*DivMOD | 加法减法乘法除法 MOD 模数（除法余数） |
| 等于和比较 | =!=<<=>>= | 等于不等于小于小于等于大于大于等于 |
| 逻辑 | OrAnd | 逻辑或逻辑与 |
| 两个节点集 | &#124; 并集 | 计算两个节点集 |

以下表格显示了使用 XPath 表达式中的运算符的一些示例：

| 表达式 | 操作 |
| --- | --- |
| `//input &#124; //div/input/..` | `Input` 联合 `div` 的父级中的 `input` |
| `//table[@border>=1]` | 获取边框大于 1 的表格 |
| `//table[@id=3]/td[2]/text()+1` | 从第三行第二列检索文本并将其加 1 |
| `//tr[@id=1] and //tr[@id=3]` | 如果两个表达式都为真，则返回 true |

使用 XPath 在查找兄弟元素时的示例，允许找到与它们的兄弟相关的对象，而不是与它们的位置相关：

```js
<html>
<Body>
<br><br>
<table name="maintable" border="1">
<th>Item</th><th>Quantity</th><th>Price</th><th></th>
<tr id=1 width="600px">
<td> HP LoadRuuner </td>
<td > 01 </td>
<td> $27 </td>
<td> <input type="button" value="AddToCart"> </td>
</tr> 
<tr id=2 width="600px">
<td> HP Quality Center </td>
<td> 01 </td>
<td> $29 </td>
<td> <input type="button" value="AddToCart"> </td>
</tr>
<tr id=3 width="600px">
<td > HP QuickTest Professional </td>
<td > 01 </td>
<td> $25 </td>
<td> <input type="button" value="AddToCart"> </td>
</tr>
</table>
</body>
</html>
```

将代码添加到文件中，并保存为 `.html` 文件。点击 **AddToCart** 按钮。

删除以下记录的脚本：

```js
Browser("Browser").Page("Page").WebButton("AddToCart").Click
```

替换并使用以下脚本：

```js
Browser("Browser").Page("Page").WebButton("xpath:=//td[contains(text(),'HP QuickTest')]/following-sibling::td[3]/input").Click
```

此前的脚本允许点击按钮（**AddToCart**），该按钮是 HP QuickTest Professional 文本的兄弟。要点击与 HP Quality Center 文本为兄弟的 **AddToCart** 按钮，请使用以下代码：

```js
Browser("Browser").Page("Page").WebButton("xpath:=//td[contains(text(),'HP Quality Center')]/following-sibling::td[3]/input").Click
```

XPath 可以在 QTP 脚本中实现。以下部分解释了 XPath 在网页中的各种用法。XPath 用法的示例包括：

+   在文本框中输入值

+   从下拉列表中选择值

+   点击图片和按钮

+   通过 ID、标签名、名称和属性查找对象

+   允许提取信息进行验证

+   将重复对象作为数组下标处理

使用 XPath 创建框架的关键步骤包括：

1.  识别要记录的流程。

1.  创建 OR（为每个页面创建 OR 或使用对象描述）。

1.  识别唯一标识对象的 XPath。

1.  定义允许我们检索元素并定义为常量的 XPath。

1.  将它们用作对象描述。

使用 XPath 编写脚本的技巧：

+   使用描述性编程（使用 `index` 属性）处理 XPath 表达式中的重复对象或数组下标

+   XPath 允许使用索引来收集元素

+   使用 `GetROProperty` 获取 `innerHTML` 以获取值

+   使用 XPath 作为常量定义对象描述，以便更容易维护，并且只需在一个地方更改，无需在整个脚本中更改

# 摘要

在本章中，我们学习了各种概念和示例。DOM、JavaScript 和 XPath 的知识对于编写脚本非常有用。XPath 和 DOM 是在 QTP 中用于脚本创建和验证最少使用的概念，但通过方便地从网页中检索数据，这些概念在创建手动检查点时非常有帮助。当需要处理或靠近的元素没有 ID 时，路径是一个真正好的导航网站的方法。下一章将讨论在测试自动化框架设计过程中捕获和分享所学到的经验，并将它们用于未来的参考。
