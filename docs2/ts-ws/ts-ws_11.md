# 10. 事件循环与异步行为

概述

在本章中，你将探究网页如何在浏览器中实际工作，特别关注浏览器如何、何时以及为什么执行我们提供的 JavaScript 代码。你将深入了解事件循环的复杂性，并了解我们如何管理它。最后，你将了解 TypeScript 为你提供的工具。到本章结束时，你将能够更好地管理执行的异步特性。

# 简介

在上一章中，你学习了泛型和条件类型的基础知识。本章将向你介绍事件循环和异步行为。然而，在继续学习这些主题之前，让我们先看看一个假设场景，以便真正理解同步和异步执行是如何工作的。

想象一家小银行，它只有一位出纳员。他的名字叫汤姆，他全天都在为客户服务。由于银行规模小，客户不多，所以没有排队。因此，当客户进来时，他们会得到汤姆的全神贯注。客户提供了所有必要的文件，汤姆进行处理。如果这个过程需要某种外部输入，例如来自信用局或银行的后台部门，汤姆会提交请求，他和客户一起等待响应。他们可能会聊一会儿，当响应到来时，汤姆继续他的工作。如果需要打印文件，汤姆会将文件发送到他的办公桌上方的打印机，他们等待并聊天。当打印完成时，汤姆继续他的工作。一旦工作完成，银行又有一位满意的客户，汤姆继续他的工作日。如果有人来的时候汤姆正在为客户服务（这种情况很少发生），他们必须等待，直到汤姆完全完成前一个客户的服务，然后才开始他们的流程。即使汤姆正在等待外部响应，其他客户也必须等待他们的轮次，同时汤姆与当前客户闲聊。

汤姆有效地以同步和顺序的方式工作。这种工作方式有很多好处，即汤姆（和他的上司）总能知道他是否在为客户服务，他总是知道他的当前客户是谁，并且一旦客户离开，他可以完全忘记所有关于客户的数据，因为他知道他们已经得到了完全的服务。没有混淆来自不同客户的文件的问题。任何问题都容易诊断和修复。由于队列永远不会拥挤，这种设置让每个人都满意。

到目前为止，一切顺利。但是，当银行突然增加更多客户时会发生什么呢？随着越来越多的客户到来，我们排起了长队，每个人都等待着，而汤姆在与当前客户聊天，等待信用局的回复。可以理解，汤姆的老板对这种情况并不满意。当前系统根本无法扩展。因此，他想以某种方式改变系统，以便能够服务更多客户。他该如何做呢？你将在下一节中看到几个解决方案。

# 多线程方法

基本上，有两种不同的方法。一种是有多个汤姆。所以，每个柜员仍然会以前完全相同简单同步的方式工作——我们只是有很多个。当然，老板需要某种组织来了解哪个柜员可用，哪个柜员在工作，是否有为每个柜员单独的队列，或者一个大型队列，以及某种分配机制（即，为每个客户分配一个号码的系统）。老板可能还会得到一台大型的办公打印机，而不是每个柜员一台打印机，并制定某种规则以避免打印作业混淆。组织将会复杂，但每个柜员的任务将会简单明了。

到现在为止，你知道我们实际上并不是在讨论银行。这是服务器端处理通常采用的方法。粗略简化一下，服务器进程将会有多个子进程（称为线程）并行工作，而主进程将协调一切。每个线程将同步执行，有一个明确的开始、中间和结束。由于服务器通常是拥有大量资源的机器，在重负载下，这种方法是有意义的。它可以很好地适应低或高负载，处理每个请求的代码可以相对简单且易于调试。甚至有理由让线程等待某些外部资源（例如文件系统中的文件，或来自网络或数据库的数据），因为我们总是可以启动新的线程来处理更多的请求。然而，对于现实生活中的柜员来说并非如此。当客户增多时，我们无法简单地克隆一个新的柜员。线程（或汤姆）所进行的等待通常被称为*忙等待*。线程并没有做任何事情，但它无法进行任何工作，因为它正忙于做某事——它正忙于等待。就像汤姆实际上在等待信用局的回复时，正忙于与客户聊天一样。

我们有一个可以大规模并行和并发运行的系统，但每个部分仍然同步运行。这种方法的优点是我们可以同时服务成千上万的客户。一个明显的缺点是成本，无论是硬件还是复杂性。虽然我们设法保持了客户处理的简单性，但我们还需要一个巨大的基础设施来处理其他所有事情——添加柜员、移除柜员、排队等候的客户、管理办公室打印机的访问，以及类似的任务。

这将使用银行（或服务器）的所有可用资源，但这没关系，因为这就是整个目的——尽可能快地服务尽可能多的客户，没有其他。

然而，还有另一种方法——异步执行。

## 异步执行方法

另一种方法，即网络和扩展到 JavaScript 和 TypeScript 采用的方法，是只使用单个线程——所以汤姆仍然独自一人。但是，汤姆不会无所事事地与等待的客户聊天，他可以做其他事情。如果出现需要从后台办公室获取验证的情况，他只需在一张纸上写下他正在做什么以及他做到了哪一步，然后将那张纸交给客户，并让他们去队伍的后面。现在，汤姆可以开始服务下一个排队等候的客户。如果那个客户不需要外部资源，他们的请求将被完全处理，并可以自由离开。如果他们需要汤姆需要等待的其他东西，他们也会被送到队伍的后面。以此类推。这样，如果汤姆有任何客户，他正在处理他们的请求。他永远不会忙于等待，相反，他总是忙于工作。如果客户需要等待响应，他们会在汤姆之外单独等待。汤姆唯一空闲的时候是他没有任何客户的时候。

这种方法的优点相当明显——以前，汤姆花了很多时间聊天，现在他一直在工作（当然，这个好处是从汤姆老板的角度来看的——汤姆喜欢闲聊）。另一个好处是我们事先就知道我们的资源消耗。如果我们只有一个出纳员，我们就知道我们需要多少办公面积。然而，也有一些缺点。最重要的缺点是，我们的客户现在必须非常了解我们的流程。他们需要了解如何排队和重新排队，如何从他们离开的地方继续工作，等等。汤姆的工作也变得更加复杂。他需要知道如何暂停客户的处理，如何继续，如果未收到外部响应应该如何处理等等。这种工作模式通常被称为异步和并发。在执行他的工作时，汤姆会同时跳转到多个客户。不止一个客户会开始处理但未完成。而且客户无法估计一旦开始处理他们的任务将需要多长时间——这取决于汤姆同时处理多少其他客户。

从一开始，这个模型对网络来说就更有意义。首先，网络应用程序是在客户端的设备上处理的。我们不应该对此有任何技术假设——因为我们无法确定客户端可能会使用哪种设备。本质上，一个网页是客户端设备上的访客——并且它应该表现得体。例如，使用所有设备资源来展示一些花哨的动画根本不是合适的行为。另一个重要问题是安全性。如果我们把网页看作是包含一些代码的应用程序，那么每次我们在浏览器的地址栏中输入网址时，我们基本上就是在我们的机器上执行某人的代码。

浏览器需要确保页面上即使是有害的代码，也不能对我们的机器造成太大的影响。如果访问一个网站能让你的电脑爆炸，那么网络今天就不会像现在这样受欢迎了。

因此，由于浏览器无法事先知道它将用于哪些页面，所以决定每个网页只能访问一个线程。此外，出于安全考虑，每个网页将获得一个单独的线程，因此正在运行的网页不能干涉同时可能执行的其它页面的执行（通过如网络工作者和 Chrome 应用程序等特性，这些限制有所放宽，但原则上仍然适用）。

简单来说，网页无法生成足够的线程来系统性地蜂拥而至，或者网页无法从另一个网页获取数据。而且，由于网页需要同时做很多事情，使用同步和顺序的方法是不可能的。这就是为什么所有的 JavaScript 执行环境完全采用了异步、并发的方法。这样做到了这样的程度，以至于一些常见的同步技术，故意地，在 JavaScript 中不可用。

例如，许多其他语言都有一个“等待一段时间”的原语，或者一个执行此操作的库函数。例如，在 C#编程语言中，我们可以有如下代码：

```js
Console.WriteLine("We will wait 10 s");
Thread.Sleep(10000);
Console.WriteLine("... 10 seconds later");
Thread.Sleep(15000);
Console.WriteLine("... 15 more seconds later");
```

这段代码将向控制台写入一些文本，然后在 10 秒后写入更多文本。在等待的 25 秒内，执行此代码的线程将完全无响应，但代码简单且线性——易于理解、易于更改和易于调试。JavaScript 没有这样的同步原语，但它有一个异步变体，即`setTimeout`函数。最简单的等效代码如下：

```js
console.log("We will wait 10 s");
setTimeout(() => {
    console.log("... 10 seconds later");
    setTimeout(() => {
        console.log("... 15 more seconds later");
    }, 15000);
}, 10000);
```

很明显，这段代码比 C#的等效代码复杂得多，但我们获得的优势是这段代码是非阻塞的。在这段代码执行的 25 秒总时间内，我们的网页可以完成它需要做的所有事情。它可以响应用户事件，图片可以加载并显示，我们可以调整窗口大小，滚动文本——基本上，应用程序将恢复正常和预期的功能。

注意，虽然使用一些特殊的同步代码可以阻塞 JavaScript 执行，但这并不容易。当它实际上发生时，浏览器可以检测到这一行为并终止有问题的页面：

![图 10.1：无响应页面![图片](img/B14508_10_01.jpg)

图 10.1：无响应页面

# 执行 JavaScript

当一个 JavaScript 执行环境，如 node 或浏览器加载一个 JavaScript 文件时，它会解析它然后运行它。在 JavaScript 文件中定义的所有函数都会被注册，所有不在函数中的代码都会被执行。执行的顺序是根据代码在文件中的位置来确定的。所以，考虑一个包含以下代码的文件：

```js
console.log("First");
console.log("Second");
```

控制台将始终显示以下内容：

```js
First
Second
```

输出的顺序不能改变，除非修改代码本身。这是因为带有`First`的行将完全执行——总是如此——然后，并且只有然后，带有`Second`的行才会开始执行。这种方法是同步的，因为执行是由环境同步的。我们保证，除非上面的行完全执行完毕，否则第二行不会开始执行。但如果这一行调用了某个函数呢？让我们看看以下这段代码：

```js
function sayHello(name){
    console.log(`Hello ${name}`);
}
function first(){
    second();
}
function second(){
    third();
}
function third(){
    sayHello("Bob");
}
first();
```

当代码被解析时，环境将检测到我们有四个函数——`first`、`second`、`third`和`sayHello`。它还会执行不在函数内部的代码行（`first();`），这将启动`first`函数的执行。但是，当这个函数正在执行时，它会调用`second`函数。然后，运行时会挂起`first`函数的执行，记住它的位置，并开始执行`second`函数。这个函数反过来又调用`third`函数。同样的事情再次发生——运行时开始执行`third`函数，记住一旦该函数完成，它应该继续执行`second`函数，一旦`second`函数完成，它应该继续执行`first`函数。

运行时用来记住哪个函数是活动的，哪些在等待的结构被称为**栈**，具体来说，是*调用栈*。

注意

“栈”这个术语在这里是指盘子堆或煎饼堆。我们只能从顶部添加，也只能从顶部移除。

正在执行的函数被一个接一个地堆叠起来，最顶部的函数是正在积极执行的函数，如下面的表示所示：

![图 10.2：栈![图 10.2：栈](img/B14508_10_02.jpg)

图 10.2：栈

在示例中，`third`函数将调用`sayHello`函数，该函数反过来又调用`console`对象的`log`函数。一旦`log`函数执行完成，栈将开始回溯。这意味着一旦某个函数完成执行，它将从栈中移除，下面的函数将能够继续执行。因此，一旦`sayHello`函数执行完成，`third`函数将恢复并依次完成。这将触发`second`函数的继续执行，当该函数也完成时，`first`函数将继续，并最终完成。当`first`函数执行完成时，栈将变为空——运行时将停止执行代码。

值得注意的是，所有这些执行都是严格同步和确定性的。我们可以仅从查看代码中推断出函数调用的确切顺序和数量。我们还可以使用常见的调试工具，如断点和堆栈跟踪。

## 练习 10.01：堆叠函数

在这个练习中，我们将定义几个简单的相互调用的函数。每个函数在开始执行和即将完成执行时都会向控制台记录日志。我们将分析输出何时以及以何种顺序映射到控制台：

注意

这个练习的代码文件可以在[`packt.link/X7QZQ`](https://packt.link/X7QZQ)找到。

1.  创建一个新的文件，`stack.ts`。

1.  在`stack.ts`中定义三个函数，分别命名为`inner`、`middle`和`outer`。它们都不需要参数或返回类型：

    ```js
    function inner () {
    }
    function middle () {
    }
    function outer () {
    }
    ```

1.  在`inner`函数的主体中，添加一个缩进四个空格的`log`语句：

    ```js
    function inner () {
        console.log("    Inside inner function");
    }
    ```

1.  在`middle`函数的主体中，添加对`inner`函数的调用。在调用前后添加一个缩进两个空格的`log`语句：

    ```js
    function middle () {
        console.log("  Starting middle function");
        inner();
        console.log("  Finishing middle function");
    }
    ```

1.  在`outer`函数的主体中，添加对`middle`函数的调用。在调用前后添加一个`log`语句：

    ```js
    function outer () {
        console.log("Starting outer function");
        middle();
        console.log("Finishing outer function");
    }
    ```

1.  函数声明后，仅创建对`outer`函数的调用：

    ```js
    outer();
    ```

1.  保存文件，并使用以下命令进行编译：

    ```js
    tsc stack.ts
    ```

1.  验证编译是否成功完成，并且同一文件夹中生成了一个`stack.js`文件。在`node`环境中使用以下命令执行它：

    ```js
    node stack.js
    ```

    你会看到输出看起来像这样：

    ```js
    Starting outer function
      Starting middle function
        Inside inner function
      Finishing middle function
    Finishing outer function
    ```

输出显示了哪个函数首先开始执行（`outer`），因为这是首先显示的消息。还可以注意到，`middle`函数在`inner`函数完成后，但在`outer`函数完成之前已经执行完毕。

# 浏览器和 JavaScript

当用户请求网页时，浏览器需要做很多事情。我们不会深入到每个细节，但我们会看看它是如何处理我们的代码的。

首先，浏览器向服务器发送请求，并接收一个 HTML 文件作为响应。在该 HTML 文件中，嵌入了对页面所需资源的链接，例如图片、样式表和 JavaScript 代码。然后浏览器也下载这些资源，并将它们应用到下载的 HTML 文件上。图片被显示，元素被样式化，JavaScript 文件被解析并执行。

代码执行的顺序是按照 HTML 文件中的顺序，然后根据代码在文件中的位置。但是函数何时被调用呢？假设我们在文件中有以下代码：

```js
function sayHello() {
    console.log("Hello");
}
sayHello();
```

首先，注册`sayHello`函数，然后当它稍后被调用时，函数实际上执行并将`Hello`写入控制台。现在看看以下代码：

```js
function sayHello() {
    console.log("Hello");
}
function sayHi() {
    console.log("Hi");
}
sayHello();
sayHi();
sayHello();
```

当处理包含上述代码的文件时，它会注册两个函数，`sayHello`和`sayHi`。然后它会检测到有三个调用，也就是说，有三个任务需要处理。环境有一个称为**任务队列**的东西，它会将所有需要执行的函数依次放入其中。因此，我们的代码将被转换为三个任务。然后，环境会检查栈是否实际上为空，如果是，它将从队列中取出第一个任务并开始执行。栈的大小会根据第一个任务的代码执行而增长和缩小，最终，当第一个任务完成时，它将是空的。因此，在第一个任务执行后，情况将如下所示：

1.  执行栈将为空。

1.  任务队列将包含两个任务。

1.  第一个任务将完全完成。

一旦栈为空，下一个任务将被出队并执行，依此类推，直到任务队列和栈都为空，所有代码都执行完毕。再次强调，整个过程是同步进行的，按照指定的顺序。

## 浏览器中的事件

现在，看看另一个例子：

```js
function sayHello() {
    console.log("Hello");
}
document.addEventListener("click", sayHello);
```

如果你有一个由浏览器加载的 JavaScript 文件中的此代码，你可以看到`sayHello`函数已被注册但尚未执行。然而，如果你在页面上任何地方点击，你会在控制台看到`Hello`字符串出现，这意味着`sayHello`函数已被执行。如果你多次点击，你将在控制台看到多个`"Hello"`实例。而且这段代码甚至没有调用一次`sayHello`函数；代码中根本就没有`sayHello()`的调用。

发生的事情是，你*注册*了我们的函数作为事件监听器。考虑一下你根本就没有调用我们的函数，但浏览器环境会在某些事件发生时为我们调用它——在这个例子中，是整个`document`上的`click`事件。由于这些事件是由用户生成的，我们无法知道我们的代码何时以及是否会被执行。事件监听器是我们代码与所在页面通信的主要方式，它们是以异步方式调用的——你不知道函数何时以及是否会被调用，也不知道它会被调用多少次。

当事件发生时，浏览器会查找其内部注册的事件处理程序表。在我们的例子中，如果`document`（即整个网页）上的任何地方发生`click`事件，浏览器会看到你已经注册了`sayHello`函数来响应它。该函数不会直接执行——相反，浏览器会将函数的调用放入任务队列。之后，将采取之前解释的常规行为。如果队列和栈都为空，事件处理程序将立即开始执行。否则，我们的处理程序将等待它的轮次。

这是对异步行为的一个核心影响——我们根本无法保证事件处理程序会立即执行。可能它确实会立即执行，但没有办法知道在特定时刻队列和栈是否为空。如果它们是空的，我们将立即执行，但如果它们不是空的，我们就必须等待我们的轮次。

# 环境 API

我们与浏览器的交互大部分将遵循相同的模式——你将定义一个函数，并将该函数作为参数传递给某个浏览器 API。该函数何时以及是否会被调度执行将取决于该 API 的具体情况。在前一个例子中，你使用了事件处理程序 API，即`addEventListener`，它接受两个参数，一个是事件的名称，另一个是当该事件发生时将被调度的代码。

注意

你可以在[`developer.mozilla.org/en-US/docs/Web/Events`](https://developer.mozilla.org/en-US/docs/Web/Events)找到不同可能事件的列表。

在本章的其余部分，你还将使用另外两个 API，即环境方法来延迟一些代码的后续执行（称为`setTimeout`）以及调用外部资源的能力（通常称为 AJAX）。我们将使用两种不同的 AJAX 实现，原始的`XMLHttpRequest`实现和更现代、更灵活的`fetch`实现。

## `setTimeout`

如前所述，环境不提供暂停 JavaScript 执行一定时间的可能性。然而，在经过一定时间后执行某些代码的需求相当常见。因此，我们不是暂停执行，而是执行一些具有相同结果的不同操作。我们可以安排一段代码在经过一段时间后执行。为此，我们使用`setTimeout`函数。此函数接受两个参数：一个需要执行的函数，以及以毫秒为单位延迟执行该函数的时间：

```js
setTimeout(function() {
    console.log("After one second");
}, 1000);
```

这意味着传递给参数的匿名函数将在 1,000 毫秒后执行，即一秒后。

## 练习 10.02：探索 setTimeout

在这个练习中，你将使用`setTimeout`环境 API 调用来调查异步执行的行为以及它做了什么：

注意

本练习的代码文件可以在[`packt.link/W0mlS`](https://packt.link/W0mlS)找到。

1.  创建一个新的文件，`delays-1.ts`。

1.  在`delays-1.ts`中，在文件开头记录一些文本：

    ```js
    console.log("Printed immediately");
    ```

1.  添加两个对`setTimeout`函数的调用：

    ```js
    setTimeout(function() {
        console.log("Printed after one second");
    }, 1000);
    setTimeout(function() {
        console.log("Printed after two second");
    }, 2000);
    ```

    在这里，我们不是创建一个函数并将其通过其名称传递给`setTimeout`函数，而是使用我们就地创建的匿名函数。我们也可以使用箭头函数代替使用`function`关键字定义的函数。

1.  保存文件，并使用以下命令编译它：

    ```js
    tsc delays-1.ts
    ```

1.  验证编译是否成功完成，并在同一文件夹中生成一个`delays-1.js`文件。使用以下命令在`node`环境中执行它：

    ```js
    node delays-1.js
    ```

    你会看到输出如下：

    ```js
    Printed immediately
    Printed after one second
    Printed after two second
    ```

    输出的第二行和第三行不应立即出现，而应在 1 秒和 2 秒后分别出现。

1.  在`delays-1.ts`文件中，交换对`setTimeout`函数的两个调用：

    ```js
    console.log("Printed immediately");
    setTimeout(function() {
        console.log("Printed after two second");
    }, 2000);
    setTimeout(function() {
        console.log("Printed after one second");
    }, 1000);
    ```

1.  再次编译并运行代码，并验证输出行为是否相同。即使先前的`setTimeout`先执行，其`function`参数也不计划在 2 秒后运行。

1.  在`delays-1.ts`文件中，将初始的`console.log`移动到文件底部：

    ```js
    setTimeout(function() {
        console.log("Printed after two second");
    }, 2000);
    setTimeout(function() {
        console.log("Printed after one second");
    }, 1000);
    console.log("Printed immediately");
    ```

1.  再次编译并运行代码，并验证输出行为是否相同。这说明了代码行为异步时最常见的一个问题之一。即使该行位于我们文件的底部，它也是首先执行的。要心理追踪不遵循我们习惯的从上到下范式的代码要困难得多。

1.  创建一个新的文件，`delays-2.ts`。

1.  在`delays-2.ts`中，添加对`setTimeout`函数的单次调用，并将其延迟时间设置为`0`。这意味着我们的代码需要等待 0 毫秒才能执行：

    ```js
    setTimeout(function() {
        console.log("#1 Printed immediately?");
    }, 0);
    ```

1.  在调用`setTimeout`之后添加一个`console.log`语句：

    ```js
    console.log("#2 Printed immediately.");
    ```

1.  保存文件，并使用以下命令编译它：

    ```js
    tsc delays-2.ts
    ```

1.  确认编译成功结束，并且在同一文件夹中生成了一个`delays-2.js`文件。在`node`环境中使用以下命令执行它：

    ```js
    node delays-2.js
    ```

    你会看到输出看起来像这样：

    ```js
    #2 Printed immediately.
    #1 Printed immediately?;
    ```

    嗯，这看起来有些意外。两行几乎立即就出现了，但位于`setTimeout`块中的那一行，在代码中排在第一位，却在脚本底部的行之后。我们明确地告诉`setTimeout`不要等待，也就是说，在代码执行前等待 0 毫秒。

为了理解发生了什么，我们需要回到调用队列。当文件加载时，环境检测到我们有两个任务需要完成，即对`setTimeout`的调用和脚本底部的对`console.log`的调用（#2）。因此，这两个任务被放入任务队列。由于当时栈是空的，`setTimeout`调用开始执行，而#2 被留在任务队列中。环境看到它有一个零延迟，所以立即取出了函数（#1），并将其*放在*任务队列的末尾，在#2 之后。因此，在`setTimeout`调用完成后，我们剩下两个`console.log`任务在队列中，#2 是第一个，#1 是第二个。

它们是顺序执行的，在我们的控制台中，我们首先得到#2，然后是#1。

## AJAX（异步 JavaScript 和 XML）

在网络发展的早期，一旦页面加载完成，就无法从服务器获取数据。这对开发动态网页来说是一个巨大的不便，而这个问题是通过引入一个名为`XMLHttpRequest`的对象来解决的。这个对象使开发者能够在初始页面加载后从服务器获取数据——由于从服务器加载数据意味着使用外部资源，因此它必须以异步方式进行（即使它的名字中包含 XML，目前它将主要用于 JSON 数据）。要使用这个对象，你需要实例化它并使用它的一些属性。

为了说明其用法，我们将尝试从 Open Library 项目获取关于威廉·莎士比亚的数据。我们将使用以下 URL 来检索这些信息：[`openlibrary.org/authors/OL9388A.json`](https://openlibrary.org/authors/OL9388A.json)，我们将使用的方法是`GET`，因为我们只会获取数据。

收到的数据是 Open Library 定义的特定格式，因此你将首先为你要实际使用的数据创建一个接口。你将只显示莎士比亚的图像（作为照片 ID 的数组接收），以及姓名，因此你可以将接口定义如下：

```js
interface OpenLibraryAuthor {
  personal_name: string;
  photos: number[];
}
```

接下来，创建`XMLHttpRequest`对象，并将其分配给名为`xhr`的变量：

```js
const xhr = new XMLHttpRequest();
```

现在你需要`打开`一个连接到我们的 URL：

```js
const url = "https://openlibrary.org/authors/OL9388A.json";
xhr.open("GET", url);
```

这个调用实际上没有发送任何内容，但它为访问外部资源准备了系统。最后，您需要实际发送请求，使用`send`方法：

```js
xhr.send();
```

由于请求是异步的，这个调用将立即执行并完成。为了在请求完成后实际处理数据，您需要向这个对象添加一些内容——一个回调。这是一个函数，它不会由我们执行，而是由`xhr`对象在某个事件发生时执行。这个对象有几个事件，如`onreadystatechange`、`onload`、`onerror`、`ontimeout`，您可以为不同的事件设置不同的函数来响应，但在这个情况下，您将只使用`onload`事件。创建一个函数，它将从响应中获取数据并在我们的脚本运行的网页上显示它：

```js
const showData = () => {
  if (xhr.status != 200) {
    console.log(`An error occured ${xhr.status}: ${xhr.statusText}`);
  } else {
    const response: OpenLibraryAuthor = JSON.parse(xhr.response);
    const body = document.getElementsByTagName("body")[0];

    const image = document.createElement("img");
    image.src = `http://covers.openlibrary.org/a/id/${response.photos[0]}-M.jpg`;
    body.appendChild(image);
    const name = document.createElement("h1");
    name.innerHTML = response.personal_name;
    body.appendChild(name);
  }
};
```

在这个方法中，您将使用之前定义的`xhr`变量的某些属性，例如`status`，它给我们提供了请求的 HTTP 状态码，或者`response`，它给我们实际的响应。如果我们自己调用`showData`方法，我们很可能会得到空字段或错误，因为响应还没有完成。所以，我们需要将这个函数给`xhr`对象，它将使用它来调用`showData`回：

```js
xhr.onload = showData;
```

将此代码保存为`shakespeare.ts`，编译它，并使用以下方式将其添加到 HTML 页面中：

```js
    <script src="img/shakespeare.js"></script>
```

您将得到以下类似的结果：

![图 10.3：威廉·莎士比亚的检索图像](img/B14508_10_03.jpg)

图 10.3：威廉·莎士比亚的检索图像

## 活动十.01：使用 XHR 和回调的影片浏览器

作为 TypeScript 开发者，您被分配了一个创建一个简单的页面来查看电影数据的任务。网页将很简单，有一个文本输入字段和一个按钮。当您在搜索输入字段中输入电影名称并按下按钮时，电影的一般信息将显示在网页上，以及一些与电影相关的图像。

您可以使用*电影数据库*（[`www.themoviedb.org/`](https://www.themoviedb.org/））作为一般数据源，特别是它的 API。您需要使用`XmlHttpRequest`发出 AJAX 请求，并使用网站提供的数据来格式化您自己的对象。当使用 API 时，数据很少，如果不是永远，会是我们需要的形式。这意味着您将需要使用多个 API 请求来获取我们的数据，并逐步构建我们的对象。TypeScript 解决此问题的常见方法是用两组接口——一组与 API 格式完全匹配，另一组与您在应用程序中使用的数据匹配。在这个活动中，您需要使用`Api`后缀来表示与 API 格式匹配的接口。

另一个需要注意的重要事项是，这个特定的 API 不允许完全开放访问。您需要注册 API 密钥，然后在每个 API 请求中发送它。在本活动的设置代码中，将提供三个函数（`getSearchUrl`、`getMovieUrl`、`getPeopleUrl`），这些函数将在将`apiKey`变量设置为从 The Movie Database 接收到的值后生成所需的 API 请求的正确 URL。还将提供基本 HTML、样式以及实际显示数据的代码——唯一缺少的是数据本身。

这些资源在此列出：

+   `display.ts` – 一个 TypeScript 文件，包含`showResult`和`clearResults`方法，您将调用这些方法来显示电影和清除屏幕。

+   `interfaces.ts` – 一个 TypeScript 文件，包含您将使用的接口。所有带有`Api`后缀的接口都是您将从 The Movie Database API 接收到的对象，其余的（`Movie`和`Character`）将用于显示数据。

+   `script.ts` – 一个包含启动应用程序的样板代码的 TypeScript 文件。`search`函数在这里，这个函数将是本活动的重点。

+   `index.html` – 一个包含我们网页基本标记的 HTML 文件。

+   `styles.css` – 一个用于样式化结果的样式表文件。

以下步骤将有助于您解决问题：

备注

本活动的代码文件可以在[https://packt.link/Qo4dB](https://packt.link/Qo4dB)找到。

1.  在`script.ts`文件中定位`search`函数，并验证它接受一个字符串参数，并且其主体为空。

1.  构造一个新的`XMLHttpRequest`对象。

1.  使用`getSearchUrl`方法构造一个新的搜索结果 URL 字符串。

1.  调用`xhr`对象的`open`和`send`方法。

1.  为`xhr`对象的`onload`事件添加一个事件处理器。获取响应并将其解析为 JSON 对象。将结果存储在`SearchResultApi`接口的变量中。此数据将在`results`字段中包含我们的搜索结果。如果没有结果，这意味着我们的搜索失败。

1.  如果搜索没有返回结果，调用`clearResults`方法。

1.  如果搜索返回了一些结果，只需取第一个并将其存储在变量中，忽略其他结果。

1.  在`onload`处理器内部，在成功的搜索分支中，创建一个新的`XMLHttpRequest`对象。

1.  使用`getMovieUrl`方法构造一个新的搜索结果 URL 字符串。

1.  调用构造的`xhr`对象的`open`和`send`方法。

1.  为`xhr`对象的`onload`事件添加一个事件处理器。获取响应，并将其解析为 JSON 对象。将结果存储在`MovieResultApi`接口的变量中。此响应将包含我们电影的通用数据，具体来说，是除了参与电影的人员之外的所有内容。您将需要另一个 API 调用以获取有关人员的数据。

1.  在`onload`处理程序内部，在成功的搜索分支中，创建一个新的`XMLHttpRequest`对象。

1.  使用`getPeopleUrl`方法构造一个新的搜索结果 URL 字符串。

1.  调用构造的`xhr`对象的`open`和`send`方法。

1.  为`xhr`对象的`onload`事件添加事件处理程序。获取响应，并将其解析为 JSON 对象。将结果存储在`PeopleResultApi`接口的变量中。此响应将包含有关参与电影的人物数据。

1.  现在，你实际上已经拥有了所有需要的数据，因此你可以在`people onload`处理程序内部创建自己的对象，该处理程序位于电影`onload`处理程序内部，而电影`onload`处理程序又位于搜索`onload`处理程序内部。

1.  人物数据具有`cast`和`crew`属性。你将只取前六个演员，所以首先根据演员的`order`属性对`cast`属性进行排序。然后从第一个六个演员中切出一个新的数组。

1.  将演员数据（即`CastResultApi`对象）转换为我们的`Character`对象。你需要将`CastResultApi`的`character`字段映射到`Character`的`name`字段，将`name`字段映射到演员名字，将`profile_path`字段映射到`image`属性。

1.  从人物数据的`crew`属性中，你只需要导演和编剧。由于可能有多个导演和编剧，你需要分别连接所有导演和编剧的名字。对于导演，从`crew`属性中筛选出具有`Directing`部门`Director`职位的导演。对于这些对象，取其`name`属性，并用`&`连接起来。

1.  从人物数据的`crew`属性中，你只需要导演和编剧。由于可能有多个导演和编剧，你将分别连接所有导演和编剧的名字。对于导演，从`crew`属性中筛选出具有`Directing`部门`Director`职位的导演。对于这些对象，取其`name`属性，并用`&`连接起来。

1.  创建一个新的`Movie`对象（使用对象字面量语法）。使用你迄今为止准备的电影和人物响应中的数据填写`Movie`对象的全部属性。

1.  使用你构造的电影调用`showResults`函数。

1.  在你的父目录（在这种情况下为`Activity01`）中，使用`npm i`安装依赖项。

1.  使用`tsc ./script.ts ./interfaces.ts ./display.ts`编译程序。

1.  验证编译是否成功结束。

1.  使用你选择的浏览器打开`index.html`。

    你应该在浏览器中看到以下内容：

    ![图 10.4：最终的网页

    ![图 10.4：最终的网页

图 10.4：最终的网页

注意

此活动的解决方案可以通过此链接找到。

我们将在*活动 10.02，使用 fetch 和 Promises 的电影浏览器*和*活动 10.03，使用 fetch 和 async/await 的电影浏览器*中进一步改进此应用程序。然而，在我们这样做之前，你需要了解 TypeScript 中的 Promises 和`async`/`await`。

# Promises

使用回调函数进行异步开发可以完成任务——这真是太好了。然而，在许多应用中，我们的代码需要一直使用外部或异步资源。所以，很快我们就会遇到这样的情况：在我们的回调函数内部，还有一个异步调用，这需要回调函数嵌套在回调函数中，而这个回调函数又需要自己的回调……

在某些情况下，深入回调函数的层级多达十多层并不罕见。

## 练习 10.03：数到五

在这个练习中，我们将创建一个函数，当执行时，将输出从一至五的英文单词。每个单词将在上一个单词显示后 1 秒钟出现在屏幕上：

注意

本练习的代码文件可以在 [`packt.link/zD7TT`](https://packt.link/zD7TT) 找到。

1.  创建一个新的文件，`counting-1.ts`。

1.  在 `counting-1.ts` 中添加一个包含从一至五（包括五）的英文数字名称的数组：

    ```js
    const numbers = ["One", "Two", "Three", "Four", "Five"];
    ```

1.  对 `setTimeout` 函数进行单个调用，并在一秒后打印出第一个数字：

    ```js
    setTimeout(function() {
        console.log(numbers[0]);
    }, 1000);
    ```

1.  保存文件，并使用以下命令进行编译：

    ```js
    tsc counting-1.ts
    ```

1.  验证编译是否成功结束，并在同一文件夹中生成了一个 `counting-1.js` 文件。在 `node` 环境中使用此命令执行它：

    ```js
    node counting-1.js
    ```

    你会看到输出看起来像这样：

    ```js
    One
    ```

    这行应该在应用程序运行后 1 秒出现。

1.  在 `counting-1.ts` 文件中，在 `setTimeout` 函数内部，在 `console.log` 下方添加另一个嵌套的 `setTimeout` 函数调用：

    ```js
    setTimeout(function() {
        console.log(numbers[0]);
        setTimeout(function() {
            console.log(numbers[1]);
        }, 1000);
    }, 1000);
    ```

1.  再次编译并运行代码，并验证输出是否多了一行，在第一个输出后 1 秒显示：

    ```js
    One
    Two
    ```

1.  在 `counting-1.ts` 文件中，在嵌套的 `setTimeout` 函数内部，在 `console.log` 之上添加另一个嵌套的 `setTimeout` 函数调用：

    ```js
    setTimeout(function() {
        console.log(numbers[0]);
        setTimeout(function() {
            setTimeout(function() {
                console.log(numbers[2]);
            }, 1000);
            console.log(numbers[1]);
        }, 1000);
    }, 1000);
    ```

1.  在最内层的 `setTimeout` 函数中，在 `console.log` 下方添加另一个嵌套的 `setTimeout` 函数调用，并且对第五个数也要重复这个过程。代码应该看起来像这样：

    ```js
    setTimeout(function() {
        console.log(numbers[0]);
        setTimeout(function() {
            setTimeout(function() {
                console.log(numbers[2]);
                setTimeout(function() {
                    console.log(numbers[3]);
                    setTimeout(function() {
                        console.log(numbers[4]);
                    }, 1000);
                }, 1000);
            }, 1000);
            console.log(numbers[1]);
        }, 1000);
    }, 1000);
    ```

1.  再次编译并运行代码，并验证输出是否按正确顺序显示，如下所示：

    ```js
    One
    Two
    Three
    Four
    Five
    ```

在这个简单的例子中，我们实现了一个简单的功能——数到五。但正如你所能看到的，代码正变得越来越混乱。想象一下，如果我们需要数到 20 而不是 5 会怎样。那将是一个完全无法维护的混乱。虽然有一些方法可以使这段特定的代码看起来更好、更易于维护，但总的来说，情况并非如此。回调函数的使用本质上与混乱且难以阅读的代码相关。而混乱且难以阅读的代码是错误隐藏的最佳场所，因此回调函数确实有导致难以诊断的错误的声誉。

回调函数的另一个问题是，不同对象之间不能有一个统一的 API。例如，我们需要明确知道，为了使用 `xhr` 对象接收数据，我们需要调用 `send` 方法并为 `onload` 事件添加回调。并且我们需要知道，为了检查请求是否成功，我们必须检查 `xhr` 对象的 `status` 属性。

## 承诺是什么？

幸运的是，我们可以向您保证有一种更好的方法。这种方法最初是由第三方库实现的，但它已被证明非常有用且被广泛采用，以至于它被包含在 JavaScript 语言本身中。这个解决方案背后的逻辑相当简单。每个异步调用基本上是一个承诺，在未来的某个时刻，某个任务将被完成，某个结果将被获得。就像现实生活中的承诺一样，我们可以为一个承诺有三种不同的状态：

+   一个承诺可能尚未解决。这意味着我们需要等待更多的时间才能得到结果。在 TypeScript 中，我们称这些承诺为“挂起。”

+   一个承诺可能被负面地解决——许诺的人违背了承诺。在 TypeScript 中，我们称这些承诺为“拒绝”，通常我们会因此得到某种错误。

+   一个承诺可能被正面解决——许诺的人履行了承诺。在 TypeScript 中，我们称这些承诺为“解决”，并且我们可以从它们中获得一个值——实际的结果。

由于承诺本身就是对象，这意味着承诺可以被分配给变量，从函数中返回，作为函数的参数传递，以及我们可以用常规对象做的许多其他事情。

承诺的另一个伟大特性是，围绕现有的基于回调的函数编写一个承诺包装器相对容易。让我们尝试将莎士比亚的例子承诺化。我们将首先查看 `showData` 函数。这个函数需要做很多事情，而且这些事情有时并不相互关联。它需要处理 `xhr` 变量以提取数据，并且它需要知道如何处理这些数据。因此，如果我们所使用的 API 发生变化，我们需要更改我们的函数。如果我们的网页结构发生变化，也就是说，如果我们需要显示一个 `div` 而不是 `h1` 元素，我们也需要更改我们的函数。如果我们需要将作者数据用于其他目的，我们也需要更改我们的函数。基本上，如果响应需要发生任何事情，它必须在那时发生。我们无法将此决策推迟到另一段代码中。这在我们代码中创建了不必要的耦合，使得维护变得更加困难。

让我们改变一下。我们可以创建一个新的函数，该函数将返回一个承诺，该承诺将提供关于作者的数据。它将不知道这些数据将被用于什么：

```js
const getShakespeareData = () => {
  const result = new Promise<OpenLibraryAuthor>((resolve, reject) => {
      const xhr = new XMLHttpRequest();
      const url = "https://openlibrary.org/authors/OL9388A.json";
      xhr.open("GET", url);
      xhr.send();
      xhr.onload = () => {
        if (xhr.status != 200) {
            reject({
                error: xhr.status,
                message: xhr.statusText
            })
        } else {
            const response: OpenLibraryAuthor = JSON.parse(xhr.response);
            resolve(response);
        }
      }
  });
  return result;
};
```

这个函数返回一个由 `Promise` 构造函数创建的 `Promise` 对象。这个构造函数接受一个单一参数，即一个函数。该函数也接受两个参数（也是函数），按照惯例称为 `resolve` 和 `reject`。你可以看到构造函数内部的函数只是创建了一个 `xhr` 对象，调用其 `open` 和 `send` 方法，设置其 `onload` 属性，然后返回。所以，基本上，没有做任何事情，除了发起请求。

这样创建的 Promise 将处于挂起状态。并且 Promise 将保持这种状态，直到在主体内部调用 `resolve` 或 `reject` 函数之一。如果调用 `reject` 函数，它将过渡到拒绝状态，我们可以使用 `Promise` 对象的 `catch` 方法来处理错误；如果调用 `resolve` 函数，它将过渡到解决状态，我们可以使用 `Promise` 对象的 `then` 方法。

我们应该注意的一件事是，这个方法不执行任何与 UI 相关的操作。它不会在控制台打印任何错误或更改任何 DOM 元素。它只是承诺给我们一个 `OpenLibraryAuthor` 对象。现在，我们可以随意使用这个对象：

```js
getShakespeareData()
  .then(author => {
    const body = document.getElementsByTagName("body")[0];

    const image = document.createElement("img");
    image.src = `http://covers.openlibrary.org/a/id/${author.photos[0]}-M.jpg`;
    body.appendChild(image);
    const name = document.createElement("h1");
    name.innerHTML = author.personal_name;
    body.appendChild(name);
  })
  .catch(error => {
    console.log(`An error occured ${error.error}: ${error.message}`);
  })
```

在这段代码中，我们调用 `getShakespeareData` 数据函数，然后在其结果上调用两个方法，`then` 和 `catch`。`then` 方法仅在 Promise 处于已解决状态时执行，并接受一个将获取结果的函数。`catch` 方法仅在 Promise 处于拒绝状态时执行，并将错误作为其函数的参数。

对于 `then` 和 `catch` 方法的一个重要注意事项——它们也返回 Promise。这意味着 `Promise` 对象是可链的，所以与其深入，就像我们处理回调那样，我们可以说得更长。为了说明这一点，让我们再次数到五。

注意

在 *第十二章，TypeScript 中的 Promise 指南* 中将更全面地讨论 Promise。

## 练习 10.04：使用 Promise 计数到五

在这个练习中，我们将创建一个函数，当执行时，将输出从一至五的英语单词。每个单词将在上一个单词显示后 1 秒钟出现在屏幕上：

注意

本练习的代码文件可以在[`packt.link/nlge8`](https://packt.link/nlge8)找到。

1.  创建一个新文件，`counting-2.ts`。

1.  在 `counting-2.ts` 中，添加一个包含从一到五（包括五）的英语数字名称的数组：

    ```js
    const numbers = ["One", "Two", "Three", "Four", "Five"];
    ```

1.  添加 `setTimeout` 函数的包装器。这个包装器只有在指定的超时时间到期时才会执行：

    ```js
    const delay = (ms: number) => {
        const result = new Promise<void>((resolve, reject) => {
            setTimeout(() => {
                resolve();
            }, ms)
        });
        return result;
    }
    ```

    由于我们的 Promise 不会返回任何有意义的值，而是在给定数量的毫秒后简单地解决，因此我们提供了 `void` 作为其类型。

1.  使用参数 `1000` 调用 `delay` 方法，并在其解决后打印出第一个数字：

    ```js
    delay(1000)
    .then(() => console.log(numbers[0]))
    ```

1.  保存文件，并使用以下命令编译它：

    ```js
    tsc counting-2.ts
    ```

1.  验证编译是否成功完成，并在同一文件夹中生成一个`counting-2.js`文件。使用以下命令在`node`环境中执行它：

    ```js
    node counting-2.js
    ```

    您将看到输出看起来像这样：

    ```js
    One
    ```

    该行应在应用程序运行后 1 秒出现。

1.  在`counting-2.ts`文件中，在`then`行之后，添加另一个`then`行。在其内部，再次调用`delay`方法，超时时间为 1 秒：

    ```js
    delay(1000)
    .then(() => console.log(numbers[0]))
    .then(() => delay(1000)) 
    ```

    我们可以这样做，因为`then`方法的结果是`Promise`，它有自己的`then`方法。

1.  在最后一个`then`行之后，添加另一个`then`行，在其内部打印出第二个数字：

    ```js
    delay(1000)
    .then(() => console.log(numbers[0]))
    .then(() => delay(1000))
    .then(() => console.log(numbers[1])) 
    ```

1.  再次编译并运行代码，并验证输出是否多了一行，在第一个输出后 1 秒显示。

1.  在`counting-2.ts`文件中，为第三个、第四个和第五个数字添加两个额外的`then`行。代码应如下所示：

    ```js
    delay(1000)
    .then(() => console.log(numbers[0]))
    .then(() => delay(1000))
    .then(() => console.log(numbers[1]))
    .then(() => delay(1000))
    .then(() => console.log(numbers[2]))
    .then(() => delay(1000))
    .then(() => console.log(numbers[3]))
    .then(() => delay(1000))
    .then(() => console.log(numbers[4])) 
    ```

1.  再次编译并运行代码，并验证输出是否按正确顺序出现。

    让我们比较一下这段代码与之前练习中的代码。这不是最干净的代码，但其功能相对明显。我们可以看到如何扩展此代码以计数到 20。这里的主要好处是，尽管这段代码是异步的，但它仍然是顺序执行的。我们可以对其进行分析，并且位于顶部的行将在位于底部的行之前执行。此外，由于我们现在有了对象，我们甚至可以将此代码重构为更简单、更易于扩展的格式——我们可以使用`for`循环。

1.  在`counting-2.ts`文件中，删除从`delay(1000)`开始的直到文件末尾的行。添加一行来定义一个已解决的 promise：

    ```js
    let promise = Promise.resolve();
    ```

1.  添加一个`for`循环，对于`numbers`数组中的每个数字，将 1 秒的延迟添加到 promise 链中，并打印该数字：

    ```js
    for (const number of numbers) {
        promise = promise
            .then(() => delay(1000))
            .then(() => console.log(number))
    };}
    ```

1.  再次编译并运行代码，并验证输出是否按正确顺序出现，如下所示：

    ```js
    One
    Two
    Three
    Four
    Five
    ```

## 活动十点零二：使用 fetch 和 Promises 实现电影浏览器

此活动将重复之前的活动。主要区别在于，我们不会使用`XMLHttpRequest`及其`onload`方法，而是使用`fetch`网络 API。与`XMLHttpRequest`类相比，`fetch`网络 API 返回一个`Promise`对象，因此我们不需要嵌套回调以进行多个 API 调用，我们可以有一个更易于理解的 promise 解析链。

此活动与之前的活动具有相同的文件和资源。

以下步骤将有助于您解决问题：

1.  在`script.ts`文件中，找到`search`函数并验证它是否接受一个字符串参数，并且其主体为空。

1.  在`search`函数上方，创建一个名为`getJsonData`的辅助函数。此函数将使用`fetch` API 从端点获取数据并将其格式化为 JSON。它应接受一个名为`url`的单个字符串参数，并且它应返回一个`Promise`。

1.  在 `getJsonData` 函数的主体中，添加调用带有 `url` 参数的 `fetch` 函数的代码，然后对返回的响应调用 `json` 方法。

1.  在 `search` 方法中，使用 `getSearchUrl` 方法构建一个新的搜索结果 URL 字符串。

1.  使用 `searchUrl` 作为参数调用 `getJsonData` 函数。

1.  在 `getJsonData` 返回的 promise 上添加一个 `then` 处理程序。处理程序接受一个类型为 `SearchResultApi` 的单个参数。

1.  在处理程序的主体中，检查是否有任何结果，如果没有，则抛出错误。如果有结果，则返回第一个项目。注意，处理程序返回一个具有 `id` 和 `title` 属性的对象，但 `then` 方法实际上返回的是该数据的 promise。这意味着在处理程序之后，我们可以链式调用其他 `then` 调用。

1.  在先前的处理程序中添加另一个 `then` 调用。此处理程序将接受一个包含电影 `id` 和 `title` 的 `movieResult` 参数。使用 `id` 属性调用 `getMovieUrl` 和 `getPeopleUrl` 方法，分别获取电影详情和演员阵容及制作团队的正确 URL。

1.  在获取到 URL 后，使用这两个 URL 调用 `getJsonData` 函数，并将结果值分配给变量。注意，`getJsonData(movieUrl)` 调用将返回一个 `MovieResultApi` 的 promise，而 `getJsonData(peopleUrl)` 将返回一个 `PeopleResultApi` 的 promise。将这些结果值分配给名为 `dataPromise` 和 `peoplePromise` 的变量。

1.  使用 `dataPromise` 和 `peoplePromise` 作为参数调用静态 `Promise.all` 方法。这将基于这两个值创建另一个 promise，并且只有当包含在内的所有 promise 都成功解析时，这个 promise 才会成功解析。它的返回值将是一个包含结果的数组 promise。

1.  从处理程序返回由 `Promise.all` 调用生成的 promise。

1.  在链中添加另一个 `then` 处理程序。此处理程序将 `Promise.all` 返回的数组作为单个参数。

1.  将参数解构为两个变量。数组的第一元素应该是 `movieData` 变量，其类型为 `MovieResultApi`，数组的第二元素应该是 `peopleData` 变量，其类型为 `PeopleResultApi`。

1.  人物数据有 `cast` 和 `crew` 属性。我们只取前六个演员，所以首先根据演员的 `order` 属性对 `cast` 属性进行排序。然后从新数组中截取前六个演员。

1.  将演员数据（即 `CastResultApi` 对象）转换为你的 `Character` 对象。我们需要将 `CastResultApi` 的 `character` 字段映射到 `Character` 的 `name` 字段，将 `name` 字段映射到演员名字，将 `profile_path` 字段映射到 `image` 属性。

1.  从人物数据的`crew`属性中，我们只需要导演和编剧。由于可能有多个导演和编剧，我们将分别连接所有导演和编剧的名字。对于导演，从`crew`属性中，筛选出那些有`Directing`部门且职位为`Director`的人。对于这些对象，取出`name`属性，并用`&`符号连接起来。

1.  对于作家，从`crew`属性中筛选出那些有`Writing`部门且职位为`Writer`的人。对于这些对象，取出`name`属性，并用`&`符号连接起来。

1.  创建一个新的`Movie`对象（使用对象字面量语法）。使用我们迄今为止准备的电影和人物响应数据填写`Movie`对象的全部属性。

1.  从处理器返回`Movie`对象。

1.  注意，我们在代码中没有进行任何 UI 交互。我们只是接收了一个字符串，进行了一些 promise 调用，并返回了一个值。现在，UI 工作可以在面向 UI 的代码中完成。在这种情况下，是在`search`按钮的`click`事件处理器中。我们应该简单地为`search`调用添加一个`then`处理器，该处理器将调用`showResults`方法，并添加一个`catch`处理器，该处理器将调用`clearResults`方法。

尽管我们在这次活动中使用了`fetch`和 promise，并且我们的代码现在更加高效但复杂，网站的基本功能将保持不变，你应该会看到一个与之前活动类似的输出。

注意

本活动的代码文件可以在[`packt.link/IeDTF`](https://packt.link/IeDTF)找到。本活动的解决方案可以通过这个链接找到。

## async/await

Promises 很好地解决了回调问题。然而，它们往往伴随着许多不必要的冗余。我们需要编写很多`then`调用，并且要小心不要忘记关闭任何括号。

下一步是向我们的 TypeScript 技能中添加一点语法糖。与其他章节中的内容不同，这个特性最初源于 TypeScript，后来也被 JavaScript 采纳。我指的是`async`/`await`关键字。这两个关键字是分开的，但它们总是一起使用，所以整个特性被称为`async`/`await`。

我们可以给某个函数添加`async`修饰符，然后，在那个函数中，我们可以使用`await`修饰符轻松地执行 promise。让我们再次回到我们的莎士比亚示例，并将我们用来调用`getShakespeareData`的代码包裹在一个名为`run`的函数中：

```js
function run() {
    getShakespeareData()
    .then(author => {
        const body = document.getElementsByTagName("body")[0];

        const image = document.createElement("img");
        image.src = `http://covers.openlibrary.org/a/id/${author.photos[0]}-M.jpg`;
        body.appendChild(image);
        const name = document.createElement("h1");
        name.innerHTML = author.personal_name;
        body.appendChild(name);
    })
    .catch(error => {
        console.log(`An error occured ${error.error}: ${error.message}`);
    })
}
run();
```

这段代码在功能上与之前的代码等效。但现在，我们有一个可以标记为`async`函数的函数，如下所示：

```js
async function run() {
```

现在，我们可以直接获取 promise 的结果并将其放入变量中。所以整个`then`调用将变成这样：

```js
    const author = await getShakespeareData();
    const body = document.getElementsByTagName("body")[0];

    const image = document.createElement("img");
    image.src = `http://covers.openlibrary.org/a/id/${author.photos[0]}-M.jpg`;
    body.appendChild(image);

    const name = document.createElement("h1");
    name.innerHTML = author.personal_name;
    body.appendChild(name);
```

你可以看到我们不再有任何包装函数调用了。可以将`catch`调用替换为简单的`try`/`catch`结构，最终版本的`run`函数将如下所示：

```js
async function run () {
  try {
    const author = await getShakespeareData();
    const body = document.getElementsByTagName("body")[0];

    const image = document.createElement("img");
    image.src = `http://covers.openlibrary.org/a/id/${author.photos[0]}-M.jpg`;
    body.appendChild(image);

    const name = document.createElement("h1");
    name.innerHTML = author.personal_name;
    body.appendChild(name);
  } catch (error) {
    console.log(`An error occured ${error.error}: ${error.message}`);
  }
}
```

你会注意到深度嵌套的代码量大大减少了。现在我们可以查看代码，并从快速浏览中了解它做什么，这仍然是相同的深度异步代码，唯一的区别是它看起来几乎是同步的，并且绝对是顺序的。

## 练习 10.05：使用 async 和 await 计数到五

在这个练习中，我们将创建一个函数，当执行时，将输出英语单词一至五。每个单词将在上一个单词显示后 1 秒出现在屏幕上：

注意

该练习的代码文件可以在[`packt.link/TaH6b`](https://packt.link/TaH6b)找到。

1.  创建一个新的文件，`counting-3.ts`。

1.  在`counting-3.ts`中，添加一个包含从一至五（包括五）的英语数字名称的数组：

    ```js
    const numbers = ["One", "Two", "Three", "Four", "Five"];
    ```

1.  添加`setTimeout`函数的 promisified 包装器。此包装器仅在指定的超时时间到期时执行：

    ```js
    const delay = (ms: number) => {
        const result = new Promise<void>((resolve, reject) => {
            setTimeout(() => {
                resolve();
            }, ms)
        });
        return result;
    }
    ```

    由于我们的承诺不会返回任何有意义的成果，因此我们不是在给定数量的毫秒后简单地解析，而是提供了`void`作为其类型。

1.  创建一个名为`countNumbers`的空`async`函数，并在文件的最后一行执行它：

    ```js
    async function countNumbers() {
    }
    countNumbers();
    ```

1.  在`countNumbers`函数内部，使用参数`1000`等待`delay`方法，并在其解决后打印出第一个数字：

    ```js
    async function countNumbers() {
        await delay(1000);
        console.log(numbers[0]);
    }
    ```

1.  保存文件，并使用以下命令编译它：

    ```js
    tsc counting-3.ts
    ```

1.  验证编译是否成功完成，并在同一文件夹中生成一个`counting-3.js`文件。使用以下命令在`node`环境中执行它：

    ```js
    node counting-3.js
    ```

    你会看到输出看起来像这样：

    ```js
    One
    ```

    该行应在应用程序运行后 1 秒出现。

1.  在`counting-3.ts`文件中，在`console.log`行之后，添加两行以显示剩余的数字。代码应如下所示：

    ```js
    async function countNumbers() {
        await delay(1000);
        console.log(numbers[0]);
        await delay(1000);
        console.log(numbers[1]);
        await delay(1000);
        console.log(numbers[2]);
        await delay(1000);
        console.log(numbers[3]);
        await delay(1000);
        console.log(numbers[4]);
    }
    ```

1.  再次编译并运行代码，并验证输出是否按正确顺序出现。

    由于所有数字的代码完全相同，因此用`for`循环替换它是微不足道的。

1.  在`counting-3.ts`文件中，删除`countNumbers`函数的主体，并用一个`for`循环替换它，该循环对`numbers`数组中的每个数字，将等待一秒钟的延迟，然后打印该数字：

    ```js
    for (const number of numbers) {
        await delay(1000);
        console.log(number);
    };
    ```

1.  再次编译并运行代码，并验证输出是否按正确顺序出现：

    ```js
    One
    Two
    Three
    Four
    Five
    ```

## 活动 10.03：使用 fetch 和 async/await 的影片浏览器

在这个活动中，我们将改进上一个活动。主要区别在于，我们不会使用`Promises`类的`then`方法，而是使用`await`关键字来神奇地完成它。我们将没有`then`调用链，而只有看起来完全正常的代码，其中穿插着一些`await`语句。

该活动与上一个活动具有相同的文件和资源。

以下步骤应有助于您解决问题：

1.  在`script.ts`文件中，找到`search`函数并验证它是否接受一个字符串参数，并且其主体为空。请注意，此函数现在已标记为`async`关键字，这允许我们使用`await`运算符。

1.  在`search`函数上方创建一个名为`getJsonData`的辅助函数。此函数将使用`fetch` API 从端点获取数据并将其格式化为 JSON。它应接受一个名为`url`的单个字符串参数，并且它应该返回一个 promise。

1.  在`getJsonData`函数的主体中，添加调用带有`url`参数的`fetch`函数的代码，然后调用返回响应上的`json`方法。

1.  在`search`方法中，使用`getSearchUrl`方法构造一个新的搜索结果 URL。

1.  使用`searchUrl`作为参数调用`getJsonData`函数，并`await`结果。将结果放入`SearchResultApi`变量中。

1.  检查我们是否有任何结果，如果没有，则抛出错误。如果有结果，将`result`属性的第一个项目设置到一个名为`movieResult`的变量中。此对象将包含电影的`id`和`title`属性。

1.  使用`id`属性调用`getMovieUrl`和`getPeopleUrl`方法，分别获取电影详情和演员及制作组的正确 URL。

1.  获取 URL 后，使用两个 URL 调用`getJsonData`函数，并将结果值分配给变量。请注意，`getJsonData(movieUrl)`调用将返回一个`MovieResultApi`的 promise，而`getJsonData(peopleUrl)`将返回一个`PeopleResultApi`的 promise。将这些结果值分配给名为`dataPromise`和`peoplePromise`的变量。

1.  使用`dataPromise`和`peoplePromise`作为参数调用静态`Promise.all`方法。这将基于这两个值创建另一个 promise，并且只有当包含在内的所有 promise 都成功解决时，这个 promise 才会成功解决。它的返回值将是一个包含结果的 promise 数组。`await`这个 promise，并将其结果放入一个类型为`array`的变量中。

1.  将该数组解构为两个变量。数组的第一个元素应该是`movieData`变量，其类型为`MovieResultApi`，数组的第二个元素应该是`peopleData`变量，其类型为`PeopleResultApi`。

1.  人员数据有`cast`和`crew`属性。我们只取前六个演员，所以首先根据演员的`order`属性对`cast`属性进行排序。然后从第一个六个演员中切出新的数组。

1.  将演员数据（即`CastResultApi`对象）转换为我们的`Character`对象。我们需要将`CastResultApi`的`character`字段映射到`Character`的`name`字段，将`name`字段映射到演员名称，将`profile_path`字段映射到`image`属性。

1.  从人物数据的 `crew` 属性中，我们只需要导演和编剧。由于可能有多个导演和编剧，我们将获取所有导演和编剧的名字并将它们分别连接起来。对于导演，从 `crew` 属性中筛选出具有 `Directing` 部门和 `Director` 职位的个人。对于这些对象，取其 `name` 属性，并用 `&` 连接起来。

1.  对于编剧，从 `crew` 属性中筛选出具有 `Writing` 部门和 `Writer` 职位的个人。对于这些对象，取其 `name` 属性，并用 `&` 连接起来。

1.  创建一个新的 `Movie` 对象（使用对象字面量语法）。使用我们迄今为止准备的电影和人物响应数据，填写 `Movie` 对象的所有属性。

1.  从函数中返回 `Movie` 对象。

1.  注意，在我们的代码中我们没有进行任何 UI 交互。我们只接收了一个字符串，进行了一些承诺调用，并返回了一个值。现在，UI 的工作可以在面向 UI 的代码中完成。在这种情况下，是在 `search` 按钮的 `click` 事件处理器中。我们应该简单地给 `search` 调用添加一个 `then` 处理器，该处理器将调用 `showResults` 方法，并添加一个 `catch` 处理器，该处理器将调用 `clearResults` 方法。

    尽管我们在这次活动中使用了 `fetch` 和 `async`/`await`，与之前的活动相比，我们的代码现在既高效又简单，但网站的基本功能将保持不变，你应该会看到与之前活动类似的结果。

    注意

    本活动的代码文件可以在 [`packt.link/fExtR`](https://packt.link/fExtR) 找到。本活动的解决方案可以通过 此链接 获取。

# 摘要

在本章中，我们探讨了在网络上使用的执行模型，以及我们如何使用它来实际执行代码。我们简要地了解了异步开发的复杂性表面——以及我们如何使用它从外部资源加载数据。我们展示了当我们深入回调的陷阱时出现的问题，并设法使用承诺退出。最后，我们能够等待异步代码的执行，并兼得两者之长——代码看起来像是同步的，但实际上是异步执行的。

我们还通过创建一个电影数据查看器浏览器来测试本章中开发的技能，最初使用 XHR 和回调，然后逐步使用 `fetch` 和承诺进行改进，最后使用 `fetch` 和 `async`/`await`。

下一章将教你关于高阶函数和回调的知识。
