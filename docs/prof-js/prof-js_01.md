# 第二章：*第一章*

# JavaScript，HTML 和 DOM

## 学习目标

在本章结束时，您将能够：

+   描述 HTML 文档对象模型（DOM）

+   使用 Chrome DevTools 源选项卡来探索网页的 DOM

+   实现 JavaScript 来查询和操作 DOM

+   使用 Shadow DOM 构建自定义组件

在本章中，我们将学习 DOM 以及如何使用 JavaScript 与其交互和操作。我们还将学习如何使用可重用的自定义组件构建动态应用程序。

## 介绍

HTML 最初是用于静态文档的标记语言，易于使用，并且可以使用任何文本编辑器编写。在 JavaScript 成为互联网世界的主要角色之后，有必要将 HTML 文档暴露给 JavaScript 运行时。这就是创建 DOM 的时候。DOM 是将 HTML 映射到可以使用 JavaScript 查询和操作的对象树。

在本章中，您将学习 DOM 是什么以及如何使用 JavaScript 与其交互。您将学习如何在文档中查找元素和数据，如何操作元素状态以及如何修改其内容。您还将学习如何创建 DOM 元素并将其附加到页面上。

了解 DOM 及其如何操作后，您将使用一些示例数据构建动态应用程序。最后，您将学习如何创建自定义 HTML 元素以构建可重用组件，使用 Shadow DOM。

## HTML 和 DOM

当浏览器加载 HTML 页面时，它会创建代表该页面的树。这棵树基于 DOM 规范。它使用标记来确定每个节点的起始和结束位置。

考虑以下 HTML 代码片段：

```js
<html>
  <head>
    <title>Sample Page</title>
  </head>
  <body>
    <p>This is a paragraph.</p>
    <div>
      <p>This is a paragraph inside a div.</p>
    </div>
    <button>Click me!</button>
  </body>
</html>
```

浏览器将创建以下节点层次结构：

![图 1.1：段落节点包含文本节点](img/C14587_01_01.jpg)

###### 图 1.1：段落节点包含文本节点

一切都变成了节点。文本，元素和注释，一直到树的根部。这棵树用于匹配 CSS 样式并渲染页面。它还被转换为对象，并提供给 JavaScript 运行时使用。

但为什么它被称为 DOM 呢？因为 HTML 最初是设计用来共享文档，而不是设计我们今天拥有的丰富动态应用程序。这意味着每个 HTML DOM 都以一个文档元素开始，所有元素都附加到该元素上。考虑到这一点，前面的 DOM 树示意图实际上变成了以下内容：

![图 1.2：所有 DOM 树都有一个文档元素作为根](img/C14587_01_02.jpg)

###### 图 1.2：所有 DOM 树都有一个文档元素作为根

当我说浏览器使 DOM 可用于 JavaScript 运行时时，这意味着如果您在 HTML 页面中编写一些 JavaScript 代码，您可以访问该树并对其进行一些非常有趣的操作。例如，您可以轻松访问文档根元素并访问页面上的所有节点，这就是您将在下一个练习中要做的事情。

### 练习 1：在文档中迭代节点

在这个练习中，我们将编写 JavaScript 代码来查询 DOM 以查找按钮，并向其添加事件侦听器，以便在用户单击按钮时执行一些代码。事件发生时，我们将查询所有段落元素，计数并存储它们的内容，然后在最后显示一个警报。

此练习的代码文件可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson01/Exercise01`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson01/Exercise01)找到。

执行以下步骤完成练习：

1.  打开您喜欢的文本编辑器，并创建一个名为`alert_paragraphs.html`的新文件，其中包含上一节中的示例 HTML（可以在 GitHub 上找到：[`bit.ly/2maW0Sx`](https://bit.ly/2maW0Sx)）：

```js
<html>
  <head>
    <title>Sample Page</title>
  </head>
  <body>
    <p>This is a paragraph.</p>
    <div>
      <p>This is a paragraph inside a div.</p>
    </div>
    <button>Click me!</button>
  </body>
</html>
```

1.  在`body`元素的末尾，添加一个`script`标签，使最后几行看起来像下面这样：

```js
    </div>
    <button>Click me!</button>
    <script>
    </script>
  </body>
</html>
```

1.  在`script`标签内，为按钮的点击事件添加一个事件监听器。为此，你需要查询文档对象以找到所有带有`button`标签的元素，获取第一个（页面上只有一个按钮），然后调用`addEventListener`：

```js
document.getElementsByTagName('button')[0].addEventListener('click', () => {});
```

1.  在事件监听器内部，再次查询文档以查找所有段落元素：

```js
const allParagraphs = document.getElementsByTagName('p');
```

1.  之后，在事件监听器内创建两个变量，用于存储你找到的段落元素的数量和存储它们的内容：

```js
let allContent = "";
let count = 0;
```

1.  迭代所有段落元素，计数它们，并存储它们的内容：

```js
for (let i = 0; i < allParagraphs.length; i++) {  const node = allParagraphs[i];
  count++;
  allContent += `${count} - ${node.textContent}\n`;
}
```

1.  循环结束后，显示一个警报，其中包含找到的段落数和它们所有内容的列表：

```js
alert(`Found ${count} paragraphs. Their content:\n${allContent}`);
```

你可以在这里看到最终的代码应该是什么样子的：[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Exercise01/alert_paragraphs.html`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Exercise01/alert_paragraphs.html)。

在浏览器中打开 HTML 文档并点击按钮，你应该会看到以下警报：

![图 1.3：显示页面上段落信息的警报框](img/C14587_01_03.jpg)

###### 图 1.3：显示页面上段落信息的警报框

在这个练习中，我们编写了一些 JavaScript 代码，查询了特定元素的 DOM。我们收集了元素的内容，以在警报框中显示它们。

我们将在本章的后续部分探索其他查询 DOM 和迭代节点的方法。但是从这个练习中，你已经可以看到这是多么强大，并开始想象这开启了哪些可能性。例如，我经常使用它来计数或从互联网上的网页中提取我需要的数据。

## 开发者工具

现在我们了解了 HTML 源代码和 DOM 之间的关系，我们可以使用一个非常强大的工具来更详细地探索它：浏览器开发者工具。在本书中，我们将探索谷歌 Chrome 的**DevTools**，但你也可以在所有其他浏览器中轻松找到等效的工具。

我们要做的第一件事是探索我们在上一节中创建的页面。当你在谷歌 Chrome 中打开它时，你可以通过打开**Chrome**菜单来找到开发者工具。然后选择**更多工具**和**开发者工具**来打开开发者工具：

![图 1.4：在谷歌 Chrome 中访问开发者工具](img/C14587_01_04.jpg)

###### 图 1.4：在谷歌 Chrome 中访问开发者工具

**开发者工具**将在页面底部打开一个面板：

![图 1.5：谷歌 Chrome DevTools 打开时的面板](img/C14587_01_05.jpg)

###### 图 1.5：谷歌 Chrome DevTools 打开时的面板

你可以在顶部看到提供加载页面上发生的不同视角的各种选项卡。在本章中，我们将主要关注三个选项卡：

+   **元素** – 显示浏览器看到的 DOM 树。你可以检查浏览器如何查看你的 HTML，CSS 如何被应用，以及哪些选择器激活了每个样式。你还可以改变节点的状态，模拟特定状态，比如`hover`或`visited`：

![图 1.6：元素选项卡的视图](img/C14587_01_05.jpg)

###### 图 1.6：元素选项卡的视图

+   **控制台** – 在页面的上下文中提供对 JavaScript 运行时的访问。在加载页面后，可以使用控制台来测试简短的代码片段。它还可以用于打印重要的调试信息：

![图 1.7：控制台选项卡的视图](img/C14587_01_07.jpg)

###### 图 1.7：控制台选项卡的视图

+   **源** – 显示当前页面加载的所有源代码。这个视图可以用来设置断点和开始调试会话：

![](img/C14587_01_08.jpg)

###### 图 1.8：源选项卡的视图

选择**元素**选项卡，你会看到当前文档的 DOM 树：

![图 1.9：在 Chrome DevTools 中查看的元素选项卡中的 DOM 树](img/C14587_01_09.jpg)

###### 图 1.9：在 Chrome DevTools 中查看的元素选项卡中的 DOM 树

### 练习 2：从元素选项卡操作 DOM

为了感受到这个工具有多强大，我们将对*练习 1：遍历文档中的节点*中的页面进行一些更改。我们将在其中添加一个新段落并删除一个现有的段落。然后，我们将使用**样式**侧边栏来更改元素的一些样式。

执行以下步骤完成练习：

1.  首先，*右键单击*`body`元素，然后选择**编辑为 HTML**：![图 1.10：编辑 HTML 主体元素](img/C14587_01_10.jpg)

###### 图 1.10：编辑 HTML 主体元素

1.  这将把节点更改为一个可以输入的文本框。在第一个段落下面，添加另一个文本为**另一个段落**的段落。它应该看起来像下面这样：![图 1.11：在 HTML 主体中添加一个新段落](img/C14587_01_11.jpg)

###### 图 1.11：在 HTML 主体中添加一个新段落

1.  按下*Ctrl + Enter*（或 Mac 上的*Cmd + Enter*）保存您的更改。

1.  再次单击**点击我！**按钮，您会看到新段落及其内容现在显示在列表中：![图 1.12：显示所有段落内容的警报，包括添加到页面中的段落](img/C14587_01_12.jpg)

###### 图 1.12：显示所有段落内容的警报，包括添加到页面中的段落

1.  您还可以玩弄元素的样式，并在页面上实时看到变化。让我们将第一个段落的背景更改为黑色，颜色更改为白色。首先，通过单击 DOM 树上的它来选择它；它会变成蓝色以表示已选择：![图 1.13：在元素选项卡上选择 DOM 元素](img/C14587_01_13.jpg)

###### 图 1.13：在元素选项卡上选择 DOM 元素

1.  现在，在右侧，您会看到**样式**选项卡。它包含已应用于元素的样式和一个用于元素样式的空占位符。单击它，您将获得一个输入框。输入**background: black**，按下*Enter*，然后输入**color: white**，再次按下*Enter*。您会看到随着您的输入，元素会发生变化。最终，它将看起来像下面这样：![图 1.14：左侧的样式化段落和右侧的应用样式](img/C14587_01_14.jpg)

###### 图 1.14：左侧的样式化段落和右侧的应用样式

1.  您还可以通过单击**样式**选项卡右上角的**新规则按钮**来创建一个应用于页面的新 CSS 规则：![图 1.15：当您单击添加新规则时，它将基于所选元素（在本例中为段落）添加一个新规则](img/C14587_01_15.jpg)

###### 图 1.15：当您单击添加新规则时，它将基于所选元素（在本例中为段落）添加一个新规则

1.  让我们添加类似的规则来影响所有段落，输入**background: green**，按下*Enter*，输入**color: yellow**，然后按下*Enter*。现在除了第一个段落外，所有段落都将具有绿色背景和黄色文本。页面现在应该是这样的：

![图 1.16：向段落添加规则](img/C14587_01_16.jpg)

###### 图 1.16：向段落添加规则

在这个练习中，您改变了页面的 DOM，并实时看到了变化。您向页面添加了元素，更改了一个元素的样式，然后添加了一个新的 CSS 规则来影响更广泛的元素组。

像这样实时操作 DOM 对于您试图弄清布局并测试一些迭代或操作 DOM 元素的代码的情况非常有用。在我们的情况下，我们可以轻松测试如果我们向页面添加一个新段落元素会发生什么。

### 练习 3：从源选项卡调试代码

我们之前提到过，您可以从**源**选项卡调试代码。要做到这一点，您只需要设置一个断点，并确保代码通过该点。在这个练习中，我们将在调试我们的代码时探索**源**选项卡。

执行以下步骤完成练习：

1.  您需要做的第一件事是在“开发者工具”面板中选择“源”选项卡。然后，打开我们目前拥有的一个源文件。您可以通过在左侧面板中点击它来实现这一点：![图 1.17：源选项卡显示了如何找到您的源文件](img/C14587_01_17.jpg)

###### 图 1.17：源选项卡显示了如何找到您的源文件

1.  要在源代码中设置断点，您需要点击行号所在的边栏，在您想要设置断点的行处点击。在这个练习中，我们将在事件处理程序内的第一行设置一个断点。一个蓝色的箭头符号将出现在那一行上：![图 1.18：断点显示为源文件边栏上的箭头标记](img/C14587_01_18.jpg)

###### 图 1.18：断点显示为源文件边栏上的箭头标记

1.  点击页面上的“点击我！”按钮来触发代码执行。您会注意到发生了两件事情 - 浏览器窗口冻结了，并且有一条消息表明代码已经暂停了：![图 1.19：当浏览器遇到断点时，执行会暂停](img/C14587_01_19.jpg)

###### 图 1.19：当浏览器遇到断点时，执行会暂停

1.  此外，正在执行的代码行在“源”选项卡中得到了突出显示：![图 1.20：源代码中的执行暂停，突出显示将要执行的下一行](img/C14587_01_20.jpg)

###### 图 1.20：源代码中的执行暂停，突出显示将要执行的下一行

1.  在侧边栏中，注意当前执行的堆栈和当前作用域中的所有内容，无论是全局还是局部。这是右侧面板的视图，显示了有关运行代码的所有重要信息：![图 1.21：源选项卡右侧显示了当前暂停执行的执行上下文和堆栈跟踪](img/C14587_01_21.jpg)

###### 图 1.21：源选项卡右侧显示了当前暂停执行的执行上下文和堆栈跟踪

1.  顶部的工具栏可以用来控制代码执行。每个按钮的功能如下：

“播放”按钮结束暂停并正常继续执行。

“步过”按钮会执行当前行直到完成，并在下一行再次暂停。

点击“步入”按钮将执行当前行并步入任何函数调用，这意味着它将在被调用的函数内的第一行暂停。

“步出”按钮将执行所有必要的步骤以退出当前函数。

“步”按钮将执行下一个操作。如果是函数调用，它将步入。如果不是，它将继续执行下一行。

1.  按下“步过”按钮，直到执行到第 20 行：![图 1.22：突出显示的行显示了执行暂停以进行调试](img/C14587_01_22.jpg)

###### 图 1.22：突出显示的行显示了执行暂停以进行调试

1.  在右侧的“作用域”面板上，您会看到四个作用域：两个“块”作用域，然后一个“局部”作用域和一个“全局”作用域。作用域将根据您在代码中的位置而变化。在这种情况下，第一个“块”作用域仅包括`for`循环内的内容。第二个“块”作用域是整个循环的作用域，包括在`for`语句中定义的变量。“局部”是函数作用域，“全局”是浏览器作用域。这是您应该看到的：![图 1.23：作用域面板显示了当前执行上下文中不同作用域中的所有变量](img/C14587_01_23.jpg)

###### 图 1.23：作用域面板显示了当前执行上下文中不同作用域中的所有变量

1.  此时要注意的另一件有趣的事情是，如果你将鼠标悬停在当前页面中的 HTML 元素上，Chrome 会为你突出显示该元素：

![图 1.24：Chrome 在不同位置悬停时突出显示 DOM 元素](img/C14587_01_24.jpg)

###### 图 1.24：Chrome 在不同位置悬停时突出显示 DOM 元素

使用**源**选项卡调试代码是作为 Web 开发人员最重要的事情之一。了解浏览器如何看待你的代码，以及每行中变量的值是解决复杂应用程序中问题的最简单方法。

#### 注意

内联值：当你在**源**选项卡中调试时逐步执行代码时，你会注意到 Chrome 在每行的侧边添加了一些浅橙色的突出显示，显示了在该行中受影响的变量的当前值。

### 控制台选项卡

现在你知道如何在**元素**选项卡中遍历和操作 DOM 树，以及如何在**源**选项卡中探索和调试代码，让我们来探索一下**控制台**选项卡。

**控制台**选项卡可以帮助你调试问题，也可以探索和测试代码。为了了解它能做什么，我们将使用本书代码库中`Lesson01/sample_002`文件夹中的示例商店。

打开商店页面，你会看到这是一个食品产品的商店。它看起来是这样的：

![图 1.25：商店示例页面的屏幕截图](img/C14587_01_25.jpg)

###### 图 1.25：商店示例页面的屏幕截图

在底层，你可以看到 DOM 非常简单。它有一个`section`元素，其中包含所有的页面内容。里面有一个带有类项的`div`标签，代表产品列表，以及每个产品的一个带有类项的`div`。在**元素**选项卡中，你会看到这样的内容：

![图 1.26：商店页面的 DOM 树非常简单](img/C14587_01_26.jpg)

###### 图 1.26：商店页面的 DOM 树非常简单

回到**控制台**选项卡：你可以在这个 DOM 中运行一些查询来了解更多关于元素和内容的信息。让我们写一些代码来列出所有产品的价格。首先，我们需要找到 DOM 树中的价格在哪里。我们可以查看**元素**选项卡，但现在，我们将只使用**控制台**选项卡来学习更多。在**控制台**选项卡中运行以下代码将打印一个包含 21 个项目的`HTMLCollection`对象：

```js
document.getElementsByClassName('item')
```

让我们打开第一个，看看里面有什么：

```js
document.getElementsByClassName('item')[0]
```

现在你看到 Chrome 打印了一个 DOM 元素，如果你在上面悬停，你会看到它在屏幕上被突出显示。你也可以打开在**控制台**选项卡中显示的迷你 DOM 树，看看元素是什么样子的，就像在**元素**选项卡中一样：

![图 1.27：控制台选项印刷 DOM 中的元素](img/C14587_01_27.jpg)

###### 图 1.27：控制台选项印刷 DOM 中的元素

你可以看到价格在一个`span`标签内。要获取价格，你可以像查询根文档一样查询元素。

#### 注意：自动完成和之前的命令

在**控制台**选项卡中，你可以通过按下*Tab*来使用基于当前上下文的自动完成，并通过按上/下箭头键快速访问之前的命令。

运行以下代码来获取列表中第一个产品的价格：

```js
document.getElementsByClassName('item')[0]
  .getElementsByTagName('span')[0].textContent
```

产品的价格将显示在控制台中作为一个字符串：

![图 1.28：查询包含价格的 DOM 元素并获取其内容](img/C14587_01_28.jpg)

###### 图 1.28：查询包含价格的 DOM 元素并获取其内容

### 活动 1：从页面中提取数据

假设您正在编写一个需要来自 Fresh Products Store 的产品和价格的应用程序。商店没有提供 API，其产品和价格大约每周变化一次-不够频繁以证明自动化整个过程是合理的，但也不够慢以至于您可以手动执行一次。如果他们改变了网站的外观方式，您也不想麻烦太多。

您希望以一种简单生成和解析的方式为应用程序提供数据。最终，您得出结论，最简单的方法是生成一个 CSV，然后将其提供给您的应用程序。

在这个活动中，您将编写一些 JavaScript 代码，可以将其粘贴到商店页面的**控制台**选项卡中，并使用它从 DOM 中提取数据，将其打印为 CSV，以便您的应用程序消费。

#### 注意：在控制台选项卡中的长代码

在 Chrome 控制台中编写长代码时，我建议在文本编辑器中进行，然后在想要测试时粘贴它。控制台在编辑代码时并不糟糕，但在尝试修改长代码时很容易搞砸事情。

执行以下步骤：

1.  初始化一个变量来存储 CSV 的整个内容。

1.  查询 DOM 以找到表示每个产品的所有元素。

1.  遍历找到的每个元素。

1.  从`product`元素中，查询带有单位的价格。使用斜杠拆分字符串。

1.  再次，从`product`元素中查询名称。

1.  将所有信息附加到步骤 1 中初始化的变量中，用逗号分隔值。不要忘记为附加的每一行添加换行字符。

1.  使用`console.log`函数打印包含累积数据的变量。

1.  在打开商店页面的**控制台**选项卡中运行代码。

您应该在**控制台**选项卡中看到以下内容：

```js
name,price,unit
Apples,$3.99,lb
Avocados,$4.99,lb
Blueberry Muffin,$2.50,each
Butter,$1.39,lb
...
```

#### 注意

此活动的解决方案可在第 582 页找到。

在这个活动中，您可以使用**控制台**选项卡查询现有页面并从中提取数据。有时，从页面中提取数据非常复杂，而且爬取可能会变得非常脆弱。根据您需要从页面获取数据的频率，可能更容易在**控制台**选项卡中运行脚本，而不是编写一个完整的应用程序。

### 节点和元素

在之前的章节中，我们学习了 DOM 以及如何与其交互。我们看到浏览器中有一个全局文档对象，表示树的根。然后，我们观察了如何查询它以获取节点并访问其内容。

但在前几节探索 DOM 时，有一些对象名称、属性和函数是在没有介绍的情况下访问和调用的。在本节中，我们将深入研究这些内容，并学习如何找到每个对象中可用的属性和方法。

关于本节将讨论的内容，最好的文档位置是 Mozilla 开发者网络网页文档。您可以在[developer.mozilla.org](http://developer.mozilla.org)找到。他们对所有 JavaScript 和 DOM API 都有详细的文档。

节点是一切的起点。节点是表示 DOM 树中的接口。如前所述，树中的一切都是节点。所有节点都有一个`nodeType`属性，用于描述节点的类型。它是一个只读属性，其值是一个数字。节点接口对于每个可能的值都有一个常量。最常见的节点类型如下：

+   `Node.ELEMENT_NODE` - HTML 和 SVG 元素属于这种类型。在商店代码中，如果您从产品中获取`description`元素，您将看到它的`nodeType`属性是`1`，这意味着它是一个元素：

![图 1.29：描述元素节点类型为 Node.ELEMENT_NODE](img/C14587_01_29.jpg)

###### 图 1.29：描述元素节点类型为 Node.ELEMENT_NODE

这是我们从**元素**选项卡中获取的元素：

![图 1.30：在元素选项卡中查看的描述节点](img/C14587_01_30.jpg)

###### 图 1.30：在元素选项卡中查看的描述节点

+   `Node.TEXT_NODE` - 标签内的文本变成文本节点。如果您从`description`节点获取第一个子节点，您会发现它的类型是`TEXT_NODE`：

![图 1.31：标签内的文本变成文本节点](img/C14587_01_31.jpg)

###### 图 1.31：标签内的文本变成文本节点

这是在**元素**选项卡中查看的节点：

![图 1.32：在元素选项卡中选择的文本节点](img/C14587_01_32.jpg)

###### 图 1.32：在元素选项卡中选择的文本节点

+   `Node.DOCUMENT_NODE` - 每个 DOM 树的根是一个`document`节点：

![图 1.33：树的根始终是文档节点](img/C14587_01_33.jpg)

###### 图 1.33：树的根始终是文档节点

一个重要的事情要注意的是`html`节点不是根节点。当创建 DOM 时，`document`节点是根节点，它包含`html`节点。您可以通过获取`document`节点的第一个子节点来确认：

![图 1.34：html 节点是文档节点的第一个子节点](img/C14587_01_34.jpg)

###### 图 1.34：html 节点是文档节点的第一个子节点

`nodeName`是节点具有的另一个重要属性。在`element`节点中，`nodeName`将为您提供它们的 HTML 标签。其他节点类型将返回不同的内容。`document`节点将始终返回`#document`（如前图所示），而`Text`节点将始终返回`#text`。

对于`TEXT_NODE`、`CDATA_SECTION_NODE`和`COMMENT_NODE`等类似文本的节点，您可以使用`nodeValue`来获取它们所包含的文本。

但节点最有趣的地方在于你可以像遍历树一样遍历它们。它们有子节点和兄弟节点。让我们在下面的练习中稍微练习一下这些属性。

### 练习 4：遍历 DOM 树

在这个练习中，我们将遍历*图 1.1*中示例页面中的所有节点。我们将使用递归策略来迭代所有节点并打印整个树。

执行以下步骤以完成练习：

1.  第一步是打开文本编辑器并设置它以编写一些 JavaScript 代码。

1.  要使用递归策略，我们需要一个函数，该函数将被调用以处理树中的每个节点。该函数将接收两个参数：要打印的节点和节点在 DOM 树中的深度。以下是函数声明的样子：

```js
function printNodes(node, level) {
}
```

1.  函数内部的第一件事是开始标识将要打开此节点的消息。为此，我们将使用`nodeName`，对于`HTMLElements`，它将给出标签，对于其他类型的节点，它将给出一个合理的标识符：

```js
let message = `${"-".repeat(4 * level)}Node: ${node.nodeName}`;
```

1.  如果节点也有与之关联的`nodeValue`，比如`Text`和其他文本行节点，我们还将将其附加到消息中，然后将其打印到控制台：

```js
if (node.nodeValue) {
  message += `, content: '${node.nodeValue.trim()}'`;
}
console.log(message);
```

1.  之后，我们将获取当前节点的所有子节点。对于某些节点类型，`childNodes`属性将返回 null，因此我们将添加一个空数组的默认值，以使代码更简单：

```js
var children = node.childNodes || [];
```

1.  现在我们可以使用`for`循环来遍历数组。对于我们找到的每个子节点，我们将再次调用该函数，启动算法的递归性质：

```js
for (var i = 0; i < children.length; i++) {
  printNodes(children[i], level + 1);
}
```

1.  函数内部的最后一件事是打印具有子节点的节点的关闭消息：

```js
if (children.length > 0) {
  console.log(`${"-".repeat(4 * level)}End of:${node.nodeName}`);}
```

1.  现在我们可以通过调用该函数并将文档作为根节点传递，并在函数声明结束后立即将级别设置为零来启动递归：

```js
printNodes(document, 0);
```

最终的代码应该如下所示：[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Exercise04/open_close_tree_print.js`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Exercise04/open_close_tree_print.js)。

1.  在 Chrome 中打开示例 HTML。文件位于：[`bit.ly/2maW0Sx`](https://bit.ly/2mMje1K)。

1.  打开**开发者工具**面板，在**控制台**选项卡中粘贴 JavaScript 代码，然后运行。以下是您应该看到的输出：

图 1.35：遍历 DOM 并递归打印所有节点及其子节点

###### 图 1.35：遍历 DOM 并递归打印所有节点及其子节点

在这个练习中，您学会了如何使用递归来逐个节点地遍历整个 DOM 树。您还学会了如何检查节点的属性，因为在遍历整个树时，您会看到不是 HTML 的节点，比如文本和注释。

非常有趣的一点是浏览器还保留了您添加到 HTML 中的空格。以下截图将源代码与练习中打印的树进行了比较：

图 1.36：演示空格也成为 DOM 树中的节点

###### 图 1.36：演示空格也成为 DOM 树中的节点

您可以使用颜色代码查看映射：

+   红色标记了包含标题文本的文本节点。

+   绿色标记了整个`title`元素。

+   蓝色框和箭头标记了`title`元素之前和之后的空格。

#### 注意：注意间隔

在处理 DOM 节点时，非常重要的一点是要记住并非所有节点都是 HTML 元素。有些甚至可能是您没有故意放入文档中的东西，比如换行符。

我们谈论了很多关于节点的内容。您可以查看 Mozilla 开发者网络文档以了解其他节点属性和方法。但您会注意到节点接口主要关注 DOM 树中节点之间的关系，比如兄弟节点和子节点。它们非常抽象。因此，让我们更具体一些，探索`Element`类型的节点。

所有 HTML 元素都被转换为`HTMLElement`节点，它们继承自`Element`，后者又继承自一个节点。它们继承了父类型的所有属性和方法。这意味着元素是一个节点，而`HTMLElement`实例是一个元素。

因为`element`代表一个元素（带有其所有属性和内部标签的标签），所以您可以访问其属性。例如，在`image`元素中，您可以读取`src`属性。以下是获取商店页面第一个`img`元素的`src`属性的示例：

图 1.37：获取页面第一个图像的 src 属性

###### 图 1.37：获取页面第一个图像的 src 属性

HTML 元素还具有的另一个有用属性是`innerHTML`属性。使用它，您可以获取（和设置）元素的 HTML。以下是获取具有`image`类的第一个`div`并打印其`innerHTML`的示例：

图 1.38：innerHTML 可用于访问元素内部的 HTML

###### 图 1.38：innerHTML 可用于访问元素内部的 HTML

还有`outerHTML`属性，它将给出元素本身的 HTML，包括其中的所有内容：

图 1.39：outerHTML 给出了元素及其内部的 HTML

###### 图 1.39：outerHTML 给出了元素及其内部的 HTML

最后但同样重要的是`className`属性，它可以让您访问应用于元素的类：

图 1.40：className 可以访问元素的类

###### 图 1.40：className 可以访问元素的类

关于这些属性更重要的是它们是可读/可写的，这意味着您可以使用它们来修改 DOM，添加类并更改元素的内容。在接下来的部分中，我们将使用这里所学到的内容来创建根据用户交互而变化的动态页面。

### 特殊对象

到目前为止，我们在许多示例和练习中都访问了`document`对象。但它到底是什么，还能做什么？文档是一个代表浏览器中加载的页面的全局对象。正如我们所见，它作为 DOM 树中元素的入口点。

它还有一个我们到目前为止还没有讨论的重要作用，那就是在页面中创建新节点和元素的能力。这些元素可以附加到树的不同位置，以在页面加载后修改它。我们将在接下来的章节中探讨这种能力。

除了`document`，还有另一个对象是 DOM 规范的一部分，那就是`window`对象。`window`对象是一个全局对象，也是所有在浏览器中运行的 JavaScript 代码的绑定目标。这意味着该变量是指向`window`对象的指针：

![图 1.41：浏览器中的全局范围和默认绑定目标是窗口对象](img/C14587_01_41.jpg)

###### 图 1.41：浏览器中的全局范围和默认绑定目标是窗口对象

`window`对象包含您需要从浏览器访问的所有内容：位置、导航历史、其他窗口（弹出窗口）、本地存储等等。`document`和`console`对象也归属于`window`对象。当您访问`document`对象时，实际上是在使用`window.document`对象，但绑定是隐式的，因此您不需要一直写`window`。而且因为`window`是一个全局对象，这意味着它必须包含对自身的引用：

![图 1.42：窗口对象包含对自身的引用](img/C14587_01_42.jpg)

###### 图 1.42：窗口对象包含对自身的引用

### 使用 JavaScript 查询 DOM

我们一直在讨论通过`document`对象查询 DOM。但是我们用来查询 DOM 的所有方法也可以从 DOM 中的元素中调用。本节介绍的方法也可以从 DOM 中的元素中调用。我们还将看到一些只能从元素中而不是`document`对象中使用的方法。

从元素中查询非常方便，因为查询的范围仅限于执行查询的位置。正如我们在*Activity 1, Extracting Data from the DOM*中看到的，我们可以从一个查询开始，找到所有基本元素 - 在这种特定情况下是产品元素，然后我们可以从执行查询的元素中执行一个新的查询，该查询将仅搜索在执行查询的元素内部的元素。

我们在上一节中用来查询 DOM 的方法包括直接从 DOM 中使用`childNodes`列表访问元素，或者使用`getElementsByTagName`和`getElementsByClassName`方法。除了这些方法，DOM 还提供了一些其他非常强大的查询元素的方法。

首先，有`getElement*`方法系列：

+   `getElementsByTagName` - 我们之前见过并使用过这个方法。它获取指定标签的所有元素。

+   `getElementsByClassName` - 这是`getElement`的一个变体，它返回具有指定类的所有元素。请记住，一个元素可以通过用空格分隔它们来包含一个类的列表。以下是在商店页面中运行的代码的屏幕截图，您可以看到选择`ui`类名将获取还具有`items`、`teal`（颜色）和`label`类的元素：

![](img/C14587_01_43.jpg)

###### 图 1.43：按类名获取元素通常返回包含其他类的元素

+   `getElementById` - 注意该方法名称中的单数形式。该方法将获取具有指定 ID 的唯一元素。这是因为在页面上预期 ID 是唯一的。

`getElement*`方法族非常有用。但有时，指定类或标记名称是不够的。这意味着您必须使用一系列操作来使您的代码非常复杂：获取所有具有此类的元素，然后获取具有此其他标记的元素，然后获取具有此类的元素，然后选择第三个，依此类推。

多年来，jQuery 是唯一的解决方案，直到引入了`querySelector`和`querySelectorAll`方法。这两种方法可以用来在 DOM 树上执行复杂的查询。它们的工作方式完全相同。两者之间唯一的区别是`querySelector`只会返回与查询匹配的第一个元素，而`querySelectorAll`会返回一个可以迭代的列表。

`querySelector*`方法使用 CSS 选择器。您可以使用任何 CSS 选择器来查询元素。让我们在下一个练习中更深入地探索一下。

### 练习 5：使用 querySelector 查询 DOM

在这个练习中，我们将探索在之前章节学到的各种查询和节点导航技术。为此，我们将使用商店代码作为基本 HTML 来探索，并编写 JavaScript 代码来查找商店页面上所有有机水果的名称。为了增加难度，有一个标记为有机的蓝莓松饼。

在开始之前，让我们看一下`product`元素及其子元素。以下是从`Elements`选项卡查看的`product`元素的 DOM 树：

![图 1.44：产品元素及其子元素](img/C14587_01_44.jpg)

###### 图 1.44：产品元素及其子元素

您可以看到每个产品的根元素是一个带有`class`项的`div`标记。名称和标记位于一个带有类 content 的子 div 中。产品的名称位于一个带有类 header 的锚点中。标记是一组带有三个类`ui`、`label`和`teal`的`div`标记。

在处理这样的问题时，您想要查询和过滤一组在一个共同父级下相关的元素时，有两种常见的方法：

+   首先查询根元素，然后进行过滤和查找所需的元素。以下是这种方法的图形表示：

![图 1.45：第一种方法涉及从根元素开始](img/C14587_01_45.jpg)

###### 图 1.45：第一种方法涉及从根元素开始

+   从匹配过滤条件的子元素开始，如果需要，应用额外的过滤，然后导航到您要查找的元素。以下是这种方法的图形表示：

![图 1.46：第二种方法涉及从过滤条件开始](img/C14587_01_46.jpg)

###### 图 1.46：第二种方法涉及从过滤条件开始

执行以下步骤完成练习：

1.  为了使用第一种方法解决练习，我们需要一个函数来检查产品是否包含指定的标签列表。这个函数的名称将是`the`，它接收两个参数-产品根元素和要检查的标签列表：

```js
function containLabels(element, ...labelsToCheck) {
}
```

1.  在这个函数中，我们将使用一些数组映射和过滤来找到参数中指定的标签和被检查产品的标签之间的交集：

```js
const intersection = Array.from(element.querySelectorAll('.label'))
  .map(e => e.innerHTML)
  .filter(l => labelsToCheck.includes(l));
```

1.  函数中的最后一件事是返回一个检查，告诉我们产品是否包含所有标签。检查告诉我们交集的大小是否与要检查的所有标签的大小相同，如果是，我们就有一个匹配：

```js
return intersection.length == labelsToCheck.length;
```

1.  现在我们可以使用查询方法来查找元素，将它们添加到数组中，进行过滤和映射到我们想要的内容，然后打印到控制台：

```js
//Start from the product root element
Array.from(document.querySelectorAll('.item'))
//Filter the list to only include the ones with both labels
.filter(e => containLabels(e, 'organic', 'fruit'))
//Find the product name
.map(p => p.querySelector('.content a.header'))
.map(a => a.innerHTML)
//Print to the console
.forEach(console.log);
```

1.  要使用第二种方法解决问题，我们需要一个函数来查找指定元素的所有兄弟元素。打开您的文本编辑器，让我们从声明带有数组的函数开始存储我们找到的所有兄弟元素。然后，我们将返回数组：

```js
function getAllSiblings(element) {
  const siblings = [];
  // rest of the code goes here
  return siblings;
}
```

1.  然后，我们将使用`while`循环和`previousElementSibling`属性迭代所有先前的兄弟元素。在迭代兄弟元素时，我们将它们推入数组中：

```js
let previous = element.previousElementSibling;
while (previous) {
  siblings.push(previous);
  previous = previous.previousElementSibling;
}
```

#### 注意：再次注意间隙

我们使用`previousElementSibling`而不是`previousNode`，因为这将排除所有文本节点和其他节点，以避免不得不为每个节点检查`nodeType`。

1.  对于指定元素之后的所有兄弟元素，我们做同样的操作：

```js
let next = element.nextElementSibling;
while (next) {
  siblings.push(next);
  next = next.nextElementSibling;
}
```

1.  现在我们有了`getAllSiblings`函数，我们可以开始查找产品。我们可以使用`querySelectorAll`函数，以及一些数组映射和过滤来找到并打印我们想要的数据：

```js
//Start by finding all the labels with content 'organic'
Array.from(document.querySelectorAll('.label'))
.filter(e => e.innerHTML === 'organic')
//Filter the ones that don't have a sibling label 'fruit'
.filter(e => getAllSiblings(e).filter(s => s.innerHTML === 'fruit').length > 0)
//Find root product element
.map(e => e.closest('.item'))
//Find product name
.map(p => p.querySelector('.content a.header').innerHTML)
//Print to the console
.forEach(console.log);
```

1.  在**开发者工具**的**控制台**选项卡中执行代码，您将看到以下输出：

![图 1.47：练习中代码的输出。打印所有有机水果的名称。](img/C14587_01_47.jpg)

###### 图 1.47：练习中代码的输出。打印所有有机水果的名称。

#### 注意

此练习的代码可以在 GitHub 上找到。包含第一种方法代码的文件路径是：[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Exercise05/first_approach.js`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Exercise05/first_approach.js)。

包含第二种方法代码的文件路径是：[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Exercise05/second_approach.js`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Exercise05/second_approach.js)。

在这个练习中，我们使用了两种不同的技术从页面中获取数据。我们使用了许多查询和节点导航方法和属性来查找元素并在 DOM 树中移动。

当构建现代 Web 应用程序时，了解这些技术是至关重要的。在这种类型的应用程序中，导航 DOM 和获取数据是最常见的任务。

### 操作 DOM

现在我们知道了 DOM 是什么，以及如何查询元素和在其周围导航，是时候学习如何使用 JavaScript 来更改它了。在本节中，我们将重写商店前端，通过使用 JavaScript 加载产品列表并创建页面元素，使其更具交互性。

本节的示例代码可以在 GitHub 上找到：[`bit.ly/2mMje1K`](https://bit.ly/2mMje1K)。

在使用 JavaScript 创建动态应用程序时，我们需要知道的第一件事是如何创建新的 DOM 元素并将它们附加到树中。由于 DOM 规范完全基于接口，没有具体的类可实例化。当您想要创建 DOM 元素时，需要使用`document`对象。`document`对象有一个名为`createElement`的方法，它接收一个标签名称作为字符串。以下是创建`div`元素的示例代码：

```js
const root = document.createElement('div');
```

`product`项元素具有`item`类。要将该类添加到它，我们只需设置`className`属性，如下所示：

```js
root.className = 'item';
```

现在我们可以将元素附加到需要去的地方。但首先，我们需要找到它需要去的地方。此示例代码的 HTML 可以在 GitHub 上找到[`bit.ly/2nKucVo`](https://bit.ly/2nKucVo)。您可以看到它有一个空的`div`元素，产品项将被添加到其中：

```js
<div class="ui items"></div>
```

我们可以使用`querySelector`来找到该元素，然后在其上调用`appendChild`方法，这是每个节点都有的方法，并将刚刚创建的元素节点传递给它，以便将其添加到 DOM 树中：

```js
const itemsEl = document.querySelector('.items');
products.forEach((product) => {
  itemsEl.appendChild(createProductItem(product));
});
```

在这里，`createProductItem`是一个函数，它接收一个产品并使用先前提到的`createElement`函数为其创建 DOM 元素。

创建一个 DOM 元素并没有太大的用处。对于动态商店示例，我们有一个包含我们构建页面所需的所有数据的对象数组。对于每一个对象，我们需要创建所有的 DOM 元素，并将它们粘合在正确的位置和顺序上。但首先，让我们来看看数据是什么样子的。以下显示了每个`product`对象的外观：

```js
{
  "price": 3.99,
  "unit": "lb",
  "name": "Apples",
  "description": "Lorem ipsum dolor sit amet, ...",
  "image": "../images/products/apples.jpg",
  "tags": [ "fruit", "organic" ]
}
```

以下是我们在之前章节中使用的静态商店代码中相同产品的 DOM 看起来的方式：

![图 1.48：产品的 DOM 树部分](img/C14587_01_48.jpg)

###### 图 1.48：产品的 DOM 树部分

您可以看到有许多嵌套的元素需要创建才能得到所需的最终 DOM 树。因此，让我们看看在使用 JavaScript 构建复杂应用程序时非常有用的一些技术。

让我们开始看一下示例代码中的`createProductItem`：

```js
function createProductItem(product) {
  const root = document.createElement('div');
  root.className = 'item';
  root.appendChild(createProductImage(product.image));
  root.appendChild(createContent(product));
  return root;
}
```

我们通过创建产品树的根元素开始这个方法，这是一个`div`元素。从前面的截图中，您可以看到这个`div`需要一个`item`类，这就是在元素创建后的下一行发生的事情，就像本节开头描述的那样。

元素准备好后，就可以开始向其添加子元素了。我们不是在同一个方法中完成所有操作，而是创建其他负责创建每个子元素的函数，并直接调用它们，将每个函数的结果附加到根元素：

```js
root.appendChild(createProductImage(product.image));
root.appendChild(createContent(product));
```

这种技术很有用，因为它将每个子元素的逻辑隔离在自己的位置上。

现在让我们来看一下`createProductImage`函数。从之前的示例代码中，您可以看到该函数接收`product`图像的路径。这是该函数的代码：

```js
function createProductImage(imageSrc) {
  const imageContainer = document.createElement('div');
  imageContainer.className = 'image';
  const image = document.createElement('img');
  image.setAttribute('src', imageSrc);
  imageContainer.appendChild(image);
  return imageContainer;
}
```

该函数分为两个主要部分：

1.  它创建图像的容器元素。从 DOM 截图中，您可以看到`img`元素位于一个带有`image`类的`div`内。

1.  它创建`img`元素，设置`src`属性，然后将其附加到`container`元素。

这种代码风格简单、可读且易于理解。但这是因为需要生成的 HTML 相当简短。它只是一个`div`标签中的一个`img`标签。

不过，有时树变得非常复杂，使用这种策略使得代码几乎无法阅读。因此，让我们看看另一种策略。附加到产品根元素的另一个子元素是`content`元素。这是一个具有许多子元素的`div`标签，包括一些嵌套的子元素。

我们可以像`createProductImage`函数一样处理它。但是该方法需要执行以下操作：

1.  创建`container`元素并为其添加一个类。

1.  创建包含产品名称的锚元素并将其附加到容器。

1.  创建价格的容器并将其附加到根容器。

1.  创建带有价格的`span`元素并将其附加到上一步中创建的元素。

1.  创建包含描述的元素并将其附加到容器。

1.  为`tag`元素创建`container`元素并将其附加到根容器。

1.  对于每个标签，创建`tag`元素并将其附加到上一步中的容器。

听起来像是一长串步骤，不是吗？我们可以使用模板字符串来生成 HTML，然后为容器元素设置`innerHTML`，而不是试图编写所有那些代码。因此，步骤看起来会像下面这样：

1.  创建`container`元素并为其添加一个类。

1.  使用字符串模板创建内部内容的 HTML。

1.  在`container`元素上设置`innerHTML`。

这听起来比以前的方法简单得多。而且，正如我们将看到的那样，它也会更加可读。让我们来看看代码。

如前所述，第一步是创建根容器并为其添加类：

```js
function createContent(product) {
  const content = document.createElement('div');
  content.className = 'content';
```

然后，我们开始生成`tag`元素的 HTML。为此，我们有一个函数，它接收标签作为字符串并返回其 HTML 元素。我们使用它将所有标签映射到使用`tags`数组上的`map`函数的元素。然后，我们通过使用其`outerHTML`属性将元素映射到 HTML：

```js
 const tagsHTML = product.tags.map(createTagElement)
    .map(el => el.outerHTML)
    .join('');
```

有了`container`元素创建和标签的 HTML 准备好后，我们可以使用模板字符串设置`content`元素的`innerHTML`属性并返回它：

```js
  content.innerHTML = `
    <a class="header">${product.name}</a>
    <div class="meta"><span>$${product.price} / ${product.unit}</span></div>
    <div class="description">${product.description}</div>
    <div class="extra">${tagsHTML}</div>
  `;
  return content;
}
```

与生成 HTML 元素并附加它们所需的许多步骤相比，这段代码要简短得多，更容易理解。在编写动态应用程序时，您可以决定在每种情况下哪种方式最好。在这种情况下，权衡基本上是可读性和简洁性。但对于其他情况，权衡也可以是根据某些过滤器要求缓存元素以添加事件侦听器或隐藏/显示它们。

### 练习 6：过滤和搜索产品

在这个练习中，我们将为我们的商店应用程序添加两个功能，以帮助我们的客户更快地找到产品。首先，我们将使标签可点击，这将通过所选标签过滤产品列表。然后，我们将在顶部添加一个搜索框，供用户按名称或描述查询。页面将如下所示：

![图 1.49：顶部带有搜索栏的新商店前端](img/C14587_01_49.jpg)

###### 图 1.49：顶部带有搜索栏的新商店前端

在这个新的商店前端，用户可以点击标签来过滤具有相同标签的产品。当他们这样做时，用于过滤列表的标签将显示在顶部，呈橙色。用户可以点击搜索栏中的标签以删除过滤器。页面如下所示：

![图 1.50：顶部标签过滤的工作原理](img/C14587_01_50.jpg)

###### 图 1.50：顶部标签过滤的工作原理

用户还可以使用右侧的搜索框按名称或描述搜索产品。随着他们的输入，列表将被过滤。

此练习的代码可以在 GitHub 上找到：[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson01/Exercise06`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson01/Exercise06)。

执行以下步骤以完成练习：

1.  我们将首先编写基本的 HTML 代码，稍后将使用 JavaScript 添加所有其他元素。此 HTML 现在包含一个基本的`div`容器，其中将包含所有内容。其中的内容分为两部分：一个包含标题的部分，其中包含标题和搜索栏，以及一个`div`，其中将包含所有产品项目。创建一个名为`dynamic_storefront.html`的文件，并在其中添加以下代码：

```js
<html>
  <head>
    <link rel="stylesheet" type="text/css" href="../css/semantic.min.css" />
    <link rel="stylesheet" type="text/css" href="../css/store_with_header.css" />
  </head>
  <body>
    <div id="content">
      <section class="header">
        <h1 class="title">Welcome to Fresh Products Store!</h1>
        <div class="ui menu">
          <div class="right item">
            <div class="ui icon input">
              <input type="text" placeholder="Search..." />
              <i class="search icon"></i>
            </div>
          </div>
        </div>
      </section>
      <div class="ui items"></div>
    </div>
    <script src="../data/products.js"></script>
    <script src="../sample_003/create_elements.js"></script>
    <script src="filter_and_search.js"></script>
  </body>
</html>
```

此 HTML 使用了`products.js`和`create_elements.js`脚本，这与本节中使用的示例代码相同。它还使用了`Lesson01`文件夹中的 CSS 文件。如果您在同一个文件夹中，可以直接参考它们，或者将它们复制粘贴到您的项目中。

1.  创建一个名为`filter_and_search.js`的文件，这是在 HTML 代码中加载的最后一个 JavaScript 代码。这是我们将为此练习添加所有代码的地方。我们需要做的第一件事是存储过滤器状态。用户可以应用到页面的两种可能过滤器：选择标签和/或输入一些文本。为了存储它们，我们将使用一个数组和一个字符串变量：

```js
const tagsToFilterBy = [];
let textToSearch = '';
```

1.  现在我们将创建一个函数，该函数将为页面中的所有标签添加事件侦听器。此函数将查找所有`tag`元素，将它们包装在一个数组中，并使用`Element`中的`addEventListener`方法添加事件侦听器以响应`click`事件：

```js
function addTagFilter() {
  Array.from(document.querySelectorAll('.extra .label')).forEach(tagEl => {
    tagEl.addEventListener('click', () => {
      // code for next step goes here
    });
  });
}
```

1.  在事件侦听器中，我们将检查标签是否已经在要按其进行过滤的标签数组中。如果没有，我们将添加它并调用另一个名为`applyTagFilters`的函数：

```js
if (!tagsToFilterBy.includes(tagEl.innerHTML)) {
  tagsToFilterBy.push(tagEl.innerHTML);
  applyFilters();
}
```

1.  `applyFilters`只是一个包含与更新页面相关的所有逻辑的捕捉函数。您将只调用我们将在接下来的步骤中编写的函数：

```js
function applyFilters() {
  createListForProducts(filterByText(filterByTags(products)));
  addTagFilter();
  updateTagFilterList();
}
```

1.  在继续`applyFilters`函数之前，我们将添加另一个函数来处理文本搜索输入框上的事件。这个处理程序将监听`keyup`事件，当用户完成输入每个字母时触发。处理程序将获取输入框中的当前文本，将值设置为`textToSearch`变量，并调用`applyFilters`函数：

```js
function addTextSearchFilter() {
  document.querySelector('.menu .right input'
.addEventListener('keyup', (e) => {
      textToSearch = e.target.value;
      applyFilters();
    });
}
```

1.  现在，回到`applyFilters`函数。在其中调用的第一个函数几乎是隐藏的。这就是`filterByTags`函数，它使用`tagsToFilterBy`数组对产品列表进行过滤。它使用递归的方式对传入的产品列表使用选择的标签进行过滤：

```js
function filterByTags() {
  let filtered = products;
  tagsToFilterBy
    .forEach((t) => filtered = filtered.filter(p => p.tags.includes(t)));
  return filtered;
}
```

1.  无论过滤函数的输出是什么，都会传递给另一个过滤函数，即基于文本搜索过滤产品的函数。`filterByText`函数在比较之前将所有文本转换为小写。这样，搜索将始终不区分大小写：

```js
function filterByText(products) {
  const txt = (textToSearch || '').toLowerCase();
  return products.filter((p) => {
    return p.name.toLowerCase().includes(txt)
      || p.description.toLowerCase().includes(txt);
  });
}
```

在通过选择的标签进行过滤和通过输入的文本进行过滤之后，我们将过滤后的数值传递给`createListForProducts`，这是`create_elements.js`中的一个函数，在本节练习之前已经描述过。

1.  现在我们已经在页面上显示了新产品列表，我们需要重新注册标签过滤器事件监听器，因为 DOM 树元素已经被重新创建。所以我们再次调用`addTagFilter`。如前所示，这就是`applyFilters`函数的样子：

```js
function applyFilters() {
  createListForProducts(filterByText(filterByTags(products)));
  addTagFilter();
  updateTagFilterList();
}
```

1.  `applyTagFilter`函数中调用的最后一个函数是`updateTagFilterList`。此函数将找到将保存过滤器指示器的元素，检查是否有选定的标签进行过滤，并相应地进行更新，要么将文本设置为`无过滤器`，要么为每个应用的标签添加指示器：

```js
function updateTagFilterList() {
  const tagHolder = document.querySelector('.item span.tags');
  if (tagsToFilterBy.length == 0) {
    tagHolder.innerHTML = 'No filters';
  } else {
    tagHolder.innerHTML = '';
    tagsToFilterBy.sort();
    tagsToFilterBy.map(createTagFilterLabel)
      .forEach((tEl) => tagHolder.appendChild(tEl));
  }
}
```

1.  我们需要将所有这些联系在一起的最后一个函数是`createTagFilterLabel`函数，它用于在搜索栏中创建标签被选中的指示器。此函数将创建 DOM 元素并添加一个事件侦听器，当单击时，将从数组中删除标签并再次调用`applyTagFilter`函数：

```js
function createTagFilterLabel(tag) {
  const el = document.createElement('span');
  el.className = 'ui label orange';
  el.innerText = tag;
  el.addEventListener('click', () => {
    const index = tagsToFilterBy.indexOf(tag);
    tagsToFilterBy.splice(index, 1);
    applyTagFilter();
  });

  return el;
}
```

1.  使页面工作的最后一步是调用`applyTagFilter`函数，以便将页面更新到初始状态，即未选择任何标签。此外，它将调用`addTextSearchFilter`以添加文本框的事件处理程序：

```js
addTextSearchFilter();
applyFilters();
```

在 Chrome 中打开页面，您会看到顶部的过滤器为空，并且所有产品都显示在列表中。它看起来像本练习开头的截图。单击一个标签或在文本框中输入内容，您会看到页面更改以反映新状态。例如，选择两个**饼干**和**面包店**标签，并在文本框中输入**巧克力**，页面将只显示具有这两个标签和名称或描述中包含**巧克力**的产品：

![图 1.51：商店前端通过两个面包店和饼干标签以及单词巧克力进行过滤](img/C14587_01_51.jpg)

###### 图 1.51：商店前端通过两个面包店和饼干标签以及单词巧克力进行过滤

在本练习中，您已经学会了如何响应用户事件并相应地更改页面，以反映用户希望页面处于的状态。您还学会了当元素被移除并重新添加到页面时，事件处理程序会丢失并需要重新注册。

### 影子 DOM 和 Web 组件

在之前的部分中，我们已经看到一个简单的 Web 应用可能需要复杂的编码。当应用程序变得越来越大时，它们变得越来越难以维护。代码开始变得混乱，一个地方的变化会影响其他意想不到的地方。这是因为 HTML、CSS 和 JavaScript 的全局性质。

已经创建了许多解决方案来尝试规避这个问题，**万维网联盟**（**W3C**）开始着手提出标准的方式来创建自定义的、隔离的组件，这些组件可以拥有自己的样式和 DOM 根。Shadow DOM 和自定义组件是从这一倡议中诞生的两个标准。

Shadow DOM 是一种创建隔离的 DOM 子树的方式，可以拥有自己的样式，并且不受添加到父树的样式的影响。它还隔离了 HTML，这意味着在文档树上使用的 ID 可以在每个影子树中多次重用。

以下图示了处理 Shadow DOM 时涉及的概念：

![图 1.52：Shadow DOM 概念](img/C14587_01_52.jpg)

###### 图 1.52：Shadow DOM 概念

让我们描述一下这些概念的含义：

+   **文档树**是页面的主要 DOM 树。

+   **影子宿主**是附加影子树的节点。

+   **影子树**是附加到文档树的隔离 DOM 树。

+   **影子根**是影子树中的根元素。

影子宿主是文档树中附加影子树的元素。影子根元素是一个不显示在页面上的节点，就像主文档树中的文档对象一样。

要理解这是如何工作的，让我们从一些具有奇怪样式的 HTML 开始：

```js
<style>
  p {
    background: #ccc;
    color: #003366;
  }
</style>
```

这将使页面上的每个段落都具有灰色背景，并带有一些蓝色文字。这是页面上段落的样子：

![图 1.53：应用了样式的段落](img/C14587_01_53.jpg)

###### 图 1.53：应用了样式的段落

让我们添加一个影子树，并在其中添加一个段落，看看它的行为。我们将使用`div`元素将段落元素包装起来，并添加一些文本：

```js
<div><p>I'm in a Shadow DOM tree.</p></div>
```

然后我们可以在元素中使用`attachShadow`方法创建一个影子根元素：

```js
const shadowHost = document.querySelector('div');
const shadowRoot = shadowHost.attachShadow({ mode: 'open' });
```

上面的代码选择了页面上的`div`元素，然后调用`attachShadow`方法，将配置对象传递给它。配置表示这个影子树是打开的，这意味着可以通过元素的`shadowRoot`属性访问它的影子根元素 - 在这种情况下是`div`：

![图 1.54：可以通过附加树的元素访问打开的影子树](img/C14587_01_54.jpg)

###### 图 1.54：可以通过附加树的元素访问打开的影子树

影子树可以关闭，但不建议采用这种方法，因为这会产生一种虚假的安全感，并且会让用户的生活变得更加困难。

在我们将影子树附加到文档树后，我们可以开始操纵它。让我们将影子宿主中的 HTML 复制到影子根中，看看会发生什么：

```js
shadowRoot.innerHTML = shadowHost.innerHTML;
```

现在，如果您在 Chrome 中加载页面，您会看到以下内容：

![图 1.55：加载了影子 DOM 的页面](img/C14587_01_55.jpg)

###### 图 1.55：加载了影子 DOM 的页面

您可以看到，即使向页面添加了样式来选择所有段落，但向影子树添加的段落不受其影响。Shadow DOM 中的元素与文档树完全隔离。

现在，如果您查看 DOM，您会发现有些地方看起来很奇怪。影子树替换并包装了原来在`div`元素内部的段落，这就是影子宿主：

![图 1.56：影子树与影子宿主中的其他节点处于同一级别](img/C14587_01_56.jpg)

###### 图 1.56：影子树与影子宿主中的其他节点处于同一级别

但是影子宿主内部的原始段落不会在页面上呈现。这是因为当浏览器渲染页面时，如果元素包含具有新内容的影子树，它将替换宿主下的当前树。这个过程称为平铺，下面的图表描述了它的工作原理：

![](img/C14587_01_57.jpg)

###### 图 1.57：平铺时，浏览器会忽略影子宿主下的节点

现在我们了解了 Shadow DOM 是什么，我们可以开始使用它来构建或者自己的 HTML 元素。没错！通过自定义组件 API，你可以创建自己的 HTML 元素，然后像任何其他元素一样使用它。

在本节的其余部分，我们将构建一个名为**counter**的自定义组件，它有两个按钮和中间的文本。你可以点击按钮来增加或减少存储的值。你还可以配置它具有初始值和不同的增量值。下面的屏幕截图显示了组件完成后的外观。这个代码存放在 GitHub 上，网址是[`bit.ly/2mVy1XP`](https://bit.ly/2mVy1XP)：

![图 1.58：计数器组件及其在 HTML 中的使用](img/C14587_01_58.jpg)

###### 图 1.58：计数器组件及其在 HTML 中的使用

要定义你的自定义组件，你需要在自定义组件注册表中调用`define`方法。有一个名为`customElements`的全局注册表实例。要注册你的组件，你调用`define`，传递你的组件将被引用的字符串。它至少需要有一个破折号。你还需要传递实例化你的组件的构造函数。下面是代码：

```js
customElements.define('counter-component', Counter);
```

你的构造函数可以是一个普通函数，或者，就像这个例子中一样，你可以使用新的 JavaScript `class`定义。它需要扩展`HTMLElement`：

```js
class Counter extends HTMLElement {
}
```

为了使自定义组件与页面的其余部分隔离，你可以使用一个阴影树，其中阴影主机是你的组件元素。你不需要使用 Shadow DOM 来构建自定义组件，但建议对于更复杂的组件也包装一些样式。

在你的元素的构造函数中，通过调用`attachShadow`来创建自己的实例的阴影根：

```js
constructor() {
  super(); // always call super first
  // Creates the shadow DOM to attach the parts of this component
  this.attachShadow({mode: 'open'});
  // ... more code here
}
```

记住，当你使用`open`模式将阴影 DOM 附加到元素时，元素将把该阴影根存储在`shadowRoot`属性中。所以，从现在开始我们可以使用`this.shadowRoot`来引用它。

在前面的图中，你看到`counter`组件有两个属性，它用来配置自身：`value`和`increment`。这些属性在构造函数的开始使用`Element`的`getAttribute`方法设置，并在没有可用时设置合理的默认值：

```js
this.value = parseInt(this.getAttribute('value') || 0);
this.increment = parseInt(this.getAttribute('increment') || 1);
```

之后，我们为这个组件创建了所有的 DOM 元素，并将它们附加到阴影根。我们不会深入细节，因为你现在已经看到了足够的 DOM 操作。在构造函数中，我们只是调用创建这些元素的函数，并使用`this.shadowRoot.appendChild`将它们附加：

```js
// Create and attach the parts of this component
this.addStyles();
this.createButton('-', () => this.decrementValue());
this.createValueSpan();
this.createButton('+', () => this.incrementValue());
```

第一个方法创建一个`link`元素，导入`counter`组件的 CSS 文件。第二和第四个方法创建`decrement`和`increment`按钮，并附加事件处理程序。第三个方法创建一个`span`元素，并在`property`下保留对它的引用。

`incrementValue`和`decrementValue`方法通过指定的数量增加当前值，然后调用`updateState`方法，将值的状态与 DOM（在这种情况下是 Shadow DOM）同步。`incrementValue`和`updateState`方法的代码如下：

```js
incrementValue() {
  this.value += this.increment;
  this.triggerValueChangedEvent();
  this.updateState();
}
updateState() {
  this.span.innerText = `Value is: ${this.value}`;
}
```

在`incrementValue`函数中，我们还调用函数来触发事件，通知用户值已经改变。这个函数将在后面讨论。

现在你已经定义并注册了你的新的`HTMLElement`，你可以像任何其他现有的 HTML 元素一样使用它。你可以通过 HTML 代码中的标签添加它，如下所示：

```js
<counter-component></counter-component>
<counter-component value="7" increment="3"></counter-component>
```

或者，通过 JavaScript，通过创建一个元素并将其附加到 DOM 中：

```js
const newCounter = document.createElement('counter-component');
newCounter.setAttribute('increment', '2');
newCounter.setAttribute('value', '3');
document.querySelector('div').appendChild(newCounter);
```

要完全理解 Web 组件的强大之处，还有两件事情你需要知道：回调和事件。

自定义组件有生命周期回调，你可以在你的类中设置它们，以便在它们周围的事情发生变化时得到通知。最重要的两个是`connectedCallback`和`attributeChangedCallback`。

第一个对于当你想要在组件附加到 DOM 后操纵 DOM 时很有用。对于**counter**组件，我们只是在控制台上打印一些东西，以显示组件现在连接到了 DOM：

```js
connectedCallback() {
  console.log("I'm connected to the DOM!");
}
```

当页面加载时，你可以看到为每个**counter**组件添加到 DOM 中打印的语句：

![图 1.59：当计数器组件附加到 DOM 时在控制台中打印的语句](img/C14587_01_59.jpg)

###### 图 1.59：当计数器组件附加到 DOM 时在控制台中打印的语句

`attributeChangedCallback`在组件中的某个属性被更改时被调用。但是为了让它工作，你需要一个静态的 getter，它会告诉你想要被通知属性的更改。以下是静态 getter 的代码：

```js
static get observedAttributes() {
  return ['value', 'increment'];
}
```

它只是返回一个包含我们想要被通知的所有属性的数组。`attributeChangedCallback`接收几个参数：更改的属性名称，旧值（如果没有设置，则为 null），和新值。以下是**counter**组件的回调代码：

```js
attributeChangedCallback(attribute, _, newValue) {
  switch(attribute) {
    case 'increment':
      this.increment = parseInt(newValue);
      break;
    case 'value':
      this.value = parseInt(newValue);
      break;
  }
  this.updateState();
}
```

我们的回调检查属性名称，忽略旧值，因为我们不需要它，将其转换为整数，解析为整数，并根据属性的名称相应地设置新值。最后，它调用`updateState`函数，该函数将根据其属性更新组件的状态。

关于网络组件的最后一件事是你需要知道如何分发事件。事件是标准组件的重要部分；它们构成了与用户的所有交互的基础。因此，将逻辑封装到组件中的一个重要部分是理解你的组件的用户将对哪些事件感兴趣。

对于我们的**counter**组件，每当值更改时分发事件是非常有意义的。在事件中传递值也是有用的。这样，用户就不需要查询你的组件来获取当前值。

要分发自定义事件，我们可以使用`Element`的`dispatchEvent`方法，并使用`CustomEvent`构造函数来使用自定义数据构建我们的事件。我们的事件名称将是`value-changed`。用户可以添加事件处理程序来监听此事件，并在值更改时收到通知。

以下代码是`triggerValueChangedEvent`函数，之前提到过；这个函数从`incrementValue`和`decrementValue`函数内部调用： 

```js
triggerValueChangedEvent() {
  const event = new CustomEvent('value-changed', { 
    bubbles: true,
    detail: { value: this.value },
  });
  this.dispatchEvent(event);
}
```

这个函数创建了一个`CustomEvent`的实例，它在 DOM 中冒泡，并在`detail`属性中包含当前值。我们本可以创建一个普通的事件实例，并直接在对象上设置属性，但是对于自定义事件，建议使用`CustomEvent`构造函数，它可以正确处理自定义数据。创建事件后，调用`dispatchEvent`方法，传递事件。

现在我们已经发布了事件，我们可以注册并在页面上显示信息。以下是查询所有`counter-components`并为`value-changed`事件添加事件侦听器的代码。处理程序在每次单击组件时向现有的`div`添加一个段落：

```js
const output = document.getElementById('output');
Array.from(document.querySelectorAll('counter-component'))
  .forEach((el, index) => {
    el.addEventListener('value-changed', (e) => {
    output.innerHTML += '<p>Counter ${index} value is now ${e.detail.value}</p>';
  });
});
```

这是在不同计数器上点击几次后页面的外观：

![图 1.60：页面上添加的段落，显示计数器被点击](img/C14587_01_60.jpg)

###### 图 1.60：页面上添加的段落，显示计数器被点击

### 练习 7：用网络组件替换搜索框

要完全理解网络组件的概念，你需要看看一个应用程序如何被分解为封装的、可重用的组件。我们在上一个练习中构建的商店页面是我们开始的好地方。

在这个练习中，我们将编写一个网络组件，以替换页面右上角的搜索框。这就是我们谈论的组件：

![图 1.61：将转换为 Web 组件的搜索框](img/C14587_01_61.jpg)

###### 图 1.61：将转换为 Web 组件的搜索框

这个组件将处理它的外观、渲染和状态，并在状态改变时发出事件。在这种情况下，搜索框只有一个状态：**搜索**文本。

执行以下步骤以完成练习：

1.  将代码从`Exercise 6`复制到一个新文件夹中，这样我们就可以在不影响现有 storefront 的情况下进行更改。

1.  让我们开始创建一个 Web 组件。创建一个名为`search_box.js`的文件，添加一个名为`SearchBox`的新类，并使用这个类定义一个新组件：

```js
class SearchBox extends HTMLElement {
}
customElements.define('search-box', SearchBox);
```

1.  在类中，添加一个构造函数，调用`super`，并将组件附加到一个影子根。构造函数还将通过设置一个名为`_searchText`的变量来初始化状态：

```js
constructor() {
  super();
  this.attachShadow({ mode: 'open' });
  this._searchText = '';
}
```

1.  为了公开当前状态，我们将为`_searchText`字段添加一个 getter：

```js
get searchText() {
  return this._searchText;
```

1.  仍然在类中，创建一个名为`render`的方法，它将把`shadowRoot.innerHTML`设置为我们想要的模板组件。在这种情况下，它将是搜索框的现有 HTML 加上一个指向 semantic UI 样式的链接，以便我们可以重用它们：

```js
render() {
  this.shadowRoot.innerHTML = '
    <link rel="stylesheet" type="text/css" href="../css/semantic.min.css" />
    <div class="ui icon input">
      <input type="text" placeholder="Search..." />
      <i class="search icon"></i>
    </div>
  ';
}
```

1.  创建另一个名为`triggerTextChanged`的方法，它将触发一个事件来通知监听器搜索文本已更改。它接收新的文本值并将其传递给监听器：

```js
triggerTextChanged(text) {
  const event = new CustomEvent('changed', {
    bubbles: true,
    detail: { text },
  });
  this.dispatchEvent(event);
}
```

1.  在构造函数中，在附加影子根后，调用`render`方法并注册一个监听器到输入框，以便我们可以为我们的组件触发 changed 事件。构造函数现在应该是这样的：

```js
constructor() {
  super();
  this.attachShadow({ mode: 'open' });
  this._searchText = '';
  this.render();
  this.shadowRoot.querySelector('input').addEventListener('keyup', (e) => {
    this._searchText = e.target.value;
    this.triggerTextChanged(this._searchText);
  });
}
```

1.  准备好我们的 Web 组件后，我们可以用它替换旧的搜索框。在`dynamic_storefront.html` HTML 中，用我们创建的新组件`search-box`替换`div`标签和它们的所有内容。还要将新的 JavaScript 文件添加到 HTML 中，放在所有其他脚本之前。您可以在 GitHub 上查看最终的 HTML，网址为[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Exercise07/dynamic_storefront.html`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Exercise07/dynamic_storefront.html)。

1.  通过使用文档的`querySelector`方法保存对`search-box`组件的引用：

```js
const searchBoxElement = document.querySelector('search-box');
```

1.  注册一个 changed 事件的事件监听器，这样我们就知道何时有新值可用，并调用`applyFilters`：

```js
searchBoxElement.addEventListener('changed', (e) => applyFilters());
```

1.  现在我们可以清理`filter_and_search.js` JavaScript，因为部分逻辑已经移动到新组件中。我们将进行以下清理：

删除`textToSearch`变量（第 2 行），并将其替换为`searchBoxElement.searchText`（第 40 行）。

删除`addTextSearchFilter`函数（第 16-22 行）和脚本末尾对它的调用（第 70 行）。

如果一切顺利，在 Chrome 中打开文件将得到完全相同的 storefront，这正是我们想要的。

现在，处理搜索框和搜索文本的逻辑已经封装起来，这意味着如果我们需要更改它，我们不需要四处寻找分散在各处的代码片段。当我们需要知道搜索文本的值时，我们可以查询保存它的组件。

### 活动 2：用 Web 组件替换标签过滤器

现在我们已经用 web 组件替换了搜索框，让我们使用相同的技术替换标签过滤器。这个想法是我们将有一个组件来存储选定的标签列表。

这个组件将封装一个可以通过使用`mutator`方法（`addTag`和`removeTag`）来修改的选定标签列表。当内部状态发生变化时，会触发一个 changed 事件。此外，当列表中的标签被点击时，将触发一个`tag-clicked`事件。

步骤：

1.  首先将代码从练习 7 复制到一个新文件夹中。

1.  创建一个名为`tags_holder.js`的新文件，在其中添加一个名为`TagsHolder`的类，它扩展了`HTMLElement`，然后定义一个名为`tags-holder`的新自定义组件。

1.  创建两个`render`方法：一个用于渲染基本状态，另一个用于渲染标签或指示未选择任何标签进行过滤的文本。

1.  在构造函数中，调用`super`，将组件附加到影子根，初始化所选标签列表，并调用两个`render`方法。

1.  创建一个 getter 来公开所选标签的列表。

1.  创建两个触发器方法：一个用于触发`changed`事件，另一个用于触发`tag-clicked`事件。

1.  创建两个`mutator`方法：`addTag`和`removeTag`。这些方法接收标签名称，如果不存在则添加标签，如果存在则删除标签。如果列表被修改，触发`changed`事件并调用重新渲染标签列表的方法。

1.  在 HTML 中，用新组件替换现有代码，并将新的脚本文件添加到其中。

1.  在`filter_and_search.js`中，删除`tagsToFilterBy`变量，并用新创建的组件中的新`mutator`方法和事件替换它。

#### 注意。

此活动的解决方案可在第 584 页找到。

## 总结

在本章中，我们通过学习基本接口、属性和方法来探索 DOM 规范。我们了解了你编写的 HTML 与浏览器从中生成的树之间的关系。我们查询了 DOM 并导航 DOM 树。我们学会了如何创建新元素，将它们添加到树中，并操作现有元素。最后，我们学会了如何使用 Shadow DOM 来创建隔离的 DOM 树和可以在 HTML 页面中轻松重用的自定义组件。

在下一章中，我们将转向后端世界。我们将开始学习有关 Node.js 及其基本概念。我们将学习如何使用**nvm**安装和管理多个 Node.js 版本，最后但同样重要的是，我们还将学习有关**npm**以及如何查找和使用外部模块。
