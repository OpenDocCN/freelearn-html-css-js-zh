# 第三章：异步 JavaScript

## 学习目标

在本章结束时，您将能够：

+   定义异步编程

+   描述 JavaScript 事件循环

+   利用回调函数和 promises 来编写异步代码

+   使用 async/await 语法简化异步代码

在本章中，我们将学习异步 JavaScript 及其用途。

## 介绍

在上一章中，我们涵盖了 ES6 中发布的许多新功能和强大功能。我们讨论了 JavaScript 的发展，并突出了 ES6 中的关键添加。我们讨论了作用域规则、变量声明、箭头函数、模板文字、增强对象属性、解构赋值、类和模块、转译以及迭代器和生成器。

在本章中，我们将学习什么是异步编程语言，以及如何编写和理解异步代码。在第一个主题中，我们将定义异步编程，并展示 JavaScript 是一种异步、事件驱动的编程语言。然后，我们将概述回调函数，并展示如何使用回调函数来编写异步 JavaScript。然后，我们将定义 promises，并演示如何使用 promises 来编写异步 JavaScript。在最后一个主题中，我们将介绍 async/await 语法，并使用 promises 和这种语法简化我们的异步代码。

## 异步编程

JavaScript 是单线程、事件驱动、异步编程语言。这意味着 JavaScript 在单个线程上运行，并通过事件队列延迟/处理某些事件或函数调用。我们将通过以下主题来分解 JavaScript 如何做到这一点的基础知识。

### 同步与异步

代码是同步还是异步意味着什么？这两个词在 JavaScript 中经常被提及。**同步**源自希腊词根**syn**，意思是"与"，**chronos**，意思是"时间"。同步字面上意味着"与时间"，或者说，与时间协调的代码。代码一次运行一行，并且在处理完前一行之前不会开始下一行。**异步**或**async**源自希腊词根*async*，意思是"不与"，chronos，因此异步字面上意味着"不与时间"，或者说，与解释器首次遇到代码行的时间不协调的代码。运行的代码顺序与解释器首次遇到代码行的时间不协调。

### 同步与异步的时间控制

有两种类型的代码——**同步**和**异步**。我们将在本节中涵盖它们。

在异步 JavaScript 中，JavaScript 引擎以不同的方式处理慢速和快速代码。我们知道"快"和"慢"这两个词的含义，但这在我们的代码中如何实际应用呢？异步 JavaScript 允许线程在等待来自慢速时间相关操作的响应时执行新的代码行。例如文件系统 I/O。要理解这一点，我们必须了解一些关于计算机操作速度的知识。

CPU 非常非常快，可以处理每秒数百万到数十亿次操作。计算机或网络的其他部分比 CPU 慢得多。例如，硬盘每秒只能执行数百到数千次操作，计算机网络可能每秒只能执行一次操作。对内存的调用比 CPU 周期慢得多个数量级。

硬盘操作比内存操作慢几个数量级。网络调用比硬盘调用慢几个数量级。

在**同步代码**中，我们一次执行一行代码。下一行代码直到前一行代码完成运行后才执行。由于同步代码一次只执行一行代码并等待操作完成后才开始新的一行，如果我们的代码向较慢的介质（如内存、硬盘或网络）发出请求，程序将不会继续执行下一行代码，直到慢介质（HDD、网络等）的请求完成。CPU 将空闲，浪费宝贵的时间，等待操作完成。在网络调用的情况下，这可能需要几秒钟。在编写复杂的同步代码时，程序员通常编写多线程代码。操作系统会在一个线程等待缓慢操作时切换到其他线程。这有助于减少 CPU 的空闲时间。

在**异步代码**中，我们可以按非时间顺序执行代码行。这意味着我们可以在前一行代码完成操作之前开始处理新的代码行。JavaScript 通过事件循环实现这一点，这将在本章后面介绍。

在异步代码中，当 JavaScript 引擎遇到使用缓慢的、非 CPU 依赖操作的代码行时，操作会被启动，而不是等待完成，程序会继续执行下一行代码并继续运行。当缓慢操作完成时，CPU 会跳回到该操作，处理操作的响应，然后继续运行之前的代码。这样可以让 CPU 不浪费宝贵的资源等待可能需要几秒钟的操作。下图显示了同步和异步时间图的示例：

![图 2.1：同步与异步时间图](img/Figure_2.1.jpg)

###### 图 2.1：同步与异步时间图

在上图中，我们有四个操作：A、B、C 和 D。操作 C 调用网络并在完成之前有延迟，由网络延迟表示。在同步示例中，我们按顺序运行每个操作。当到达操作 C 时，我们必须等待网络延迟才能完成操作 C。操作 C 完成后，我们运行操作 D。在此等待期间，CPU 处于空闲状态，无法进行其他工作。

在异步示例中，我们按顺序运行前三个操作。当到达操作 C 时，不会等待网络延迟，而是运行操作 D。当网络延迟结束时，我们完成操作 C。在异步示例中，我们可以清楚地看到所有操作的整体完成时间和 CPU 空闲时间都更短。

如果这个概念还有点混乱，我们可以用现实生活中的情况来解释。想象同步代码就像在火车站排队买票的人群。一次只能有一个人使用售票机。在我前面的人都买完票之前，我无法从机器上取票。同样，站在我后面的人在我取票之前也无法开始取票。即使我前面的人决定花五分钟来取票，我也得等到轮到我。就像排队买票一样，同步代码一次只运行一步，按顺序进行。无论一步需要多长时间，都不会运行新的代码行，直到前一步完成。

异步代码更像是在餐厅用餐。每位顾客依次点餐，并且必须等待厨房烹饪订单。订单完成烹饪后会被上菜，但不是按照它们被厨房接收的顺序。烹饪时间较短的订单可能会在烹饪时间较长的订单之前上菜。这与异步代码非常相似。每个异步代码操作，或者我们例子中的食物订单，都是按顺序开始的。当操作等待响应时，可以开始下一个操作。CPU 可以在等待前一个操作的响应时处理其他操作。这显然与同步代码不同。如果厨房以同步方式运行，你将无法在厨房完成前一个订单的烹饪之前点餐。想象一下这会有多低效！

### 引入事件循环

由于其异步事件循环特性，JavaScript 是一个事件驱动、异步、单线程语言。在 JavaScript 中，异步操作以事件的形式处理。当我们进行异步调用时，一旦调用完成，就会触发一个事件。然后 JavaScript 引擎通过调用回调函数来处理该事件，然后继续执行代码中的下一个操作。

**事件循环**是我们用来管理 JavaScript 中所有操作的四部分系统的名称。这个系统的部分包括堆栈、堆、事件队列和（主）事件循环。堆栈、堆和事件队列都是 JavaScript 引擎维护的数据结构。主事件循环是在后台运行并管理这三个数据结构的过程。在其最简单的形式中，这个系统很容易理解。堆栈跟踪函数调用。当函数进行异步操作时，它会将事件处理程序放入堆中。当异步操作完成时，事件被推送到事件队列中。事件循环轮询队列以获取事件，然后从堆中获取相关的处理程序，然后调用函数并将其添加到堆栈中。这是事件循环的最基本形式。事件循环数据结构的可视化表示如下：

![图 2.2：事件循环数据结构可视化模型](img/Figure_2.2-01.jpg)

###### 图 2.2：事件循环数据结构可视化模型

这就是事件循环的最简单形式——三个数据结构：一个用于跟踪函数调用，一个用于跟踪事件处理程序，一个用于跟踪事件完成，以及一个循环将它们全部连接在一起。这些各个部分将在接下来的小节中进行更详细的讨论。

### 堆栈

JavaScript 引擎有一个单一的调用堆栈，事件循环堆栈。**事件循环堆栈**是一个传统的调用堆栈——它跟踪当前正在执行的函数以及在之后要执行的函数。堆栈中保存的函数被称为帧。事件循环采用先进后出的方式。它本质上是一种类似数组的数据结构，具有特殊的限制。函数帧只能从堆栈顶部添加和移除，就像厨房里的一叠盘子。放在堆栈上的第一项始终在底部，这将是最后一个被取走的。

堆栈跟踪堆栈顶部的当前执行函数以及较低级别的函数调用链。当函数被执行时，会创建一个帧并添加到堆栈顶部。当函数执行完成时，其帧会从堆栈顶部移除。这些帧包含函数、参数和局部变量。

如果一个函数 A 调用另一个函数 B，那么为新执行的函数 B 会创建一个新的帧。函数 B 的新帧会被放在堆栈的顶部，即调用它的函数 A 的帧的顶部。当函数 B 执行完成时，它的帧会从堆栈中移除，函数 A 的帧现在位于顶部。函数 A 继续执行，直到完成，完成后它的帧被移除。以下代码片段和图示例了这一点。

考虑以下代码片段：

```php
function foo( x ) { return 2 * x; }
function bar( y ) { return foo( y + 5 ) - 10; }
console.log( bar( 15 ) ); // Expected output: 30
```

###### 片段 2.1：调用堆栈示例代码

程序启动时，会创建第一个帧。该帧包含全局状态。然后，当调用`console.log`时，会创建第二个帧。该帧被放置在全局帧的顶部。当调用`bar`函数时，会创建第三个帧并添加到堆栈中。该帧包含`bar`的参数和局部变量。当 bar 调用`foo`时，会在 bar 帧的顶部添加第四个帧。完整的调用堆栈如下图所示：

![图 2.3：调用堆栈](img/Figure_2.3.jpg)

###### 图 2.3：调用堆栈

当`foo`返回时，它的帧从堆栈中移除。堆栈现在只包含一个包含 bar 的参数和变量、`console.log`调用和全局帧的帧。当**bar**返回时，它的帧从堆栈中移除，堆栈只包含最后 2 个帧。

### 堆和事件队列

**堆**

**堆**是一个大的、大部分是无结构的内存块，用于跟踪事件完成时应调用哪些函数。当启动异步操作时，它会被添加到堆中。一旦异步操作完成，项目就会从堆中移除。当异步操作完成时，堆会将必要的数据推送到事件队列中。

**队列**

**队列**是用于跟踪异步事件完成的消息队列。它是一个传统的先进先出队列。这意味着它是一个类似数组的数据结构，其中项目被推到队列的末尾并从队列的前端移除。最旧的项目首先被移除和处理。

消息队列中的每条消息都有一个关联的函数，当处理消息时会调用该函数。要处理消息，它会从队列中移除，并以消息的数据作为输入参数调用相应的函数。预期地，当调用函数时会创建一个新的堆栈帧。

让我们考虑一个网页中有两个按钮`button1`和`button2`，设置为使用`clickHandler`处理函数处理点击事件。用户快速点击`button1`和`button2`。事件队列将包含以下简化信息：

```php
Queue: { event: 'click', target: 'button1', handler: clickHandler }, { event: 'click', target: 'button2', handler: clickHandler }
```

###### 片段 2.2：调用堆栈示例代码

### 事件循环

**事件循环**负责处理事件队列中的消息。它通过一个不断的轮询循环来实现这一点。在事件循环的每个“tick”中，事件队列最多执行三件事：检查堆栈，检查队列，等待。

#### 注意

事件队列的“tick”是同步调用与 JavaScript 事件相关的零个或多个回调函数。这是处理事件并运行相关回调的时间。

在每个时刻，事件循环首先检查调用栈是否为空，以及我们是否可以做其他工作。如果调用栈不为空，事件队列将等一会儿，然后再次检查。如果调用栈为空，事件循环将检查事件队列以处理事件。如果事件队列为空，那么我们没有工作要做，事件循环将等待下一个时刻，然后重新开始这个过程。如果有事件要处理，事件循环将从事件队列中取消息并调用与消息相关联的函数。被调用的函数在栈上创建一个帧，JavaScript 引擎开始执行函数指定的工作。事件循环继续其轮询循环。

观察事件循环轮询，我们可以注意到一次只能处理一个事件。如果调用栈中有任何内容，事件循环将不会从事件队列中取出消息。这个功能被称为**运行到完成**。每条消息在任何其他消息开始处理之前都会被完全处理。

**运行到完成**在编写应用程序时提供了一些好处。其中一个好处是函数不能被抢占，将在任何其他代码运行之前运行，可能修改函数正在操作的数据。

然而，这种模式的缺点是，如果代码中的事件回调或循环花费很长时间才能完成，应用程序可能会延迟其他待处理的事件。在浏览器中，用户交互事件如点击或滚动可能会因为另一个事件回调花费很长时间而挂起。在服务器端代码中，数据库查询或 HTTP 请求的结果可能会因为另一个事件回调花费很长时间而挂起。

确保由事件调用的回调函数很短是一个良好的实践。长回调函数可以使用`setTimeout`函数分成几条消息。延迟问题的示例如下所示：

```php
setTimeout( () => { 
  // WARNING: this may take a long time to run on a slow computer
  // Try with smaller numbers first
  for( let i = 0; i < 2000000000; i++ ) {}
  console.log( 'done delaying' );
}, 0 );
setTimeout( () => { console.log( 'done!' ) }, 0 );
```

###### 代码片段 2.3：阻塞循环示例

在前面的例子中，我们使用`setTimeout`创建了两个异步调用。第一个计数到 20 亿，然后记录`done delaying`，第二个只记录`done!`。当第一个消息从事件队列中取出时，回调被放入调用栈。在大多数计算机上，计数到 20 亿会导致明显的延迟。

#### 注意

如果你的电脑比较旧，那么这种延迟可能会很长。如果你运行这段代码，从较小的数字开始，比如 200 万。

当计算机在计数时，事件循环不会从事件队列中取出下一个消息。对`done!`的异步调用将在计数完成后才会得到处理。要小心，因为制作回调函数可能需要很长时间。如果被阻塞的`console.log('done!')`回调是网站中的用户输入事件，网站将阻塞用户输入，可能导致用户不满意，甚至可能失去宝贵的用户。

### 需要考虑的事情

在处理事件循环时，我们在编写异步代码时有三个重要的事情要考虑。第一件要考虑的事情是事件可能会出现不同步。第二个是同步代码是阻塞的。第三个是零延迟函数不会在 0 毫秒后执行。这三个概念如下所述：

**事件可能会出现无序**

+   事件按照它们发生或解决的顺序添加到事件队列中。

+   这可能不是异步调用启动的顺序。

+   如果一个异步操作很慢，在它完成之前触发的事件将首先得到处理。

+   我们必须考虑回调和承诺的程序定时。

+   我们必须确保在数据可用之前不要访问由异步调用填充的数据。

**同步代码是阻塞的**

+   通过使用执行相同或类似任务的同步模块来避免异步代码是非常不好的做法。

+   JavaScript 是单线程的。

+   如果使用大量同步代码，事件消息可能无法及时处理。

+   例如鼠标点击或滚动等事件可能会挂起。

**零延迟函数实际上不会在 0 毫秒后执行**

+   `setTimeout`在超时后将事件添加到事件队列中。

+   如果事件队列有很多消息要处理，超时消息可能需要几毫秒才能得到处理。

+   延迟参数表示的是最小时间，而不是保证时间。

零延迟函数和事件循环状态的概念可以在以下片段中得到展示：

```php
setTimeout( () => { console.log( 'step1' ) }, 0 );
setTimeout( () => { console.log( 'done!' ) }, 0 );
console.log( 'step0' );
//Expected output:
// step0
// step1
// done!
```

###### 片段 2.4：处理异步代码

在前面的片段中，我们看到主代码文件中有工作要做。运行主程序体，并向调用堆栈添加一个帧。然后解释第一行代码，`setTimeout`函数将其回调添加到堆中，并安排在 0 毫秒后触发事件。然后事件触发，消息被添加到事件队列。JavaScript 引擎解释下一行代码，即第二个`setTimeout`调用。回调被添加到堆中，并注册在 0 毫秒后触发事件。第二个超时事件立即触发，并将第二个消息添加到事件队列。JavaScript 引擎处理`console.log`调用，并将`step0`记录到控制台。主程序体没有更多同步工作要做，调用堆栈为空。事件循环现在开始处理事件队列中的事件。事件队列包含两条消息，一条是第一个超时事件的消息，另一条是第二个超时事件的消息。然后事件循环获取第一条消息，并将相关的`callback`函数添加到调用堆栈。JavaScript 引擎处理该调用堆栈帧并记录`step1`。然后 JavaScript 引擎处理事件队列中的第二条消息。事件队列消息从队列中移除，并向调用堆栈添加一个帧。JS 引擎处理堆栈中的帧并记录`done!`。没有更多的工作可以做了。所有事件都已触发，堆栈和队列都为空。

**结论**

与大多数编程语言不同，JavaScript 是一种异步编程语言。更具体地说，它是一种单线程、事件驱动的异步编程语言。这意味着 JavaScript 在等待长时间运行操作的结果时不会空闲。它在等待时运行其他代码块。JavaScript 通过事件循环来管理这一点。事件循环由四个部分组成，即函数堆栈、内存堆、事件队列和事件循环。这四个部分共同处理来自操作完成的事件。

### 练习 16：使用事件循环处理堆栈

为了更好地理解程序中事件按预期顺序触发和处理的原因，请查看下面提供的程序，并在不运行程序的情况下，写出程序的预期输出。

对于程序的前 10 个步骤，在每个步骤写出预期的堆栈、队列和堆。步骤是指事件触发时，事件循环出列一个事件，或 JS 引擎处理函数调用的任何时间：

```php
step 0
stack: <global>
queue: <empty>
heap: <empty>
```

###### 片段 2.5：调用堆栈示例代码（起始步骤）

程序显示在以下片段中：

```php
function f1() { console.log( 'f1' ); }
function f2() { console.log( 'f2' ); }
function f3() {
  console.log( 'f3' );
  setTimeout( f5, 90 );
}
function f4() { console.log( 'f4' ); }
function f5() { console.log( 'f5' ); }
setTimeout( f1, 105 );
setTimeout( f2, 15 );
setTimeout( f3, 10 );
setTimeout( f4, 100 );
```

###### 片段 2.6：调用堆栈示例代码（程序）

为了演示事件循环处理堆栈、队列和堆的简化形式，执行以下步骤：

1.  如果调用并处理函数，则向堆栈中添加事件循环堆栈帧。

处理函数并将必要的事件和处理程序信息添加到堆中。在下一步中移除事件和处理程序。

1.  如果事件完成，则将其推送到事件队列中。

1.  从事件队列中拉取并调用处理程序函数。

1.  对程序的其余步骤重复此操作（仅限前 10 步）。

**代码**

https://bit.ly/2R5YGPA

**结果**

![图 2.4：作用域输出]

](image/Figure_2.4.jpg)

###### 图 2.4：作用域输出

![图 2.5：作用域输出](img/Figure_2.5.jpg)

###### 图 2.5：作用域输出

![图 2.6：作用域输出](img/Figure_2.6.jpg)

###### 图 2.6：作用域输出

您已成功演示了事件循环如何处理堆栈的简化形式。

## 回调

**回调**是 JavaScript 异步编程的最基本形式。简单来说，回调是在另一个函数完成后被调用的函数。回调用于处理异步函数调用的响应。

在 JavaScript 中，函数被视为对象。它们可以作为参数传递，被函数返回，并保存到变量中。回调是作为参数传递到高阶函数中的函数对象。高阶函数简单地是一个数学和计算机科学术语，用于指代接受一个或多个函数作为参数（回调）或返回一个函数的函数。在 JavaScript 中，高阶函数将回调作为参数。一旦高阶函数完成某种形式的工作，比如 HTTP 请求或数据库调用，它将调用回调函数并传递错误或返回值。

如在*异步编程*中的事件循环部分所述，JavaScript 是一种事件驱动的语言。由于 JavaScript 是单线程的，任何长时间运行的操作都会阻塞。JavaScript 通过使用事件来处理这种阻塞效应。当操作完成并触发事件时，事件会有一个附加的处理程序函数来处理结果。这些函数就是**回调**。回调是允许 JavaScript 事件在处理异步事件时执行工作的关键。

### 构建回调

JavaScript 中的回调遵循一个简单的非官方约定。回调函数应至少接受两个参数：**error**和**result**。在构建回调 API 或编写回调函数时，我们建议您遵循这个约定，以便您的代码可以无缝地集成到其他库中。下面是一个回调函数的示例：

```php
TwitterAPI.listFollowers( { user_id: "example_user" }, (err, result) => {   
  console.log( err, result ); 
} );
```

###### 代码段 2.7：基本回调示例

在前面的示例中，我们使用了一个假的 Twitter API。我们的假 API 有一个高阶函数`listFollowers`，它接受一个对象和一个回调函数作为参数。一旦`listFollowers`完成其内部工作，比如在这种情况下是对 Twitter API 的 HTTP 请求，我们的回调函数将被调用。

回调可以接受高阶函数需要的或指定的任意数量的参数，但第一个参数必须是错误对象。几乎每个 API 都遵循这个约定。在编写 API 时违反这个约定将使您的代码更难与任何第三方 API 或应用程序集成。

如果高阶函数在运行时遇到错误，回调的错误参数将被设置。错误参数的内容可以是任何合法的 JavaScript 值。在大多数情况下，它是 Error 类的一个实例；然而，错误对象的内容没有约定。一些 API 可能返回一个对象、字符串或数字，而不是 Error 实例。请确保阅读任何第三方 API 的文档，以确保您的代码可以处理返回的错误格式。

如果高阶函数没有遇到错误，则错误参数应设置为 null。在构建自己的 API 时，建议您也遵循这个惯例。一些第三方 API 可能会返回一个不是 null 的假值，但这是不鼓励的，因为它会使错误处理逻辑变得更加复杂。

#### 注意

**Falsy**是 JavaScript 类型比较和转换中使用的术语。在 JavaScript 中，Falsy 值在类型比较时转换为布尔值 false。Falsy 值的示例包括 null、undefined、0 和布尔值 false。

回调函数的结果参数包含了高阶函数的评估结果。这可能是一个 HTTP 请求的结果，数据库查询的结果，或者任何其他异步操作的结果。当返回错误时，一些 API 还可能在结果字段中提供更详细的错误信息。重要的是不要假设函数成功完成，如果结果对象存在的话，你必须检查错误字段。

在处理回调函数中的错误时，我们必须检查错误参数。如果错误参数不是 null 或 undefined，那么我们必须以某种方式处理错误。下面的示例中显示了一个错误处理程序：

```php
TwitterAPI.listFollowers( { user_id: "example_user" }, (err, result) => {   
  if ( err ) {
    // HANDLE ERROR
  }
  console.log( err, result ); 
} );
```

###### 片段 2.8：基本回调错误处理

大多数开发人员会检查错误值是否为真值。如果`err`是真值，那么将执行错误处理代码。这是一种通用做法；然而，这是编码的懒惰方式。在某些情况下，错误对象可能是布尔值 false，数字 0，空字符串等。这些都会评估为假值，即使值不是 null 或 undefined。如果你正在使用 API，请确保它不会返回一个评估为假值的错误。如果你正在构建一个 API，我们不建议返回一个可能评估为假值的错误。

### 回调陷阱

回调很容易使用，并且非常有效地实现了它们的目的，但在使用回调时需要考虑一些陷阱。最常见的两个陷阱是回调地狱和回调存在假设。只要有远见地编写代码，这两个陷阱都很容易避免。

最常见的回调陷阱是**回调地狱**。在异步工作完成并调用回调后，回调函数可以调用另一个异步函数来进行更多的异步工作。当它调用新的异步函数时，将提供另一个回调。新的回调将嵌套在旧的回调内。回调嵌套的示例在下面的片段中显示：

```php
TwitterAPI.listFollowers( { user_id: "example_user" }, (err, result) => { 
  if ( err ) { throw err; }
  TwitterAPI.unfollow( { user_id: result[ 0 ].id }, ( err, result ) => {
    if ( err ) { throw err; }
    console.log( "Unfollowed someone!" );
  } );
 } );
```

###### 片段 2.9：回调嵌套

在前面的片段中，我们有嵌套的回调。第一个异步操作的回调`listFollowers`调用了第二个异步操作。取消关注操作也有一个回调，只是处理错误或记录文本。由于回调可以嵌套，经过几层嵌套后，代码可能变得很难阅读。这就是回调地狱。回调地狱的示例在下面的片段中显示：

```php
TwitterAPI.listFollowers( { user_id: "example_user" }, (err, result) => { 
  const [ id1, id2, id3 ] = [ result[ 0 ].id, result[ 1 ].id, result[ 2 ].id ];
  TwitterAPI.unfollow( { user_id: id1 }, ( err, result ) => {
    TwitterAPI.block( { user_id: id1 }, ( err, result ) => {
      TwitterAPI.unfollow( { user_id: id2 }, ( err, result ) => {
        TwitterAPI.block( { user_id: id2 }, ( err, result ) => {
          TwitterAPI.unfollow( { user_id: id3 }, ( err, result ) => {
            TwitterAPI.block( { user_id: id3 }, ( err, result ) => {
              console.log( "Unfollowed and blocked 3 users!" );
```

###### 片段 2.10：回调地狱

在前面的片段中，我们列出了我们的关注者，然后取消关注并阻止前三个关注者。这是非常简单的代码，但由于回调嵌套，代码变得更加混乱。这就是回调地狱。

#### 注意

回调地狱是关于代码呈现的凌乱，而不是其背后的逻辑。回调嵌套可能导致代码运行无误，但非常难以阅读。非常难以阅读的代码可能非常难以向新开发人员解释，或者在发生错误时进行调试。

### 修复回调地狱

回调地狱可以通过两种技巧轻松避免：**命名函数**和**模块**。命名函数非常简单；定义回调并将其分配给标识符（变量）。定义的回调函数可以保存在同一个文件中或放入一个模块并导入。在回调中使用命名函数将有助于防止回调嵌套使代码混乱。这在下面的示例中显示：

```php
function listHandler( err, result ) {
  TwitterAPI.unfollow( { user_id: result[ 0 ].id }, unfollowHandler );
}
function unfollowHandler( err, result) {
  TwitterAPI.block( { user_id: result.id }, blockHandler );
}
function blockHandler( err, result ) {
  console.log( "User unfollowed and blocked!" );
}
TwitterAPI.listFollowers( { user_id: "example_user" }, listHandler);
```

###### 片段 2.11：修复回调地狱

从前面的片段中可以看出，没有嵌套的代码要清晰得多。如果我们有 30 层的回调嵌套深度，使代码可读的唯一方法就是将回调拆分为命名函数。

另一个潜在的陷阱是回调函数的不存在。如果我们正在编写一个 API，我们必须考虑到 API 的用户可能不会将有效的回调函数传递给 API。如果预期的回调不是一个函数或不存在，那么尝试调用它将导致运行时错误。在尝试调用之前，验证回调存在且是一个函数是一个很好的做法。如果用户传入了无效的回调，那么我们可以优雅地失败。以下是一个示例：

```php
Function apiFunction( args, callback ){
  if ( !callback || !( typeof callback === "function" ) ){
    throw new Error( "Invalid callback. Provide a function." );
  }
  let result = {};
  let err = null;
  // Do work
  // Set err and result
  callback( err, result );
}
```

###### 代码片段 2.12：检查回调存在

在前面的代码片段中，我们检查了`callback`参数是否存在且为真，并且它是函数类型。如果回调不存在或不是函数，我们会抛出一个错误，让用户知道出了什么问题。如果`callback`是一个函数，我们继续。

**结论**

回调只是作为参数传递给另一个函数的函数，称为**高阶**函数。JavaScript 使用回调来处理事件。回调使用错误参数和结果参数进行定义。如果在高阶函数中出现错误，回调错误字段将被设置。如果高阶函数完成了结果，结果字段将包含已完成操作的结果。

在使用回调时，我们应该注意两个陷阱。我们必须小心不要嵌套太多的回调并创建回调地狱。我们必须确保验证传递给我们的高阶函数的参数，以确保回调是一个函数。

### 练习 17：使用回调

您的团队正在构建一个基于回调的 API。为了防止运行时错误，您需要验证传递给回调 API 函数的回调参数是否是有效的可调用函数。为您的 API 创建一个函数。在该函数的主体中，验证回调参数是否是一个函数。如果不是一个函数，抛出一个错误。延迟后，记录传递给 API 函数的数据并调用回调。

要构建一个具有回调函数的回调 API，请执行以下步骤：

1.  编写一个名为`higherOrder`的函数，该函数接受两个参数；一个名为`data`的对象和一个名为`cb`的回调函数。

1.  在函数中，检查回调是否是一个函数参数（`cb`是一个函数）。

如果`cb`不存在或者不是`function`类型，则抛出一个错误。

1.  在函数中，记录`data`对象。

1.  在函数中，延迟 10 毫秒后调用`callback`函数。

1.  在函数外部，创建一个`try-catch`块。

1.  在 try 部分内，使用一个数据对象和没有回调函数调用`higherOrder`函数。

1.  在 catch 部分内，捕获错误并记录我们收到的错误消息。

1.  在`try-catch`块之后，使用一个数据对象和一个`callback`函数调用`higherOrder`函数。回调函数应记录字符串`Callback Called!`

**代码**

##### Index.js

```php
function higherOrder( data, cb ) {
 if ( !cb || !( typeof cb === 'function' ) ) {
   throw new Error( 'Invalid callback. Please provide a function.' );
 }
 console.log( data );
 setTimeout( cb, 10 );
}
try {
 higherOrder( 1, null );
} catch ( err ) {
 console.log( 'Got error: ${err.message}' );
}
higherOrder( 1, () => {
 console.log( 'Callback Called!' )
} );
```

###### 代码片段 2.13：实现回调

https://bit.ly/2VTGG9L

**结果**

![图 2.7：回调输出](img/Figure_2.7.jpg)

###### 图 2.7：回调输出

您已成功构建了一个具有回调函数的回调 API。

## 承诺

在 JavaScript 中，**promise**是一个包装异步操作并在异步操作完成时通知程序的对象。承诺对象表示包装操作的最终完成或失败。承诺是一个代理值，不一定是已知的。它承诺在将来的某个时刻提供一个值，而不是立即提供值，就像同步程序一样。承诺允许您将成功和错误处理程序与异步操作关联起来。这些处理程序在包装的异步过程完成或失败时被调用。

### 承诺状态

每个 promise 都有一个状态。一个 promise 只能成功一次，带有一个值，或者失败一次，带有一个错误。promise 的状态定义了 promise 在朝向值的解决过程中的工作状态。

一个 promise 有三种状态：**pending**，**fulfilled**或**rejected**。一个 promise 开始于 pending 状态。这意味着 promise 内部进行的异步操作尚未完成。一旦异步操作完成，promise 被视为已解决，并将进入 fulfilled 或 rejected 状态。

当一个 promise 进入完成状态时，意味着异步操作已经完成，没有错误。promise 已经完成，并且有一个值可用。异步操作生成的值已经返回，并且可以使用。

当一个 promise 进入拒绝状态时，意味着异步操作已经以错误完成。当一个 promise 被拒绝时，将不会进行任何未来的工作，也不会提供任何值。异步操作的错误已经返回，并可以从 promise 对象中引用。

### 解决或拒绝一个 promise

通过实例化`Promise`类的新对象来创建一个 promise。promise 构造函数接受一个参数，一个函数。这个函数必须有两个参数：**resolve**和**reject**。下面的片段展示了 promise 的创建示例：

```php
const myPromise = new Promise( ( resolve, reject ) => {
  // Do asynchronous work here and call resolve or reject
} );
```

###### 片段 2.14：promise 创建语法

promise 的主要异步工作将在传递给构造函数的函数体中完成。`resolve`和`reject`是可以用来完成 promise 的函数。要完成带有错误的 promise，调用带有错误作为参数的 reject 函数。要标记 promise 为成功，调用`resolve`函数并将结果作为参数传递给 resolve。下面的两个片段展示了 promise 的拒绝和解决的例子：

```php
// Reject promise with an error
const myPromise = new Promise( ( resolve, reject ) => {
  // Do asynchronous work here
  reject( new Error( 'Oh no! Promise was rejected' ) );
} );
```

###### 片段 2.15：拒绝一个 promise

```php
// Resolve the promise with a value
const myPromise = new Promise( ( resolve, reject ) => {
  // Do asynchronous work here
  resolve( { key1: 'value1' } );
} );
```

###### 片段 2.16：解决一个 promise

下面的片段展示了解决执行异步工作的 promise 的示例：

```php
const myPromise = new Promise( ( resolve, reject ) => {
  setTimeout( () => { resolve( 'Done!' ) }, 1000 )
} );
```

###### 片段 2.17：解决一个 promise

### 使用 Promises

promise 类有三个成员函数，可以用来处理 promise 的完成和拒绝。这些函数被称为 promise 处理程序。这些函数是`then()`，`catch()`和`finally()`。当一个 promise 完成时，其中一个处理程序函数被调用。如果 promise 完成，将调用`then()`函数。如果 promise 被拒绝，要么调用`catch()`函数，要么调用带有拒绝处理程序的`then()`函数。

`then()`成员函数旨在处理并获取 promise 的完成或拒绝结果。`then`函数接受两个函数参数，一个完成回调和一个拒绝回调。下面的例子展示了这一点：

```php
// Resolve the promise with a value or reject with an error
myPromise.then( 
  ( result ) => { /* handle result */ }, // Promise fulfilled handler
  ( err ) => { /* handle error here */ } // Promise rejected handler
 ) ;
```

###### 片段 2.18：Promise.then()语法

`then()`函数中的第一个参数是 promise 完成处理程序。如果 promise 以一个值完成，将调用 promise 完成处理程序回调。promise 完成处理程序接受一个参数。这个参数的值将是传递给 promise 函数体中完成回调的值。下面的片段展示了一个例子：

```php
// Resolve the promise with a value
const myPromise = new Promise( ( resolve, reject ) => {
  // Do asynchronous work here
  resolve( 'Promise was resolved!' );
} );
myPromse.then( value => console.log( value ) );
// Expected output: 'Promise was resolved'
```

###### 片段 2.19：使用已解决的 promise 的 Promise.then()

`then()`函数中的第二个参数是 promise 拒绝处理程序。如果 promise 以一个错误被拒绝，将调用 promise 拒绝处理程序回调。promise 拒绝处理程序接受一个参数。这个参数的值是传递给 promise 函数体中 reject 回调的值。下面的片段展示了一个例子：

```php
// Reject the promise with a value
const myPromise = new Promise( ( resolve, reject ) => {
  // Do asynchronous work here
  reject( new Error ( 'Promise was rejected!' ) );
} );
myPromse.then( () => {}, error => console.log( error) );
// Expected output: Error: Promise was rejected! 
// ** output stack trace omitted
```

###### 片段 2.20：使用 Promise.then()拒绝 promise

### 练习 18：创建和解决你的第一个 promise

要构建我们的第一个异步 promise，请执行以下步骤：

1.  创建一个 promise 并将其保存到名为`myPromise`的变量中。

1.  在 promise 的主体内，记录`开始异步工作！`

1.  在 promise 的主体内，使用超时进行异步工作。

让`timeout`回调在 1000 毫秒后触发。在`timeout`回调函数内，调用 promise 解决函数并传入值`完成！`

1.  将一个 then 处理程序附加到保存在`myPromise`中的 promise。

1.  将一个函数传递给 then 处理程序，该函数接受一个参数并记录参数的值。

**代码**

##### Index.js

```php
const myPromise = new Promise( ( resolve, reject ) => {
  console.log( 'Starting asynchronous work!' );
  setTimeout( () => { resolve( 'Done!' ); }, 1000 );
} );
myPromise.then( value => console.log( value ) );
```

###### 片段 2.21：使用 Promise.then()拒绝 Promise

https://bit.ly/2TVQNcz

**结果**

![图 2.8：作用域输出](img/Figure_2.8.jpg)

###### 图 2.8：作用域输出

你已成功利用你刚学到的语法来构建我们的第一个异步 promise。

### 处理 Promise

当调用`Promise.then()`时，它会返回一个处于挂起状态的新 promise。在已调用完成或拒绝的 promise 处理程序之后，`Promise.then()`中的处理程序会异步调用。当从`Promise.then()`调用的处理程序返回一个值时，该值将用于解决或拒绝`promise.then()`返回的 promise。以下表格提供了处理程序函数在任何阶段返回值、错误或 promise 时所采取的操作：

![图 2.9：返回一个 promise](img/Figure_2.9.jpg)

###### 图 2.9：返回一个 promise

`Promise.catch`接受一个参数，一个处理程序函数，用于处理 promise 的拒绝值。当调用`Promise.catch`时，内部会调用`Promise.then( undefined, rejectHandler )`。这意味着在内部，只调用了`Promise.then()`处理程序，只有 promise 拒绝回调`rejectHandler`，没有 promise 完成回调。`Promise.catch()`返回内部`Promise.then()`调用的值：

```php
const myPromise = new Promise( ( resolve, reject ) => {
  reject( new Error 'Promise was resolved!' );
} );
myPromise.catch( err => console.log( err ) );
```

###### 片段 2.22：使用 Promise.then()拒绝 Promise

promise 成员函数`Promise.finally()`是一个用于捕获所有 promise 完成情况的 promise 处理程序。`Promise.finally()`处理程序将被用于处理 promise 的拒绝和解决。它接受一个单一函数参数，在 promise 被拒绝或解决时调用。`Promise.finally()`将捕获被拒绝和解决的 promise，并运行指定的函数。它为我们提供了一个捕获所有情况的处理程序来处理任何完成情况。`Promise.finally()`应该用于防止在 then 和 catch 处理程序之间重复代码。传递给`Promise.finally()`的函数不接受任何参数，因此忽略了传递给 promise 的解决或拒绝的任何值。因为在使用`Promise.finally()`时没有可靠的区分拒绝和解决的方法，所以只有在我们不关心 promise 是否被拒绝或解决时才应该使用`Promise.finally()`。以下片段中显示了一个示例：

```php
// Resolve the promise with a value
const myPromise = new Promise( ( resolve, reject ) => {
  resolve( 'Promise was resolved!' );
} );
myPromse.finally( value => { 
  console.log( 'Finally!' );
 } );
// Expected output:
// Finally!
```

###### 片段 2.23：Promise.then()

在使用 promise 时，有时我们可能希望创建一个已经处于完成状态的 promise。Promise 类有两个静态成员函数，允许我们这样做。这些函数是`Promise.reject()`和`Promise.resolve()`。`Promise.reject()`接受一个参数，并返回一个已经被拒绝的带有传入拒绝函数值的 promise。`Promise.resolve()`接受一个参数，并返回一个已经被解决的带有传入解决值的 promise。

```php
Promise.resolve( 'Resolve value!' ).then( console.log );
Promise.reject( 'Reject value!' ).catch( console.log );
//Expected output:
// Resolve value!
// Reject value!
```

###### 片段 2.24：Promise.then()

### Promise 链

在使用承诺时，我们可能会遇到**承诺地狱**。这与回调地狱非常相似。当承诺主体在获得值后需要执行更多的异步工作时，可以嵌套另一个承诺。当嵌套链变得非常深时，嵌套的承诺调用可能变得难以跟踪。为了避免承诺地狱，我们可以将承诺链接在一起。`Promise.then()`、`Promise.catch()`和`Promise.finally()`都返回承诺，这些承诺将根据处理程序函数的结果被实现或拒绝。这意味着我们可以在这个承诺上附加另一个 then 处理程序，并创建一个承诺链来处理新返回的承诺。这在以下片段中显示：

```php
function apiCall1( result ) { // Function that returns a promise
 return new Promise( ( resolve, reject ) => { 
    resolve( 'value1' );
  } );
}
function apiCall2( result ) {// Function that returns a promise
  return new Promise( ( resolve, reject ) => { 
    resolve( 'value2' );
  } );
}
myPromse.then( apiCall1 ).then( apiCall2 ).then( result =>  console.log( 'done!') ) ;
```

###### 片段 2.25：承诺链接示例

在前面的示例中，我们创建了两个函数`apiCall1()`和`apiCall2()`。这些函数返回一个承诺，执行更多的异步工作。出于简洁起见，此示例中省略了异步工作。当原始承诺`myPromise`完成时，`Promise.then()`处理程序调用`apiCall1()`，它返回另一个承诺。第二个`Promise.then()`处理程序应用于这个新返回的承诺。当`apiCall1()`返回的承诺被解析时，处理程序函数调用`apiCall2()`，它也返回一个承诺。当`apiCall2()`返回的承诺被返回时，将调用最终的`Promise.then()`处理程序。如果这些具有异步工作的处理程序函数被嵌套，那么跟踪程序将变得非常困难。通过回调链接，跟踪程序流程变得非常容易。

在链接承诺时，承诺处理程序可以返回一个值，而不是一个新的承诺。如果返回一个值，该值将作为输入传递给链中的下一个`Promise.then()`处理程序。

例如，第一个承诺完成并调用`Promise.then()`处理程序。此处理程序执行同步工作并返回数字 10。下一个`promise.then()`处理程序将输入参数设置为 10，并可以继续执行异步工作。这允许您将同步步骤嵌入到承诺链中。

在链接承诺时，我们必须小心处理 catch 处理程序。当承诺被拒绝时，它会跳转到下一个承诺拒绝处理程序。这可以是`then`处理程序的第二个参数或`catch`处理程序。在承诺被拒绝的地方和下一个拒绝处理程序之间的所有实现处理程序都将被忽略。当 catch 处理程序完成时，由`catch()`返回的承诺将以拒绝处理程序的返回值被实现。这意味着下一个承诺实现处理程序将获得一个值来运行。如果`catch`处理程序不是承诺链中的最后一个处理程序，承诺链将继续以`catch`处理程序的返回值运行。这可能是一个棘手的错误调试；然而，它允许我们捕获承诺拒绝，以特定方式处理错误，并继续承诺链。它允许承诺链以不同的方式处理拒绝或接受，然后继续进行异步工作。这在以下片段中显示：

```php
// Promise chain handles rejection and continues
// apiCall1 is a function that returns a rejected promise
// apiCall2 is a function that returns a resolved promise
// apiCall3 is a function that returns a resolved promise
// errorHandler1 is a function that returns a resolved promise
myPromse.then( apiCall1 ).then( apiCall2, errorHandler1 ).then( apiCall3 ).catch( errorHandler2 );
```

###### 片段 2.26：处理错误并继续

在前面的片段中，我们有一个承诺链，其中有三个连续的异步 API 调用，在`myPromise`解决后。第一个 API 调用将拒绝带有错误的承诺。拒绝的承诺由第二个 then 处理程序处理。由于承诺被拒绝，它忽略了`apiCall2()`并路由到`errorHandler1()`函数。`errorHandler1()`将执行一些工作并返回一个值或承诺。该值或承诺传递给下一个处理程序，该处理程序调用`apiCall3()`，它返回一个解决的承诺。由于承诺已解决且没有更多的`then`处理程序，承诺链结束。最终的 catch 被忽略。

要从一个拒绝处理程序跳到下一个拒绝处理程序，我们需要在拒绝处理程序函数内部抛出一个错误。这将导致返回的 promise 被拒绝，并跳到下一个`catch`处理程序。

如果我们希望在 promise 被拒绝时提前退出 promise 链并且不继续，应该只在链的末尾包含一个 catch 处理程序。当 promise 被拒绝时，拒绝会被找到的第一个处理程序处理。如果这个处理程序是 promise 链中的最后一个处理程序，链就结束了。如下面的片段所示：

```php
// Promise chain handles rejection and continues
// apiCall1 returns a rejected promise
myPromse.then( apiCall1 ).then( apiCall2 ).then( apiCall3 ).catch( errorHandler1 );
```

###### 片段 2.27：在链的末尾处理错误以中止

在前面片段中显示的 promise 链中，当 myPromise 解析为一个值时，第一个`then`处理程序被调用。`apiCall1()`被调用并返回一个被拒绝的 promise。由于接下来的两个`then`处理程序没有处理 promise 拒绝的参数，拒绝被传递给`catch`处理程序。catch 处理程序调用`errorHandler1`，然后 promise 链结束。

链接 promise 用于确保所有 promise 按照链的顺序完成。如果 promise 不需要按顺序完成，我们可以使用`Promise.all()`静态成员函数。`Promise.all()`函数不是在 promise 类的实例上创建的。它是一个静态类函数。`Promise.all()`接受一个 promise 数组，当所有 promise 都解决时，将调用 then 处理程序。then 处理程序函数的参数将是原始`Promise.all()`调用中每个 promise 的解决值的数组。解决值的数组将与输入到`Promise.all()`的数组的顺序匹配。如下面的片段所示：

```php
// Create promises
let promise1 = new Promise( ( resolve, reject ) => setTimeout( () => resolve( 10 ), 100 ) );
let promise2 = new Promise( ( resolve, reject ) => setTimeout( () => resolve( 20 ), 200 ) );
let promise3 = new Promise( ( resolve, reject ) => setTimeout( () => resolve( 30 ), 10 ) );
Promise.all( [ promise1, promise2, promise3 ] ).then( results => console.log( results ) );
//Expected output: [ 10, 20, 30 ]
```

###### 片段 2.28：Promise.all()示例

在上面的例子中，我们创建了三个 promise，分别在 100ms、200ms 和 10ms 后解决。然后将这些 promise 传递给`Promise.all()`函数。一旦所有 promise 都解决了，附加到`Promise.all()`函数的 then 处理程序将被调用。此处理程序记录 promise 的结果。请注意，结果数组的顺序与 promise 数组的顺序匹配，而不是 promise 的完成顺序。

如果`Promise.all()`调用中的一个或多个 promise 被拒绝，`reject`处理程序将被调用，并且会使用第一个 promise 的拒绝值。所有其他 promise 将继续运行，但是这些 promise 的拒绝或解决不会调用`Promise.all()` promise 链的任何`then`或`catch`处理程序。如下面的片段所示：

```php
// Create promises
let promise1 = new Promise( ( resolve, reject ) => {
  setTimeout( () => { reject( 'Error 1' ); }, 100 );
} );
let promise2 = new Promise( ( resolve, reject ) => {
  setTimeout( () => { reject( 'Error 2' ); }, 200 );
} );
let promise3 = new Promise( ( resolve, reject ) => {
  setTimeout( () => { reject( 'Error 3' ); }, 10 );
} );
Promise.all( [ promise1, promise2, promise3 ] ).then( console.log ).catch( console.log );
// Expected output: 
// Error: Error 3
```

###### 片段 2.29：Promise.all()拒绝

在这个例子中，我们创建了三个 promise，记录了 promise 编号，然后都被不同的错误拒绝。我们将这些 promise 传递给`Promise.all`调用。`Promise3`的超时时间最短，因此是第一个被拒绝的 promise。当`Promise3`被拒绝时，promise 拒绝被传递给最近的错误处理程序（`.catch()`），它记录了 promise 的拒绝。之后不久，promise1 和 promise2 都完成运行，并且都被拒绝。对于这些 promise，拒绝处理程序不会再次被调用。

处理多个 promise 的最后一个函数是`Promise.race()`函数。`Promise.race()`函数设计用来处理第一个被完成或拒绝的 promise。

#### 注意

如果由于某种原因，您的程序存在有意的竞争条件或多个代码路径，只应该在成功的响应处理程序被调用一次时，`Promise.race()`是完美的解决方案。

像`Promise.all()`一样，`Promise.race()`传递一个承诺数组；然而，`Promise.race()`只调用第一个完成的承诺的承诺完成处理程序。然后它按照正常的承诺链继续。其他承诺的结果被丢弃，无论它们是拒绝还是解决。使用`Promise.race()`处理承诺拒绝的方式与`Promise.all()`相同。只处理第一个拒绝的承诺。其他承诺被忽略，无论完成状态如何。`Promise.race()`的示例如下所示：

```php
// Create promises
let promise1 = new Promise( ( resolve, reject ) => setTimeout( resolve( 10 ), 100 ) );
let promise2 = new Promise( ( resolve, reject ) => setTimeout( resolve( 20 ), 200 ) );
let promise3 = new Promise( ( resolve, reject ) => setTimeout( resolve( 30 ), 10 ) );
Promise.race( [ promise1, promise2, promise3 ] ).then( result => console.log( result ) );
//Expected output: 30
```

###### 片段 2.30：Promise.race()示例

在上面的示例中，我们创建了三个承诺。这些承诺在各种超时后都会解决。`Promise3`首先解决，因为它的超时时间最短。当`promise3`解决时，then 处理程序被调用，并记录了`promise3`的结果。当`promise1`和`promise2`解决时，它们的结果被忽略。

### 承诺和回调

承诺和回调永远不应该混合在一起。编写同时利用回调和承诺进行异步工作的代码可能会变得非常复杂，并导致极其难以调试的错误。为了防止混合回调逻辑和承诺逻辑，我们必须在我们的代码中添加 Shim 来处理回调作为承诺和承诺作为回调。有两种方法可以做到这一点：承诺可以包装在回调中，或者回调可以包装在承诺中。

#### 注意

Shim 是用于向代码库添加缺失功能的代码文件。Shim 通常用于确保 Web 应用程序的跨浏览器兼容性。

### 将承诺包装在回调中

要将承诺函数包装在回调中，我们只需创建一个包装器函数，该函数接受`promise`函数、参数和`callback`。在`wrapper`函数内部，我们调用`promise`函数并传入提供的参数。我们附加`then`和`catch`处理程序。当这些处理程序解决时，我们调用`callback`函数并传递承诺返回的结果或错误。这在下面的片段中显示：

```php
// Promise function to be wrapped
function promiseFn( args ){
  return new Promise( ( resolve, reject ) => {
    /* do work */ 
    /* resolve or reject */
  } );
}
// Wrapper function
function wrapper( promiseFn, args,  callback ){
  promiseFn( args ).then( value => callback( null, value )
         .catch( err => callback( err, null );
}
```

###### 片段 2.31：在回调中包装承诺

在上面的示例中，我们使用承诺的结果调用了回调。如果承诺以一个值解决，我们将该值传递到回调中，错误字段设置为 null。如果承诺被拒绝，我们将错误传递到回调中，结果字段为 null。

要将基于回调的函数包装在承诺中，我们只需创建一个包装器函数，该函数接受要包装的函数和函数参数。在包装器函数内部，我们在一个新的承诺中调用被包装的函数。当回调返回结果或错误时，如果有错误，我们拒绝承诺，如果没有错误，我们解决承诺。这在下面的片段中显示：

```php
// Callback function to be wrapped
function wrappedFn( args, cb ){
  /* do work */ 
  /* call cb with error or result */
}
// Wrapper function
function wrapper( wrappedFn, args ){
  return new Promise( ( resolve, reject ) => {
    wrappedFn( args, ( err, result ) => {
      if( err ) {
        return reject( err );
      }
      resolve( result );
    } );
  } );
}
```

###### 片段 2.32：在承诺中包装回调

在上面的示例中，我们创建了一个包装器函数，该函数接受一个函数和该函数的参数。我们返回一个调用此函数的承诺，并根据结果拒绝或解决承诺。由于此函数返回一个承诺，因此它可以嵌入在承诺链中，或者可以附加 then 或 catch 处理程序。

**结论**

承诺是处理 JavaScript 中异步编程的另一种方式。创建时，承诺处于挂起状态，并根据异步工作的结果进入完成或拒绝状态。为了处理承诺的结果，我们使用`.then()`、`.catch()`和`.finally()`成员函数。`.then()`函数接受两个处理程序函数，一个用于承诺完成，一个用于承诺拒绝。`.catch()`函数只接受一个函数并处理承诺拒绝。`Promise.finally()`接受一个函数，并在承诺完成或拒绝时调用。

当需要运行多个 promise 但顺序不重要时，我们可以使用`Promise.all()`和`Promise.race()`静态函数。当所有 promise 都完成运行时，将调用`Promise.all()`解析处理程序。当第一个 promise 完成运行时，将调用`Promise.race()`解析处理程序。

Promises 和回调不兼容，不应该在程序主体中混合使用。为了允许使用 promises 或回调函数的函数和模块之间的兼容性，我们可以编写一个包装函数。我们可以将回调包装在 promise 中，或将 promise 包装在回调中。这使我们能够使第三方模块与我们的代码兼容。

### 练习 19：使用 Promises

您正在构建一个基于 promise 的 API。在您的 API 中，您必须验证用户输入，以确保传递到数据库模型的数据是正确的类型。编写一个返回 promise 的函数。这个 promise 应该验证传递给 API 函数的数据值不是一个数字。如果用户将数字传递给函数，用错误拒绝 promise。如果用户将非数字传递给 API 函数，用单词`Success!`解析 promise。

构建一个使用实际场景的 promise 的函数，执行以下步骤：

1.  编写一个名为`promiseFunction`的函数，它接受一个数据参数并返回一个 promise。

1.  将一个接受两个参数 resolve 和 reject 的函数传递到 promise 的构造函数中。

1.  在 promise 中，通过创建一个在 10ms 后运行的超时来开始执行异步工作。

1.  在`timeout`回调函数中，记录提供给`promiseFunction`的输入数据。

1.  在`timeout`回调中，检查数据的类型是否为数字。如果是，用错误拒绝 promise，否则用字符串`Success!`解析 promise。

1.  运行`promiseFunction`并提供一个数字作为参数。将`then()`处理程序和`catch()`处理程序附加到函数返回的 promise 上。

#### 注意

`then`处理程序应记录 promise 解析值。`catch`处理程序应记录错误的消息属性。

**代码**

##### Index.js

```php
function promiseFunction( data ) {
 return new Promise( ( resolve, reject ) => {
   setTimeout( () => {
     console.log( data );
     if ( typeof data === 'number' ) {
       return reject( new Error( 'Data cannot be of type \'number\'.' ) );
     }
     resolve( 'Success!' );
   }, 10 );
 } );
}
promiseFunction( 1 ).then( console.log ).catch( err => console.log( 'Error: ${err.message}' ) );
promiseFunction( 'test' ).then( console.log ).catch( err => console.log( 'Error: ${err.message}' ) );
```

###### 片段 2.33：实现 promises

https://bit.ly/2SRZapq

**结果**

![图 2.10：作用域输出](img/Figure_2.10.jpg)

###### 图 2.10：作用域输出

## 异步/等待

**异步/等待**是一种新的语法形式，用于简化使用 promises 的代码。异步/等待引入了两个新关键字：`async`和`await`。`async`添加到函数声明中，`await`用于`async`函数内部。这是令人惊讶地易于理解和使用。在其最简单的形式中，异步/等待允许我们编写基于 promise 的异步代码，看起来几乎与执行相同任务的同步代码相同。我们将使用异步/等待来简化使用 promises 的代码，并使其更容易阅读和理解。

### 异步/等待语法

`async`关键字被添加到函数声明中；它必须在函数关键字之前。`async`函数声明定义了一个异步函数。以下是`async`函数声明的示例声明：

```php
async function asyncExample( /* arguments */  ){ /* do work */ }
```

###### 片段 2.34：实现 promises

`async`函数隐式返回一个 promise，无论指定的返回值是什么。如果返回值被指定为非 promise 类型，JavaScript 会自动创建一个 promise，并用返回的值解析该 promise。这意味着所有异步函数都可以对返回值应用`Promise.then()`和`Promise.catch()`处理程序。这允许与现有基于 promise 的代码非常轻松地集成。这在以下片段中显示：

```php
async function example1( ){ return 'Hello'; }
async function example2( ){ return Promise.resolve( 'World' ); }
example1().then( console.log ); // Expected output: Hello
example2().then( console.log ); // Expected output: World
```

###### 片段 2.35：异步函数输出

`await`关键字只能在`async`函数内部使用。Await 告诉 JavaScript 等待相关的承诺解决并返回其结果。这意味着 JavaScript 暂停执行该代码块，等待承诺被解决，同时做其他异步工作，然后在承诺解决后恢复该代码块。这使得等待的代码块像同步函数一样运行，但不会消耗任何资源，因为 JavaScript 引擎仍然可以做其他工作，比如运行脚本或处理事件，而异步代码正在等待。下面的片段中展示了`await`关键字的示例。

#### 注意

尽管 async/await 功能使 JavaScript 代码看起来和行为上都像是同步的，但 JavaScript 仍然通过事件循环异步运行代码。

```php
async function awaitExample( /* arguments */ ){ 
  let promise = new Promise( ( resolve, reject ) => {
    setTimeout( () => resolve( 'done!'), 100 );
  });
  const result = await promise;
  console.log( result ); // Expected output: done!
}
awaitExample( /* arguments */ );
```

###### 片段 2.36：等待关键字

在前面的示例中，我们定义了一个`async`函数`awaitExample()`。由于它是一个`async`函数，我们可以使用 await 关键字。在函数内部，我们创建一个进行异步工作的承诺。在这种情况下，它只是等待 100 毫秒，然后用字符串`done!`解决承诺。然后我们等待创建的承诺。当承诺以一个值解决时，await 获取该值并返回它，该值保存在变量 result 中。然后我们将 result 的值记录在控制台中。我们不是使用 then 处理程序来获取解决值，而是简单地等待该值。这段代码的 await 块看起来类似于同步代码块。

### 异步/等待承诺拒绝

既然我们知道如何处理异步/等待的承诺兑现，那么我们如何处理承诺的拒绝呢？使用异步/等待处理错误拒绝非常简单，并且与标准的 JavaScript 错误处理非常契合。如果一个承诺被拒绝，等待该承诺解决的 await 语句会抛出一个错误。当在`async`函数内部抛出错误时，JavaScript 引擎会自动捕获，并且由`async`函数返回的承诺会被拒绝并携带该错误。这听起来有点复杂，但实际上非常简单。这些关系在下面的片段中展示：

```php
async function errorExample1( /* arguments */ ){ 
  return Promise.reject( 'Rejected!' );
}
async function errorExample2( /* arguments */ ){ 
  throw 'Rejected!';
}
async function errorExample3( /* arguments */ ){ 
  await Promise.reject( 'Rejected!' );
}
errorExample1().catch( console.log ); // Expected output: Rejected!
errorExample2().catch( console.log ); // Expected output: Rejected!
errorExample3().catch( console.log ); // Expected output: Rejected!
```

###### 片段 2.37：异步/等待承诺拒绝

在前面的片段中，我们创建了三个异步函数。在第一个函数`errorExample1()`中，我们返回一个被拒绝的承诺，携带字符串`Rejected!`。在第二个函数`errorExample2()`中，我们抛出字符串`Rejected!`。由于这是在`async`函数内部抛出的错误，`async`函数会将其包装在一个承诺中并返回一个携带抛出值的被拒绝的承诺。在这种情况下，它返回一个携带字符串`Rejected!`的被拒绝的承诺。在第三个函数`errorExmaple3`中，我们等待一个被拒绝的承诺。等待被拒绝的承诺会导致 JavaScript 抛出承诺拒绝值，即`Rejected!`。然后`async`函数捕获抛出的错误值，将其包装在一个承诺中，拒绝该承诺，并返回被拒绝的承诺。所有三个示例函数都返回一个携带相同值的被拒绝的承诺。

由于如果等待的承诺被拒绝，await 会抛出一个错误，我们可以简单地使用 JavaScript 中的标准 try/catch 错误处理机制来处理异步错误。这非常有用，因为它允许我们以相同的方式处理所有错误，无论是异步还是同步的。这在下面的示例中展示：

```php
async function tryCatchExample() {
  // Try to do asynchronous work
  try{
    const value1 = await Promise.resolve( 'Success 1' );
    const value2 = await Promise.resolve( 'Success 2' );
    const value3 = await Promise.reject( 'Oh no!' );
  } 

  // Catch errors
  catch( err ){
    console.log( err ); // Expected output: Oh no!
  }
}
tryCatchExample()
```

###### 片段 2.38：错误处理

在前面的示例中，我们创建了一个尝试进行异步工作的 async 函数。该函数尝试连续等待三个承诺。最后一个被拒绝，导致抛出一个错误。这个错误被`catch`块捕获和处理。

由于错误被包裹在承诺中，并且被异步函数拒绝，当一个承诺被拒绝时，等待会抛出错误，异步/等待函数错误向最高级别的等待调用传播。这意味着除非需要在各种嵌套级别上以特殊方式处理错误，否则我们可以简单地在最外层错误处使用一个 try catch 块。错误将通过被拒绝的承诺在异步/等待函数堆栈上传播，并且只需要被顶层等待块捕获。这在以下片段中显示：

```php
async function nested1() { return await Promise.reject( 'Error!' ); }
async function nested2() { return await nested1; }
async function nested3() { return await nested2; }
async function nestedErrorExample() {
  try{ const value1 = await nested3; }
  catch( err ){ console.log( err ); } // Expected output: Oh no!
}
nestedErrorExample();
```

###### 片段 2.39：嵌套错误处理

在前面的例子中，我们创建了几个异步函数，它们等待另一个异步函数的结果。它们按顺序调用`nextedErrorExample() -> nested3() -> nested2() -> nested1()`。`nested1()`的主体等待一个被拒绝的承诺，这会引发错误。`Nested1()`捕获此错误并返回一个被拒绝的承诺。`nested2()`的主体等待`nested1()`返回的承诺。`nested1()`返回的承诺被原始错误拒绝，因此`nested2()`中的等待引发错误，并被`nested2()`包装在一个承诺中。这一直传播到`nestedErrorExample()`中的`await`。嵌套错误示例中的`await`引发错误，被捕获和处理。由于我们只需要在最高级别处理错误，因此我们将 try/catch 块放在最外层的等待调用处，并允许错误向上传播，直到遇到该 try/catch 块。

### 使用异步等待

现在我们知道如何使用异步/等待，我们需要将其集成到我们的承诺代码中。要将我们的承诺代码转换为使用异步/等待，我们只需要将承诺链分解为异步函数，并等待每个步骤。承诺处理程序链在每个处理程序函数（`then()`，`catch()`等）处分开。承诺返回的值用`await`语句捕获并保存到一个变量中。然后将此值传递给第一个承诺`then()`承诺处理程序的`回调`函数，并且函数的结果应该用`await`语句捕获并保存到一个新变量中。对于承诺链中的每个`then()`处理程序都是如此。

为了处理错误和承诺拒绝，我们用 try catch 块包围整个块。以下片段中显示了一个例子：

```php
// Promise chain - API functions return a promise
myPromse.then( apiCall1 ).then( apiCall2 ).then( apiCall3 ).catch( errorHandler );
async function asyncAwaitUse( myPromise ) {
  try{
    const value1 = await myPromise;
    const value2 = await apiCall1( value1 );
    const value3 = await apiCall2( value2 );
    const value4 = await apiCall3( value3 );
  } catch( err ){
    errorHandler( err );
  }
}
asyncAwaitUse( myPromise );
```

###### 片段 2.40：集成异步/等待

正如我们在承诺链中看到的，我们将三个 API 调用和一个错误处理程序链接到`myPromise`的解决方案上。在每个承诺链步骤中，都会返回一个承诺，并附加一个新的`Promise.then()`处理程序。如果承诺链的某个步骤被拒绝，将调用 catch 处理程序。

在异步/等待示例中，我们在每个`Promise.then()`处理程序处中断承诺链。然后，我们将`then`处理程序转换为返回承诺的函数。在这种情况下，`apiCall1()`，`apiCall2()`和`apiCall3()`已经返回承诺。然后我们等待每个 API 调用步骤。要处理承诺的拒绝，我们必须用 try catch 语句包围整个块。

就像承诺链中有多个链接的 then 处理程序一样，具有多个等待调用的`async`函数将依次运行每个等待调用，直到前一个等待调用从相关承诺中接收到一个值为止，才开始下一个等待调用。如果我们试图同时完成几个异步任务，这可能会减慢异步工作的速度。我们必须等待每个步骤完成，然后才能开始下一步。为了避免这种情况，我们可以使用`Promise.all`和`await`。

正如我们之前学到的，`Promise.all` 同时运行所有子承诺，并返回一个未完成的承诺，直到所有子承诺都以一个值解决。我们可以像附加 then 处理程序到 `Promise.all` 一样等待 `Promise.all`。通过等待 `Promise.all` 调用返回的值，只有当所有子承诺都完成时才能使用。这在下面的片段中显示：

```php
async function awaitPromiseAll(){
  let promise1 = new Promise( ( resolve, reject ) => setTimeout( () => resolve( 10 ), 100 ) );
  let promise2 = new Promise( ( resolve, reject ) => setTimeout( () => resolve( 20 ), 200 ) );
  let promise3 = new Promise( ( resolve, reject ) => setTimeout( () => resolve( 30 ), 10 ) );
  const result = await Promise.all( [ promise1, promise2, promise3 ] );
  console.log( result ); //Expected output: [ 10, 20, 30 ]
}
awaitPromiseAll();
```

###### 片段 2.41：并行等待承诺

从前面的示例中可以看出，我们创建了几个承诺，将这些承诺传递给 `Promise.all` 调用，然后等待 `Promise.all` 返回的承诺的解决。这遵循了 async/await 的规则，就像我们期望的那样。这个逻辑也可以应用到 `Promise.race`。

在下面的片段中显示了一个 promise 竞赛的示例：

```php
async function awaitPromiseAll(){
  let promise1 = new Promise( ( resolve, reject ) => setTimeout( () => resolve( 10 ), 100 ) );
  let promise2 = new Promise( ( resolve, reject ) => setTimeout( () => resolve( 20 ), 200 ) );
  const result = await Promise.race( [ promise1, promise2 ] );
  console.log( result ); //Expected output: 10]
}
awaitPromiseAll();
```

###### 片段 2.42：Promise 竞赛示例

**结论**

Async/await 是一个令人惊奇的新语法格式，它帮助我们简化基于承诺的代码。它允许我们编写看起来像同步代码的代码。Async/await 引入了两个关键字，**async** 和 **await**。Async 用于表示一个 `async` 函数。在声明函数时，它在函数关键字之前添加。Async 函数总是返回一个承诺。await 关键字只能在承诺上的 async 函数中使用。它告诉 JavaScript 引擎等待承诺解决，并在拒绝或实现时抛出错误或返回值。Async/await 错误处理通过抛出的错误和拒绝的承诺来完成。`async` 函数自动捕获抛出的错误并返回一个以该错误拒绝的承诺。等待的承诺在拒绝时抛出错误。这使得错误处理可以轻松地与标准的 JavaScript try/catch 错误处理相结合。Async/await 非常容易集成到基于承诺的代码中，并且可以使其非常易于阅读。

### 活动 2：使用 Async/Await

你被要求构建一个与数据库交互的服务器。你必须编写代码在数据库中创建和查找基本用户对象。导入 `simple_db.js` 文件。使用 `get` 和 `insert` 命令，使用 async/await 语法编写以下程序：

1.  查找 `john` 键，如果存在，记录结果对象的年龄字段。

1.  查找 `sam` 键，如果存在，记录结果对象的年龄。

1.  查找你的名字。如果不存在，插入你的名字。如果必须添加一个对象，查找新对象并记录年龄。

对于任何失败的 `db.get` 操作，将键保存到数组中。在程序结束时，打印失败的键。

DB API：

`db.get( index ):`

这需要一个索引并返回一个承诺。如果索引不存在，查找失败，或者未指定键，则承诺将以错误拒绝。

`db.insert( index, insertData ):`

这需要一个索引和数据，并返回一个承诺。如果操作完成，承诺将以插入的键实现。如果操作失败，或者没有指定键或插入数据，承诺将以错误拒绝。

利用承诺和 async/await 语法构建程序，执行以下步骤：

1.  编写一个名为 `main` 的 `async` 函数。所有操作都将在这里进行。

1.  创建一个数组来跟踪导致 db 错误的键。

1.  捕获所有错误并记录它们。

1.  在所有 try-catch 块之外，在 `main` 函数的末尾，返回数组。

1.  调用主函数并附加 `then()` 和 `catch()` 处理程序到返回的承诺。

**结果**

![图 2.11：作用域输出](img/Figure_2.11.jpg)

###### 图 2.11：作用域输出

您成功地使用了承诺和 async/await 语法来构建一个访问数据库的程序。

#### 注意

此活动的解决方案可以在第 282 页找到。

## 总结

JavaScript 是一种异步、事件驱动、单线程的语言。JavaScript 不会在长时间运行的操作中挂起到另一个资源，而是在任何待处理的工作时进行其他操作。JavaScript 通过事件循环实现这一点。事件循环由调用堆栈、堆、事件队列和主事件循环组成。这四个组件共同工作，安排 JavaScript 何时运行代码的不同部分。为了利用 JavaScript 的异步特性，我们使用回调或者 Promise。回调只是作为参数传递给其他函数的简单函数。Promise 是具有事件处理函数的特殊类。当异步操作完成时，JavaScript 引擎运行回调或调用与该操作的完成事件相关联的 Promise 处理程序。这就是 JavaScript 异步的最简单形式。

在下一章中，我们将学习**文档对象模型**（DOM）、**JavaScript 事件对象**和**jQuery 库**。
