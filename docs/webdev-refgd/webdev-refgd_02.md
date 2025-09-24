# 第二章。HTML 属性

HTML 是使用元素构建的。这些元素中的每一个都将具有属性。这些属性可以改变浏览器渲染元素的方式、配置它以及涉及该元素的行为。

本章的重点将完全集中在属性上。如果你不确定哪些属性适用于哪些元素，上一章几乎涵盖了几乎所有元素及其属性。

# 全局属性

这些是适用于每个 HTML 元素的属性。然而，你应该注意，尽管属性是可用的，但这并不意味着它实际上会做任何事情。

## accesskey

`accesskey` 属性创建一个键盘快捷键以激活或聚焦到元素：

```js
<element accesskey></element>
```

### 描述

`accesskey` 属性允许您创建键盘快捷键。这可以是一个由空格分隔的字符列表。大多数 Windows 上的浏览器将使用 *Alt* + accesskey，而大多数 Mac 上的浏览器将使用 *Ctrl* + *Option* + accesskey。

这里是一个使用可以聚焦于字符 `q` 的文本框的示例：

```js
<input type="search" name="q" accesskey="q"/>
```

## class

`class` 属性通常用于帮助对相似元素进行分组以供 CSS 选择器使用：

```js
<element class></element>
```

### 描述

`class` 属性是最常用的属性之一。`class` 属性允许 CSS 定位多个元素并将样式应用于它们。除此之外，许多人还使用 `class` 属性来帮助在 JavaScript 中定位元素。

类名使用空格分隔的列表。

这里是一个将 `search-box` 类应用于元素的示例：

```js
<input type="search" name="q" class="search-box"/>
```

## contenteditable

`contenteditable` 属性将元素的内容设置为可编辑：

```js
<element contenteditable></element>
```

### 描述

`contenteditable` 属性告诉浏览器用户可以编辑元素中的内容。`contenteditable` 属性的值应该是 `true` 或 `false`，其中 `true` 表示元素是可编辑的。

这里是一个使用 `div` 元素的示例：

```js
<div contenteditable="true">Click here and edit this sentence!</div>
```

## data-*

`data-*` 属性是元素的定制属性：

```js
<element data-*></-></element>
```

### 描述

你可以给数据属性起任何名字，只要名字不以 XML 开头，不使用任何分号，并且没有大写字母。值可以是任何内容。

这里是一个具有 `data-id` 属性的项目列表。请注意，属性名 `data-id` 是任意的。你在这里可以使用任何有效的名称：

```js
<li data-id="1">First Row</li>
<li data-id="2">Second Row</li>
<li data-id="3">Third Row</li>
```

## dir

`dir` 属性定义文本方向：

```js
<element dir></element>
```

### 描述

`dir` 属性是方向属性。它指定文本方向。以下是其可能的值：

+   `auto`：这允许浏览器自动选择方向

+   `ltr`：这允许浏览器选择从左到右的方向

+   `rtl`：浏览器选择从右到左的方向

这里是 `ltr` 和 `rtl` 属性的示例。

```js
<div dir="ltr">Left to Right</div>
<div dir="rtl">Right to Left</div>
```

## draggable

`draggable` 属性定义元素是否可拖动：

```js
<element draggable-></element>
```

### 描述

`draggable` 属性允许元素被拖动。请注意，大多数元素还需要 JavaScript 才能完全工作：

这里是一个示例：

```js
<div draggable="true">You can drag me.</div>
```

### hidden

`hidden`属性阻止元素的渲染：

```js
<element hidden></element>
```

### 描述

`hidden`属性用于隐藏元素。但是，隐藏的元素可以通过 CSS 或 JavaScript 覆盖并显示。这是一个布尔属性，因此包含此属性将值设置为`true`，排除它将值设置为`false`。

这里是一个示例：

```js
<div hidden>This should not show</div>
```

## id

`id`属性是元素的唯一标识符：

```js
<element id></element>
```

### 描述

`id`属性是元素的唯一标识符。这用于片段链接以及在 JavaScript 和 CSS 中轻松访问元素。

这里是一个使用`div`元素的示例：

```js
<div id="the-first-div">This is the first div.</div>
```

## lang

`lang`属性定义元素中使用的语言：

```js
<element lang></element>
```

### 描述

`lang`属性设置元素的语言。可接受的值应该是 BCP47 语言。这里不一一列举，但您可以在[`www.ietf.org/rfc/bcp/bcp47.txt`](http://www.ietf.org/rfc/bcp/bcp47.txt)上阅读标准。语言对于屏幕阅读器使用正确的发音等重要事项很重要。

这里是一个简单的英文示例：

```js
<div lang="en">The language of this element is English.</div>
```

## spellcheck

`spellcheck`属性指定是否可以使用拼写检查：

```js
<element spellcheck></element>
```

### 描述

`spellcheck`属性是在 HTML5 中引入的。它将告诉浏览器是否对该元素进行拼写检查。值应该是`true`或`false`。

这里是一个使用`textarea`的示例：

```js
<textarea spellcheck="false">Moste fo teh worsd r mispeld.</textarea>
```

## style

`style`属性用于设置内联样式：

```js
<element style></element>
```

### 描述

您可以直接使用`style`属性将 CSS 样式添加到元素中。您在这里可以使用任何在 CSS 中可以使用的样式规则。请记住，这将覆盖 CSS 中定义的任何其他样式。

这里是一个示例，将背景设置为红色，文本设置为白色：

```js
<div style="background: #ff0000; color: #ffffff">This has inline styles.</div>
```

## tabindex

`tabindex`属性设置 Tab 顺序：

```js
<element tabindex></element>
```

### 描述

`tabindex`属性元素定义了当使用*Tab*键时元素将聚焦的顺序。您可以使用三种不同类型的值。第一个是负数。这表示它不在元素列表中。`zero`值表示浏览器应确定此元素的顺序。这通常是元素在文档中出现的顺序。这是一个正数，它将设置 Tab 顺序。

以下示例演示了您可以设置与文档中元素顺序不同的`tabindex`：

```js
<input type="text" tabindex="1" />
<input type="text" tabindex="3" />
<input type="text" tabindex="2" />
```

## title

`title`属性是提示文本：

```js
<element title></element>
```

### 描述

`title`属性提供了关于元素的额外信息。通常，标题会作为元素的提示显示。例如，当使用图像时，这可能是图像的名称或照片的版权信息：

```js
<p title="Some extra information.">This is a paragraph of text.</p>
```

# 杂项

属性的杂项分组将没有层次结构，因为它们可以用于许多不同的元素。

## accept

`accept`属性为服务器提供类型列表：

```js
<element accept></element>
```

### 元素

在`accept`属性中使用的元素是`form`和`input`。

### 描述

`accept` 属性允许你建议此表单或输入应接受的文件类型。你可以使用 `audio/*`、`video/*`、`image/*`、MIME 类型或扩展名。

这里有一个寻找 PNG 文件的例子：

```js
<input type="file" accept=".png, image/png"/>
```

## accept-charset

`accept-charset` 属性提供了支持的字符集列表：

```js
<element accept-charset></element>
```

### 元素

`form` 元素用于 `accept-charset` 属性。

### 描述

`accept-charset` 属性设置了表单将接受的字符集。UTF-8 最常用，因为它接受许多语言中的许多字符。

这里是使用 `charset` 属性的一个例子：

```js
<form accept-charset="UTF-8"></form>
```

## action

`action` 属性是表单被处理的地方，其语法如下：

```js
<element action ></element>
```

### 元素

`form` 元素用于 `action` 元素。

### 描述

`action` 属性包含将处理表单数据的 URL。

这里有一个例子：

```js
<form action="form-process.php"></form>
```

## alt

`alt` 属性是元素的替代文本：

```js
<element alt ></element>
```

### 元素

在 `alt` 属性中使用的元素有 `applet`、`area`、`img` 和 `input`。

### 描述

`alt` 属性是当元素无法渲染代码时的替代文本。文本应向用户传达与图像相同的信息。

一些浏览器（Chrome 是最常用的）不会显示文本，你可能需要使用 `title` 属性。这不是标准化的行为。

这里使用了一个图像的例子：

```js
<img alt="Login button." src="img/non-loading-image.png"/>
```

## async

`async` 属性用于脚本的异步执行：

```js
<element async ></element>
```

### 元素

`script` 元素用于 `async` 属性。

### 描述

`async` 属性告诉浏览器异步加载和执行脚本。默认情况下，浏览器将同步加载脚本。异步加载将立即下载并解析脚本，而不会阻塞页面渲染。

这里有一个例子：

```js
<script src="img/application.js" async></script>
```

## autocomplete

`autocomplete` 属性定义了元素是否可以被自动完成：

```js
<element autocomplete ></element>
```

### 元素

`form` 和 `input` 元素用于 `autocomplete` 属性。

### 描述

`autocomplete` 属性让浏览器知道是否可以从之前的值自动完成表单或输入。这可以有 `on` 和 `off` 作为值。

这里有一个例子：

```js
<input type="text" autocomplete="off" placeholder="Will not autocomplete"/>
```

## autofocus

`autofocus` 属性定义了元素是否会在元素上自动聚焦：

```js
<element autofocus ></element>
```

### 元素

`button`、`input`、`select` 和 `textarea` 元素用于 `autofocus` 属性。

### 描述

`autofocus` 属性将焦点设置到元素上。这应该只用于一个元素。

这里是 `autofocus` 属性与文本输入的例子：

```js
<input type="text" autofocus/>
```

## autoplay

`autoplay` 属性定义了音频或视频轨道是否应该尽可能快地播放：

```js
<element autoplay ></element>
```

### 元素

`audio` 和 `video` 元素用于 `autoplay` 属性。

### 描述

`autoplay` 属性将使元素一有机会就播放，而无需停止加载。

这里有一个包含音频文件的例子：

```js
<audio autoplay src="img/audio.mps"></audio>
```

## autosave

`autosave` 属性定义是否应保存先前值：

```js
<element autosave ></element>
```

### 元素

`input` 元素用于 `autosave` 属性。

### 描述

`autosave` 属性指示浏览器保存输入到此输入框中的值。这意味着在下次页面加载时，这些值将被保留。它应该有一个浏览器可以关联保存值的唯一名称。此属性在某些浏览器中可能不起作用，因为它尚未完全标准化。

这里是一个示例：

```js
<input type="text" autosave="textautosave" />
```

## cite

`cite` 属性包含引用的来源：

```js
<element cite ></element>
```

### 元素

`blockquote`、`del`、`ins` 和 `q` 元素用于 `cite` 属性。

### 描述

`cite` 属性通过提供 URL 来指向引用的来源。

这里是一个示例：

```js
<blockquote cite="http://en.wikiquote.org/wiki/The_Good,_the_Bad_and_the_Ugly">
    After a meal there's nothing like a good cigar.
</blockquote>
```

## cols

`cols` 属性指定列数：

```js
<element cols ></element>
```

### 元素

`textarea` 元素与 `cols` 属性一起使用。

### 描述

`cols` 属性指定 `textarea` 元素中的列数。

这里是它在使用中的示例：

```js
<textarea cols="30"></textarea>
```

## colspan

`colspan` 属性指定单元格应跨越的列数：

```js
<element colspan ></element>
```

### 元素

`td` 和 `th` 元素用于 `colspan` 属性。

### 描述

`colspan` 属性指定表格单元格应跨越的列数。

这里是一个使用表格元素的示例：

```js
<table>
    <tr><td colspan="2">1 and 2</td></tr>
    <tr><td>1</td><td>2</td></tr>
</table>
```

## datetime

`datetime` 属性提供与此元素关联的日期和时间：

```js
<element datetime ></element>
```

### 元素

`del`、`ins` 和 `time` 元素与 `datetime` 属性一起使用。

### 描述

`datetime` 属性应该是执行元素所暗示动作的时间。此属性主要用于 `del` 和 `ins` 元素，以显示删除或插入发生的时间。

这里是一个使用 `del` 的示例：

```js
My name is <del datetime="2014-12-16T23:59:60Z">John</del>Josh.
```

## disabled

`disabled` 属性定义元素是否可使用：

```js
<element disabled ></element>
```

### 元素

`button`、`fieldset`、`input`、`optgroup`、`option`、`select` 和 `textarea` 元素用于 `disabled` 属性。

### 描述

`disabled` 属性使元素不可用。如果元素被禁用，则无法使用。这意味着按钮无法点击，文本区域和文本输入无法输入文本，下拉列表无法更改，等等。

这里是一个使用 `button` 元素的示例：

```js
<button disabled>This is a disabled button</button>
```

## download

`download` 属性设置链接以下载资源：

```js
<element download ></element>
```

### 元素

`a` 元素用于 `download` 属性。

### 描述

`download` 属性指示当点击时浏览器下载资源。这意味着当点击 `a` 元素时，将出现一个保存对话框，并将属性值作为默认名称。

这里是一个示例：

```js
<a href="example.pdf" download="example.pdf">Save the PDF</a>
```

## content

`content` 属性为 `name` 属性提供值：

```js
<element content ></element>
```

### 元素

`meta` 元素用于 `content` 属性。

### 描述

`content` 属性是 `meta` 标签的属性。它用作 `name` 属性的值。

这里是一个示例：

```js
<meta name="example" content="value for example" />
```

## controls

`controls` 属性定义是否应显示控件：

```js
<element controls ></element>
```

### Elements

`audio`和`video`元素用于`controls`属性。

### 描述

`controls`属性告诉浏览器显示媒体文件的控件。这是一个布尔属性。

下面是一个音频示例：

```js
<audio controls src="img/example.mp3"></audio>
```

## for

`for`属性设置与该属性关联的元素：

```js
<element for ></element>
```

### 元素

`label`元素与`for`属性一起使用。

### 描述

`for`属性指定与标签关联的表单输入。这使用输入元素的 ID 指定。标签还将允许用户点击它并聚焦到输入。

下面是一个带有文本输入的示例：

```js
<label for="username">Username</label>
<input type="text" id="username" name="username" />
```

## 表单

`form`属性设置与该输入关联的表单：

```js
<element form ></element>
```

### 元素

`button`、`fieldset`、`input`、`labellable`、`object`、`output`、`select`和`textarea`元素用于`form`属性。

### 描述

`form`属性引用这些控件所在的表单：

```js
<form method="get" id="example-form">

</form>
<input type="text" form="example-form" />
```

## formaction

`formaction`属性设置元素的形式操作：

```js
<element formaction ></element>
```

### 元素

`button`和`input`元素用于`formaction`属性中。

### 描述

`formaction`属性将覆盖此元素的表单操作。这应该是一个 URL。如果用于表单元素本身，此属性指定表单数据的目标。如果用于表单内的元素（例如，按钮），则覆盖表单本身声明的值。

下面是一个带有按钮的示例：

```js
<form method="get" action="formaction.php">
    <button formaction="buttonaction.php">Press me</button>
</form>
```

## 高度

`height`属性设置元素的高度：

```js
<element height ></element>
```

### 元素

`canvas`、`embed`、`iframe`、`img`、`input`、`object`和`video`元素用于`height`属性。

### 描述

`height`属性设置元素的高度。只有上一节中列出的元素应该使用此属性，所有其他元素应使用 CSS 设置其高度。

### 注意

您可能会看到许多使用`height`的 HTML 文档。这不再有效，应使用 CSS 设置任何其他元素的高度。

下面是一个带有 canvas 元素的示例：

```js
<canvas height="400" width="400"></canvas>
```

## href

`href`属性给出资源的 URL：

```js
<element href ></element>
```

### 元素

`a`、`base`和`link`元素用于`href`属性。

### 描述

元素的 URL 由`href`属性给出。

下面是一个带有锚点元素的示例：

```js
<a href="http://www.google.com">Google</a>
```

## hreflang

`hreflang`属性声明资源的语言：

```js
<element hreflang ></element>
```

### 元素

`a`和`link`元素用于`hreflang`属性。

### 描述

`hreflang`属性是链接文档的语言。可接受的值应遵循 BCP47 语言标准。这里不一一列出，但您可以在[`www.ietf.org/rfc/bcp/bcp47.txt`](http://www.ietf.org/rfc/bcp/bcp47.txt)上阅读标准。

下面是一个示例：

```js
<a href="http://www.google.com" hreflang="en">Google</a>
```

## 标签

`label`属性声明轨道的标题：

```js
<element label ></element>
```

### 元素

`track`元素用于`label`属性。

### 描述

`label`属性与`track`元素一起使用，为轨道提供标题。

下面是一个带有视频副标题的示例：

```js
<video src="img/sample.mp4">
    <track kind="subtitles" label="English Subtitles" src="img/en.vtt" />
</video>
```

## 列表

`list` 属性给出选项列表：

```js
<element list ></element>
```

### 元素

`input` 元素用于 `list` 属性。

### 描述

`list` 属性与 `datalist` 属性相关联，为 `input` 提供选项列表。

这个示例有一个用于文本输入的水果列表：

```js
<input type="text" list="fruit" />
<datalist id="fruit">
    <option>Apple</option>
    <option>Banana</option>
</datalist>
```

## loop

`loop` 属性定义元素是否应该循环媒体：

```js
<element loop ></element>
```

### 元素

`audio` 和 `video` 元素用于 `loop` 属性。

### 描述

`loop` 属性是一个布尔属性，它将循环播放媒体。

这里是一个带有 `audio` 元素的示例：

```js
<audio src="img/example.mp3" loop></audio>
```

## max

`max` 属性定义最大值：

```js
<element max ></element>
```

### 元素

`input` 和 `progress` 元素用于 `max` 属性。

### 描述

`max` 属性设置允许的最大数值或日期时间值。

这里是一个带有输入的示例：

```js
<input type="number" min="0" max="5" >
```

## maxlength

`maxlength` 属性定义最大字符数：

```js
<element maxlength ></element>
```

### 元素

`input` 和 `textarea` 元素用于 `maxlength` 属性。

### 描述

`maxlength` 属性设置最大字符数。

这里是一个带有输入的示例：

```js
<input type="text" maxlength="5">
```

## media

`media` 属性设置链接资源的媒体：

```js
<element media ></element>
```

### 元素

`a`、`area`、`link`、`source` 和 `style` 元素用于 `media` 属性。

### 描述

`media` 属性指定此资源针对的媒体。通常，此属性与链接和 CSS 一起使用。标准值是 `screen`、`print` 和 `all`。`screen` 值用于在显示器上显示，`print` 值用于打印，`all` 值是两者都适用。不同的浏览器确实有几个其他值，但没有一个可以在所有浏览器上工作。

这里是一个使用 CSS 创建打印样式的示例：

```js
<link rel="stylesheet" href="print.css" media="print"/>
```

## method

`method` 属性定义表单的 HTTP 方法：

```js
<element method ></element>
```

### 元素

`form` 元素用于 `method` 属性。

### 描述

`method` 属性设置表单的 HTTP 方法。两个值是 `GET`，这是默认值，和 `POST`。

这里是一个使用 `POST` HTTP 方法提交的表单示例：

```js
<form method="post" action="formaction.php"></form>
```

## min

`min` 属性定义最小值：

```js
<element min ></element>
```

### 元素

`input` 元素用于 `min` 属性。

### 描述

`min` 属性是 `max` 的反义词。它为输入设置最小值。

这里是一个示例：

```js
<input type="number" min="2">
```

## multiple

`multiple` 属性定义是否可以选择多个值：

```js
<element multiple ></element>
```

### 元素

`select` 元素用于 `multiple` 属性。

### 描述

`multiple` 属性允许你选择多个值。这是一个布尔属性。

这里是一个示例：

```js
<select multiple>
    <option>First</option>
    <option>Second</option>
</select>
```

## name

`name` 属性是元素的名字：

```js
<element name ></element>
```

### 元素

`button`、`form`、`fieldset`、`iframe`、`input`、`object`、`select`、`textarea` 和 `meta` 元素用于 `name` 属性。

### 描述

`name` 属性命名一个元素。这允许你在提交的表单中获取元素的值。

这里是一个带有输入的示例：

```js
<input type="text" id="username" name="username" />
```

## novalidate

`novalidate` 属性定义是否跳过验证：

```js
<element novalidate ></element>
```

### 元素

`form` 元素用于 `novalidate` 属性。

### 描述

`novalidate` 属性设置表单在提交时不进行验证。浏览器将在不添加任何客户端代码的情况下验证输入。这是一个布尔属性。

这里是一个示例：

```js
<form method="post" action="formaction.php" novalidate></form>
```

## pattern

`pattern` 属性定义一个正则表达式：

```js
<element pattern ></element>
```

### 元素

`input` 元素用于 `pattern` 属性。

### 描述

您可以使用正则表达式在此属性中验证输入。

这里是一个只有数字才有效的示例：

```js
<input pattern="[0-9].+" type="text" />
```

## placeholder

`placeholder` 属性为元素中的用户提供提示：

```js
<element placeholder ></element>
```

### 元素

`input` 和 `textarea` 元素用于 `placeholder` 属性。

### 描述

当元素未被交互（没有值且不在焦点中）时，它将显示此属性中的文本。一旦元素被交互，值将消失。

这里是一个使用 `input` 的示例：

```js
<input type="text" name="username" placeholder="Please enter username"/>
```

## poster

`poster` 属性为视频提供图像：

```js
<element poster ></element>
```

### 元素

`video` 元素用于 `poster` 属性。

### 描述

`poster` 属性应指向一个图像，该图像将成为视频元素的封面（或占位符），直到视频加载。

这里是一个示例：

```js
<video src="img/video-about-dogs.mp4" poster="images/image-of-dog.png"></video>
```

## readonly

`readonly` 属性定义元素是否可编辑：

```js
<element readonly ></element>
```

### 元素

`input` 和 `textarea` 元素用于 `readonly` 属性。

### 描述

`readonly` 属性使元素只读或不可编辑。这是一个布尔属性。

这里是一个使用文本输入的示例：

```js
<input type="text" readonly />
```

## rel

`rel` 属性定义元素的关系：

```js
<element rel ></element>
```

### 元素

`a` 和 `link` 元素用于 `rel` 属性。

### 描述

`rel` 属性是链接资源与文档之间的关系。通常，您会看到它与 `link` 元素和 CSS 或与 `a` 元素和 `nofollow` 的值一起使用。

这里是一个 CSS 示例：

```js
<link rel="stylesheet" href="style.css" type="text/css" media="screen" />
```

## required

`required` 属性定义在提交表单时元素是否为必填项：

```js
<element required ></element>
```

### 元素

`input`、`select` 和 `textarea` 元素用于 `required` 属性。

### 描述

`required` 属性使元素在表单中成为必填项。表单在元素填写完毕之前不会提交。这是一个布尔属性。

这里是一个使用文本输入的示例：

```js
<input required type="text" />
```

## reversed

`reversed` 属性改变列表的显示顺序：

```js
<element reversed ></element>
```

### 元素

`ol` 元素用于 `reversed` 属性。

### 描述

`reversed` 属性仅是有序列表的属性，它将以反向顺序渲染列表。项目本身不是反向的，而是编号列表的指示器是。这是一个布尔属性。

这里是一个示例：

```js
<ol reversed>
    <li>1</li>
    <li>2</li>
    <li>3</li>
</ol>
```

## 行

`rows` 属性设置文本区域的行数：

```js
<element rows ></element>
```

### 元素

`textarea` 元素用于 `rows` 属性。

### 描述

`rows` 属性设置 `textarea` 中的行数。

这里是一个示例：

```js
<textarea rows="30"></textarea>
```

## rowspan

`rowspan` 属性设置单元格跨越的行数：

```js
<element rowspan ></element>
```

### 元素

`td` 和 `th` 元素用于 `rowspan` 属性。

### 描述

`rowspan` 属性用于表格单元格元素。它将使单元格跨越指定行数的行。

这里是一个示例：

```js
<table>
    <tr><td rowspan="2">Pets</td><td>Dogs</td></tr>
    <tr><td>Cats</td></tr>
</table>
```

## scope

`scope` 属性定义了与元素关联的单元格：

```js
<element scope ></element>
```

### 元素

`td` 和 `th` 元素用于 `scope` 属性。

### 描述

`scope` 属性指示单元格关联的内容。例如，这可以是 `row` 或 `col`。这是一个很大的好处，因为它有助于提高可访问性。当使用屏幕阅读器时，它有助于传达表格中行和列之间的关系。

这里是一个带有表格单元格的示例：

```js
<table>
    <tr><th>Name</th><th>Age</th></tr>
    <tr><td scope="row">Gizmo</td><td>4</td></tr>
</table>
```

## selected

`selected` 属性设置默认选择：

```js
<element selected ></element>
```

### 元素

`option` 元素用于 `selected` 属性。

### 描述

`selected` 属性将在页面加载时设置选项为选中状态。每个 `select` 元素中应只设置一个 `option` 元素的 `selected` 属性。这是一个布尔属性。

这里是一个示例：

```js
<select>
    <option>Cats</option>
    <option selected>Dogs</option>
</select>
```

## size

`size` 属性设置元素的宽度：

```js
<element size ></element>
```

### 元素

`input` 和 `select` 元素用于 `size` 属性。

### 描述

`size` 属性确定元素的宽度，除非元素是 `input` 和文本或密码。在这种情况下，它将确定元素可以显示的字符数。这以整数形式指定字符数。默认值是 `20`。

这里是一个带有文本输入的示例：

```js
<input type="text" size="100" />
```

## src

`src` 属性给出了元素的 URL：

```js
<element src ></element>
```

### 元素

`audio`、`embed`、`iframe`、`img`、`input`、`script`、`source`、`track` 和 `video` 元素用于 `src` 属性。

### 描述

`src` 属性给出了元素的资源 URL。

这里是一个带有 `image` 元素的示例：

```js
<img src="img/dogs.png" />
```

## start

`start` 属性设置有序列表的起始数字：

```js
<element start ></element>
```

### 元素

`ol` 元素用于 `start` 属性。

### 描述

`start` 属性将改变有序列表的起始数字。

这里是一个示例：

```js
<ol start="10">
    <li>1</li>
    <li>2</li>
</ol>
```

## step

`step` 属性确定每个数字之间的跳跃：

```js
<element step ></element>
```

### 元素

`input` 元素用于 `step` 属性。

### 描述

`step` 属性与 `input` 元素和 `type` 属性为数字或日期时间的 `input` 元素一起使用。它确定每次增加的跳跃距离。

这里是一个带有数字输入的示例。在达到最大值之前，您只能增加 `5` 四次：

```js
<input type="number" min="0" max="20" step="5" />
```

## type

`type` 属性定义了元素的类型：

```js
<element type ></element>
```

### 元素

`button`、`input`、`embed`、`object`、`script`、`source` 和 `style` 元素用于 `type` 属性。

### 描述

`type` 属性是最复杂的属性之一。它可以完全改变元素的外观和行为。例如，`input`。

这里是每个元素的列表：

+   `button`：以下为 `button` 属性的值：

    +   `button`：这是默认值

    +   `reset`：这会重置表单

    +   `submit`：这会提交表单

+   `input`：请参阅上一章中的输入元素部分。

+   `embed`：这将是指定嵌入资源的 MIME 类型。

+   `object`：这将是指定对象的 MIME 类型。

+   `script`：这将是指定的 MIME 类型。通常使用 `text`/`javascript`。

+   `source`：这将是指定的 MIME 类型。

+   `style`：这将是指定的 MIME 类型。通常使用 `text`/`css`。

这里有一个使用输入元素的例子：

```js
<input type="password" />
```

## value

`value` 属性设置元素的值：

```js
<element value ></element>
```

### 元素

`button`、`input`、`li`、`option`、`progress` 和 `param` 元素用于 `value` 属性。

### 描述

`value` 属性将在页面加载时设置元素的值。

这里有一个使用文本输入元素的例子：

```js
<input type="text" value="Hey!"/>
```

## width

`width` 属性设置元素的宽度：

```js
<element width ></element>
```

### 元素

`canvas`、`embed`、`iframe`、`img`、`input`、`object` 和 `video` 元素用于 `width` 属性。

### 描述

`width` 属性设置元素的宽度。

### 注意

你可能会看到许多使用宽度属性的 HTML 文档。这已经不再有效；应该使用 CSS 来设置其他元素的宽度。

这里有一个使用 `canvas` 元素的例子：

```js
<canvas width="200" height="200"></canvas>
```

## wrap

`wrap` 属性设置文本的换行方式：

```js
<element wrap ></element>
```

### 元素

`textarea` 元素用于 `wrap` 属性。

### 描述

`wrap` 属性决定文本是否可以在 `textarea` 中换行。其值可以是硬值或软值。硬值会在文本中插入换行符。它还必须与 `cols` 属性一起使用，以便知道行尾的位置。软值不会插入任何换行符。

这里有一个例子：

```js
<textarea cols="10" wrap="hard"></textarea>
```
