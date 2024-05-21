# 第二章：不可变性与可变性-安全与速度之间的平衡

近年来，开发实践已经转向更加功能化的编程风格。这意味着更少关注可变编程（在修改某些东西时改变变量而不是创建新变量）。当我们将变量从一种东西改变为另一种东西时，就会发生可变性。这可能是更新数字，改变消息内容，甚至将项目从字符串更改为数字。可变状态会导致编程陷阱的许多领域，例如不确定状态，在多线程环境中死锁，甚至在我们不希望的情况下更改数据类型（也称为副作用）。现在，我们有许多库和语言可以帮助我们遏制这种行为。

所有这些都导致了对使用不可变数据结构和基于输入创建新对象的函数的推动。虽然这会减少可变状态的错误，但它也带来了一系列其他问题，主要是更高的内存使用和更低的速度。大多数 JavaScript 运行时都没有优化，允许这种编程风格。当我们关注内存和速度时，我们需要尽可能多地获得优势，这就是可变编程给我们带来的优势。

在本章中，我们将重点关注以下主题：

+   当前网络上的不可变性趋势

+   编写安全的可变代码

+   网络上的类似功能的编程

# 技术要求

本章的先决条件如下：

+   一个网络浏览器，最好是 Chrome

+   编辑器；首选 VS Code

+   了解 Redux 或 Vuex 等当前状态库

+   相关代码可以在[`github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter02`](https://github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter02)找到。

# 当前对不可变性的迷恋

当前网络趋势显示了对利用不可变性的迷恋。诸如 React 之类的库可以在没有其不可变状态的情况下使用，但它们通常与 Redux 或 Facebook 的 Flow 库一起使用。这些库中的任何一个都将展示不可变性如何可以导致更安全的代码和更少的错误。

对于那些不了解的人，不可变性意味着一旦设置了数据，就无法更改变量。这意味着一旦我们给变量分配了某些内容，我们就不能再更改该变量。这有助于防止不必要的更改发生，并且还可以导致一个称为**纯函数**的概念。我们不会深入讨论纯函数是什么，但要知道这是许多函数式程序员一直在引入 JavaScript 的概念。

但是，这是否意味着我们需要它，它是否会导致更快的系统？在 JavaScript 的情况下，这可能取决于情况。一个管理良好的项目，有文档和测试，可以很容易地展示出我们可能不需要这些库。除此之外，我们可能需要实际改变对象的状态。我们可能在一个位置写入对象，但有许多其他部分从该对象中读取。

有许多开发模式可以给我们带来与不可变性相似的好处，而不需要创建大量临时对象或者甚至进入完全纯粹的功能化编程风格。我们可以利用诸如**资源获取即初始化**（**RAII**）的系统。我们可能会发现自己想要使用一些不可变性，在这种情况下，我们可以利用内置的浏览器工具，如`Object.freeze()`或`Object.seal()`。

然而，我们在走得太快了。让我们来看看其中提到的一些库，看看它们如何处理不可变状态，以及在编码时可能会导致问题。

# 深入 Redux

**Redux**是一个很好的状态管理系统。当我们开发诸如 Google Docs 或者一个报告系统这样的复杂系统时，它可以管理我们应用程序的状态。然而，它可能会导致一些过于复杂的系统，这些系统可能并不需要它所代表的状态管理。

Redux 的理念是没有一个对象应该能够改变应用程序的状态。所有的状态都需要托管在一个单一的位置，并且应该有处理状态变化的函数。这意味着写入的单一位置，以及多个位置能够读取数据。这与我们以后想要利用的一些概念类似。

然而，它会进一步进行许多文章都希望我们传回全新的对象。这是有原因的。许多对象，特别是那些具有多层的对象，不容易复制。简单的复制操作，比如使用`Object.assign({}, obj)`或者利用数组的扩展运算符，只会复制它们内部持有的引用。在我们编写基于 Redux 的应用程序之前，让我们看一个例子。

如果我们从我们的存储库中打开`not_deep_copy.html`，我们将看到控制台打印相同的内容。如果我们看一下代码，我们将看到一个非常常见的复制对象和数组的情况：

```js
const newObj = Object.assign({}, obj);
const newArr = [...arr];
```

如果我们只将其复制一层深，我们将看到它实际上执行了一次复制。以下代码将展示这一点：

```js
const obj2 = {item : 'thing', another : 'what'};
const arr2 = ['yes', 'no', 'nope'];

const newObj2 = Object.assign({}, obj2);
const newArr2 = [...arr2]
```

我们将更详细地讨论这个案例，以及如何真正执行深层复制，但我们可以开始看到 Redux 可能隐藏了仍然存在于我们系统中的问题。让我们构建一个简单的 Todo 应用程序，至少展示 Redux 及其能力。所以，让我们开始：

1.  首先，我们需要拉取 Redux。我们可以通过**Node Package Manager**（**npm**）来做到这一点，并在我们的系统中安装它。只需简单地`npm install redux`。

1.  我们现在将进入新创建的文件夹，获取`redux.min.js`文件并将其放入我们的工作目录中。

1.  现在我们将创建一个名为`todo_redux.html`的文件。这将包含我们的所有主要逻辑。

1.  在顶部，我们将将 Redux 库作为依赖项添加进来。

1.  然后，我们将添加我们要在存储库上执行的操作。

1.  接下来，我们将设置我们想要在应用程序中使用的 reducers。

1.  然后，我们将设置存储并准备好进行数据更改。

1.  然后我们将订阅这些数据变化并更新 UI。

我们正在处理的示例是 Redux 示例中 Todo 应用程序的略微修改版本。其中一个好处是我们将利用原始 DOM，而不是使用其他库，比如 React，所以我们可以看到 Redux 如何适用于任何应用程序，如果需要的话。

1.  所以，我们的操作将是添加一个`todo`元素，切换一个`todo`元素以完成或未完成，并设置我们想要看到的`todo`元素。这段代码如下所示：

```js
const addTodo = function(test) {
    return { type : ACTIONS.ADD_TODO, text };
}
const toggleTodo = function(index) {
    return { type : ACTIONS.TOGGLE_TODO, index };
}
const setVisibilityFilter = function(filter) {
    return { type : ACTIONS.SET_VISIBILITY_FILTER, filter };
}
```

1.  接下来，reducers 将被分开，一个用于我们的可见性过滤器，另一个用于实际的`todo`元素。

可见性 reducer 非常简单。它检查操作的类型，如果是`SET_VISIBILITY_FILTER`类型，我们将处理它，否则，我们只是传递状态对象。对于我们的`todo` reducer，如果我们看到一个`ADD_TODO`操作，我们将返回一个新的项目列表，其中我们的项目位于底部。如果我们切换其中一个项目，我们将返回一个将该项目设置为与其原来设置相反的新列表。否则，我们只是传递状态对象。所有这些看起来像下面这样：

```js
const visibilityFilter = function(state = 'SHOW_ALL', action) {
    switch(action.type) {
        case 'SET_VISIBILITY_FILTER': {
            return action.filter;
        }
        default: {
            return state;
        }
    }
}

const todo = function(state = [], action) {
    switch(action.type) {
        case 'ADD_TODO': {
            return [
                ...state,
                {
                    text : action.text,
                    completed : false
                }
        }
        case 'TOGGLE_TODO': {
            return state.map((todo, index) => {
                if( index === action.index ) {
                    return Object.assign({}, todo, {
                        completed : !todo.completed
                    });
                }
                return todo;
            }
        }
        default: {
            return state;
        }
    }
}
```

1.  完成后，我们将两个 reducer 放入一个单一的 reducer 中，并设置`state`对象。

我们逻辑的核心在于 UI 实现。请注意，我们设置了这个工作基于数据。这意味着数据可以传递到我们的函数中，UI 会相应地更新。我们也可以反过来，但是让 UI 由数据驱动是一个很好的范例。我们首先有一个先前的状态存储。我们可以进一步利用它，只更新实际更新的内容，但我们只在第一次检查时使用它。我们获取当前状态并检查两者之间的差异。如果我们看到长度已经改变，我们知道应该添加一个`todo`项目。如果我们看到可见性过滤器已更改，我们将相应地更新 UI。最后，如果这两者都不是真的，我们将检查哪个项目被选中或取消选中。代码如下所示：

```js
store.subscribe(() => 
    const state = store.getState();
    // first type of actions ADD_TODO
    if( prevState.todo.length !== state.todo.length ) {
     container.appendChild(createTodo(state.todo[state.todo.length
     - 1].text));
    // second type of action SET_VISIBILITY_FILTER
    } else if( prevState.visibilityFilter !== 
      state.visibilityFilter ) {
        setVisibility(container.children, state);
    // final type of action TOGGLE_TODO
    } else {
        const todos = container.children;
        for(let i = 0; i < todos.length; i++) {
            if( state.todo[i].completed ) {
                todos[i].classList.add('completed');
            } else {
                todos[i].classList.remove('completed');
            }
        }
    }
    prevState = state;
});
```

如果我们运行这个，我们应该得到一个简单的 UI，我们可以以以下方式进行交互：

+   添加`todo`项目。

+   将现有的`todo`项目标记为已完成。

我们还可以通过点击底部的三个按钮之一来查看不同的视图，如下面的截图所示。如果我们只想看到我们所有已完成的任务，我们可以点击“更新”按钮。

![](img/a46cdc61-ccb1-453c-a5e8-ed9318b16997.png)

现在，我们可以保存状态以进行离线存储，或者我们可以将状态发送回服务器进行常规更新。这就是使 Redux 非常好的地方。但是，在使用 Redux 时也有一些注意事项，与我们之前所述的相关：

1.  首先，我们需要在我们的 Todo 应用程序中添加一些内容，以便能够处理我们状态中的嵌套对象。这个 Todo 应用程序中遗漏的一部分信息是设置一个截止日期。因此，让我们添加一些字段供我们填写以设置完成日期。我们将添加三个新的数字输入，如下所示：

```js
<input id="year" type="number" placeholder="Year" />
<input id="month" type="number" placeholder="Month" />
<input id="day" type="number" placeholder="Day" />
```

1.  然后，我们将添加另一种`Overdue`的过滤器类型：

```js
<button id="SHOW_OVERDUE">Overdue</button>
```

1.  确保将其添加到`visibilityFilters`对象中。现在，我们需要更新我们的`addTodo`操作。我们还将传递一个`Date`对象。这也意味着我们需要更新我们的`ADD_TODO`情况，以将`action.date`添加到我们的新`todo`对象中。然后，我们将更新我们的 Add 按钮的`onclick`处理程序，并调整为以下内容：

```js
const year = document.getElementById('year');
const month = document.getElementById('month');
const day = document.getElementById('day');
store.dispatch(addTodo(input.value), {year : year.value, month : month.value, day : day.value}));
year.value = "";
month.value = "";
day.value = "";
```

1.  我们可以将日期保存为`Date`对象（这样更有意义），但为了展示可能出现的问题，我们将只保存一个带有`year`、`month`和`day`字段的新对象。然后，我们将通过添加另一个`span`元素并用这些字段的值填充它来在 Todo 应用程序上展示这个日期。最后，我们需要更新我们的`setVisibility`方法，以便显示我们过期的项目。它应该如下所示：

```js
case visibilityFilters.SHOW_OVERDUE: {
    const currTodo = state.todo[i];
    const tempTime = currTodo.date;
    const tempDate = new Date(`${tempTime.year}/${tempTime.month}/${tempTime.day}`);
    if( tempDate < currDay && !currTodo.completed ) {
        todos[i].classList.remove('hide');
    } else {
        todos[i].classList.add('hide');
    }
}
```

有了所有这些，我们现在应该有一个可工作的 Todo 应用程序，同时展示我们的过期项目。现在，这就是在处理 Redux 等状态管理系统时可能变得混乱的地方。当我们想要对已创建的项目进行修改，而它不是一个简单的扁平对象时会发生什么？好吧，我们可以只获取该项目并在状态系统中对其进行更新。让我们添加这段代码：

1.  首先，我们将创建一个新的按钮和输入，用于更改最后一个条目的年份。我们将为“更新”按钮添加一个点击处理程序：

```js
document.getElementById('UPDATE_LAST_YEAR').onclick = function(e) {
    store.dispatch({ type : ACTIONS.UPDATE_LAST_YEAR, year :  
     document.getElementById('updateYear').value });
}
```

1.  然后，我们将为`todo`系统添加这个新的操作处理程序：

```js
case 'UPDATE_LAST_YEAR': {
    const prevState = state;
    const tempObj = Object.assign({}, state[state.length - 
     1].date);
    tempObj.year = action.year;
    state[state.length - 1].date = tempObj;
    return state;
}
```

现在，如果我们运行我们的系统，我们会注意到一些情况。我们的代码在订阅中的检查对象条件中没有通过：

```js
if( prevState === state ) {
    return;
}
```

我们直接更新了状态，因此 Redux 从未创建新对象，因为它没有检测到更改（我们直接更新了一个对象的值，而我们没有一个 reducer）。现在，我们可以创建另一个专门用于日期的 reducer，但我们也可以重新创建数组并将其传递：

```js
case 'UPDATE_LAST_YEAR': {
    const prevState = state;
    const tempObj = Object.assign({}, state[state.length - 1].date);
    tempObj.year = action.year;
    state[state.length - 1].date = tempObj;
    return [...state];
}
```

现在，我们的系统检测到有变化，我们能够通过我们的方法来更新代码。

更好的实现方式是将我们的`todo` reducer 拆分为两个单独的 reducer。但是，由于我们正在进行示例，所以尽可能简单。

通过所有这些，我们可以看到我们需要遵守 Redux 为我们制定的规则。虽然这个工具在大规模应用中可能会带来巨大的好处，但对于较小的状态系统甚至组件化系统，我们可能会发现直接使用真正的可变状态更好。只要我们控制对可变状态的访问，我们就能充分利用可变状态的优势。

这并不是要贬低 Redux。它是一个很棒的库，即使在更重的负载下也能表现良好。但是，有时我们想直接使用数据集并直接进行变异。Redux 可以做到这一点，并为我们提供其事件系统，但是我们可以在不使用 Redux 提供的所有其他部分的情况下自己构建这个。记住，我们希望尽可能地精简代码库，并使其尽可能高效。当我们处理成千上万的数据项时，额外的方法和额外的调用会累积起来。

通过这个对 Redux 和状态管理系统的介绍，我们还应该看一下一个使不可变系统成为必需的库：Immutable.js。

# Immutable.js

再次利用不可变性，我们可以以更易于理解的方式编写代码。然而，这通常意味着我们无法满足真正高性能应用所需的规模。

首先，Immutable.js 在 JavaScript 中提供了一种很好的函数式数据结构和方法，这通常会导致更清晰的代码和更清晰的架构。但是，我们在这些优势方面得到的东西会导致速度的降低和/或内存的增加。

记住，当我们使用 JavaScript 时，我们处于一个单线程环境。这意味着我们实际上没有死锁、竞争条件或读/写访问问题。

在使用诸如`SharedArrayBuffers`之类的东西在工作线程或不同的标签之间可能会遇到这些问题，但这是以后章节的讨论。现在，我们正在一个单线程环境中工作，多核系统的问题并不会真正出现。

让我们举一个现实生活中可能出现的用例的例子。我们想将一个列表的列表转换为对象列表（想象一下 CSV）。在普通的 JavaScript 中构建这种数据结构的代码可能如下所示：

```js
const fArr = new Array(fillArr.length - 1);
const rowSize = fillArr[0].length;
const keys = new Array(rowSize);
for(let i = 0; i < rowSize; i++) {
    keys[i] = fillArr[0][i];
}
for(let i = 1; i < fillArr.length; i++) {
    const obj = {};
    for(let j = 0; j < rowSize; j++) {
        obj[keys[j]] = fillArr[i][j];
    }
    fArr[i - 1] = obj;
}
```

我们构建一个新的数组，大小为输入列表的大小减一（第一行是键）。然后，我们存储行大小，而不是每次在内部循环中计算。然后，我们创建另一个数组来保存键，并从输入数组的第一个索引中获取它们。接下来，我们循环遍历输入中的其余条目并创建对象。然后，我们循环遍历每个内部数组，并将键设置为值和位置`j`，并将值设置为输入的`i`和`j`值。

通过嵌套数组和循环读取数据可能会令人困惑，但可以获得快速的读取时间。在一个双核处理器和 8GB RAM 的计算机上，这段代码花了 83 毫秒。

现在，让我们在 Immutable.js 中构建类似的东西。它应该看起来像下面这样：

```js
const l = Immutable.List(fillArr);
const _k = Immutable.List(fillArr[0]);
const tFinal = l.map((val, index) => {
    if(!index ) return;
    return Immutable.Map(_k.zip(val));
});
const final = tfinal.shift();
```

如果我们理解函数式概念，这将更容易解释。首先，我们想要根据我们的输入创建一个列表。然后我们创建另一个临时列表用于存储键称为`_k`*.*。对于我们的临时最终列表，我们利用`map`函数。如果我们在`0`索引处，我们就从函数中`return`（因为这是键）。否则，我们返回一个通过将键列表与当前值进行 zip 的新映射。最后，我们移除最终列表的前部，因为它将是未定义的。

这段代码在可读性方面很棒，但它的性能特征如何？在当前的机器上，它运行大约需要 1 秒。这在速度方面有很大的差异。让我们看看它们在内存使用方面的比较。

已解决的内存（运行代码后内存返回的状态）似乎是相同的，回到了大约 1.2 MB。然而，不可变版本的峰值内存约为 110 MB，而 Vanilla JavaScript 版本只达到了 48 MB，所以内存使用量略低于一半。让我们看另一个例子并看看发生的结果。

我们将创建一个值数组，除了我们希望其中一个值是不正确的。因此，我们将使用以下代码将第 50,000 个索引设置为`wrong`：

```js
const tempArr = new Array(100000);
for(let i = 0; i < tempArr.length; i++) {
    if( i === 50000 ) { tempArr[i] = 'wrong'; }
    else { tempArr[i] = i; }
}
```

然后，我们将使用简单的`for`循环遍历一个新数组，如下所示：

```js
const mutArr = Array.apply([], tempArr);
const errs = [];
for(let i = 0; i < mutArr.length; i++) {
    if( mutArr[i] !== i ) {
        errs.push(`Error at loc ${i}. Value : ${mutArr[i]}`);
        mutArr[i] = i;
    }
}
```

我们还将测试内置的`map`函数：

```js
const mut2Arr = Array.apply([], tempArr);
const errs2 = [];
const fArr = mut2Arr.map((val, index) => {
    if( val !== index ) {
        errs2.push(`Error at loc: ${index}. Value : ${val}`);
        return index;
    }
    return val;
});
```

最后，这是不可变版本：

```js
const immArr = Immutable.List(tempArr);
const ierrs = [];
const corrArr = immArr.map((item, index) => {
    if( item !== index ) {
        ierrs.push(`Error at loc ${index}. Value : ${item}`);
        return index;
    }
    return item;
});
```

如果我们运行这些实例，我们会发现最快的将在基本的`for`循环和内置的`map`函数之间切换。不可变版本仍然比其他版本慢 8 倍。当我们增加不正确值的数量时会发生什么？让我们添加一个随机数生成器来构建我们的临时数组，以便产生随机数量的错误，并看看它们的表现。代码应该如下所示：

```js
for(let i = 0; i < tempArr.length; i++) {
    if( Math.random() < 0.4 ) {
        tempArr[i] = 'wrong';
    } else {
        tempArr[i] = i;
    }
}
```

运行相同的测试，我们发现不可变版本大约会慢十倍。现在，这并不是说不可变版本在某些情况下不会运行得更快，因为我们只涉及了它的 map 和 list 功能，但这确实提出了一个观点，即在将其应用于 JavaScript 库时，不可变性在内存和速度方面是有代价的。

我们将在下一节中看到为什么可变性可能会导致一些问题，但也会看到我们如何通过利用类似 Redux 处理数据的想法来处理它。

不同的库总是有其适用的时间和场合，并不是说 Immutable.js 或类似的库是不好的。如果我们发现我们的数据集很小或其他考虑因素起作用，Immutable.js 可能适合我们。但是，当我们在高性能应用程序上工作时，这通常意味着两件事。一是我们将一次性获得大量数据，二是我们将获得大量导致数据积累的事件。我们需要尽可能使用最有效的方法，而这些通常内置在我们正在使用的运行时中。

# 编写安全的可变代码

在我们继续编写安全的可变代码之前，我们需要讨论引用和值。值可以被认为是任何原始类型。在 JavaScript 中，原始类型是指不被视为对象的任何内容。简单来说，数字、字符串、布尔值、null 和 undefined 都是值。这意味着如果你创建一个新变量并将其分配给原始变量，它实际上会给它一个新值。那么这对我们的代码意味着什么呢？嗯，我们之前在 Redux 中看到，它无法看到我们更新了状态系统中的属性，因此我们的先前状态和当前状态显示它们是相同的。这是由于浅相等测试。这个基本测试测试传入的两个变量是否指向同一个对象。一个简单的例子是在以下代码中看到的：

```js
let x = {};
let y = x;
console.log( x === y );
y = Object.assign({}, x);
console.log( x === y );
```

我们会发现第一个版本说这两个项目是相等的。但是，当我们创建对象的副本时，它会声明它们不相等。`y`现在有一个全新的对象，这意味着它指向内存中的一个新位置。虽然对*按值传递*和*按引用传递*的更深入理解可能有好处，但这应该足以继续使用可变代码。

在编写安全的可变代码时，我们希望给人一种错觉，即我们正在编写不可变的代码。换句话说，接口应该看起来像我们在使用不可变的系统，但实际上我们在内部使用的是可变的系统。因此，接口与实现之间存在分离。

我们可以通过以可变的方式编写代码来使实现变得非常快速，但提供一个看起来不可变的接口。一个例子如下：

```js
Array.prototype._map = function(fun) {
    if( typeof fun !== 'function' ) {
        return null;
    }
    const arr = new Array(this.length);
    for(let i = 0; i < this.length; i++) {
        arr[i] = fun(this[i]);
    }
    return arr;
}
```

我们在数组原型上编写了一个`_map`函数，以便每个数组都可以使用它，并且我们编写了一个简单的`map`函数。如果我们现在测试运行这段代码，我们会发现一些浏览器使用这种方式更快，而其他浏览器使用内置选项更快。如前所述，内置选项最终会变得更快，但往往一个简单的循环会更快。现在让我们看另一个可变实现的例子，但具有不可变的接口：

```js
Array.prototype._reduce = function(fun, initial=null) {
    if( typeof fun !== 'function' ) {
        return null;
    }
    let val = initial ? initial : this[0];
    const startIndex = initial ? 0 : 1;
    for(let i = startIndex; i < this.length; i++) {
        val = fun(val, this[i], i, this);
    }
    return val;
}
```

我们编写了一个`reduce`函数，在每个浏览器中都有更好的性能。现在，它没有相同数量的类型检查，这可能会导致更好的性能，但它展示了我们如何编写可以提供更好性能但给用户提供相同类型接口的函数。

到目前为止，我们讨论的是，如果我们为别人编写一个库来使他们的生活更轻松。如果我们正在编写一些我们自己或内部团队将要使用的东西，这是大多数应用程序开发人员的情况，会发生什么呢？

在这种情况下，我们有两个选择。首先，我们可能会发现我们正在处理一个传统系统，并且我们将不得不尝试以与已有代码类似的风格进行编程，或者我们正在开发一些全新的东西，我们可以从头开始。

编写传统代码是一项艰巨的工作，大多数人通常会做错。虽然我们应该致力于改进代码库，但我们也在努力匹配风格。对于开发人员来说，尤其困难的是，他们需要浏览代码并看到使用了 10 种不同代码选择，因为在项目的整个生命周期中有 10 个不同的开发人员参与其中。如果我们正在处理其他人编写的东西，通常最好匹配代码风格，而不是提出完全不同的东西。

有了一个新系统，我们可以按照自己的意愿编写代码，并且在适当的文档支持下，我们可以编写出非常快速的代码，同时也容易让其他人理解。在这种情况下，我们可以编写可变的代码，函数中可能会产生副作用，但我们可以记录这些情况。

副作用是指当一个函数不仅返回一个新变量或者变量的引用时发生的情况。当我们更新另一个变量，而我们对其没有当前范围时，这构成了一个副作用。一个例子如下：

```js
var glob = 'a single point system';
const implement = function(x) {
    glob = glob.concat(' more');
    return x += 2;
}
```

我们有一个名为`glob`的全局变量，我们在函数内部对其进行更改。从技术上讲，这个函数对`glob`有范围，但我们应该尝试将实现的范围定义为仅限于传入的内容以及实现内部定义的临时变量。由于我们正在改变`glob`，我们在代码库中引入了一个副作用。

现在，在某些情况下，副作用是必需的。我们可能需要更新一个单一点，或者我们可能需要将某些东西存储在一个单一位置，但我们应该尝试实现一个接口来为我们完成这些操作，而不是直接影响全局项目（这听起来很像 Redux）。通过编写一个或两个函数来影响超出范围的项目，我们现在可以诊断问题可能出现的地方，因为我们有这些单一的入口点。

那么这会是什么样子呢？我们可以创建一个状态对象，就像一个普通的对象一样。然后，我们可以在全局范围内编写一个名为`updateState`的函数，如下所示：

```js
const updateState = function(update) {
    const x = Object.keys(update);
    for(let i = 0; i < x.length; i++) {
        state[x[i]] = update[x[i]];
    }
}
```

现在，虽然这可能是好的，但我们仍然容易受到通过实际全局属性更新我们的状态对象的影响。幸运的是，通过将我们的状态对象和函数设为`const`，我们可以确保错误的代码无法触及这些实际的名称。让我们更新我们的代码，以确保我们的状态受到直接更新的保护。我们可以通过两种方式来实现这一点。第一种方法是使用模块编码，然后我们的状态对象将被限定在该模块中。我们将在本书中进一步讨论模块和导入语法。在这种情况下，我们将使用第二种方法，即**立即调用函数表达式**（**IIFE**）的方式进行编码。以下展示了这种实现方式：

```js
const state = {};
(function(scope) {
    const _state = {};
    scope.update = function(obj) {
        const x = Object.keys(obj);
        for(let i = 0; i < x.length; i++) {
            _state[x[i]] = obj[x[i]];
        }
    }
    scope.set = function(key, val) {
        _state[key] = val;
    }
    scope.get = function(key) {
        return _state[key];
    }
    scope.getAll = function() {
        return Object.assign({}, _state);
    }
})(state);
Object.freeze(state);
```

首先，我们创建一个常量状态。然后我们使用 IIFE 并传入状态对象，在其上设置一堆函数。它在一个内部`scoped _state`变量上工作。我们还拥有所有我们期望的内部状态系统的基本函数。我们还冻结了外部状态对象，因此它不再能被操纵。可能会出现的一个问题是，为什么我们要返回一个新对象而不是一个引用。如果我们试图确保没有人能够触及内部状态，那么我们不能传递一个引用出去；我们必须传递一个新对象。

我们仍然有一个问题。如果我们想要更新多层深度，会发生什么？我们将再次遇到引用问题。这意味着我们需要更新我们的更新函数以执行深度更新。我们可以用多种方式来做到这一点，但一种方法是将值作为字符串传递，然后在小数点上分割。

这并不是处理这个问题的最佳方式，因为我们在技术上可以让对象的属性以小数点命名，但这将允许我们快速编写一些东西。在编写高性能代码库时，平衡编写功能性代码和被认为是完整解决方案的东西之间的平衡是两回事，必须在写作时加以平衡。

因此，我们现在将有一个如下所示的方法：

```js
const getNestedProperty = function(key) {
    const tempArr = key.split('.');
    let temp = _state;
    while( tempArr.length > 1 ) {
        temp = temp[tempArr.shift()];
        if( temp === undefined ) {
            throw new Error('Unable to find key!');
        }
    }
    return {obj : temp, finalKey : tempArr[0] };
}
scope.set = function(key, val) {
    const {obj, finalKey} = getNestedProperty(key);
    obj[finalKey] = val;
}
scope.get = function(key) {
    const {obj, finalKey} = getNestedProperty(key);
    return obj[finalKey];
}
```

我们正在通过小数点来分解键。我们还获取了对内部状态对象的引用。当列表中仍有项目时，我们会在对象中向下移动一级。如果我们发现它是未定义的，那么我们将抛出一个错误。否则，一旦我们在我们想要的位置的上一级，我们将返回一个具有该引用和最终键的对象。然后我们将在 getter 和 setter 中使用这个对象来替换这些值。

现在，我们仍然有一个问题。如果我们想要使引用类型成为内部状态系统的属性值，会怎么样呢？嗯，我们将遇到之前看到的相同问题。我们将在单个状态对象之外有引用。这意味着我们将不得不克隆每一步，以确保外部引用不指向内部副本中的任何内容。我们可以通过添加一堆检查并确保当我们到达引用类型时，以一种高效的方式进行克隆来创建这个系统。代码如下所示：

```js
const _state = {},
checkPrimitives = function(item) {
    return item === null || typeof item === 'boolean' || typeof item === 
     'string' || typeof item === 'number' || typeof item === 'undefined';
},
cloneFunction = function(fun, scope=null) {
    return fun.bind(scope);
},
cloneObject = function(obj) {
    const newObj = {};
    const keys = Object.keys(obj);
    for(let i = 0; i < keys.length; i++) {
        const key = keys[i];
        const item = obj[key];
        newObj[key] = runUpdate(item);
    }
    return newObj;
},
cloneArray = function(arr) {
    const newArr = new Array(arr.length);
    for(let i = 0; i < arr.length; i++) {
        newArr[i] = runUpdate(arr[i]);
    }
    return newArr;
},
runUpdate = function(item) {
    return checkPrimitives(item) ?
        item : 
        typeof item === 'function' ?
            cloneFunction(item) :
        Array.isArray(item) ?
            cloneArray(item) :
            cloneObject(item);
};

scope.update = function(obj) {
    const x = Object.keys(obj);
    for(let i = 0; i < x.length; i++) {
        _state[x[i]] = runUpdate(obj[x[i]]);
    }
}
```

我们所做的是编写一个简单的克隆系统。我们的`update`函数将遍历键并运行更新。然后我们将检查各种条件，比如我们是否是原始类型。如果是，我们只需复制值，否则，我们需要弄清楚我们是什么复杂类型。我们首先搜索是否是一个函数；如果是，我们只需绑定值。如果是一个数组，我们将遍历所有的值，并确保它们都不是复杂类型。最后，如果是一个对象，我们将遍历所有的键，并尝试运行相同的检查来更新这些键。

然而，我们刚刚做了我们一直在避免的事情；我们已经创建了一个不可变的状态系统。我们可以为这个集中的状态系统添加更多的功能，比如事件，或者我们可以实现一个已经存在很长时间的编码标准，称为**Resource Allocation Is Initialization**（**RAII**）。

有一个名为**proxies**的内置 Web API 非常好。这些基本上是系统，我们能够在对象发生某些事情时执行某些操作。在撰写本文时，这些仍然相当慢，除非是在我们不担心时间敏感的对象上，否则不应该真正使用它们。我们不打算对它们进行详细讨论，但对于那些想要了解它们的读者来说，它们是可用的。

# 资源分配即初始化（RAII）

RAII 的概念来自 C++，在那里我们没有内存管理器。我们封装逻辑，可能希望共享需要在使用后释放的资源。这确保我们没有内存泄漏，并且正在使用该项的对象是以安全的方式进行的。这个概念的另一个名称是**scope-bound resource management**（**SBRM**），也在另一种最近的语言 Rust 中使用。

我们可以在 JavaScript 代码中应用与 C++和 Rust 相同类型的 RAII 思想。我们可以处理这个问题的几种方法，我们将对它们进行讨论。第一种方法是，当我们将一个对象传递给一个函数时，我们可以从调用函数中将该对象`null`掉。

现在，我们将不得不在大多数情况下使用`let`而不是`const`，但这是一种有用的范式，可以确保我们只保留我们需要的对象。

这个概念可以在以下代码中看到：

```js
const getData = function() {
    return document.getElementById('container').value;
};
const encodeData = function(data) {
    let te = new TextEncoder();
    return te.encode(data);
};
const hashData = function(algorithm) {
    let str = getData();
    let finData = encodeData(str);
    str = null;
    return crypto.subtle.digest(algorithm, finData);
};
{
    let but = document.getElementById('submit');
    but.onclick = function(ev) {
        let algos = ['SHA-1', 'SHA-256', 'SHA-384', 'SHA-512'];
        let out = document.getElementById('output');
        for(let i = 0; i < algos.length; i++) {
            const newEl = document.createElement('li');
            hashData(algos[i]).then((res) => {
                let te = new TextDecoder();
                newEl.textContent = te.decode(res);
                out.append(newEl);
            });
        }
        out = null;
    }
    but = null;
}
```

如果我们运行以下代码，我们会注意到我们正在尝试追加到一个`null`。这就是这种设计可能会让我们陷入麻烦的地方。我们有一个异步方法，我们正在尝试使用一个我们已经使无效的值，尽管我们仍然需要它。处理这种情况的最佳方法是什么？一种方法是在使用完毕后将其`null`掉。因此，我们可以将代码更改为以下内容：

```js
for(let i = 0; i < algos.length; i++) {
    let temp = out;
    const newEl = document.createElement('li');
    hashData(algos[i]).then((res) => {
        let te = new TextDecoder();
        newEl.textContent = te.decode(res);
        temp.append(newEl);
        temp = null
    });
}
```

我们仍然有一个问题。在`Promise`的下一部分（`then`方法）运行之前，我们仍然可以修改值。一个最后的好主意是将此输入输出包装在一个新函数中。这将给我们所寻找的安全性，同时也确保我们遵循 RAII 背后的原则。以下代码是由此产生的：

```js
const showHashData = function(parent, algorithm) {
    const newEl = document.createElement('li');
    hashData(algorithm).then((res) => {
        let te = new TextDecoder();
        newEl.textContent = te.decode(res);
        parent.append(newEl);
    });
}
```

我们还可以摆脱一些之前的 null，因为函数将处理这些临时变量。虽然这个例子相当琐碎，但它展示了在 JavaScript 中处理 RAII 的一种方式。

除此范式之外，我们还可以向传递的项目添加属性，以表明它是只读版本。这将确保我们不会修改该项目，但如果我们仍然想从中读取，我们也不需要在调用函数上将元素`null`掉。这使我们能够确保我们的对象可以被利用和维护，而不必担心它们会被修改。

我们将删除以前的代码示例，并更新它以利用这个只读属性。我们首先定义一个函数，将其添加到任何传入的对象中，如下所示：

```js
const addReadableProperty = function(item) {
    Object.defineProperty(item, 'readonly', {
        value : true,
        writable :false
    });
    return item;
}
```

接下来，在我们的`onclick`方法中，我们将输出传递给此方法。现在，它已经附加了`readonly`属性。最后，在我们的`showHashData`函数中，当我们尝试访问它时，我们已经在`readonly`属性上设置了保护。如果我们注意到对象具有它，我们将不会尝试追加到它，就像这样：

```js
if(!parent.readonly ) {
    parent.append(newEl);
}
```

我们还将此属性设置为不可写，因此如果一个恶意的行为者决定操纵我们对象的`readonly`属性，他们仍然会注意到我们不再向 DOM 追加内容。`defineProperty`方法非常适用于编写无法轻易操纵的 API 和库。另一种处理方法是冻结对象。使用`freeze`方法，我们可以确保对象的浅拷贝是只读的。请记住，这仅适用于浅实例，而不适用于持有引用类型的任何其他属性。

最后，我们可以利用计数器来查看是否可以设置数据。我们基本上正在创建一个读取端锁。这意味着在读取数据时，我们不希望设置数据。这意味着我们必须采取许多预防措施，以确保我们在读取所需内容后正确释放数据。这可能看起来像下面这样：

```js
const ReaderWriter = function() {
    let data = {};
    let readers = 0;
    let readyForSet = new CustomEvent('readydata');
    this.getData = function() {
        readers += 1;
        return data;
    }
    this.releaseData = function() {
        if( readers ) {
            readers -= 1;
            if(!readers ) {
                document.dispatchEvent(readyForSet);
            }
        }
        return readers;
    }
    this.setData = function(d) {
        return new Promise((resolve, reject) => {
            if(!readers ) {
                data = d;
                resolve(true);
            } else {
                document.addEventListener('readydata', function(e) {
                    data = d;
                    resolve(true);
                }, { once : true });
            }
        });
    }
}
```

我们所做的是设置一个构造函数。我们将数据、读者数量和自定义事件作为私有变量保存。然后创建三种方法。首先，`getData`将获取数据，并为使用它的人添加一个计数器。接下来是`release`方法。这将递减计数器，如果计数器为 0，我们将触发一个事件，告诉`setData`事件可以最终写入可变状态。最后是`setData`函数。返回值将是一个 promise。如果没有人持有数据，我们将立即设置并解析它。否则，我们将为我们的自定义事件设置一个事件监听器。一旦触发，我们将设置数据并解析 promise。

现在，这种锁定可变数据的最终方法不应该在大多数情况下使用。可能只有少数情况下你会想要使用它，比如热缓存，我们需要确保在读者从中读取时不要覆盖某些东西（这在 Node.js 方面尤其可能发生）。

所有这些方法都有助于创建一个安全的可变状态。通过这些方法，我们能够直接改变对象并共享内存空间。大多数情况下，良好的文档和对数据的谨慎控制将使我们不需要采取我们在这里所做的极端措施，但是当我们发现某些问题出现并且我们正在改变不应该改变的东西时，拥有这些 RAII 方法是很好的。

大多数情况下，不可变和高度函数式的代码最终会更易读，如果某些东西不需要高度优化，建议以易读性为重。但是，在高度优化的情况下，例如编码和解码或装饰表中的列，我们需要尽可能地提高性能。这将在本书的后面部分看到，我们将利用各种编程技术的混合。

尽管可变编程可能很快，但有时我们希望以函数方式实现事物。接下来的部分将探讨以这种函数方式实现程序的方法。

# 函数式编程风格

即使我们谈论了关于函数概念在原始速度方面不是最佳的，但在 JavaScript 中利用它们仍然可能非常有帮助。有许多语言不是纯函数式的，所有这些语言都给了我们利用许多范式的最佳思想的能力。例如 F#和 Scala 等语言。在这种编程风格方面有一些很棒的想法，我们可以利用 JavaScript 中的内置概念。

# 惰性评估

在 JavaScript 中，我们可以进行所谓的惰性评估。惰性评估意味着程序不运行不需要的部分。一个思考这个问题的方式是，当有人得到一个问题的答案列表，并被告知把正确的答案放在问题的答案列表中。如果他们发现答案是他们查看的第二个项目，他们就不会继续查看他们得到的其他答案；他们会在第二个项目处停下来。我们在 JavaScript 中使用惰性评估的方式是使用生成器。

生成器是一种函数，它会暂停执行，直到在它们上调用`next`方法。一个简单的例子如下所示：

```js
const simpleGenerator = function*() {
    let it = 0;
    for(;;) {
        yield it;
        it++;
    }
}

const sg = simpleGenerator();
for(let i = 0; i < 10; i++) {
    console.log(sg.next().value);
}
sg.return();
console.log(sg.next().value);
```

首先，我们注意到`function`旁边有一个星号。这表明这是一个生成器函数。接下来，我们设置一个简单的变量来保存我们的值，然后我们有一个无限循环。有些人可能会认为这将持续运行，但惰性评估表明我们只会运行到`yield`。这个`yield`意味着我们将在这里暂停执行，并且我们可以获取我们发送回来的值。

所以，我们启动函数。我们没有什么要传递给它，所以我们只是简单地启动它。接下来，我们在生成器上调用`next`并获取值。这给了我们一个单独的迭代，并返回`yield`语句上的任何内容。最后，我们调用`return`来表示我们已经完成了这个生成器。如果我们愿意，我们可以在这里获取最终值。

现在，我们会注意到当我们调用`next`并尝试获取值时，它返回了 undefined。我们可以看一下生成器，并注意到它有一个叫做`done`的属性。这可以让我们看到有限生成器是否已经完成。那么当我们想要做一些事情时，这怎么会有帮助呢？一个相当琐碎的例子是一个计时函数。我们将在想要计时的东西之前启动计时器，然后我们将再次调用它来计算某个东西运行所花费的时间（与`console.time`和`timeEnd`非常相似，但它应该展示了生成器的可用性）。

这个生成器可能看起来像下面这样：

```js
const timing = function*(time) {
    yeild Date.now() - time;
}
const time = timing(Date.now());
let sum = 0;
for(let i = 0; i < 1000000; i++) {
    sum = sum + i;
}
console.log(time.next().value);
```

我们现在正在计时一个简单的求和函数。所有这个函数做的就是用当前时间初始化计时生成器。一旦调用下一个函数，它就会运行到`yield`语句并返回`yield`中保存的值。这将给我们一个新的时间与我们传入的时间进行比较。现在我们有了一个用于计时的简单函数。这对于我们可能无法访问控制台并且需要在其他地方记录这些信息的环境特别有用。

就像前面的代码块所示，我们也可以使用许多不同类型的惰性加载。其中利用这个接口的最好类型之一是流。流在 Node.js 中已经有一段时间了，但是浏览器的流接口有一个基本的标准化，某些部分仍在讨论中。这种类型的惰性加载或惰性读取的一个简单例子可以在下面的代码中看到：

```js
const nums = function*(fn=null) {
    let i = 0;
    for(;;) {
        yield i;
        if( fn ) {
            i += fn(i);
        } else {
            i += 1;
        }
    }
}
const data = {};
const gen = nums();
for(let i of gen) {
    console.log(i);
    if( i > 100 ) {
        break;
    }
    data.push(i);
}

const fakestream = function*(data) {
    const chunkSize = 10;
    const dataLength = data.length;
    let i = 0;
    while( i < dataLength) {
        const outData = [];
        for(let j = 0; j < chunkSize; j++) {
            outData.push(data[i]);
            i+=1;
        }
        yield outData;
    }
}

for(let i of fakestream(data)) {
    console.log(i);
}
```

这个例子展示了惰性评估的概念，以及我们将在后面章节中看到的流的一些概念。首先，我们创建一个生成器，它可以接受一个函数，并可以利用它来创建我们的逻辑函数中的数字。在我们的例子中，我们只会使用默认情况，并且让它一次生成一个数字。接下来，我们将通过`for/of`循环运行这个生成器，以生成 101 个数字。

接下来，我们创建一个`fakestream`生成器，它将为我们分块数据。这类似于流，允许我们一次处理一块数据。我们可以对这些数据进行转换（称为`TransformStream`），或者只是让它通过（称为`PassThrough`的一种特殊类型的`TransformStream`）。我们在`10`处创建一个假的块大小。然后我们再次对之前的数据运行另一个`for/of`循环，并简单地记录它。但是，如果我们愿意，我们也可以对这些数据做些什么。

这不是流使用的确切接口，但它展示了我们如何在生成器中实现惰性求值，并且这也内置在某些概念中，比如流。生成器和惰性求值技术还有许多其他潜在的用途，这里不会涉及，但对于寻求更功能式风格的列表和映射理解的开发人员来说，它们是可用的。

# 尾递归优化

这是许多功能性语言具有的另一个概念，但大多数 JavaScript 引擎没有（WebKit 是个例外）。尾递归优化允许以一定方式构建的递归函数运行得就像一个简单的循环一样。在纯函数语言中，没有循环这样的东西，所以处理集合的唯一方法是通过递归进行。我们可以看到，如果我们将一个函数构建为尾递归函数，它将破坏我们的堆栈。以下代码说明了这一点：

```js
const _d = new Array(100000);
for(let i = 0; i < _d.length; i++) {
    _d[i] = i;
}
const recurseSummer = function(data, sum=0) {
    if(!data.length ) {
        return sum;
    }
    return recurseSummer(data.slice(1), sum + data[0]);
}
console.log(recurseSummer(_d));
```

我们创建了一个包含 100,000 个项目的数组，并为它们分配了它们索引处的值。然后我们尝试使用递归函数来对数组中的所有数据进行求和。由于函数的最后一次调用是函数本身，一些编译器能够在这里进行优化。如果它们注意到最后一次调用是对同一个函数的调用，它们知道当前的堆栈可以被销毁（函数没有剩余工作要做）。然而，非优化的编译器（大多数 JavaScript 引擎）不会进行这种优化，因此我们不断向我们的调用系统添加堆栈。这导致调用堆栈大小超出限制，并使我们无法利用这个纯粹的功能概念。

然而，JavaScript 还是有希望的。一个叫做 trampolining 的概念可以通过修改函数和我们调用它的方式来实现尾递归。以下是修改后的代码，以利用 trampolining 并得到我们想要的结果：

```js
const trampoline = (fun) => {
    return (...arguments) => {
        let result = fun(...arguments);
        while( typeof result === 'function' ) {
            result = result();
        }
        return result;
    }
}

const _d = new Array(100000);
for(let i = 0; i < _d.length; i++) {
    _d[i] = i;
}
const recurseSummer = function(data, sum=0) {
    if(!data.length ) {
        return sum;
    }
    return () => recurseSummer(data.slice(1), sum + data[0]);
}
const final = trampoline(recurseSummer);
console.log(final(_d));
```

我们所做的是将我们的递归函数包装在一个我们通过简单循环运行的函数中。`trampoline`函数的工作方式如下：

+   它接受一个函数，并返回一个新构造的函数，该函数将运行我们的递归函数，但通过循环进行检查返回类型。

+   在这个内部函数中，它通过执行函数的第一次运行来启动循环。

+   当我们仍然将一个函数作为我们的返回类型时，它将继续循环。

+   一旦我们最终得到了一个函数，我们将返回结果。

现在我们能够利用尾递归来做一些在纯粹的功能世界中会做的事情。之前看到的一个例子（可以看作是一个简单的 reduce 函数）是一个例子。

```js
const recurseFilter = function(data, con, filtered=[]) {
    if(!data.length ) {
        return filtered;
    }
    return () => recurseFilter(data.slice(1), con, con(data[0]) ? 
     filtered.length ? new Array(...filtered), data[0]) : [data[0]] : filtered);

const finalFilter = trampoline(recurseFilter);
console.log(finalFilter(_d, item => item % 2 === 0));
```

通过这个函数，我们模拟了纯函数语言中基于过滤的操作可能是什么样子。同样，如果没有长度，我们就到达了数组的末尾，并返回我们过滤后的数组。否则，我们返回一个新函数，该函数用一个新列表、我们要进行过滤的函数以及过滤后的列表递归调用自身。这里有一些奇怪的语法。如果我们有一个空列表，我们必须返回一个带有新项的单个数组，否则，它将给我们一个包含我们传入的项目数量的空数组。

我们可以看到，这两个函数都通过了尾递归的检查，并且也是纯函数语言中可以编写的函数。但是，我们也会看到，这些函数运行起来比简单的`for`循环甚至这些类型函数的内置数组方法要慢得多。归根结底，如果我们想要使用尾递归来编写纯粹的函数式编程，我们可以，但在 JavaScript 中这样做是不明智的。

# 柯里化

我们将要看的最后一个概念是柯里化。柯里化是一个接受多个参数的函数实际上是一系列接受单个参数并返回另一个函数或最终值的函数。让我们看一个简单的例子来看看这个概念是如何运作的：

```js
const add = function(a) {
    return function(b) {
        return a + b;
    }
}
```

我们正在接受一个接受多个参数的函数，比如`add`函数。然后我们返回一个接受单个参数的函数，这里是`b`。这个函数然后将数字`a`和`b`相加。这使我们能够像通常一样使用函数（除了我们运行返回给我们的函数并传入第二个参数），或者我们得到运行它的返回值，并使用该函数来添加接下来的任何值。这些概念中的每一个都可以在以下代码中看到：

```js
console.log(add(2)(5), 'this will be 7');
const add5 = add(5);
console.log(add5(5), 'this will be 10');
```

柯里化有一些用途，它们也展示了一个可以经常使用的概念。首先，它展示了部分应用的概念。这样做是为我们设置一些参数并返回一个函数。然后我们可以将这个函数传递到语句链中，并最终用它来填充剩下的函数。

只需记住，所有柯里化函数都是部分应用函数，但并非所有部分应用函数都是柯里化函数。

部分应用的示例可以在以下代码中看到：

```js
const fullFun = function(a, b, c) {
    console.log('a', a);
    console.log('b', b);
    console.log('c', c);
}
const tempFun = fullFun.bind(null, 2);
setTimeout(() => {
    const temp2Fun = tempFun.bind(null, 3);
    setTimeout(() => {
        const temp3Fun = temp2Fun.bind(null, 5);
        setTimeout() => {
            console.log('temp3Fun');
            temp3Fun();
        }, 1000);
    }, 1000);
    console.log('temp2Fun');
    temp2Fun(5);
}, 1000);
console.log('tempFun');
tempFun(3, 5);
```

首先，我们创建一个接受三个参数的函数。然后我们创建一个新的临时函数，将`2`绑定到该函数的第一个参数。`Bind`是一个有趣的函数。它将我们想要的作用域作为第一个参数（`this`指向的内容），然后接受任意长度的参数来填充我们正在处理的函数的参数。在我们的例子中，我们只将第一个变量绑定到数字`2`。然后我们创建一个第二个临时函数，其中我们将第一个临时函数的第一个变量绑定到`3`。最后，我们创建一个第三个临时函数，其中我们将第二个函数的第一个参数绑定到数字`5`。

我们可以在每次运行时看到，我们能够运行这些函数中的每一个，并且它们根据我们使用的函数版本不同而接受不同数量的参数。`bind`是一个非常强大的工具，它允许我们传递函数，这些函数可能会在最终使用函数之前从其他函数中获取参数填充。

柯里化是我们将使用部分应用，但我们将用多个嵌套函数组成多参数函数的概念。那么柯里化给我们带来了什么，而其他概念无法做到呢？如果我们处于纯函数世界，实际上我们可以得到很多。例如，数组上的`map`函数。它希望一个单个项目的函数定义（我们将忽略通常不使用的其他参数），并希望函数返回一个单个项目。当我们有一个像下面这样的函数，并且它可以在`map`函数中使用，但它有多个参数时会发生什么？以下代码展示了我们可以用柯里化和这种用例做些什么：

```js
const calculateArtbitraryValueWithPrecision = function(prec=0, val) {
    return function(val) {
        return parseFloat((val / 1000).toFixed(prec));
    }
}
const arr = new Array(50000);
for(let i = 0; i < arr.length; i++) {
    arr[i] = i + 1000;
}
console.log(arr.map(calculatorArbitraryValueWithPrecision(2)));
```

我们正在接受一个通用函数（甚至是任意的），并通过使其更具体（在本例中是保留两位小数）在`map`函数中使用它。这使我们能够编写非常通用的函数，可以处理任意数据并从中制作特定的函数。

我们将在我们的代码中使用部分应用，并且可能会使用柯里化。然而，总的来说，我们不会像在纯函数式语言中那样使用柯里化，因为这可能会导致减速和更高的内存消耗。最重要的是要理解部分应用和外部作用域变量如何在内部作用域位置中使用的概念。

这三个概念对于纯函数式编程的理念非常关键，但我们将不会利用其中大部分。在高性能代码中，我们需要尽可能地提高速度和内存利用率，而其中大部分构造占用的资源超出了我们的承受范围。某些概念可以在高性能代码中大量使用。以下内容将在后续章节中使用：部分应用、流式/惰性求值，可能还有一些递归。熟悉函数式代码将有助于在使用利用这些概念的库时更加得心应手，但正如我们长时间讨论过的那样，它们并不像我们的迭代方法那样高性能。

# 总结

在本章中，我们已经了解了可变性和不可变性的概念。我们已经看到不可变性可能会导致减速和更高的内存消耗，并且在编写高性能代码时可能会成为一个问题。我们已经看了可变性以及如何确保我们编写的代码利用了它，但也使其安全。除此之外，我们还进行了可变和不可变代码的性能比较，并看到了不可变类型的速度和内存消耗增加的情况。最后，我们看了 JavaScript 中的函数式编程以及我们如何利用这些概念。函数式编程可以帮助解决许多问题，比如无锁并发，但我们也知道 JavaScript 运行时是单线程的，因此这并没有给我们带来优势。总的来说，我们可以从不同的编程范式中借鉴许多概念，拥有这些概念可以使我们成为更好的程序员，并帮助我们编写干净、安全和高性能的代码。

在下一章中，我们将看一下 JavaScript 作为一种语言是如何发展的。我们还将看一下浏览器是如何改变以满足开发人员的需求的，新的 API 涵盖了从访问 DOM 到长期存储的所有内容。
