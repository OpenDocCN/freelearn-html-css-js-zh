# 第三章。CSS 概念和应用

**层叠样式表**（**CSS**）是设置 HTML 样式的首选方式。HTML 有一个样式元素和一个全局样式属性。这些使得编写难以维护的 HTML 变得非常容易。例如，让我们假设我们在一个 HTML 页面上有 10 个元素，我们希望这些元素的字体颜色为红色。我们创建一个`span`元素来包裹具有红色字体颜色的文本，如下所示：

```js
<span style="color: #ff0000;"></span>
```

如果我们决定将颜色更改为蓝色，我们不得不更改该元素 10 次实例，然后乘以我们在页面上使用`span`元素的数量。这是完全不可维护的。

这就是 CSS 发挥作用的地方。我们可以针对我们希望应用特定样式的特定元素或元素组进行定位。CSS 允许我们定义这些样式，轻松更新它们，并从一处更改到另一处。

本书将重点介绍最常用的 CSS 选择器、单位、规则、函数和属性，这些来自 CSS1、CSS2.1 和 CSS3。在大多数情况下，这些都应该在任何浏览器中工作，但也有一些例外。一个很好的经验法则是新浏览器将出现更少的问题。

我们将快速概述不同类型的基本选择器。

# 基本选择器

选择器代表一个结构。这种表示随后用于 CSS 规则中，以确定哪些元素被选中并应用此规则中的样式。CSS 样式规则以瀑布效应的形式应用。每个匹配的规则也会传递给其每个子规则，根据选择器的权重进行匹配和应用。本节将仅关注最基本的选择器。

基本选择器包括类型选择器、通用选择器、属性选择器、类选择器、ID 选择器和伪类。

### 注意

所有 CSS 选择器都是不区分大小写的。选择器也可以分组在一起以共享规则。要分组选择器，只需用逗号分隔它们。考虑以下示例：

```js
p { color: #ffffff; }
article { color: #ffffff }
```

这里，以下声明与前面的声明相同

```js
p, article { color: #ffffff }
```

## 简单选择器

以下选择器都是 CSS 的简单选择器。

## 类型选择器

`type`选择器根据元素名称进行选择：

```js
E
ns|E
```

在这里，`E`是元素名称，`ns`是命名空间。

### 描述

这是选择元素的最简单方式——使用它们的名称。在大多数情况下，当仅使用 HTML 时，你不需要担心命名空间，因为所有 HTML 元素都在默认命名空间中。可以使用星号来指定所有命名空间，例如，`*|Element`。

当使用此选择器时，它将匹配文档中的所有元素。例如，如果你有十五个`h2`元素，并使用单个`h2`元素，那么这个规则将匹配所有十五个。

这里有一些`type`选择器的示例。第一段代码将所有`h1`元素的字体颜色设置为红色。接下来的代码将红色作为所有`p`元素的背景颜色：

```js
h1 { color: #ff0000; }
p { background: #ff0000; }
```

## 通用选择器

星号（`*`）代表任何和所有合格元素：

```js
*
ns|*
*|*
```

在这里，`ns` 是一个命名空间。

### 描述

这实际上是一个通配符选择器。它将匹配每个元素。即使与其他选择器一起使用，这也是 `true`。例如，`*.my-class` 和 `.my-class` 是相同的。

虽然你可以将其用作单个选择器来匹配每个元素，但它在与其他选择器一起使用时最有用。按照我们前面的例子，我们可能想要选择任何是 `article` 元素后代的元素。这个选择器非常明确且易于阅读，请看以下语法：

```js
article *
```

这里有一个例子。第一个例子使用属性选择器选择任何具有 `hreflang` 的元素，并且是英文的，第二个例子将选择文档中的所有元素：

```js
*[hreflang="en"] { display: block; background:url(flag_of_the_UK); }
* { padding: 0; }
```

## 属性选择器

这些选择器将匹配元素的属性。有七种不同的属性选择器类型，如下所示：

```js
[attribute]
[attribute=value]
[attribute~=value]
[attribute|=value]
[attribute^=value]
[attribute$=value]
[attribute*=value]
```

这些选择器通常由类型选择器或通用选择器 precede。

### 描述

这个选择器是一种在选择器规则中使用正则表达式语法的方法。每个选择器根据其使用方式会有不同的行为，因此它们会根据差异在此列出：

+   `[attribute]`: 这匹配具有 `[attribute]` 属性的元素，无论属性的值如何。

+   `[=]`: 值必须与精确匹配。

+   `[~=]`: 当属性接受值列表时使用。列表中的某个值必须匹配。

+   `[|=]`: 这个属性必须要么是精确匹配，要么值必须以紧跟其后的值开始，后面跟着一个 `-`。

+   `[^=]`: 这个属性匹配具有此前缀的值。

+   `[$=]`: 这个属性匹配具有此后缀的值。

+   `[*=]`: 这个属性匹配值的任何子串。

真正展示这些选择器之间差异的最好方法是使用一些例子。我们将查看 `lang` 和 `href` 属性。例子将按照它们被引入的顺序排列。

这里是选择器将要选择的 HTML 文件。

```js
<span lang="en-us en-gb">Attribute Selector</span>
<span lang="es">selector de atributo</span>
<span lang="de-at de">German (Austria)</span>
<a href="https://example.com">HTTPS</a>
<a href="http://google.com">Google</a>
<a href="http://example.com/example.pdf">Example PDF</a>
```

使用以下方法，我们应该有所有具有 `lang` 属性的 span 元素具有黑色背景，西班牙语将是灰色，德语将是红色，英语将是蓝色，具有 `https` 属性的锚点元素将是黄色，任何 PDF 将是红色，任何指向 Google 的锚点将是绿色。以下是前面描述的样式：

```js
span[lang] { background-color: #000000; color: #ffffff; }
span[lang="es"] { color: #808080; }
span[lang~="de-at"] { color: #ff0000; }
span[lang|="en"] { color: #0000ff; }
a[href^="https"] { color: #ffff00; }
a[href$="pdf"] { color: #ff0000; }
a[href*="google"] { color: #00ff00; }
```

## 类选择器

这个选择器将根据元素的类属性匹配 HTML 元素。

```js
element.class
.class or *.class
element.class.another-class
```

### 描述

这是最常用的选择器。基于 `class` 属性的选择允许你以正交的方式对元素进行样式化。类可以应用于许多不同的元素，并且我们可以以相同的方式为每个元素设置样式。

`class` 选择器可以堆叠，以便两个类都必须存在。

这里有一些包含不同元素的 HTML 代码，这些元素具有类属性：

```js
<h1 class="red">Red Text</h1>
<span class="red">Red Text</span>
<span class="green black">Green text, black background</span>
<span class="green">Normal text</span>
```

这里是用于样式化 HTML 的 CSS：

```js
.red { color: #ff0000; }
.green.black { color: #00ff00; background-color: #000000; }
```

当将`red`类应用于一个元素时，它将改变文本颜色为红色。复合的绿色和黑色将只选择定义了这两个类的元素。

## ID 选择器

这个选择器将基于元素的`ID`属性进行匹配：

```js
#id
element#id or *#id
```

### 描述

`ID`属性应该对文档是唯一的，所以`ID`选择器应该只针对一个元素。这与`class`选择器形成对比，`class`选择器可以用来选择多个元素。例如，你可以使用`class`选择器来使页面上的每个图像都有一定量的边距，并有一个专门针对你的标志的规则，使其有不同的边距。

这里是一个针对标志 ID 的 CSS 规则示例：

```js
#logo { margin: 10px; }
```

# 组合器

组合器用于选择更复杂的结构。它们可以帮助选择其他方式难以选择的具体元素或元素组。

## 后代组合器

这个选择器指定一个元素必须被另一个元素包含。

组合器是空白字符。我们在这里明确定义它，以便清晰：

```js
selector-a selector-b
```

### 描述

在这个选择器中使用的两个或多个语句可以是任何有效的选择器语句。例如，第一个可以是`class`选择器后面跟着`type`选择器。选择器之间的距离无关紧要。每个中间元素不需要列出，选择器就可以工作。

组合器可以堆叠。每个语句周围都有一个空白字符。这个选择器列表不需要包含所有内容，但为了选择器匹配层次结构，它确实需要存在。

这个选择器最好用于你只想在特定情况下样式化元素的情况。以下示例说明了这一点。

在这个第一个例子中，我们将针对具有“presidents”ID 的有序列表中的图像，并给它们添加一个红色边框。这是它的 HTML 代码：

```js
<ol id="presidents">
  <li><img src="img/pres01.png" alt="PortraitProtrait of George Washington" />George Washington</li>
  <li><img src="img/pres02.png" alt="PortraitProtrait of John Adams" />John Adams</li>
</ol>
<img src="img/not_pres.png" alt="not a President - no border" />
```

这里是 CSS 规则：

```js
ol#presidents img { border: 1px solid #ff0000; }
```

这里是一个例子，演示了选择器之间可以存在许多元素。这是非常随意的 HTML。

```js
<div class="example">
  I am normal.
  <div>
    <div class="select-me">
      I am red.
      <span class="select-me">I am red as well.</span>
    </div>
  </div>
</div>
```

这里是 CSS 规则：

```js
.example .select-me { color: #ff0000; }
```

最后，这里是一个多重选择器层次结构的例子，其 HTML 如下：

```js
<div class="first">Not the target
  <div class="second">Not the target
    <div class="third">I am the target.</div>
  </div>
</div>
```

CSS 规则：

```js
.first .second .third { color: #ff0000; }
```

## 子组合器

这个选择器针对特定的子元素：

```js
element-a > element-b
```

### 描述

这与后代组合器非常相似，只是这个只针对子关系。第二个选择器必须是第一个直接包含在父元素中的直接后代。

这里是一个例子，它将只针对这个 HTML 中的第一个`span`：

```js
<div>Here is an <span>arbitrary</span> <p><span>structure.</span></p></div>
```

这里是只设置第一个`span`颜色的 CSS 规则：

```js
div > span { color: #ff0000; }
```

## 邻接兄弟组合器

这个选择器针对在层次结构中相邻的元素：

```js
element-a + element-b
```

### 描述

这两个元素必须具有相同的父元素，第一个元素必须紧接第二个元素之后。

这里有一个示例，突出显示了选择器的工作方式。只有第二个 span 将应用此规则。最后一个 span 的前一个兄弟不是 span，因此它不会被选择器匹配。以下是 HTML 代码。

```js
<p>Here are a few spans in a row: <span>1</span> <span>2</span> <em>3</em> <span>4</span></p>
```

**CSS**:

```js
span + span { color: #ff0000; }
```

## 一般兄弟组合器

这个选择器针对任何与第一个元素有相同父元素并跟随它的元素。

```js
element-a ~ element-b
```

### 描述

这与相邻兄弟组合器类似；不同之处在于，两个元素可以由其他元素分隔。

这里有一个示例，说明尽管第二个和第三个 span 之间有一个 `em` 元素，但第二个和第三个 span 都将被目标化。以下是 HTML 代码。

```js
<p>Here are a few spans in a row: <span>1</span> <span>2</span> <em>3</em> <span>4</span></p>
```

这里是 CSS 规则。

```js
span ~ span { color: #ff0000; }
```

## 选择器特定性

这不是本节中其他选择器规则之一。一个元素可以被多个规则目标化，那么如何知道哪个规则具有优先级？这就是特定性发挥作用的地方。你可以计算出哪个规则将被应用。以下是计算方法。请注意，内联样式将覆盖任何选择器特定性：

+   选择器中的 `ID` 选择器数量用 `a` 表示

+   选择器中的类选择器、属性选择器和伪类的数量用 `b` 表示

+   选择器中的 `类型` 选择器和伪元素的数量用 `c` 表示

+   任何 `通用` 选择器都被忽略

这些数字然后被连接起来。数值越大，规则优先级越高。让我们看看一些选择器示例。示例将由选择器和计算值组成：

+   `h1`: `a=0` `b=0` `c=1`, `001` 或 `1`

+   `h1 span`: `a=0` `b=0` `c=2`, `002` 或 `2`

+   `h1 p > span`: `a=0` `b=0` `c=3`, `003` 或 `3`

+   `h1 *[lang="en"]`: `a=0` `b=1` `c=1`, `011` 或 `11`

+   `h1 p span.green`: `a=0` `b=1` `c=3`, `013` 或 `13`

+   `h1 p.example span.green`: `a=0` `b=2` `c=3`, `023` 或 `23`

+   `#title`: `a=1` `b=0` `c=0`, `100`

+   `h1#title`: `a=1` `b=0` `c=1`, `101`

考虑这个问题的最简单方法是认为每个分组（`a`、`b` 或 `c`）应该是一个更小的元素组，从中选择。这意味着每一步都有更多的权重。例如，一个页面上可以有多个 `h1` 实例。所以，仅仅选择 `h1` 就有点模糊。接下来，我们可以添加一个 `class`、`attribute` 或伪类选择器。这应该是 `h1` 实例的子集。然后，我们可以通过 ID 搜索。这具有最高的权重，因为整个文档中应该只有一个。

这里是一个包含三个标题的示例 HTML。

```js
<h1>First Heading</h1>
<h1 class="headings"><span>Second Heading</span></h1>
<h1 class="headings" id="third">Third Heading</h1>
```

这里是针对每个标题不同目标的 CSS 规则。第一个规则针对所有元素，但它的特定性最低，下一个规则位于中间，最后一个规则只针对一个元素。在下面的示例中，`/* */` 表示注释文本：

```js
h1 { color: #ff0000; } /* 001 */
h1.headings span { color: #00ff00; } /* 012 */
h1#third { color: #0000ff; } /* 101 */
```

# 伪类

伪类是使用文档树之外的信息的选择器。这些信息不在元素的属性中。这些信息可能在访问之间或甚至在访问期间改变。伪类总是有一个冒号后跟伪类的名称。

## 链接伪类

有两个互斥的链接伪类，即 `:link` 和 `:visited`。

### :link

这选择未访问的链接。语法如下：

```js
:link
```

#### 描述

这个伪类存在于任何未访问的锚点元素上。浏览器可能会在一段时间后决定将链接切换回来。

这里有一个与 `:visited` 伪类一起的例子。这是它的 HTML：

```js
<a href="#test">Probably not visited</a>
<a href="https://www.google.com">Probably visited</a>
```

这里是 CSS。我们可以假设你已经访问过 Google，所以链接可能呈绿色：

```js
a:link { color: #ff0000; }
a:visited { color: #00ff00; }
```

### :visited

这选择已访问的链接。语法如下：

```js
:visited
```

#### 描述

这个伪类存在于任何已访问的锚点元素上。

这里有一个与 `:link` 伪类一起的例子。这是它的 HTML：

```js
<a href="#test">Probably not visited</a>
<a href="https://www.google.com">Probably visited</a>
```

这里是 CSS。我们可以假设你已经访问过 Google，因此第一个链接应该是红色，第二个链接将是绿色：

```js
a:link { color: #ff0000; }
a:visited { color: #00ff00; }
```

## 用户操作伪类

这些类基于用户的操作生效。这些选择器不是互斥的，一个元素可以同时匹配多个。

### :active

这用于元素被激活时：

```js
:active
```

#### 描述

`:active` 选择器在鼠标按钮按下但未释放时最常使用。这种样式可以被其他用户操作或链接伪类所取代。

这里有一个例子：

```js
<a href="https://www.google.com">Google</a>
```

当你点击链接时，链接会变成绿色。这是它的 CSS：

```js
a:active { color: #00ff00; }
```

### :focus

这个选择器针对需要聚焦的元素。语法如下：

```js
:focus
```

#### 描述

当元素接受键盘输入时，它被认为是具有焦点。例如，一个文本输入元素，你可能已经通过制表符切换到它或点击在其中。

这里是一个文本输入示例：

```js
<input type="text" value="Red when focused">
```

这里是 CSS。这也强调了你可以使用伪类，这允许使用更复杂的选择器：

```js
input[type="text"]:focus { color: #ff0000; }
```

### :hover

这个选择器在用户将鼠标悬停在元素上时针对元素：

```js
:hover
```

#### 描述

这用于用户将光标悬停在元素上时。一些浏览器（一个很好的例子是移动触摸设备，如手机）可能不会实现这个伪类，因为没有方法可以确定用户是否悬停在元素上。

这里有一个例子：

```js
<span>Hover over me!</span>
```

当鼠标悬停在 span 中的文本上时，文本颜色会变成红色。这是它的 CSS：

```js
span:hover { color: #ff0000; }
```

## 结构选择器

这些选择器允许你根据文档树选择元素；这使用其他选择器很难或不可能做到。这仅选择节点是元素，不包括不在元素内的文本。编号是 `1-` 基础索引。

## :first-child

这针对另一个元素的第一个子元素：

```js
:first-child
```

### 描述

这与 `:nth-child(1)` 相同。这个选择器很简单，将选择应用到的元素类型的第一个子元素。

这里有一个例子，它只会选择第一个段落元素。以下是 HTML 代码：

```js
<p>First paragraph.</p>
<p>Second paragraph.</p>
```

这里是 CSS 代码。这将改变第一段文本的颜色为红色：

```js
p:first-child { color: #ff0000; }
```

## :first-of-type

这针对的是另一个元素的子元素中的第一个元素类型：

```js
:first-of-type
```

### 描述

`:first-of-type` 属性与 `:first-child` 不同，因为它只有在它是第一个子元素时才会选择该元素。这等同于 `:nth-of-type(1)`。

这里有一个例子，即使它不是第一个子元素，也会选择第一个段落元素。以下是 HTML 代码：

```js
<article>
  <h1>Title of Article</h1>
  <p>First paragraph.</p>
  <p>Second paragraph.</p>
</article>
```

这里是 CSS 代码：

```js
p:first-of-type { color: #ff0000; }
```

## :last-child

这针对的是另一个元素的最后一个子元素：

```js
:last-child
```

### 描述

这与 `:nth-last-child(1)` 相同。这个选择器很简单，将选择应用到的元素类型的最后一个子元素。

这里有一个例子，它只会选择最后一个段落元素。以下是 HTML 代码：

```js
<p>First paragraph.</p>
<p>Second paragraph.</p>
```

这里是 CSS 代码。这将改变第二段和第一段的颜色为红色。这个选择器之所以有效，是因为即使在最基础的页面上，`p` 元素也是 `body` 元素的子元素：

```js
p:last-child { color: #ff0000; }
```

## :last-of-type

这针对的是另一个元素的子元素中的最后一个元素类型：

```js
:last-of-type
```

### 描述

`:last-of-type` 属性与 `:last-child` 不同，因为它只有在它是第一个 `last-child` 属性时才会选择该元素。这等同于 `:nth-last-of-type(1)`。

这里有一个例子，它将选择最后一个段落元素。以下是它的 HTML 代码：

```js
<article>
  <p>First paragraph.</p>
  <p>Second paragraph.</p>
  <a href="#">A link</a>
</article>
```

这里是 CSS 代码：

```js
p:last-of-type { color: #ff0000; }
```

## :nth-child()

这将把所有子元素分割开，并根据它们存在的位置来选择它们：

```js
:nth-child(an+b)
```

### 描述

这个选择器有一个参数，它在选择方面非常表达。这也意味着它比大多数其他 CSS 规则更复杂。以下是该参数的技术规范。这个选择器基于其前面的元素来选择元素。

参数可以分为两部分：部分 `a` 和部分 `b`。部分 `a` 是一个整数，后面跟着字符 `n`。部分 `b` 有一个可选的加号或减号，后面跟着一个整数。参数还接受两个关键字：even 和 odd。以 `2n+1` 为例。

以这种方式查看时，更容易理解。第一部分，`an`，是子元素被分割的部分。`2n` 值将使元素成对分组，`3n` 值将使元素成三组，依此类推。下一部分 `+1` 将选择分组中的该元素。`2n+1` 将选择每个奇数行项，因为它针对的是每个两个元素分组中的第一个元素。`2n+0` 或 `2n+2` 将选择每个偶数行项。第一部分，部分 `a`，可以省略，然后它将只选择整个组中的第 `n` 个子元素。例如，`:nth-child(5)` 将只选择第五个子元素，而不选择其他元素。

表行是使用此选择器的绝佳示例，因此我们将针对每个奇数行。以下是 HTML：

```js
<table>
  <tr><td>First (will be red)</td></tr>
  <tr><td>Second</td></tr>
  <tr><td>Third (will be red)</td></tr>
</table>
```

这里是 CSS：

```js
tr:nth-child(2n+1) { color: #ff0000; }
```

## :nth-last-child

这将针对从末尾开始的第 n 个元素：

```js
:nth-last-child(an+b)
```

### 描述

此选择器计算后续元素的数量。计数逻辑与 `:nth-child` 相同。

这里有一个使用表格的示例。以下是 HTML：

```js
<table>
  <tr><td>First</td></tr>
  <tr><td>Second</td></tr>
  <tr><td>Third</td></tr>
  <tr><td>Fourth</td></tr>
</table>
```

第一条 CSS 规则将改变每行中每隔一行的颜色，但由于它从末尾开始计数，第一行和第三行将被选中。第二条 CSS 规则将仅针对最后一行：

```js
tr:nth-last-child(even) { color: #ff0000; }
tr:nth-last-child(-n+1) { color: #ff0000; }
```

### 参见

上一节 `:nth-child`。

## :nth-last-of-type 和 :nth-of-type

这根据元素的类型和它们在文档中的位置选择元素：

```js
:nth-last-of-type(an+b)
:nth-of-type(an+b)
```

### 描述

像所有其他 *n*th 选择器一样，这个选择器使用与 `:nth-child` 相同的逻辑。不同之处在于 nth-of-type 是 `:nth-last-of-type` 只按相同类型的元素分组。

这里有一个使用段落和跨度示例：

```js
<p>First</p>
<span>First Span</span>
<span>Second Span</span>
<p>Second</p>
<p>Third</p>
```

这里是 CSS。此规则将仅针对段落，并将奇数行变为红色：

```js
p:nth-of-type(2n+1) { color: #ff0000; }
```

### 参见

上一节 `:nth-child`。

## :only-child

这针对没有兄弟元素的元素：

```js
:only-child
```

### 描述

当 `:only-child` 属性是元素的唯一子元素时，这将匹配。

这里有两个表格的示例，其中一个有多个行，另一个只有一个：

```js
<table>
  <tr><td>First</td></tr>
  <tr><td>Second</td></tr>
  <tr><td>Third</td></tr>
</table>
<table>
  <tr><td>Only</td></tr>
</table>
```

这里是针对第二个表格中唯一行的 CSS：

```js
tr:only-child { color: #ff0000; }
```

## :only-of-type

这针对只有一个此类元素的情况：

```js
:only-of-type
```

### 描述

这将仅在父元素下没有其他相同类型的兄弟元素时匹配。

这里有一个使用任意分隔来创建一个结构的示例，其中一个段落元素是其类型的唯一元素。以下是 HTML：

```js
<div>
  <p>Only p.</p>
  <div>
    <p>Not the </p>
    <p>only one.</p>
  </div>
</div>
```

这里是仅针对第一段元素的 CSS 规则：

```js
p:only-of-type { color: #ff0000; }
```

# 验证

这些是伪类，可以用来针对输入元素的状态等。

## :checked

此属性针对已选中的单选按钮或复选框：

```js
:checked
```

### 描述

任何可以打开或关闭的元素都可以使用此选择器。到目前为止，这些是单选按钮、复选框和选择列表中的选项。

这里有一个包含 `checkbox` 和 `label` 值的示例：

```js
<input type="checkbox" checked value="check-value" name="test" />
<label for="test">Check me out</label>
```

这里是一个 CSS 规则，它将仅在复选框被选中时针对标签：

```js
input:checked + label { color: #ff0000; }
```

## :default

这针对多个类似元素中的默认元素：

```js
:default
```

### 描述

使用此选择器帮助定义一组元素中的默认元素。在表单中，这将默认按钮或从 `select` 元素中最初选中的选项。

这里有一个使用表单的示例：

```js
<form method="post">
  <input type="submit" value="Submit" />
  <input type="reset" value="Reset" />
</form>
```

这里是 CSS。这将仅针对提交输入，因为它默认：

```js
:default { color: #ff0000; }
```

## :disabled 和 :enabled

这些将根据它们的启用状态选择元素：

```js
:disabled
:enabled
```

### 描述

在交互式元素上有一个可用的禁用属性。使用 `:disabled` 将针对具有 `:disabled` 属性的元素，而 `:enabled` 将执行相反的操作。

这里有一些包含两个输入的 HTML，其中一个被禁用：

```js
<input type="submit" value="Submit" disabled/>
<input type="reset" value="Reset" />
```

这里是 CSS。禁用的输入项将文本颜色设置为红色，而其他输入项为绿色：

```js
input:disabled { color: #ff0000; }
input:enabled { color: #00ff00; }
```

## `:empty`

这针对没有子节点的元素：

```js
:empty
```

### 描述

这针对没有子节点的节点。子节点可以是任何其他元素，包括文本节点。这意味着即使是一个空格也会算作一个子节点。然而，注释不会算作子节点。

这里是一个有三个`div`标签的示例。第一个是空的，下一个有文本，最后一个有一个空格。这是 HTML：

```js
<div></div>
<div>Not Empty</div>
<div> </div>
```

这里是 CSS。只有第一个`div`将具有红色背景：

```js
div { height: 100px; width: 100px; background-color: #00ff00; }
div:empty { background-color: #ff0000; }
```

## `:in-range` 和 `:out-of-range`

这些选择器定位具有范围限制的元素：

```js
:in-range
:out-of-range
```

### 描述

一些元素现在有可以应用的范围限制。当值超出这个范围时，`:out-of-range`选择器将定位它，而当值在这个范围内时，`:in-range`将定位它。

这里是一个使用数字类型输入的示例：

```js
<input type="number" min="1" max="10" value="11" />
<input type="number" min="1" max="10" value="5" />
```

这里是 CSS。第一个输入项将具有红色文本，因为它超出了最大范围，而第二个输入项将具有绿色文本：

```js
:in-range {color: #00ff00; }
:out-of-range { color: #ff0000; }
```

## `:invalid` 和 `:valid`

`:invalid` 和 `:valid` 属性根据数据的有效性来定位元素：

```js
:invalid
:valid
```

### 描述

某些输入元素具有数据有效性，一个很好的例子是电子邮件元素。选择器根据数据是否有效来选择。你应该注意，一些元素始终是有效的，例如文本输入，而一些元素永远不会被这些选择器定位，例如`div`标签。

这里是一个电子邮件输入的示例：

```js
<input type="email" value="test@test.com" />
<input type="email" value="not a valid email" />
```

这里是 CSS。第一个输入项将是绿色，因为它有效，而其他输入项将是红色：

```js
:valid {color: #00ff00; }
:invalid { color: #ff0000; }
```

## `:not` 或否定

`:not` 属性否定一个选择器：

```js
:not(selector)
```

### 描述

`:not` 参数必须是一个简单选择器，并将定位到`:not`参数不是`true`的元素。这个选择器不会增加规则的特定性。

这里是一个使用段落的示例：

```js
<p>Targeted Element</p>
<p class="not-me">Non targeted element</p>
```

这里是 CSS。只有第一个段落将被定位：

```js
p:not(.not-me) {color: #ff0000; }
```

## `:optional` 和 `:required`

`:optional` 和 `:required` 属性分别定位可选或必需的元素。

```js
:optional
:required
```

### 描述

这用于任何必需或可选的输入元素。

这里是一个有两个输入的示例——一个是必需的，另一个不是：

```js
<input type="text" value="Required" required />
<input type="text" value="Optional" />
```

这里是 CSS。必需的输入项将具有红色文本，而可选的输入项将具有绿色文本：

```js
:required { color: #ff0000; }
:optional { color: #00ff00; }
```

## `:lang()`

`:lang()` 属性根据语言来定位：

```js
:lang(language)
```

### 描述

这个选择器与属性选择器的工作方式不同；即，这将定位所有在特定语言中的元素，即使它们没有明确定义。属性选择器只会定位具有`lang`属性的元素。

这里是一个`span`元素的示例，它没有`lang`属性，但它是具有`lang`属性的 body 元素的子元素：

```js
<body lang="en-us">
<span>This is English.</span>
</body>
```

这里是 CSS。第一个规则将匹配元素，但第二个规则不会匹配任何内容：

```js
:lang(en) { color: #ff0000; }
span[lang|=en] { color: #00ff00; }
```

# 伪元素

这些是超出文档中指定内容的选择器。选择器选择的东西可能是文档中不存在的元素。伪元素不被视为简单选择器的一部分。这意味着您不能将伪元素用作`:not()`选择器的一部分。最后，每个选择器只能有一个伪元素。

### 注意

注意，所有伪元素都以双冒号（`::`）开头。这是在 CSS3 中引入的，以帮助区分具有单个冒号（`:`）的伪类。这很重要，因为在 CSS2 中，伪元素只有单个冒号。在大多数情况下，您应该使用双冒号。

## ::before 和 ::after

这些用于在所选元素之前或之后插入生成内容：

```js
::before
::after e
```

### 描述

这将根据选择器将内容插入文档。内容是放置在目标元素之前还是之后取决于所使用的伪元素。请参阅*生成内容*部分，了解您可以插入的内容。

这里是一个使用`::before`和`::after`的例子。这将创建一个火鸡三明治。以下是 HTML 代码。

```js
<p class="sandwich">Turkey</p>
```

这里是 CSS，它将在火鸡前后放置一片面包：

```js
p.sandwich::before, p.sandwich::after 
{ content: ":Slice of Bread:"; }
```

### 相关内容

*生成内容*

## ::first-letter

这针对元素的第一个字母：

```js
::first-letter
```

### 描述

这将选择元素中的第一个字母，前提是它前面没有任何其他内容，例如一个`img`元素在字符之前会使`::first-letter`不选择第一个字符。任何在第一个字母之前或之后出现的标点符号都将包含在第一个字母中。这将选择任何字符，包括不仅仅是字母，还包括数字。

这仅适用于类似块容器，如块、`list-item`、`table-cell`、`table-caption`和`inline-block`元素。

### 注意

`::first-letter`伪元素只有在第一个字母位于第一格式行上时才会匹配。如果第一个字母出现之前有换行符，则不会选择它。

这里是一个例子，它不会选择第一个字母：

```js
<p><br />First letter</p>
```

这里是 CSS：

```js
p::first-letter { font-size: 2em; }
```

这里是一个例子：

```js
<p>This is a long line of text that may or may not be broken up across lines.</p>
```

这里是 CSS。在"This"中的 T 将是所有其他字符字体大小的两倍：

```js
p::first-letter { font-size: 2em; }
```

## ::first-line

`::first-line`属性针对元素的第一行：

```js
::first-line
```

### 描述

这将针对类似块容器（如`block box`、`inline-block`、`table-caption`或`table-cell`）的第一格式行。

这里是一个例子，以下面的 HTML 为例：

```js
<p>This is a long line of text that may or may not be broken up across lines based on the width of the page and the width of the element it is in.</p>
<p>This is the entire first line.</p>
```

这里是 CSS。这将使第一行（无论是什么）显示为红色：

```js
p::first-line { color: #ff0000; }
```

## ::selection

这针对用户突出显示的文本：

```js
::selection
```

### 描述

这个伪元素允许您对用户突出显示的任何文本进行样式设置。这个伪元素在 CSS3 中不存在，但它将是下一个版本的一部分。大多数浏览器仍然会尊重这个伪元素。

这里是一个例子：

```js
<p>Highlight this to text.</p>
```

这里是 CSS。当文本被选中时，文本将在红色背景上显示为白色：

```js
::selection { color: #ffffff; background: #ff0000; }
```

# 生成内容

这不是一个选择器，但与伪元素 `::before` 和 `::after` 一起使用。你可以生成的内容类型有限。以下是概述。

## 内容

这是将被放置在元素前后的内容：

```js
content(none, <string>, <uri>, <counter>, open-quote, close-quote, no-open-quote, no-close-quote, attr(x))
```

### 参数

以下是参数及其描述：

+   `none`: 此参数不会生成任何内容

+   `normal`: 这是默认参数，等同于无

+   `<string>`: 这是指任何字符串文本内容

+   `<uri>`: 这将映射到一个资源，例如，一个图像

+   `<counter>`: 这可以用来作为 `counter()` 或 `counters()` 函数，在元素前后放置计数器

+   `open-quote` 和 `close-quote`: 这与生成的引用内容属性一起使用

+   `no-open-quote` 和 `no-close-quote`: 这不会添加内容，但会递增或递减引用的嵌套层级

+   `attr(x)`: 这将返回目标元素的属性值

### 描述

此属性用于向文档添加内容。输出由使用的值控制。这些值可以组合起来创建更复杂的内容。

可以使用字符 `\A` 插入新行。只需记住，HTML 默认会忽略换行符。

这里有一些示例。这些示例将展示如何使用许多内容值：

```js
<h1>First</h1>
<h1>Second</h1>
<h1 class="test">Attribute</h1>
<h2>Line Break</h2>
<blockquote>Don't quote me on this.</blockquote>
```

这里是 CSS 代码。`h1` 元素将带有“`章节`”这个词，并在每个前面加上一个数字。`h2` 元素的内容将有一个换行符。最后，`blockquote` 将有一个开引号和一个闭引号：

```js
h1 { counter-increment: chapter; }
h1::before { content: "Chapter" counter(chapter) ": " attr(class) ; }
h2::before { content: "New\A Line"; white-space: pre; }
blockquote::before { content: open-quote; }
blockquote::after { content: close-quote; }
```

## 引号

引号指定了哪些字符用作开引号和闭引号：

```js
quotes: [<string> <string>]+
```

### 参数

`<string> <string>`: 这些是表示开引号和闭引号的字符对。你可以多次使用它来创建引用的层级。

### 描述

我们可以使用此属性来设置使用的引号类型。

这里有一个包含嵌套引用的示例：

```js
<blockquote>Don't quote me <blockquote>on</blockquote> this.</blockquote>
```

引号是完全任意的。以下是 CSS 代码：

```js
blockquote { quotes: ":" "!" "&" "*"; }
```
