# 第七章。不要重复造轮子-函数式响应式编程工具

在本章中，我们将介绍一些用于在“裸金属”JavaScript 之上构建的许多优秀工具中的一些，正如上一章中简要讨论的那样。JavaScript 不仅因为其作为核心语言的特性而有趣；浏览器 JavaScript 是一个生态系统，或者说可能是多个生态系统的家园。关于函数式响应式编程的工具，总体的提供代表了一个良好、健康和庞大的集市，与之相比，仅仅使用 JavaScript 进行所有 Web 开发看起来更像是一座大教堂。我们将从这个集市中取一小部分，理解到本章并不打算涵盖所有好的、有趣的或值得的东西。在集市中做到这一点是非常困难的！

我们将介绍的工具包括以下内容：

+   ClojureScript 和 Om

+   Bacon.js

+   Brython

+   Immutable.js

+   Jest

+   Fluxxor

我们将在这样一个章节中包含或不包含的一组工具，涉及到需要划定界限和做出判断。对于更全面的处理感兴趣的读者可以查看[`tinyurl.com/reactjs-complementary-tools`](http://tinyurl.com/reactjs-complementary-tools)上的链接汇编，并深入研究他们特定关注的工具。那里有很多东西，几乎可以满足任何目的的很多宝贝。

# ClojureScript

ClojureScript，也许是 Clojure 总体上，代表了软件和 Web 开发的一个重要分水岭。ClojureScript 通过示例证明，除了 JavaScript 之外，还可以在其他语言中拥有坚实的基础和开发环境，而这种开创性的语言是一种 Lisp 方言。（这或许很合适，因为它是两种最常用的最古老的编程语言之一。Lisp 在诞生时就很好，今天仍然是一种很好的语言。）此外，与 JavaScript 相比，Lisp 可能具有很大的优势，并且由于一些相同的原因而存在。JavaScript 是 Web 浏览器的语言，而 Lisp 是 Emacs 的语言。此外，Lisp 提供了一种原始的 JavaScript；在可以用 JavaScript 编程的 Web 浏览器出现之前，可以用 Lisp 编程的 Emacs 就已经存在了，而且任何人说 Lisp 比 JavaScript 更好的话几乎不会受到质疑。

有充分的理由表明，Lisp 而不是 Emacs 默认的键绑定，是导致在互联网上流传的“经典学习曲线”漫画中的经典 Emacs 学习曲线的原因：

![ClojureScript](img/B04148_07_01.jpg)

正如在前面的章节中建议的那样，每个人直接在 JavaScript 中编程的统一性可能会让位于美丽的多样性，或者一块拼布。在这个美丽的拼布中，JavaScript 可能仍然是卓越的，但 JavaScript 的卓越地位可能成为新的“裸金属”。我们可能会有一系列用于前端开发的高级语言和工具。再次引用 Alan Perlis 的话，“当一个语言需要关注无关紧要的事情时，它就是低级的。”基于这些理由，JavaScript 是低级的。

这些工具中的一些可能在好的部分方面比坏的部分更好。它们可能适用于前端工作，最终仍然会在 JavaScript 中执行。但它们也可能开启前端 Web 开发，其中新的开发人员不再被告知，“这是我们将使用的语言，这是语言的一些大部分，你应该尽量避免，因为它们从根本上是有毒的。”ECMAScript（JavaScript 的正式名称，与 Emacs 没有特别的联系）的新版本可能提供更好的功能集合，但在高级语言中工作仍然是可取的，因为它们提供了更好的生产工作和结果的平台。

ClojureScript 毫不犹豫地表示，可以在浏览器上运行一个良好的高级语言，这不仅对 Lisp 黑客是个好消息。这对每个人都是好消息。它展示了在其他高级语言中进行 Web 开发的可能性，并且可能会有一个更好的 Web 开发环境，减少了沥青坑的可能性。

ClojureScript 既可以用于客户端工作，也可以在 Node.js 上用于服务器端。*Hello, World!* 如下所示：

```js
(ns nodehello
  (:require [cljs.nodejs :as nodejs]))

(defn -main [& args]
  (println (apply str (map [\space "world" "hello"] [2 0 1]))))

(nodejs/enable-util-print!)
(set! *main-cli-fn* -main)

(comment
; Compile this using a command line like:

CLOJURESCRIPT_HOME=".../clojurescript/" \
  bin/cljsc samples/nodehello.cljs {:target :nodejs} \
  > out/nodehello.js

; Then run using:
nodejs out/nodehello.js

)
```

# Om

Om 是一个包装器，使 ReactJS 可用于 ClojureScript。除了 ClojureScript 通常很快之外，Om 的某个部分实际上比 JavaScript 快大约两倍。这种差异与识别变化有关，以便在 ReactJS 执行 DOM 更新时进行最佳和适当的更新。原因是 ReactJS 在其差异算法中（通过处理可变的 JavaScript 数据结构）必须执行深度比较，以查看（纯 JavaScript）合成虚拟 DOM 中的内容是否有变化。

与直接 DOM 操作相比，这仍然非常快，以至于对大多数 ReactJS 用户来说并不是瓶颈。但在 Om 中更快。原因是 ClojureScript 像一种良好的函数式编程语言一样具有不可变数据。你可以很容易地获得某物的突变副本，但你不能篡改原始副本或使访问原始副本的任何人受到影响。这意味着 Om 只需比较顶层引用而不深入数据结构的深度就足够了。这足以使 Om 比原始的 JavaScript 使用 ReactJS 更快。在 Om 中，*Hello, World!* 是这样写的：

```js
(ns example
  (:require [om.core :as om]
            [om.dom :as dom]))

(defn widget [data owner]
  (reify
    om/IRender
    (render [this]
      (dom/h1 nil (:text data)))))

(om/root widget {:text "Hello world!"}
  {:target (. js/document (getElementById "my-app"))})
```

# Bacon.js

请注意，仅讨论 ReactJS 和 Bacon.js 并不足以构成一个详尽的列表。提到另一个替代套件，微软已经尝试创建了 RxJS、RxCpp [Rx for C++]、Rx.NET 和 Rx*，适用于各种 JavaScript 框架和库，并且他们至少尝试为多种语言和多种 JavaScript 框架和库的优化版本创建了一个多语言友好的组合。实际上有很多可用的提供某种形式的函数式响应式编程。虽然大多数（在撰写本书时）Web 上的函数式响应式编程和 ReactJS 资源都是宝贵的，但也有一些不是。

安德烈·斯塔尔兹写道：

> *“所以你对学习这个叫做响应式编程的新东西感到好奇，特别是它的变体，包括 Rx、Bacon.js、RAC 等。”*
> 
> *学习它很难，缺乏好的材料使它更加困难。当我开始时，我试图寻找教程。我只找到了少数实用指南，但它们只是皮毛，从未解决围绕它构建整个架构的挑战。当你试图理解某个函数时，库文档通常没有帮助。我是说，老实说，看看这个：*
> 
> *Rx.Observable.prototype.flatMapLatest(selector, [thisArg])*
> 
> *通过将可观察序列的每个元素投影到一个新的可观察序列中，该新序列将元素的索引合并，然后将可观察序列转换为仅从最近的可观察序列产生值的可观察序列。*

我现在明白这句引语的意思，但那是因为我从其他沟通更好的资源中学到了。你正在阅读的这本书的目的之一是让好的文档变得更容易理解一些。

在开源社区中有一个著名的问题：你会买一个发动机盖被焊死的汽车吗？ReactJS 可以被描述为大多数人可以在不打开发动机盖的情况下驾驶的汽车。这并不是说 ReactJS 是闭源的，或者 Facebook 显示出任何使其更难阅读源代码的迹象。但举一个显著的例子，**指示性连续时间语义**是 Conal Elliott 对现在称为函数式反应式编程的东西的更好名称的第二次思考的一部分。无论一个人是否同意他对更好和更具描述性名称的建议，这位领军人物的第二次思考可能非常有洞察力和启发性。而且对于 ReactJS，如果它工作正常，一个新手程序员可以得到与 Calvin 的父亲（一位专利律师！）在 Calvin and Hobbes 中给出的相同解释，当 Calvin 问一个灯或者吸尘器是如何工作的时候——*这是魔术！*看着一个新手的问题，“连续时间是如何处理的？”回答是*这是魔术！*“你怎么能够丢弃和重新创建 DOM 每一次？”——*这是魔术！*；“但是 ReactJS 如何在非 JIT iPhone 上实现惊人的 60fps？”——*这是魔术！*

函数式反应式编程描述了需要完成的某些任务，比如适当处理事件流，但 ReactJS 的文档似乎没有解释如何处理这些任务，因为这个责任被转移到了 ReactJS；*这是魔术！*

Bacon.js 不仅没有焊死发动机盖，而且还期望你在发动机盖下进行调整。Bacon.js 似乎更接近基本函数式反应式编程的根源。一些打算在 ReactJS 中工作的程序员可能会发现用 Bacon.js“举重”一点并用 Bacon.js 加强自己是有利可图的。函数式反应式编程的一个重要领域是处理事件流的发射，就 ReactJS 而言，*这是魔术！*

在 Bacon.js 中，事实上并不是魔术，所有这些都是在你没有动手的情况下完成的；这是程序员需要解决的问题，并且他们有很好的工具来做到这一点。基于这些理由，使用 ReactJS 可能有助于为开发人员打下坚实的反应式编程基础。如果 ReactJS 的卖点是它是一个优化工具，可以在利用函数式反应式编程的优势的同时允许良好的用户界面工作，那么 Bacon.js 的卖点是它是一个在 JavaScript 中优化的工具，可以在理论和实践中（学习和）执行扎实的函数式反应式编程。

ReactJS 和 Bacon.js 之间的区别似乎不是挖掘出一个框架比另一个更好的问题。相反，这更多地是关于审视你想要做和实现的事情，认识到 ReactJS 和 Bacon.js（除了是值得竞争对手之外）在它们真正擅长的不同领域，并决定你的工作更像是 ReactJS 的甜蜜点还是 Bacon.js 的甜蜜点。此外，关于甜蜜点的话题，Bacon.js（不像 ReactJS）有一个让你垂涎欲滴的名字，而`~`函数操作符在参考文献中被称为“bacon”。

# Brython - 一个 Python 浏览器实现

Brython ([`brython.info`](http://brython.info))是一个浏览器和 Python 实现，是另一个用 Python 编程浏览器的替代方案的例子，虽然将 Brython 仅仅称为实验性的有点不公平，但也不一定适合称其为成熟的——至少不像 ClojureScript 具有一定的成熟度。ClojureScript 的发展足够好，可以基本上替代前端开发人员真正希望使用 Lisp 而不是 JavaScript 的"裸金属"JavaScript。换句话说，除非我们谈论一些性能关键的问题或可能的特殊情况，否则 ClojureScript 专家不会回答"我在 ClojureScript 中怎么做这个？"这样的问题，而是会说"对于这种问题直接使用 JavaScript。" Brython 被包含在这里并不是因为 Python 是唯一的非 JavaScript 语言，可以用于前端开发，而是作为一个例证，即 ClojureScript 中的 Lisp 并不是在前端 Web 开发方面的基本例外，而可能是许多例外中的第一个。

Brython 旨在征服世界。它的主页大胆宣布："Brython 旨在取代 JavaScript 成为 Web 的脚本语言"，也许永远无法达到这个相当天真的目标。Brython 加载时间长，加载后运行速度慢。也许最好使用其中一个 Python 到 JavaScript 编译器（更接近 ClojureScript），但 Brython 确实提供了相当多 Python 的优点，也许有一天会被视为重要。然而，我认为试图成为下一个 JavaScript 并取代其他渲染 JavaScript 的转译器是愚蠢的。

在 Brython 中，征服世界的目标也导致了一个盲点：未能看到与其他语言编写的工具进行互操作的重要性。但好消息是，Brython 或其他 Python 到 JavaScript 的方法可能是重要的，而无需成为"统治所有语言的一种语言"。Python 并不是唯一可用的后端语言，但它是一个很好的选择，并且有充分的理由让 Python 的良好实现成为前端 Web 开发中可以有利用的多种语言拼贴拼图中的有价值的一部分。

此外，使用 ReactJS 编写至少一个*Hello,World!*程序在 Brython 中也很容易实现。在将 Brython 和 ReactJS 放在同一页后，运行*Hello, World!*程序，首先是 JavaScript（不是 JSX）被注释掉，然后是 Python 代码通过 Brython 在浏览器中调用 React：

```js
<!DOCTYPE html>
<html>
  <head>
    <title>Hello, Brython!</title>
    <script src="img/brython.js"></script>
    <script src="img/react.js"></script>
  </head>
  <body onload="brython()">
    <p>Hello, static world!</p>
    <div id="dynamic"></div>
    <!--
      <script type="text/javascript">
        React.render(
          React.createElement('p', null,
          'Hello, JavaScript world!'),
          document.getElementById('dynamic')
        );
      </script>
      -->
    <script type="text/python3">
      from browser import document, window

      window.React.render(window.React.createElement(
        'p', None, 'Hello, Python world!'),
        document['dynamic']);

    </script>
  </body>
</html>
```

这里显示了以下内容：

![Brython - 一个 Python 浏览器实现](img/B04108_07_02.jpg)

请注意，整个第一个脚本标签和内容，而不仅仅是其中的 JavaScript，都在 HTML 注释中。这意味着第一个（JavaScript）脚本在这里仅用于清晰显示，并不活跃，第二个（Python）脚本才是运行并显示其消息的脚本。

第二个脚本很有趣；包含的 Python 代码（除了消息之外）与被注释掉的 JavaScript 文本相当，并且执行相同的操作。这是一个相当了不起的成就，特别是当 Brython 成功地实现了 Python 3.x 分支中的大多数功能时。即使 Brython 被用于一个项目并被认为不是正确的解决方案，它仍然是一个成就。

在某种意义上，Brython 在这里被提出作为一个可能性的例子，而不是任何意义上值得关注的唯一成员。重点不是特别是 Python 可以用于前端开发；而是 ClojureScript Lisp 可能不是除 JavaScript 之外唯一可用于前端开发的其他语言。

# Immutable.js - 永久保护免受更改

Immutable.js 的主页位于 [`facebook.github.io/immutable-js`](http://facebook.github.io/immutable-js)，标语是 **Immutable collections for JavaScript**，最初是为持久性而命名的。然后它经历了一个更快地注册不可变的名称更改。Immutable.js 填补了 JavaScript 作为函数语言的空白，并为集合提供了更多功能友好的数据结构（这是它创建的目的）。

它为不可变的集合提供了数据结构支持。它们优雅地支持创建修改后的副本，但始终是副本发生了变化，而不是原始数据。尽管这更多是一个小点，但它大大减少了“防御性复制”和相关解决方法的需求，以便在有多个程序员的地方不使用不可变数据。原始代码可能会使用您想要的数据结构的不同和修改后的副本，但您保留为参考的副本保证完全不受影响。该库旨在支持其他便利功能，比如轻松地转换为和从基本的 JavaScript 数据结构。

然而，Immutable.js 的数据结构不仅是不可变的；在某些方面它们也是懒惰的，文档清楚地标记了应用程序的哪些方面是急切的。 （作为提醒，懒惰的数据结构在需要时以按需打印的方式处理，而急切的操作是一次性和前置的）。此外，Immutable.js 设施中还包含了某些函数习语。例如，它提供了一个 `.take(n)` 方法。它以经典的函数方式返回列表的前 *n* 个项目。其他函数标准，如 `map()`、`filter()` 和 `reduce()`，也是可用的。总的来说，运行时复杂度和计算机科学家合理要求的一样好。

Immutable.js 提供了几种数据类型；其中包括以下内容（本表和下一个表中的描述部分基于官方文档）：

| Immutable.js 类 | 描述 |
| --- | --- |
| `Collection` | 这是 Immutable.js 数据结构的抽象基类。不能直接实例化。 |
| `IndexedCollection` | 代表特定顺序中的索引值的集合。 |
| `IndexedIterable` | 这是一个可迭代对象，具有支持一些类似数组接口功能的索引数字键，比如 `indexOf()`（可迭代对象是可以像列表一样迭代的东西，但在内部可能是也可能不是列表）。 |
| `IndexedSeq` | 支持有序索引值列表的 `Seq`。 |
| `Iterable` | 一组（键和索引）值，可以进行迭代。这个类是所有集合的基类。 |
| `KeyedCollection` | 代表键值对的集合。 |
| `KeyedIterable` | 一个与每个可迭代对象相关联的离散键的可迭代对象。 |
| `KeyedSeq` | 代表键值对的序列。 |
| `List` | 一个有序的集合，有点像（密集的）JavaScript 数组。 |
| `Map` | 一个键值对的可迭代对象。 |
| `OrderedMap` | 一个地图，除了执行地图的所有操作之外，还保证迭代会按照设置的顺序产生键。 |
| `OrderedSet` | 一个集合，除了保证迭代会按照设置的顺序产生值之外，还可以执行集合的所有操作。 |
| `Record` | 一个产生具体记录的类。在概念上，这与其他记录不同。其他元素在概念上是“杂物”的集合，可能是具有类似结构的对象。`Record` 类更接近于学校里遇到的记录，其中一个记录类似于数据库表中的一行，而结果集或表更像容器对象。 |
| `Seq` | 一个值的序列，可能有可能没有由具体数据结构支持。 |
| `Set` | 一组唯一的值。 |
| `SetCollection` | 一组没有键或索引的值。 |
| `SetIterable` | 代表没有键或索引的值的可迭代对象。 |
| `SetSeq` | 代表一组值的序列。 |
| `Stack` | 一个标准的堆栈，带有`push()`和`pop()`。语义总是指向第一个元素，不像 JavaScript 数组。 |

### 注意

`Record`与其他元素略有不同；它类似于满足某些条件的 JavaScript 对象。其他元素是相关的容器类，提供对一些对象集合的功能访问，并且通常具有类似的方法列表。

列表的方法，以一个例子为例，包括以下内容：

| Immutable.List 方法 | 描述 |
| --- | --- |
| `asImmutable` | 一个函数，接受一个（可变的）JavaScript 集合，并渲染一个 Immutable.js 集合。 |
| `asMutable` | 这是对“不是最佳”编程的让步。基于 Immutable.js 集合处理变化的正确方法是使用 Mutations。即使`asMutable`可用，也应该只在函数内部使用，永远不要公开或返回。 |
| `butLast` | 这会产生一个类似的新列表，但它缺少最后一个条目。 |
| `concat` | 连接（即追加）两个相同类型的可迭代对象。 |
| `contains` | 如果值存在于此列表中，则为真。 |
| `count` | 返回此列表的大小。 |
| `countBy` | 使用分组函数对列表的内容进行分组，然后按分组器分区发出键的计数。 |
| `delete` | 创建一个没有此键的新列表。 |
| `deleteIn` | 从键路径中删除一个键，这允许从外部集合到内部集合的遍历，就像文件系统路径允许从外部目录到内部目录的遍历一样。 |
| `entries` | 作为`key`，`value`元组的列表迭代。 |
| `entrySeq` | 创建一个新的键值元组的`IndexedSeq`。 |
| `equals` | 这是完全相等的比较。 |
| `every` | 如果断言对此列表中的所有条目都为真，则为真。 |
| `filter` | 返回提供的断言为真的列表元素。 |
| `filterNot` | 返回提供的断言返回 false 的列表元素。 |
| `find` | 返回满足提供的断言的值。 |
| `findIndex` | 返回提供的断言第一次为真的索引。 |
| `findLast` | 返回最后一个满足提供的断言的元素。 |
| `findLastIndex` | 返回提供的断言最后为真的索引。 |
| `first` | 列表中的第一个值。 |
| `flatMap` | 这将潜在的列表列表展平成一个深度为一的列表。 |
| `flatten` | 这会展平嵌套的可迭代对象。 |
| `forEach` | 对列表中的每个条目执行一个函数。 |
| `fromEntrySeq` | 返回任何键值元组的可迭代对象的`KeyedSeq`。 |
| `get` | 返回键的值。 |
| `getIn` | 遍历键路径（类似于文件系统路径）以获取键（如果可用）。 |
| `groupBy` | 将列表转换为由提供的分组函数分组的列表的列表。 |
| `has` | 如果键存在于此列表中，则为真。 |
| `hashCode` | 为此集合计算哈希码。适用于哈希表。 |
| `hasIn` | 如果集合的等效于文件系统遍历找到了问题的值，则为真。 |
| `indexOf` | 例如，`Array.prototype.indexOf`中的第一个出现的索引。 |
| `interpose` | 在单个列表条目之间插入分隔符。 |
| `interleave` | 将提供的列表交错成一个相同类型的列表。 |
| `isEmpty` | 这告诉这个可迭代对象是否有值。 |
| `isList` | 如果值是列表，则为真。 |
| `isSubset` | 如果比较可迭代对象中的每个值都在此列表中，则为真。 |
| - `isSuperset` | 如果此列表中的每个值都在比较可迭代对象中，则为 true。 |
| - `join` | 使用分隔符（默认）将条目连接成字符串。 |
| - `keys` | 此列表的键的迭代器。 |
| - `keySeq` | 返回此可迭代对象的 `KeySeq`，丢弃所有值。 |
| - `last` | 列表中的最后一个值。 |
| - `lastIndexOf` | 返回此列表中可以找到值的最后索引。 |
| - `List` | 列表的构造函数。 |
| - `map` | 通过映射函数返回一个新的列表。 |
| - `max` | 返回此集合中的最大值。 |
| - `maxBy` | 这类似于 max，但具有更精细的控制。 |
| - `merge` | 将可迭代对象或 JavaScript 对象合并为一个列表。 |
| - `mergeDeep` | 合并的递归模拟。 |
| - `mergeDeepIn` | 从给定的键路径开始执行深度合并。 |
| - `mergeDeepWith` | 这类似于 `mergeDeep`，但在节点冲突时使用提供的合并函数。 |
| - `mergeIn` | 这是更新和合并的组合。它在指定的键路径执行合并。 |
| - `mergeWith` | 这类似于 merge，但在节点冲突时使用提供的合并函数。 |
| - `min` | 返回列表中的最小值。 |
| - `minBy` | 根据您提供的辅助函数确定列表中的最小值。 |
| - `of` | 创建一个包含其参数作为值的新列表。 |
| - `pop` | 返回列表中除最后一个条目之外的所有内容。请注意，这与标准的推送语义不同，但可以通过在 `push()` 之前调用 `last()` 来模拟常规的 `push()`。 |
| - `push` | 返回一个在末尾附加指定值（或值）的新列表。 |
| - `reduce` | 对每个值调用减少函数，并返回累积值。 |
| - `reduceRight` | 这类似于 reduce，但从右边开始，逐渐向左移动，与基本的 reduce 相反。 |
| - `rest` | 返回列表的尾部，即除第一个条目之外的所有条目。 |
| - `reverse` | 以相反的顺序提供列表。 |
| - `set` | 返回具有指定索引处的值的新列表。 |
| - `setIn` | 在键路径处返回一个新的列表与此值。 |
| - `setSize` | 创建一个具有您指定大小的新列表，根据需要截断或添加未定义的值。 |
| - `shift` | 创建一个减去第一个值并将所有其他值向下移动的新列表。 |
| - `skip` | 当不包括前 *n* 个条目时，返回列表中剩余的所有条目。 |
| - `skipLast` | 当不包括最后 n 个条目时，返回列表中剩余的所有条目。 |
| - `skipUntil` | 返回一个新的可迭代对象，其中包含第一个满足提供的谓词的条目之后的所有条目。 |
| - `skipWhile` | 返回一个新的可迭代对象，其中包含在提供的谓词为 false 之前的所有条目。 |
| - `slice` | 返回一个新的可迭代对象，其中包含从起始值到倒数第二个值（包括）的列表内容。 |
| - `some` | 如果谓词对列表的任何元素返回 true，则为 true。 |
| - `sort` | 返回一个按可选比较器排序的新列表。 |
| - `sortBy` | 返回一个按可选比较器值映射器排序的新列表，比较器提供了更详细的信息，因此结果更精细。 |
| - `splice` | 用第二个列表替换第一个列表的一部分，如果没有提供第二个列表，则删除它。 |
| - `take` | 创建一个包含列表中前 n 个条目的新列表。 |
| - `takeLast` | 创建一个包含列表中最后 *n* 个条目的新列表。 |
| - `takeUntil` | 返回一个新的列表，其中包含只要谓词返回 false 的所有条目；然后停止。 |
| - `takeWhile` | 只要谓词返回 true，就返回一个包含所有条目的新列表；然后停止。 |
| - `toArray` | 将此列表浅层转换为数组，丢弃键。 |
| - `toIndexedSeq` | 返回此列表的 `IndexedSeq`，丢弃键。 |
| `toJS` | 深度将此列表转换为数组。这个方法有`toJSON()`作为别名，尽管文档并没有清楚地说明`toJS()`是否返回 JavaScript 对象，而`toJSON()`返回一个 JSON 编码的字符串。 |
| `toKeyedSeq` | 从此列表返回一个`KeyedSeq`，其中索引被视为键。 |
| `toList` | 返回自身。 |
| `toMap` | 将此列表转换为 Map。 |
| `toObject` | 浅层将此列表转换为对象。 |
| `toOrderedMap` | 将此列表转换为 Map，保留迭代顺序。 |
| `toSeq` | 返回一个`IndexedSeq`。 |
| `toSet` | 将此列表转换为 Set，丢弃键。 |
| `toSetSeq` | 将此列表转换为`SetSeq`，丢弃键。 |
| `toStack` | 将此列表转换为 Stack，丢弃键。 |
| `unshift` | 将提供的值添加到列表的开头。 |
| `update` | 通过提供的更新函数更新列表中的条目。 |
| `updateIn` | 更新条目，就像`update()`一样，但在给定的键路径上。 |
| `values` | 此列表值的迭代器。 |
| `valueSeq` | 此列表值的`IndexedSeq`。 |
| `withMutations` | 这是一个优化（回想一下，“过早的优化是万恶之源”，唐纳德·克努斯说过），旨在在执行多个变异时允许更高性能的工作。当已知和持久的性能问题存在，并且其他工具明显没有解决问题时，应该使用它。 |
| `zip` | 与此列表一起返回一个被压缩的可迭代对象（即成对连接以生成 2 元组列表）。 |
| `zipWith` | 返回与自定义压缩函数一起压缩的可迭代对象。 |

API 的文档位于主页上的**Documentation**链接下，非常清晰。但是作为一个规则，Immutable.js 集合尽可能地做到了函数式程序员所期望的，实际上似乎有一个可以推测的主要设计考虑是“尽可能地做到函数式程序员所希望的”。

### 注意

可能会让函数式程序员感到不愉快的一件事是，文档没有解释如何创建无限列表。不明显如何创建列表的生成器（如果有的话），或者产生数学序列的列表，比如所有计数所有数字，正偶数，平方数，质数，斐波那契数，2 的幂，阶乘等等。这样的功能显然不受支持（在撰写本书时）。由于构造集合包括列表中的所有元素的急切包含，因此不可能使用 Immutable.js 构建无限列表。在 Immutable.js 中创建惰性序列不能构建无限列表，因为构造集合包括列表中的所有元素的急切包含，因此必须是有限的。在 Immutable.js 的风格中创建惰性和潜在无限的数据结构应该不是非常困难，这样的数据结构内部有一个记忆生成器，并允许你 XYZ.take(5)。但是 Immutable.js 似乎还没有扩展到这个领域。

# Jest - 来自 Facebook 的 BDD 单元测试

Jest 是一个旨在支持行为驱动开发的 JavaScript 单元测试框架。它是建立在 Jasmine 之上的，并且在未来可能能够与其他基础互动。它已经被使用了几年，并且在 Facebook 上被使用，尽管似乎没有明确的认可，即 ReactJS 开发最好使用 Jest。（Facebook 在内部使用 JSX 与 ReactJS，但倾向于发表一个相对不带偏见的声明，大约一半的 ReactJS 用户选择使用 JSX。它实际上被设计为完全可选的。）

### 注意

JSX——*X*大胆地表示 XML，这是在 XML 已经不受青睐的时候制作的一种良好的语法糖，它“在您的代码中放置尖括号”。这松散地意味着您可以在`.jsx`文件中将 HTML 放入 JavaScript 中，一切都可以正常工作。此外，您可以使用几乎任何可以构建在 ReactJS 组件中的页面上的东西。您可以包括一些从一开始就包含在 HTML 中的图像，也可以轻松地包括在本标题中定义的日历、线程化的网络讨论或可拖动和可缩放的分形。与子例程一样，一旦定义了组件，它就可以在 Web 应用程序的任何位置零次、一次或多次使用。JSX 语法糖允许您像旧的 HTML 标签一样轻松地包含您和其他人定义的组件。在第 8 到 11 章的项目的外壳中，JSX“非常简单”，因为它允许我们合并我们开发的其他组件：

```js
var Pragmatometer = React.createClass({
  render: function() {
    return (
      <div className="Pragmatometer">
      <Calendar />
      <Todo />
      <Scratch />
      <YouPick />
      </div>
    );
  }
});
```

Facebook 的一名员工表示，他出于“自私的原因”使 Jest 成为开源项目，即他想在自己的个人项目中使用它。这可能对为什么至少值得考虑 Jest 提供了一个很好的提示。至少有一个用户真的想要使用 Jest，以至于他愿意将专有的知识产权开源，即使没有人告诉他这样做。

可以说，在其开始阶段，单元测试已经为最容易进行单元测试的内容提供了服务，这意味着单元测试已经摆脱了集成和用户界面测试。因此，您可能会看到一篇关于单元测试的博客文章，测试并确认了将您语言的整数转换为罗马数字的函数的“红色、绿色、重构”方法，这是一个很好的例子，可以满足原始单元测试的需求。如果您想测试您的代码是否与数据库适当地交互，那就是一个稍微高一点的要求。而且，Jest 等其他框架并没有真正具有消除对好的、老式的预算可用性测试的需求的虚假倾向，就像 Jakob Nielsen 和其他人所主张的那样。在（IT 之前）业务上有一个区别，即询问“我们是否正在正确地构建产品？”和“我们是否正在构建正确的产品？”。

这两个问题都很有价值，都有其存在的理由，但是单元测试对第一个问题的帮助更大，而不是第二个问题，让一个很好地解决了第一个问题的测试套件让您对解决第二个问题的测试套件产生危险。尽管如此，Jest 提供的东西比仅仅测试代码单元是否能成功接受原始数据类型（例如整数、浮点数或字符串）的输入，并返回原始数据类型的正确和预期输出（例如输入整数的正确罗马数字）更有用。尽管这不仅适用于 Jest，但 Jest 模拟用户界面以支持（例如）用户界面事件，例如单击元素，并支持测试用户界面更改，例如标签上的文本（比较 Jasmine 主页，那里的前几个示例只涉及使用原始数据类型的断言）。

Jest 旨在在 Jasmine（以及将来可能的其他后端）之上提供层，但具有显著的附加值。除了某些功能，例如并行运行测试，使测试变得更加响应，Jest 是一种解决方案，旨在需要最少的时间和麻烦来获得良好的测试覆盖率，基于这样的想法，即开发人员应该花费大部分时间在主要开发上，而不是编写单元测试。

Jest 旨在模拟使用`require()`导入的所有内容，或几乎所有内容。您可以通过调用`jest.dontMock()`来选择不模拟单个元素，测试通常会调用`jest.dontMock()`来取消模拟它们正在测试的组件。它会自动查找并运行`__tests__`目录中的测试。如果在例如`preprocessor.js`中包含了 ReactJS 的预处理器，Jest 可以处理 JSX：

```js
var ReactTools = require('react-tools');
module.exports = {
  process: function(source) {
    return ReactTools.transform(source);
  }
};
```

`package.json`文件需要告诉它要加载什么：

```js
'dependencies': {
  'react': '*',
  'react-tools': '*'
},
}, 'jest': {
  'scriptPreprocessor': '<root directory>/preprocessor.js',
  'unmockedModulePathPatterns':['<root directory>/node_modules/react']
},
```

现在我们将轻微地改编 Facebook 的示例。Facebook 提供了一个`CheckboxWithLabel`类的示例。这个类在复选框未选中时显示一个标签，在选中时显示另一个标签。这里的 Jest 单元测试模拟了一次点击，并确认标签是否适当地更改。

`CheckboxWithLabel.js`文件的内容如下：

```js
/** @jsx React.DOM */

var React = require('react/addons');
var CheckboxWithLabel = React.createClass({
  getInitialState: function() {
    return {
      isChecked: false
    };
  },
  onChange: function() {
    this.setState({isChecked: !this.state.isChecked});
  },
  render: function() {
    return (
      <label>
        <input
          type="checkbox"
          checked={this.state.isChecked}
          onChange={this.onChange}
        />
        {(this.state.isChecked ?
        this.props.labelOn :
        this.props.labelOff)}
      </label>
    );
  }
});

module.exports = CheckboxWithLabel;
```

`__tests__/CheckboxWithLabel-test.js`测试文件中写道：

```js
/** @jsx React.DOM */

jest.dontMock('../CheckboxWithLabel.js');

describe('CheckboxWithLabel', function() {
  it('changes the text after click', function() {
    var React = require('react/addons');
    var CheckboxWithLabel = require('../CheckboxWithLabel.js');
    var TestUtils = React.addons.TestUtils;

    // Verify that it's Off by default.
    var label = TestUtils.findRenderedDOMComponentWithTag(
      checkbox, 'label');
    expect(label.getDOMNode().textContent).toEqual('Off');

    // Simulate a click and verify that it is now On.
    var input = TestUtils.findRenderedDOMComponentWithTag(
      checkbox, 'input');
    TestUtils.Simulate.change(input);
    expect(label.getDOMNode().textContent).toEqual('On');
  });
});
```

# 使用 Fluxxor 实现 Flux 架构

如前几章所述，Flux 是 Facebook 开发并被他们用作 ReactJS 的一个大部分补充的架构。它帮助解开了一个真正的交叉线的乱麻，并让 Facebook 彻底消除了一个反复出现的消息计数错误——Flux 架构永久地杀死了它。**Fluxxor**，由 Brandon Tilley ([`fluxxor.com`](http://fluxxor.com))，是一个旨在帮助人们在他们的应用程序中实现 Flux 架构的工具。没有必要使用 Flux 架构来使用 ReactJS，或者使用 Fluxxor 工具来实现 Flux 架构。但是 Flux，也许是 Fluxxor，至少值得考虑，以使事情变得更容易。

Fluxxor 具有用于整体 Flux 架构的类，包括`Fluxxor.Flux`容器（其中包括一个分发器）和`Action`和`Store`类。示例代码简洁易读，看起来几乎没有样板。还提供了两个适用于 ReactJS 的 mixin 类以方便使用。示例代码使用 JSX 编写。

我还可能评论 Fluxxor 的作者，[`fluxxor.com`](http://fluxxor.com)在页面底部有一个链接，要求人们在 GitHub 上报告问题，如果有什么不清楚或有问题。我注意到一个常见的可用性缺陷——访问和未访问的链接颜色相同——并在 GitHub 上报告了这个问题。作者立即道歉，我提出的问题在不到 15 分钟内就被*关闭并修复*了。我认为他是那种人们愿意一起工作的人。

# 总结

现在让我们看看本章涵盖了什么。我们解释了 Om 和 ClojureScript，它们允许利用 ReactJS 的能力进行基于 Lisp 的开发。据说 ClojureScript 可能是允许美丽的不同语言拼接的解决方案的领头羊，这些语言可用于前端开发、编译或解释 JavaScript 作为新的“裸金属”。

Bacon.js 是一种非常受尊敬的技术，与 ReactJS 竞争，允许在浏览器中进行良好的函数式响应式编程。这不是作为“唯一”良好示例，而是作为超出本书范围的好东西的一个例子。

我们还介绍了 Brython，一个基于浏览器的 Python 环境。它并不完美，但很有趣。它被作为一个可以在 Lisp 之外的语言中用作网页开发的例子。提醒一下，[`tinyurl.com/reactjs-compiled-javascript`](http://tinyurl.com/reactjs-compiled-javascript)提供了一个目录，其中包括了编译为 JavaScript 或可以在 Web 浏览器中解释的其他语言，从语法糖如 CoffeeScript 到 JavaScript 扩展到独立语言如 Ruby、Python（包括 Brython）、Erlang、Perl 等等。

Immutable.js 通过提供主要是允许在不破坏不可变数据的功能优势的情况下进行复制的集合，填补了函数式 JavaScript 中的漏洞。

Jest 是一个由 Facebook 用于 ReactJS 的行为驱动开发 JavaScript 单元测试框架。Fluxxor 是一个控制器、动作和存储的实现，旨在使将 Flux 架构应用到 JavaScript 开发中更容易，包括 ReactJS。

在下一章中，让我们一起探索使用 ReactJS 的更深入的示例。
