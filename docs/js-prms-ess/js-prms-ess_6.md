# 第六章：综合运用——承诺在行动

在第五章中，我们介绍了 WinJS 库，并详细了解了 WinJS 承诺对象。我们还快速浏览了如何在 Windows 应用程序开发中使用 WinJS 承诺的基本示例。最后，我们来到了最后一章，在这一章中，我们将把本书关于承诺的学习付诸实践。我们将尝试通过创建一个简单的实现来更深入地了解承诺是如何工作的。创建实现库之后，我们将在一个基本示例中使用它，利用该库执行异步操作。在本章中，我们将介绍以下主题：

+   总结我们已经涵盖和学到的内容

+   在简单的 JavaScript 库中创建承诺实现

# 实现承诺库

承诺已经变得非常流行，这可以从它们的许多独立实现中看出。此外，Promises/A+已经有超过 35 个符合要求的实现，随着 ECMAScript 6 的推出，这个数字还在增长。值得注意的是，JavaScript 中 Promise/A+的日益普及在其他语言中也得到了体现，ActionScript、Python 和 Objective C 中都有许多实现。尽管从语义上讲，由于不同的语言能力，这些实现可能并不一定与 JavaScript 规范中的实现相匹配，但直接将它们与 Promise/A+的 JavaScript 测试套件进行测试是无法验证它们是否符合要求的。然而，提及这些实现并展示所付出的努力是有价值的。

让我们通过一个代码示例来了解承诺的基本实现；这将使我们更好地了解承诺是如何工作的。深入理解事物的工作原理可以提高我们利用代码和当它出错时更轻松、更快地调试的能力。我们将创建一个最小的 JavaScript 库来实现承诺，并从承诺的状态开始编写这个库。我们在第二章中了解到，承诺有三种不同的状态：pending（等待中）、fulfilled（已履行）和 rejected（已拒绝）。

承诺的规范没有为这些状态指定一个值，所以让我们声明它们，并将值分配给一个枚举器，如下面的代码所示：

```js
var promState = {
 pending: 1,
 fulfilled: 2,
 rejected: 3
};
```

这个枚举将允许我们通过名称来调用状态，例如，`promState.fulfilled`。接下来，我们将创建一个对象，它包含了从状态转换到`then`方法的整个承诺逻辑，并解决承诺。让我们称这个对象为`PromiseMe`。

首先，我们需要定义承诺状态的变化及其从一个状态转换到另一个状态的过渡。规范详细说明了状态间转换的一些规则和考虑因素，我们在第二章，*承诺 API 及其兼容性*中进行了深入的讨论。这些规则可以总结如下：

+   承诺在某一时间点只能处于一个状态。

+   当一个承诺从待处理状态转换为其他任何状态，无论是已履行还是已拒绝，它都不能回去。

+   当一个承诺被履行时，它必须有一个值（甚至可以是 undefined），而当它失败时，它必须有一个原因（任何指定承诺被拒绝原因的值）。

在`PromiseMe`对象内部，我们首先定义一个名为`changeMyState`的函数，该函数根据前面的规则处理和管理这个承诺的状态转换，如下面的代码所示：

```js
var PromiseMe = {
    //set default state
 myState: promState.pending,
 changeMyState: function(newState, newValue) {

  // check if we are changing to same state and report it
  if (this.myState == newState) {
   throw new Error("Sorry, But you can't do this to me! You are transitioning to same state: " + newState);
  }

  // trying to get out of the fulfilled or rejected states
  if ( this.myState == promState.fulfilled ||
    this.myState == promState.rejected ) {
   throw new Error("You can't leave this state now: " + this.myState);
  }
  // if promise is rejected with a null reason
  if ( newState == promState.rejected &&
    newValue === null ) {
   throw new Error("If you get rejected there must be a reason. It can't be null!");
  }

  // if there was no value passed with fulfilled
  if (newState == promState.fulfilled &&
    arguments.length < 2 ) {
   throw new Error("I am sorry but you must have a non-null value to proceed to fulfilled!");
  }

  //we passed all the conditions, we can now change the state
  this.myState = newState;
  this.value = newValue;return this.myState;
 }
};
```

对象内的代码首先设置一个名为`myState`的属性，将其值设置为枚举`promState`的待处理值`promState.pending`。随后，我们设置一个名为`changeMyState`的属性，其值为一个匿名函数，该函数接受两个参数：`newState`和`value`。在这个函数中，我们处理状态转换并检查它是否符合规则。在我们继续编写代码之前，有四个检查点：

1.  首先，我们检查我们是否正在转换到同一个状态，并抛出错误。

1.  在第二个检查中，我们确保承诺不是试图从已拒绝或已履行的状态转换，并相应地抛出错误。

1.  第三个检查是针对传递给拒绝的值。如果它是 null，将抛出一个错误，这确保了承诺因除 null 之外的值而被拒绝。我们编写这个检查点，因为根据规范，承诺只接受非 null 值。

1.  最后的检查将是履行状态及其值；我们用`arguments.length < 2`来确定是否有在第二个参数中传递的值；如果没有，我们抛出一个错误。

    ### 提示

    我给错误信息赋予了有意义的措辞，以便更好地理解我们在这些条件下检查的内容。在我们通过所有的条件语句后，我们通过将`changeMyState`方法的`myState`属性设置为通过参数传递的`newState`，来关闭`changeMyState`方法。我们还将值分配给`newValue`参数，并以返回`this.myState`结束，反过来返回承诺的状态。

## 实现 then 方法

在我们的实现中，接下来是`then`方法。这是承诺的核心，也是使承诺变得有用的关键。这个方法允许并实现承诺的链式调用和错误处理。我们将实现一个基本的`then`方法，该方法首先检查承诺的有效性规则。

让我们将`then`方法定义如下：

```js
then: function (onFulfilled, onRejected) {
        // define an array named handlers
        this.handlers = this.handlers || [];
        // create a promise object to return
        var returnedPromise = Object.create(PromiseMe);

        this.handlers.push({
            fulfillPromise: onFulfilled,
            rejectPromise: onRejected,
            promise: returnedPromise
        });
        return returnedPromise;
    }
```

之前代码所做的基本工作是为这个 promise 定义一个`then`方法。`Then`被定义为一个匿名函数，它接受两个参数：`onFulfilled`和`onRejected`。我们为这个 promise 定义一个数组，并初始化为`this.handlers`（如果存在的话）当前数组或一个新的数组（如果不存在）。我们实例化一个新的 promise 并将其存储在`returnedPromise`变量中。我们将`onFulfilled`、`onRejected`和`returnedPromise`存储在数组中，这样我们可以在返回 promise 之后调用这些处理程序。这个函数以返回 promise 结束。

### 注意

根据 Promise/A+规范，`then`方法的规则指出，函数参数：`onFulfilled`和`onRejected`，只能在 promise 被满足或拒绝后调用。这就是为什么在实现中，我们将这两个函数存储在一个数组中，以便我们稍后可以调用它们。

你可能会注意到`handlers`数组包含两个属性：`fulfillPromise`和`rejectPromise`。这两个函数被设置为传递给`then`方法的处理器。让我们定义这两个函数，这样我们稍后就可以在`resolve`方法中使用它们。这些函数是辅助方法，它们允许我们手动改变 promise 的状态。此外，这些函数将调用`changeMyState`方法来改变 promise 的状态，进而返回一个状态。

```js
fulfillPromise: function (value) {
//change state to fulfilled and return a promise with a value
        this.changeMyState(promState.fulfilled, value);
    },
rejectPromise: function (reason) {
//change state to rejected and return a promise rejected with a reason
        this.changeMyState(promState.rejected, reason);
    }
```

## 定义一个解决方法

接下来，我们需要解决 promise 的解析问题。我们需要定义一个解决方法，该方法将处理 promise 并且将根据 promise 的状态来满足它或拒绝它。你可以把`resolve`方法看作是一个内部方法，promise 调用它，并且旨在仅在 promise 被满足时执行`then`调用；从字面上讲，它解决了一个被满足的 promise。实际上，为了满足一个 promise 或拒绝它，你需要调用一个函数，在我们的案例中是`changeMyState`。让我们先根据以下代码为`resolve`方法创建一个基本逻辑：

```js
    resolve: function () {
        // check for pending and exist
        if (this.myState == promState.pending) {
            return false;
        }
```

之前的代码将`resolve`属性分配给一个函数。在这个函数内部，我们首先检查这个 promise 的状态。如果它是 pending 状态，我们返回`false`。在接下来的代码中，我们将遍历包含我们在`then`方法中定义的处理器的数组：

```js
// loop through each then as long as handlers array contains items
while(this.handlers && this.handlers.length) {

//return and remove the first item in array
var handler = this.handlers.shift();
```

在循环内部，我们对数组应用了`shift()`函数。该`shift()`函数允许我们从数组中检索第一个元素并直接删除它。因此，`handler`变量将包含`handlers`数组中的第一个元素，而作为回应，`handlers`数组将包含所有元素减去现在存储在`var` handler 中的第一个元素。

在`resolve`函数中接下来，我们将定义一个名为`doResolve`的变量，其值根据状态要么是`fulfillPromise`函数，要么是`rejectPromise`处理程序，如下面的代码所示：

```js
//set the function depending on the current state
var doResolve = (this.myState == promState.fulfilled ? handler.fulfillPromise : handler.rejectPromise);
```

### 提示

前面的语法使用了三元运算符。它被称为三元运算符，是因为与其他所有需要两个值的运算符不同，这个运算符实际上需要第三个值放在运算符的中间。它就像是一个单条语句的`if`语句的简写形式，其中`if`和`else`子句将不同的值赋给同一个变量，如下面的示例所示：

```js
if (condition == true) result = "pick me"; else result = "No! pick me instead";
```

三元运算符将`if`语句转换为以下单行条件语句：

```js
result = (condition == true) ? "pick me" : "No! pick me instead";
```

我们需要对`doResolve`函数进行一些逻辑检查。如果它不是函数类型或者该函数不存在，那么我们调用`changeMyState`方法来改变承诺的状态并传递状态和值：

```js
//if doResolve is not a function
if (typeof doResolve != 'function') {
handler.promise.changeMyState(this.myState, this.value);

}
```

## 实现 doResolve 函数

这段代码的另一种情况是`doResolve`函数存在，我们需要用值返回承诺，或者用错误拒绝它。所以，我们在`if`条件后跟一个`else`语句来实现这个情况，如下面的代码所示：

```js
else {
//fulfill the promise with value or reject with error
try {
```

根据目前的代码逻辑，我们现在应该有`doResolve`包含`handler.fulfillPromise`或`handler.rejectPromise`函数之一。这两个函数可以手动改变承诺的状态，并接受一个参数，即当前值或当前原因。这两个值都包含在`this.value`变量中。因此，我们将当前值传递给`doResolve`，并将结果赋给一个名为`promiseValue`的变量，如下面的代码行所示：

```js
var promiseValue = doResolve(this.value);
```

接下来，我们需要管理随`promiseValue`返回的承诺。首先，我们检查承诺是否存在，并且是否有一个有效的`then`函数，如下面的代码所示：

```js
// deal with promise returned
        if (promiseValue && typeof promiseValue.then == 'function') {
```

假设我们通过了这个条件，我们可以在其中调用`promiseValue`的`then`方法，因为现在它包含了一个由`doResolve`函数返回的承诺。我们将两个参数传递给它的`then`方法：一个函数参数`onFullfilled`，另一个参数`onRejected`，如下面的代码所示：

```js
//invoke then on the promise
promiseValue.then(function (val) {
    handler.promise.changeMyState(promState.fulfilled, val);
}, function (error) {
    handler.promise.changeMyState(promState.rejected, error);
});
}
```

另一方面，如果`promiseValue`返回的值不是一个承诺，我们将不需要调用`then`方法。相反，我们简单地将状态更改为已兑现，并传递值。我们将处理这个情况，如下面的代码所示：

```js
// if the value returned is not a promise
else {
handler.promise.changeMyState(promState.fulfilled, promiseValue);
}
```

最后，因为我们处于一个`try`语句中，我们将相应地提供一个`catch`语句，以处理操作失败时抛出的任何错误。在那个`catch`语句中，我们将承诺的状态更改为已拒绝，并传递产生的错误。我们还将关闭所有尾随的花括号：

```js
// deal with error thrown
} catch (error) {
handler.promise.changeMyState(promState.rejected, error);
   }
}
}
}
```

解决 promise 包括一些繁琐的检查，但这些是确保 promise 实现与规范保持一致的必要条件。正如你所见，我们在进行中添加了逻辑，开始时只是一个简单的检查，看看我们是否根据 promise 状态运行`onFulfilled`或`onRejected`函数。接着，根据返回值改变它们对应的 promise 状态。

### 提示

请记住，实现需要遵循规范中存在的考虑和规则。在任何时间点，你可以通过查看本书第二章*The Promise API and Its Compatibility*中解释的 Promise API 的详细信息来核对代码。

我们已经接近完成，剩下的是我们还没有解决的两种场景。第一个场景是`onFulfilled`和`onRejected`处理程序必须在事件循环的同一轮中（当`this.handlers && this.handlers.length`时）不得调用。我们进行这个检查是因为`while`正在遍历每个`then`调用。在`then`调用中，promise 要么被解决要么被拒绝。因此，在我们这里，我们有`onFulfilled`和`onRejected`处理程序。为了解决这个问题，我们将在事件循环之后仅将`then`方法添加到数组中。我们可以使用`setTimeout`函数来实现这一点，从而确保我们始终以异步方式运行。让我们在`then`方法中添加`setTimeout`函数，并将存储 promise 处理程序的函数包装起来，如下面的代码所示：

```js
var that = this;setTimeout(function () {
    that.handlers.push({
         fulfillPromise: onFulfilled,
         rejectPromise: onRejected,
         promise: returnedPromise
      });
    that.resolve();
 }, 2);
```

## 包装代码

在这个实现中的最后一步将是指出我们实际上何时解决 promise。我们需要检查两个条件。第一个条件是我们添加`then`方法时，因为 promise 的状态可能已经在那里设置。第二个情况是在`changeMyState`函数中改变 promise 状态。因此，我们需要在`changeMyState`函数的末尾添加一个`this.resolve()`调用。在最终确定实现之前，我们需要做的一切就是将所有代码包裹在一个名为`PromiseMe`的无名函数中。它将使用`Object.create`给我们一个 promise。有了这个，这个 promise 实现的最终代码将如下所示：

```js
var PromiseMe = function () {
    var promState = {
        pending: 1,
        fulfilled: 2,
        rejected: 3
    };
    //check the enumeration of promise states

    var PromiseMe = {
        //set default state
        myState: promState.pending,
        changeMyState: function (newState, newValue) {

            // check 1: if we are changing to same state and report it
            if (this.myState == newState) {
                throw new Error("Sorry, But you can't do this to me! You are transitioning to same state: " + newState);
            }

            // check2: trying to get out of the fulfilled or rejected states
            if (this.myState == promState.fulfilled || this.myState == promState.rejected) {
                throw new Error("You can't leave this state now: " + this.myState);
            }
            // check 3: if promise is rejected with a null reason
            if (newState == promState.rejected && newValue === null) {
                throw new Error("If you get rejected there must be a reason. It can't be null!");
            }
            //check: 4 if there was no value passed with fulfilled
            if (newState == promState.fulfilled && arguments.length < 2) {
                throw new Error("I am sorry but you must have a non-null value to proceed to fulfilled!");
            }

            // we passed all the conditions, we can now change the state
            this.myState = newState;
            this.value = newValue;
            this.resolve();
            return this.myState;
        },
        fulfillPromise: function (value) {
            this.changeMyState(promState.fulfilled, value);
        },
        rejectPromise: function (reason) {
            this.changeMyState(promState.rejected, reason);
        },
        then: function (onFulfilled, onRejected) {
            // define an array named handlers
            this.handlers = this.handlers || [];
            // create a promise object
            var returnedPromise = Object.create(PromiseMe);
            var that = this;
            setTimeout(function () {
                that.handlers.push({
                    fulfillPromise: onFulfilled,
                    rejectPromise: onRejected,
                    promise: returnedPromise
                });
                that.resolve();
            }, 2);

            return returnedPromise;
        },
        resolve: function () {
            // check for pending and exist
            if (this.myState == promState.pending) {
                return false;
            }
            // loop through each then as long as handlers array contains items
            while (this.handlers && this.handlers.length) {
                //return and remove the first item in array
                var handler = this.handlers.shift();

                //set the function depending on the current state
                var doResolve = (this.myState == promState.fulfilled ? handler.fulfillPromise : handler.rejectPromise);
                //if doResolve is not a function
                if (typeof doResolve != 'function') {
                    handler.promise.changeMyState(this.myState, this.value);

                } else {
                    // fulfill the promise with value or reject with error
                    try {
                        var promiseValue = doResolve(this.value);

                        // deal with promise returned
                        if (promiseValue && typeof promiseValue.then == 'function') {
                            promiseValue.then(function (val) {
                                handler.promise.changeMyState(promState.fulfilled, val);
                            }, function (error) {
                                handler.promise.changeMyState(promState.rejected, error);
                            });
                            //if the value returned is not a promise
                        } else {
                            handler.promise.changeMyState(promState.fulfilled, promiseValue);
                        }
                        // deal with error thrown
                    } catch (error) {
                        handler.promise.changeMyState(promState.rejected, error);
                    }
                }
            }
        }
    };
    return Object.create(PromiseMe);
};
```

前面的代码代表了一个小型 JavaScript 库中的基本 promises 实现。它实现了一个具有`t0068en`方法的 promise 对象，考虑到如何根据规范要求解决和拒绝 promise，并在实现中进行必要的检查以避免异常。我们可以使用这个库并开始调用其`PromiseMe`对象及其相应的函数`then`、`fulfillPromise`和`rejectPromise`，以实现一些异步操作。

### 注意

这个实现是一个基本的实现；我们可以扩展它，包括许多可以在 Promises API 之上构建的功能和帮助方法。此外，我们可以构建这个实现，并将其与 Promises/A+兼容性测试套件进行测试，该测试套件可以通过这个链接找到：[`github.com/promises-aplus/promises-tests`](https://github.com/promises-aplus/promises-tests)。

在前面信息框中提供的链接中，我们可以找到完成测试所需的步骤，这些测试需要在 Node.js 环境中运行，我们需要确保 Node.js 已经安装。

## 将承诺付诸行动

我们可以将刚刚编写的这个 promise 的基本实现用于我们的代码中，来处理我们的异步操作。让我们来看一个如何使用这个`PromiseMe`库的例子。可以在`PromiseMe`对象的代码之后添加以下代码：

```js
var multiplyMeAsync = function (val) {
    var promise = new PromiseMe();
    promise.fulfillPromise(val * 2);

    return promise;
};
multiplyMeAsync(2)
    .then(function (value) {
    alert(value);
});
```

在上面的代码中，我们只是创建了一个名为`multiplyMeAsync`的函数，该函数进而将`PromiseMe`实例化给名为`promise`的变量，然后在我们创建的`PromiseMe`对象的`promise`变量上调用`fulfillPromise`方法。`fulfillPromise`方法所做的仅仅是将`val`参数乘以数字 2。随后，我们调用`multiplyAsync`，并将 2 作为其参数的值传递给它；由于它返回一个承诺，我们可以调用其`then`方法。`then`方法有一个单一的处理程序，处理成功并弹出一个带有现在应该为 4 的值的警告。

在 HTML 页面中运行脚本，我们应该会看到一个显示数字 4 的警告。

### 注意

您可以在 jsFiddle 中找到完整的代码并通过[`jsfiddle.net/RamiSarieddine/g8oj4guo/`](http://jsfiddle.net/RamiSarieddine/g8oj4guo/)进行测试。确保浏览器支持承诺。

让我们尝试给这段代码添加一些错误处理。首先，为了简单和可读性，我将创建一个名为`alertResult`的函数来替代`alert(value);`。

因此，我们将有一个如下所示的函数：

```js
var alertResult = function (value) {
    alert(value);
};
```

我们将添加另一个名为`onError`的函数，该函数基本上会带有传递给它的错误消息的警告。该函数将有以下语法：

```js
var onError = function(errorMsg) {
 alert(errorMsg);
};
```

现在，让我们添加一个包含错误处理的函数，通过检测异常并拒绝承诺来包含错误处理。下面的代码显示了这一点：

```js
var divideAsync = function (val) {
    var promise2 = new PromiseMe();
    if (val == 0) {
        promise2.rejectPromise("cannot divide by zero");
    }
    else{
        promise2.fulfillPromise(1 / val);
    }
    return promise2;
};
```

上一个函数所做的仅仅是检查值；如果值为零，函数拒绝承诺；否则，通过将数字 1 除以`val`来履行承诺。为了测试这个，我们将把值 0 传递给`multiplyAsync`，在其`then`调用中调用`divideAsync`，最后在`divideAsync`的`then`方法中调用一个错误函数。代码将如下所示：

```js
multiplyMeAsync(0)
    .then(divideAsync)
    .then(undefined, onError);
```

最终结果将是一个显示“不能除以零”的错误信息。这是因为零被传递给了`divideAsync`，它进而拒绝了承诺，并将错误信息传递给了`onError`处理器。

### 注意

您可以在以下 jsFiddle URL 上找到带有此错误处理场景的更新代码：

请参考以下链接：[`jsfiddle.net/RamiSarieddine/g8oj4guo/15/`](http://jsfiddle.net/RamiSarieddine/g8oj4guo/15/)

总之，承诺为异步操作的复杂性提供了一个非常好的解决方案。承诺提供的抽象让我们能更容易地做很多事情，尤其是使用回调的常见异步模式，具有以下主要特性：

+   一个承诺可以附加到多个回调函数上

+   值和错误会在承诺中传递

# 总结

在前五章中，我们已经学习了关于承诺（promises）的大量知识。我们从 JavaScript 的异步编程开始，了解了承诺在其中的地位，并详细讨论了为什么你应该关心承诺。接下来，我们深入探讨了 Promises API 及其`then`方法。然后，我们了解了目前支持承诺的浏览器和实现类似承诺特性的库。之后，我们覆盖了承诺的链式操作，并详细解释了如何实现以及如何使用承诺链来队列异步操作。第三个主题是错误处理，这是承诺概念中最重要的方面之一。我们退一步看了看 JavaScript 中的异常以及它们如何在承诺中得到处理。我们还学习了作为错误处理一部分的`catch`方法。

现在，承诺已经在 JavaScript 中有了原生支持，这是在网页和客户端开发世界中利用这项技术的焦点时刻。随着技术的发展，更多的浏览器将开始采用承诺作为标准，并在浏览器中实现原生支持。

如果你周围的人一直在谈论 JavaScript 的承诺，现在你知道原因了。这本书是你学习承诺的一个全面的参考点，它帮你节省了在网上找寻零散信息的麻烦。你可以立即开始实现这些学习内容。你随时可以回来查阅这本书，了解 API 的详细信息。从这里开始，你可以开始实现自己的承诺库，并利用其他可用的库，以及深入研究其他实现，如 Node.js。你还可以开始使用承诺来进行数据库、网络或文件服务器上的异步请求。

我希望您喜欢阅读这本书，并且它为您提供了正确的知识、工具和小贴士，以便将学习付诸实践，并开发出利用 JavaScript 承诺力量的一些绝佳应用程序。
