# 第九章：异步编程

## 学习目标

在本章结束时，你将能够：

+   描述异步操作的工作原理

+   使用回调处理异步操作

+   演示回调和事件循环

+   实现承诺来处理异步操作

+   使用承诺重写带有回调的异步代码

+   重构您的传统代码，使用 async 和 await 函数

在本章中，我们将探讨 JavaScript 的异步（后面简称为 async）特性。重点将放在传统语言如何处理需要时间完成的操作以及 JavaScript 如何处理这些操作上。之后，我们将讨论在 JavaScript 中处理这些情况的各种方法。

## 介绍

在上一章中，我们学习了如何使用数组和对象以及它们的辅助函数。在本章中，我们将更多地了解 JavaScript 的运行方式以及如何处理耗时操作。

在处理 JavaScript 的大型项目时，通常我们必须处理网络请求、磁盘 IO 和数据处理。许多这些操作需要时间完成，对于刚开始使用 JavaScript 的初学者来说，很难理解如何检索这些耗时操作的结果。这是因为，与其他语言不同，JavaScript 有一种特殊的处理这些操作的方式。在编写程序时，我们习惯于线性思维；也就是说，程序逐行执行，只有在有循环或分支时才会打破这种流程。例如，如果你想在 Java 中进行简单的网络请求，你将不得不做类似于下面代码中所示的事情：

```js
import java.net.*;
import java.io.*;
public class SynchronousFetch{
  public static void main(String[] args){
   StringBuilder content = new StringBuilder();
   try {
    URL url = new URL("https://www.packtpub.com");
    URLConnection urlConnection = url.openConnection();
    BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(urlConnection.getInputStream()));
    String line;
    while ((line = bufferedReader.readLine()) != null){
      content.append(line + "\n");
    }
    bufferedReader.close();
   } catch(Exception e) {
    e.printStackTrace();
   }
   System.out.println(content.toString());
   System.exit(0);
  }//end main
}//end class SynchronousFetch 
```

理解起来很简单：你创建一个 HTTP 客户端，并在客户端内调用一个方法来请求该 URL 的内容。一旦请求被发出并收到响应，它将继续运行返回响应主体的下一行代码。在此期间，整个函数将暂停并等待`fetch`，只有在请求完成后才会继续。这是其他语言中处理这些操作的正常方式。处理耗时操作的这种方式称为**同步处理**，因为它强制程序暂停，只有在操作完成后才会恢复。

由于这种线性思维，许多开发人员（包括我）在开始使用 JavaScript 编码时会感到非常困惑。大多数人会开始编写这样的代码：

```js
const request = require('request');
let response;
request('SOMEURL', (err, res) => {
   response = res.body;
});
console.log(response);
```

从代码的外观来看，它应该像我们之前的代码一样运行。它将发出请求，一旦完成，将响应变量设置为响应主体，然后输出响应。大多数尝试过这种方法的开发人员都会知道，这不是 JavaScript 的工作方式；代码将运行，产生'undefined'输出，然后退出。

### JavaScript 如何处理耗时操作

在 JavaScript 中，这些操作通常使用异步编程来处理。在 JavaScript 中有多种方法可以做到这一点；最常用的方法，也是你在传统程序中最常见的方法，就是**回调**。回调只是一个传递包含应用程序其余逻辑的函数给另一个函数的花哨术语；它们实际上非常容易理解。考虑传统函数在逻辑完成后返回它们的值。在异步编程中，它们通常不返回值；相反，它们将它们的结果传递给调用者提供的回调函数。考虑以下代码：

```js
const request = require('request');
let response;
request('SOMEURL', (err, res) => {
   response = res.body;
});
console.log(response);
```

让我们看看为什么这不会产生我们想要的结果。我们使用的`request`库可以被视为执行一些耗时操作逻辑的函数。`request`函数希望你传递一个回调函数作为参数，该回调函数包括你接下来要做的一切。在回调函数中，我们接受两个参数，`err`和`res`；在函数内部，我们将之前声明的响应变量赋值给`res`体（响应体）。在`request`函数外部，我们有`console.log`来记录响应。因为回调函数将在将来的某个时刻被调用，所以我们会在给它设置任何值之前记录响应的值。大多数开发人员在处理 JavaScript 时会感到非常沮丧，因为上面的代码不是线性的。执行的顺序如下：

```js
1const request = require('request');
2 let response;
3 request('SOMEURL', (err, res) => {
   5 response = res.body;
});
4 console.log(response);
```

从上面的代码执行顺序可以看出，前三行的工作正如我们所期望的那样。我们导入了`request`库并声明了一个响应变量，然后调用了带有回调的`request`库。因为回调只有在网络请求完成时才会被调用，程序将继续执行其余的代码，输出响应。

最后，当网络请求完成时，它将调用我们的回调函数并运行将体分配给我们的响应的行。为了使这段代码表现如我们所期望的那样，我们需要修改代码如下：

```js
const request = require('request');
let response;
request('SOMEURL', (err, res) => {
   response = res.body;
   console.log(response);
});
```

在上面的代码中，我们将`console.log`放在回调函数内部，这样它只有在赋值完成后才会被执行。现在，当我们运行这段代码时，它将输出实际的响应体。

### 使用回调处理异步操作

在介绍中，我们谈到了 JavaScript 如何与其他语言不同地处理异步操作。在本章中，我们将探讨如何使用回调方法编写包含许多异步操作的复杂 JavaScript 应用程序。

### 练习 61：编写您的第一个回调

在这个练习中，我们将首先编写一个模拟需要一段时间才能完成的函数。之后，我们将编写另一个消耗我们异步函数的函数。

#### 注意

此练习的代码文件可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Exercise61`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Exercise61)找到。

执行以下步骤完成练习：

1.  创建一个`slowAPI`对象来创建一个模拟 API 库；它的目的是在合理的时间内返回结果。我们首先编写这个来介绍如何模拟异步函数而无需执行异步操作。

```js
const slowAPI = {}
```

1.  在我们刚刚定义的`slowAPI`对象中创建一个`getUsers`函数，它不返回任何内容，需要一个回调函数。在`getUsers`内部调用`setTimeout`函数，用于在需要时给我们的代码添加 1 秒的延迟：

```js
slowAPI.getUsers = (callback) => {
      setTimeout(() => {
        callback(null, {
           status: 'OK',
           data: {
              users: [
                {
                   name: 'Miku'
                }, 
                {
                   name: 'Len'
                }
              ]
           }
        });
      }, 1000);
}
```

1.  在`slowAPI`对象中创建一个`getCart`函数，并在函数内部创建一个`if-else`循环，匹配用户名并在不匹配时返回错误：

```js
slowAPI.getCart = (username, callback) => {
      setTimeout(() => {
        if (username === 'Miku') {
           callback(null, {
              status: 'OK',
              data: {
                cart: ['Leek', 'Cake']
              }
           })
        } else {
           callback(new Error('User not found'));
        }
      }, 500);
}
```

1.  创建一个`runRequests`函数，调用`getUsers`来获取用户列表。在回调函数内部，我们将打印出响应或错误：

```js
function runRequests() {
   slowAPI.getUsers((error, response) => {
      if (error) {
        console.error('Error occurred when running getUsers');
        throw new Error('Error occurred');
      }
      console.log(response);
   });
}
```

1.  调用`run Request`函数：

```js
runRequests();
```

输出应该如下：

![图 8.1：runRequest 的输出](img/C14587_08_01.jpg)

###### 图 8.1：runRequest 的输出

我们可以看到`runRequest`函数已经运行完毕，我们的响应被正确打印出来。

1.  修改`runRequest`函数以调用`getCart`：

```js
function runRequests() {
   slowAPI.getUsers((error, response) => {
      if (error) {
        console.error('Error occurred when running getUsers');
        throw new Error('Error occurred');
      }
      console.log(response);
   });
   slowAPI.getCart('Miku', (error, result) => {
        if (error) {
           console.error(error);
           throw new Error('Error occurred');
        }
        console.log(result);
   });
}
```

在这里，我们在`runRequest`函数内部放置了一个类似的对`slowAPI`的调用；其他都没有改变。当我们运行这个时，我们得到了一个非常有趣的输出，如下所示：

![图 8.2：修改 runRequest 函数后的输出](img/C14587_08_02.jpg)

###### 图 8.2：修改 runRequest 函数后的输出

这非常有趣，因为它首先输出了`getCart`的结果，然后是`getUsers`的结果。程序之所以表现如此，是因为 JavaScript 的异步和非阻塞特性。在我们的操作中，因为`getCart`函数只需要 500 毫秒就能完成，所以它将是第一个输出。

1.  修改前面的函数以输出第一个用户的购物车：

```js
function runRequests() {
   slowAPI.getUsers((error, response) => {
      if (error) {
        console.error('Error occurred when running getUsers');
        throw new Error('Error occurred');
      }
      slowAPI.getCart(response.data.users[0].name,(error,result) => {
        if (error) {
           console.error(error);
           throw new Error('Error occurred');
        }
        console.log(result);
     });
   });
}
```

输出应该如下所示：

![图 8.3：第一个用户的购物车输出](img/C14587_08_03.jpg)

###### 图 8.3：第一个用户的购物车输出

因为我们将使用第一个请求的数据，所以我们必须在第一个请求的回调函数中编写我们下一个请求的逻辑。

1.  在访问未知用户的购物车时触发错误：

```js
function runRequests() {
   slowAPI.getUsers((error, response) => {
    if (error) {
        console.error('Error occurred when running getUsers');
        throw new Error('Error occurred');
      }
      slowAPI.getCart(response.data.users[1].name,(error,result) => {
        if (error) {
           console.error(error);
           throw new Error('Error occurred');
        }
        console.log(result);
      });
   });
}
```

我们知道从`getCart`返回的数据是，最后一个用户不匹配任何`if`语句。因此，在调用时会抛出错误。当我们运行代码时，将会看到以下错误：

![图 8.4：打印错误](img/C14587_08_04.jpg)

###### 图 8.4：打印错误

我们在白色中看到的第一个错误输出是通过`console.error`输出的错误。这可以根据您的喜好定制为特定格式的错误消息或输出，使用日志框架。第二个错误是由于我们在`console.log`后立即抛出新错误导致进程崩溃。

在这个练习中，我们检查了如何使用`setTimeout`模拟异步函数。`setTimeout`是一个非常有用的函数。虽然在实际代码中并不推荐使用，但在测试中需要模拟需要时间的网络请求或在调试软件时产生竞争条件时，它非常有用。之后，我们讨论了使用回调函数使用异步函数的方法以及异步函数中的错误处理方式。

接下来，我们将简要讨论为什么回调函数正在逐渐过时，以及如果不正确使用回调函数会发生什么。

### 事件循环

您可能以前听说过这个术语，指的是 JavaScript 如何处理耗时操作。了解事件循环在底层是如何工作也非常重要。

当考虑 JavaScript 最常用于什么时，它用于制作动态网站，主要在浏览器中使用。让很多人惊讶的是，JavaScript 代码在单个线程中运行，这简化了开发人员的很多工作，但在处理同时发生的多个操作时会带来挑战。在 JavaScript 运行时，后台运行一个无限循环，用于管理代码的消息和处理事件。事件循环负责消耗回调队列中的回调、运行堆栈中的函数和调用 Web API。JavaScript 中大多数操作可分为两种类型：阻塞和非阻塞。阻塞意味着阻塞事件循环（您可以将其视为其他语言的正常 UI 线程）。当事件循环被阻塞时，它无法处理来自应用程序其他部分的更多事件，应用程序将冻结直到解除阻塞。以下是示例操作及其分类的列表：

![图 8.5：带有示例操作及其分类的表](img/C14587_08_05.jpg)

###### 图 8.5：带有示例操作及其分类的表

从前面的列表中可以看到，几乎所有 JavaScript 中的 I/O 都是非阻塞的，这意味着即使完成时间比预期时间长，也不会阻塞事件循环。像任何语言一样，阻塞事件循环是一件糟糕的事情，因为它会使应用程序不稳定和无响应。这带来了一个问题：我们如何知道非阻塞操作是否已完成。

### JavaScript 如何执行代码

当 JavaScript 执行阻塞代码时，它会阻塞循环并在程序继续执行之前完成操作。如果你运行一个迭代 100 万次的循环，你的其余代码必须等待该循环完成才能继续。因此，在你的代码中不建议有大量阻塞操作，因为它们会影响性能、稳定性和用户体验。当 JavaScript 执行非阻塞代码时，它通过将进程交给 Web API 来进行获取、超时和休息。一旦操作完成，回调将被推送到回调队列中，以便稍后被事件循环消耗。

在现代浏览器中，这是如何实现的，我们有一个堆来存储大部分对象分配，和一个用于函数调用的堆栈。在每个事件循环周期中，事件循环首先优先处理堆栈，并通过调用适当的 Web API 来执行这些事件。一旦操作完成，该操作的回调将被推送到回调队列中，稍后会被事件循环消耗：

![图 8.6：事件循环周期](img/C14587_08_06.jpg)

###### 图 8.6：事件循环周期

为了了解一切是如何在幕后运作的，让我们考虑以下代码：

```js
setTimeout(() => {console.log('hi')}, 2000)
while(true) {
   ;
}
```

从外观上看，这段代码做了两件事：创建一个在 2 秒后打印`hi`的超时，以及一个什么都不做的无限循环。当你运行上述代码时，它会表现得有点奇怪 - 什么都不会被打印出来，程序就会挂起。它表现得像这样的原因是事件循环更偏向于堆栈中的项目，而不是回调队列中的项目。因为我们有一个无限的`while`循环不断推入调用堆栈，事件循环忙于运行循环并忽略了回调队列中已完成的`setTimeout`回调。关于`setTimeout`工作方式的另一个有趣事实是，我们可以使用它来延迟我们的函数到事件循环的下一个周期。考虑以下代码：

```js
setTimeout(() => {console.log('hi again')}, 0)
console.log('hi');
```

在这里，我们有`setTimeout`后面跟着`console.log`，但这里我们使用`0`作为超时，意味着我们希望立即完成。一旦超时完成并且回调被推送到回调队列，由于我们的事件循环优先处理调用堆栈，你可以期待这样的输出：

![图 8.7：超时完成后的输出](img/C14587_08_07.jpg)

###### 图 8.7：超时完成后的输出

我们看到`hi`在`hi again`之前被打印出来，因为即使我们将超时设置为零，它仍然会最后执行，因为事件循环会在调用堆栈中的项目之前执行回调队列中的项目。

### 活动 11：使用回调接收结果

在这个活动中，我们将使用回调来接收结果。假设你正在为一家当地燃气公司担任软件工程师，并且他们希望你为他们编写一个新功能：

+   你有一个客户端 API 库，可以用来请求本地用户列表。

+   你需要实现一个功能，计算这些用户的账单，并以以下格式返回结果：

```js
{
   id: 'XXXXX',
   address: '2323 sxsssssss',
   due: 236.6
}
```

+   你需要实现一个`calculateBill`函数，它接受`id`并计算该用户的燃气费用。

为了实现这一点，你需要请求用户列表并获取这些用户的费率和使用情况。最后，计算最终应付金额并返回合并结果。

#### 注意

这个活动的代码文件可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Activity11`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Activity11)找到。

执行以下步骤完成这个活动：

1.  创建一个`calculate`函数，它接受`id`和回调函数作为参数。

1.  调用`getUsers`来获取所有用户，这将给我们需要的地址。

1.  调用`getUsage`来获取我们用户的使用情况。

1.  最后，调用`getRate`来获取我们正在为其计算的用户的费率。

1.  使用现有 ID 调用`calculate`函数。

1.  使用不存在的 ID 调用`calculate`函数以检查返回的错误。

您应该看到返回的错误如下：

![图 8.8：使用不存在的 ID 调用函数](img/C14587_08_08.jpg)

###### 图 8.8：使用不存在的 ID 调用函数

#### 注意

此活动的解决方案可在第 613 页找到。

在这个活动中，我们实现的功能与实际世界中可能看到的非常相似。我们在一个函数中处理了多个异步操作。接下来，我们将讨论回调地狱以及在处理多个异步操作时可能出现的问题。

## 回调地狱

回调地狱指的是 JavaScript 开发人员在处理大型项目时遇到的障碍。回调地狱的原因并不完全是开发人员的错，部分原因是 JavaScript 处理异步操作的方式。通过使用回调来处理多个异步操作，很容易让事情失控。以下代码举例说明了回调地狱的例子：

```js
request('url', (error, response) => {
   // Do something here
   request('another url', (error, response) => {
      disk.write('filename', (result) => {
        if (result.this) {
           process(something, (result) => {
              request('another url', (error, response) => {
                if (response.this) {
                   request('this', (error, response) => {
                      // Do something for this
                   })
                } else {
                   request('that', (error, response) => {
                      if (error) {
                        request('error fallback', (error, response) => {
                           // Error fallback
                        })
                      }
                      if (response.this) {
                      }
                   })
                }
              });
           })
        } else {
           process(otherthing, (result) => {
              // Do something else
           })
        }
      })
   })
})
```

前面的代码示例是回调地狱的典型例子。虽然这段代码比实际世界中找到的回调地狱代码要短，但同样糟糕。回调地狱是指一段代码中嵌套了太多回调，使得开发人员难以理解、维护甚至调试代码。如果前面的代码被用来实现实际的业务逻辑，它将会扩展到超过 200 行。有这么多行和这么多层嵌套，会产生以下问题：

+   很难弄清楚你当前在哪个回调中。

+   它可能会导致变量名冲突和覆盖。

+   几乎不可能调试和断点代码。

+   代码将非常难以重用。

+   代码将无法进行测试。

这些问题只是由回调地狱引起的问题清单中的一部分。这些问题是为什么许多公司甚至在面试问题中包括关于回调地狱的问题的原因。有许多提出的方法可以使代码比前面的代码更可读。一种方法是将几乎每个回调都作为单独的函数提取出来。使用这种技术，前面的代码可以修改如下：

```js
function doAnotherUrl(error, response) {
   if (response.this) {
      request('this', (error, response) => {
        // Do something for this
      })
   } else {
      request('that', (error, response) => {
        if (error) {
           request('error fallback', (error, response) => {
              // Error fallback
           })
        }
        if (response.this) {
        }
      })
   }
}
function process(result) {
   request('another url', doAnotherUrl);
}
function afterWrite(result) {
   if (result.this) {
      process(something, afterProcess)
   } else {
      process(otherthing, afterProcess)
   }
}
function doAnotherThing(error, response) {
   disk.write('filename', afterWrite)
}
function doFirstThing(error, response) {
   // Do something here
   request('another url', doAnotherThing)
}
request('url', doFirstThing)
```

当代码像这样重写时，我们可以看到所有的处理函数都被分开了。稍后，我们可以将它们放在一个单独的文件中，并使用`require()`来引用它们。这解决了将所有代码放在一个地方和可测试性问题。但它也使代码库变得不必要地庞大和分散。在 ES6 中，引入了承诺。它开辟了一种全新的处理异步操作的方式。在下一节中，我们将讨论承诺的工作原理以及如何使用它们来摆脱回调地狱。

### 承诺

在 JavaScript 中，承诺是代表将来某个值的对象。通常，它是异步操作的包装器。承诺也可以在函数中传递并用作承诺的返回值。因为承诺代表一个异步操作，它可以有以下状态之一：

+   待定，意味着承诺正在等待，这意味着可能仍有异步操作正在运行，没有办法确定其结果。

+   实现，意味着异步操作已经完成，没有错误，值已准备好接收。

+   拒绝，意味着异步操作以错误完成。

承诺只能有前面三种状态之一。当承诺被实现时，它将调用提供给`.then`承诺函数的处理程序，当它被拒绝时，它将调用提供给`.catch`承诺函数的处理程序。

要创建一个 promise，我们在`Promise`构造函数中使用`new`关键字。构造函数接受一个包含异步操作代码的函数。它还将两个函数作为参数传递，`resolve`和`reject`。当异步操作完成并且值准备好被传递时，将调用`resolve`。当异步操作失败并且你想要返回失败原因时，通常是一个错误对象，将调用`reject`：

```js
 const myPromise = new Promise((resolve, reject) => {

});
```

以下代码使用 Promise.resolve 返回一个 promise：

```js
const myPromiseValue = Promise.resolve(12);
```

`Promise.resolve`返回一个解析为你传递的值的 promise。当你想要保持代码库一致，或者不确定一个值是否是 promise 时，它非常有用。一旦你使用`Promise.resolve`包装值，你可以使用`then`处理程序开始处理 promise 的值。

在下一个练习中，我们将看看如何使用 promise 处理异步操作，以及如何在不导致回调地狱的情况下将多个异步操作与 promise 结合起来。

### 练习 62：使用 Promise 作为回调的替代方案

在上一个活动中，我们讨论了如何将多个异步操作组合成一个单一的结果。这很容易理解，但也会使代码变得很长并且难以管理。我们讨论了回调地狱以及如何避免它。我们可以做的一件事是利用 ES6 中引入的`Promise`对象。在这个练习中，我们将讨论如何在我们的应用程序中使用 promise。

#### 注意

此练习的代码文件可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Exercise62`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Exercise62)找到。

执行以下步骤完成练习：

1.  创建一个 promise：

```js
const myPromise = new Promise(() => {

});
```

创建 promise 时，我们需要在`Promise`构造函数中使用`new`关键字。`Promise`构造函数要求你提供一个解析器函数来执行异步操作。当创建 promise 时，它将自动调用解析器函数。

1.  向解析器函数添加一个操作：

```js
const myPromise = new Promise(() => {
   console.log('hi');
});
```

输出应该如下所示：

![图 8.9：向解析器函数添加一个操作](img/C14587_08_09.jpg)

###### 图 8.9：向解析器函数添加一个操作

即使`console.log`不是一个异步操作，当我们创建一个 promise 时，它将自动执行我们的解析器函数并打印出`hi`。

1.  使用`resolve`解决 promise：

```js
const myPromise = new Promise((resolve) => {
   resolve(12);
});
myPromise
```

当调用函数时，会将一个`resolve`函数传递给我们的解析器函数。当它被调用时，promise 将被解决：

![图 8.10：调用函数后解决的 promise](img/C14587_08_10.jpg)

###### 图 8.10：调用函数后解决的 promise

1.  使用`then()`函数检索值。通过附加一个`then`处理程序，你期望从回调中读取解析的 promise 值：

```js
const myPromise = new Promise((resolve) => {
   resolve(12);
}).then((value) => {
   console.log(value);
});
```

输出应该如下所示：

![图 8.11：使用 then 函数检索值](img/C14587_08_11.jpg)

###### 图 8.11：使用 then 函数检索值

每当你创建一个 promise 时，你期望异步函数完成并返回一个值。

1.  创建一个立即解决的 promise：

```js
const myPromiseValue = Promise.resolve(12);
```

1.  创建一个立即被拒绝的 promise：

```js
const myRejectedPromise = Promise.reject(new Error('rejected'));
```

输出应该如下所示：

![图 8.12：立即被拒绝的 promise 创建](img/C14587_08_12.jpg)

###### 图 8.12：立即被拒绝的 promise 创建

就像`Promise.resolve`一样，使用`Promise.reject`创建 promise 将返回一个被提供的原因拒绝的 promise。

1.  使用`catch`在 promise 中处理`error`：

```js
myRejectedPromise.catch((error) => {
   console.log(error);
});
```

你可以使用`catch`提供一个错误处理程序。这会向 promise 添加一个拒绝回调。当你提供一个 catch 处理程序时，从 promise 返回的错误将作为处理程序的参数传递：

图 8.13：使用 catch 处理 promise 中的错误

](Images/C14587_08_13.jpg)

###### 图 8.13：使用 catch 处理 promise 中的错误

1.  创建一个返回 promise 的`wait`函数：

```js
function wait(seconds) {
   return new Promise((resolve) => {
      setTimeout(() => {
        resolve(seconds);
      }, seconds * 1000);
   })
}
```

1.  使用`async`函数延迟我们的控制台日志：

```js
wait(2).then((seconds) => {
   console.log('i waited ' + seconds + ' seconds');
});
```

输出应该如下所示：

![图 8.14：使用异步函数延迟控制台日志](img/C14587_08_14.jpg)

###### 图 8.14：使用异步函数延迟控制台日志

如你所见，使用它非常简单。我们的`wait`函数每次调用时都返回一个新的 promise。在操作完成后运行我们的代码，将其传递给`then`处理程序。

1.  使用`then`函数链式调用 promise：

```js
wait(2)
   .then(() => wait(2))
   .then(() => {
      console.log('i waited 4 seconds');
   });
```

输出应该如下所示：

![图 8.15：使用 then 函数链接的 Promise](img/C14587_08_15.jpg)

###### 图 8.15：使用 then 函数链式调用的 Promise

例如，当我们想要将两个 promise 链在一起时，我们只需要将它们传递到`then`处理程序中，并确保结果也是一个 promise。在这里，我们看到在调用`wait`等待 2 秒后，我们调用另一个`wait`等待 2 秒，并确保计时器在第一个完成后开始。

在这个练习中，我们讨论了几种创建 promise 的方法，以及如何创建一个使用 promise 而不是回调处理操作的异步函数。最后，我们使用`then`函数链式调用了 promise。这些都是使用 promise 的非常简单的方法。在下一章中，我们将讨论如何有效地链式调用它们以及如何处理 promise 的错误。

### 链式调用 Promise

在上一个练习中，我们看了一种非常简单的方法来链式调用 promise。Promise 链式调用也可能很复杂，正确地使用它可以避免代码中的许多潜在问题。当你设计一个需要同时执行多个异步操作的复杂应用程序时，使用回调时很容易陷入回调地狱。使用 promise 解决了与回调地狱相关的一些问题，但它并不是万能的。通常，你会看到像这样编写的代码：

```js
getUser('name').then((user) => {
   increaseLike(user.id).then((result) => {
      readUser(user.id).then((user) => {
        if (user.like !== result.like) {
           generateErrorLog(user, 'LIKE').then((result) => {
              response.send(403);
           })
        } else {
           updateAvatar(user).then((result) => {
              optimizeImage(result.image).then(() => {
                response.send(200);
              })
           })
        }
      });
   });
}).catch((error) => {
   response.send(403);
});
```

当你看到像这样编写的代码时，很难判断是否转换为 promise 解决了任何问题。前面的代码与我们的回调地狱代码有相同的问题；所有逻辑都是分散和嵌套的。我们还有其他问题，比如上层作用域的值可能会被意外覆盖。

当我们编写带有 promise 的代码时，我们应该考虑尽可能使代码模块化，并将操作集合视为管道。对于我们前面的示例，管道将如下所示：

![图 8.16：示例管道（一系列操作）](img/C14587_08_16.jpg)

###### 图 8.16：示例管道（一系列操作）

你会发现我们希望将值从一个过程传递到下一个过程。这有助于我们链式调用 promise，并且可以使我们的代码非常清晰和易于维护。我们可以将前面的代码重写为以下内容：

```js
function increaseLike(user) {
   return new Promise((resolve) => {
      resolve({
        // Some result
      })
   });
};
function readUser(result) {
   return new Promise((resolve) => {
      resolve({
        // Return user
      })
   });
}
function updateAvatar(user) {
   return new Promise((resolve) => {
      resolve({
        // Return updated avatar
      })
   });
}
function optimizeImage(user) {
   return new Promise((resolve) => {
      resolve({
        // Return optimized images
      })
   });
}
function generateErrorLog(error) {
   // Handle some error
}
readUser('name')
   .then(increaseLike)
   .then(readUser)
   .then(updateAvatar)
   .then(optimizeImage)
   .catch(generateErrorLog)
```

正如你所看到的，重写的代码更易读，任何查看这段代码的人都会准确知道将会发生什么。当我们以这种方式链式调用 promise 时，我们基本上是将值从一个过程传递到另一个过程。通过使用这种方法，我们不仅解决了回调地狱的问题，而且使代码更具可测试性，因为这些辅助函数中的每一个都是完全独立的，它们不需要任何比传递给它们的参数更多的东西。更不用说，如果你的应用程序中有任何部分想要执行类似的操作（例如，`optimizeImage`），你可以轻松地重用代码的这部分。在下一个练习中，我们将讨论如何使用 promise 链式调用编写具有多个异步操作的复杂功能。

### 练习 63：高级 JavaScript Promise

在这个练习中，我们将编写一个简单的程序，运行多个异步操作，并使用 promise 链式调用它们的结果。之后，我们还将使用`Promise`类的有用静态方法来帮助我们同时管理多个 promise。

#### 注意

此活动的代码文件可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Exercise63`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Exercise63)找到。

执行以下步骤完成练习：

1.  创建`getProfile`和`getCart`函数，它们返回一个 promise。`getProfile`应该以`id`字符串作为输入，并根据输入解析不同的结果：

```js
function getProfile(id) {
   return new Promise((resolve, reject) => {
      switch(id) {
        case 'P6HB0O':
           resolve({ id: 'P6HB0O', name: 'Miku', age: 16, dob: '0831' });
        break;
        case '2ADN23':
           resolve({ id: '2ADN23', name: 'Rin', age: 14, dob: '1227' });
        break;
        case '6FFQTU':
           resolve({ id:'6FFQTU', name: 'Luka', age: 20, dob: '0130' });
        break;
        default:
           reject(new Error('user not found'));
      }
   });
}
function getCart(user) {
   return new Promise((resolve, reject) => {
      switch(user.id) {
        case 'P6HB0O':
           resolve(['leek', 'cake', 'notebook']);
        break;
        case '2ADN23':
           resolve(['ice cream', 'banana']);
        break;
        case '6FFQTU':
           resolve(['tuna', 'tako']);
        break;
        default:
           reject(new Error('user not found'));
      }
   });
}
```

1.  创建另一个异步函数`getSubscription`，它接受一个 ID 并为该 ID 解析`true`和`false`值：

```js
function getSubscription(id) {
   return new Promise((resolve, reject) => {
      switch(id) {
        case 'P6HB0O':
           resolve(true);
        break;
        case '2ADN23':
           resolve(false);
        break;
        case '6FFQTU':
           resolve(false);
        break;
        default:
           reject(new Error('user not found'));
      }
   });
}
```

在这里，函数只接受一个字符串 ID 作为输入。如果我们想在我们的 promise 链中链接它，我们需要确保提供给该函数的 promise 解析为单个字符串值。

1.  创建`getFullRecord`，它返回`id`的组合记录：

```js
function getFullRecord(id) {
   return {
      id: '',
      age: 0,
      dob: '',
      name: '',
      cart: [],
      subscription: true
   };
}
```

在`getFullRecord`函数中，我们希望调用所有前面的函数并将记录组合成前面代码中显示的返回值。

1.  调用我们之前在`getFullRecord`中声明的函数，并返回`getProfile`，`getCart`和`getSubscription`的组合结果：

```js
function getFullRecord(id) {
   return getProfile(id).then((user) => {
      return getCart(user).then((cart) => {
        return getSubscription(user.id).then((subscription) => {
           return {
              ...user,
              cart: cart,
              subscription
           };
        });
      });
   });
}
```

这个函数也返回一个 promise。我们可以调用该函数并打印出它的值：

```js
getFullRecord('P6HB0O').then(console.log);
```

这将返回以下输出：

![图 8.17：在 getFullRecord 中调用已声明的函数](img/C14587_08_17.jpg)

###### 图 8.17：在`getFullRecord`中调用已声明的函数

但是我们的代码非常混乱，并且并没有真正利用我们之前提到的 promise 链式调用。为了解决这个问题，我们需要对`getCart`和`getSubscription`进行修改。

1.  更新`getCart`函数，该函数返回一个新对象，包括`user`对象的每个属性和`cart`项，而不仅仅返回`cart`项：

```js
function getCart(user) {
   return new Promise((resolve, reject) => {
      switch(user.id) {
        case 'P6HB0O':
           resolve({ ...user, cart: ['leek', 'cake', 'notebook'] });
        break;
        case '2ADN23':
           resolve({ ...user, cart: ['ice cream', 'banana'] });
        break;
        case '6FFQTU':
           resolve({ ...user, cart: ['tuna', 'tako'] });
        break;
        default:
           reject(new Error('user not found'));
      }
   });
}
```

1.  更新`getSubscription`函数，该函数以`user`对象作为输入并返回一个对象，而不是单个值：

```js
function getSubscription(user) {
   return new Promise((resolve, reject) => {
      switch (user.id) {
        case 'P6HB0O':
           resolve({ ...user, subscription: true });
           break;
        case '2ADN23':
           resolve({ ...user, subscription: false });
           break;
        case '6FFQTU':
           resolve({ ...user, subscription: false });
           break;
        default:
           reject(new Error('user not found'));
      }
   });
}
```

1.  更新`getFullRecord`函数：

```js
function getFullRecord(id) {
   return getProfile(id)
      .then(getCart)
      .then(getSubscription);
}
```

现在，这比以前的所有嵌套要可读得多。我们只是通过对之前的两个函数进行最小的更改，大大减少了`getFullRecord`。当我们再次调用此函数时，它应该产生完全相同的结果：

![图 8.18：更新的 getFullRecord 函数](img/C14587_08_18.jpg)

###### 图 8.18：更新的 getFullRecord 函数

1.  创建`getFullRecords`函数，我们将使用它来调用多个记录并将它们组合成一个数组：

```js
function getFullRecords() {
   // Return an array of all the combined user record in our system
   return [
      {
        // Record 1
      },
      {
        // Record 2
      }
   ]
}
```

1.  使用`array.map`生成 promise 列表：

```js
function getFullRecords() {
   const ids = ['P6HB0O', '2ADN23', '6FFQTU'];
   const promises = ids.map(getFullRecord);
}
```

在这里，我们利用了`array.map`函数来迭代数组并返回一个新数组。因为数组只包含 ID，所以我们可以简单地传递`getFullRecord`函数。

1.  使用`Promise.all`来合并一系列 promise 的结果：

```js
function getFullRecords() {
   const ids = ['P6HB0O', '2ADN23', '6FFQTU'];
   const promises = ids.map(getFullRecord);
   return Promise.all(promises);
}
```

`Promise.all`只是接受一个 promise 数组并返回一个等待所有 promise 解析的 promise。一旦数组中的所有 promise 都解析了，它将解析为这些 promise 的结果数组。因为我们的目标是返回完整记录列表，这正是我们想要的。

1.  测试`getFullRecords`：

```js
getFullRecords().then(console.log);
```

输出应该如下所示：

![图 8.19：测试 getFullRecords 函数](img/C14587_08_19.jpg)

###### 图 8.19：测试 getFullRecords 函数

在这个练习中，我们使用了多个异步函数和它们的 promise 返回来实现复杂的逻辑。我们还尝试链式调用它们，并修改了一些函数以便于链式调用。最后，我们使用了`array.map`和`Promise.all`来使用数组创建多个 promise 并等待它们全部解析。这有助于我们管理多个 promise 并跟踪它们的结果。接下来，我们将讨论 promise 中的错误处理。

### Promise 中的错误处理

当我们向 web 服务器发出请求或访问磁盘上的文件时，不能保证我们要执行的操作会 100%成功。当它不按我们想要的方式工作时，我们需要确保我们的应用程序能够处理这些错误，以便它不会意外退出或损坏我们的数据。在以前编写异步函数的处理程序时，我们可以简单地从错误参数中获取返回的错误。当我们使用 promises 时，我们也可以从`catch`处理程序中获取错误。

但当我们处理错误时，我们不仅仅是在尝试防止发生对我们或用户有害的事情；我们还需要确保我们的错误足够有意义，以便我们使用这些信息并防止该错误再次发生。通常，如果我们想要处理 promises 中的错误，我们可以简单地这样做：

```js
aFunctionReturnsPromise()
   .then(dosomething)
   .catch((error) => {
   // Handle some error here
});
```

当我们想要处理某种类型的错误时，我们可以调用`catch`函数并传递一个错误处理程序。但如果我们同时处理多个 promises 呢？如果我们使用 promise 链呢？当处理多个 promises 时，你可能会认为我们需要做类似这样的事情：

```js
aFunctionReturnsPromise().then((result) => {
   anotherFunctionReturnsPromise().then((anotherResult) => {
   }).catch((error) => {
      // Handle error here
   });
}).catch((error) => {
   // handle error
})
```

在这里，我们处理了`aFunctionReturnsPromise`函数返回的 promise 的任何类型的错误。在该 promise 的`then`处理程序中，我们调用`anotherFunctionReturnsPromise`，在其`then`处理程序中，我们处理了该 promise 的错误。这看起来并不太糟糕，因为我们只使用了两个嵌套的 promises，所以严格来说不需要链式调用，而且我们分别处理了每个错误。但通常，当你看到人们写这样的代码时，你也会看到类似这样的东西：

```js
aFunctionReturnsPromise().then((result) => {
   return anotherFunctionReturnsPromise().then((anotherResult) => {
      // Do operation here
   }).catch((error) => {
      // Handle error here
      logError(error);
      throw new Error ('something else');
   });
}).catch((error) => {
   // handle error
   logError(error);
   throw new Error ('something else');
});
```

我甚至看到过像这样写的生产级代码。虽然这看起来对很多开发者来说是个好主意，但这并不是处理 promises 中错误的理想方式。有一些使用情况适合这种错误处理方式。其中一种情况是，如果你确定了你将要得到的错误类型，并且想要为每种不同类型做自定义处理。当你的代码像这样时，很容易在日志文件中出现重复，因为你可以从前面的代码中看到，错误被记录了两次：一次在嵌套 promise 的 catch 处理程序中，一次在父 promise 中。为了减少错误处理的重复，你可以简单地移除嵌套 promise 中的任何处理程序，这样前面的代码看起来会像这样：

```js
aFunctionReturnsPromise().then((result) => {
   return anotherFunctionReturnsPromise().then((anotherResult) => {
      // Do operation here
   });
}).catch((error) => {
   // handle error
   logError(error);
   throw new Error ('something else');
});
```

你不必担心嵌套 promise 中的错误没有被处理 - 因为我们在`then`处理程序中返回了 promise，并且传递了状态而不是值。所以，当嵌套 promise 遇到错误时，最终会被父错误处理程序中的`catch`处理程序捕获。

我们必须记住的一件事是，当我们使用 promises 时，当出现错误时，`then`处理程序不会被调用。考虑以下例子：

```js
processSomeFile().then(() => {
   // Cleanup temp files
   console.log('cleaning up');
}).catch((error) => {
   console.log('oh no');
});
```

假设你正在创建一个文件处理函数，并且在处理完成后，在`then`处理程序中运行清理逻辑。当出现错误时，这会创建一个问题，因为当该 promise 被拒绝时，清理过程将永远不会被调用。这可能会引起很多问题。我们可能会因为临时文件没有被删除而耗尽磁盘空间。如果我们没有正确关闭连接，我们也可能会面临内存泄漏的风险。为了解决这个问题，一些开发者采取了简单的方法并复制了清理逻辑：

```js
processSomeFile().then(() => {
   // Cleanup temp files
   console.log('cleaning up');
}).catch((error) => {
   // Cleanup temp files
   console.log('cleaning up');
   console.log('oh no');
})
```

虽然这解决了我们的问题，但也创建了一个重复的代码块，所以最终，当我们想要更改清理过程中的某些逻辑时，我们需要记住在两个地方都进行更改。幸运的是，`Promise`类给了我们一个非常有用的处理程序，我们可以设置它以确保无论状态如何，处理程序都会被调用：

```js
   processSomeFile().then(() => {
}).catch((error) => {

   console.log('oh no');
}).finally(() => {
   // Cleanup temp files
   console.log('cleaning up');
})
```

在这里，我们正在附加一种新类型的处理程序到我们的 promise。`.finally`处理程序将在 promise 被`settled`时始终被调用，无论它是解决还是被拒绝。这是一个非常有用的处理程序，我们可以在我们的 promises 上设置它，以确保我们正确清理连接或删除文件。

在上一个练习中，我们设法使用`Promise.all`从一系列 promises 中获取结果列表。在我们的示例中，所有 promises 最终都解决了，并且我们得到了一个非常干净的数组返回给我们。我们如何处理我们不确定 promises 结果的情况？考虑上一个练习中的`getFullRecords`函数；当我们运行该函数时，它执行以下操作：

![图 8.20：执行 getFullRecords 函数](img/C14587_08_20.jpg)

###### 图 8.20：执行 getFullRecords 函数

该函数同时执行所有三个操作，并在它们解决时解决。让我们修改`getFullRecords`函数以使其输出错误：

```js
function getFullRecords() {
   const ids = ['P6HB0O', '2ADN23', 'Not here'];
   const promises = ids.map(getFullRecord);
   return Promise.all(promises);
}
```

我们知道我们提供的第三个 ID 在我们的`getProfile`函数中不存在，因此它将被拒绝。当我们运行此函数时，我们将得到如下输出：

![图 8.21：运行 getProfile 函数时出错](img/C14587_08_21.jpg)

###### 图 8.21：运行 getProfile 函数时出错

`Promise.all`等待数组中的所有 promises 解决，并且如果其中一个请求被拒绝，它将返回一个拒绝的 promise。在处理多个 promises 时，请记住这一点；如果一个 promise 请求被拒绝，请确保您在错误消息中包含尽可能多的信息，以便您可以知道哪个操作被拒绝。

### 练习 64：使用 Promises 重构账单计算器

在上一个练习中，我们使用回调函数编写了账单计算逻辑。假设您工作的公司现在升级了他们的 Node.js 运行时，并且要求您使用 promises 重写该部分逻辑。打开`promises.js`文件，您将看到使用 promises 重写的更新后的`clientApi`：

#### 注意

Promises.js 可在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Exercise64`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Exercise64)找到。

+   您已经得到了支持 promises 的`clientApi`。

+   您需要实现一个功能，该功能计算用户的账单并以此格式返回结果：

```js
{
   id: 'XXXXX',
   address: '2323 sxsssssss',
   due: 236.6
}
```

+   您需要实现一个`calculateBill`函数，该函数接受一个 ID 并计算该用户的燃气账单。

+   您需要实现一个新的`calculateAll`函数来计算从`getUsers`获取的所有用户的账单。

我们将打开包含`clientApi`的文件并在那里进行工作。

执行以下步骤来实现练习：

1.  我们将首先创建`calculate`函数。这次，我们只会传递`id`：

```js
function calculate(id) {}
```

1.  在`calculate`中，我们将首先调用`getUsers`：

```js
function calculate(id) {
return clientApi.getUsers().then((result) => {
   const currentUser = result.users.find((user) => user.id === id);
   if (!currentUser) { throw Error('user not found'); }
}
}
```

因为我们想要计算并返回一个 promise，并且`getUsers`返回一个 promise，所以当我们调用`getUsers`时，我们将简单地返回 promise。在这里，我们将运行相同的`find`方法来找到我们当前正在计算的用户。然后，如果用户不存在，我们可以在`then`处理程序中直接抛出错误。

1.  在`getUsers`的`then`处理程序中调用`getUsage`：

```js
function calculate(id) {
return clientApi.getUsers().then((result) => {
   const currentUser = result.users.find((user) => user.id === id);
   if (!currentUser) { throw Error('user not found'); }
return clientApi.getUsage(currentUser.id).then((usage) => {
});
}
}
```

在这里，我们返回`clientApi`，因为我们想要链接我们的 promise，并且希望最内层的 promise 出现并被解决。

1.  在`getUsage`的`then`处理程序中调用`getRate`：

```js
function calculate(id) {
   return clientApi.getUsers().then((result) => {
      const currentUser = result.users.find((user) => user.id === id);
      if (!currentUser) { throw Error('user not found'); }
      return clientApi.getUsage(currentUser.id).then((usage) => {
         return clientApi.getRate(currentUser.id).then((rate) => {
   return {
      id,
      address: currentUser.address,
      due: (rate * usage.reduce((prev, curr) => curr + prev)).toFixed(2)
   };
});
});
}
}
```

这是我们需要调用的最后一个函数。我们也将在这里使用`return`。在我们的`then`处理程序中，我们将拥有所有我们需要的信息。在这里，我们可以直接运行我们的计算并直接返回值。该值将是我们返回的 promise 的解决值。

1.  创建一个`calculateAll`函数：

```js
function calculateAll() {}
```

1.  调用`getUsers`以获取我们用户的列表：

```js
function calculateAll() {
   return clientApi.getUsers().then((result) => {});
}
```

1.  在这里，结果将是我们系统中用户的列表。然后，我们将在每个用户上运行`calculate`。使用`Promise.all`和一个 map 数组来调用`calculate`函数对每个用户进行计算：

```js
function calculateAll() {
   return clientApi.getUsers().then((result) => {
      return Promise.all(result.users.map((user) => calculate(user.id)));
});
}
```

我们使用一个 map 数组来返回一个新的 promise 数组。当我们调用现有的`calculate`函数时，返回的 promise 数组将是 promise。当我们将该数组传递给`Promise.all`时，它将返回一个 promise，该 promise 将解析为来自 promise 列表的结果列表。

1.  在我们的一个用户上调用`calculate`：

```js
calculate('DDW2AU').then(console.log)
```

输出应该如下：

![图 8.22：调用我们的一个用户上的 calculate](img/C14587_08_22.jpg)

###### 图 8.22：在我们的一个用户上调用 calculate

1.  调用`calculateAll`函数：

```js
calculateAll().then(console.log)
```

输出应该如下：

![图 8.23：调用 calculateAll 函数](img/C14587_08_23.jpg)

###### 图 8.23：调用 calculateAll 函数

在以前的练习和活动中，我们创建了函数，使用回调从多个异步函数计算结果，然后使用 promise 重写了这些函数。现在，您知道如何使用 promise 重构旧的回调风格代码。当您在重构需要您开始使用 promise 的大型项目时，这是非常有用的。在下一章中，我们将介绍一种新的方法，可以用来处理异步函数。

## 异步和等待

JavaScript 开发人员一直梦想着处理异步函数而无需在其周围编写包装器。然后，引入了一个新功能，这改变了我们对 JavaScript 异步操作的认识。考虑我们在上一个练习中使用的代码：

```js
function getFullRecord(id) {
   return getProfile(id)
      .then(getCart)
      .then(getSubscription);
}
```

这很简单，因为我们使用了 promise 链式调用，但它并没有告诉我们更多的信息，看起来我们只是调用了一堆函数。如果我们可以有这样的东西会怎样：

```js
function getFullRecord(id) {
   const profile = getProfile(id);
   const cart = getCart(id);
   const subscription = getSubscription(id);
   return {
      ...profile,
      cart,
      subscription
   };
}
```

现在，当你看前面的代码时，它就更有意义了，看起来就像我们只是调用一些非异步函数来获取数据，然后返回组合数据。这就是 async 和 await 可以实现的。通过使用 async 和 await，我们可以像这样编写我们的代码，同时保持对异步操作的完全控制。考虑一个简单的`async`函数，它返回一个 promise：

```js
function sayHello() {
   return Promise.resolve('hello world');
}
```

这只是一个简单的`async`函数，就像我们在以前的练习和活动中使用的那样。通常，如果我们想调用这个函数并获取返回的 promise 的值，我们需要执行以下命令：

```js
sayHello().then(console.log)
```

输出应该如下：

![图 8.24：获取返回的 promise 的值](img/C14587_08_24.jpg)

###### 图 8.24：获取返回的 promise 的值

这种方法并不新鲜；我们仍然调用函数返回一个 promise，然后通过`then`处理程序获取解析后的值。如果我们想要使用新的 async 和 await 功能，我们首先创建一个将运行操作的函数：

```js
async function printHello() {
   // Operation here
}
```

我们所做的就是在`function`关键字之前添加`async`。我们这样做是为了将这个函数标记为`async`函数，这样我们就可以在`printHello()`函数中使用`await`来调用`sayHello`函数，而不需要使用`then`处理程序：

```js
async function printHello() {
   // Operation here
   const message = await sayHello();
   console.log(message);
}
```

在这个`async`函数中，我们调用了我们的`sayHello`函数，它返回一个 promise。因为我们在之前使用了`await`关键字，它将尝试解析该 promise 并将解析后的值传递给我们声明为消息的常量。通过使用这个，我们让我们的`async`函数看起来像一个同步函数。稍后，我们可以像调用普通函数一样调用该函数：

```js
printHello();
```

输出应该如下：

![图 8.25：调用 printHello 函数](img/C14587_08_25.jpg)

###### 图 8.25：调用 printHello 函数

### 练习 65：异步和等待函数

在这个练习中，我们将学习创建 async 函数并在其他 async 函数中调用它们。在单个函数中处理大量的 async 操作时，使用 async 和 await 可以帮助我们。我们将一起编写我们的第一个`async`函数，并探索在应用程序中处理 async 和 await 时需要牢记的一些事情。

#### 注意

此活动的代码文件可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Exercise65`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Exercise65)找到。

执行以下步骤完成练习：

1.  创建一个`getConcertList`函数：

```js
function getConcertList() {
   return Promise.resolve([
      'Magical Mirai 2018',
      'Magical Mirai 2019'
   ]);
}
```

1.  调用函数并使用`await`：

```js
const concerts = await getConcertList();
```

当我们运行上述代码时，我们将会得到如下错误：

![图 8.26：使用 await 调用函数](img/C14587_08_26.jpg)

###### 图 8.26：使用 await 调用函数

我们会得到这个错误的原因是我们只能在`async`函数内部使用`await`关键字。如果我们想使用它，我们必须将语句包装在`async`函数中。

1.  修改语句并将其包装在`async`函数中：

```js
async function printList() {
   const concerts = await getConcertList();
   console.log(concerts);
}
printList();
```

输出应该如下：

![图 8.27：修改语句并将其包装在 async 函数中](img/C14587_08_27.jpg)

###### 图 8.27：修改语句并将其包装在 async 函数中

当我们运行这个函数时，我们将看到列表被打印出来，一切都运行正常。我们也可以将`async`函数视为返回 promise 的函数，因此如果我们想在操作结束后运行代码，我们可以使用`then`处理程序。

1.  使用`async`函数的`then()`函数调用处理程序：

```js
printList().then(() => {
   console.log('I am going to both of them.')
});
```

输出应该如下：

![图 8.28：使用 async 函数的 then 函数调用处理程序](img/C14587_08_28.jpg)

###### 图 8.28：使用 async 函数的 then 函数调用处理程序

现在，我们知道`async`函数的行为就像返回 promise 的普通函数一样。

1.  创建一个`getPrice`函数来获取音乐会的价格：

```js
function getPrice(i) {
   const prices = [9900, 9000];
   return Promise.resolve(prices[i]);
}
```

1.  修改`printList`以包括从`getPrice`获取的价格：

```js
async function printList() {
   const concerts = await getConcertList();
   const prices = await Promise.all(concerts.map((c, i) => getPrice(i)));
   return {
      concerts,
      prices
   };
}
printList().then(console.log);
```

在这个函数中，我们只是尝试使用`getPrice`函数获取所有的价格。在上一节中，我们提到了如何使用`Promise.all`将一个 promise 数组包装在一个 promise 中，该 promise 只有在数组中的每个 promise 都解析后才会解析。因为`await`关键字可以用于返回 promise 并解析其值的任何函数，我们可以使用它来获取一个价格数组。当我们运行上述代码时，我们将看到这个函数解析为以下内容：

![图 8.29：修改 printList 以包括从 getPrice 获取的价格](img/C14587_08_29.jpg)

###### 图 8.29：修改 printList 以包括从 getPrice 获取的价格

这意味着如果我们有一个返回 promise 的函数，我们不再需要使用`then`处理程序。在`async`函数中，我们可以简单地使用`await`关键字来获取解析后的值。但是，在`async`函数中处理错误的方式有点不同。

1.  创建一个返回 rejected promise 的`buggyCode`函数：

```js
function buggyCode() {
   return Promise.reject(new Error('computer: dont feel like working today'));
}
```

1.  在`printList`中调用`buggyCode`：

```js
async function printList() {
   const concerts = await getConcertList();
   const prices = await Promise.all(concerts.map((c, i) => getPrice(i)));
   await buggyCode();
   return {
      concerts,
      prices
   };
}
printList().then(console.log);
```

输出应该如下：

![图 8.30：在 printList 中调用 buggyCode](img/C14587_08_30.jpg)

###### 图 8.30：在 printList 中调用 buggyCode

因为`buggyCode`抛出了一个错误，这会停止我们的函数执行，并且将来甚至可能终止我们的进程。为了处理这种类型的错误，我们需要捕获它。

1.  在 buggyCode 上使用`catch`处理程序：

```js
async function printList() {
   const concerts = await getConcertList();
   const prices = await Promise.all(concerts.map((c, i) => getPrice(i)));
   await buggyCode().catch((error) => {
      console.log('computer produced error');
      console.log(error);
   });
   return {
      concerts,
      prices
   };
}
printList().then(console.log);
```

我们可以像处理常规 promise 一样处理`buggyCode`的错误，并传递一个`catch`处理程序。这样，promise rejection 将被标记为已处理，并且不会返回`UnhandledPromiseRejectionWarning`：

![图 8.31：在 buggyCode 上使用 catch 处理程序](img/C14587_08_31.jpg)

###### 图 8.31：在 buggyCode 上使用 catch 处理程序

这是处理`async`函数中的 promise rejection 的一种方法。还有一种更常见的方法。

1.  使用`try`…`catch`修改错误处理：

```js
async function printList() {
   const concerts = await getConcertList();
   const prices = await Promise.all(concerts.map((c, i) => getPrice(i)));
   try {
      await buggyCode();
   } catch (error) {
      console.log('computer produced error');
      console.log(error);
   }
   return {
      concerts,
      prices
   };
}
printList().then(console.log);
```

输出应该如下所示：

![图 8.32：使用 try…catch 修改错误处理](img/C14587_08_32.jpg)

###### 图 8.32：使用 try…catch 修改错误处理

使用`try`…`catch`是许多开发人员在处理可能抛出错误的函数时熟悉的。使用`try`…`catch`块来处理我们的`buggyCode`的错误将使代码更易读，并实现异步的目标，即消除传递 promise 处理程序。接下来，我们将讨论如何正确处理多个 promise 和并发性。

### 异步等待并发性

在处理 JavaScript 中的多个异步操作时，了解你想要运行的操作的顺序至关重要。你编写代码的方式可以很大程度上改变应用程序的行为。让我们看一个例子：

```js
function wait(seconds) {
   return new Promise((resolve) => {
      setTimeout(() => {
        resolve();
      }, seconds * 1000);
   });
}
```

这是一个非常简单的函数，它返回一个 promise，只有在经过`n`秒后才会解析。为了可视化并发性，我们声明了`runAsync`函数：

```js
async function runAsync() {
   console.log('starting', new Date());
   await wait(1);
   console.log('i waited 1 second', new Date());
   await wait(2);
   console.log('i waited another 2 seconds', new Date());
}
```

当我们运行这个函数时，我们会看到我们的程序会等待 1 秒并打印出第一条语句，然后在 2 秒后打印出另一条语句。总等待时间将是 3 秒：

![图 8.33：返回在 n 秒后解析的 promise 的函数](img/C14587_08_33.jpg)

###### 图 8.33：返回在 n 秒后解析的 promise 的函数

如果我们想要同时运行两个`wait`函数呢？在这里，我们可以使用`Promise.all`：

```js
async function runAsync() {
   console.log('starting', new Date());
   await Promise.all([wait(1), wait(2)]);
   console.log('i waited total 2 seconds', new Date());
}
```

输出应该如下所示：

![图 8.34：使用 Promise.all 运行两个等待函数](img/C14587_08_34.jpg)

###### 图 8.34：使用 Promise.all 运行两个等待函数

我们在这里做的是移除了`await`，并将`wait`函数返回的两个 promise 放入数组中，然后将其传递给`Promise.all`。当我们移除`await`关键字并使用`Promise.all`时，我们可以确保代码不会失控并将继续执行。如果你在循环中处理 promise，就像下面的代码一样：

```js
async function runAsync() {
   console.log('starting', new Date());
   for (let i = 0; i < 2; i++) {
      await wait(1);
   }
   console.log('i waited another 2 seconds', new Date());
}
```

这不提供并发性。想象一下，我们不是在等待，而是从数据库中获取用户信息：

```js
async function runAsync() {
   const userProfiles = [];
   for (let i = 0; i < 2; i++) {
      const profile = await getProfile(i);
      userProfiles.push(profile);
   }
   return userProfiles;
}
```

在这里，我们的用例是从数据库中获取多个用户配置文件。虽然前面的代码可以工作，但它不是最高效的实现。正如我们之前提到的，这段代码会等到最后一个请求完成后才会获取下一个请求。为了优化这段代码，我们可以简单地使用`array.map`和`Promise.all`结合使用：

```js
async function runAsync() {
   return await Promise.all([0, 1].map(getProfile));
}
```

这样，我们不是等待每个操作完成；我们只是等待包装 promise 被解析。在 map 数组中，我们只是生成了 promises，一旦它们被创建，它将执行我们的操作。与`for`循环方法相比，我们不需要等待前一个 promise 在执行下一个 promise 之前解决。我们将在下一章讨论它们的区别。

### 何时使用 await

在之前的例子中，我们讨论了在我们的`async`函数中使用`await`关键字。但是什么时候应该使用`await`，什么时候应该避免呢？在上一节中，我们讨论了当我们想要启用并发并确保操作不会互相等待时，应避免使用`await`。考虑以下代码示例：

```js
async function example() {
   const result1 = await operation1();
   const result2 = await operation2(result1.something);
   return result2;
}
```

在这个例子中，`operation2`函数只有在`operation1`完成后才会执行。当你有依赖关系并且`result2`依赖于`result1`中的某些内容时，这是很有用的，就像例子中所示的那样。如果它们之间没有相互依赖，你可以利用`Promise.all`来确保并发性：

```js
async function example() {
   const result1 = operation1();
   const result2 = operation2();
   return await Promise.all([result1, result2]);
}
```

没有`await`关键字，代码只是将从两个操作返回的 promise 分配给我们声明的常量。这确保了`operation2`在`operation1`之后立即触发，并且没有等待。我们还需要注意的另一点是错误处理。考虑我们在上一个练习中使用的`buggyCode`：

```js
function buggyCode() {
   return Promise.reject(new Error('computer: dont feel like working today'));
}
```

这个函数只是返回一个被拒绝的 promise。在使用它时，我们应该使用`catch`来处理 promise 的错误：

```js
async function printList() {
   try {
      await buggyCode();
   } catch (error) {
      console.log('computer produced error');
      console.log(error);
   }
}
```

当我们运行这段代码时，我们会看到我们的错误被很好地处理，并且错误消息被记录下来。在这里，我们在运行`buggyCode`函数时使用了`await`，但是当我们删除`await`关键字时，我们将看到以下内容：

![图 8.35：删除 await 关键字后运行 buggyCode 函数

将以下文本按行翻译成中文：

###### 图 8.35：删除 await 关键字后运行 buggyCode 函数

您会看到我们有一个未处理的 promise 拒绝；它似乎没有出现，因为我们的`try`…`catch`什么也没做。这是因为没有`await`关键字，JavaScript 不会尝试等待 promise 解析；因此，它不知道将来会抛出错误。这个`try`…`catch`块将捕获的是在执行函数时抛出的错误。这是我们在使用`async`和`await`编写代码时需要牢记的事情。在下一个练习中，我们将编写一个调用多个`async`函数并能够从错误中恢复的复杂函数。

### 练习 66：复杂的异步实现

在这个练习中，我们将构建一个非常复杂的`async`函数，并使用我们之前学到的一切来确保函数具有高性能并对错误具有弹性。

#### 注意

此活动的代码文件可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Exercise66`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson08/Exercise66)找到。

完成练习的步骤如下：

1.  创建一个`getPlaylists`函数，根据播放列表名称返回一个 ID 数组：

```js
function getPlaylist(id) {
   const playLists = {
      'On the road': [0, 6, 5, 2],
      'Favorites' : [1, 4, 2],
      'Corrupted': [2, 4, 7, 1]
   };
   const playList = playLists[id];
   if (!playList) {
      throw new Error('Playlist does not exist');
   }
   return Promise.resolve(playLists[id]);
}
```

该函数将返回一个歌曲 ID 数组作为播放列表。如果未找到，它将简单地返回`null`。

1.  创建一个`getSongUrl`函数，根据编号`id`返回一个歌曲 URL：

```js
function getSongUrl(id) {
   const songUrls = [
      'http://example.com/1.mp3',
      'http://example.com/2.mp3',
      'http://example.com/3.mp3',
      'http://example.com/4.mp3',
      'http://example.com/5.mp3',
      'http://example.com/6.mp3',
      'http://example.com/7.mp3',
   ];
   const url = songUrls[id];
   if (!url) {
      throw new Error('Song does not exist');
   }
   return Promise.resolve(url); // Promise.resolve returns a promise that is resolved with the value given
}
```

1.  创建一个`playSong`异步函数，该函数接受歌曲的 ID 并生成两个输出-一个显示正在播放的歌曲，另一个通知用户歌曲已经完成：

```js
async function playSong(id) {
   const url = await getSongUrl(id);
   console.log(`playing song #${id} from ${url}`);
   return new Promise((resolve) => {
      setTimeout(() => {
        console.log(`song #${id} finished playing`);
        resolve();
      }, Math.random() * 3 * 1000);
   });
}
```

1.  创建一个`playPlaylist`函数，该函数接受一个播放列表 ID，并在播放列表中的每首歌曲上调用`playSong`：

```js
async function playPlaylist(id) {
   const playList = await getPlayLlist(id);
   await Promise.all(playList.map(playSong));
}
```

这是一个简单的实现，没有进行错误处理。

1.  运行`playPlaylist`函数：

```js
playPlaylist('On the road').then(() => {
   console.log('finished playing playlist');
});
```

输出应该如下：

![图 8.36：运行 playPlaylist 函数](img/C14587_08_36.jpg)

###### 图 8.36：运行 playPlaylist 函数

我们得到了一个非常有趣的输出；它同时播放所有歌曲。而且，它没有优雅地处理错误。

1.  不带参数调用`playPlaylist`：

```js
playPlaylist().then(() => {
   console.log('finished playing playlist');
});
```

输出应该如下：

![图 8.37：不带参数调用 playPlaylist](img/C14587_08_37.jpg)

###### 图 8.37：不带参数调用 playPlaylist

我们之所以出现这个错误是因为当`getPlaylist`抛出错误时，我们没有处理错误。

1.  修改`playPlaylist`以处理错误：

```js
async function playPlaylist(id) {
   try {
      const playList = await getPlaylist(id);
      return await Promise.all(playList.map(playSong));
   } catch (error) {
      console.log(error);
   }
}
```

我们在这里没有做任何特别的事情；我们只是在`getPlaylist`周围添加了一个`try…catch`块，这样当 promise 被拒绝时，它将被正确处理。更新后，当我们再次运行我们的代码时，我们将收到以下输出：

![图 8.38：修改 playPlaylist 以处理错误](img/C14587_08_38.jpg)

###### 图 8.38：修改`playPlaylist`以处理错误

我们看到错误已经被正确处理，但是我们仍然在最后得到了`finished`消息。这是我们不想要的，因为当发生错误时，我们不希望 promise 链继续。

1.  修改`playPlaylist`函数和调用者：

```js
async function playPlaylist(id) {
   const playList = await getPlaylist(id);
   return await Promise.all(playList.map(playSong));
}
playPlaylist().then(() => {
   console.log('finished playing playlist');
}).catch((error) => {
   console.log(error);
});
```

在编写`async`代码时，最好将 promise 处理放在父级，并让错误冒泡。这样，我们可以为此操作只有一个错误处理程序，并能够一次处理多个错误。

1.  尝试调用一个损坏的播放列表：

```js
playPlaylist('Corrupted').then(() => {
   console.log('finished playing playlist');
}).catch((error) => {
   console.log(error);
});
```

](Images/C14587_08_35.jpg)

![图 8.39：调用一个损坏的播放列表](img/C14587_08_39.jpg)

###### 图 8.39：调用损坏的播放列表

这段代码运行良好，并且错误已经处理，但仍然一起播放。我们想要显示`finished`消息，因为`歌曲不存在`错误是一个小错误，我们想要抑制它。

1.  修改`playPlaylist`以按顺序播放歌曲：

```js
async function playPlaylist(id) {
   const playList = await getPlaylist(id);
   for (const songId of playList) {
      await playSong(songId);
   }
}
```

输出应如下所示：

![图 8.40：修改后的 playPlaylist 以按顺序播放歌曲](img/C14587_08_40.jpg)

###### 图 8.40：修改`playPlaylist`以按顺序播放歌曲

在修改中，我们删除了`Promise.all`，并用`for`循环替换了它，对每首歌曲使用`await`。这确保我们在继续下一首歌曲之前等待每首歌曲完成。

1.  修改`playSong`以抑制`未找到`错误：

```js
async function playSong(id) {
   try {
      const url = await getSongUrl(id);
      console.log('playing song #${id} from ${url}');
      return new Promise((resolve) => {
        setTimeout(() => {
           console.log('song #${id} finished playing');
           resolve();
        }, Math.random() * 3 * 1000);
      });
   } catch (error) {
      console.log('song not found');
   }
}
```

输出应如下所示：

![图 8.41：修改后的 playSong 以抑制未找到的错误](img/C14587_08_41.jpg)

###### 图 8.41：修改`playSong`以抑制未找到的错误

我们在这里做的是用`try`...`catch`块包装我们的逻辑。这使我们能够抑制代码生成的任何错误。当`getSongUrl`抛出错误时，它不会上升到父级；它将被`catch`块捕获。

在这个练习中，我们使用`async`和`await`实现了一个播放列表播放器，并使用了我们对`Promise.all`和`async`并发的了解来优化我们的播放列表播放器，使其一次只播放一首歌曲。这使我们能够更深入地了解 async 和 await，并在将来实现我们自己的`async`函数。在下一节中，我们将讨论如何将现有的基于 promise 或回调的代码迁移到 async 和 await。

### 活动 12：使用 Async 和 Await 重构账单计算器

您的公司再次更新了其 Node.js 运行时。在此活动中，我们将使用 async 和 await 重构之前创建的账单计算器：

+   您获得了使用承诺实现的`clientApi`。

+   您需要将`calculate()`更新为`async`函数。

+   您需要将`calculateAll()`更新为`async`函数。

+   `calculateAll()`需要使用`Promise.all`一次获取所有结果。

打开`async.js`文件，使用`async`和`await`实现`calculate`和`calculateAll`函数。

#### 注意

此活动的代码文件可以在[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson08/Activity12/Activity12.js`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson08/Activity12/Activity12.js)找到。

执行以下步骤完成活动：

1.  创建一个`calculate`函数，以 ID 作为输入。 

1.  在`calculate`中，使用`await`调用`clientApi.getUsers()`来检索所有用户。

1.  使用`array.find()`使用`id`参数找到`currentUser`。

1.  使用`await`调用`getUsage()`来获取该用户的使用情况。

1.  使用`await`调用`getRate`以获取用户的费率。

1.  返回一个新对象，其中包括`id`、`address`和总应付金额。

1.  将`calculateAll`函数编写为`async`函数。

1.  使用`await`调用`getUsers`以检索所有用户。

1.  使用数组映射创建一个承诺列表，并使用`Promise.all`将它们包装起来。然后，在由`Promise.all`返回的承诺上使用等待，并返回其值。

1.  在一个用户上调用`calculate`。

1.  调用`calculateAll`。

输出应如下所示：

![图 8.42：调用 calculateAll 函数](img/C14587_08_42.jpg)

###### 图 8.42：调用 calculateAll 函数

#### 注意

此活动的解决方案可在第 615 页找到。

### 将回调和 Promise-Based 代码迁移到 Async 和 Await

在处理大型项目时，经常需要使用 async 和 await 重构现有代码。在进行这些重构时，我们需要牢记应保持相同的功能和错误处理类型。在本节中，我们将学习如何将现有的回调和基于 promise 的代码迁移到 async 和 await。

### 将基于回调的代码迁移到 Async 和 Await

当我们迁移基于回调的代码时，我们需要重写函数，并确保它返回一个 promise 而不是使用回调。考虑以下代码：

```js
function makeRequest(param, callback) {
   request(param, (err, data) => {
      if (err) {
        return callback(err);
      }
      const users = data.users;
      callback(null, users.map((u) => u.id));
   });
}
```

上述代码接受一个参数并调用`request`模块，我们无法修改它，并返回用户 ID 的列表。一旦完成，如果出现错误，它将通过回调简单地返回。当我们想要使用 async 和 await 重构这段代码时，我们可以首先确保它返回一个 promise。这样做的同时，我们也想删除`callback`参数：

```js
function makeRequest(param) {
   return new Promise((resolve, reject) => {
      // Logic here
   });
}
```

然后，我们需要把我们的逻辑复制到：

```js
function makeRequest(param) {
   return new Promise((resolve, reject) => {
      request(param, (err, data) => {
        if (err) {
           return callback(err);
        }
        const users = data.users;
        callback(null, users.map((u) => u.id));
      });
   });
}
```

在这里，我们需要进行修改。我们需要删除所有对`callback`的引用，并改用`reject`和`resolve`：

```js
function makeRequest(param) {
   return new Promise((resolve, reject) => {
      request(param, (err, data) => {
        if (err) {
           return reject(err);
        }
        const users = data.users;
        resolve(users.map((u) => u.id));
      });
   });
}
```

您可以在这里看到，我们在调用`request`时仍在使用回调样式。那是因为我们无法控制外部库。我们能做的是确保每次调用它时，我们都返回一个 promise。现在，我们已经完全将我们的传统代码转换为现代标准。您现在可以像这样在`async`函数中使用它：

```js
async function use() {
   const userIds = await makeRequest({});
}
```

通常，代码重构要困难得多。建议从最基本的级别开始，随着重构的进行逐步提升。当处理嵌套回调时，确保使用`await`来确保保留依赖关系。

## 总结

在本章中，我们讨论了如何使用 promise 和 async 和 await 更好地管理代码中的异步操作。我们还谈到了将现有的回调代码重构为 async 和 await 的各种方法。在我们的应用程序中使用 async 和 await 不仅有助于使我们的代码更易读，还将帮助我们对实现进行未来测试。在下一章中，我们将讨论如何在我们的应用程序中使用基于事件的编程。
