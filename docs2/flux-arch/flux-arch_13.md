# 第十三章：测试与性能

我们希望我们应用程序的架构尽可能优秀。虽然这样说可能显得有些荒谬，但确实有必要时不时地重复这一点，以提醒我们使用 Flux 所做的工作有可能决定应用程序的成功与否。我们拥有的最佳工具是单元测试和性能测试。这两个活动同等重要。功能正确但速度极慢是失败。速度极快但充满错误也是失败。

实施成功的测试的一个巨大贡献因素是关注相关的内容。在本章中，我们将花时间思考对于 Flux 架构来说，哪些是重要的测试——从功能和性能两个角度来看。鉴于 Flux 对于社区来说相对较新，这一点尤为重要。我们将关注特定的 Flux 组件，并为它们设计一些单元测试。然后，我们将思考基准测试底层代码与性能测试端到端场景之间的区别。

# Hello Jest

当涉及到为 JavaScript 代码编写有效的单元测试时，Jasmine 是广泛接受的选择工具。对于 Jasmine 的附加工具并不缺乏，这使得测试几乎任何东西和使用任何工具运行测试成为可能。例如，使用任务运行器（如 Grunt 或 Gulp）来运行测试，以及与项目相关的其他各种构建任务是常见的做法。

Jest 是由 Facebook 开发的一个单元测试工具，它借鉴了 Jasmine 的优点，并增加了新的功能。在项目中运行 Jest 也同样简单。例如，依赖于 Webpack 的项目通常依赖于 NPM 脚本来执行各种任务，而不是使用任务运行器。使用 Jest 就可以轻松做到这一点，正如我们接下来将要看到的。

Jest 有三个关键方面可以帮助我们测试 Flux 架构：

+   Jest 提供了一个虚拟化的 JavaScript 环境，包括 DOM 接口

+   Jest 会启动多个工作进程来运行我们的测试，这导致等待测试完成的时间更少，并且整体上加快了开发周期

+   Jest 可以为我们模拟 JavaScript 模块，这使得隔离代码单元以进行测试变得更加容易

让我们通过一个快速示例来开始。假设我们有一个我们想要测试的以下函数：

```js
// Builds and returns a string based
// on the "name" argument.
export default function sayHello(name = 'World') {
  return `Hello ${name}!`;
}
```

这应该很容易，我们只需要编写一个单元测试来检查预期的输出。让我们看看这个测试在 Jest 中的样子：

```js
// Tells Jest that we want the real "hello"
// module, not the mocked version.
jest.unmock('../hello');

// Imports the function we want to test.
import sayHello from '../hello';

// Your typical Jasmine test suite, test cases,
// and test assertions.
describe('sayHello()', () => {
  it('says hello world', () => {
    expect(sayHello()).toBe('Hello World!');
  });

  it('says hello flux', () => {
    expect(sayHello('Flux')).toBe('Hello Flux!');
  });
});
```

如果这看起来很像 Jasmine，那是因为它确实如此。实际上，Jasmine 被用于底层执行所有的测试断言。然而，在测试模块的顶部，你可以看到有一个 Jest 函数调用 `unmock()`。这告诉 Jest 我们不想要 `sayHello()` 函数的模拟版本。我们想要测试真实的东西。

### 注意

在将 Jest 设置成与 ES2015 模块导入一起工作方面，实际上涉及了很多调整。但在这里尝试解释它可能不太合适，我建议查看这本书附带源代码。现在，回到重要的事情上。

让我们创建一个`main.js`模块，它导入`sayHello()`函数并调用它：

```js
import sayHello from './hello';
sayHello();
```

我们为`sayHello()`函数创建的 Jest 单元测试将`sayHello()`函数进行了隔离。也就是说，我们不需要测试任何其他代码来测试这个函数。如果我们将这种逻辑应用到主模块，我们就不必依赖于实现`sayHello()`的代码。这正是 Jest 的模拟功能派上用场的地方。我们上次的测试关闭了 hello 模块的模拟功能，其中定义了`sayHello()`。这次，我们实际上想要模拟这个函数。让我们看看主测试看起来像什么：

```js
jest.unmock('../main');

// The "main" module is the real deal. The
// "sayHello()" function is a mock.
import '../main';
import sayHello from '../hello';

describe('main', () => {

  // We're expecting the "main" module to call
  // "sayHello()" exactly once. Since the "sayHello()"
  // function we've imported here is the same mock
  // called by main, we can verify this is indeed
  // what main is actually doing.
  it('calls sayHello()', () => {
    expect(sayHello.mock.calls.length).toBe(1);
  });
});
```

这次，我们确保`main.js`模块不会被 Jest 模拟。这意味着我们导入的`sayHello()`函数实际上是模拟版本。为了验证主模块是否按预期工作，尽管模块很简单，我们只需要验证`sayHello()`函数被调用了一次。

# 测试动作创建者

现在我们对 Jest 的工作原理有了大致的了解，是时候开始测试我们 Flux 架构的各个组件了。我们将从动作创建者函数开始，因为它们决定了进入系统的数据，并且是单向数据流的起点。我们想要测试两种类型的动作创建者。首先，我们有基本的同步函数，然后是异步函数。这两种类型的动作会导致非常不同的单元测试。

## 同步函数

动作创建者的任务是创建必要的有效负载数据并将其分发给存储。因此，为了测试这个功能，我们需要真实的动作创建者函数和一个模拟的分发器组件。记住，我们的想法是将组件作为单独的测试单元进行隔离——我们不想任何来自分发器代码的副作用影响测试结果。有了这些话，让我们看看动作创建者：

```js
import dispatcher from '../dispatcher';

export const SYNC = 'SYNC';

// Your typical synchronous action creator
// function. Dispatches an action with
// payload data.
export function syncFunc(payload) {
  dispatcher.dispatch({
    type: SYNC,
    payload
  });
}
```

这种类型的函数现在可能看起来很熟悉了。我们希望我们的单元测试验证这个函数是否正确地调用了`dispatch()`方法。现在让我们看看这个测试：

```js
// We want to test the real "syncFunc()" implementation.
jest.unmock('../actions/sync-func');

import dispatcher from '../dispatcher';
import { syncFunc } from '../actions/sync-func';

// The "dispatch()" method is mocked by
// Jest. We'll use it in the test to validate
// our action.
const { dispatch } = dispatcher;

describe('syncFunc()', () => {
  it('calls dispatch()', () => {

    // Calling "syncFunc()" should dispatch an
    // action. We can verify this by making sure
    // that the "dispatch()" was called.
    syncFunc('data');
    expect(dispatch.mock.calls.length).toBe(1);
  });

  it('calls dispatch() with correct payload', () => {
    syncFunc('data');

    // After calling "syncFunc()", we can get
    // argument information from the mock.
    const args = dispatch.mock.calls[1];
    const [ action ] = args;

    // Make sure the correct information was
    // passed to the dispater.
    expect(action).toBeDefined();
    expect(action.type).toBe('SYNC');
    expect(action.payload).toBe('data');
  });
});
```

这完全符合我们的预期。第一步是告诉 Jest 不要模拟 sync-func 模块中的内容，使用`unmock()`函数。Jest 仍然会模拟其他所有内容，包括分发器。所以当这个测试调用`syncFunc()`时，它实际上是调用模拟的分发器。当它这样做时，模拟记录了关于调用的信息，然后我们使用这些信息在我们的测试断言中确保一切按预期工作。

很简单，对吧？当我们需要模拟异步动作创建器函数时，事情会变得有点复杂，但我们在下一节会尝试简化一切。

## 异步函数

Jest 使我们能够通过模拟所有无关部分来轻松隔离给定单元测试应该测试的代码。一些事情可以通过 Jest 模拟生成器轻松处理。其他事情则需要我们的干预，正如你将在下面的例子中看到的那样。所以，让我们开始，看看我们试图测试的异步动作创建器函数：

```js
import dispatcher from '../dispatcher';
import request from '../request';

export const ASYNC = 'ASYNC';

// Makes a "request()" call (which is really
// just an alias for "fetch()") and dispatches
// the "ASYNC" action with the JSON response
// as the action payload.
export function asyncFunc() {
  return request('https://httpbin.org/ip')
    .then(resp => resp.json())
    .then(resp => dispatcher.dispatch({
      type: ASYNC,
      payload: resp
    }));
}
```

这个动作创建器向一个公共 JSON 端点发送请求，然后将响应作为动作负载来分发`ASYNC`动作。如果我们使用的`request()`函数看起来很像全局的`fetch()`函数，那是因为它就是那个函数。请求模块只是简单地导出它，如下所示：

```js
// We're exporting the global "fetch()" function
// so that Jest has an opportunity to mock it.
export default fetch;
```

这看起来似乎没有意义，但实际上并没有涉及任何开销。这就是我们能够轻松模拟代码中所有网络请求的方式。如果我们为单元测试模拟这个请求模块，这意味着我们的代码不会尝试连接到远程服务器。为了模拟这个模块，我们只需要在`__mocks__`目录中创建一个同名的模块，与`__tests__`目录并列。Jest 将找到这个模拟并替换导入时的真实模块。现在让我们看看模拟`request()`函数的源代码：

```js
// Exports the mocked version of the "request()"
// function our action creators use. In this case,
// we're emulating the "fetch()" function and the
// "Response" object that it resolves.
export default function request() {
  return new Promise((resolve, reject) => {
    process.nextTick(() => {
      resolve({
        json: () => new Promise((resolve, reject) => {

          // This is where we put all of our mock fetch
          // data. A given function should just test
          // the properties that it's interested in,
          // ignoring the rest.
          resolve({ origin: 'localhost' });
        })
      });
    });
  });
}
```

如果这段代码看起来有点糟糕，不要担心——它只限于这个位置。它所做的只是复制这个模块所替代的本地`fetch()`函数的接口（因为我们实际上不想获取任何东西）。这种方法棘手的地方在于，我们代码中的任何`request()`调用都将获得相同的解析值。但只要我们的代码可以忽略它不关心的属性，并且我们可以将测试数据保持到最小，这应该就没有问题。

到目前为止，我们已经有一个模拟的网络层，这意味着我们现在可以实施实际的单元测试了。让我们继续进行：

```js
jest.unmock('../actions/async-func');

// The "dispatcher" is mock while "asyncFunc()"
// is not.
import dispatcher from '../dispatcher';
import { asyncFunc } from '../actions/async-func';

describe('asyncFunc()', () => {

  // For testing asynchronous code that returns
  // promises, we use "pit()" in place of "it()".
  pit('dispatch', () => {

    // Once the call to "asyncFunc()" has resolved,
    // we can perform our test assertions.
    return asyncFunc().then(() => {
      // Collect stats about he mock
      // "dispatch()" method.
      const { calls } = dispatcher.dispatch.mock;
      const { type, payload } = calls[0][0];

      // Make sure that the asynchronous function
      // dispatches an action with the appropriate
      // payload.
      expect(calls.length).toBe(1);
      expect(type).toBe('ASYNC');
      expect(payload.origin).toBe('localhost');
    });
  });
});
```

关于这个测试，有两点需要注意。一是它使用`pit()`函数作为`it()`函数的直接替代。二是`asyncFunc()`函数本身返回一个承诺。这两个方面是 Jest 使编写异步单元测试变得如此简单的原因。这个例子中困难的部分不是测试，而是我们需要建立的基础设施，以便模拟像网络请求这样的东西。多亏了 Jest 为我们处理的一切，我们的单元测试代码实际上比其他情况下要小得多。

# 测试存储

在上一节中，我们使用了 Jest 来测试动作创建函数。这与测试任何其他 JavaScript 函数没有太大区别，只是 Flux 动作创建者需要以某种方式将它们创建的动作调度到存储上。Jest 通过自动模拟某些组件来帮助我们实现这一点，这无疑将帮助我们测试存储组件。

在本节中，我们将查看测试动作被调度到存储并存储发出更改事件的路径。然后，我们将思考初始存储状态以及这可能导致单元测试应该能够捕获的 bug。使所有这些工作涉及考虑实现可测试的存储代码，这是我们在这本书中还没有考虑过的。

## 测试存储监听器

存储组件可能很难与其他组件隔离。这反过来使得为存储编写单元测试变得困难。例如，一个存储通常会通过传递一个回调函数来向调度器注册自己。这个函数将根据传递给它的动作有效载荷改变存储的状态。这之所以是一个挑战，是因为它与调度器紧密耦合。

理想情况下，我们希望将调度器完全从单元测试中移除。我们只是在单元测试中测试我们的存储代码，所以不希望调度器中发生的任何事情干扰测试结果。这种情况发生的可能性很小，因为调度器实际上并没有太多事情要做。然而，与我们的所有 Flux 组件保持一致并尽可能完全隔离它们会更好。我们在上一节中看到了 Jest 如何帮助我们。我们只需要以某种方式将这个原则应用到存储上——在单元测试期间将它们与调度器解耦。

这是一个可能需要重新考虑我们编写存储代码的方式的情况——有时为了代码质量，需要稍作修改以便它既好又可测试。例如，我们通常注册到调度器的匿名函数变成了存储方法。这允许测试直接调用该方法，跳过整个调度机制，这正是我们想要的。现在让我们看看存储代码：

```js
import { EventEmitter } from '../events';
import dispatcher from '../dispatcher';
import { DO_STUFF } from '../actions/do-stuff';

var state = {};

class MyStore extends EventEmitter {
  constructor() {
    super();

    // Registers a method of this store as the
    // handler, to better support unit testing.
    this.id = dispatcher.register(this.onAction.bind(this));
  }

  // Instead of performing the state transformation
  // in the function that's registered with the
  // dispatcher, it just determines which store
  // method to call. This approach better supports
  // testability.
  onAction(action) {
    switch (action.type) {
      case DO_STUFF:
        this.doStuff(action.payload);
        break;
    }
  }

  // Changes the "state" of the store, and emits
  // a "change" event.
  doStuff(payload) {
    this.emit('change', (state = payload));
  }
}

export default new MyStore();
```

如您所见，`onAction()`方法已注册到调度器，并且每当有动作被调度时都会被调用。`doStuff()`方法将响应`DO_STUFF`动作发生的特定状态转换从`onAction()`方法中分离出来。这并不是严格必要的，但它为我们提供了另一个单元测试的目标。例如，我们本可以保留匿名回调函数不变，并直接针对`doStuff()`方法进行测试。然而，如果我们的测试使用来自调度器的相同类型的有效载荷数据调用`onAction()`，我们将获得更好的存储测试覆盖率。

精明的读者可能已经注意到，这个商店从不同于平常的地方导入`EventEmitter`——`../events`。我们有自己的事件模块吗？我们现在有了，并且它与上一节中`fetch()`函数的思路相同。我们提供了一个自己的模块，Jest 可以对其进行模拟。这是 Jest 模拟`EventEmitter`类的一个简单方法。我们当时太忙于思考分发器，以至于忘记了在测试中将我们的商店与事件发射器解耦。让我们看看事件模块，这样你就可以看到我们仍然在暴露大家所熟知和喜爱的“老式”`EventEmitter`：

```js
// In order to mock the Node "EventEmitter" API,
// we need to expose it through one of our own modules.
import { EventEmitter } from 'events';
export { EventEmitter as EventEmitter } ;
```

这意味着我们的商店继承的方法将由 Jest 进行模拟，这是完美的，因为现在我们的商店完全与其他组件代码隔离，我们可以使用模拟收集的数据来进行一些测试断言。现在让我们为这个商店实现单元测试：

```js
// We want to test the real store...
jest.unmock('../stores/my-store');

import myStore from '../stores/my-store';

describe('MyStore', () => {
  it('does stuff', () => {

    // Directly calls the store method that's
    // registered with the dispatcher, passing it
    // the same type of data that the dispatcher
    // would.
    myStore.onAction({
      type: 'DO_STUFF',
      payload: { foo: 'bar' }
    });

    // Get some of the mocked "emit()" call info...
    const calls = myStore.emit.mock.calls;
    const [ args ] = calls;

    // We can now assert that the store emits a
    // "change" event and that it has the correct info.
    expect(calls.length).toBe(1);
    expect(args[0]).toBe('change');
    expect(args[1].foo).toBe('bar');
  });
});
```

这种方法的优点在于，它非常接近数据在商店中流动的方式，但实际上并不依赖于其他组件来运行测试。测试数据以与实际分发器组件相同的方式进入商店。同样，我们知道商店通过测量模拟实现正确地发出了事件数据。这就是商店的责任结束的地方，测试的责任也是如此。

## 测试初始条件

当我们的 Flux 商店变得庞大且复杂后，我们很快就会学到，它们变得越来越难以测试。例如，如果一个商店响应的动作数量增加，那么我们想要测试的状态配置数量也会增加。为了帮助适应我们商店的单元测试，能够设置商店的初始状态将非常有帮助。让我们看看一个允许我们设置初始状态并响应几个动作的商店：

```js
import { EventEmitter } from '../events';
import dispatcher from '../dispatcher';
import { POWER_ON } from '../actions/power-on';
import { POWER_OFF } from '../actions/power-off';

// The initial state of the store...
var state = {
  power: 'off',
  busy: false
};

class MyStore extends EventEmitter {

  // Sets the initial state of the store to the given
  // argument if provided.
  constructor(initialState = state) {
    super();
    state = initialState;
    this.id = dispatcher.register(this.onAction.bind(this));
  }

  // Figure out which action was dispatched and call the
  // appropriate method.
  onAction(action) {
    switch (action.type) {
      case POWER_ON:
        this.powerOn();
        break;
      case POWER_OFF:
        this.powerOff();
        break;
    }
  }

  // Changes the power state to "on", if the power state is
  // currently "off".
  powerOn() {
    if (state.power === 'off') {
      this.emit('change', 
        (state = Object.assign({}, state, {
          power: 'on'
        }))
      );
    }
  }

  // Changes the power state to "off" if "busy" is false and
  // if the current power state is "on".
  powerOff() {
    if (!state.busy && state.power === 'on') {
      this.emit('change', 
        (state = Object.assign({}, state, {
          power: 'off'
        }))
      );
    }
  }

  // Gets the state...
  get state() {
    return state;
  }
}

export default MyStore;
```

这个商店响应`POWER_ON`和`POWER_OFF`动作。如果你查看处理这两个动作状态转换的方法，你会发现结果取决于当前状态。例如，开启商店需要商店已经关闭。关闭商店的限制更加严格——商店必须关闭且不能忙碌。这类状态转换需要使用不同的初始商店状态进行测试，以确保预期中的正常路径以及边缘情况都能按预期工作。现在让我们看看这个商店的测试：

```js
// We want to test the real store...
jest.unmock('../stores/my-store');

import MyStore from '../stores/my-store';

describe('MyStore', () => {

  // The default initial state of the store is
  // powered off. This test makes sure that
  // dispatching the "POWER_ON" action changes the
  // power state of the store.
  it('powers on', () => {
    let myStore = new MyStore();

    myStore.onAction({ type: 'POWER_ON' });

    expect(myStore.state.power).toBe('on');
    expect(myStore.state.busy).toBe(false);
    expect(myStore.emit.mock.calls.length).toBe(1);
  });

  // This test changes the initial state of the store
  // when it is first instantiated. The initial state
  // is now powered off, and we've also marked the
  // store as busy. This test makes sure that the
  // logic of the store works as expected - the state
  // shouldn't change, and no events are emitted.
  it('does not powers off if busy', () => {
    let myStore = new MyStore({
      power: 'on',
      busy: true
    });

    myStore.onAction({ type: 'POWER_OFF' });

    expect(myStore.state.power).toBe('on');
    expect(myStore.state.busy).toBe(true);
    expect(myStore.emit.mock.calls.length).toBe(0);
  });

  // This test is just like the one above, only the
  // "busy" property is false, which means that we
  // should be able to power off the store when the
  // "POWER_OFF" action is dispatched.
  it('does not powers off if busy', () => {
    let myStore = new MyStore({
      power: 'on',
      busy: false
    });
    myStore.onAction({ type: 'POWER_OFF' });

    expect(myStore.state.power).toBe('off');
    expect(myStore.state.busy).toBe(false);
    expect(myStore.emit.mock.calls.length).toBe(1);
  });
});
```

第二个测试可能是最有趣的，因为它确保由于商店的状态转换逻辑方式，没有事件被动作触发。

# 性能目标

是时候转换思路，思考测试我们 Flux 架构性能的问题了。测试特定组件的性能可能会很困难，原因与测试组件功能一样——我们必须将其与其他代码隔离开来。另一方面，我们的用户并不一定关心单个组件的性能——他们只关心整体的用户体验。

在本节中，我们将讨论我们在性能方面通过我们的 Flux 架构试图实现的目标。我们将从应用程序的用户感知性能开始，因为这是性能不佳架构中最重要的一面。接下来，我们将考虑测量我们 Flux 组件的原始性能。最后，我们将考虑为开发新组件时设定性能要求的好处。

## 用户感知性能

从用户的角度来看，我们的应用程序要么感觉响应迅速，要么运行缓慢。这种*感觉*被称为用户感知性能，因为用户实际上并没有测量完成某件事情所需的时间。一般来说，用户感知性能关乎挫败感阈值。每当我们需要等待某事时，挫败感就会增长，因为我们感觉无法掌控局势。我们无法做任何事情来让它加快速度，换句话说。

一种解决方案是分散用户的注意力。有时候，我们的代码必须处理某些事情，而没有办法缩短它所需的时间。在这个过程中，我们可以让用户了解任务进度。根据任务类型，我们甚至可能能够展示一些已经处理过的输出。另一个答案是编写高性能的代码，这是我们始终应该努力追求的。

用户感知性能对我们正在构建的软件产品至关重要，因为如果它被认为运行缓慢，也会被认为质量低下。最终，用户的意见才是最重要的——这就是我们衡量我们的 Flux 架构是否扩展到可接受水平的方法。用户感知性能的缺点是它无法量化，至少在细粒度层面上是这样。这就是我们需要工具来帮助我们衡量组件性能的地方。

## 测量性能

性能指标告诉我们代码中性能瓶颈的具体位置。如果我们知道性能问题的所在，那么我们就能更好地解决它们。从 Flux 架构的角度来看，例如，我们想知道动作创建者是否需要很长时间才能响应，或者存储是否需要很长时间来转换其状态。

有两种类型的性能测试可以帮助我们在开发 Flux 架构的过程中保持对任何性能问题的控制。第一种测试类型是分析，我们将在下一节中更详细地探讨这一点。第二种性能测试类型是基准测试。这种测试类型在较低级别进行，适用于比较不同的实现。

唯一的问题是——我们如何使性能测量成为日常生活的常态，我们可以用这些结果做什么？

## 性能要求

考虑到我们拥有必要的性能测试工具，似乎可以定义一些关于性能的要求。例如，如果有人正在实现一个存储库，我们能否引入一个性能要求，即存储库在发出更改事件时不能超过 x 毫秒？好处是我们可以对我们的架构的性能有合理的信心，甚至到组件级别。坏处是涉及的复杂性。

首先，新代码的开发会明显减慢，因为我们不仅要测试功能正确性，还要达到严格性能标准。这需要时间，而且回报可能微乎其微。假设我们花费大量时间提高某个组件的性能，因为它勉强满足要求。这意味着我们在做一些对用户来说无形的事情上白费力气。

这并不是说性能测试不能自动化，或者根本不应该进行。我们只是需要明智地选择在哪里投入我们的时间来测试 Flux 代码的性能。性能的最终决定者是用户，因此很难设定具体的要求，这些要求意味着“足够好”的性能，但尝试达到无人能察觉的优化性能却很容易浪费时间。

# 分析工具

通过网络浏览器提供的各种分析工具通常足以解决我们界面中出现的任何性能问题。这些工具包括构成我们 Flux 架构的组件。在本节中，我们将介绍浏览器开发者工具中发现的三个主要工具，我们将使用这些工具来分析我们的 Flux 架构。

首先是动作创建函数，特别是异步函数。然后我们将考虑我们的 Flux 组件的内存消耗。最后，我们将讨论 CPU 利用率。

## 异步操作

网络总是应用中最慢的一层。即使我们进行的 API 调用相对较快，它与其他 JavaScript 代码相比仍然很慢。如果我们的应用程序没有进行任何网络请求，它将会非常快。但它也不会很有用。一般来说，JavaScript 应用程序依赖于远程 API 端点作为它们的数据资源。

为了确保这些网络调用不会导致性能问题，我们可以利用浏览器开发者工具的网络分析器。这以极大的细节显示了任何给定请求正在做什么，以及它需要多长时间来完成。例如，如果服务器响应请求需要很长时间，这将在请求的时间线上反映出来。

使用这个工具，我们还可以看到在任何给定时刻未完成的请求数量。例如，可能在我们应用程序中有一个页面正在用请求猛烈地敲击服务器并使其不堪重负。在这种情况下，我们必须重新思考设计。在这个工具中查看的每个请求都允许我们深入到发起请求的代码。在 Flux 应用程序中，这应该始终是一个动作创建函数。有了这个工具，我们总是知道哪些动作创建函数从网络角度来看是有问题的，并且我们可以对它们采取措施。

## 存储内存

下一个可以帮助我们测试 Flux 架构性能的开发者工具是内存分析器。内存显然是我们必须小心处理的东西。一方面，我们必须考虑系统上运行的其它应用程序，避免占用过多内存。另一方面，当我们试图小心处理内存时，我们最终会频繁地进行分配/释放，触发垃圾回收器。很难确定一个组件应该使用的最大内存量。应用程序需要它需要的。

在 Flux 方面，我们最感兴趣的是内存分析器能告诉我们关于我们存储的信息。记住，随着应用程序的增长，我们很可能会在存储上遇到可扩展性问题，因为它们将不得不处理更多的输入数据。当然，我们也会想关注我们的视图组件消耗的内存，但最终是存储控制视图将消耗多少或多少内存。

内存分析器有两种方式可以帮助我们更好地理解 Flux 存储的内存消耗。首先，是内存时间线。这个视图显示了内存随时间分配/释放的情况。这很有用，因为它让我们看到内存是如何被使用的，就像用户与应用程序交互一样。其次，内存分析器允许我们拍摄当前内存分配的快照。这是我们确定正在分配的数据类型及其代码的方式。例如，通过快照，我们可以看到哪个存储占用了最多的内存。

## CPU 利用率

如前一个关于内存分析器的章节所示，频繁的垃圾回收可能会影响响应性。这是因为垃圾回收器会阻止其他 JavaScript 代码的运行。CPU 分析器实际上可以显示垃圾回收器从其他代码中夺走了多少 CPU 时间。如果很多，那么我们可以找到更好的内存策略。

然而，当我们分析 CPU 时，我们再次应该将注意力转向 Flux 架构的商店组件。简单的原因是这将带来最大的投资回报。我们可能面临的可扩展性问题集中在用于处理商店中动作负载的数据转换函数。除非这些函数足够高效以处理进入系统的数据，否则架构不会扩展，因为我们的代码过度利用了 CPU。有了这个，我们将把注意力转向基准测试对我们系统可扩展性至关重要的函数。

# 基准测试工具

在性能测试的谱系一端，是用户感知的性能。这就是我们的客户在抱怨卡顿的地方，而且确实很容易复现这个问题。这可能是因为视图组件、网络请求，或者是我们商店中某些导致用户体验不佳的问题。在谱系的另一端，我们有代码的原始基准测试，我们希望获得准确的计时，以确保我们使用的是最高效的实现。

在本节中，我们将简要介绍基准测试的概念，然后我们将展示一个使用 `Benchmark.js` 比较两种状态转换实现的示例。

## 基准测试代码

当我们基准测试我们的代码时，我们是在比较一种实现与另一种实现，或者我们可以比较三个或更多实现。关键是隔离实现与其他组件，并确保它们各自有相同的输入并产生相同的输出。在某种程度上，基准测试就像单元测试，因为我们有一个被隔离作为单元的代码块，并使用工具来测量和测试其性能。

在执行这类微基准测试时，一个挑战是准确的计时。另一个挑战是创建一个不受其他事物干扰的环境。例如，尝试在网页中运行 JavaScript 基准测试可能会面临来自 DOM 等事物的干扰。`Benchmark.js`处理获取我们代码最准确测量的繁琐细节。话虽如此，让我们跳入一个示例。

### 注意

与单元测试不同，基准测试不一定是我们希望保留和维护的东西。这会带来太多的负担，而且当有成百上千个基准测试时，基准测试的价值往往会降低。可能有一些例外，我们希望为了说明目的而在存储库中保留基准测试。但一般来说，一旦代码实现或现有代码的性能得到改善，基准测试可以安全地被丢弃。

## 状态转换

当我们尝试扩展系统时，Flux 存储内部发生的状态转换有可能使系统停止运行。正如你所知，我们架构中的其他 Flux 组件扩展性良好。问题在于增加的请求量和数据量。需要高效执行转换这些数据的基本函数。我们可以使用 `Benchmark.js` 这样的工具为与存储数据一起工作的代码构建基准测试。以下是一个示例：

```js
import { Suite } from 'benchmark';

// The "setup()" function is used by each benchmark in
// the suite to create data to used within the test.
// This is run before anything is measured.
function setup() {

  // The "coll" array will be available in each
  // benchmark function because this source gets
  // compiled into the benchmark function.
  const coll = new Array(10000)
    .fill({
      first: 'First',
      last: 'Last',
      disabled: false
    });

  // Disable some of the items...
  for (let i = 0; i<coll.length; i += 10) {
    coll[i].disabled = true;
  }
}

new Suite()

  // Adds a benchmark that tests the "filter()"
  // function to remove disabled items and the
  // "map()" function to transform the string
  // properties.
  .add('filter() + map()', () => {
    const results = coll
      .filter(item => !item.disabled)
      .map(item => ({
        first: item.first.toUpperCase(),
        last: item.last.toUpperCase()
      }));
  }, { setup: setup })

  // Adds a benchmark that tests a "for..of" loop
  // to build the "results" array.
  .add('for..of', () => {
    const results = [];

    for (let item of coll) {
      if (!item.disabled) {
        results.push({
          first: item.first.toUpperCase(),
          last: item.last.toUpperCase()
        });
      }
    }
  }, { setup: setup })

  // Adds a benchmark that tests a "reduce()"
  // call to filter out disabled items
  // and perform the string transforms.
  .add('reduce()', () => {
    const results = coll
      .reduce((res, item) => !item.disabled ?
        res.concat({
          first: item.first.toUpperCase(),
          last: item.last.toUpperCase()
        }) : res);
  }, { setup: setup })

  // Setup event handlers for logging output...
  .on('cycle', function(event) {
    console.log(String(event.target));
  })
  .on('start', () => {
    console.log('Running...');
  })
  .on('complete', function() {
    const name = this.filter('fastest').map('name');
    console.log(`Fastest is "${name}"`);
  })
  .on('error', function(e) {
    console.error(e.target.error);
  })
  // Runs the benchmarks...
  .run({ 'async': true });
  // →
  // Running...
  // filter() x 1,470 ops/sec ±1.00% (86 runs sampled)
  // for..of x 1,971 ops/sec ±2.39% (81 runs sampled)
  // reduce() x 1,479 ops/sec ±0.89% (87 runs sampled)
  // Fastest is "for..of"
```

如你所见，我们只需将两个或更多基准函数添加到测试套件中，然后运行它。输出是特定性能数据，比较了各种实现。在这种情况下，我们正在过滤和映射一个包含 10,000 项的数组。从性能角度来看，`"for..of"`方法脱颖而出，是最佳选择。

基准测试的重要之处在于它可以很容易地排除错误的假设。例如，我们可能会假设因为`"for..of"`比其他实现表现更好，所以它自动是最好的选择。然而，这两种替代方案并不落后太多。所以，如果我们真的想使用 `reduce()` 实现功能，这样做可能没有扩展风险。

### 注意

这本书附带代码实现了一些技巧，以便使用 Babel 使此示例与 ES2015 语法兼容。如果你使用 Babel 对生产代码进行转换，这是一个特别好的主意，这样你的基准测试就能反映现实情况。同时，在 `package.json` 中添加一个 `npm bench` 脚本也很方便，以便轻松访问。

# 摘要

本章的重点是测试我们的 Flux 架构。我们采用两种类型的测试来完成这项工作：功能和性能。使用功能单元，我们验证构成我们的 Flux 架构的代码单元是否按预期运行。使用性能单元，我们验证代码是否达到预期的性能水平。

我们介绍了 Jest 测试框架，用于为我们的动作创建者和存储实现单元测试。然后，我们讨论了浏览器中可以帮助我们解决高级别性能问题的各种工具。这些都是以可感知的方式影响用户体验的类型。

我们以对代码进行基准测试来结束这一章。这是在低级别发生的事情，很可能与我们的存储的状态转换功能有关。现在，我们需要考虑 Flux 架构对整个软件开发生命周期的影响。
