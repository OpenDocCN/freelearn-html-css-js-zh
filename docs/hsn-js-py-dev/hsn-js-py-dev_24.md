# 第二十章：评估

# 第一章

1.  哪个国际组织维护 JavaScript 的官方规范？

1.  W3C

1.  **Ecma International**

1.  网景

1.  太阳

1.  哪些后端可以与 JavaScript 通信？

1.  PHP

1.  Python

1.  Java

1.  **以上所有**

1.  谁是 JavaScript 的原始作者？

1.  蒂姆·伯纳斯-李

1.  **布兰登·艾奇**

1.  Linus Torvalds

1.  比尔·盖茨

1.  DOM 是什么？

1.  JavaScript 在内存中对 HTML 的表示

1.  允许 JavaScript 修改页面的 API

1.  **以上两者**

1.  以上都不是

1.  Ajax 的主要用途是什么？

1.  与 DOM 通信

1.  DOM 的操作

1.  监听用户输入

1.  **与后端通信**

# 第二章

1.  **真**或假：Node.js 是单线程的。

1.  真或**假**：Node.js 的架构使其不受**分布式拒绝服务**（**DDoS**）攻击的影响。

1.  谁最初创建了 Node.js？

1.  布兰登·艾奇

1.  Linux Torvalds

1.  阿达·洛夫莱斯

1.  **Ryan Dahl**

1.  真或**假**：服务器端的 JavaScript 本质上是不安全的，因为代码在前端暴露。

1.  真或**假**：Node.js 本质上优于 Python。

# 第三章

1.  以下哪个不是有效的 JavaScript 变量声明？

1.  var myVar = 'hello';

1.  const myVar = "hello"

1.  **String myVar = "hello";**

1.  let myVar = "hello"

1.  以下哪个开始了函数声明？

1.  **功能**

1.  const

1.  功能

1.  def

1.  以下哪个不是基本循环类型？

1.  for..in

1.  为

1.  当

1.  **映射**

1.  JavaScript *需要*使用分号进行行分隔：

1.  真

1.  **假**

1.  在 JavaScript 中，空格*永远*不计算：

1.  真

1.  **假**

# 第四章

1.  JavaScript 本质上是：

1.  同步

1.  异步

1.  **两者**

1.  一个`fetch()`调用返回：

1.  `then`

1.  `next`

1.  `最后`

1.  **Promise**

1.  使用原型继承，我们可以（选择所有适用的选项）：

1.  **在基本数据类型中添加方法。**

1.  **从基本数据类型中减去方法。**

1.  重命名我们的数据类型。

1.  将我们的数据转换为另一种格式。

```js
let x = !!1
console.log(x)
```

1.  在给定的代码中，预期的输出是什么？

1.  1

1.  假

1.  0

1.  **真**

```js
const Officer = function(name, rank, posting) {
 this.name = name
 this.rank = rank
 this.posting = posting
 this.sayHello = () => {
 console.log(this.name)
 }
}

const Riker = new Officer("Will Riker", "Commander", "U.S.S. Enterprise")
```

1.  在这段代码中，输出“威尔·莱克”最好的方法是什么？

1.  **`Riker.sayHello() `***

1.  `console.log(Riker.name)`

1.  `console.log(Riker.this.name)`

1.  `Officer.Riker.name()`

# 第五章

考虑以下代码：

```js
function someFunc() {
  let bar = 1;

  function zip() {
    alert(bar); // 1
    let beep = 2;

    function foo() {
      alert(bar); // 1
      alert(beep); // 2
    }
  }

  return zip
}

function sayHello(name) {
  const sayAlert = function() {
    alert(greeting)
  }

  const sayZip = function() {
    someFunc.zip()
  }

  let greeting = `Hello ${name}`
  return sayAlert
}
```

1.  如何获得警报`'你好，鲍勃'`？

1.  `sayHello()('Bob')`

1.  `sayHello('Bob')()`*****

1.  `sayHello('Bob')`

1.  `someFunc()(sayHello('Bob'))`

1.  在上述代码中，`alert(greeting)`会做什么？

1.  警报`'问候'`

1.  警报`'你好，爱丽丝'`

1.  **抛出错误**

1.  以上都不是

1.  我们如何获得警报消息`1`？

1.  `someFunc()()`*****

1.  `sayHello().sayZip()`

1.  `alert(someFunc.bar)`

1.  `sayZip()`

1.  我们如何获得警报消息`2`？

1.  `someFunc().foo()`

1.  `someFunc()().beep`

1.  **我们不能，因为它不在范围内**

1.  我们不能，因为它没有定义

1.  我们如何将`someFunc`更改为警报 1 1 2？

1.  我们不能。

1.  在`return zip`之后添加`return foo`。

1.  将`return zip`更改为`return foo`。

1.  **在`foo`声明之后添加`return foo`**。

1.  在给定上一个问题的正确解决方案的情况下，我们如何实际获得三个警报，即 1、1、2？

1.  `someFunc()()()`*****

1.  `someFunc()().foo()`

1.  `someFunc.foo()`

1.  `alert(someFunc)`

# 第六章

考虑以下代码：

```js
  <button>Click me!</button>
```

1.  选择按钮的正确语法是什么？

1.  document.querySelector('点击我！')

1.  document.querySelector('.button')

1.  document.querySelector('#button')

1.  **document.querySelector('button')**

看看这段代码：

```js
<button>Click me!</button>
<button>Click me two!</button>
<button>Click me three!</button>
<button>Click me four!</button>
```

1.  真或**假**：document.querySelector('button')将满足我们对每个按钮放置点击处理程序的需求。

1.  要将按钮的文本从“点击我！”更改为“先点我！”，我们应该使用什么？

1.  **document.querySelectorAll('button')[0].innerHTML = "先点我！"**

1.  document.querySelector('button')[0].innerHTML = "先点我！"

1.  document.querySelector('button').innerHTML = "先点我！"

1.  document.querySelectorAll('#button')[0].innerHTML = "先点我！"

1.  我们可以使用哪种方法添加另一个按钮？

1.  document.appendChild('button')

1.  document.appendChild('<button>')

1.  **document.appendChild(document.createElement('button'))**

1.  document.appendChild(document.querySelector('button'))

1.  如何将第三个按钮的类更改为“third”？

1.  document.querySelector('button')[3].className = 'third'

1.  **document.querySelectorAll('button')[2].className = 'third'**

1.  document.querySelector('button[2]').className = 'third'

1.  document.querySelectorAll('button')[3].className = 'third'

# 第七章

回答以下问题以衡量您对事件的理解：

1.  以下哪个是事件生命周期的第二阶段？

1.  捕获

1.  **定位**

1.  冒泡

1.  （选择所有正确答案）事件对象为我们提供了什么？

1.  **触发的事件类型**

1.  **目标 DOM 节点（如果适用）**

1.  **鼠标坐标（如果适用）**

1.  父 DOM 节点（如果适用）

看看这段代码：

```js
container.addEventListener('click', (e) => {
  if (e.target.className === 'box') {
    document.querySelector('#color').innerHTML = e.target.style.backgroundColor
    document.querySelector('#message').innerHTML = e.target.innerHTML
    messageBox.style.visibility = 'visible'
    document.querySelector('#delete').addEventListener('click', (event) => {
      messageBox.style.visibility = 'hidden'
      e.target.remove()
    })
  }
})
```

1.  它使用了哪些 JavaScript 特性？选择所有适用的答案：

1.  **DOM 操作**

1.  **事件委托**

1.  **事件注册**

1.  **样式更改**

1.  当容器被点击时会发生什么？

1.  `box` 将可见。

1.  `#color` 将是红色的。

1.  选项 1 和 2 都是。

1.  **没有足够的上下文来说。**

1.  在事件生命周期的哪个阶段我们通常采取行动？

1.  **定位**

1.  捕获

1.  冒泡

# 第九章

1.  内存问题的根本原因是什么？

1.  程序中的变量是全局的。

1.  **低效的代码。**

1.  JavaScript 的性能限制。

1.  硬件不足。

1.  在使用 DOM 元素时，应该将对它们的引用存储在本地，而不是总是访问 DOM。

1.  真

1.  假

1.  **当多次使用它们时为真**

1.  JavaScript 在服务器端进行预处理，因此比 Python 更有效。

1.  真

1.  **假**

1.  设置断点无法找到内存泄漏。

1.  真

1.  **假**

1.  将所有变量存储在全局命名空间中是一个好主意，因为它们更有效地引用。

1.  真

1.  **假**
