# TypeScript 函数式编程库目录

在本附录中，您将找到按以下类别分组的与 TypeScript 兼容的函数式编程库列表：

+   **函数式编程**：通用函数式编程工具，包括 `compose` 函数

+   **类别论**：提供代数数据类型实现的库

+   **懒惰性**：提供用于实现懒惰评估的库

+   **不可变性**：提供用于实现不可变数据结构的库

+   **光学**：提供函数光学和透镜实现的库。

+   **函数式响应式编程**：通用、函数式响应式编程工具，例如观察者

+   **其他**：不专注于函数式编程，但高度受其原则影响的库

# 函数式编程

以下库允许我们在 TypeScript 中利用不可变性：

| **库** | **描述** | **链接** |
| --- | --- | --- |
| Ramda | 一个实用的 JavaScript 程序员纯函数式库。 | [`github.com/ramda/ramda`](https://github.com/ramda/ramda) |
| `fp-ts` | TypeScript 应用程序的纯函数式编程工具 | [`github.com/gcanti/fp-ts`](https://github.com/gcanti/fp-ts) |
| Underscore | 包含一些函数式编程辅助函数的辅助函数集合 | [`github.com/jashkenas/underscore`](https://github.com/jashkenas/underscore) |
| Lodash | 包含一些函数式编程辅助函数的辅助函数集合 | [`github.com/lodash/lodash`](https://github.com/lodash/lodash) |
| `wu.js` | ES6 迭代器的更高阶函数。 | [`github.com/fitzgen/wu.js/`](https://github.com/fitzgen/wu.js/) |

# 类别论

以下库允许我们在 TypeScript 中利用不可变性：

| **库** | **描述** | **链接** |
| --- | --- | --- |
| Ramda-fantasy | 与 Fantasyland 规范兼容的代数数据类型，易于与 Ramda.js 集成。 | [`github.com/ramda/ramda-fantasy`](https://github.com/ramda/ramda-fantasy) |
| `io-ts` | TypeScript 兼容的 IO 解码/编码的运行时类型系统 | [`github.com/gcanti/io-ts`](https://github.com/gcanti/io-ts) |
| Funfix | Funfix 是一个用于 JavaScript、TypeScript 和 Flow 的函数式编程类型类和数据类型的库。 | [`github.com/funfix/funfix`](https://github.com/funfix/funfix) |

# 懒惰性

以下库允许我们在 TypeScript 中利用不可变性：

| **库** | **描述** | **链接** |
| --- | --- | --- |
| `Lazy.js` | Lazy.js 是一个 JavaScript 的函数式实用库，类似于 Underscore 和 Lodash，但底层有一个懒惰引擎，力求尽可能少地工作，同时尽可能灵活。 | [`github.com/dtao/lazy.js/`](https://github.com/dtao/lazy.js/) |
| Transducers-js | JavaScript 的高性能转换器实现。 | [`github.com/cognitect-labs/transducers-js`](https://github.com/cognitect-labs/transducers-js) |

# 不可变性

以下库让我们能够在 TypeScript 中利用不可变性：

| **库** | **描述** | **链接** |
| --- | --- | --- |
| `Immutable.js` | 为 JavaScript 提供的不可变持久数据集合，可提高效率和简洁性。 | [`github.com/facebook/immutable-js`](https://github.com/facebook/immutable-js) |
| Immer | Immer 是一个小巧的包，允许你以更方便的方式处理不可变状态。它基于写时复制机制。 | [`github.com/mweststrate/immer`](https://github.com/mweststrate/immer) |

# 光学和镜头

以下库让我们能够在 TypeScript 中利用不可变性：

| **库** | **描述** | **链接** |
| --- | --- | --- |
| `monocle-ts` | 函数式光学：Scala monocle 到 TypeScript 的（部分）移植。 | [`github.com/gcanti/monocle-ts`](https://github.com/gcanti/monocle-ts) |
| `lens.ts` | 一个带有属性代理的 TypeScript 镜头实现。 | [`github.com/utatti/lens.ts`](https://github.com/utatti/lens.ts) |
| Lenses | 一个小型函数式镜头库，旨在保持小巧，无依赖，类型强大且精确。它受到 F# 中的 Aether 的启发。 | [`github.com/atomicobject/lenses`](https://github.com/atomicobject/lenses) |
| `Lenticular.ts` | JavaScript/TypeScript 中功能镜头的实现。 | [`github.com/tomasdeml/lenticular.ts`](https://github.com/tomasdeml/lenticular.ts) |

# 函数式响应式编程

以下库让我们能够在 TypeScript 中利用响应式编程：

| **库** | **描述** | **链接** |
| --- | --- | --- |
| RxJS | JavaScript 的响应式编程库。 | [`github.com/ReactiveX/rxjs`](https://github.com/ReactiveX/rxjs) |
| Xstream | 一个直观、小巧且快速的 JavaScript 函数式响应式流库。 | [`github.com/x-stream/xstream`](https://github.com/x-stream/xstream) |
| `Bacon.js` | 一个小巧的 JavaScript 函数式响应式编程库。 | [`github.com/baconjs/bacon.js/`](https://github.com/baconjs/bacon.js/) |

# 其他

以下库让我们能够在 TypeScript 中利用不可变性：

| **库** | **描述** | **链接** |
| --- | --- | --- |
| React | 一个受函数式编程原则高度影响的用户界面开发库。 | [`github.com/facebook/react`](https://github.com/facebook/react) |
| Redux | Redux 是 JavaScript 应用程序的状态容器，并高度受到函数式编程原则的影响。 | [`github.com/reduxjs/redux`](https://github.com/reduxjs/redux) |
| `Cycle.js` | 一个受函数式响应式编程高度影响的用户界面开发库。 | [`github.com/cyclejs/cyclejs`](https://github.com/cyclejs/cyclejs) |
| Mobx | 一个受函数式响应式编程高度影响的用户界面开发库。 | [`github.com/mobxjs/mobx`](https://github.com/mobxjs/mobx) |

# 摘要

本附录为您提供了对一些流行的函数式编程和函数式响应式编程库的快速参考。这些库，连同我们在本书中之前学到的技术和原则，应该为您提供创建多个现实世界函数式编程应用所需的一切。
