# 第十一章。编码和设计模式

既然您已经了解了 JavaScript 中的所有对象，掌握了原型和继承，并看到了使用特定于浏览器的对象的一些实际示例，让我们继续前进，或者说，向上移动一级。让我们来看看一些常见的 JavaScript 模式。

但首先，什么是模式？简而言之，模式是对常见问题的良好解决方案。将解决方案编码为模式使其可重复使用。

有时，当您面对一个新的编程问题时，您可能立即意识到您以前解决过另一个非常相似的问题。在这种情况下，值得将这类问题隔离出来，并寻找一个共同的解决方案。模式是一种经过验证和可重复使用的解决方案（或解决方案的方法）。

有时，模式只是一个想法或一个名称。有时，仅仅使用一个名称可以帮助您更清晰地思考问题。此外，在团队中与其他开发人员合作时，当每个人使用相同的术语讨论问题或解决方案时，沟通会更容易。

有时，您可能会遇到一个独特的问题，看起来与您以前见过的任何东西都不一样，并且不容易适应已知的模式。盲目地应用模式只是为了使用模式，这不是一个好主意。最好不要使用任何已知的模式，而是尝试调整问题，使其适应现有的解决方案。

本章讨论了以下两种模式：

+   **编码模式**：这些主要是 JavaScript 特定的最佳实践

+   **设计模式**：这些是与语言无关的模式，由著名的*四人帮*书籍推广

# 编码模式

让我们从一些反映 JavaScript 独特特性的模式开始。一些模式旨在帮助您组织代码，例如命名空间；其他与改进性能有关，例如延迟定义和初始化时分支；还有一些弥补了缺失的功能，例如私有属性。本节讨论的模式包括以下主题：

+   分离行为

+   命名空间

+   初始化时分支

+   延迟定义

+   配置对象

+   私有变量和方法

+   特权方法

+   将私有函数作为公共方法

+   立即函数

+   链接

+   JSON

## 分离行为

如前所述，网页的三个构建块如下：

+   内容（HTML）

+   演示（CSS）

+   行为（JavaScript）

### 内容

HTML 是网页的内容，实际文本。理想情况下，内容应该使用尽可能少的 HTML 标记进行标记，以充分描述该内容的语义含义。例如，如果您正在处理导航菜单，最好使用`<ul>`和`<li>`标记，因为导航菜单本质上只是一个链接列表。

您的内容（HTML）应该不包含任何格式化元素。视觉格式应属于演示层，并且应通过**CSS**（层叠样式表）来实现。这意味着以下内容：

+   如果可能的话，不应该使用 HTML 标记的样式属性。

+   根本不应该使用`<font>`等呈现 HTML 标签。

+   标记应该根据其语义含义使用，而不是因为浏览器默认呈现它们。例如，开发人员有时会在更适合使用`<p>`的地方使用`<div>`标记。使用`<strong>`和`<em>`而不是`<b>`和`<i>`也是有利的，因为后者描述的是视觉呈现而不是含义。

### 演示

将演示内容与内容分开的一个好方法是重置或清空所有浏览器默认设置，例如使用来自 Yahoo! UI 库的`reset.css`。这样，浏览器的默认呈现不会让您分心，而是会让您有意识地考虑使用适当的语义标记。

### 行为

网页的第三个组件是行为。行为应该与内容和表现分开。通常使用隔离在`<script>`标签中的 JavaScript 来添加，最好包含在外部文件中。这意味着不使用任何内联属性，如`onclick`，`onmouseover`等。相反，您可以使用上一章中的`addEventListener`/`attachEvent`方法。

将行为与内容分离的最佳策略如下：

+   最小化`<script>`标签的数量

+   避免内联事件处理程序

+   不要使用 CSS 表达式

+   在内容的末尾，当您准备关闭`<body>`标签时，插入一个`external.js`文件

#### 行为分离示例

假设您在页面上有一个搜索表单，并且希望使用 JavaScript 验证表单。因此，您可以继续保持`form`标签不受任何 JavaScript 的影响，然后在关闭`</body>`标签之前立即插入一个链接到外部文件的`<script>`标签，如下所示：

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
      <script src="behaviors.js"></script> 
    </body> 

```

在`behaviors.js`中，您可以将事件侦听器附加到提交事件。在您的侦听器中，您可以检查文本输入字段是否为空，如果是，则阻止表单提交。这样，您将节省服务器和客户端之间的往返，并使应用程序立即响应。

`behaviors.js`的内容如下所示。它假定您已经根据上一章的练习创建了您的`myevent`实用程序：

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

您注意到脚本是在 HTML 结束前加载的，就在关闭 body 之前。原因是 JavaScript 会阻止页面的 DOM 构建，并且在某些浏览器中，甚至会阻止后续组件的下载。通过将脚本移动到页面底部，您可以确保脚本不会妨碍，并且当它到达时，它只是增强了已经可用的页面。

防止外部 JavaScript 文件阻止页面的另一种方法是异步加载它们。这样您可以更早地开始加载它们。HTML5 具有此目的的`defer`属性。请考虑以下代码行：

```js
    <script defer src="behaviors.js"></script> 

```

不幸的是，`defer`属性不受旧版浏览器支持，但幸运的是，有一个可以跨浏览器（新旧）工作的解决方案。解决方案是动态创建一个`script`节点并将其附加到 DOM。换句话说，您可以使用一点内联 JavaScript 来加载外部 JavaScript 文件。您可以在文档顶部放置此脚本加载程序片段，以便下载可以尽早开始。请看以下代码示例：

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

应避免全局变量以减少变量命名冲突的可能性。通过为变量和函数命名空间化，您可以最小化全局变量的数量。这个想法很简单，您只会创建一个全局对象，而您的所有其他变量和函数都成为该对象的属性。

### 对象作为命名空间

让我们创建一个名为`MYAPP`的全局对象：

```js
    // global namespace 
    var MYAPP = MYAPP || {}; 

```

现在，不再需要全局的`myevent`实用程序（来自上一章），您可以将其作为`MYAPP`对象的`event`属性，如下所示：

```js
    // sub-object 
    MYAPP.event = {}; 

```

向`event`实用程序添加方法仍然是相同的。请考虑以下示例：

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

使用命名空间不妨碍您创建构造函数。以下是如何创建具有`Element`构造函数的 DOM 实用程序，它允许您轻松创建 DOM 元素：

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

类似地，您可以有一个`Text`构造函数来创建文本节点。请考虑以下代码示例：

```js
    MYAPP.dom.Text = function (txt) { 
      return document.createTextNode(txt); 
    }; 

```

使用构造函数在页面底部创建链接可以按以下方式完成：

```js
    var link = new MYAPP.dom.Element('a',  
      {href: 'http://phpied.com', target: '_blank'}); 
    var text = new MYAPP.dom.Text('click me'); 
    link.appendChild(text); 
    document.body.appendChild(link); 

```

### 一个命名空间()方法

您可以创建一个命名空间实用程序，使您的生活更轻松，以便您可以使用更方便的语法，如下所示：

```js
    MYAPP.namespace('dom.style'); 

```

而不是更冗长的语法如下：

```js
    MYAPP.dom = {}; 
    MYAPP.dom.style = {}; 

```

以下是如何创建`namespace()`方法的方法。首先，您将使用句点（`.`）作为分隔符拆分输入字符串，创建一个数组。然后，对于新数组中的每个元素，如果全局对象中不存在该属性，则添加一个属性，如下所示：

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

通过以下方式进行新方法的测试：

```js
    MYAPP.namespace('event'); 
    MYAPP.namespace('dom.style'); 

```

前面代码的结果与以下操作相同：

```js
    var MYAPP = { 
      event: {}, 
      dom: { 
        style: {} 
      } 
    }; 

```

## 初始化时分支

在前一章中，您注意到有时不同的浏览器对相同或类似的功能有不同的实现。在这种情况下，您需要根据当前执行脚本的浏览器支持的内容对代码进行分支。根据您的程序，这种分支可能会发生得太频繁，结果可能会减慢脚本的执行速度。

您可以通过在初始化时对代码的某些部分进行分支来缓解这个问题，当脚本加载时，而不是在运行时。借助动态定义函数的能力，您可以根据浏览器的不同分支和定义相同的函数，具体取决于浏览器。让我们看看如何。

首先，让我们定义一个命名空间和`event`实用程序的占位符方法。

```js
    var MYAPP = {}; 
    MYAPP.event = { 
      addListener: null, 
      removeListener: null 
    }; 

```

此时，添加或删除侦听器的方法尚未实现。根据特性嗅探的结果，可以以不同的方式定义这些方法，如下所示：

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

脚本执行后，您将以与浏览器相关的方式定义`addListener()`和`removeListener()`方法。现在，每次调用这些方法时，都不再需要特性嗅探，这将减少工作量并加快执行速度。

在嗅探特性时要注意的一点是，在检查一个特性后不要假设太多。在前面的示例中，这条规则被打破了，因为代码只检查了`addEventListener`的支持，但随后定义了`addListener()`和`removeListener()`。在这种情况下，可以假设如果浏览器实现了`addEventListener()`，那么它也实现了`removeEventListener()`。然而，想象一下，如果浏览器实现了`stopPropagation()`但没有实现`preventDefault()`，而您没有单独检查这些情况会发生什么。您假设因为`addEventListener()`未定义，浏览器必须是一个旧的 IE，并使用您对 IE 工作方式的知识和假设来编写代码。请记住，您所有的知识都是基于某个浏览器今天的工作方式，但不一定是明天的工作方式。因此，为了避免在新的浏览器版本发布时多次重写代码，最好单独检查您打算使用的特性，并不要对某个浏览器支持的特性进行概括。

## 懒惰定义

懒惰定义模式类似于先前的初始化时分支模式。不同之处在于分支只会在第一次调用函数时发生。当调用函数时，它会使用最佳实现重新定义自身。与初始化时分支不同，初始化时分支只发生一次，在加载时，而在这里，当函数从未被调用时，可能根本不会发生。懒惰定义还使初始化过程更轻松，因为不需要进行初始化时分支工作。

让我们通过定义一个`addListener()`函数的示例来说明这一点。首先，该函数使用通用的主体进行定义。当首次调用函数时，它会检查浏览器支持的功能，然后使用最合适的实现重新定义自身。在第一次调用结束时，函数会调用自身，以便执行实际的事件附加。下次调用相同的函数时，它将使用新的主体进行定义，并准备好使用，因此不需要进一步的分支。以下是代码片段：

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

当您有一个接受许多可选参数的函数或方法时，这种模式很方便。由您决定多少个构成了很多。但一般来说，一个具有三个以上参数的函数不方便调用，因为您必须记住参数的顺序，当一些参数是可选的时，这更加不方便。

而不是有许多参数，您可以使用一个参数并将其设置为对象。对象的属性是实际参数。这适用于传递配置选项，因为这些 tend to be numerous and optional (with smart defaults). 使用单个对象而不是多个参数的美妙之处如下所述：

+   顺序无关紧要

+   您可以轻松跳过不想设置的参数

+   很容易添加更多的可选配置属性

+   它使代码更易读，因为配置对象的属性与它们的名称一起出现在调用代码中

想象一下，您有一些 UI 小部件构造函数，用于创建漂亮的按钮。它接受要放在按钮内部的文本（`<input>`标签的`value`属性）以及`type`按钮的可选参数。为简单起见，让我们假设漂亮的按钮采用与常规按钮相同的配置。看一下以下代码：

```js
    // a constructor that creates buttons 
    MYAPP.dom.FancyButton = function (text, type) { 
      var b = document.createElement('input'); 
      b.type = type || 'submit'; 
      b.value = text; 
      return b; 
    }; 

```

使用构造函数很简单；您只需给它一个字符串。然后，您可以将新按钮添加到文档的主体中，如下所示：

```js
    document.body.appendChild( 
      new MYAPP.dom.FancyButton('puuush') 
    ); 

```

这一切都很好，运行良好，但是然后您决定还想能够设置按钮的一些样式属性，比如颜色和字体。您最终可能会得到以下定义：

```js
    MYAPP.dom.FancyButton =  
      function (text, type, color, border, font) { 
      // ... 
    }; 

```

现在，使用构造函数可能会变得有点不方便，特别是当您想设置第三个和第五个参数，但不想设置第二个或第四个时。考虑以下示例：

```js
    new MYAPP.dom.FancyButton( 
      'puuush', null, 'white', null, 'Arial'); 

```

更好的方法是使用一个`config`对象参数来设置所有的设置。函数定义可以变成以下代码片段：

```js
    MYAPP.dom.FancyButton = function (text, conf) { 
      var type = conf.type || 'submit'; 
      var font = conf.font || 'Verdana'; 
      // ... 
    }; 

```

使用构造函数如下所示：

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

如您所见，设置只有一些参数并且切换它们的顺序很容易。此外，当您在调用方法的地方看到参数的名称时，代码更友好，更易于理解。

这种模式的缺点与其优点相同。很容易不断添加更多的参数，这意味着滥用这种技术很容易。一旦您有理由向这个自由的属性包中添加更多内容，您会发现很容易不断添加一些并非完全可选的属性，或者一些依赖于其他属性的属性。

作为一个经验法则，所有这些属性都应该是独立的和可选的。如果您必须在函数内部检查所有可能的组合（“哦，A 已设置，但只有在 B 也设置了 A 才会被使用”），这将导致一个庞大的函数体，很快就会变得令人困惑和难以理解，甚至是不可能测试，因为所有的组合。

## 私有属性和方法

JavaScript 没有访问修饰符的概念，它设置对象中属性的特权。其他语言通常有访问修饰符，如下所示：

+   `Public`: 对象的所有用户都可以访问这些属性或方法

+   `Private`: 只有对象本身才能访问这些属性

+   `Protected`: 只有继承所讨论的对象的对象才能访问这些属性

JavaScript 没有特殊的语法来表示私有属性或方法，但如第三章中所讨论的 *函数*，您可以在函数内部使用局部变量和方法，并实现相同级别的保护。

继续使用`FancyButton`构造函数的示例，您可以有一个包含所有默认值的本地变量 styles 和一个本地的`setStyle()`函数。这些对于构造函数外部的代码是不可见的。以下是`FancyButton`如何利用本地私有属性：

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

在此实现中，`styles`是一个私有属性，`setStyle()`是一个私有方法。构造函数在内部使用它们（它们可以访问构造函数内部的任何内容），但它们对函数外部的代码不可用。

## 特权方法

特权方法（这个术语是由 Douglas Crockford 创造的）是可以访问私有方法或属性的普通公共方法。它们可以充当桥梁，以受控的方式包装特定的私有功能，使其可访问。

## 私有函数作为公共方法

假设您已经定义了一个绝对需要保持完整的函数，因此将其设置为私有。但是，您还希望提供对相同函数的访问权限，以便外部代码也可以从中受益。在这种情况下，您可以将私有函数分配给公开可用的属性。

让我们将`_setStyle()`和`_getStyle()`定义为私有函数，然后将它们分配给公共的`setStyle()`和`getStyle()`，考虑以下示例：

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

现在，当您调用`MYAPP.dom.setStyle()`时，它会调用私有的`_setStyle()`函数。您也可以从外部覆盖`setStyle()`如下：

```js
    MYAPP.dom.setStyle = function () {alert('b');}; 

```

现在，结果如下：

+   `MYAPP.dom.setStyle`指向新函数

+   `MYAPP.dom.yetAnother`仍然指向`_setStyle()`

+   `_setStyle()`在任何其他内部代码依赖它按预期工作时始终可用，而不受外部代码的影响

当您公开私有内容时，请记住对象（函数和数组也是对象）是通过引用传递的，因此可以从外部修改。

## 立即函数

帮助您保持全局命名空间清晰的另一种模式是将代码包装在匿名函数中并立即执行该函数。这样，只要使用`var`语句，函数内部的任何变量都是局部的，并且在函数返回时被销毁，如果它们不是闭包的一部分。这种模式在第三章*函数*中有更详细的讨论。看一下以下代码：

```js
    (function () { 
      // code goes here... 
    }()); 

```

此模式特别适用于一次性初始化任务，在脚本加载时执行。

立即自执行函数模式可以扩展到创建和返回对象。如果创建这些对象更复杂并涉及一些初始化工作，那么您可以在自执行函数的第一部分中执行此操作，并返回一个可以访问和受益于顶部私有属性的单个对象，如下所示：

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

结合前面几种模式可以得到一个新模式，通常称为模块模式。编程中的模块概念很方便，因为它允许您编写单独的代码片段或库，并根据需要组合它们，就像拼图一样。

模块模式包括以下内容：

+   命名空间以减少模块之间的命名冲突

+   立即函数提供私有作用域和初始化

+   私有属性和方法

### 注意

ES5 没有内置的模块概念。有来自[`www.commonjs.org`](http://www.commonjs.org)的模块规范，它定义了一个`require()`函数和一个 exports 对象。然而，ES6 支持模块。第八章类和模块已经详细介绍了模块。

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

而且，您可以以以下方式使用模块：

```js
    MYAPP.module.amazing.hi(); // "hello" 

```

## 链接

链接是一种模式，允许你在一行上调用多个方法，就好像这些方法是链条中的链接一样。当调用几个相关的方法时，这是很方便的。你在前一个方法的结果上调用下一个方法，而不使用中间变量。

假设你已经创建了一个构造函数，可以帮助你处理 DOM 元素。创建一个新的添加到`<body>`标签的`<span>`标签的代码可能如下所示：

```js
    var obj = new MYAPP.dom.Element('span'); 
    obj.setText('hello'); 
    obj.setStyle('color', 'red'); 
    obj.setStyle('font', 'Verdana'); 
    document.body.appendChild(obj); 

```

如你所知，构造函数返回所谓的`this`关键字所创建的对象。你可以让你的方法，比如`setText()`和`setStyle()`，也返回`this`关键字，这样你就可以在前一个方法返回的实例上调用下一个方法。这样，你可以链式调用方法，如下所示：

```js
    var obj = new MYAPP.dom.Element('span'); 
    obj.setText('hello') 
       .setStyle('color', 'red') 
       .setStyle('font', 'Verdana'); 
    document.body.appendChild(obj); 

```

如果你在新元素添加到树之后不打算使用`obj`变量，那么代码看起来像下面这样：

```js
    document.body.appendChild( 
      new MYAPP.dom.Element('span') 
        .setText('hello') 
        .setStyle('color', 'red') 
        .setStyle('font', 'Verdana') 
    );    

```

这种模式的一个缺点是，当长链中的某个地方发生错误时，它会使得调试变得有点困难，因为你不知道哪个链接有问题，因为它们都在同一行上。

## JSON

让我们用几句话来总结本章的编码模式部分关于 JSON 的内容。JSON 在技术上并不是一个编码模式，但你可以说使用它是一个很好的模式。

JSON 是一种流行的轻量级数据交换格式。在使用`XMLHttpRequest()`从服务器检索数据时，它通常优先于 XML。**JSON**除了它极其方便之外，没有什么特别有趣的地方。JSON 格式由使用对象和数组文字定义的数据组成。以下是一个 JSON 字符串的示例，你的服务器可以在`XHR`请求之后用它来响应：

```js
    { 
      'name':   'Stoyan', 
      'family': 'Stefanov', 
      'books':  ['OOJS', 'JSPatterns', 'JS4PHP'] 
    } 

```

这个的 XML 等价物将是以下代码片段：

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

首先，你可以看到 JSON 在字节数量上更轻。然而，主要好处不是较小的字节大小，而是在 JavaScript 中使用 JSON 非常简单。比如，你已经发出了一个`XHR`请求，并在`XHR`对象的`responseText`属性中收到了一个 JSON 字符串。你可以通过简单地使用`eval()`将这个数据字符串转换为一个可用的 JavaScript 对象。考虑以下示例：

```js
    // warning: counter-example 
    var response = eval('(' + xhr.responseText + ')'); 

```

现在，你可以像下面这样访问`obj`中的数据作为对象属性：

```js
    console.log(response.name); // "Stoyan" 
    console.log(response.books[2]); // "JS4PHP" 

```

问题在于`eval()`是不安全的，所以最好使用 JSON 对象来解析 JSON 数据（旧版浏览器的备用方案可在[`json.org/`](http://json.org/)找到）。从 JSON 字符串创建对象仍然很简单，如下所示：

```js
    var response = JSON.parse(xhr.responseText); 

```

要做相反的事情，也就是将对象转换为 JSON 字符串，你可以使用`stringify()`方法，如下所示：

```js
    var str = JSON.stringify({hello: "you"}); 

```

由于其简单性，JSON 很快就成为了一种独立于语言的数据交换格式，并且你可以使用你喜欢的语言在服务器端轻松地生成 JSON。例如，在 PHP 中，有`json_encode()`和`json_decode()`函数，让你将 PHP 数组或对象序列化为 JSON 字符串，反之亦然。

## 高阶函数

到目前为止，函数式编程一直局限于有限的一组语言。随着越来越多的语言添加支持函数式编程的特性，人们对这一领域的兴趣正在增长。JavaScript 正在发展以支持函数式编程的常见特性。你将逐渐看到很多以这种风格编写的代码。重要的是要理解函数式编程风格，即使你现在还不想在你的代码中使用它。

高阶函数是函数式编程的重要支柱之一。高阶函数是至少做以下一种事情的函数：

+   以一个或多个函数作为参数

+   返回一个函数作为结果

由于 JavaScript 中函数是一等对象，因此将函数传递给函数并从函数返回函数是一件相当常见的事情。回调函数是高阶函数。让我们看看如何将这两个原则结合起来编写一个高阶函数。

让我们编写一个`filter`函数；这个函数根据由函数确定的条件从数组中过滤出值。这个函数接受两个参数-一个返回布尔值`true`以保留此元素的函数。

例如，使用这个函数，我们正在从数组中过滤出所有奇数值。考虑以下代码行：

```js
    console.log([1, 2, 3, 4, 5].filter(function(ele){
      return ele % 2 == 0; })); 
    //[2,4] 

```

我们将一个匿名函数作为第一个参数传递给`filter`函数。这个函数根据一个条件返回一个布尔值，检查元素是奇数还是偶数。

这是 ECMAScript 5 中添加的几个高阶函数之一的示例。我们试图表达的观点是，您将越来越多地看到 JavaScript 中类似的使用模式。您必须首先了解高阶函数的工作原理，然后，一旦您对概念感到舒适，尝试在您的代码中也加入它们。

随着 ES6 函数语法的变化，编写高阶函数变得更加优雅。让我们以 ES5 中的一个小例子来看看它如何转换为 ES6：

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

`add`函数接受`x`并返回一个接受`y`作为参数的函数，然后返回表达式`y+x`的值。

当我们讨论箭头函数时，我们讨论了箭头函数隐式返回单个表达式的结果。因此，前面的函数可以通过将箭头函数的主体变为另一个箭头函数来转换为箭头函数。看看下面的例子：

```js
    const add = x => y => y + x; 

```

在这里，我们有一个外部函数，`x =>` [带有`x`作为参数的内部函数]，以及一个内部函数，`y => y+x`。

这个介绍将帮助您熟悉高阶函数的增加使用，以及它们在 JavaScript 中的增加重要性。

# 设计模式

本章的第二部分介绍了 JavaScript 对《设计模式：可复用面向对象软件的元素》中引入的设计模式子集的方法，这是一本有影响力的书，通常被称为《四人帮》或《GoF》（四位作者的缩写）。《GoF》书中讨论的模式分为以下三组：

+   处理对象如何创建（实例化）的创建模式

+   描述不同对象如何组合以提供新功能的结构模式

+   描述对象之间通信方式的行为模式

《四人帮》中有 23 种模式，自该书出版以来已经发现了更多模式。讨论所有这些模式远远超出了本书的范围，因此本章的其余部分仅演示了四种模式，以及它们在 JavaScript 中的实现示例。请记住，这些模式更多关于接口和关系而不是实现。一旦您了解了设计模式，通常很容易实现它，特别是在 JavaScript 这样的动态语言中。

本章剩余部分讨论的模式如下：

+   单例

+   工厂

+   装饰器

+   观察者

## 单例模式

单例是一种创建型设计模式，意味着它的重点是创建对象。当您想要确保只有一个给定种类或类的对象时，它会帮助您。在经典语言中，这意味着只创建一个类的实例，并且任何后续尝试创建相同类的新对象都将返回原始实例。

在 JavaScript 中，由于没有类，单例是默认和最自然的模式。每个对象都是单例对象。

JavaScript 中单例的最基本实现是对象字面量。看一下下面的代码行：

```js
    var single = {}; 

```

那很容易，对吧？

## 单例 2 模式

如果您想使用类似类的语法并且仍然实现单例模式，事情会变得更有趣一些。假设您有一个名为`Logger()`的构造函数，并且希望能够执行以下操作：

```js
    var my_log = new Logger(); 
    my_log.log('some event'); 

    // ... 1000 lines of code later in a different scope ... 

    var other_log = new Logger(); 
    other_log.log('some new event'); 
    console.log(other_log === my_log); // true 

```

思想是，尽管使用了`new`，但只需要创建一个实例，然后在连续调用中返回该实例。

### 全局变量

一种方法是使用全局变量来存储单个实例。您的构造函数可能如下代码片段所示：

```js
    function Logger() { 
      if (typeof global_log === "undefined") { 
        global_log = this; 
      } 
      return global_log; 
    } 

```

使用此构造函数会产生预期的结果，如下所示：

```js
    var a = new Logger(); 
    var b = new Logger(); 
    console.log(a === b); // true 

```

缺点显而易见，就是使用全局变量。它可以在任何时候被意外覆盖，您可能会丢失实例。相反，覆盖别人的全局变量也是可能的。

### 构造函数的属性

如您所知，函数是对象，它们有属性。您可以将单个实例分配给构造函数的属性，如下所示：

```js
    function Logger() { 
      if (!Logger.single_instance) { 
        Logger.single_instance = this; 
      } 
      return Logger.single_instance; 
    } 

```

如果您编写`var a = new Logger()`，`a`指向新创建的`Logger.single_instance`属性。随后的`var b = new Logger()`调用会导致`b`指向相同的`Logger.single_instance`属性，这正是您想要的。

这种方法确实解决了全局命名空间问题，因为不会创建全局变量。唯一的缺点是`Logger`构造函数的属性是公开可见的，因此可以随时被覆盖。在这种情况下，单个实例可能会丢失或修改。当然，您只能提供有限的保护，以防止其他程序员自食其力。毕竟，如果有人可以干扰单实例属性，他们也可以直接干扰`Logger`构造函数。

### 在私有属性中

解决公开可见属性被覆盖的问题的方法不是使用公共属性，而是使用私有属性。您已经知道如何使用闭包保护变量，因此作为练习，您可以实现这种方法来实现单例模式。

## 工厂模式

工厂是另一种创建型设计模式，因为它涉及创建对象。当您有类似类型的对象并且事先不知道要使用哪个时，工厂可以帮助您。根据用户输入或其他条件，您的代码可以动态确定所需的对象类型。

假设您有三种不同的构造函数，实现类似的功能。它们创建的所有对象都需要一个 URL，但对其执行不同的操作。一个创建文本 DOM 节点；第二个创建一个链接；第三个创建一个图像，如下所示：

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

使用三种不同的构造函数完全相同-传递`url`变量并调用`insert()`方法，如下所示：

```js
    var url = 'http://www.phpied.com/images/covers/oojs.jpg'; 

    var o = new MYAPP.dom.Image(url); 
    o.insert(document.body); 

    var o = new MYAPP.dom.Text(url); 
    o.insert(document.body); 

    var o = new MYAPP.dom.Link(url); 
    o.insert(document.body); 

```

想象一下，您的程序事先不知道需要哪种类型的对象。用户在运行时通过单击按钮等方式决定。如果`type`包含所需的对象类型，则需要使用`if`或`switch`语句，并编写以下代码片段：

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

这样做效果很好；但是，如果您有很多构造函数，代码会变得太长且难以维护。此外，如果您正在创建允许扩展或插件的库或框架，您甚至不知道所有构造函数的确切名称。在这种情况下，有一个工厂函数来负责创建动态确定类型的对象是很方便的。

让我们向`MYAPP.dom`实用程序添加一个工厂方法：

```js
    MYAPP.dom.factory = function (type, url) { 
      return new MYAPP.domtype; 
    }; 

```

现在，您可以用更简单的代码替换三个`if`函数，如下所示：

```js
    var image = MYAPP.dom.factory("Image", url); 
    image.insert(document.body); 

```

先前代码中的示例`factory()`方法很简单；但是，在实际情况下，您可能希望针对类型值进行一些验证（例如，检查`MYAPP.dom[type]`是否存在），并且可能对所有对象类型进行一些通用的设置工作（例如，设置所有构造函数使用的 URL）。

## 装饰器模式

装饰者设计模式是一种结构模式；它与对象如何创建没有太多关系，而是与它们的功能如何扩展有关。你可以有一个基础对象和一组不同的装饰者对象，它们提供额外的功能，而不是使用继承，继承是线性的（父-子-孙），你的程序可以选择想要的装饰者，以及顺序。对于不同的程序或代码路径，你可能有不同的需求集，并从同一个池中选择不同的装饰者。看一下以下代码片段，看看装饰者模式的使用部分如何实现：

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

你可以看到如何从一个具有`doSomething()`方法的简单对象开始。然后，你可以选择你手头上的一个装饰者对象，并通过名称进行识别。所有装饰者都提供一个`doSomething()`方法，首先调用前一个装饰者的相同方法，然后继续执行自己的代码。每次添加一个装饰者，都会用改进版本的`obj`覆盖基础对象。最后，当你添加完装饰者后，调用`doSomething()`。结果，所有装饰者的`doSomething()`方法都按顺序执行。让我们看一个例子。

### 装饰一棵圣诞树

让我们用一个装饰一棵圣诞树的例子来说明装饰者模式。你可以按照以下方式开始`decorate()`方法：

```js
    var tree = {}; 
    tree.decorate = function () { 
      alert('Make sure the tree won't fall'); 
    }; 

```

现在，让我们实现一个`getDecorator()`方法，添加额外的装饰者。装饰者将作为构造函数实现，并且它们都将从基础`tree`对象继承，如下所示：

```js
    tree.getDecorator = function (deco) { 
      tree[deco].prototype = this; 
      return new tree[deco]; 
    }; 

```

现在，让我们创建第一个装饰者`RedBalls()`，作为`tree`的属性，以保持全局命名空间更清洁。红色球对象也提供一个`decorate()`方法，但它们确保首先调用它们父级的`decorate()`。例如，看一下以下代码：

```js
    tree.RedBalls = function () { 
      this.decorate = function () { 
        this.RedBalls.prototype.decorate(); 
        alert('Put on some red balls'); 
      }; 
    }; 

```

同样，按照以下方式实现`BlueBalls()`和`Angel()`装饰者：

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

现在，让我们将所有装饰者添加到基础对象中，如下所示的代码片段：

```js
    tree = tree.getDecorator('BlueBalls'); 
    tree = tree.getDecorator('Angel'); 
    tree = tree.getDecorator('RedBalls'); 

```

最后，按照以下方式运行`decorate()`方法：

```js
    tree.decorate(); 

```

这个单一的调用会导致以下警报，具体顺序如下：

1.  确保树不会倒下。

1.  添加蓝色的球。

1.  在顶部添加一个天使。

1.  添加一些红色的球。

正如你所看到的，这个功能允许你拥有任意数量的装饰者，并以任意方式选择和组合它们。

## 观察者模式

观察者模式，也称为**订阅者-发布者**模式，是一种行为模式，意味着它处理不同对象之间的交互和通信。在实现观察者模式时，你会有以下对象：

+   一个或多个发布者对象，它们在做重要事情时会宣布。

+   一个或多个订阅者调整到一个或多个发布者。他们听取发布者的宣布然后采取适当的行动。

观察者模式可能对你来说很熟悉。它听起来与前一章讨论的浏览器事件类似，这是正确的，因为浏览器事件是这种模式的一个应用实例。浏览器是发布者；它宣布了事件（如`click`）发生的事实。订阅了这种类型事件的事件监听函数在事件发生时会收到通知。浏览器-发布者向所有订阅者发送一个事件对象。在自定义实现中，你可以发送任何你认为合适的数据。

观察者模式有两种子类型：推（push）和拉（pull）。推是指发布者负责通知每个订阅者，而拉是指订阅者监视发布者状态的变化。

让我们看一个推送模型的示例实现。让我们将观察者相关的代码保留在一个单独的对象中，然后将此对象用作混合对象，将其功能添加到任何决定成为发布者的其他对象中。这样，任何对象都可以成为发布者，任何函数都可以成为订阅者。观察者对象将具有以下属性和方法：

+   一个 `subscribers` 数组，它们只是回调函数

+   `addSubscriber()` 和 `removeSubscriber()` 方法，用于向 `subscribers` 集合添加和移除订阅者

+   一个 `publish()` 方法，它接受数据并调用所有订阅者，将数据传递给它们

+   一个 `make()` 方法，它接受任何对象，并通过向其添加之前提到的所有方法将其转换为发布者

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

现在，让我们创建一些发布者。发布者可以是任何对象，其唯一职责是在发生重要事件时调用 `publish()` 方法。这里有一个 `blogger` 对象，每次准备好新的博客帖子时都会调用 `publish()`：

```js
    var blogger = { 
      writeBlogPost: function() { 
        var content = 'Today is ' + new Date(); 
        this.publish(content); 
      } 
    }; 

```

另一个对象可以是 LA Times 报纸，当有新的报纸发布时调用 `publish()`。考虑以下代码行：

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

现在，让我们来看一下以下两个简单的对象，`jack` 和 `jill`：

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

`jack` 和 `jill` 对象可以通过提供他们想要在发布时调用的回调方法来订阅 `blogger` 对象，如下所示：

```js
    blogger.addSubscriber(jack.read); 
    blogger.addSubscriber(jill.gossip); 

```

现在，当 `blogger` 对象写了一个新的帖子时会发生什么？结果是 `jack` 和 `jill` 会收到通知：

```js
    > blogger.writeBlogPost(); 
       I just read that Today is Fri Jan 04 2013 19:02:12 GMT-0800 (PST) 
       You didn't hear it from me, but Today is Fri Jan 04 2013 19:02:12 GMT-0800    
         (PST) 

```

在任何时候，`jill` 可能决定取消她的订阅。然后，在写另一篇博客文章时，已取消订阅的对象将不再收到通知。考虑以下代码片段：

```js
    > blogger.removeSubscriber(jill.gossip); 
    > blogger.writeBlogPost();
    I just read that Today is Fri Jan 04 2013 19:03:29 GMT-0800 (PST) 

```

`jill` 对象可以决定订阅 LA Times，因为一个对象可以订阅多个发布者，如下所示：

```js
    > la_times.addSubscriber(jill.gossip); 

```

然后，当 LA Times 发布新问题时，`jill` 被通知并执行 `jill.gossip()`，如下所示：

```js
    > la_times.newIssue();
    You didn't hear it from me, but Martians have landed on Earth! 

```

# 总结

在本章中，您了解了常见的 JavaScript 编码模式，并学会了如何使您的程序更清洁、更快速，并更好地与其他程序和库一起工作。然后，您看到了《四人组设计模式》中一些设计模式的讨论和示例实现。您可以看到 JavaScript 是一种功能齐全的动态编程语言，而在动态弱类型语言中实现经典模式是相当容易的。总的来说，模式是一个大主题，您可以加入本书的作者在 [JSPatterns.com](http://www.jspatterns.com/) 进一步讨论 JavaScript 模式，或者查看 *JavaScript Patterns* 书籍。下一章将重点介绍测试和调试方法论。
