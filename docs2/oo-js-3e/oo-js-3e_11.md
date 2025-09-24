# 第十一章。编码和设计模式

现在你已经了解了 JavaScript 中的所有对象，掌握了原型和继承，也看到了一些使用浏览器特定对象的实际例子，让我们继续前进，或者说，提升一个层次。让我们来看看一些常见的 JavaScript 模式。

但首先，什么是模式？简而言之，模式是针对常见问题的良好解决方案。将解决方案编纂成模式使其可重复。

有时候，当你面对一个新的编程问题时，你可能会立刻意识到你之前已经解决过另一个类似的问题。在这种情况下，值得隔离这类问题并寻找一个共同的解决方案。模式是对一类问题的经过验证的可重复解决方案（或解决方案的方法）。

有时候，模式可能仅仅是一个想法或一个名称。有时，仅仅使用一个名称就能帮助你更清晰地思考问题。此外，当与团队中的其他开发者一起工作时，如果每个人都使用相同的术语来讨论问题或解决方案，沟通会更容易。

有时，你可能会遇到一个独特的问题，它看起来不像你之前见过的任何东西，也不容易适应已知的模式。盲目地应用模式只是为了使用模式，这不是一个好主意。与其尝试调整你的问题以适应现有的解决方案，不如不使用任何已知的模式。

本章讨论两种类型的模式，如下所示：

+   **编码模式**：这些主要是 JavaScript 特定的最佳实践

+   **设计模式**：这些是语言无关的模式，由著名的*四人帮*书籍推广

# 编码模式

让我们从一些反映 JavaScript 独特特性的模式开始。有些模式旨在帮助你组织代码，例如命名空间；其他与提高性能相关，如懒定义和初始化时分支；还有一些是为了弥补缺失的功能，如私有属性。本节讨论的模式包括以下主题：

+   分离行为

+   命名空间

+   初始化时分支

+   懒定义

+   配置对象

+   私有变量和方法

+   特权方法

+   将私有函数作为公共方法

+   立即函数

+   链式调用

+   JSON

## 分离行为

如前所述，网页的三个构建块如下：

+   内容（HTML）

+   展示（CSS）

+   行为（JavaScript）

### 内容

HTML 是网页的内容，即实际文本。理想情况下，内容应该使用尽可能少的 HTML 标签进行标记，以充分描述该内容的语义意义。例如，如果你正在制作导航菜单，使用`<ul>`和`<li>`标签作为导航菜单是一个好主意，因为导航菜单本质上只是一个链接列表。

你的内容（HTML）应该不包含任何格式化元素。视觉格式化属于表现层，应该通过使用**CSS**（**层叠样式表**）来实现。这意味着以下内容：

+   如果可能的话，不应使用 HTML 标签的`style`属性。

+   应当完全不使用如`<font>`之类的表现性 HTML 标签。

+   标签应该用于它们的语义意义，而不是因为浏览器默认如何渲染它们。例如，开发者有时会在更适合使用`<p>`标签的地方使用`<div>`标签。使用`<strong>`和`<em>`代替`<b>`和`<i>`也是有益的，因为后者描述的是视觉表现而不是意义。

### 表现性

将表现性内容从内容中分离的一个好方法是重置或取消所有浏览器默认设置，例如，使用 Yahoo! UI 库中的`reset.css`。这样，浏览器默认的渲染不会分散你对于正确语义标签的思考。

### 行为

网页的第三个组成部分是行为。行为应该与内容和表现性内容保持分离。通常是通过使用隔离到`<script>`标签中的 JavaScript 来添加的，并且最好包含在外部文件中。这意味着不要使用任何内联属性，例如`onclick`、`onmouseover`等。相反，你可以使用上一章中提到的`addEventListener`/`attachEvent`方法。

将行为与内容分离的最佳策略如下：

+   最小化`<script>`标签的数量

+   避免使用内联事件处理器

+   不要使用 CSS 表达式

+   在你的内容末尾，当你准备好关闭`<body>`标签时，插入一个单独的`external.js`文件

#### 分离行为的示例

假设你有一个页面上的搜索表单，并且你想要使用 JavaScript 来验证这个表单。因此，你继续并确保`form`标签不包含任何 JavaScript，然后在`</body>`标签关闭之前立即插入一个指向外部文件的`<script>`标签，如下所示：

```js
    <body> 
      <form id="myform" method="post" action="server.php"> 
      <fieldset> 
        <legend>Search</legend> 
        <input 
          name="search" 
          id="search" 
          type="text"   
        /> 
        <input type="submit" /> 
        </fieldset> 
      </form> 
      <script src="img/behaviors.js"></script> 
    </body> 

```

在`behaviors.js`中，你为提交事件附加了一个事件监听器。在你的监听器中，你可以检查文本输入字段是否被留空，如果是这样，就阻止表单提交。这样，你将节省服务器和客户端之间的往返，并使应用程序立即响应。

`behaviors.js`的内容如下所示代码。它假设你已经从上一章末尾的练习中创建了你的`myevent`实用工具：

```js
    // init 
    myevent.addListener('myform', 'submit', function (e) { 
      // no need to propagate further 
      e = myevent.getEvent(e); 
      myevent.stopPropagation(e); 
      // validate 
      var el = document.getElementById('search'); 
      if (!el.value) { // too bad, field is empty 
        myevent.preventDefault(e); // prevent the form submission 
        alert('Please enter a search string'); 
      } 
    }); 

```

### 异步 JavaScript 加载

你注意到了脚本是在 HTML 的末尾加载的，就在关闭`body`标签之前。原因是 JavaScript 会阻止页面 DOM 的构建，在某些浏览器中，甚至还会阻止后续组件的下载。通过将脚本移到页面底部，你可以确保脚本不会妨碍，当它到达时，它只是简单地增强了已经可用的页面。

防止外部 JavaScript 文件阻塞页面的另一种方法是异步加载它们。这样你可以更早地开始加载它们。HTML5 有`defer`属性用于此目的。考虑以下代码行：

```js
    <script defer src="img/behaviors.js"></script> 

```

不幸的是，`defer`属性不被旧版浏览器支持，但幸运的是，有一个适用于新旧浏览器的解决方案。该解决方案是动态创建一个`script`节点并将其附加到 DOM 中。换句话说，你可以使用一些内联 JavaScript 来加载外部 JavaScript 文件。你可以在文档的顶部放置这个脚本加载器代码片段，以便下载可以尽早开始。请看以下代码示例：

```js
    ... 
    <head> 
    <script> 
    (function () { 
      var s = document.createElement('script'); 
      s.src = 'behaviors.js'; 
      document.getElementsByTagName('head')[0].appendChild(s); 
    }()); 
    </script> 
    </head> 
    ... 

```

## 命名空间

应该避免使用全局变量，以减少变量命名冲突的可能性。你可以通过命名空间你的变量和函数来最小化全局变量的数量。这个想法很简单，你将只创建一个全局对象，而你所有的其他变量和函数都成为该对象属性的属性。

### 对象作为一个命名空间

让我们创建一个名为`MYAPP`的全局对象：

```js
    // global namespace 
    var MYAPP = MYAPP || {}; 

```

现在，你不再需要全局的`myevent`实用工具（来自上一章），你可以将其作为`MYAPP`对象的`event`属性，如下所示：

```js
    // sub-object 
    MYAPP.event = {}; 

```

将方法添加到`event`实用工具中仍然相同。考虑以下示例：

```js
    // object together with the method declarations 
    MYAPP.event = { 
      addListener: function (el, type, fn) { 
        // .. do the thing 
      }, 
      removeListener: function (el, type, fn) { 
        // ... 
      }, 
      getEvent: function (e) { 
        // ... 
      } 
      // ... other methods or properties 
    }; 

```

### 命名空间构造函数

使用命名空间不会阻止你创建构造函数。以下是如何创建一个具有`Element`构造函数的 DOM 实用工具，这允许你轻松创建 DOM 元素：

```js
    MYAPP.dom = {}; 
    MYAPP.dom.Element = function (type, properties) { 
      var tmp = document.createElement(type); 
      for (var i in properties) { 
        if (properties.hasOwnProperty(i)) { 
          tmp.setAttribute(i, properties[i]); 
        } 
      } 
       return tmp; 
    }; 

```

同样，你可以有一个`Text`构造函数来创建文本节点。考虑以下代码示例：

```js
    MYAPP.dom.Text = function (txt) { 
      return document.createTextNode(txt); 
    }; 

```

使用构造函数在页面底部创建链接可以这样做：

```js
    var link = new MYAPP.dom.Element('a',  
      {href: 'http://phpied.com', target: '_blank'}); 
    var text = new MYAPP.dom.Text('click me'); 
    link.appendChild(text); 
    document.body.appendChild(link); 

```

### namespace()方法

你可以创建一个命名空间实用工具，使你的生活更轻松，这样你就可以使用更方便的语法，如下所示：

```js
    MYAPP.namespace('dom.style'); 

```

而不是如下更冗长的语法：

```js
    MYAPP.dom = {}; 
    MYAPP.dom.style = {}; 

```

这里是如何创建这样的`namespace()`方法的。首先，你将通过使用点（`.`）作为分隔符来分割输入字符串创建一个数组。然后，对于新数组中的每个元素，你将向你的全局对象添加一个属性，如果尚未存在，如下所示：

```js
    var MYAPP = {}; 
    MYAPP.namespace = function (name) { 
      var parts = name.split('.'); 
      var current = MYAPP; 
      for (var i = 0; i < parts.length; i++) { 
        if (!current[parts[i]]) { 
          current[parts[i]] = {}; 
        } 
        current = current[parts[i]]; 
      } 
    }; 

```

测试新方法如下：

```js
    MYAPP.namespace('event'); 
    MYAPP.namespace('dom.style'); 

```

上述代码的结果与以下操作相同：

```js
    var MYAPP = { 
      event: {}, 
      dom: { 
        style: {} 
      } 
    }; 

```

## 初始化时分支

在上一章中，你注意到有时，不同的浏览器对相同或类似功能有不同的实现。在这种情况下，你需要根据当前执行脚本的浏览器支持的内容来分支你的代码。根据你的程序，这种分支可能会发生得太频繁，从而导致脚本执行速度变慢。

你可以通过在初始化期间，即脚本加载时而不是在运行时，对代码的一些部分进行分支来减轻这个问题。基于能够动态定义函数的能力，你可以根据浏览器进行分支，并使用不同的主体定义相同的函数。让我们看看如何做。

首先，让我们为`event`实用工具定义一个命名空间和占位符方法：

```js
    var MYAPP = {}; 
    MYAPP.event = { 
      addListener: null, 
      removeListener: null 
    }; 

```

到目前为止，添加或删除监听器的方法尚未实现。根据功能嗅探的结果，这些方法可以定义得不同，如下所示：

```js
    if (window.addEventListener) { 
      MYAPP.event.addListener = function (el, type, fn) { 
        el.addEventListener(type, fn, false); 
      }; 
      MYAPP.event.removeListener = function (el, type, fn) { 
        el.removeEventListener(type, fn, false); 
      }; 
    } else if (document.attachEvent) { // IE 
      MYAPP.event.addListener = function (el, type, fn) { 
        el.attachEvent('on' + type, fn); 
      }; 
      MYAPP.event.removeListener = function (el, type, fn) { 
        el.detachEvent('on' + type, fn); 
      }; 
    } else { // older browsers 
      MYAPP.event.addListener = function (el, type, fn) { 
        el['on' + type] = fn; 
      }; 
      MYAPP.event.removeListener = function (el, type) { 
        el['on' + type] = null; 
      }; 
    } 

```

在此脚本执行之后，你将定义了浏览器依赖的`addListener()`和`removeListener()`方法。现在，每次调用这些方法之一时，就不再进行功能嗅探，这导致工作量减少，执行速度更快。

在嗅探功能时要留意的一点是，在检查了一个功能之后不要假设太多。在之前的例子中，这条规则被违反了，因为代码只检查了`addEventListener`的支持，但随后定义了`addListener()`和`removeListener()`。在这种情况下，假设如果浏览器实现了`addEventListener()`，它也实现了`removeEventListener()`可能是安全的。然而，想象一下如果浏览器实现了`stopPropagation()`但没有实现`preventDefault()`，而你没有单独检查这些功能会发生什么。你假设因为`addEventListener()`未定义，浏览器必须是旧 IE，并使用你对 IE 如何工作的知识和假设来编写代码。记住，你所有的知识都是基于今天某个浏览器的工作方式，但不一定是明天的工作方式。因此，为了避免在新浏览器版本发布时多次重写代码，最好单独检查你打算使用的功能，不要对某个浏览器支持的功能进行泛化。

## 懒定义

懒定义模式与之前的初始化时分支模式类似。不同之处在于分支仅在函数第一次被调用时发生。当函数被调用时，它会用最佳实现重新定义自己。与初始化时分支不同，那里的 if 只发生一次，在加载期间，这里可能根本不会发生，在函数从未被调用的情况下。懒定义还使得初始化过程更轻量，因为没有初始化时分支的工作要做。

让我们通过`addListener()`函数的定义来举例说明这一点。该函数首先定义了一个通用的主体。它在第一次被调用时检查浏览器支持哪些功能，然后使用最合适的实现重新定义自己。在第一次调用结束时，函数会调用自己，以便执行实际的事件附加。下次你调用同一个函数时，它将使用新的主体定义并准备好使用，因此不需要进一步的分支。以下是一个代码片段：

```js
    var MYAPP = {}; 
    MYAPP.myevent = { 
     addListener: function (el, type, fn) { 
        if (el.addEventListener) { 
          MYAPP.myevent.addListener = function (el, type, fn) { 
            el.addEventListener(type, fn, false); 
          }; 
        } else if (el.attachEvent) { 
          MYAPP.myevent.addListener = function (el, type, fn) { 
            el.attachEvent('on' + type, fn); 
          }; 
        } else { 
          MYAPP.myevent.addListener = function (el, type, fn) { 
            el['on' + type] = fn; 
          }; 
        } 
        MYAPP.myevent.addListener(el, type, fn); 
      } 
    }; 

```

## 配置对象

当你有一个接受很多可选参数的函数或方法时，这种模式很方便。由你决定多少构成“很多”。但通常，具有三个以上参数的函数不太方便调用，因为你必须记住参数的顺序，而当一些参数是可选的时候，这甚至更加不方便。

而不是有很多参数，你可以使用一个参数并将其作为一个对象。对象属性是实际参数。这适合传递配置选项，因为这些通常数量众多且可选（具有智能默认值）。使用单个对象而不是多个参数的优点如下所述：

+   顺序无关紧要

+   你可以轻松跳过你不想设置的参数

+   添加更多可选配置属性很容易

+   这使得代码更易于阅读，因为配置对象的属性及其名称都出现在调用代码中

假设你有一种 UI 小部件构造函数，用于创建花哨的按钮。它接受要放在按钮内的文本（`<input>`标签的`value`属性）和按钮的`type`类型的可选参数。为了简单起见，让我们假设花哨按钮的配置与普通按钮相同。看看以下代码：

```js
    // a constructor that creates buttons 
    MYAPP.dom.FancyButton = function (text, type) { 
      var b = document.createElement('input'); 
      b.type = type || 'submit'; 
      b.value = text; 
      return b; 
    }; 

```

使用构造函数很简单；你只需给它一个字符串。然后，你可以将新按钮添加到文档的主体中如下所示：

```js
    document.body.appendChild( 
      new MYAPP.dom.FancyButton('puuush') 
    ); 

```

这一切都很顺利，工作得很好，但后来你决定你还想能够设置一些按钮的样式属性，例如颜色和字体。你可能会得到以下定义：

```js
    MYAPP.dom.FancyButton =  
      function (text, type, color, border, font) { 
      // ... 
    }; 

```

现在，使用构造函数可能会变得有点不方便，尤其是当你想设置第三个和第五个参数，但不设置第二个或第四个参数时。考虑以下示例：

```js
    new MYAPP.dom.FancyButton( 
      'puuush', null, 'white', null, 'Arial'); 

```

更好的方法是使用一个`config`对象参数来设置所有设置。函数定义可以变成以下代码片段：

```js
    MYAPP.dom.FancyButton = function (text, conf) { 
      var type = conf.type || 'submit'; 
      var font = conf.font || 'Verdana'; 
      // ... 
    }; 

```

使用构造函数的示例如下：

```js
    var config = { 
      font: 'Arial, Verdana, sans-serif', 
      color: 'white' 
    }; 
    new MYAPP.dom.FancyButton('puuush', config); 

```

另一个用法示例如下：

```js
    document.body.appendChild( 
      new MYAPP.dom.FancyButton('dude', {color: 'red'}) 
    ); 

```

如你所见，很容易只设置一些参数并改变它们的顺序。此外，当你看到参数的名称与调用方法的位置相同时，代码更友好且易于理解。

这种模式的缺点与它的优点相同。很容易继续添加更多参数，这意味着很容易滥用这项技术。一旦你找到了添加到这个免费属性袋中的理由，你会发现继续添加一些非完全可选的或依赖于其他属性的属性很有诱惑力。

作为一项经验法则，所有这些属性应该是独立的和可选的。如果你必须在函数内部检查所有可能的组合（“哦，A 已经设置，但只有在 B 也设置的情况下才会使用 A”），这将是大型函数体的配方，这会很快变得令人困惑，难以测试，如果不是不可能的话，因为所有这些组合。

## 私有属性和方法

JavaScript 没有访问修饰符的概念，它不会设置对象中属性的权限。其他语言通常有访问修饰符，如下所示：

+   `公共`：对象的所有用户都可以访问这些属性或方法

+   `私有`：只有对象本身可以访问这些属性

+   `受保护`：只有继承该对象的子对象可以访问这些属性

JavaScript 没有特殊的语法来表示私有属性或方法，但如 第三章 中所讨论的，*函数*，你可以在函数内部使用局部变量和方法，达到相同级别的保护。

继续以 `FancyButton` 构造函数为例，你可以拥有包含所有默认值的局部变量样式，以及一个局部的 `setStyle()` 函数。这些对构造函数外部的代码来说是不可见的。以下是 `FancyButton` 如何利用局部私有属性的方式：

```js
    var MYAPP = {}; 
    MYAPP.dom = {}; 
    MYAPP.dom.FancyButton = function (text, conf) { 
      var styles = { 
        font: 'Verdana', 
        border: '1px solid black', 
        color: 'black', 
        background: 'grey' 
      }; 
      function setStyles(b) { 
        var i; 
        for (i in styles) { 
          if (styles.hasOwnProperty(i)) { 
            b.style[i] = conf[i] || styles[i]; 
          } 
       } 
      } 
      conf = conf || {}; 
      var b = document.createElement('input'); 
      b.type = conf.type || 'submit'; 
      b.value = text; 
      setStyles(b); 
      return b; 
    }; 

```

在这个实现中，`styles` 是一个私有属性，`setStyle()` 是一个私有方法。构造函数内部使用它们（并且它们可以访问构造函数内部的所有内容），但它们对函数外部的代码不可用。

## 特权方法

特权方法（这个术语是由 Douglas Crockford 提出的）是正常的公共方法，可以访问私有方法或属性。它们可以像桥梁一样在受控的方式下使一些私有功能可访问，但仍然通过特权方法进行封装。

## 将私有函数作为公共方法

假设你定义了一个你绝对需要保持完整的函数，因此你将其设置为私有。然而，你仍然希望提供对该函数的访问，以便外部代码也能从中受益。在这种情况下，你可以将私有函数分配给一个公开可用的属性。

让我们定义 `_setStyle()` 和 `_getStyle()` 作为私有函数，然后将它们分配给公共的 `setStyle()` 和 `getStyle()`，以下是一个示例：

```js
    var MYAPP = {}; 
    MYAPP.dom = (function () { 
      var _setStyle = function (el, prop, value) { 
        console.log('setStyle'); 
      }; 
      var _getStyle = function (el, prop) { 
        console.log('getStyle'); 
      }; 
      return { 
        setStyle: _setStyle, 
        getStyle: _getStyle, 
        yetAnother: _setStyle 
      }; 
    }()); 

```

现在当你调用 `MYAPP.dom.setStyle()` 时，它将调用私有 `_setStyle()` 函数。你还可以如下从外部覆盖 `setStyle()`：

```js
    MYAPP.dom.setStyle = function () {alert('b');}; 

```

现在，结果是如下所示：

+   `MYAPP.dom.setStyle` 指向新的函数

+   `MYAPP.dom.yetAnother` 仍然指向 `_setStyle()`

+   `_setStyle()`始终可用，当任何其他内部代码依赖于它按预期工作，而不管外部代码如何

当你公开某些私有内容时，请记住对象（函数和数组也是对象）是通过引用传递的，因此可以从外部修改。

## 立即函数

另一种帮助你保持全局命名空间清洁的模式是将你的代码包裹在一个匿名函数中并立即执行该函数。这样，只要使用`var`语句，函数内的任何变量都是局部的，当函数返回时，如果它们不是闭包的一部分，就会被销毁。这种模式在第三章*函数*中进行了更详细的讨论。看看以下代码：

```js
    (function () { 
      // code goes here... 
    }()); 

```

这种模式特别适合在脚本加载时执行的开关初始化任务。

立即自执行的函数模式可以扩展为创建和返回对象。如果这些对象的创建更复杂且涉及一些初始化工作，那么你可以在自执行函数的第一部分完成这些工作，并返回一个可以访问和利用顶部部分任何私有属性的单个对象，如下所示：

```js
    var MYAPP = {}; 
    MYAPP.dom = (function () { 
      // initialization code... 
      function _private() { 
        // ...  
      } 
      return { 
        getStyle: function (el, prop) { 
          console.log('getStyle'); 
          _private(); 
        }, 
        setStyle: function (el, prop, value) { 
          console.log('setStyle'); 
        } 
      }; 
    }()); 

```

## 模块

将前面几种模式结合起来，可以得到一种新的模式，通常称为模块模式。在编程中，模块的概念很方便，因为它允许你编写独立的片段或库，并根据需要将它们组合起来，就像拼图一样。

模块模式包括以下内容：

+   命名空间用于减少模块之间的命名冲突

+   提供私有作用域和初始化的即时函数

+   私有属性和方法

### 注意

ES5 没有内置的模块概念。有来自[`www.commonjs.org`](http://www.commonjs.org)的模块规范，它定义了一个`require()`函数和一个 exports 对象。然而，ES6 支持模块。第八章详细介绍了模块。

+   返回具有模块公共 API 的对象，如下所示：

    ```js
            namespace('MYAPP.module.amazing'); 

            MYAPP.module.amazing = (function () { 

              // short names for dependencies 
              var another = MYAPP.module.another; 

              // local/private variables 
              var i, j; 

              // private functions 
              function hidden() {} 

              // public API 
              return { 
                hi: function () { 
                  return "hello"; 
                } 
              }; 
            }()); 

    ```

你可以使用模块如下所示：

```js
    MYAPP.module.amazing.hi(); // "hello" 

```

## 链式

链式是一种模式，允许你在一行中调用多个方法，就像链中的链接一样。当调用多个相关方法时，这很方便。你可以在前一个方法的结果上调用下一个方法，而不需要使用中间变量。

假设你创建了一个构造函数，帮助你处理 DOM 元素。创建一个新`<span>`标签并将其添加到`<body>`标签的代码可能看起来像以下这样：

```js
    var obj = new MYAPP.dom.Element('span'); 
    obj.setText('hello'); 
    obj.setStyle('color', 'red'); 
    obj.setStyle('font', 'Verdana'); 
    document.body.appendChild(obj); 

```

如你所知，构造函数通过`this`关键字返回它们创建的对象。你可以让你的方法，例如`setText()`和`setStyle()`，也返回`this`关键字，这允许你在前一个方法返回的实例上调用下一个方法。这样，你可以链式调用方法，如下所示：

```js
    var obj = new MYAPP.dom.Element('span'); 
    obj.setText('hello') 
       .setStyle('color', 'red') 
       .setStyle('font', 'Verdana'); 
    document.body.appendChild(obj); 

```

如果您在将新元素添加到树中后不打算使用 `obj` 变量，那么您甚至不需要这个变量，代码看起来如下所示：

```js
    document.body.appendChild( 
      new MYAPP.dom.Element('span') 
        .setText('hello') 
        .setStyle('color', 'red') 
        .setStyle('font', 'Verdana') 
    );    

```

这种模式的缺点是，当错误发生在长链中的某个地方，而您不知道哪个环节是问题所在，因为它们都在同一行时，这会使调试变得稍微困难一些。

## JSON

让我们用关于 JSON 的几句话来总结本章的编码模式部分。JSON 严格来说不是一种编码模式，但可以说使用它是良好的模式。

JSON 是一种流行的轻量级数据交换格式。在用 `XMLHttpRequest()` 从服务器获取数据时，它通常比 XML 更受欢迎。除了它极其方便之外，**JSON** 没有什么特别有趣的地方。JSON 格式由使用对象和数组字面量定义的数据组成。以下是一个 JSON 字符串的示例，您的服务器可以在 `XHR` 请求后响应：

```js
    { 
      'name':   'Stoyan', 
      'family': 'Stefanov', 
      'books':  ['OOJS', 'JSPatterns', 'JS4PHP'] 
    } 

```

相当于这个的 XML 将类似于以下代码片段：

```js
    <?xml version="1.1" encoding="iso-8859-1"?> 
    <response> 
      <name>Stoyan</name> 
      <family>Stefanov</family> 
      <books> 
        <book>OOJS</book> 
        <book>JSPatterns</book> 
        <book>JS4PHP</book> 
      </books> 
    </response> 

```

首先，您可以看到 JSON 在字节数量上更轻。然而，主要的好处不是更小的字节大小，而是它在 JavaScript 中处理 JSON 的简单性。假设，您已经发出了一个 `XHR` 请求，并在 `XHR` 对象的 `responseText` 属性中收到了一个 JSON 字符串。您可以通过简单地使用 `eval()` 将这串数据转换为可工作的 JavaScript 对象。考虑以下示例：

```js
    // warning: counter-example 
    var response = eval('(' + xhr.responseText + ')'); 

```

现在，您可以通过以下方式将 `obj` 中的数据作为对象属性访问：

```js
    console.log(response.name); // "Stoyan" 
    console.log(response.books[2]); // "JS4PHP" 

```

问题在于 `eval()` 是不安全的，因此最好使用 JSON 对象来解析 JSON 数据（对于旧浏览器，[`json.org/`](http://json.org/) 提供了备选方案）。从 JSON 字符串创建对象仍然很简单，如下所示：

```js
    var response = JSON.parse(xhr.responseText); 

```

要做相反的操作，即把对象转换为 JSON 字符串，您可以使用 `stringify()` 方法，如下所示：

```js
    var str = JSON.stringify({hello: "you"}); 

```

由于其简单性，JSON 很快成为了一种流行的、与语言无关的数据交换格式，您可以使用您喜欢的语言轻松地在服务器端生成 JSON。例如，在 PHP 中，有 `json_encode()` 和 `json_decode()` 函数，可以让您将 PHP 数组或对象序列化为 JSON 字符串，反之亦然。

## 高阶函数

函数式编程到目前为止仅限于有限的语言集合。随着更多语言添加支持函数式编程的功能，该领域的兴趣正在增长。JavaScript 正在演变以支持函数式编程的常见特性。你将逐渐看到大量以这种风格编写的代码。即使你现在还没有打算在代码中使用它，理解函数式编程风格也很重要。

高阶函数是函数式编程的重要支柱之一。高阶函数是指至少执行以下操作之一的函数：

+   接受一个或多个函数作为参数

+   返回一个函数作为结果

由于函数在 JavaScript 中是一等对象，因此将函数传递给函数并从函数返回函数是一件相当常规的事情。回调函数是高阶函数。让我们看看如何将这两个原则结合起来，编写一个高阶函数。

让我们编写一个`filter`函数；这个函数根据一个函数确定的准则从数组中过滤出值。这个函数接受两个参数——一个函数，它返回一个布尔值，`true`表示保留该元素。

例如，使用这个函数，我们正在过滤数组中的所有奇数。考虑以下代码行：

```js
    console.log([1, 2, 3, 4, 5].filter(function(ele){
      return ele % 2 == 0; })); 
    //[2,4] 

```

我们将一个匿名函数传递给`filter`函数作为第一个参数。这个函数根据一个条件返回布尔值，该条件检查元素是奇数还是偶数。

这是 ECMAScript 5 中添加的几个高阶函数之一。我们在这里试图说明的是，你将越来越多地在 JavaScript 中看到类似的用法模式。你必须首先理解高阶函数是如何工作的，然后，一旦你对这个概念感到舒适，尝试在你的代码中融入它们。

随着 ES6 函数语法的改变，编写高阶函数变得更加优雅。让我们以 ES5 中的一个简单例子来看一下它是如何转换为 ES6 等价的：

```js
    function add(x){ 
      return function(y){ 
        return y + x; 
      }; 
    } 
     var add3 = add(3); 
    console.log(add3(3));          // => 6 
    console.log(add(9)(10));       // => 19 

```

`add`函数接受`x`并返回一个函数，该函数接受`y`作为参数，然后返回表达式`y+x`的值。

当我们查看箭头函数时，我们讨论了箭头函数隐式返回单个表达式的结果。因此，前面的函数可以通过将箭头函数的主体变成另一个箭头函数来转换为箭头函数。看看下面的例子：

```js
    const add = x => y => y + x; 

```

在这里，我们有一个外部函数`x =>` [以`x`为参数的内部函数]，以及一个内部函数`y => y+x`。

这篇介绍将帮助你熟悉高阶函数日益增加的使用，以及它们在 JavaScript 中的重要性日益增加。

# 设计模式

本章的第二部分介绍了 JavaScript 对由影响深远的书籍《设计模式：可复用面向对象软件元素》中介绍的设计模式子集的方法，这本书通常被称为**四书**、**四人帮**或**GoF**（根据其四位作者命名）。在 GoF 书中讨论的模式分为以下三个组：

+   处理对象创建（实例化）的**创建模式**

+   描述不同对象如何组合以提供新功能的**结构模式**

+   描述对象之间通信方式的**行为模式**

在《四书》中，有 23 种模式，自该书出版以来，已经识别出更多模式。讨论所有这些模式超出了本书的范围，因此本章的剩余部分仅演示四种，以及它们在 JavaScript 中的实现示例。记住，这些模式更多地关于接口和关系，而不是实现。一旦你理解了一个设计模式，通常实现它并不困难，尤其是在像 JavaScript 这样的动态语言中。

本章其余部分讨论的模式如下：

+   单例

+   工厂

+   装饰器

+   观察者

## 单例模式

单例是一种创建型设计模式，这意味着它的重点是创建对象。当你想要确保只有一个给定种类或类的对象时，它很有帮助。在经典语言中，这意味着类的实例只创建一次，任何随后的尝试创建相同类的新的对象都会返回原始实例。

在 JavaScript 中，因为没有类，单例是默认且最自然的模式。每个对象都是单例对象。

JavaScript 中最基本的单例实现是对象字面量。看看以下代码行：

```js
    var single = {}; 

```

这很简单，对吧？

## 单例 2 模式

如果你想要使用类似类的语法并仍然实现单例模式，事情会变得有点更有趣。假设你有一个名为`Logger()`的构造函数，你想要能够做以下类似的事情：

```js
    var my_log = new Logger(); 
    my_log.log('some event'); 

    // ... 1000 lines of code later in a different scope ... 

    var other_log = new Logger(); 
    other_log.log('some new event'); 
    console.log(other_log === my_log); // true 

```

这个想法是，尽管你使用了`new`，但只需要创建一个实例，然后在连续调用中返回这个实例。

### 全局变量

一种方法是用全局变量来存储单个实例。你的构造函数可能看起来像以下代码片段：

```js
    function Logger() { 
      if (typeof global_log === "undefined") { 
        global_log = this; 
      } 
      return global_log; 
    } 

```

使用这个构造函数会得到预期的结果，如下所示：

```js
    var a = new Logger(); 
    var b = new Logger(); 
    console.log(a === b); // true 

```

显然，缺点是使用全局变量。它可以在任何时候被覆盖，甚至可能是意外地，你可能会丢失实例。相反，你的全局变量覆盖了别人的也是可能的。

### 构造函数的属性

如你所知，函数是对象，并且它们有属性。你可以将单例分配给构造函数的属性，如下所示：

```js
    function Logger() { 
      if (!Logger.single_instance) { 
        Logger.single_instance = this; 
      } 
      return Logger.single_instance; 
    } 

```

如果你写下`var a = new Logger()`，`a`指向新创建的`Logger.single_instance`属性。随后的`var b = new Logger()`调用会导致`b`指向同一个`Logger.single_instance`属性，这正是你想要的。

这种方法确实解决了全局命名空间问题，因为没有创建全局变量。唯一的缺点是`Logger`构造函数的属性是公开可见的，因此可以被随时覆盖。在这种情况下，单例可能会丢失或被修改。当然，你可以提供一定程度的保护来防止其他程序员自己给自己挖坑。毕竟，如果有人可以篡改单例属性，他们也可以直接篡改`Logger`构造函数。

### 在私有属性中

解决公开可见属性被覆盖问题的方法不是使用公开属性，而是使用私有属性。你已经知道如何使用闭包来保护变量，所以作为一个练习，你可以将这种方法应用到单例模式中。

## 工厂模式

工厂模式是另一种创建型设计模式，因为它涉及到对象的创建。当你有相似类型的对象，但事先不知道要使用哪一个时，工厂模式可以帮助你。根据用户输入或其他标准，你的代码可以动态确定所需的类型对象。

假设有三个不同的构造函数实现了类似的功能。它们创建的对象都接受一个 URL，但对其的处理方式不同。一个创建文本 DOM 节点；第二个创建链接；第三个创建图像，如下所示：

```js
    var MYAPP = {}; 
    MYAPP.dom = {}; 
    MYAPP.dom.Text = function (url) { 
      this.url = url; 
      this.insert = function (where) { 
        var txt = document.createTextNode(this.url); 
        where.appendChild(txt); 
      }; 
    }; 
    MYAPP.dom.Link = function (url) { 
      this.url = url; 
      this.insert = function (where) { 
        var link = document.createElement('a'); 
        link.href = this.url; 
        link.appendChild(document.createTextNode(this.url)); 
        where.appendChild(link); 
      }; 
    }; 
    MYAPP.dom.Image = function (url) { 
      this.url = url; 
      this.insert = function (where) { 
        var im = document.createElement('img'); 
        im.src = this.url; 
        where.appendChild(im); 
      }; 
    }; 

```

使用三个不同的构造函数完全相同——传递`url`变量并调用`insert()`方法，如下所示：

```js
    var url = 'http://www.phpied.com/images/covers/oojs.jpg'; 

    var o = new MYAPP.dom.Image(url); 
    o.insert(document.body); 

    var o = new MYAPP.dom.Text(url); 
    o.insert(document.body); 

    var o = new MYAPP.dom.Link(url); 
    o.insert(document.body); 

```

想象一下，你的程序事先并不知道需要哪种类型的对象。用户在运行时通过点击按钮等方式来决定。如果`type`包含所需的类型对象，你需要使用`if`或`switch`语句，并编写如下代码片段：

```js
    var o; 
    if (type === 'Image') { 
      o = new MYAPP.dom.Image(url); 
    } 
    if (type === 'Link') { 
      o = new MYAPP.dom.Link(url); 
    } 
    if (type === 'Text') { 
      o = new MYAPP.dom.Text(url); 
    } 
    o.url = 'http://...'; 
    o.insert(); 

```

这样做是可行的；然而，如果你有很多构造函数，代码会变得过长且难以维护。此外，如果你正在创建一个允许扩展或插件的库或框架，你甚至事先不知道所有构造函数的确切名称。在这种情况下，有一个负责创建动态确定类型对象的工厂函数是非常方便的。

让我们在`MYAPP.dom`实用工具中添加一个工厂方法：

```js
    MYAPP.dom.factory = function (type, url) { 
      return new MYAPP.domtype; 
    }; 

```

现在，你可以将三个`if`函数替换为更简单的代码，如下所示：

```js
    var image = MYAPP.dom.factory("Image", url); 
    image.insert(document.body); 

```

之前代码中的`factory()`方法很简单；然而，在实际场景中，你可能想要对类型值进行一些验证（例如，检查`MYAPP.dom[type]`是否存在）并可选地进行一些对所有对象类型都通用的设置工作（例如，设置所有构造函数使用的 URL）。

## 装饰器模式

装饰器设计模式是一种结构型模式；它与对象的创建方式关系不大，而是与如何扩展其功能有关。你不需要使用继承，在线性方式（父-子-孙）中扩展，你可以有一个基本对象和一个提供额外功能的装饰器对象池。你的程序可以选择它想要的装饰器，以及它们的顺序。对于不同的程序或代码路径，你可能有不同的要求，并从同一个池中挑选不同的装饰器。看看下面的代码片段，了解装饰器模式的用法部分是如何实现的：

```js
    var obj = { 
      doSomething: function () { 
        console.log('sure, asap'); 
      } 
      //  ... 
    }; 
    obj = obj.getDecorator('deco1'); 
    obj = obj.getDecorator('deco13'); 
    obj = obj.getDecorator('deco5'); 
    obj.doSomething(); 

```

你可以看到你可以从一个简单的具有`doSomething()`方法的对象开始。然后，你可以选择你拥有的一个装饰器对象，它可以被名称识别。所有装饰器都提供了一个`doSomething()`方法，该方法首先调用前一个装饰器的相同方法，然后继续执行自己的代码。每次添加装饰器时，你都会用改进后的版本覆盖基础`obj`。最后，当你完成添加装饰器后，你调用`doSomething()`。结果，所有装饰器的`doSomething()`方法都按顺序执行。让我们看看一个例子。

### 装饰圣诞树

让我们用一个装饰圣诞树的例子来说明装饰器模式。你可以从`decorate()`方法开始，如下所示：

```js
    var tree = {}; 
    tree.decorate = function () { 
      alert('Make sure the tree won't fall'); 
    }; 

```

现在，让我们实现一个`getDecorator()`方法，该方法添加额外的装饰器。装饰器将被实现为构造函数，并且它们都将从基本`tree`对象继承，如下所示：

```js
    tree.getDecorator = function (deco) { 
      tree[deco].prototype = this; 
      return new tree[deco]; 
    }; 

```

现在，让我们创建第一个装饰器`RedBalls()`，作为`tree`的一个属性，以保持全局命名空间更干净。红色球对象也提供了一个`decorate()`方法，但它们确保首先调用父对象的`decorate()`方法。例如，看看下面的代码：

```js
    tree.RedBalls = function () { 
      this.decorate = function () { 
        this.RedBalls.prototype.decorate(); 
        alert('Put on some red balls'); 
      }; 
    }; 

```

类似地，按照以下方式实现`BlueBalls()`和`Angel()`装饰器：

```js
    tree.BlueBalls = function () { 
      this.decorate = function () { 
        this.BlueBalls.prototype.decorate(); 
        alert('Add blue balls'); 
      }; 
    }; 
    tree.Angel = function () { 
      this.decorate = function () { 
        this.Angel.prototype.decorate(); 
        alert('An angel on the top'); 
      }; 
    }; 

```

现在，让我们将所有装饰器添加到基本对象中，如下面的代码片段所示：

```js
    tree = tree.getDecorator('BlueBalls'); 
    tree = tree.getDecorator('Angel'); 
    tree = tree.getDecorator('RedBalls'); 

```

最后，按照以下方式运行`decorate()`方法：

```js
    tree.decorate(); 

```

这单个调用会导致以下警报，具体顺序如下：

1.  确保树不会倒下。

1.  添加蓝色球。

1.  在顶部添加一个天使。

1.  添加一些红色球。

如你所见，这种功能允许你拥有你想要的任何数量的装饰器，并以任何你喜欢的方式选择和组合它们。

## 观察者模式

观察者模式，也称为**订阅者-发布者**模式，是一种行为型模式，这意味着它处理不同对象如何相互交互和通信。在实现观察者模式时，你有以下对象：

+   一个或多个发布者对象，在它们做重要的事情时宣布。

+   一个或多个订阅者调入了一个或多个发布者。他们倾听发布者宣布的内容，然后采取适当的行动。

观察者模式可能对您来说很熟悉。它听起来与上一章中讨论的浏览器事件很相似，这是有道理的，因为浏览器事件是这个模式的一个示例应用。浏览器是发布者；它宣布一个事件，如 `click` 已经发生的事实。当此类事件发生时，您订阅的用于监听此类事件的监听器函数将会收到通知。浏览器发布者向所有订阅者发送事件对象。在您的自定义实现中，您可以发送您认为合适的任何类型的数据。

观察者模式有两种子类型：推送和拉取。推送是发布者负责通知每个订阅者的情况，而拉取是订阅者监控发布者状态变化的情况。

让我们看看推送模型的一个示例实现。让我们将观察者相关的代码放在一个单独的对象中，然后使用这个对象作为混合对象，将其功能添加到任何决定成为发布者的其他对象中。这样，任何对象都可以成为发布者，任何函数都可以成为订阅者。观察者对象将具有以下属性和方法：

+   一个仅包含回调函数的 `subscribers` 数组

+   `addSubscriber()` 和 `removeSubscriber()` 方法，分别向 `subscribers` 集合添加和从中删除

+   一个 `publish()` 方法，它接受数据并调用所有订阅者，将数据传递给他们

+   一个 `make()` 方法，它接受任何对象并将其转换为发布者，通过向其添加之前提到的所有方法来实现

这是一个包含所有订阅相关方法的观察者混合对象，可以用来将任何对象转换为发布者：

```js
    var observer = { 
      addSubscriber: function (callback) { 
        if (typeof callback === "function") { 
          this.subscribers[this.subscribers.length] = callback; 
        } 
      }, 
      removeSubscriber: function (callback) { 
        for (var i = 0; i < this.subscribers.length; i++) { 
          if (this.subscribers[i] === callback) { 
            delete this.subscribers[i]; 
          } 
        } 
      }, 
      publish: function (what) { 
        for (var i = 0; i < this.subscribers.length; i++) { 
          if (typeof this.subscribers[i] === 'function') { 
            this.subscribersi; 
          } 
        } 
      }, 
      make: function (o) { // turns an object into a publisher 
        for (var i in this) { 
          if (this.hasOwnProperty(i)) { 
            o[i] = this[i]; 
            o.subscribers = []; 
          } 
        } 
      } 
   }; 

```

现在，让我们创建一些发布者。发布者可以是任何对象，它的唯一职责是在发生重要事件时调用 `publish()` 方法。以下是一个 `blogger` 对象，每次有新的博客文章准备就绪时都会调用 `publish()`：

```js
    var blogger = { 
      writeBlogPost: function() { 
        var content = 'Today is ' + new Date(); 
        this.publish(content); 
      } 
    }; 

```

另一个对象可以是洛杉矶时报报纸，当有新的报纸发行时调用 `publish()`。考虑以下代码行：

```js
    var la_times = { 
      newIssue: function() { 
        var paper = 'Martians have landed on Earth!'; 
        this.publish(paper); 
      } 
    }; 

```

您可以将这些对象转换为发布者，如下所示：

```js
    observer.make(blogger); 
    observer.make(la_times); 

```

现在，让我们考虑以下两个简单的对象，`jack` 和 `jill`：

```js
    var jack = { 
      read: function(what) { 
        console.log("I just read that " + what) 
      } 
    }; 
    var jill = { 
      gossip: function(what) { 
        console.log("You didn't hear it from me, but " + what) 
      } 
    }; 

```

`jack` 和 `jill` 对象可以通过提供它们希望在发布时调用的回调方法来订阅 `blogger` 对象，如下所示：

```js
    blogger.addSubscriber(jack.read); 
    blogger.addSubscriber(jill.gossip); 

```

那么，当 `blogger` 对象撰写新文章时会发生什么？结果是 `jack` 和 `jill` 将会收到通知：

```js
    > blogger.writeBlogPost(); 
       I just read that Today is Fri Jan 04 2013 19:02:12 GMT-0800 (PST) 
       You didn't hear it from me, but Today is Fri Jan 04 2013 19:02:12 GMT-0800    
         (PST) 

```

在任何时候，`jill` 都可以决定取消她的订阅。然后，在撰写另一篇博客文章时，未订阅的对象将不再收到通知。考虑以下代码片段：

```js
    > blogger.removeSubscriber(jill.gossip); 
    > blogger.writeBlogPost();
    I just read that Today is Fri Jan 04 2013 19:03:29 GMT-0800 (PST) 

```

`jill` 对象可能会决定订阅洛杉矶时报，因为一个对象可以是多个发布者的订阅者，如下所示：

```js
    > la_times.addSubscriber(jill.gossip); 

```

然后，当洛杉矶时报发布新的一期时，`jill` 会收到通知，并执行 `jill.gossip()`，如下所示：

```js
    > la_times.newIssue();
    You didn't hear it from me, but Martians have landed on Earth! 

```

# 摘要

在本章中，你学习了常见的 JavaScript 编程模式，并了解了如何让你的程序更加整洁、快速，并且更好地与其他程序和库协同工作。然后，你看到了关于《四书》中一些设计模式的讨论和示例实现。你可以看到 JavaScript 是一个功能齐全的动态编程语言，在动态弱类型语言中实现经典模式相当容易。这些模式通常是一个大主题，你可以加入本书作者的行列，在 [JSPatterns.com](http://www.jspatterns.com/) 上进一步讨论 JavaScript 模式，或者查看《JavaScript Patterns》这本书。下一章将重点介绍测试和调试方法。
