# 第十二章。利用 Flux 库

首先，Flux 是一套架构指南，指定为我们遵循的模式。虽然这提供了最大的灵活性，但在决定如何实现特定的 Flux 组件时，有时可能会让人感到无所适从。幸运的是，有一些非常好的 Flux 库提供了对 Flux 组件有见地的实现，这减少了我们需要编写的许多样板代码。本章的目的是查看这些库中的两个，以展示 Flux 实现可以有多么不同。目标不是合规，而是一个稳固的架构，有助于我们的应用程序完成任务。

# 实现核心 Flux 组件

在本节中，我们将重申我们可以改变架构中各种 Flux 组件的具体实现的想法。我们将从讨论分发器本身开始，并考虑我们可能做出的各种更改。然后，我们将考虑存储以及我们可能想要在那里做出的增强。最后，我们将讨论动作和动作创建函数。

## 自定义分发器

在第十章，*实现分发器*，我们实现了自己的分发器组件。Facebook 提供的参考实现完全可以使用，但它并不是每个生产 Flux 架构中默认的组件。相反，它是一个起点，这样我们可以看到 Flux 分发器规范应该如何工作。

我们的解决方案是公开分发器模块中的`dispatch()`和`register()`函数。通过这样做，我们在代码的其他部分使用分发器变得更加直接。不再需要考虑分发器实例——一切都被封装在分发器模块中。

一个通用的 Flux 库可能希望更进一步，完全取消分发器，这听起来可能有些疯狂——它是一个基本的 Flux 组件。然而，我们仍然可以在不明确实现这个抽象的情况下实现分发器的相同架构原则。这就是将 Flux 作为一套规范而不是具体实现发布出来的全部意义。我们在概念上知道 Flux 架构应该做什么和不应该做什么——我们有机会选择如何通过我们的实现来强制执行这些规则。

## 实现基础存储

在第十章实现分发器中，我们做出的另一个改进是对存储层次结构的改进。我们的每个存储都继承自一个基类。我们实现这个功能的主要原因是为了自动化存储与分发器的注册，这是有益的，因为一个没有监听分发器发出的事件的 Flux 存储是没有意义的。也许 Flux 库应该为我们处理这种基础功能。

我们还实现了方法动作处理器。实际上，在我们的实现中，这是派发器本身的一个功能，而且相当受限。也许基础存储是这种功能合适的地点。库应该包含这种类型的通用复杂性，而不是我们的应用程序。

使用 Flux 存储继承基本功能的好处在于，这是我们应用程序的大脑所在之处。如果我们发现一些通用的状态转换行为适用于多个存储，那么有一个基础存储在位，就使我们能够轻松地将通用代码提取出来。也许 Flux 库可以在其基础存储中提供一些基本的转换，这样我们就可以从中继承。

## 创建动作

常量是 Flux 架构中明确动作的绝佳方式。动作模块定义了常量，动作创建函数将常量传递给派发器。存储在确定如何处理派发的动作时也会使用这些常量。这就在动作创建器和响应此动作的存储中的代码之间建立了一个明确的联系。

在第十章《实现派发器》中，我们采用了不同的方法。动作创建函数仍然定义了常量，并在派发动作时使用它们。然而，我们进行了更改，允许我们的存储定义方法处理器。因此，而不是一个监听派发器的函数，存储定义了与动作定义的常量相匹配的方法。从存储的角度来看，这很方便，但如果常量只被动作创建函数使用，那么它们的价值就会降低。

Flux 库可以帮助使派发和处理动作变得更加直接。使用常量和`switch`语句是好的，因为它们使发生的事情变得明确。我们喜欢在 Flux 架构中保持明确性。挑战在于，这种方法要求程序员在实现系统时保持警惕。换句话说，有大量机会出现人为错误。Flux 库可以消除处理常量时可能出现的错误。

另一个 Flux 库可以帮助的领域是与异步动作创建函数相关。我们应用程序的异步行为可能遵循类似的模式：

+   在异步代码运行之前派发一个改变存储状态的动作

+   当响应到达时派发一个动作

+   如果异步行为失败，派发不同的动作。

这几乎就像异步动作有一个生命周期，可以通过 Flux 库将其抽象成一个通用函数。

# 实现痛点

在上一节中，我们讨论了 Flux 中可能从自定义实现中受益的领域。在我们深入探讨`Alt.js`和 Redux 之前，我们将简要讨论一些实现 Flux 架构的痛点。在任意架构中，异步动作都是难以做对的，更不用说 Flux 了。我们将应用程序状态划分到存储中的方式可能是一个棘手的设计问题。如果我们做错了，可能很难恢复。最后，我们还有数据依赖问题需要考虑。

## 分发异步动作

正如我们在上一节中讨论的，异步动作创建者难以实现。这很具挑战性，因为我们通常必须让存储知道这个异步动作即将发生，以便 UI 可以更新以反映这一点。例如，当点击一个发送一个或多个 AJAX 请求的按钮时，我们可能希望在实际上发送请求之前禁用该按钮，以防止重复请求。在 Flux 中，唯一的方法是分发一个动作，因为一切都是单向的。

库可以在一定程度上帮助解决这个问题。例如，预请求动作和成功/错误响应动作可以抽象成更易于使用的东西，因为这是一个常见的模式。然而，即使这样做，也留下了组装请求以获取给定动作所需的所有数据、同步响应并将它们每个传递给存储以便将其转换为视图所需的内容的问题。

也许最好的办法是将这个异步问题排除在 Flux 的作用范围之外。例如，Facebook 已经引入了 GraphQL，这是一种简化从后端服务构建复杂数据并只响应存储实际需要的内容的语言。所有这些都在一个响应中完成，因此我们节省了带宽和延迟。这种方法并不适合每个人，因此 Flux 的实现者需要选择他们想要如何处理异步性，只要客户端的单向数据流保持完整即可。

## 划分存储

在我们的 Flux 架构中错误地划分存储可能是我们面临的最大设计风险之一。通常发生的情况是存储大致平衡；然后，随着系统的演变，所有的新功能最终都进入了一个存储，而其他存储的责任并不明确。换句话说，存储变得不平衡。持有大部分应用程序状态的存储变得过于复杂，难以维护。

我们存储分区可能存在的另一个潜在问题是它们变得过于细粒度。我们也不想看到这种情况发生。尽管由单个存储管理的状态足够简单，但复杂性在于所有这些存储之间的依赖关系。即使没有太多的依赖关系，当需要考虑更多的存储时，在试图推理某事时，我们的大脑中很难保持足够的状态。当相关状态都在一个地方时，预测会发生什么要容易得多。

如果一个 Flux 库，比如 Redux，采取激进的措施，只允许一个存储，从而消除所有混淆的来源，会怎么样？这确实防止了设计问题，如存储分区。相反，正如我们将在本章后面看到的那样，Redux 使用 reducer 函数来转换单个存储的状态。

# 使用 Alt

`Alt.js`是一个为我们实现大量模板代码的 Flux 库。它完全遵循 Flux 概念和模式，但让我们从应用架构的角度关注我们的应用，而不是担心动作常量和`switch`语句。

在本节中，我们将在深入研究简单的待办事项列表示例之前，简要介绍 Alt 的核心概念。示例故意很简单——你将能够将代码映射到你在本书中迄今为止学到的 Flux 概念。

## 核心思想

Facebook Flux 包的主要目标是提供一个基本派发组件的参考实现。这很好地作为 Flux 概念的辅助——动作以同步、单向的方式派发到存储中。正如我们在书中所看到的，派发概念甚至不一定需要暴露给实现 Flux 的人。我们可以简化 Flux 抽象，同时仍然符合 Flux 架构的约束。

Alt 是一个旨在用于生产应用的 Flux 库——它不是一个参考实现。在我们深入代码之前，让我们先了解一下它作为 Flux 库的一些目标。

+   **合规性**：Alt 不借鉴 Flux 中的想法——它真正是为 Flux 系统设计的。例如，存储、动作和视图的概念都相关。同样，Alt 紧密遵循 Flux 架构的原则。像同步更新轮次和单向数据流这样的原则都得到了强制执行。

+   **自动化模板代码**：与实现 Flux 相关的一些繁琐编程任务，Alt 处理得很好。这包括自动创建动作创建函数和动作常量。Alt 还会为我们处理存储动作处理方法——减少了对长`switch`语句的需求。

+   **没有分发器**：我们的代码没有分发器与之交互。将动作分发到所有存储库是在我们调用动作创建者函数时在幕后处理的。存储库之间的依赖管理是在存储库内部直接处理的。

## 创建存储库

我们将要创建的简单应用将为用户显示两个列表。一个列表用于待办事项，另一个列表用于已完成的事项。我们将使用两个存储库——每个列表一个。让我们看看如何使用 `Alt.js` 创建存储库。首先，我们有 `Todo` 存储库：

```js
import alt from '../alt';
import actions from '../actions';

class Todo {
  constructor() {

    // This is the state of the input element
    // used to create a new Todo item.
    this.inputValue = '';

    // The initial list of todo items...
    this.todos = [
      { title: 'Build this thing' },
      { title: 'Build that thing' },
      { title: 'Build all the things' }
    ];

    // Sets up the handler methods to be called
    // when the corresponding action is dispatched.
    this.bindListeners({
      createTodo: actions.CREATE_TODO,
      removeTodo: actions.REMOVE_TODO,
      updateInputValue: actions.UPDATE_INPUT_VALUE
    });
  }

  // Creates a new Todo using the action "payload"
  // as the title.
  createTodo(payload) {
    this.todos.push({ title: payload });
  }

  // Removes the Todo based on the index, which is
  // passed in as the action payload.
  removeTodo(payload) {
    this.todos.splice(payload, 1);
  }

  // Updates the Todo value that the user is currently
  // entering in the Todo input box.
  updateInputValue(payload) {
    this.inputValue = payload;
  }
}

// The "createStore()" function hooks our store class
// up with all the relevant action dispatching machinery,
// returning an instance of the store.
export default alt.createStore(Todo, 'Todo');
```

与我们在本书中迄今为止看到的内容相比，这可能看起来不太熟悉。不用担心；我们现在会逐步解释这些组件。你可能首先会问——状态在哪里？从代码中看不出来，但状态是类的任何实例变量。在这种情况下，是 `inputValue` 字符串和 `todos` 数组。

接下来，我们有一个调用 `bindListeners()` 并传递一个配置对象给它。这就是 Alt 存储库如何将动作映射到方法的方式。你可以看到我们定义了与传递给 `bindListeners()` 的内容相对应的方法。最后，我们有调用 `createStore()` 的操作。这个函数为我们实例化了 `Todo` 存储库类，同时也连接了分发机制。

存储库的定义就到这里——它已经准备好供需要渲染其状态的视图使用。现在让我们看看 `Done` 存储库，它遵循相同的做法，只是组件更少：

```js
import alt from '../alt';
import actions from '../actions';
import todo from './todo';

class Done {
  constructor() {

    // The "done" state holds an array of
    // completed items.
    this.done = [];

    // Binds the only listener of this store.
    this.bindListeners({
      createDone: actions.CREATE_DONE
    });
  }

  // This action payload is the index of an item
  // from the "todo" store. This is called when
  // the item is clicked, and the item is added
  // to the "done" array.
  //
  // Note that this action handler does not mutate
  // the "todo" state as that is not allowed.
  createDone(payload) {
    const { todos } = todo.getState();
    this.done.splice(0, 0, todos[payload]);
  }
}

// Creates the store instance, and hooks it
// up with the Alt dispatching machinery.
export default alt.createStore(Done, 'Done');
```

你可以看到，这个存储库实际上使用 `Todo` 存储库在项目标记为已完成时复制项目数据。然而，这个存储库不会修改 `Todo` 存储库，因为这会违反单向数据流。

### 注意

这些存储库类不是事件发射器，因此它们在状态改变时不会显式地发射任何内容。例如，当待办事项被添加时，视图如何知道有任何变化？由于 `createTodo()` 方法会自动为我们调用，一旦我们的方法执行完毕，通知机制也会自动发生。我们稍后会看到更多关于状态改变通知语义的内容。

## 声明动作创建者

我们已经看到了存储库如何响应分发的动作。现在我们需要一种实际分发这些动作的手段。这可能是我们 Alt 应用程序中最容易的部分。Alt 可以生成我们需要的函数，以及在我们存储库中由 `bindListeners()` 调用使用的常量。让我们看看动作模块，看看它是如何与 Alt 一起工作的：

```js
import alt from './alt';

// Exports an object with functions that accept
// a payload argument. These are the action
// creators. Also creates action constants
// based on the names passed to "generateActions()"
export default alt.generateActions(
  'createTodo',
  'createDone',
  'removeTodo',
  'updateInputValue'
);
```

这将导出一个包含具有与传递给 `generateActions()` 的字符串相同名称的动作创建函数的对象。它还将生成存储使用的动作常量。由于我们的动作创建函数都非常相似，`generateActions()` 具有很高的实用性。我们不再需要维护大量的样板代码。另一方面，还有一些更复杂的案例，涉及需要更多代码的异步动作。如果您对在项目中使用此库感兴趣，请查看 Alt 文档中的异步动作。

## 监听状态变化

在整本书中，我们都在我们的存储发出的更改事件上添加了事件处理函数。使用像 Alt 这样的库，这已经为我们管理了一部分。让我们看看我们应用程序的主要模块，它使用 `AltContainer` React 组件将存储数据馈送到我们的其他 React 组件：

```js
// The React and Alt components we need...
import React from 'react';
import { render } from 'react-dom';
import AltContainer from 'alt-container';

// The stores and React components from
// this application...
import todo from './stores/todo';
import done from './stores/done';
import TodoList from './views/todo-list';
import DoneList from './views/done-list';

// Renders the "AltContainer" component. This
// is where the stores are tied to the views.
// The "TodoList" and "DoneList" components
// are children of the "AltContainer", so
// they get the "todo" and the "done" stores
// as props.
render(
  <AltContainer stores={{ todo, done }}>
    <TodoList/>
    <DoneList/>
  </AltContainer>,
  document.getElementById('app')
);
```

`AltContainer` 组件接受一个 `stores` 属性。容器将监听这些存储中的每一个，并在任何存储的状态发生变化时重新渲染其子组件。这是获取我们的视图监听存储的唯一设置——无需在各个地方手动调用 `on()` 或 `listen()`。在下一节中，我们将查看 `TodoList` 和 `DoneList` 组件，看看它们是如何与 `AltContainer` 一起工作的。

## 渲染视图和分发动作

`TodoList` 组件的职责是渲染来自 `Todo` 存储的项目。这个视图还需要处理两件事。首先，有一个用户用来输入新待办事项的 `input` 元素。其次，我们需要在点击项目时将其标记为完成，通过将其移动到完成列表中。后两个职责涉及事件处理和分发动作。让我们看看待办事项视图的实现：

```js
import React from 'react';
import { Component } from 'react';

import actions from '../actions';

export default class TodoList extends Component {
    render() {

      // The relevant state from the "todo" store
      // that we're rendering here.
      const { todos, inputValue } = this.props.todo;

      // Renders an input for new todos, and the list
      // of current todos. When the user types
      // and then hits enter, the new todo is created.
      // When the user clicks a todo, it's moved to the
      // "done" store.
      return (
        <div>
          <h3>TODO</h3>
          <div>
            <input
              value={inputValue}
              placeholder="TODO..."
              onKeyUp={this.onKeyUp}
              onChange={this.onChange}
              autoFocus
            />
          </div>
          <ul>
            {todos.map(({ title }, i) =>
              <li key={i}>
                <a
                  href="#"
                  onClick={this.onClick.bind(null, i)}
                >{title}</a>
              </li>
            )}
          </ul>
        </div>
      );
    }

    // An active Todo was clicked. The "key" is the
    // index of the Todo within the store. This is
    // passed as the payload to the "createDone()"
    // action, and next to the "removeTodo()" action.
    onClick(key) {
      actions.createDone(key);
      actions.removeTodo(key);
    }

    // If the user has entered some text and the
    // "enter" key is pressed, we use the
    // "createTodo()" action to create a new
    // item using the entered text. Then we clear
    // the input using the "updateInputValue()"
    // action, passing it an empty string.
    onKeyUp(e) {
      const { value } = e.target;

      if (e.which === 13 && value) {
        actions.createTodo(value);
        actions.updateInputValue('');
      }
    }

    // The text input value changed - update the store.
    onChange(e) {
      actions.updateInputValue(e.target.value);
    }
}
```

### 注意

您可能想知道为什么我们不能在按下 **Enter** 键时直接清除 `e.target.value`。确实，我们可以这样做，但这会违反 Flux 的本质，其中状态被保存在存储中。这包括用户输入时的临时值。如果应用程序的另一个部分想要了解文本输入值，它只需要依赖于 `Todo` 存储。如果状态不存在，那么我们的代码将不得不查询 DOM，而我们不想这样做。

最后，让我们看看完成列表组件。这个组件比待办事项列表更简单，因为没有事件处理：

```js
import React from 'react';
import { Component } from 'react';

export default class DoneList extends Component {
  render() {

    // The "done" array is the only state we need
    // from the "done" store.
    const { done } = this.props.done;

    // We want to display these items
    // as strikethrough text.
    const itemStyle = {
      textDecoration: 'line-through'
    }

    // Renders the list of done items, with
    // the "itemStyle" applied to each item.
    return (
      <div>
        <h3>DONE</h3>
        <ul>
          {done.map(({ title }) =>
            <li style={itemStyle}>{title}</li>
          )}
        </ul>
      </div>
    );
  }
}
```

# 使用 Redux

在本节中，我们将探讨 Redux 库来实现 Flux 架构。与 `Alt.js` 不同，Redux 并不旨在实现 Flux 兼容。Redux 的目标是借鉴 Flux 的重要思想，而将繁琐的部分留在了后面。尽管 Redux 没有按照官方文档中指定的方式实现 Flux 组件，但现在 Redux 是 React 架构的首选解决方案。Redux 证明了简洁性总是胜过高级功能。

## 核心思想

在实现一些 Redux 代码之前，让我们花一点时间来了解一下 Redux 的核心思想：

+   **没有分发器**：这就像`Alt.js`，它也从其 API 中移除了分发器的概念。这些 Flux 库没有暴露分发器组件的事实有助于说明 Flux 只是一套思想和模式，而不是一个实现。Alt 和 Redux 都分发动作，只是它们不需要分发器来完成。

+   **一个存储器统治一切**：Redux 摒弃了 Flux 架构需要多个存储器的观点。相反，使用一个存储器来保存整个应用程序状态。乍一看，这可能会让人觉得存储器会变得太大，难以理解。这种情况在多个存储器中同样可能发生，唯一的区别是应用程序状态被分割成不同的模块。

+   **向存储器分发**：当只有一个存储器需要担心时，我们可以做出一些设计上的让步，比如将存储器和分发器视为同一概念。这正是 Redux 所做的事情——它直接向存储器分发动作。

+   **纯减法器**：多个 Flux 存储背后的想法是将应用程序状态分割成几个逻辑上分离的域。我们仍然可以使用 Redux 来实现这一点，区别在于我们使用减法函数将状态分割成域。这些函数负责在分发动作时转换存储的状态。它们是纯的，因为它们返回新的数据并避免引入任何副作用。

## 减法器和存储器

我们现在将实现与使用 Alt 所制作的相同简单的待办事项应用程序——这次使用 Redux。这两个库之间有很多重叠，尤其是与 React 组件本身；那里不需要做太多改变。Redux 与 Alt 和 Flux 的一般区别在于它的单个存储器以及改变其状态的减法函数。话虽如此，我们将首先查看存储器和它的减法函数。

我们将为 Redux 存储的初始状态创建一个模块。这是一个重要的第一步，因为它为转换存储状态的减法函数提供了初始结构。让我们看看初始状态模块：

```js
import Immutable from 'immutable';

// The initial state of the Redux store. The
// "shape" of the application state includes
// two domains - "Todo" and "Done". Each domain
// is an Immutable.js structure.
const initialState = {
  Todo: Immutable.fromJS({
    inputValue: '',
    todos: [
      { title: 'Build this thing' },
      { title: 'Build that thing' },
      { title: 'Build all the things' }
    ]
  }),
  Done: Immutable.fromJS({
    done: []
  })
};

export default initialState;
```

状态是一个简单的 JavaScript 对象。你可以看到单个存储器并不是一个杂乱无章的属性集合，而是由两个主要属性——`Todo`和`Done`——组织起来的。这就像拥有多个存储器，只是它们在一个对象中。你还会注意到每个存储属性都是一个`Immutable.js`数据结构。之所以这样做，是因为我们需要将传递给我们的减法函数的状态视为不可变的。这个库使得强制不可变性变得容易。

存储器状态发生的状态转换将被分为两个减法函数。实际上，这两个函数映射到存储器的两个初始属性：`Todo`和`Done`。让我们首先看看`Todo`减法函数：

```js
import Immutable from 'immutable';

import initialState from '../initial-state';
import {
  UPDATE_INPUT_VALUE,
  CREATE_TODO,
  REMOVE_TODO
} from '../constants';

export default function Todo(state = initialState, action) {
  switch (action.type) {

    // When the "UPDATE_INPUT_VALUE" action is dispatched,
    // we set the "inputValue" key of the Immutable.Map.
    case UPDATE_INPUT_VALUE:
      return state.set('inputValue', action.payload);

    // When the "CREATE_TODO" action is dispatched,
    // we push the new item to the end of the
    // Immutable.List
    case CREATE_TODO:
      return state.set('todos',
        state.get('todos').push(Immutable.Map({
          title: action.payload
        }))
      );

    // When the "REMOVE_TODO" action is dispatched,
    // we delete the item at the given index from
    // the Immutable.List.
    case REMOVE_TODO:
      return state.set('todos',
        state.get('todos').delete(action.payload));
    default:
      return state;
  }
}
```

这里使用的 `switch` 语句应该看起来很熟悉——这是我们一直在本书中实现存储的模式。实际上，这个函数就像一个存储，有两个主要区别。第一个区别是它是一个函数而不是一个类。这意味着我们不是设置状态属性值，而是返回新状态。第二个区别是 Redux 处理监听存储和调用此 reducer 函数的机制。使用类时，我们必须自己编写大量这样的代码。

### 注意

确保这些 reducer 函数不修改状态参数非常重要。这就是为什么我们使用 `Immutable.js` 库——使通过创建新数据来转换现有状态变得更容易。对于转换 Redux 存储状态来说，使用 `Immutable.js` 不是必需的，但它确实有助于代码简洁性。

现在让我们看看 `Done` reducer 函数：

```js
import Immutable from 'immutable';

import initialState from '../initial-state';
import { CREATE_DONE } from '../constants';

export default function Done(state = initialState, action) {
  switch (action.type) {

    // When the "CREATE_DONE" action is dispatched,
    // we insert the new item into the beginning
    // of the Immutable.List.
    case CREATE_DONE:
      return state.set('done',
        state.get('done')
          .insert(0, Immutable.Map(action.payload))
      );

    // Nothing to do, return the state "as-is".
    default:
      return state;
  }
}
```

我们几乎完成了我们的 Redux 存储。到目前为止，我们有两个 reducer 函数，每个函数都在它们自己的模块中。我们需要使用 `combineReducers()` 和 `createStore()` 将它们结合起来。现在让我们看看我们的存储模块：

```js
import { combineReducers, createStore } from 'redux';

import initialState from './initial-state';
import Todo from './reducers/todo.js';
import Done from './reducers/done.js';

export default createStore(combineReducers({
  Todo,
  Done
}), initialState);
```

如你所见，`combineReducers()` 函数创建了一个新的函数。这是维护应用程序状态的主要 reducer 函数。所以，与需要处理将动作发送到多个存储的典型 Flux 分发器不同，Redux 动作被发送到这个单一存储，并且我们的 reducer 函数会相应地被调用。

## Redux 动作

如你所知，动作和动作创建者之间有一个区别。动作是将要发送到各个 Flux 存储的负载，而动作创建者负责创建动作负载，然后将它们发送到分发器。在 Redux 中，动作创建者函数略有不同，因为它们只创建动作负载，并不直接与分发器通信。

在我们实现视图组件时，将在下一节中看到动作创建者是如何被调用的。但就目前而言，这是我们的动作模块的样子：

```js
import {
  CREATE_TODO,
  CREATE_DONE,
  REMOVE_TODO,
  UPDATE_INPUT_VALUE
} from './constants';

// Creates a new Todo item. The "payload" should
// be an object with a "title" property.
export function createTodo(payload) {
  return {
    type: CREATE_TODO,
    payload
  };
}

// Creates a new Done item. The "payload" should
// be an object with a "title" property.
export function createDone(payload) {
  return {
    type: CREATE_DONE,
    payload
  };
}

// Removes the todo and the given "payload" index.
export function removeTodo(payload) {
  return {
    type: REMOVE_TODO,
    payload
  };
}

// Updates the "inputValue" state with the given
// "payload" string value.
export function updateInputValue(payload) {
  return {
    type: UPDATE_INPUT_VALUE,
    payload
  };
}
```

这些函数只是返回存储将要分发的数据——它们实际上并不分发数据。例外情况是当涉及到异步操作时。在这种情况下，我们需要在异步值解决后实际分发动作。请参阅官方 Redux 文档，其中包含大量的异步动作示例。

## 渲染组件和分发动作

到目前为止，我们有了 Redux 存储和动作创建者函数。剩下要做的就是实现我们的 React 组件并将它们连接到存储。我们将从 `TodoList` 视图开始：

```js
import React from 'react';
import { Component } from 'react';
import { connect } from 'react-redux';

import {
  updateInputValue,
  createTodo,
  createDone,
  removeTodo
} from '../actions';

class TodoList extends Component {
  constructor(...args) {
    super(...args);
    this.onClick = this.onClick.bind(this);
    this.onKeyUp = this.onKeyUp.bind(this);
    this.onChange = this.onChange.bind(this);
  }

  render() {

    // The relevant state from the "todo" store
    // that we're rendering here.
    const { todos, inputValue } = this.props;

    // Renders an input for new todos, and the list
    // of current todos. When the user types
    // and then hits enter, the new todo is created.
    // When the user clicks a todo, it's moved to the
    // "done" array.
    return (
      <div>
        <h3>TODO</h3>
        <div>
          <input
            value={inputValue}
            placeholder="TODO..."
            onKeyUp={this.onKeyUp}
            onChange={this.onChange}
            autoFocus
          />
        </div>
        <ul>
          {todos.map(({ title }, i) =>
            <li key={i}>
              <a
                href="#"
                onClick={this.onClick.bind(null, i)}
              >{title}</a>
            </li>
          )}
        </ul>
      </div>
    );
  }

  // An active Todo was clicked. The "key" is the
  // index of the Todo within the store. This is
  // passed as the payload to the "createDone()"
  // action, and next to the "removeTodo()" action.
  onClick(key) {
    const { dispatch, todos } = this.props;

    dispatch(createDone(todos[key]));
    dispatch(removeTodo(key));
  }

  // If the user has entered some text and the
  // "enter" key is pressed, we use the
  // "createTodo()" action to create a new
  // item using the entered text. Then we clear
  // the input using the "updateInputValue()"
  // action, passing it an empty string.
  onKeyUp(e) {
    const { dispatch } = this.props;
    const { value } = e.target;

    if (e.which === 13 && value) {
      dispatch(createTodo(e.target.value));
      dispatch(updateInputValue(''));
    }
  }

  // The text input value changed - update the store.
  onChange(e) {
    this.props.dispatch(
    updateInputValue(e.target.value));
  }
}

// The props that get passed to this component
// from the store. We just need to convert the
// "Todo" Immutable.js structure to plain JS.
function mapStateToProps(state) {
  return state.Todo.toJS();
}

// Exports the "connected" version of the
// component that's connect to the Redux store.
export default connect(mapStateToProps)(TodoList);
```

关于这个模块的关键点是它不是导出的组件类。相反，我们使用来自`react-redux`包的`connect()`函数。这个函数将 Redux 存储与这个视图连接起来。存储中的状态通过`mapStateToProps()`函数传递，该函数决定了 React 组件属性是如何分配的。在这种情况下，我们只需要将`Immutable.js`结构转换成普通的 JavaScript 对象。

事件处理器的缺点是我们需要在构造函数中绑定它们的上下文，因为 React 不会自动绑定 ES2015 风格组件的上下文。处理器需要访问`this.props`，因为它包含用于将我们的动作数据派发到存储的`dispatch()`函数，以及用于构建动作负载的存储数据。现在让我们看看`DoneList`组件：

```js
import React, { Component } from 'react';
import { connect } from 'react-redux';

class DoneList extends Component {
  render() {

    // The "done" array is the only state we need
    // from the "done" store.
    const { done } = this.props;

    // We want to display these items
    // as strikethrough text.
    constitemStyle = {
      textDecoration: 'line-through'
    }

    // Renders the list of done items, with
    // the "itemStyle" applied to each item.
    return (
      <div>
        <h3>DONE</h3>
        <ul>
          {done.map(({ title }, i) =>
            <li key={i} style={itemStyle}>{title}</li>
          )}
        </ul>
      </div>
    );
  }
}

// The props that get passed to this component
// from the store. We just need to convert the
// "Done" Immutable.js structure to plain JS.
function mapStateToProps(state) {
  return state.Done.toJS();
}

// Exports the "connected" version of the
// component that's connect to the Redux store.
export default connect(mapStateToProps)(DoneList);
```

正如你所见，这与`TodoList`组件的工作方式非常相似。实际上，这些组件与同一应用的 Alt 实现相比变化不大。最后一步是将这两个组件与 Redux 存储连接起来，这可以通过使用`Provider`组件来完成：

```js
import React from 'react';
import { render } from 'react-dom';
import { Provider } from 'react-redux';

import store from './store';
import TodoList from './views/todo-list';
import DoneList from './views/done-list';

// Renders the "TodoList" and the "DoneList"
// components. The "Provider" component is
// used to connect the store to the components.
// When the store changes state, the children
// of "Provider" are re-rendered.
render(
  <Provider store={store}>
    <div>
      <TodoList/>
      <DoneList/>
    </div>
  </Provider>,
  document.getElementById('app')
);
```

# 摘要

在本章中，你学习了如何利用 Flux 库。特别是，我们查看了两款流行的库，这些库可以用来实现 Flux 架构。

我们在本章的开头进行了一次讨论，主要内容是对 Flux 的基本原则的回顾，以及我们在本书的前几章中如何实现这些原则。然后我们讨论了实现 Flux 的一些痛点——如单例派发器、重复的动作代码和存储模块的分区。这些都是`Alt.js`或 Redux 等库可以为我们解决的问题。

然后，我们继续使用`Alt.js` Flux 库实现了一个简单的待办事项应用。Flux 背后的想法是在后台自动化所有相关的 Flux 组件的实现，同时自动化一些典型的繁琐实现工作。之后，我们将注意力转向 Redux 库。Redux 不太关心严格遵循 Flux 模式。相反，Redux 追求简洁，同时借鉴了一些重要的 Flux 思想，如单向数据流。

在下一章中，我们将介绍任何 Flux 架构的两个非常重要的方面——功能测试和性能测试。
