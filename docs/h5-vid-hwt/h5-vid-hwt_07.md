# 第七章：使用 JavaScript 进行交互

在本章中，我们将涵盖：

+   使用 JavaScript 播放音频文件

+   使用拖放 API 与文本

+   使用`vid.ly`和 jQuery 实现跨浏览器视频支持

+   使用 jQuery 动态显示视频

+   使用 jQuery 创建可移动的视频广告

+   使用`Easel.js`和`canvas`标签控制图像的显示

+   使用`Easel.js`和`canvas`标签来制作图像序列的动画

+   使用`canvas`标签和 JavaScript 进行随机动画播放音频

# 介绍

虽然 HTML5 可能会结束对 Flash 的使用，但它使 JavaScript 比以前更受欢迎。有许多库和插件可用于增强和扩展 HTML5 和 CSS3，以创建丰富的交互体验。

本章包含了一些示例，展示了 JavaScript 如何与 HTML5 标签一起使用，例如音频、视频和画布，以及 CSS3 选择器和元素。

# 使用 JavaScript 播放音频文件

HTML5 在互联网上使用音频文件方面引入了更多的灵活性。在这个示例中，我们将创建一个游戏来练习使用音频标签和 JavaScript 加载和播放声音。

## 准备工作

您需要一个音频文件来播放，一张图片和一个支持 HTML5 的现代浏览器。本章的示例文件可以从[`www.packtpub.com/support?nid=7940`](http://www.packtpub.com/support?nid=7940)下载。Free Sound Project ([`freesound.org`](http://freesound.org))有您可以使用的音频文件，只要给予制作人信用，照片可以在[`www.Morguefile.com`](http://www.Morguefile.com)找到，用于您的个人项目。

## 如何做...

现在我们准备创建一系列按钮和一个简短的 JavaScript 程序，当其中一个按钮被按下时，将播放一个随机的音频文件。

打开您的 HTML 编辑器并创建一个 HTML5 页面的开头部分。

```html
<!DOCTYPE html><html lang="en"><head><meta http-equiv="Content-Type" content="text/html; charset=utf-8"> <title>Playing a sound file with JavaScript</title>

```

因为我们只有几种样式，所以我们将它们添加到 HTML 页面的头部区域。

```html
<style>h1{font-family:"Comic Sans MS", cursive; font-size:large; font-weight:bold;}
button{ padding:5px;margin:5px;}
button.crosshair { cursor: crosshair; }
button.crosshairthree {margin-left:40px;
cursor:crosshair;} </style>

```

脚本需要创建三个变量。开头的脚本标签和变量应该看起来像以下代码块：

```html
<script>//variables
var mySounds=new Array();
mySounds[0]="boing";
mySounds[1]="impact";
mySounds[2]="squeak";
mySounds[3]="whack";
mySounds[4]="space";
var soundElements;
var soundChoice;

```

现在我们已经为脚本创建了全局变量，我们可以创建函数。键入`function whackmole(){`开始函数，然后在新行上键入`var i = Math.floor(Math.random() * 5)`;使用 JavaScript 数学库生成一个相对随机的数字。接下来，键入`soundChoice = mySounds[i]`;将数组值分配给`soundChoice`。使用`soundElements[soundChoice].play();}`关闭函数。您的函数代码目前应该看起来像以下内容：

```html
function whackmole() {
var i = Math.floor(Math.random() *5);
soundChoice = mySounds[i];
soundElements[soundChoice].play();}

```

键入`function init(){`开始函数。在新行上，键入`soundElements = document.getElementsByTagName("audio");} </script>`来完成我们的 JavaScript 代码块。它应该看起来像以下代码块：

```html
function init(){
soundElements = document.getElementsByTagName("audio");}
</script>

```

关闭头标签并键入 body 标签，添加一个`init()`函数调用，使其看起来像：

```html
</head><body onLoad="init();">

```

使用`<header>`标签为页面的页眉区域创建一个标题区域。使用标题标签`<h1>`显示页面的标题：

```html
<header><h1>Whack A Mole!</h1></header>

```

有五个按钮来创建一个平衡的外观，它们都被分配了一个类。

```html
<section> <p> <button class="crosshair" onclick="whackmole();"> <img src="img/downmole.png" width="37" height="24" alt="Mole peeping out of hole"></button>
<button class="crosshair" onclick="whackmole();"> <img src="img/downmole.png" width="37" height="24" alt="Mole peeping out of hole"></button></p>

```

第三个按钮的类名为`crosshairthree`，以便更好地控制其在屏幕上的位置。

```html
<p style="padding-left:30px;"><button class="crosshair" onclick="whackmole();"><img src="img/downmole.png" width="37" height="24" alt="Mole peeping out of hole"></button></p> <p><button class="crosshair" onclick="whackmole();"> <img src="img/downmole.png" width="37" height="24" alt="Mole peeping out of hole"></button><button class="crosshair" onclick="whackmole();"><img src="img/downmole.png" width="37" height="24" alt="Mole peeping out of hole"></button></p></section>

```

如果您正在使用本书的代码文件，那么音频文件标签应该类似于下面的代码块：

```html
<section><audio id ="boing" autobuffer>
<source src="img/cartoonboing.ogg" />
<source src="img/cartoonboing.mp3" /></audio>
<audio id ="impact" autobuffer>
<source src="img/cartoonimpact.ogg" />
<source src="img/cartoonimpact.mp3" /></audio>
<audio id ="squeak" autobuffer>
<source src="img/cartoonsqueak.ogg" />
<source src="img/cartoonsqueak.mp3" /></audio>
<audio id ="whack" autobuffer>
<source src="img/cartoonwhack.ogg" />
<source src="img/cartoonwhack.mp3" /></audio>
<audio id="space" autobuffer>
<source src="img/cartoonspaceboing.ogg" />
<source src="img/cartoonspaceboing.mp3" /></audio>

```

用关闭标签完成页面：

```html
</section></body></html>

```

将文件保存为`playing-audio-files-with-javascript.html`并在浏览器中查看。它应该看起来类似于以下屏幕截图：

![如何做...](img/1048_07_01.jpg)

## 它是如何工作的...

首先，我们创建了一个基本的 HTML5 页面的开头。然后，我们添加了 CSS 样式，为按钮添加背景图像，并在鼠标或指针设备移动到按钮上时将鼠标图标更改为十字准线。这给了我们一个视觉上的模拟目标武器，比默认的鼠标图标更有趣。

创建了三个变量供脚本使用：`mySounds, soundElements`和`soundch`。我们创建的第一个函数名为`whackmole()`包含一个内部变量`i`，它保存了一个随机生成的数字的结果。`Math.random()`导致生成一个伪随机数。然后我们将其乘以`5`，我们的音频文件数量，并使用`Math.floor()`的结果创建一个值范围从零到五的整数。然后将该值分配给临时变量`i`，然后用于使用随机生成的数组值填充变量`mySounds`。这个新的数组值存储在变量`soundChoice`中，`soundChoice = mySounds[i]`。这使我们能够在按下按钮时使用`soundElements[soundChoice].play()`触发`audio`标签的`play()`动作。

我们创建的第二个函数是`init()`，稍后我们将其与`body`标签绑定，使用`onLoad`，这样我们就可以使用`audio`标签及其数组值，通过`getElementsByTagName`获取音频文件，如`soundElements`变量中所包含的。

接下来，我们添加了`<body onLoad="init();">`标签，以及一系列包含我们可爱的鼹鼠图像的按钮到页面上。每个按钮都包含一个`onClick()`事件，调用了`whackmole()`函数。我们的第三个按钮与其他按钮的类不同，`crosshairthree`，它在按钮左侧添加了额外的边距，使其看起来更加居中。

### 注意

火狐目前有一个怪癖，如果你不先列出`.ogg`音频源，它就找不到。

最后，我们使用`<audio>`和`<source>`标签将声音文件添加到页面。使用源标签列出了每个文件的`ogg`和`mp3`格式。因为源标签被认为是其所包围的父音频标签的“子”标签，所以根据使用的浏览器，任何文件格式都会播放，因为不同的浏览器目前更喜欢不同的声音文件格式。

## 还有更多...

您可以看到，通过为不同的图像播放不同的声音文件，非常容易创建一个儿童的形状或动物朗读页面等应用程序。

### 使用 jQuery 控制音频剪辑的外观

jQuery 中的`.animate`函数打开了新的方式，当访问者采取行动或作为丰富媒体体验的一部分时，可以使音频控件出现、淡出和消失。以下是一个示例，演示了如何使音频控件淡出，然后迅速重新出现：

```html
<script> $(document).ready(function(){
$('audio').delay(500).hide('fade', {}, 1000 ).slideDown('fast'); }); </script>
<!- - the HTML -- ><audio id ="boing" autobuffer> <source src="img/cartoonboing.ogg" /> <source src="img/cartoonboing.mp3" /></audio>

```

我们将在本章的一个示例中使用视频文件执行类似的技巧。

## 另请参阅

第八章，“拥抱音频和视频”将涵盖更多关于音频标签及其使用方式的信息。

# 使用文本的拖放 API

虽然所有浏览器都可以原生地拖放图像或链接，但放置对象以前需要复杂的 JavaScript 或第三方库。拖放 API 旨在提供一种更简单、标准化的方式，使用户能够将任何类型的对象拖放到标识区域中。实际上，在各种浏览器中使用该 API 是一项挑战。目前主要支持此 API 的浏览器是 Firefox、Chrome 和 Safari。

## 准备工作

在[`www.packtpub.com/support?nid=7940`](http://www.packtpub.com/support?nid=7940)下载本教程的代码。本教程标题中使用的字体来自[`www.fontsquirrel.com`](http://www.fontsquirrel.com)，您也可以在该网站下载其他字体。本教程可能无法在 Internet Explorer 中使用。我们将创建一个井字棋游戏，演示拖放 API 的工作原理。

## 如何做...

打开您的 HTML 编辑器，首先创建一个基本的 HTML5 页面。我们将添加两个样式表链接，一个用于支持我们将加载到页面标题的`@fontface`字体，另一个是我们的主样式表。输入如下所示的代码，然后将文件保存为`using-drag-drop-api.html`。

```html
<!DOCTYPE html><html lang="en"> <head> <meta charset="utf-8"> <title>Using the drag-and-drop API element</title> <link rel="stylesheet" href="fonts/specimen_files/specimen_stylesheet.css" type="text/css" charset="utf-8" /> <link rel="stylesheet" href="stylesheet.css" type="text/css" charset="utf-8" />

```

让我们继续对页面进行样式设置。创建或打开名为`stylesheet.css`的 CSS 文件。将页面的整体`margin`设置为`100px`，默认颜色设置为`#666`。

```html
@charset "UTF-8";/* CSS Document */body { margin:100px; color:#666; }

```

页面的内容标签应该都设置为`display:block`，如下所示的代码：

```html
article, aside, figure, footer, header, hgroup, menu, nav, section { display:block; }

```

现在，我们要指定`@fontface`信息。代码和字体文件来自`www.fontsquirrel.com`字体包，已包含在本教程的代码文件中。

```html
@font-face { /* This declaration targets Internet Explorer */ font- family: '3DumbRegular';src: url('3dumb-webfont.eot');}@font-face {/* This declaration targets everything else */font-family: '3DumbRegular';src: url(//:) format('no404'), url('fonts/3dumb- webfont.woff') format('woff'), url('fonts/3dumb-webfont.ttf') format('truetype'), url('fonts/3dumb-webfont.svg#webfontlNpyKhxD') format('svg');font-weight: normal;font-style: normal;}

```

为`h1`标签添加颜色，并将`font-family`属性设置为`3DumbRegular`，这是我们字体的名称。

```html
h1{color:#C60;font-family: '3DumbRegular';}

```

创建一个名为`gametilebox`的新 div，用于容纳组成游戏瓷砖的字母。将盒子的`float`属性设置为`left`，宽度和高度设置为`280px`。根据以下代码片段设置`padding, margin-right, border`和`background-color`。

```html
#gametilebox{ float:left;width:280px; height:280px; padding:10px; margin-right:30px; border:1px solid #000; background-color:#ccc; }

```

游戏板将共享许多与瓷砖框相同的属性，因此复制`gametilebox`的样式，然后粘贴并命名为"gameboard"。添加一个`background-image`属性，url 为`images/tictactoegrid.jpg`，并将`background-color`设置为`aa`。

`gameboard div`应该如下所示代码：

```html
#gameboard { float:left; width:280px; height:280px; padding:10px; margin-right:30px;border:1px solid #000; background-image:url(images/tictactoegrid.jpg); background-color:#aaa;}

```

让我们对`div`块进行样式设置，用于放置我们的字母。所有`block` div 的`float`应设置为`left`。`width`不应大于`85px`，`height`不应大于`80px`。它们将位于 3x3 的网格上，因此第二行和第三行的第一个块也需要具有`clear:both`属性。第二行和第三行的第三个块应该具有较低或没有`padding`和`margin-right`属性。因为有九个，所以这里只显示了一个块代码的示例：

```html
#blockA {float:left; width:75px; height:75px; padding:5px 5px 5px 2px; margin-right:10px; border:none; background-color:red;}
#blockB {float:left; width:75px; height:75px; padding:5px; margin-right:10px; border:none; background-color:blue;}

```

现在，我们将为字母游戏瓷砖设置样式。在样式表中创建一个名为`lettertile`的新类，然后按照这里显示的属性设置类的属性：

```html
.lettertile { width:60px; height:60px; padding:5px; margin:5px; text-align:center; font-weight:bold;font-size:36px;color:#930; background-color:transparent;display:inline-block;}

```

我们将添加的最后一个样式是`draggable`属性。创建下面的样式以帮助跨浏览器兼容性：

```html
*[draggable=true] { -moz-user-select:none; -khtml-user-drag: element; cursor: move;}

```

样式表已经完成，现在我们可以开始编写脚本来拖动字母瓷砖并放置它们。

打开之前创建的 html 页面`using-drag-drop-api.html`，并为 IE 浏览器输入以下代码：

```html
<!--[if IE]><script src="img/html5.js"> </script><![endif]-->

```

在样式表链接的下方直接添加一个开头的`<script>`标签，并输入第一个函数`dragDefine(ev)`，该函数接受一个事件参数，并在后面加上`{`。在大括号后面，输入`ev.dataTransfer.effectAllowed ='move'`；然后，在新的一行上，输入`ev.dataTransfer.setData("text/plain", ev.target.getAttribute('id'))`；以设置数据类型和目标属性。最后，输入`return true`；并加上一个闭合的`}`来完成函数。

```html
function dragDefine(ev) {ev.dataTransfer.effectAllowed = 'move'; ev.dataTransfer.setData("text/plain", ev.target.getAttribute('id')); return true;}

```

现在，我们需要定义`dragOver`函数。输入`dragOver(ev)`和一个开头的`{`，然后通过添加`ev.preventDefault()`来调用`preventDefault()`函数。函数块应该类似于下面的代码：

```html
function dragOver(ev) { ev.preventDefault();}

```

我们需要的下一个函数是指示拖动完成的函数。输入`function dragEnd(ev)`，然后一个开头的`{`。输入`return true; }`来完成函数。

输入`function dragDrop(ev)`并加上一个开头的`{`，然后换行添加我们的第一个方法。输入`var idDrag = ev.dataTransfer.getData("Text")`来创建一个将保存文本字符串的拖动变量，然后输入`ev.target.appendChild (document.getElementById(idDrag))`。完成函数后，输入`ev.preventDefault()`。函数块应该如下所示代码：

```html
function dragDrop(ev) {
var idDrag = ev.dataTransfer.getData("Text");
ev.target.appendChild(document.getElementById(idDrag));
ev.preventDefault();} </script>

```

关闭页面的头部部分。输入`<body><header>`，然后`<h1>拖放井字棋</h1></header>`来完成页面的标题。

```html
</head><body><header><h1>Drag and Drop Tic Tac Toe</h1></header>

```

接下来，输入`<section><h3>将字母从灰色框拖到游戏板上（然后再拖回去！）</h3>`。

创建一个 ID 为`"gametilebox"`的 div，并设置`ondragover ="dragOver(event)"`和`ondrop="dragDrop(event)"`。它应该如下所示：

```html
<div id="gametilebox" ondragover="dragOver(event)" ondrop="dragDrop(event)">

```

现在，我们将为每个游戏瓷砖创建一个`div`。创建六个**"X"**瓷砖和六个**"O"**瓷砖，每个都以从`1-12`的数字结尾的`id`开始。每个`div`将包含类`"lettertile"`，每个`draggable`属性将包含值`"true"`。每个瓷砖还将包含`ondragstart="return dragDefine(event)"`和`ondragend="dragEnd(event)"`。`div`块应该看起来像以下代码：

```html
<div id="lettertile1" class="lettertile" draggable="true" ondragstart="return dragDefine(event)" ondragend="dragEnd(event)">X</div>
<div id="lettertile2" class="lettertile" draggable="true" ondragstart="return dragDefine(event)" ondragend="dragEnd(event)">X</div>
<div id="lettertile3" class="lettertile" draggable="true" ondragstart="return dragDefine(event)" ondragend="dragEnd(event)">X</div>

```

现在，我们可以为我们在**stylesheet.css**中创建的块样式创建实际的`divs`。首先键入`<div id= "gameboard">`。应该为每个块 id 创建一个`div`，从"blockA"到"blockI"。它们每个都将包含一个`ondragover="return dragOver(event)"`和一个`ondrop="dragDrop(event)"`。它们应该看起来像以下代码块。

```html
<div id="blockA" ondragover="return dragOver(event)" ondrop="dragDrop(event)"></div>
<div id="blockB" ondragover="return dragOver(event)" ondrop="dragDrop(event)"></div>
<div id="blockC" ondragover="return dragOver(event)" ondrop="dragDrop(event)"></div>

```

用`body`和`html`的闭合标签关闭页面，将文件命名为`"using-drag-drop-api.html"`，然后在浏览器窗口中查看结果。拖动几个字母，结果应该类似于以下截图：

![如何使用拖放 API，使用文本](img/1048_07_02.jpg)

## 它是如何工作的...

首先，我们创建了一个基本的 HTML5 页面，并使用`@fontface`添加了一个草图字体作为标题，以使我们的游戏具有有趣的视觉效果。接下来，我们通过将`margin`设置为`body`和所有块级元素的`display:block`来为页面设置样式，以便更好地控制这些元素的呈现。在样式化标题字体后，我们为游戏瓷砖框定义了`width`和`height`。这将是容纳组成游戏瓷砖的字母的容器。

我们通过在 IE 浏览器中键入一个特殊的注释标签来开始我们的脚本，以指向一个额外的脚本文件来触发 HTML5 元素：`<!--[if IE]><script src="img/html5.js"></script><![endif]-->`。这是由 Remy Sharp（http://remysharp.com/html5-enabling-script/）根据 MIT 许可证提供的，以在处理 Internet Explorer 时保持我们的理智。

当用户开始拖动物品时，`dragDefine()`函数被调用。它首先检查物品是否可拖动，使用`dataTransfer.effectAllowed='move'`。然后，它将要传输的数据类型设置为`text`，并使用`dataTransfer.setData("text/plain")`和`target.getAttribute('id'))`来识别目标的`id`。该函数返回 true，表示可以拖动对象。

接下来，我们定义了`dragOver`函数，当拖动的物品位于另一个物品上时调用，接受一个名为`ev`的事件参数，然后使用它来调用`preventDefault()`以允许放置物品。拖放 API 规范明确规定，我们必须取消拖动，然后准备放置。

然后创建了`dragEnd()`函数，当拖动完成时返回 true。它还接受一个事件参数。

完成所有拖动功能后，我们准备创建代码来放置物品。`dragDrop()`函数接受一个事件参数，并使用该值获取文本对象的值，然后将其传递给一个新变量`var idDrag`来保存文本字符串，然后再使用`getElementById`来识别正确的元素 ID 进行放置。就像`dragEnd()`一样，我们必须调用拖放 API 中的`preventDefault()`函数来指示可以放置对象。

在关闭页面的头部区域后，在 body 中放置了内容框来容纳我们的字母瓷砖和游戏板块。这些由两个父 div 容器组成，每个容器都包含包含字母瓷砖或游戏板网格部分的子 div。

游戏瓷砖框在拖动字母瓷砖时调用了`dragOver()`函数。字母瓷砖 div 本身通过`draggable="true"`变得可拖动，并在拖动时返回`dragDefine()`。当拖动停止时，它们调用`dragEnd()`函数。

因为我们希望字母块能够被放下并停留在游戏板的特定区域，我们为网格上的每个单独的块创建了 div，以便在它们被放到板上时保持我们的字母在那里，并在对象被拖动到它们上方时返回`dragOver`事件，并在对象被放到它们上时调用`dragDrop()`。

为什么要费心设置块 div？我们本可以在左边设置我们的游戏块盒子，右边设置游戏板，然后就完成了。结果会是，当我们从左边的盒子拖动块到游戏板时，它们会被放下并按照它们被放下的顺序排列，而不是我们想要放置它们的地方。这种默认行为在你想要对列表进行排序时很好，但当需要精确控制对象放置位置时就不行了。

我们需要覆盖当对象被放下时产生的默认行为。我们创建了九个游戏板块，都是相同的基本大小。每个块的主要变化是`padding`和`margin`。

花几分钟时间阅读[`www.whatwg.org/specs/web-apps/current-work/multipage/dnd.html`](http://www.whatwg.org/specs/web-apps/current-work/multipage/dnd.html)上的拖放规范，你会注意到他们明确表示他们只定义了一个拖放机制，而不是你必须执行的操作。为什么？因为使用智能手机或其他触摸屏设备的用户可能没有鼠标等指针设备。

## 还有更多...

这个拖放 API 的演示可以通过多种方式构建成一个完整的游戏，包括计分、游戏板重置按钮和其他交互元素。

### 创建一个基于画布的井字棋游戏

可以使用两个画布，一个用于游戏块盒子，另一个用于游戏板。可以使用画布动态绘制板和游戏块，然后在屏幕上写入分数或消息，比如“你赢了”。

### 在用户玩游戏时显示响应消息

Remy Sharp 在[`html5demos.com/drag-anything`](http://html5demos.com/drag-anything)上有一个很棒的演示，展示了当一个对象被放下时如何在屏幕上显示消息。

要被放下的对象的源标签类似于：

```html
<div id="draggables"><img src="img/picean.png" alt="Fish" data-science-fact="Fish are aquatic vertebrates (animals with backbones) with fins for appendages." /> </div>

```

当对象被拖动到时，“放置区”框可能如下所示：

```html
<div class="drop" id="dropnames" data-accept="science-fact"> <p>Learn a science fact!</p> </div>

```

当图像被放入框中时，你会看到包含在“data-science-fact”中的文本，而不是图像。

## 另请参见

jQuery 的 Packt 书籍，本书中的其他配方，以及高级 HTML5 的 Packt 书籍。

# 使用 vid.ly 和 jQuery 支持跨浏览器视频

支持大多数浏览器需要将视频编码为多种格式，然后将正确的格式提供给浏览器。在这个配方中，我们将使用一个名为 vid.ly 的在线视频显示库([`www.vid.ly`](http://www.vid.ly))，在多个浏览器上可靠地准备和分享视频，并在页面上改变背景颜色。

## 准备工作

你需要一个视频上传到[`www.vid.ly`](http://www.vid.ly)。一些浏览器不允许本地提供文件，所以你可能也需要一个地方来上传你的文件和测试页面。

## 如何做...

输入`<!DOCTYPE html> <html lang="en"> <head>`，然后开始添加样式声明，输入`<style type="text/css"> h2{color:#303;}`。

样式一个 div 来包含特色内容：`#featured {position:relative; padding: 40px; width: 480px; background-color:#000000; outline: #333 solid 10px; }`。

输入`video {padding: 3px;background-color:black;}`来创建视频标签的样式，然后添加一个闭合的`</style>`标签。

在页面中声明使用的脚本。键入`<script src="img/jquery.min.js" type="text/javascript" charset="utf-8"></script>`来引用主要 jQuery 库的最小化版本。然后，键入`<script type="text/javascript" src="img/jquery-ui.min.js"></script>`来引用用于颜色变化效果的 jQuery UI 库。最后，我们将通过在关闭`</head>`标签之前键入`<script type="text/javascript" src="img/mycolor.js"></script>`来引用我们自己的脚本。

输入一个开放的`<body>`和`<section>`标签，然后键入`<header> <h2>Featured Video</h2></header>`来显示页面标题。

现在，我们可以创建包含我们之前样式化的特色内容的 div。键入`<div id="featured"> <p>此视频已通过<a href="http://vid.ly">vid.ly</a>转换为跨浏览器格式</p>`。

下一步是将视频剪辑上传到[`vid.ly`](http://vid.ly)进行多文件格式转换。转换过程完成后，您将收到一封电子邮件，然后可以获取视频的代码片段，如下截图所示：

![如何做...](img/1048_07_03.jpg)

复制网站上的代码，然后粘贴到您的页面中。视频和脚本标签中的`src`值应该是 vid.ly 给出的 URL。代码块应该如下所示：

```html
<video id= "vidly-video" controls="controls" width="480" height="360"> <source src="img/7m5x7w?content=video"/> <script id="vidjs" language="javascript" src="img/html5.js"></script> </video>

```

为了增加一些额外的乐趣，让我们在页面上添加另一个视频标签。键入以下代码：`<p>哎呀，这是一个宝宝视频！</p>`，为视频标签使用不同的 id，并按照以下方式调整大小：`<video id="tinymovie1" controls="controls" width="190" height="120">`，然后使用相同的源标签：`<source src="img/7m5x7w?content=video"/><script id="vidjs" language="javascript" src="img/html5.js"></script></video>`，并关闭页面：`</div> </section></body></html>`。将文件保存为`display-videos-using-videly.html`。

我们要做的最后一件事是创建一个 jQuery 脚本来改变`#featured` div 的背景颜色。打开您的编辑器，创建一个名为`myColor.js`的新文件。

键入`$(document).ready(function() {`，然后转到新的一行，键入调用动画函数并改变背景颜色的代码：`$('#featured').animate({'backgroundColor':'#ff3333', 'color': '#ffffff'}, 6000);})`;。

在浏览器中加载页面，观察主视频加载时颜色的变化。您可以看到以下截图显示的效果：

![如何做...](img/1048_07_04.jpg)

## 工作原理...

首先，我们创建了一个标准的 HTML5 页面，并开始添加样式声明。我们将`featured` div 的位置设置为相对位置，以便在将来如果决定添加额外的 jQuery 效果时具有更大的灵活性。通过将`padding`设置为`40px`，将`outline`颜色设置为深灰色并加粗为`10px`，创建了强烈的视觉效果。默认的背景颜色设置为黑色`(#000000)`，以便与最终的红色背景进行高对比度的比较。

接下来，我们样式化了`video`标签，使其在加载时具有黑色的`background-color`。我们还可以在这里添加一个背景图像作为海报。

接下来，使用`<script src="img/jquery.min.js" type="text/javascript" charset="utf-8"></script>`声明了基本的 jQuery 脚本。因为它不包含`animate()`等效果，我们还需要引用用于颜色变化效果的 jQuery UI 库的最小化版本。然后，通过键入`<script type="text/javascript" src="img/mycolor.js"></script>`来添加对我们自己脚本的引用。进一步减小脚本文件大小的另一种方法是创建一个仅包含来自 jQueryUI 库的动画效果的自定义脚本。

接下来，我们创建了主页面内容，包括指向 vid.ly 上的视频的链接。vid.ly 提供的默认代码会将 ID`'vidley video'`应用到`video`标签，但如果您想使用自己的样式 ID 或将为每个视频使用不同的 ID，则可以省略该代码。另一个选择是将所有视频分配相同的类，然后根据需要为它们分配唯一的 ID。

## 另请参阅

第八章，*拥抱音频和视频*，更详细地介绍了视频元素。

# 使用 jQuery 动态显示视频

视频元素使我们能够像处理图像一样处理视频，并以有趣和令人兴奋的方式操纵它们。

## 准备工作

您需要一个以多种文件格式提供的视频（这些文件在本书的章节代码中提供）。建议上传文件的服务器，因为并非所有浏览器都以可预测的方式本地播放文件。

## 操作步骤

首先，我们必须准备一个 HTML5 页面来放置它。输入我们页面的开放标签：`<!DOCTYPE html> <html lang="en"> <head> <meta charset="utf-8" /> <title>Video Explosion</title>`。

打开下载的代码文件中的`stylesheet.css`文件，或创建一个同名的新文件。

输入以下内容来设置 body 样式：`body {background: white;color:#333333; }`，然后按照以下方式设置 div 标签的样式：`div {float:left; border:1px solid #444444;padding:5px;margin:5px; background:#999999;}`。

我们需要创建和设置的第一个独特的 div 是`#featured`。输入`#featured {position:relative; width: 480px; background-color:#f2f1f1;}`来创建样式。

现在创建一个名为`details`的 div 来容纳一个小的信息框。输入`#details{ position:relative;display:block;background-color:#6CF;color:#333333; padding:10px;}`以创建一个将显示在`featured` div 旁边的 div。

保存`css`文件，并在 html 页面的头部引用它，方法是使用链接标签输入`<link rel="stylesheet" href="css/stylesheet.css"type="text/css" media="screen" charset="utf-8"/>`。

在样式表链接下方输入以下链接到主 jQuery 库：`<script src="img/jquery-latest.js" type="text/javascript" charset="utf-8"></script>`，然后通过输入`<script type="text/javascript" src="img/jquery-ui.min.js"></script>`来链接到 jQuery UI 库。最后，通过输入`<script type="text/javascript" src="img/explode.js"></script>`来添加对即将创建的脚本的引用。

创建一个新文件并命名为`explode.js`，并将其存储在一个名为`js`的新子文件夹中。输入`$(document).ready(function(){}`。在两个大括号（{}）之间输入`$('h1').effect('shake', {times:5}, 200)`；创建将导致 featured div 标签中包含的内容爆炸的语句。在新行上，输入`$('#featured').effect('shake', {times:3}, 100).delay(500).hide('explode',{}, 2000).slideDown('fast');)`；以完成脚本。您的代码块应该类似于以下代码块：

```html
$(document).ready(function(){ $('h1').effect('shake', {times:5}, 200); $('#featured').delay(2000).hide('explode', {}, 2000 ).slideDown('fast'); });

```

保存文件并返回 html 页面。

为 HTML 文件添加`</head>`闭合标签和`<body>`开放标签。接下来，输入一个开放的`<header>`标签和标题文本：`<h1>Featured Moto Video</h1>`，然后输入`</header>`标签来完成标题区域。

创建一个开放的`<section>`标签，然后输入`<div id="featured">`来创建一个 div，用于容纳我们的视频标签和相关元素。输入`<video id="movie" width="480" height="360" preload controls>`，然后为三种视频文件类型的每一个添加一个 source 标签：`<source src='motogoggles.ogv' type='video/ogg; codecs="theora, vorbis"'/> <source src='motogoggles.mp4' type='video/mp4; codecs="avc1.42E01E, mp4a.40.2"'/> <source src='motogoggles.webm' type='video/webm; codecs="vp8, vorbis"'/>`，然后使用`</video>`标签和`</div>`标签来关闭 featured div。

最终的内容块包含在`details` div 中。要创建它，输入`<div id="details">`，然后添加一个带有文本的标题标签`<h1>Details</h1>`，最后是一个简短的解释性文本段落：`<p>视频将爆炸然后再次出现！</p>`。关闭`</div></section> </body></html>`标签。将 HTML 文件保存为`exploding-video-dynamically.html`，在浏览器中打开它以查看结果。它们应该与以下截图类似，显示视频分成几部分并爆炸。

![如何操作...视频操作，使用 jQuery](img/1048_07_05.jpg)

## 它是如何工作的...

`stylesheet.css`文件包含了特色 div 的样式，确定了页面上视频对象的位置。要注意的第一件重要的事情是`position`设置为`relative`。这使我们能够移动视频对象并使用 jQuery 执行其他操作。

我们创建了一个名为`details`的 div，其`position`也是`relative`，但`background-color`设置为`浅蓝色(#6CF)`。不同的颜色将有助于在视觉上将其与视频对象区分开来。

接下来，我们添加了 jQuery 库脚本。为了让我们能够访问`animate`类中包含的方法和函数，需要引用 jQuery UI 库。在这个例子中，我们是在本地引用它，但您也可以像我们访问主要的 jQuery 库一样链接到它。

最后，我们能够编写自己的脚本，使页面上的元素摇晃和爆炸！我们创建了一个语句来验证页面是否准备好接受我们的代码，方法是键入`$(document).ready(function(){}`。这个函数查询 DOM 并询问页面是否已加载并准备好接受脚本。在创建 jQuery 脚本时，使用这个包装函数是最佳实践。我们使用别名符号`$`来调用 jQuery 函数，以抓取`h1`选择器，并对其应用包含`shake`参数的`effect`动作，使元素向侧面移动，其中又包含了一个关于摇动元素的次数的参数。摇动应持续的时间间隔以毫秒定义，本例中为`200`。我们使用选择器`$('#featured')`来抓取特色 div 元素，并像对`h1`标签所做的那样，对其进行摇动（只摇动三次以增加变化），每次摇动持续`100`毫秒。现在我们添加了一些新的动作。在`shakes`和爆炸之间添加了`500`毫秒的`delay`，并使用`.delay(500)`命令附加到该命令。然后我们附加了`hide`动作，参数为`explode`，默认情况下将发生一次，持续时间为`2000`毫秒。视频爆炸后，`slidedown`动作将其以`fast`参数滑回屏幕上。请注意，爆炸所用的时间有点长，这样我们可以更容易地看到它。`100-500`毫秒的时间间隔会产生更真实的爆炸效果。如果您只想要视频本身而不是特色标签提供的背景或边框，也可以直接使用`$('video')`来抓取视频标签。

回到 HTML 文件，我们将视频放在一个名为`featured`的容器 div 中，并创建了一个父`video`标签，它将`preload`并包含默认的`controls`。在关闭`video`标签之前，我们在其中嵌套了三种视频文件类型的`source`标签，以便不同浏览器的用户都可以观看视频：我们没有提供 FLASH 回退，但我们可以使用 JavaScript 库，比如`Video.js`。然后我们关闭了`</video>`标签和特色 div 标签`</div>`。

最后，我们创建了一个 div 来保存关于用户可以期待在`details` div 中发生的事情的信息。

## 还有更多...

视频元素、JavaScript 和画布标签还有很多可以做的事情。继续阅读更多实验。

### 使用视频和画布进行更多交互式爆炸

Sean Christmann 在[`www.craftymind.com`](http://www.craftymind.com)上进行了一项令人惊叹的实验，使您能够在视频播放时实时爆炸多个视频部分。您可以在此处查看：[`www.craftymind.com/2010/04/20/blowing-up-html5-video-and-mapping-it-into-3d-space/`](http://www.craftymind.com/2010/04/20/blowing-up-html5-video-and-mapping-it-into-3d-space/)，但请注意 - 在 Firefox 中资源消耗非常大。

### 所有这些爆炸是怎么回事？

起初似乎没有任何真正实际的理由来分解视频。然而，这对于模仿独特的过渡效果或对游戏中用户操作的响应可能非常有用。

### 实时 Chroma 键背景替换

Firefox 开发人员一直在尝试操纵视频元素。他们创建了一个教程，解释了他们如何使用画布、JavaScript 和视频元素的属性执行 Chroma 键替换。您可以在以下网址阅读有关此内容并查看演示：[`developer.mozilla.org/En/Manipulating_video_using_canvas`](http://https://developer.mozilla.org/En/Manipulating_video_using_canvas)。

想象一下在网站上显示视频，您可以显示异国情调的背景或创建产品和人物的互动混搭。

## 另请参阅

视频元素在本书的第八章*拥抱音频和视频*中进行了深入探讨。

# 使用 jQuery 创建可移动视频广告

我们将在网站上创建一个视频广告，当用户向下滚动页面时，它将移动使用 jQuery 和视频标签。

## 准备工作

您将需要多种格式的视频文件，如`.ogg/.ogv, .mp4`和`.webm`，或者使用视频服务，如[`www.vid.ly.com`](http://www.vid.ly.com)来提供跨浏览器视频。此示例未在 Internet Explorer 中进行测试，但应在 Safari、Google Chrome、Opera 和 Firefox 的最新版本中正常工作。

## 如何做...

我们将首先创建一个典型的网页。在编辑器中打开一个新文件，并将其保存为`movable-video-ad.html`。键入`<!DOCTYPE html> <html lang="en"><head><meta charset="utf-8" /><title>Movable Video Ad</title>`以在页面上放置第一个标签。

现在，为我们的默认样式表创建一个引用链接`<link rel="stylesheet" href="css/main.css" type="text/css" media="screen" charset="utf-8" />`和一个名为`<link rel="stylesheet" href="css/scroll.css" type="text/css" media="screen" charset="utf-8" />`的辅助样式表。

接下来，为 jQuery 脚本创建引用链接。键入`<script src="img/jquery-1.4.min.js" type="text/javascript" charset="utf-8"></script>`来引用核心 jQuery 代码。添加链接语句`<script type="text/javascript" src="img/jquery-ui-1.7.2.custom.min.js"></script>`。我们将链接到的最终脚本是我们为名为`myAd.js`的配方创建的自己的脚本，它将存储在我们创建的名为"js"的子文件夹中。键入`<script type="text/javascript" src="img/myAd.js"></script>`来链接到该文件。

键入`</head><body><div id="container">`开始页面的内容区域。通过键入`<header> <h1>Motocross Mania</h1></header>`来显示页面标题。

通过键入`<div id="content"> <h2>No dirt = no fun</h2>`来开始添加页面内容。现在可以通过输入文本`<div id="motoad"><h3>Buy this movie!</h3>`，然后在段落元素标签中包含电影标题`<p><strong>MotoHelmet</strong></p>`来向页面添加包含广告的 div。

然后应添加一个视频标签`<video width="190" height="143" preload controls>`。键入包含每种视频格式的源标签，如下面的代码块所示：

```html
<source src='video/motohelmet.ogv' type='video/ogg; codecs="theora, vorbis"'/> <source src='video/motohelmet.mp4' type='video/mp4; codecs="avc1.42E01E, mp4a.40.2"'/> <source src='video/motohelmet.webm' type='video/webm; codecs="vp8, vorbis"'/></video>

```

关闭`</div>`标签并保存到目前为止的进度。

创建一个带有 id 为 intro 的段落`<p id="intro">`来包含文本`We review the best motorcross gear ever!!!`。在段落标签和文本后面，加上一个虚拟链接列表：`<ul><li><a href="#">Helmets</a></li> <li><a href="#">Gloves</a></li><li><a href="#">Goggles</a></li></ul>`，用`</p>`关闭段落，然后创建一个新的 div 来包含一个虚拟新闻内容块，然后是另外两个虚拟 div 块，一个页脚标签，以及关闭页面元素，如下面的代码块所示：

```html
<div id="news"><h2>Latest News</h2> <p>Trip Ousplat admits he doesn't do his own stunts! "My mom makes
me use a stunt double sometimes," The shy trick-riding sensation explains.</p> <p>Gloria Camshaft smokes the competition for a suprise win at the Hidden Beverage Playoffs</p> <p>Supercross competitors report more injuries; jumps more extreme than ever</p><p>James Steward still polite, reporters bored</p>
</div><div id="filler"><h2>On Location</h2> <p>Grass is not greener as there is no grass on most motorcross trails experts claim </p></div> <p id="disclaimer">Disclaimer! Anything you choose to do is at your own risk. Got it? Good.</p><footer><p>&copy; Copyright 2011 Motocross Extreme Publications, Inc.</p></footer></div></body></html>

```

现在，我们将在`main.css`文件中为页面元素设置样式。第一个关键样式是`#container` div。它应该有一个`0 auto`的边距和`650px`的宽度。接下来，`#motoad` div 应该被设置为`右浮动`，并包含一个`200px`的宽度来容纳视频元素。最后，`#intro` div 应该包含一个较短的`450px`宽度。这三个样式应该看起来类似于下面显示的代码块：

```html
#container{ margin:0 auto;text-align:left; width: 650px;}
#motoad{ float:right;width:200px;}
#intro{width:450px;}

```

其余的样式是对填充和颜色或其他标准声明的微小调整。

现在，打开`scroll.css`文件来定义样式，以帮助我们的广告滚动。我们将级联`#motoad`的属性，形成一个可以移动的 div 块。接下来，定义`#content`属性的`height`，以及段落和`h2`元素的宽度。`scroll.css`中的样式现在应该如下所示：

```html
#motoad {display:block;position: relative; background-color:#FC0;width:200px;padding:10px;}
#content { height:1000px;}
p {width:450px;}h2 {width:460px;}

```

保存文件，并准备创建我们的 jQuery 脚本。

打开或创建`myAd.js`，并开始输入文档就绪函数`$(document).ready(function(){}`和花括号。在花括号之间点击 enter，并输入滚动函数`$(window).scroll(function() {`。在该函数的开头花括号后键入命令：`$('#motoad').stop().animate({top: $(document).scrollTop()},'slow','easeOutBack')`;。也要关闭脚本：" });});"。我们的 jQuery 脚本现在应该看起来像以下代码块：

```html
$(document).ready(function(){ $(window).scroll(function() { $('#motoad').stop().animate({top: $(document).scrollTop()},'slow','easeOutBack'); }); });

```

保存所有文件，并在浏览器窗口中加载 HTML 页面。在开始滚动页面之前，页面应该看起来像下面的截图。

![如何做...](img/1048_07_06.jpg)

尝试向上和向下滚动页面。广告也应该随着页面一起上下移动。结果应该看起来类似于以下截图：

![如何做...](img/1048_07_07.jpg)

## 工作原理...

在创建了一个包含不同内容元素的典型 HTML 页面后，我们准备好为 CSS 页面设置样式。我们将 CSS 分成两个文件，`main.css`和`scroll.css`，这样当我们在 jQuery 脚本中调用滚动函数并积极应用它时，页面上的内容元素会收缩，以便我们的广告可以轻松移动，而不会阻挡页面上的任何信息。

我们希望在调用窗口滚动事件时，使`#motoad` div 标签移动。为此，我们使用别名符号`$`来调用 jQuery 函数，从 DOM 中获取`window`选择器，并将默认滚动动作参数应用于它。使用这个函数，我们然后创建了控制`#motoad` div 块行为的命令。我们给它了`stop`的动作，这样它就准备好进行动画。`animate`动作链接到了`stop`命令。我们应用到`#motoad` div 的`animate`的第一个参数是，当文档窗口中的滚动条移动时，div 会移动。`slow`参数控制了广告上下移动的速度，`easeOutBack`参数引用了一个缓动命令，以创建流畅的动画运动，而不是突然开始或停止。

## 还有更多...

在这个示例中，我们通过使自定义 HTML 元素对页面上的用户操作做出响应来实现了动画效果。这只是我们可以微妙地添加效果的一种方式，可以用于实际解决方案。

### 有 HTML 元素，就会移动

探索 jQuery UI 库，你会被许多可以操纵和样式化任何 HTML 元素的方式所启发。访问[`jqueryui.com`](http://jqueryui.com)查看演示和文档。

## 另请参阅

学习 jQuery：使用简单的 JavaScript 技术实现更好的交互设计和 Web 开发，Packt Publishing 出版。

# 使用 Easel.js 和 canvas 标签控制图像的显示

JavaScript 库`Easel.js`减少了使用`canvas`标签创建动画和丰富交互环境的复杂性。在这个配方中，我们将使用单个文件中名为"sprites"的一系列图像，以展示如何使用`Easel.js`来控制精灵中选择性显示的图形图像。

## 准备工作

您需要下载`Easel.js`库，或者使用此配方的代码文件中的副本。

## 如何做...

创建一个 HTML5 文件的开放标签。您的代码应该类似于以下代码块：

```html
<!DOCTYPE HTML><html><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8"><title> Animating images using BitmapSequence and SpriteSheet</title>

```

接下来，链接到此配方中使用的主样式表`styles.css`：<link href="styles.css" rel="stylesheet" type="text/css" />。

接下来，我们将通过插入以下脚本文件的链接来导入`Easel.js`框架库：`UID.js, SpriteSheetUtils.js, SpriteSheet.js, DisplayObject.js, Container.js, Stage.js, BitmapSequence.js`和`Ticks.js`。您可以在这里看到每个脚本文件的路径和链接：

```html
<script src="img/UID.js"></script><script src="img/SpriteSheetUtils.js"></script><script src="img/SpriteSheet.js"></script><script src="img/DisplayObject.js"></script><script src="img/Container.js"></script><script src="img/Stage.js"></script><script src="img/BitmapSequence.js"></script><script src="img/Tick.js"></script>

```

接下来，创建并打开`<script>`标签，并声明以下三个变量：`var canvas; var stage; var critterSheet = new Image()`; 用于我们的脚本。

输入`function init(){`开始函数，然后输入`canvas = document.getElementById("testCanvas")`;将页面主体中的 canvas 与 canvas 变量绑定。准备加载一个新的`spriteSheet`，输入`critterSheet.onload = handleImageLoad`;。`critterSheet`变量存储精灵图像的来源。输入`critterSheet.src = "images/moles.png"`;来加载我们自己的一系列鼹鼠图像。函数块应如下代码块所示：

```html
function init() {
canvas = document.getElementById("testCanvas");
critterSheet.onload = handleImageLoad;
critterSheet.src = "images/moles.png";}

```

我们将创建的第二个函数是`handleImageLoad()`。输入`function handleImageLoad() {`然后输入`stage = new Stage(canvas)`;来创建一个新的舞台实例。输入`var spriteSheet = new SpriteSheet(critterSheet, 76, 80);`来创建一个新的`spriteSheet`。创建一个名为`critter1`的新位图序列变量，并定义它在舞台上的位置，使用 x 和 y 坐标来输入：`var critter1 = new BitmapSequence(spriteSheet); critter1.y = 85; critter1.x = 85`;。通过输入`critter1.gotoAndStop(1)`，从我们的精灵表`moles.png`中添加一个小动物。接下来，使用命令`stage.addChild(critter1)`将其添加到舞台上。

克隆我们创建的第一个`critter1`变量，并通过输入`var critter2 = critter1.clone()`将其值传递给一个新的 critter 变量。通过添加到其当前位置值来将新变量定位在第一个 critter 的右侧，使用`critter2.x += 120`。

输入`critter2.gotoAndStop(0)`来为`critter2`变量赋值。克隆 critter 1 和 critter 2 的代码块应如下所示：

```html
var critter2 = critter1.clone();
critter2.x += 120;
critter2.gotoAndStop(0);
stage.addChild(critter2);

```

Tick 间隔`Tick.setInterval(300)`;和监听器`Tick.addListener(stage)`;是我们将添加到脚本的最后两个语句。关闭`handleImageLoad()`函数的大括号（}），然后输入一个闭合的脚本标签。

关闭`</head>`标签，然后输入带有`onload`属性的开放`body`标签，调用`init()`函数。创建一个名为"description"的 div 用于内容。添加一个名为`canvasHolder`的 div 来包含 canvas 元素。在页面底部显示图像文件`moles.png`。

```html
<body onload="init();">
<div class="description">Using <strong>BitmapSequence</strong> to animate images from a <strong>SpriteSheet</strong>.
</div>
<div class="canvasHolder">
<canvas id="testCanvas" width="980" height="280" style="background-color:#096"></canvas> </div> </p><p>The original moles.png spritesheet file with all the images:<br/><img src="img/moles.png"/></p> </body></html>

```

将文件保存为`whack-mole-easel-test-single.html`。结果可以在以下截图中看到：

![如何做...](img/1048_07_08.jpg)

## 工作原理...

在我们设置好 HTML5 页面的开头之后，我们准备好导入`Easel.js`框架并创建我们的主要脚本。

我们创建了一个开放的`<script>`标签，并声明了以下全局变量：`var canvas; var stage; var critterSheet = new Image()`; 用于我们的脚本。

创建的`init()`函数将在页面加载时被调用。它包含了正在被分配选择器`testCanvas`的`canvas`变量的过程，使用`document.getElementById("testCanvas");`将页面主体中的 canvas 与 canvas 变量绑定。接下来，我们准备通过输入`critterSheet.onload = handleImageLoad`加载一个新的`spriteSheet`。`critterSheet`变量存储了精灵图像的来源。输入`critterSheet.src = "images/moles.png"`；让我们可以访问我们自己的一系列鼹鼠图像。

我们创建的第二个函数是`handleImageLoad()`。在这个函数中，我们做了大部分工作，首先是使用`stage = new Stage(canvas)`创建了一个舞台的新实例；接下来，我们使用`var spriteSheet = new SpriteSheet(critterSheet, 76, 80)`创建了一个新的`spriteSheet`。

现在我们有了一个精灵图实例，我们可以创建一个新的位图序列变量，称为`critter1`，并定义它在舞台上的位置，使用 x 和 y 坐标来输入：`var critter1 = new BitmapSequence(spriteSheet);critter1.y = 85;critter1.x = 85`；。接下来，我们按数字引用要添加的帧，以便首先将正确的动作应用于 critter，然后应用于舞台。我们通过输入`critter1.gotoAndStop(1)`将`critter1`变量链接到我们精灵图`moles.png`上的第二个图像。我们使用命令`stage.addChild(critter1)`将图像添加到舞台上。

我们克隆了我们创建的第一个`critter1`变量，并通过输入`var critter2 = critter1.clone()`将其值传递给一个新的 critter 变量。我们通过使用`critter2.x += 120`将新变量定位在第一个 critter 的右侧，将其当前位置值添加到其中。我们通过命令`BitSequence`去到`moles.png`上的第一个图像的位置并在那里停止，并将其分配给`critter2`变量。

我们添加了`Tick.setInterval(300)`；，这样就在`Ticks`之间应用了`300`毫秒的时间间隔。Tick 接口充当全局定时设备，如果需要，可以返回每秒帧数（FPS）。我们向舞台添加了一个监听器`Tick.addListener(stage)`；，它的行为类似于其他类型的监听器，它监听`Ticks`。这可以用来在指定的时间重新绘制舞台，或执行其他与时间相关的操作。

我们使用`onload`属性在`body`标签中调用`init()`函数。这会导致在页面加载时调用`init()`函数。

## 另请参阅

*动画序列*教程。

# 使用 Easel.js 和 canvas 标签来制作图像序列的动画

我们可以通过创建数组和使用`Easel.js` JavaScript 库的函数来制作称为精灵的图像条，然后使用`canvas`元素对它们进行操作。在这个教程中，我们将对同一条图像条进行动画处理，但显示两个不同时间序列。

## 准备工作

下载此教程的代码文件，以使用`Easel.js`框架库以及支持文件。您需要一个能够正确显示 HTML5 元素并测试本教程中使用的代码的最新浏览器。

## 操作步骤

创建一个 HTML5 文件的开头标签。您的代码应该类似于以下代码块：

```html
<!DOCTYPE HTML><html><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8"><title> Animating images using BitmapSequence and SpriteSheet</title>

```

链接到本教程中使用的主样式表`styles.css`：<link href="styles.css" rel="stylesheet" type="text/css" />。

通过插入以下脚本文件的链接来导入`Easel.js`框架库：`UID.js, SpriteSheetUtils.js, SpriteSheet.js, DisplayObject.js, Container.js, Stage.js, BitmapSequence.js`和`Ticks.js`。参考前面的示例，了解框架块应该是什么样子的。

创建一个开头的`<script>`标签，并声明以下三个变量：`var canvas;var stage;var critterSheet = new Image()`；用于我们的脚本。

输入`function init(){`开始函数，然后输入`canvas = document.getElementById("testCanvas")`。

准备通过输入`critterSheet.onload = handleImageLoad`;来加载一个新的`spriteSheet`。输入`critterSheet.src = "images/moles.png"`;来加载我们自己的一系列鼹鼠图像。函数块应该如下所示的代码块：

```html
function init() {
canvas = document.getElementById("testCanvas");
critterSheet.onload = handleImageLoad;
critterSheet.src = "images/moles.png";}

```

我们将创建的第二个函数是`handleImageLoad()`。输入`function handleImageLoad() {`然后`stage = new Stage(canvas)`;创建舞台的新实例。输入`var spriteSheet = new SpriteSheet(critterSheet, 80, 80)`;创建一个新的`spriteSheet`。现在我们有了一个精灵表，创建一个新的位图序列变量名为`critter1`，并使用 x 和 y 坐标定义其在舞台上的位置，输入：`var critter1 = new BitmapSequence(spriteSheet)`;然后`critter1.y = 100;critter1.x = 90`;。接下来，我们将创建一个数组，将其映射到原始`spritesheet`文件上的每个图像，输入`var frameData = {shymole:0, upmole:1, downmole:2, whacked:3, whackedow:4, clouds:5,tinycloud:6, cloudgroup:7}`;以便我们有八个名称值，每个名称值都与一个数组 ID 相关联。

到目前为止，`handleImageLoad()`的代码块应该如下所示：

```html
function handleImageLoad() {
stage = new Stage(canvas);
var spriteSheet = new SpriteSheet(critterSheet, 80, 80);
var critter1 = new BitmapSequence(spriteSheet);
critter1.y = 100;
critter1.x = 90;
var frameData = {shymole:0, upmole:1, downmole:2, whacked:3, whackedow:4, clouds:5,tinycloud:6, cloudgroup:7};

```

通过输入：`spriteSheet = new SpriteSheet(critterSheet, 80, 80, frameData)`;创建一个新的`spriteSheet`并将其用作参数。

创建一个名为`critter1`的新位图序列变量，并通过输入`critter1gotoAndStop(0)`;应用图像精灵。使用`stage.addchild(critter1)`;将`critter1`添加到`stage`。

通过输入`var critter2 = critter1.clone()`;克隆第一个`critter1`变量，并将其值传递给一个新的小动物变量。使用`critter2.x += 120`;定义新变量的`x`值。通过输入`critter2.gotoAndStop(5)`;为小动物分配其自己的图像，来自`moles.png`图像文件。添加一个新的`spriteSheet`，创建`critter 1`和克隆`critter 2`的代码块应该如下所示的代码块：

```html
spriteSheet = new SpriteSheet(critterSheet, 80, 80, frameData);
critter1.gotoAndStop(0);
stage.addChild(critter1);
var critter2 = critter1.clone();
critter2.x += 120;critter2.gotoAndStop(5);

```

输入：`var critter3 = critter2.clone(); critter3.spriteSheet = spriteSheet`;。就像我们之前创建的其他小动物变量一样，通过将`10`添加到其当前值来重新定义`critter3`的`x`值：`critter3.x += 10`;。以下代码块显示了我们到目前为止所做的事情：

```html
var critter3 = critter2.clone();
critter3.spriteSheet = spriteSheet;
critter3.x += 10;

```

通过输入`critter3.gotoAndStop("upmole")`;按名称引用`moles.png`中的图像`frames`。通过克隆一个新变量并引用一个新帧，将当前的`upmole`帧图像替换为不同的帧：`var critter4 = critter3.clone(); critter4.gotoAndStop("downmole")`;。通过输入`critter4.x += 10`;将该帧向右移动`10`像素。

再次交换帧并将我们的新帧向右移动`10`像素：`var critter5 = critter4.clone(); critter5.gotoAndStop("shymole"); critter5.x += 10`;。让我们看一下到目前为止我们应该有的代码块：

```html
critter3.gotoAndStop("upmole");
var critter4 = critter3.clone();
critter4.gotoAndStop("downmole");
critter4.x += 10;
var critter5 = critter4.clone();
critter5.gotoAndStop("shymole");
critter5.x += 10;

```

通过输入以下内容循环播放我们的`moles.png`文件中的帧：

```html
var critter6 = critter1.clone(); critter6.x = critter5.x + 100; critter6.gotoAndPlay(3);stage.addChild(critter6);.

```

在舞台上添加第二个动画序列，通过引用不同的起始帧来改变动画的时序，当新的小动物精灵被添加到舞台上时：`var critter7 = critter1.clone(); critter7.x = critter6.x + 100; critter7.gotoAndPlay(1); stage.addChild(critter7)`;。

我们的两个动画序列现在应该包含以下代码：

```html
var critter6 = critter1.clone();
critter6.x = critter5.x + 100;
critter6.gotoAndPlay(3);
stage.addChild(critter6);
var critter7 = critter1.clone();
critter7.x = critter6.x + 100;
critter7.gotoAndPlay(1);
stage.addChild(critter7);

```

Tick 间隔`Tick.setInterval(200)`;和监听器`Tick.addListener(stage)`;是我们将添加到脚本中的最后两个语句。关闭`handleImageLoad()`函数的大括号（}）并输入一个闭合的脚本标签。

输入`</head>`，然后`<body onload="init()">`。创建一个名为`"description"`的 div 来容纳内容。最后一个 div 是`canvasHolder`，包含 canvas 元素。将宽度设置为`600`，高度设置为`280`，背景颜色设置为浅灰色`(#ccc)`。添加指向图像文件`moles.png`的链接，以便用户可以看到`moles.png`中引用的图像精灵。

保存文件，并在浏览器窗口中打开它。你应该在左侧看到一个静止的帧（闭着眼睛的鼹鼠头像），并在屏幕右侧看到两个动画序列循环播放。以下截图显示了两个序列如何加载相同的帧，但时间不同。

![如何操作...](img/1048_07_09.jpg)

## 它是如何工作的...

创建 HTML 页面和引用画布的配方的第一步与上一个配方相同。

创建`spriteSheet`后，我们创建了一个新变量来保存我们的精灵帧，称为`critter1`，并通过输入以下内容定义了帧位置的`x`和`y`坐标：`var critter1 = new BitmapSequence(spriteSheet); critter1.y = 100;critter1.x = 90`;。

我们创建了数组`var frameData`来声明八个键/值对。然后，我们能够创建一个新的`spriteSheet`，它接受了`spriteSheet`名称的参数，每个帧的默认高度和宽度，并使用`frameData`一次性加载`moles.png`中的所有帧到`spriteSheet`中。

接下来，我们尝试使用`frameData`通过数字值和名称键引用帧，创建一系列位图序列，然后用它们的克隆替换它们。

我们对序列进行了动画处理，并将它们放在了舞台上。它们都遵循相同的格式，但通过更改`gotoAndPlay`操作中的数字参数，它们在不同的帧上开始它们的动画序列。

最后，我们添加了`Tick.setInterval(200)`;，它在 Ticks 之间应用了 200 毫秒的时间间隔。Tick 接口充当全局定时设备，如果需要，可以返回每秒帧数（FPS）。我们向舞台添加了一个监听器`Tick.addListener(stage)`;，它像其他类型的监听器一样监听 Ticks。这可以用来在指定时间重新绘制舞台，或执行其他与时间相关的操作。我们使用`onload`属性在`body`标签中调用`init()`函数。这会导致在页面加载时调用`init()`函数。

## 还有更多...

`Easel.js`和其他类似的库使控制 HTML5 元素变得更加容易。不过要小心使用它们，因为有些可能不够稳定，不能在生产环境中使用。

### 海盗爱雏菊，你也应该爱

`Easel.js`的创建者被微软要求创建一个名为 Pirates love daisies 的概念性网络游戏（[`www.pirateslovedaisies.com`](http://www.pirateslovedaisies.com)），完全使用 HTML5 和 JavaScript，并且严重依赖于`Easel.js`库来操作`canvas`元素。你可以在任何网络浏览器中玩这个游戏，也许有些讽刺的是，它还为使用 Internet Explorer 9 浏览器的访问者提供了特殊功能。

### 老派计算机动画技术的回归

当我第一次开始在计算机上玩游戏时，游戏屏幕上有 256 种颜色和 8 位动画是一件大事。计算机动画师使用了许多技巧来复制水流动等效果。重新体验那些日子（或者第一次通过 effect games 的演示发现它们：[`www.effectgames.com/demos/canvascycle/`](http://www.effectgames.com/demos/canvascycle/)。

## 另请参阅

本书中有一整章关于 canvas 的配方。如果你跳过了它们，现在就去吞食它们。

# 使用 canvas 标签和 JavaScript 进行随机动画和音频

在这个配方中，我们将使用 canvas 标签来绘制和动画一系列形状。我们还将使用音频标签循环播放音频文件，同时显示动画。我们正在改编 Michael Nutt 创建的原始动画。我们将创建一个更慢、更轻松的动画，看起来像是摇曳的草。

## 做好准备

您将需要一个最近更新的浏览器，如 Firefox 3.6 或 Google Chrome，以及多种格式的音频文件。在 Opera 浏览器 9 和 10 中，它显示的大小会有所不同（更小）。在那些版本的 Opera 中，音频也不会播放。

## 如何做...

首先，打开一个新的 HTML5 页面，并命名为`random-animation-with-audio.html`。输入一个 HTML5 页面的开头，包括页面标题：

```html
<!DOCTYPE html> <html lang="en"> <head><meta charset="utf-8" /> <title>Canvas Reggae</title>.

```

然后，添加链接到 JavaScript 和 CSS 文件，这些文件将在页面加载时导入：`<script type="text/javascript" src="img/animatedlines.js"></script><link rel="stylesheet" href="css/stylesheet.css" type="text/css" media="screen" charset="utf-8" />`，并使用`</head>`关闭 head 标签。

输入`<body onLoad="init();">`来在页面加载时激活`init()`函数。

接下来，我们创建页面的标题`<header><h1>CANVAS Reggae</h1></header>`，然后通过输入`<canvas id="tutorial" width="480" height="360"></canvas>`来添加 canvas 元素。

创建一个新的 div，其中包含一个`id`为 credits 的链接到 Michael 的网站：`<div id="credits">Based on Canvas Party by <a href="http://nuttnet.net/">Michael Nutt</a>&nbsp;&nbsp`;。然后向 div 添加一个链接，以抓取音频元素并在单击链接时应用`pause()`函数来暂停音乐。`<a href="#" onClick="document.getElementsByTagName('audio')[0].pause();">[OFF]</a></div>`。

现在，输入音频标签，并将 autoplay 设置为 true，loop 设置为 loop：`<audio autoplay="true" loop="loop">`创建两个 source 标签来包含音频格式：`<source type="audio/ogg" src="img/randomreggae.ogg" /><source type="audio/mpeg" src="img/randomreggae.mp3" />`。

在关闭音频标签之前，我们将添加一串文本，如果不支持音频标签，则会显示：`Your browser doesn't recognize the HTML5 audio tag`。

关闭音频、body 和 html 标签，并保存页面。

在创建脚本之前，打开`stylesheet.css`页面，并输入以下内容：

```html
body { margin: 0; background-color: #000; color: #FFF; font-family: Helvetica, sans-serif; }
a { color: #FFF; }
h1 { position: absolute; top: 0; margin: auto; z-index: 50; padding: 10px; background-color: #000; color: #FFF; }
div#credits { position: absolute; bottom: 0; right: 0; padding: 10px; }
audio { position: absolute; visibility: hidden; }

```

现在 HTML 和 CSS 页面已经构建完成，我们将着手处理动画脚本。创建一个新的 JavaScript 文件并命名为`animatedLines.js`。我们将把它放在一个名为`js`的新子文件夹中。

首先，我们将声明 flatten 变量并创建一个新的数组函数：`var flatten = function(array) { var r = []`。接下来，在函数内部，我们将创建一个`for`语句来声明一个以一个对象开始的数组（`var i = 0`），然后在数组长度大于`i`的情况下增加数组的大小。`for(var i = 0; i < array.length; i++) {`。使用`push`函数，我们将通过输入以下内容向数组添加新值：`r.push.apply(r, array[i]);}`，最后通过返回数组来结束函数：`return r; }`。

到目前为止，我们的脚本应该看起来像以下代码块：

```html
var flatten = function(array) { var r = [];
for(var i = 0; i < array.length; i++) {
r.push.apply(r, array[i]); }
return r; }

```

接下来，我们将创建一个名为 shuffle 的函数，它接受一个数组作为参数。输入`function shuffle(array) { var tmp, current, top = array.length`。在函数内部，我们有一个 if/while 循环来遍历数组中的值。通过输入以下内容将其添加到脚本中：`var tmp, current, top = array.length; if(top) while(--top) { current = Math.floor(Math.random() * (top + 1)); tmp = array[current]; array[current] = array[top]; array[top] = tmp; }`。在函数末尾返回数组的值。我们的随机洗牌数组值的函数现在应该看起来像以下代码块：

```html
function shuffle(array) {
var tmp, current, top = array.length;
if(top) while(--top) {
current = Math.floor(Math.random() * (top + 1));
tmp = array[current];
array[current] = array[top];
array[top] = tmp; }
return array; }

```

现在，我们准备创建一个全局的`canvas`变量和一个`context`变量，输入：`var canvas`;和`var ctx;`。

有了这些变量，我们可以向脚本添加`init()`函数，所有的动作都从这里开始。输入`function init() {`，然后输入语句将我们的 canvas 变量与 canvas 元素关联起来：`canvas = document.getElementById('tutorial')`。

现在，我们将创建一个`if`语句来设置我们的画布变量的宽度和高度属性：`if (canvas.getContext) {canvas.width = window.innerWidth; canvas.height = window.innerHeight - 100; ctx = canvas.getContext('2d'); ctx.lineJoin = "round"; setInterval("draw()", 300); }。这`完成了`init()`函数。

接下来，我们为浏览器窗口添加一个监听器，以便在调整大小时检测：`window.addEventListener('resize', function() {canvas.width = window.innerWidth;canvas.height = window.innerHeight - 100; });}`。

我们脚本的最新添加现在应该是：

```html
function init() {
canvas = document.getElementById('tutorial');
if (canvas.getContext) {
canvas.width = window.innerWidth;
canvas.height = window.innerHeight - 100;
ctx = canvas.getContext('2d');
ctx.lineJoin = "round";
setInterval("draw()", 300); }
window.addEventListener('resize', function() {
canvas.width = window.innerWidth;
canvas.height = window.innerHeight - 100; }); }

```

我们终于准备好创建一个函数来在画布上绘制形状。这个函数将包含大部分驱动形状动画的脚本。输入`function draw(){ctx.globalCompositeOperation = "darker"; ctx.fillStyle = '#000'; ctx.fillRect(0, 0, canvas.width, canvas.height);ctx.globalCompositeOperation = "lighter";`来设置画布背景的外观。

现在，我们将输入动画中要使用的颜色。我们将创建一个包含`rgba`值的数组的数组。类型：`var colors = ["rgba(134, 154, 67, 0.8)", "rgba(196, 187, 72, 0.8)", "rgba(247, 210, 82, 1)", "rgba(225, 124, 20, 0.8)"];。我们`颜色已经定义好了，现在我们将使用一个包含宽度和高度值的单独数组的数组来设置形状的宽度和高度：`var data = [ [ [5, 20], [15, 2] ], [ [50, 12], [10, 14], [3, 21] ], [ [60, 8]], [ [30, 24], [15, 4], [10, 17] ], [ [5, 10] ], [ [60, 5], [10, 6], [3, 26] ], [ [20, 18] ], [ [90, 11], [40, 13], [15, 10] ], [ [70, 19] ], ]`。

现在，我们可以通过使用`data = shuffle(data)`来改变它们的宽度和高度来使形状动画化。

为了使形状上下以及左右移动，我们需要"压扁"或压缩它们的高度。创建一个新变量来包含`var flatData = flatten(data)`；

现在，我们将扭曲线条，使它们看起来像是在不同方向上拉动并使用`bezierCurve`。这是一个大的函数块，包含在我们之前创建的`draw()`函数中，所以输入`link()`函数如下所示：

```html
link(topPos, bottomPos, width) {
var padding = 100;
ctx.lineWidth = width;
ctx.beginPath();
var height = parseInt(canvas.height - padding);
var pull = 100;
var topLeft = topPos + (width / 2) + padding;
var bottomLeft = bottomPos + (width / 2) + padding;
ctx.moveTo(topLeft, padding);
ctx.bezierCurveTo(topLeft, pull, bottomLeft, height - pull, bottomLeft, height);
ctx.stroke(); }

```

现在，当我们仍然在`draw()`函数中时，让我们添加一个新变量来表示形状的起始点，然后添加一个`for`循环来创建一个可以容纳数据值集合的新变量。以下是变量和循环代码：`Var topStartingPoint = 0; for(var i in data) { var group = data[i]; var color = colors[ i % colors.length ];ctx.strokeStyle = color`。

通过创建一个嵌套的`for`循环，将一组数据值传递给一个名为`line`的新变量来进一步操作：`for(var j in group) { var line = group[j]`；然后我们可以在创建一个初始值为零的`bottomStartingPoint`变量后进行操作：`var bottomStartingPoint = 0`。

第三个嵌套的`for`循环将允许我们进一步控制形状的定位和移动：`for(var k in flatData) { if(flatData[k][1] < line[1]) { bottomStartingPoint += flatData[k][0] + 11;} }`。

最后，我们使用 link 来设置线条的顶部和底部起始点，`link(topStartingPoint, bottomStartingPoint, line[0])`；，然后将`topStartingPoint`赋值为其当前值加上线条数组。最后的语句将`topStartingPoint`值设置为其当前值加上五：`topStartingPoint += line[0]; } topStartingPoint += 5; }}`。保存脚本文件。

在浏览器中打开文件`random-animation-with-audio.html`，您应该会看到线条来回摆动，类似于以下截图所示：

![操作步骤...](img/1048_07_10.jpg)

## 它是如何工作的...

首先，我们创建了一个 HTML5 页面，其中包含指向在页面加载时导入的 JavaScript 和 CSS 文件的链接：`<script type="text/javascript" src="img/animatedlines.js"></script><link rel="stylesheet" href="css/stylesheet.css" type="text/css" media="screen" charset="utf-8" />`。为了激活我们的动画序列，我们将`init()`函数放在 HTML 页面的 body 标签中。当页面加载时，`animatedLines.js` JavaScript 文件中的`init()`函数将通过`<body onLoad="init();">`进行初始化。

我们使用`body`样式来设置页面的全局默认`margin`为`0`，`background-color`，`font color`和`font-family`。我们为基本链接颜色设置了样式，然后对`h1`标题标签进行了样式设置，以便它以`position: absolute; top: 0`的方式显示在`top`位置，并通过将`z-index`设置为`50`始终显示在大多数其他内容块的上方。`#credits` div 被定位在页面的右下角，音频标签使用`visibility: hidden`进行隐藏。

我们创建了一个名为`animatedLines.js`的新脚本，并首先定义了一系列变量和函数来控制形状的行为。

我们设置了一个名为`flatten`的数组，它会将新值添加到自身。接下来，我们需要一个函数来随机旋转数组的值。我们使用了`Math.floor(Math.random()`语句来计算一个随机数，并将结果乘以变量`top + 1`的当前值的总和。然后我们将一个整数值返回给变量`current`。

我们通过使用`document.getElementById`在页面加载时获取`canvas`元素的 ID 来定义`canvas`变量的尺寸值。我们使用 DOM 的帮助设置了`canvas`变量的`width`和`height`属性：`canvas.height = window.innerHeight - 100; ctx = canvas.getContext('2d')`；然后创建了一个语句，对`canvas`的`2d`上下文应用了`lineJoin`，参数为`round`。我们使用`setInterval()`函数将线条在画布上绘制的速度设置为`300`毫秒。数字越大，动画看起来越慢。我们为浏览器窗口添加了一个监听器，以便检测窗口的大小和画布。

然后使用`draw()`函数将形状绘制到画布上。使用`globalCompositeOperation = "darker"`来使线条在相互移动时变暗。线条在画布舞台前部重叠时，使用`globalCompositeOperation = "lighter"`来设置画布背景的外观。

用于装饰线条的颜色需要以`rgba`格式。rgba 中的'a'指的是 alpha 值，控制每种颜色的可见性。每个 rgba 值集合都包含在一个数组中，然后成为数组列表。我们需要与线条相匹配的宽度和高度值集合。这些存储在数组`var data`中。

接下来，我们将`data`数组分配给从我们的`shuffle()`函数返回的值，以便我们可以随机化屏幕上线条的外观。然后，我们将从`flatten()`函数返回的值分配给变量`flatData`。为每条线分配一个拉动值使我们能够将其移动指定数量的像素。我们将这与`bezierCurve`结合起来，使线条弯曲。

## 还有更多...

将音频标签、画布动画和 JavaScript 结合起来，听起来像是一种有趣的方式来创建酷炫的可视化效果。然而，这些效果在很大程度上依赖于浏览器的支持，因此许多网络浏览器用户目前无法正确查看它们。我的意思是，大多数标准浏览器在一两年内都无法播放它们。

### 使用尖端浏览器可视化您的音频

如果您已经下载了测试版的 Firefox 4，您就可以访问 Firefox 音频和视频 API。您将能够使用类似 Spectrum Visualizer 的工具查看和创建自己的音频可视化效果：

[`www.storiesinflight.com/jsfft/visualizer/index.html`](http://www.storiesinflight.com/jsfft/visualizer/index.html)

### 推动 HTML5 中音频的实现

Alexander Chen 一直在尝试使用音频和画布来移植基于 Flash 的应用程序。他在博客中详细介绍了使用多个音频文件时遇到的一些问题：

[`blog.chenalexander.com/2011/limitations-of-layering-html5-audio/`](http://blog.chenalexander.com/2011/limitations-of-layering-html5-audio/)

## 另请参阅

画布和
