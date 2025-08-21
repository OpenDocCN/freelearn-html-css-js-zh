# 第一章：JavaScript 承诺 - 我为什么要关心？

曾经从未有过 JavaScript 如此受欢迎的时候。曾经，也许现在对一些人来说，它还是最受误解的编程语言，主要是因为它的名字，但现在它已经跻身最受欢迎的编程语言之列。此外，几乎每台个人电脑上可能都至少有一个 JavaScript 解释器在运行或至少已安装。JavaScript 日益受欢迎的原因完全在于它作为 Web 脚本语言的角色。当它最初被开发时，JavaScript 是设计在 Netscape Navigator 中运行的。在那里取得成功后，它几乎成为了所有网络浏览器中的标准配置，但 JavaScript 已经成长和成熟，现在暴露在大量与 Web 无关的开发中。在第一章中，我们将简要介绍以下内容：

+   异步编程在 JavaScript 中

+   开发者使用传统方法处理异步操作时所面临的问题

+   JavaScript 承诺简介

+   为什么在比较它与常见异步操作方式时，我们应该关心承诺。

# 异步编程在 JavaScript 中的运用

当涉及到 JavaScript 中的异步编程时，有二件事情需要讨论：Web 和编程语言。以浏览器为代表的 Web 环境与桌面环境不同，这也反映在我们为它们编程和编码的方式上。与桌面环境相反，浏览器为需要访问用户界面的所有事物提供了一个线程；在 HTML 术语中，就是 DOM。这种单线程模型对需要访问和修改 UI 元素的应用程序代码产生了负面影响，因为它将该代码的执行限制在同一个线程上。因此，我们将会有阻塞函数和线程，基本上会阻塞 UI，直到该线程执行完毕。这就是为什么，在 Web 开发中，充分利用浏览器提供的任何异步能力至关重要。

让我们回顾一些历史以获得更多背景信息。在过去，网站由完整的 HTML 页面组成，每次用户行动都需要从服务器加载整个网页。这给开发者带来了很多问题，尤其是当编写会影响页面的服务器端代码时。此外，它还导致了用户体验不佳。响应用户行动或 HTML 表单的变化是通过向表单所在的同一页面发送 HTTP `POST`请求来进行的；这导致服务器使用刚刚收到的信息刷新同一页面。整个过程和模型都是低效的，因为它导致页面内容的消失，然后重新出现，而且在慢速互联网环境下，有时内容会在传输过程中丢失。然后浏览器重新加载一个网页，重新发送所有内容，尽管只有部分信息发生了变化；这导致了带宽的浪费，并增加了服务器的负载。此外，它对用户体验产生了负面影响。后来，随着行业内不同方面的努力，异步网络技术开始出现以帮助解决这一局限性。在这一领域中，一个著名的角色是**异步 JavaScript 和 XML**（**AJAX**），这是在客户端用于创建以异步方式进行通信的网络应用程序的一组技术。AJAX 技术允许网络应用程序以异步方式发送和检索服务器上的数据，而不会干扰当前页面的 UI 和行为；基本上，无需重新加载整个页面。实现这一点的核心 API 是`XMLHttpRequest` API。

随着网络技术的发展和浏览器的进步，JavaScript 作为一种网络脚本语言变得越来越重要，使开发者能够访问 DOM，并动态地显示和与网页上呈现的内容进行交互。然而，JavaScript 也是单线程的，这意味着在任何给定时间，任何两条脚本线都不能同时运行；相反，JavaScript 语句是逐行执行的。同样，在浏览器中，JavaScript 与浏览器执行的其他工作负载（从绘制和更新样式到处理用户行动等）共享那个单线。一项活动将延迟另一项。

最初，JavaScript 旨在运行简短、快速的代码片段。主要的应用程序逻辑和计算是在服务器端完成的。自从网页内容的加载从重新加载整个页面改变到客户端以来，异步加载的开发人员开始更频繁地依赖 JavaScript 进行网络开发。现在，我们可以找到用 JavaScript 编写的完整应用程序逻辑，并且已经出现了许多库来帮助开发者这样做。

在网络开发中，我们有以下三个主要组件：

+   HTML 和 CSS

+   文档对象模型（DOM）

+   JavaScript

我将添加第四个在 AJAX 编程中扮演关键角色的组件：

+   XMLHttpRequest API

简要地说，HTML 和 CSS 用于网页的呈现和布局。DOM 用于动态显示和与内容交互。XHR 对象向网络服务器发送 HTTP/HTTPS 请求，并将服务器的响应数据加载回脚本中，中介一种异步通信。最后，JavaScript 允许开发者将所有这些技术结合起来，以创建美观、响应迅速和动态的网页应用程序。

为了克服多线程限制，开发者严重依赖事件和回调，因为这是浏览器将异步编程暴露给应用程序逻辑的方式。

在基于事件的异步 API 中，为给定对象注册一个事件处理程序，当事件被触发时调用该动作。浏览器通常会在不同的线程上执行这个动作，并在适当的时候在主线程上触发事件。

我们可以使用`object.addEventListener()`方法实现这种基于事件的技术。这个方法 simply 在被调用的目标对象上注册一个监听器。事件目标对象可能是一个 HTML 文档中的元素，文档本身，一个窗口，或其他支持事件的对象（如 XHR）。

以下代码展示了使用 HTML 和 JavaScript 创建的简单事件监听器的外观。

以下是 HTML 部分：

```js
<div id='testDiv' style="width:100px; height:100px; background-color:red">/</div>
```

以下是 JavaScript 部分：

```js
var testDiv = document.getElementById("testDiv");
testDiv.addEventListener("click", function(){
   // so the testDiv object has been clicked on, now we can do things
    alert("I'm here!");
});
```

在 HTML 部分，我们在 DOM 中定义了一个带有`testDiv` ID 的`div`元素。在 JavaScript 部分，我们在代码的第一行检索到这个`div`元素，并将其赋值给一个变量。然后，我们在那个对象上添加一个事件监听器，并将其传递给`click`事件，后跟一个匿名函数（一个没有名字的函数）作为监听函数。这个函数将在元素上发生点击事件后调用。

### 小贴士

如果你在包含`div`元素的 HTML 标记之前添加了这段 JavaScript 代码，它将会抛出一个错误。因为当代码针对它执行时，该元素还未被创建，所以代码将无法找到目标对象来调用`addEventListener`。

正如我们在之前的代码示例中看到的那样，`addEventListener`方法的第二个参数本身就是一个包含一些内联代码的函数。我们之所以能在 JavaScript 中这样做，是因为函数是一等对象。这个函数是一个回调。回调函数在 JavaScript 编程中非常重要且广泛应用，因为它们让我们能够异步地做事。

将回调函数作为参数传递给另一个函数，这只是传递了函数定义。因此，在参数中函数并不会立即执行；它会在容器函数体内某个指定点*回调*（因此得名）。这对于执行一些需要时间来完成的操作的脚本非常有用，例如向服务器发送 AJAX 请求或执行一些 IO 活动，而不会在这个过程中阻塞浏览器。

### 提示

如果你对 JavaScript 不熟悉，看到函数作为参数可能会有些不习惯，但不要担心；当你将它们视为对象时，它会变得容易。

一些浏览器 API，如 HTML5 Geolocation，是基于设计回调的。我将使用 Geolocation 的`getCurrentMethod`示例中使用回调函数。代码如下所示：

```js
navigator.geolocation.getCurrentPosition(function(position){  
  alert('I am here, Latitude: ' + position.coords.latitude + ' ' +  
                  '/ Longitude: ' + position.coords.longitude);  
});
```

在上一个示例中，我们简单地调用了`getCurrentPosition`方法，并传递了一个匿名函数，该函数反过来调用了 alert 方法，该方法将用我们请求的结果进行回调。这允许浏览器同步或异步执行此代码；因此，在获取位置时，代码不会阻塞浏览器。

在这个例子中，我们使用了内置浏览器 API，但我们也可以通过以异步方式暴露基本 API，并至少使用回调函数来使应用程序具备异步准备，涉及到 I/O 操作或计算量大的操作，这些操作可能需要花费大量时间。

例如，在回调场景中，检索某些数据的最简单代码如下所示：

```js
getMyData(function(myData){
   alert("Houston, we have : " + myData);
});
```

在之前的 JavaScript 代码中，我们定义了一个`getMyData`函数，该函数接受一个回调函数作为参数，进而执行一个显示我们应 retrieve 的数据的弹窗。实际上，这段代码使得与应用程序 UI 代码保持异步准备；因此，在代码检索数据时，UI 界面不会被阻塞。

让我们将其与非回调场景进行比较；代码如下所示：

```js
// WRONG: this will make the UI freeze when getting the data  
var myData = getMyData();
alert("Houston, we have : " + myData);
```

在上一个示例中，JavaScript 代码将逐行运行，尽管第一行还没有完成，下一行代码也将运行。这样的 API 设计会使代码 UI 阻塞，因为它将冻结 UI，直到数据被检索。此外，如果`getMyData()`函数的执行恰好需要一些时间，例如从互联网获取数据，整个用户体验将不会很好，因为 UI 必须等待这个函数执行完成。

此外，在前面 callback 函数的例子中，我们向包含函数传递了一个匿名函数作为参数。这是使用回调函数的最常见模式。使用回调函数的另一种方式是声明一个有名字的函数，然后将这个函数的名字作为参数传递。在下面的例子中，我们将使用一个有名字的函数。我们将创建一个通用函数，它接受一个字符串参数并在一个 alert 中显示它。我们将其称为 `popup`。然后，我们将创建另一个函数并称之为 `getContent`；这个函数接受两个参数：一个字符串对象和一个回调函数。最后，我们将调用 `getContent` 函数，并在第一个参数中传递一个字符串值，在第二个参数中传递回调函数 `popup`。运行脚本，结果将是一个包含第一个字符串参数值的 alert。以下是为这个例子准备的代码样本：

```js
//a generic function that displays an alert
    function popup(message) {
    alert(message);
    }
//A function that takes two parameters, the last one a callback function
    function getContent(content, callback) {
        callback(content); //call the callback function 
    }
getContent("JavaScript is awesome!", popup);
```

正如我们在 previous example 中所见，由于 callback 函数最终只是一般的函数，所以我们能够向其传递一个参数。我们可以将包含函数中的任何变量作为参数传递给 callback 函数，甚至可以是代码其他部分的全局变量。

总结来说，JavaScript 回调函数非常强大，极大地丰富了网页开发环境，从而使得开发者能够进行异步的 JavaScript 编程。

# 我为什么要关心承诺呢？

承诺与这一切有什么关系呢？好吧，让我们先定义一下承诺。

|   | *承诺代表异步操作的最终结果。* |   |
| --- | --- | --- |
|   | --*Promises/A+ 规格，[`promisesaplus.com/`](http://promisesaplus.com/)* |

所以，一个 Promise 对象代表了一个可能尚未可用的值，但会在未来的某个时刻被解决。

承诺有状态，在任何时刻，它可以处于以下之一的状态：

+   **挂起**：承诺的值尚未确定，其状态可能转变为已实现或已拒绝。

+   **实现**：承诺已成功实现，现在有一个必须不能改变的值。此外，它必须不能从已实现的状态转移到任何其他状态。

+   **拒绝**：承诺从一个失败的操作中返回，并且必须有一个失败的原因。这个原因不能改变，且承诺从这个状态不能转移到其他任何状态。

承诺只能从挂起状态转移到实现状态，或者从挂起状态转移到拒绝状态。然而，一旦一个承诺是实现或拒绝状态，它必须不再转移到任何其他状态，且其值不能改变，因为它是不可变的。

### 提示

承诺的不变特性是*非常重要的*。它帮助避免了监听器产生的意外副作用，这些副作用可能会导致行为上的意外变化，从而使得承诺在不影响调用函数的情况下，可以被传递给其他函数。

从 API 的角度来看，承诺被定义为一个具有`then`属性值为函数的对象。承诺对象有一个主要的`then`方法，它返回一个新的承诺对象。它的语法将如下所示：

```js
then(onFulfilled, onRejected);
```

以下两个参数基本上是在承诺完成时调用的回调函数：

+   `onFulfilled`：当一个承诺被满足时调用此参数

+   `onRejected`：当一个承诺失败时调用此参数

记住，这两个参数都是可选的。此外，非函数值的参数将被忽略，因此始终在执行它们之前检查传递的参数是否为函数可能是一个好习惯。

### 注意

值得注意的是，当你研究承诺时，你可能会遇到两个定义/规格：一个基于 Promises/A+，另一个是基于 CommonJS 的 Promises/A 的较旧的定义。虽然 Promises/A+是基于 CommonJS Promises/A 提案中呈现的概念和 API，但 A+实现与 Promises/A 在几个方面有所不同，正如我们在第二章《承诺 API 及其兼容性》中所见的那样。

`then`方法返回的新承诺在给定的`onFulfilled`或`onRejected`回调完成时被解决。实现反映了一个非常简单的概念：当一个承诺被满足时，它有一个值，当它被拒绝时，它有一个原因。

以下是如何使用承诺的一个简单示例：

```js
promise.then(function (value){
    var result = JSON.parse(data).value;
    }, function (reason) {
    alert(error.message);
});
```

从回调处理程序返回的值是返回承诺的实现值，这使得承诺操作可以连锁进行。因此，我们将会像以下这样：

```js
$.getJSON('example.json').then(JSON.parse).then(function(response) {
    alert("Hello There: ", response);
});
```

嗯，你说对了！前一个代码示例所做的就是在第一个`then()`调用返回的承诺上链第二个`then()`调用。因此，`getJSON`方法将返回一个包含 JSON 返回值的承诺。因此，我们可以在其中调用一个`then`方法，然后调用另一个返回的承诺上的`then`调用。这个承诺包括`JSON.parse`的值。最终，我们将取该值并在一个警告框中显示它。

## 我不能 just 使用回调吗？

回调很简单！我们传递一个函数，它在未来的某个时刻被调用，我们可以异步地做事情。此外，回调是轻量级的，因为我们需要添加额外的库。将函数作为高阶对象的使用已经内置在 JavaScript 编程语言中；因此，我们不需要额外的代码来使用它。

然而，如果不用心处理，JavaScript 中的异步编程可能会迅速变得复杂，尤其是回调。回调函数在嵌套在冗长的代码行中时，往往难以维护和调试。此外，在回调中使用匿名内联函数会使阅读调用堆栈变得非常繁琐。此外，当涉及到调试时，从嵌套的回调集中抛出的异常可能不会正确地传播到链中发起调用的函数，这使得确定错误的确切位置变得困难。而且，基于回调的结构化代码很难展开，就像滚雪球一样展开混乱的代码。我们最终会得到如下代码样本，但规模要大得多：

```js
function readJSON(filename, callback) {
    fs.readFile(filename, function (err, result) {
        if (err) return callback(err);
        try {
            result = JSON.parse(result, function (err, result) {
                fun.readAsync(result, function (err, result) {
                    alert("I'm inside this loop now");
                    });
                alert("I'm here now");
                });
            } catch (ex) {
        return callback(ex);
        }
    callback(null, result);
    });
}
```

### 注意

前面示例中的样本代码是被称为“末日金字塔”的深层嵌套代码的摘录。这样的代码在增长时，会使阅读、结构化、维护和调试变得令人望而却步。

另一方面，承诺提供了一个抽象概念，用于管理与异步 API 的交互，并在与回调和事件处理器的使用相比时，为 JavaScript 中的异步编程提供了一个更管理化的方法。我们可以将承诺视为异步编程的更多模式。

简单来说，承诺模式将使异步编程从广泛采用的延续传递风格转变为一种函数返回一个值（称为承诺），该值代表该特定操作的最终结果。

它允许你从：

```js
call1(function (value1) {
    call2(value1, function(value2) {
        call3(value2, function(value3) {
            call4(value3, function(value4) {
                // execute some code 
            });
        });
    });
});
```

致：

```js
Promise.asynCall(promisedStep1)
.then(promisedStep2)
.then(promisedStep3)
.then(promisedStep4)
.then(function (value4) {
    // execute some code
});
```

如果我们列出使承诺更易于处理的属性，它们将是以下内容：

+   使用更简洁的方法签名时，它更容易阅读

+   它允许我们将多个回调附加到同一个承诺

+   它允许值和错误被传递并冒泡到调用者函数

+   它允许承诺的链式操作

我们可以观察到，承诺通过返回值将功能组合带到同步能力，并通过抛出异常将错误冒泡到异步函数。这些是在同步世界中视为理所当然的能力。

以下示例（伪）代码展示了使用回调来组合相互通信的异步函数和用承诺来做同样事情的区别。

以下是使用回调的示例：

```js
    $("#testInpt").click(function () {
        firstCallBack(function (param) {
            getValues(param, function (result) {
                alert(result);
            });
        });
    });
```

以下是一个将之前的回调函数转换为可以相互链式的承诺返回函数的代码示例：

```js
    $("#testInpt").clickPromise()  // promise-returning function
    .then(firstCallBack)
    .then(getValues)
    .then(alert);
```

正如我们所看到的，承诺提供的平坦链使我们能够拥有比传统的回调方法更容易阅读和维护的代码。

# 总结

在 JavaScript 中，回调函数使我们能够拥有一个更响应灵敏的用户界面，它能够异步响应事件（即用户输入），而不会阻塞应用程序的其他部分。Promise 是一种模式，它允许在异步编程中采用一种标准化的方法，这使得开发者能够编写更易读、更易维护的异步代码。

在下一章中，我们将查看支持 Promise 的浏览器及其与 jQuery 的兼容性。你还将了解到支持类似 Promise 功能的库。
