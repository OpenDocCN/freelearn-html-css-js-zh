# 第四章：实际示例-看看 Svelte 和 Vanilla

由于过去几章讨论了现代网络和我们可用的 API，现在我们将实际示例中使用这些 API。在创建与之相关的一种*运行时*的 Web 框架方面已经有了相当多的发展。这个*运行时*几乎可以归因于**虚拟 DOM**（**VDOM**）和状态系统。当这两个东西相互关联时，我们能够创建丰富和反应灵敏的前端。这些框架的例子包括 React、Vue 和 Angular。

但是，如果我们摆脱 VDOM 和运行时概念，并以某种方式将所有这些代码编译为纯 JavaScript 和 Web API 调用，会发生什么？这就是 Svelte 框架的创建者所考虑的：利用浏览器中已有的内容，而不是创建我们自己的浏览器版本（这显然是一个过度简化，但并不太离谱）。在本章中，我们将看看 Svelte 以及它如何实现一些魔术，以及使用这个框架编写的一些应用程序示例。这应该让我们对 Svelte 和存在的*无运行时*框架有一个很好的理解，以及它们如何潜在地加快我们的应用程序运行速度。

本章涉及的主题如下：

+   纯速度的框架

+   构建基础-待办事项应用程序

+   变得更花哨-基本天气应用程序

# 技术要求

本章需要以下内容：

+   诸如**Visual Studio Code**（**VS Code**）之类的编辑器或 IDE

+   Node 环境设置

+   对 DOM 的良好理解

+   Chrome 等 Web 浏览器

+   在[`github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter04`](https://github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter04)找到的代码。

# 纯速度的框架

Svelte 框架决定将焦点从基于运行时的系统转移到基于编译器的系统。这可以在他们的网站上看到，位于[`svelte.dev`](https://svelte.dev)。在他们的首页上，甚至明确指出了以下内容：

Svelte 将您的代码编译为微小的、无框架的 vanilla JS-您的应用程序启动快速并保持快速。

通过将步骤从运行时移至初始编译，我们能够创建下载和运行速度快的应用程序。但是，在我们开始研究这个编译器之前，我们需要将其安装到我们的机器上。以下步骤应该使我们能够开始为 Svelte 编写代码（直接从[`svelte.dev/blog/the-easiest-way-to-get-started`](https://svelte.dev/blog/the-easiest-way-to-get-started)获取）：

```js
> npx degit sveltejs/template todo
> cd todo
> npm install
> npm run dev
```

有了这些命令，我们现在有一个位于`localhost:5000`的运行中的 Svelte 应用程序。让我们看看让我们如此快速启动的`package.json`中有什么。首先，我们会注意到我们有一堆基于 Rollup 的依赖项。Rollup 是 JavaScript 的模块捆绑器，还有一套丰富的工具来执行许多其他任务。它类似于 webpack 或 Parcel，但这是 Svelte 决定依赖的工具。我们将在第十二章中更深入地了解 Rollup，*构建和部署完整的 Web 应用程序*。只需知道它正在为我们编译和捆绑我们的代码。

似乎我们有一个名为`sirv`的东西（可以在`package.json`文件中看到）。如果我们在`npm`中查找`sirv`，我们会发现它是一个静态资产服务器，但是，它不是直接在文件系统上查找文件（这是一个非常昂贵的操作），而是将请求头和响应缓存在内存中一段时间。这使得它能够快速提供可能已经被提供的资产，因为它只需要查看自己的内存，而不是进行 I/O 操作来查找资产。**命令行界面**（**CLI**）使我们能够快速设置服务器。

最后，我们以开发模式启动我们的应用程序。如果我们查看`package.json`文件的`scripts`部分，我们会看到它运行以下命令：`run-p start:dev autobuild`。`run-p`命令表示并行运行所有后续命令。`start:dev`命令表示在开发模式下启动我们的`sirv`服务器，`autobuild`命令告诉 Rollup 编译和监视我们的代码。这意味着每当我们对文件进行更改时，它都会自动为我们构建。让我们快速看看它的运行情况。让我们进入`src`文件夹并对`App.svelte`文件进行更改。添加以下内容：

```js
//inside of the script tag
export let name;
export let counter;

function clicker() {
   counter += 1;
}

//add to the template
<span>We have been clicked {counter} times</span>
<button on:click={clicker}>Click me!</button>
```

我们会注意到我们的网页已经自动更新，现在我们有一个基于事件的响应式网页！这在开发模式下非常好，因为我们不必不断触发编译器。

这些示例中的首选编辑器是 VS Code。如果我们转到 VS Code 的扩展部分，那里有一个很好的 Svelte 插件。我们可以利用这个插件进行语法高亮和一些警报，当我们做错事时。如果首选编辑器没有 Svelte 插件，请尝试至少启用编辑器的 HTML 高亮显示。

好的：这个简单的例子已经给了我们很多东西可以看。首先，`App.svelte`文件给我们提供了类似 Vue 文件的语法。我们有一个 JavaScript 部分，一个样式部分，和一个增强的 HTML 部分。我们导出了两个变量，名为`name`和`counter`。我们还有一个函数，我们在按钮的点击处理程序中使用。我们还为我们的`h1`元素启用了样式。

看起来花括号添加了我们从这些响应式框架中期望的单向数据绑定。它看起来也像是我们通过简单的`on:<event>`绑定来附加事件，而不是利用内置的`on<event>`系统。

如果我们现在进入`main.js`文件，我们会看到我们正在导入刚刚查看的 Svelte 文件。然后我们创建一个新的*app*（它应该看起来很熟悉，类似其他响应式框架），并且将我们的应用程序定位到文档的主体。除此之外，我们还设置了一些属性，即我们之前导出的`name`和`counter`变量。然后我们将其作为此文件的默认导出进行导出。

所有这些都应该与前一章非常相似，当我们查看内置于浏览器中的类和模块系统时。Svelte 只是借用了这些类似的概念来编写他们的编译器。现在，我们应该看一下编译过程的输出。我们会注意到我们有一个`bundle.css`和一个`bundle.js`文件。如果我们首先查看生成的`bundle.css`文件，我们会看到类似以下的内容：

```js
h1.svelte-i7qo5m{color:purple}
```

基本上，Svelte 通过将它们放在一个唯一的命名空间下来*模仿*Web 组件，这种情况下是`svelte-i7qo5m`。这非常简单，那些使用过其他系统的人会注意到这是许多框架创建作用域样式表的方式。

现在，如果我们进入`bundle.js`文件，我们会看到一个完全不同的情况。首先，我们有一个**立即调用的函数表达式**（**IIFE**），这是实时重新加载代码。接下来，我们有另一个 IIFE，它将我们的应用程序分配给一个名为`app`的全局变量。然后，代码内部有一堆样板代码，如`noop`，`run`和`blank_object`。我们还可以看到 Svelte 包装了许多内置方法，例如 DOM 的`appendChild`和`createElement`API。以下代码可以看到：

```js
function append(target, node) {
    target.appendChild(node);
}
function insert(target, node, anchor) {
    target.insertBefore(node, anchor || null);
}
function detach(node) {
    node.parentNode.removeChild(node);
}
function element(name) {
    return document.createElement(name);
}
function text(data) {
    return document.createTextNode(data);
}
function space() {
    return text(' ');
}
```

他们甚至将`addEventListener`系统包装在自己的形式中，以便他们可以控制回调和生命周期事件。以下代码可以看到：

```js
function listen(node, event, handler, options) {
    node.addEventListener(event, handler, options);
    return () => node.removeEventListener(event, handler, options);
}
```

他们随后有一堆数组，它们被用作各种事件的队列。他们循环遍历这些数组，弹出并运行事件。这可以在他们设计的 flush 方法中看到。有一个有趣的地方是他们设置了`seen_callbacks`。这是为了通过计算可能导致无限循环的方法/事件来阻止无限循环。例如，组件*A*得到一个更新，随后发送一个更新给组件*B*，然后组件*B*再发送一个更新给组件*A*。在这里，`WeakSet`可能是一个更好的选择，但他们选择使用常规的`Set`，因为一旦 flush 方法完成，它将被清除。

一个很好查看的最终函数是`create_fragment`方法。我们会注意到它返回一个对象，其中有一个名为`c`的 create 函数。正如我们所看到的，这将创建我们在 Svelte 文件中拥有的 HTML 元素。然后我们会看到一个`m`属性，这是将我们的 DOM 元素添加到实际文档中的 mount 函数。`p`属性更新了我们绑定到这个 Svelte 组件的属性（在这种情况下是`name`和`counter`属性）。最后，我们有`d`属性，它与`destroy`方法相关，它会删除所有 DOM 元素和 DOM 事件。

通过查看这段代码，我们可以看到 Svelte 正在利用我们如果从头开始构建 UI 并自己利用 DOM API 时会使用的许多概念，但他们只是将它包装成一堆方便的包装器和巧妙的代码行。

了解一个库的一个很好的方法是阅读源代码或查看它的输出。通过这样做，我们可以找到魔力通常存在的地方。虽然这可能不会立即有益，但它可以帮助我们为框架编写代码，甚至利用我们在他们的代码中看到的一些技巧来编写我们自己的代码库。学习的一种方式是模仿他人。

在所有这些中，我们可以看到 Svelte 声称没有运行时。他们利用了 DOM 提供的基本元素，以一些方便的包装器的形式。他们还为我们提供了一个很好的文件格式来编写我们的代码。尽管这可能看起来像一些基本的代码，但我们能够以这种风格编写复杂的应用程序。

我们将编写的第一个应用程序是一个简单的待办事项应用程序。我们将为其添加一些自己的想法，但它起初将是一个传统的待办事项应用程序。

# 构建基础-一个待办事项应用程序

为了开始我们的待办事项应用程序，让我们继续使用我们已经有的模板。现在，在大多数待办事项应用程序中，我们希望能够做以下事情：

+   添加

+   删除/标记为完成

+   更新

所以我们拥有一个基本的 CRUD 应用程序，没有任何服务器操作。让我们继续编写我们期望在这个应用程序中看到的 Svelte HTML：

```js
<script>
    import { createEventDispatcher } from 'svelte';
    export let completed;
    export let num;
    export let description;

    const dispatch = createEventDispatcher();
</script>
<style>
    .completed {
        text-decoration: line-through;
    }
</style>
<li class:completed>
    Task {num}: {description}
    <input type="checkbox" bind:checked={completed} />
    <button on:click="{() => dispatch('remove', null)}">Remove</button>
</li>
```

我们将我们的待办事项应用程序分成了一个待办事项组件和一个通用应用程序。待办事项元素将包含我们的所有逻辑，用于完成和删除元素。正如我们从前面的例子中看到的，我们正在做以下事情：

+   我们公开这项任务的编号和描述。

+   我们有一个隐藏在主应用程序中的已完成属性。

+   我们有一个用于样式化已完成项目的类。

+   列表元素与完成变量绑定到完成类。

+   `num`和`description`属性与信息相关联。

+   当我们完成一个项目时，会添加一个复选框。

+   还有一个按钮，它会告诉我们的应用程序我们想要删除什么。

这是相当多的内容需要消化，但当我们把它们放在一起时，我们会发现这包含了大部分单个待办事项的逻辑。现在，我们需要添加我们应用程序的所有逻辑。它应该看起来像下面这样：

```js
<script>
    import Todo from './Todo.svelte';

    let newTodoText = '';
    const Todos = new Set();

    function addTodo() {
        const todo = new Todo({
            target: document.querySelector('#main'),
            props: {
                num : Todos.size,
                description : newTodoText
            }
        });
        newTodoText = '';
        todo.$on('remove', () => {
            Todos.delete(todo);
            todo.$destroy();
        });
        Todos.add(todo);
    }
</script>
<style></style>
<h1>Todo Application!</h1>
<ul id="main">
</ul>
<button on:click={addTodo}>Add Todo</button>
<input type="text" bind:value={newTodoText} />
```

首先导入我们之前创建的“待办事项”。然后，我们将`newTodoText`作为与我们的输入文本绑定的属性。然后，我们创建一个集合来存储我们所有的“待办事项”。接下来，我们创建一个`addTodo`方法，该方法将绑定到我们的“添加待办事项”按钮的`click`事件上。这将创建一个新的“待办事项”，将元素绑定到我们的无序列表，并将属性设置为我们的集合大小和输入文本。我们重置“待办事项”文本，并添加一个移除监听器来销毁“待办事项”，并从我们的集合中删除它。最后，我们将其添加到我们的集合中。

我们现在有了一个基本的待办事项应用程序！所有这些逻辑应该都很简单。让我们添加一些额外的功能，就像在上一章中一样。我们将向我们的待办事项应用程序添加以下内容，使其更加健壮和有用：

+   每个“待办事项”都有关联的截止日期

+   保持所有“待办事项”的计数

+   创建过期、已完成和全部过滤器

+   基于过滤器和每个“待办事项”的添加进行过渡

首先，让我们向我们的待办事项应用程序添加一个截止日期。我们将在我们的`Todo.svelte`文件中添加一个名为`dueDate`的新导出字段，并将其添加到我们的模板中，如下所示：

```js
//inside of script tag
export let dueDate;

//part of the template
<li class:completed>
    Task {num}: {description} - Due on {dueDate}
    <input type="checkbox" bind:checked={completed} />
    <button on:click="{() => dispatch('remove', null)}">Remove</button>
</li>
```

然后，在我们的`App.svelte`文件中，我们将添加一个日期控件，并确保当我们将我们的“待办事项”添加到列表时，我们还要确保将此字段放回去。这应该看起来像以下内容：

```js
//inside of the script tag
let newTodoDate = null;
function addTodo() {
    const todo = new Todo({
        target: document.querySelector('#main'),
        props: {
            num : Todos.size + 1,
            dueDate : newTodoDate,
            description : newTodoText
        }
    });
    newTodoText = '';
    newTodoDate = null;
    todo.$on('remove', () => {
        Todos.delete(todo);
        todo.$destroy();
    });
    Todos.add(todo);
}

//part of the template
<input type="date" bind:value={newTodoDate} />
```

我们现在有一个完全功能的截止日期系统。接下来，我们将在我们的应用程序中添加当前“待办事项”的数量。这只需要将一些文本绑定到我们集合的大小的 span 中，如下所示：

```js
//inside of script tag
let currSize = 0;
function addTodo() {
    const todo = new Todo({
        // code removed for readability
    });
    todo.$on('remove', () => {
        Todos.delete(todo);
        currSize = Todos.size;
        todo.$destroy();
    });
    Todos.add(todo);
    currSize = Todos.size;
}

//part of the template
<h1>Todo Application! <span> Current number of Todos: {currSize}</span></h1>
```

好了，现在我们想要对所有日期和已完成状态做一些处理。让我们添加一些过滤器，这样我们就可以删除不符合我们条件的“待办事项”。我们将添加已完成和过期过滤器。我们将把它们做成复选框，因为一项任务可以同时过期和已完成：

```js
//inside of script tag
let completed = false;
let overdue = false;

//part of the template
<label><input type="checkbox" bind:checked={completed}
    on:change={handleFilter}/>Completed</label>
<label><input type="checkbox" bind:checked={overdue}
    on:change={handleFilter}/>Overdue</label>
```

我们的处理过滤逻辑应该看起来像以下内容：

```js
function handleHide(item) {
    const currDate = Date.now();
    if( completed && overdue ) {
        item.hidden = !item.completed || new Date(item.dueDate).getTime() < currDate;
        return;
    }
    if( completed ) {
        item.hidden = !item.completed;
        return;
    }
    if( overdue ) {
        item.hidden = new Date(item.dueDate).getTime() < currDate;
        return;
    }
    item.hidden = false;
}

function handleFilter() {
    for(const item of Todos) {
        handleHide(item);
    }
}
```

我们还需要确保对任何新的“待办事项”项目都有相同的隐藏逻辑：

```js
const todo = new Todo({
    target: document.querySelector('#main'),
    props: {
        num : Todos.size + 1,
        dueDate : newTodoDate,
        description : newTodoText
    }
});
handleHide(todo);
```

最后，我们的`Todo.svelte`组件应该看起来像以下内容：

```js
<svelte:options accessors={true} />
<script>
    import { createEventDispatcher } from 'svelte';

    export let num;
    export let description;
    export let dueDate;
    export let hidden = false;
    export let completed = false;

    const dispatch = createEventDispatcher();
</script>
<style>
    .completed {
        text-decoration: line-through;
    }
    .hidden {
        display : none;
    }
</style>
<li class:completed class:hidden>
    Task {num}: {description} - Due on {dueDate}
    <input type="checkbox" bind:checked={completed} />
    <button on:click="{() => dispatch('remove', null)}">Remove</button>
</li>
```

这些大部分应该看起来很熟悉，除了顶部部分。我们可以在 Svelte 文件中添加特殊标签，以便访问某些属性，例如以下内容：

+   `<svelte:window>` 给了我们访问窗口事件的权限。

+   `<svelte:body>` 给了我们访问 body 事件的权限。

+   `<svelte:head>` 给了我们访问文档头部的权限。

+   `<svelte:component>` 给了我们访问自己作为 DOM 元素的权限。

+   `<svelete:self>` 允许我们包含自己（用于递归结构，如树）。

+   `<svelte:options>` 允许我们向组件添加编译器选项。

在这种情况下，我们希望我们的父组件能够通过 getter/setter 访问我们的属性，因此我们将`accessors`选项设置为`true`。这就是我们能够在`App.svelte`文件中更改我们的隐藏属性，并允许我们获取每个“待办事项”的属性的方式。

最后，让我们添加一些淡入淡出的过渡效果。Svelte 在添加/删除元素时带有一些不错的动画。我们要使用的是`fade`动画。因此，我们的`Todo.svelte`文件现在将添加以下内容：

```js
//inside of script tag
import { fade } form 'svelte/transition';

//part of template
{#if !hidden}
    <li in:fade out:fade class:completed>
        Task {num}: {description} - Due on {dueDate}
        <input type="checkbox" bind:checked={completed} />
        <button on:click="{() => dispatch('remove', null)}">Remove</button>
    </li>
{/if}
```

这种特殊的语法是用于条件性 DOM 添加/删除。就像我们可以用 DOM API 添加/删除子元素一样，Svelte 也在做同样的事情。接下来，我们可以看到我们在列表元素上添加了`in:fade`和`out:fade`指令。现在，当元素从 DOM 中添加或移除时，它将淡入和淡出。

我们现在有一个相当功能齐全的待办事项应用程序。我们有过滤逻辑，与截止日期相关的“待办事项”，甚至还有一点动画。下一步是稍微整理一下代码。我们可以通过 Svelte 内置的存储来实现这一点。

存储是一种在不必使用一些我们在应用程序中必须使用的技巧的情况下共享状态的方法（当我们可能不应该打开访问者系统时）。我们的`Todos`和我们的主应用程序之间的共享状态是过期和已完成的过滤器。每个`Todo`很可能应该控制这个属性，但我们目前正在利用访问者选项，并且所有的过滤都是在我们的主应用程序中完成的。有了可写存储，我们就不再需要这样做了。

首先，我们编写一个`stores.js`文件，如下所示：

```js
import { writable } from 'svelte/store';

export const overdue = writable(false);
export const completed = writable(false);
```

接下来，我们更新我们的`App.svelte`文件，不再针对`Todos`中的`hidden`属性，并将我们的复选框输入的`checked`属性绑定到存储，如下所示：

```js
//inside of script tag
import { completed, overdue } from './stores.js';

//part of the template
<label><input type="checkbox" bind:checked={$completed} />Completed</label>
<label><input type="checkbox" bind:checked={$overdue} />Overdue</label>
```

我们脚本中的存储前面的美元符号表示这些是存储而不是变量。它允许我们在销毁时更新和订阅存储，而无需取消订阅。最后，我们可以更新我们的`Todo.svelte`文件，使其如下所示：

```js
<script>
    import { overdue, completed } from './stores.js';
    import { createEventDispatcher, onDestroy } from 'svelte';
    import { fade } from 'svelte/transition';

    export let num;
    export let description;
    export let dueDate;
    let _completed = false;

    const dispatch = createEventDispatcher();
</script>
<style>
    .completed {
        text-decoration: line-through;
    }
</style>
{#if
    !(
         ($completed && !_completed) ||
         ($overdue && new Date(dueDate).getTime() >= Date.now())
     )
}
    <li in:fade out:fade class:_completed>
        Task {num}: {description} - Due on {dueDate}
        <input type="checkbox" bind:checked={_completed} />
        <button on:click="{() => dispatch('remove', null)}">Remove</button>
    </li>
{/if}
```

我们已经将过期和已完成的存储添加到我们的系统中。您可能已经注意到，我们已经摆脱了文件顶部的编译器选项。然后我们将我们的`#if`条件链接到这些存储。我们现在已经将隐藏`Todos`的责任放在了`Todos`自身上，同时也删除了相当多的代码。很明显，我们可以以多种方式构建 Svelte 应用程序，并对应用程序保持相当多的控制。

在进入下一个应用程序之前，继续查看捆绑的 JavaScript 和 CSS，并向应用程序添加新功能。接下来，我们将看看如何构建一个天气应用程序并从服务器获取这些信息。

# 变得更加花哨-一个基本的天气应用程序

很明显，Svelte 已经建立起了与大多数现代 ECMAScript 标准兼容的编译器。他们没有提供任何获取数据的包装器的领域是在这里。添加这个并看到效果的一个好方法是构建一个基本的天气应用程序。

天气应用程序在其核心需要能够输入邮政编码或城市，并输出该地区的当前天气信息。我们还可以根据这个位置得到天气的预测。最后，我们还可以将这些选择保存在浏览器中，这样我们在回到应用程序时就可以使用它们。

对于我们的天气数据，我们将从[`openweathermap.org/api`](https://openweathermap.org/api)获取。在这里，免费服务将允许我们获取当前天气。除此之外，我们还需要一个输入系统，可以接受以下内容：

+   城市/国家

+   邮政编码（如果没有给出国家，我们将假设是美国，因为这是 API 的默认值）

当我们输入正确的值时，我们将把它存储在`LocalStorage`中。在本章的后面，我们将更深入地研究`LocalStorage`API，但请注意它是浏览器中的键值存储机制。当我们输入输入值时，我们将得到所有先前搜索的下拉列表。我们还将添加删除列表中任何一个结果的功能。

首先，我们需要获取一个 API 密钥。要做到这一点，请按照以下步骤进行：

1.  前往[`openweathermap.org/api`](https://openweathermap.org/api)并按照说明获取 API 密钥。

1.  一旦我们创建了一个帐户并验证它，我们就能够添加 API 密钥。

1.  登录后，应该有一个标签，上面写着**API keys**。如果我们去那里，应该会看到一个*no api keys*的消息。

1.  我们可以创建一个密钥并为其添加一个名称（我们可以称之为`default`）。

1.  有了这个密钥，我们现在可以开始调用他们的服务器。

让我们继续设置一个测试调用。以下代码应该可以工作：

```js
let api_key = "<your_api_key>";
fetch(`https://api.openweathermap.org/data/2.5/weather?q=London&appid=${api_key}`)
    .then((res) => res.json())
    .then((final) => console.log(final));
```

如果我们将这些放入代码片段中，我们应该会得到一个包含大量数据的 JSON 对象。现在我们可以继续使用 Svelte 来利用这个 API 创建一个漂亮的天气应用程序。

让我们以与设置 Todo 应用程序相同的方式设置我们的应用程序。运行以下命令：

```js
> cd ..
> npx degit sveltejs/template weather
> cd weather
> npm install
> npm run dev
```

现在我们已经启动了环境，让我们创建一个带有一些基本样式的样板应用程序。在`global.css`文件中，将以下行添加到 body 中：

```js
display: flex;
flex-direction : column;
align-items : center;
```

这将确保我们的元素都是基于列的，并且它们将从中心开始并向外扩展。这将为我们的应用程序提供一个漂亮的外观。接下来，我们将创建两个 Svelte 组件，一个`WeatherInput`和一个`WeatherOutput`组件。接下来，我们将专注于输入。

我们需要以下项目，以便从用户那里获得正确的输入：

+   输入邮政编码或城市

+   输入国家代码

+   一个提交按钮

我们还将向我们的应用程序添加一些条件逻辑。我们将根据输入框左侧的复选框有条件地呈现文本或数字输入，而不是尝试解析输入。有了这些想法，我们的`WeatherInput.svelte`文件应该如下所示：

```js
<script>
    import { zipcode } from './stores.js';
    const api_key = '<your_api_key>'

    let city = null;
    let zip = null;
    let country_code = null;

    const submitData = function() {
        fetch(`https://api.openweathermap.org/data/2.5/weather?q=${zipcode 
         ? zip : city},${country_code}&appid=${api_key}`)
            .then(res => res.json())
            .then(final => console.log(final));
    }
</script>
<style>
    input:valid {
        border: 1px solid #333;
    }
    input:invalid {
        border: 1px solid #c71e19;
    }
</style>
<div>
    <input type="checkbox" bind:checked={$zipcode} />
    {#if zipcode}
        <input type="number" bind:value={zip} minLength="6" maxLength="10" 
         require />
    {:else}
        <input type="text" bind:value={city} required />
    {/if}
    <input type="text" bind:value={country_code} minLength="2" 
     maxLength="2" required />
    <button on:click={submitData}>Check</button>
</div>
```

有了这个，我们就有了我们输入的基本模板。首先，我们创建一个`zipcode`存储，以有条件地显示数字或文本输入。然后，我们创建了一些本地变量，将它们绑定到我们的输入值上。`submitData`函数将在我们准备好获得某种响应时提交所有内容。目前，我们只是将输出记录到开发者控制台中。

对于样式，我们只是为有效和无效的输入添加了一些基本样式。我们的模板给了我们一个复选框，用于打开`zipcode`功能或关闭它。然后我们有条件地显示`zipcode`或城市文本框。每个文本框都添加了内置验证。接下来，我们添加了另一个文本字段，以从用户那里获取国家代码。最后，我们添加了一个按钮，将会去检查数据。

在 Svelte 中，括号被大量使用。输入验证的一个特性是基于正则表达式的。该字段称为模式。如果我们在这里尝试使用括号，将会导致 Svelte 编译器失败。请注意这一点。

在进行输出之前，让我们先给我们的输入添加一些标签，以使用户更容易使用。以下内容应该可以做到：

```js
//in the style tag
input {
    margin-left: 10px;
}
label {
    display: inline-block;
}
#cc input {
    width: 3em;
}
```

对于每个`input`元素，我们已经将它们包装在`label`中，如下所示：

```js
<label id="cc">Country Code<input type="text" bind:value={country_code} minLength="2" maxLength="2" required /></label>
```

有了这个，我们有了`input`元素的基本用户界面。现在，我们需要让`fetch`调用实际输出到可以在我们创建后可用于`WeatherOutput`元素的东西。让我们创建一个自定义存储来实现`gather`方法，而不是只是将这些数据作为 props 传递出去。在`stores.js`中，我们应该有以下内容：

```js
function createWeather() {
    const { subscribe, update } = writable({});
    const api_key = '<your_api_key>';
    return {
        subscribe,
        gather: (cc, _z, zip=null, city=null) => {
            fetch(`https://api.openweathermap.org/data/2.5/weather?=${_z ? 
             zip : city},${cc}&appid=${api_key})
                .then(res => res.json())
                .then(final => update(() => { return {...final} }));
        }
    }
}
```

我们现在已经将获取数据的逻辑移到了存储中，现在我们可以订阅这个存储来更新自己。这意味着我们可以让`WeatherOutput`组件订阅这个存储以获得一些基本输出。以下代码应该放入`WeatherOtuput.svelte`中：

```js
<script>
    import { weather } from './stores.js';
</script>
<style>
</style>
<p>{JSON.stringify($weather)}</p>
```

现在我们所做的就是将我们的天气输出放入一个段落元素中，并对其进行字符串化，以便我们可以在不查看控制台的情况下阅读输出。我们还需要更新我们的`App.svelte`文件，并像这样导入`WeatherOutput`组件：

```js
//inside the script tag
import WeatherOutput from './WeatherOutput.svelte'

//part of the template
<WeatherOutput></WeatherOutput>
```

如果我们现在测试我们的应用程序，我们应该会得到一些难看的 JSON，但是我们现在通过存储将我们的两个组件联系起来了！现在，我们只需要美化输出，我们就有了一个完全功能的天气应用程序！更改`WeatherOutput.svelte`中的样式和模板如下：

```js
<div>
    {#if $weather.error}
        <p>There was an error getting your data!</p>
    {:else if $weather.data}
        <dl>
            <dt>Conditions</dt>
            <dd>{$weather.weather}</dd>
            <dt>Temperature</dt>
            <dd>{$weather.temperature.current}</dd>
            <dd>{$weather.temperature.min}</dd>
            <dd>{$weather.temperature.max}</dd>
            <dt>Humidity</dt>
            <dd>{$weather.humidity}</dd>
            <dt>Sunrise</dt>
            <dd>{$weather.sunrise}</dd>
            <dt>Sunset</dt>
            <dd>{$weather.sunset}</dd>
            <dt>Windspeed</dt>
            <dd>{$weather.windspeed}</dd>
            <dt>Direction</dt>
            <dd>{$weather.direction}</dd>
        </dl>
    {:else}
        <p>No city or zipcode has been submitted!</p>
    {/if}
</div>
```

最后，我们应该添加一个新的控件，让我们的用户可以选择输出的公制或英制单位。将以下内容添加到`WeatherInput.svelte`中：

```js
<label>Metric?<input type="checkbox" bind:checked={$metric}</label>
```

我们还将在`stores.js`文件中使用一个新的`metric`存储，默认值为`false`。有了这一切，我们现在应该有一个功能齐全的天气应用程序了！我们唯一剩下的部分是添加`LocalStorage`功能。

有两种类型的存储可以做类似的事情。它们是`LocalStorage`和`SessionStorage`。主要区别在于它们的缓存时间有多长。`LocalStorage`会一直保留，直到用户删除缓存或应用程序开发人员决定删除它。`SessionStorage`在页面的生命周期内保留在缓存中。一旦用户决定离开页面，`SessionStorage`就会清除。离开页面意味着关闭标签页或导航离开；它不意味着重新加载页面或 Chrome 崩溃并且用户恢复页面。由设计者决定使用哪种方式。

利用`LocalStorage`非常容易。在我们的情况下，该对象保存在窗口上（如果我们在工作程序中，它将保存在全局对象上）。需要记住的一件事是，当我们使用`LocalStorage`时，它会将所有值转换为字符串，因此如果我们想要存储复杂对象，我们需要进行转换。

要更改我们的应用程序，让我们专门为我们的下拉列表创建一个新组件。让我们称之为`Dropdown`。首先，创建一个`Dropdown.svelte`文件。接下来，将以下代码添加到文件中：

```js
<script>
    import { weather } from './stores.js';
    import { onDestroy, onMount } from 'svelte';

    export let type = "text";
    export let name = "DEFAULT";
    export let value = null;
    export let required = true;
    export let minLength = 0;
    export let maxLength = 100000;
    let active = false;
    let inputs = [];
    let el;

    const unsubscribe = weather.subscribe(() => {
        if(!inputs.includes(value) ) {
            inputs = [...inputs, value];
            localStorage.setItem(name, inputs);
        }
        value = '';
    });
    const active = function() {
        active = true;
    }
    const deactivate = function(ev) {
        if(!ev.path.includes(el) ) 
            active = false;
    }
    const add = function(ev) {
        value = ev.target.innerText;
        active = false;
    }
    const remove = function(ev) {
        const text = ev.target.parentNode.querySelector('span').innerText;
        const data = localStorage.getItem(name).split(',');
        data.splice(data.indexOf(text));
        inputs = [...data];
        localStorage.setItem(name, inputs);
    }
    onMount(() => {
        const data = localStorage.getItem(name);
        if( data === "" ) { inputs = []; }
        else { inputs = [...data.split(',')]; }
    });
    onDestroy(() => {
        unsubscribe();
    });
</script>
<style>
    input:valid {
        border 1px solid #333;
    }
    input:invalid {
        border 1px solid #c71e19;
    }
    div {
        position : relative;
    }
    ul {
        position : absolute;
        top : 100%;
        list-style-type : none;
        background : white;
        display : none;
    }
    li {
        cursor : hand;
        border-bottom : 1px solid black;
    }
    ul.active {
        display : inline-block;
    }
</style>
<svelte:window on:mousedown={deactivate} />
<div>
    {#if type === "text"}
        <input on:focus={activate} type="text" bind:value={value} 
         {minLength} {maxLength} {required} />
    {:else}
        <input on:focus={activate} type="number" bind:value={value} 
         {minLength} {maxLength} {required} />
    {/if}
    <ul class:active bind:this={el}>
        {#each inputs as input }
            <li><span on:click={add}>{input}</span> <button 
             on:click={remove}>&times;</button></li>
        {/each}
    </ul>
</div>
```

这是相当多的代码，让我们分解一下我们刚刚做的事情。首先，我们将我们的输入更改为`dropdown`组件。我们还将许多状态内部化到这个组件中。我们打开各种字段，以便用户能够自定义字段本身。我们需要确保设置的主要字段是`name`。这是我们用于存储搜索的`LocalStorage`键。

接下来，我们订阅`weather`存储。我们不使用实际数据，但我们确实获得事件，因此如果选择是唯一的（可以使用集合而不是数组），我们可以将其添加到存储中。如果我们想要激活下拉列表，我们还添加了一些基本逻辑，如果我们聚焦或者点击了下拉列表之外。我们还为列表元素的点击事件添加了一些逻辑（实际上是将其添加到列表元素的子元素），以将文本放入下拉列表或从我们的`LocalStorage`中删除。最后，我们为组件的`onMount`和`onDestroy`添加了行为。`onMount`将从`localStorage`中获取并将其添加到我们的输入列表中。`onDestroy`只是取消了我们的订阅，以防止内存泄漏。

其余的样式和模板应该看起来很熟悉，除了无序列表系统中的`bind:this`。这允许我们将变量绑定到元素本身。这使我们能够在事件路径中的元素不在列表中时取消激活我们的下拉列表。

有了这些，对`WeatherInput.svelte`进行以下更新：

```js
//inside the script tag
import Dropdown from './Dropdown.svelte';

//part of the template
{#if $zipcode}
    <label>Zip<Dropdown name="zip" type="number" bind:value={zip} 
     minLength="6" maxLength="10"></Dropdown></label>
{:else}
    <label>City<Dropdown name="city" bind:value={city}></Dropdown></label>
{/if}
<label>Country Code<Dropdown name="cc" bind:value={country_code} 
 minLength="2" maxLength="2"></Dropdown></label>
```

我们现在已经创建了一个半可重用的`dropdown`组件（我们依赖于天气存储，因此它实际上只适用于我们的应用程序），并且已经创建了一个看起来像单个组件的东西。

# 总结

Svelte 是一个有趣的框架，我们将代码编译成原生 JavaScript。它利用现代思想，如模块、模板和作用域样式。我们还能够以简单的方式创建可重用的组件。虽然我们可以对我们构建的应用程序进行更多的优化，但我们可以看到它们确实有多快。虽然 Svelte 可能不会成为应用程序开发的主流选择，但它是一个很好的框架，可以看到我们在之前章节中探讨的许多概念。

接下来，我们将暂时离开浏览器，看看如何利用 Node.js 在服务器上使用 JavaScript。我们在这里看到的许多想法将被应用在那里。我们还将看到编写应用程序的新方法，以及如何在整个网络生态系统中使用一种语言。
