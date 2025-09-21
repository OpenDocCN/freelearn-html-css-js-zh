# Web 组件生命周期回调方法

在上一章中，我们讨论了如何使用纯 JavaScript 和 HTML5 创建 Web 组件。我们讨论了创建 Web 组件概念所需的规范。在本章中，我们将讨论生命周期事件及其相关的回调方法。生命周期事件是在 Web 组件的生命周期中发生的事件。本章将处理这些事件以及如何通过回调方法访问它们。

在本章中，我们将涵盖以下主题：

+   生命周期回调方法的概述

+   Web 组件中当前可用的生命周期回调方法

# 生命周期回调方法的概述

生命周期事件是在 Web 组件达到执行某个阶段时在组件内部触发的事件。这些阶段反映了创建 Web 组件的整体过程，并且可以通过生命周期回调方法来控制。生命周期回调方法是当 Web 组件经过这些生命周期事件时被调用的钩子或接口。

让我用一个例子来解释这一点。假设你有一双你想穿的鞋。这双鞋的生命周期可能有一些相关的事件。比如说，你想穿上它。你把脚伸进去，系上鞋带。这会触发一个名为 `lacesTied()` 的事件。现在，作为穿着这双鞋的用户，你可以选择对其做出反应。你可以编写一个条件块来检查 `lacesTiedStrength > 100` 或 `shoeSize < requiredShoeSize`。这取决于你想做什么。同样，与 Web 组件相关联的生命周期回调方法可以帮助用户捕捉到某些执行状态，并有效地编写代码。

# 生命周期回调方法的类型

目前 Web 组件有四个生命周期回调可用。具体如下：

+   `connectedCallback()`

+   `disconnectedCallback()`

+   `adoptedCallback()`

+   `attributeChangedCallback()`

让我们详细讨论它们。

# `connectedCallback()`

这个接口/回调函数会在每次将一个 Web 组件的副本添加到 DOM 中时被调用。当需要在组件内部初始化与 DOM 相关的事件或状态管理（见第五章，*管理状态和属性*），或需要进行任何类型的初始化或预检查时，这非常有用。

让我们来看一个例子。在第一章，*Web 组件基础和规范*中，我们讨论了一个 `<student-attendance-table>` 组件，其中 Web 组件会向 `student.json` 文件发起 fetch 调用，以检索出勤数据，然后以表格的形式显示这些数据。

正确编写该 Web 组件的方法是在 `StudentAttendenceTable` 类的定义中添加一个 `connectedCallback()` 方法，然后在回调函数内部进行获取调用。

这就是我们的代码会看起来像这样：

```js
// StudentAttendanceTable.js

export default class StudentAttendanceTable extends HTMLElement {
  constructor() {
    super();

    this.innerText = this.getLoadingText();
  }

  connectedCallback() {
    // let's start our fetch call
    this.getStudentList();
  }

  getStudentList() {
    // lets use fetch api
    // https://developer.mozilla.org/en-US/docs/Web
    // /API/Fetch_API/Using_Fetch
    fetch('./student.json')
    .then(response => {

      // converts response to json
      return response.json();

    })
    .then(jsonData => {
      this.generateTable(jsonData);
    })
    .catch(e => {

      // lets set the error message for
      // the user
      this.innerText = this.getErrorText();

      // lets print out the error
     // message for the devs
      console.log(e);
    });

  }

  generateTable(names) {
    // lets loop through names
    // with the help of map
    let rows = names.map((data, index) => {
      return this.getTableRow(index, data.name);
    });

    // creating the table
    let table = document.createElement('table');
    table.innerHTML = rows.join('');

    // setting the table as html for this component
    this.appendHTMLToShadowDOM(table);
  }

  getTableRow(index, name) {
    let tableRow = `<tr>
        <td>${index + 1}</td>
        <td>${name}</td>
        <td>
          <input type="checkbox" name="${index}-attendance"/>
        </td>
      </tr>`;

    return tableRow;
  }

  appendHTMLToShadowDOM(html) {
    // clearing out old html
    this.innerHTML = '';

    let shadowRoot = this.attachShadow({mode: 'open'});

    // add a text node
    shadowRoot.append(html);
  }

  getLoadingText() {
    return `loading..`;
  }

  getErrorText() {
    return `unable to retrieve student list.`;
  }
}
```

如您在代码中所见，我们现在在 `connectedCallback()` 方法中调用获取学生列表。这确保了代码在 Web 组件附加到网页后执行。

使用 `connectedCallback` 有帮助的另一个例子是事件处理。假设我们有一个显示自定义按钮的 Web 组件。这个按钮的目的是在它旁边显示一些文本，说明按钮被点击的次数。如果我们尝试在不使用 `connectedCallback` 的情况下使用它，它看起来可能就像这样：

```js
// CustomButton.js

export default class CustomButton extends HTMLElement {
  constructor() {
    super();

    // Initializing an initial state
    this.timesClicked = 0;

    let template = `
      <button>Click Me</button>
      <span>${this.getTimesClicked()}</span>
    `;

    this.innerHTML = template;
  }

  connectedCallback() {

    // adding event handler to the button
    this.querySelector('button')
      .addEventListener('click', (e) => {
        this.handleClick(e);
      });
  }

  handleClick() {
    // updating the state
    this.timesClicked++;

    this.querySelector('span')
      .innerText = this.getTimesClicked();
  }

  getTimesClicked() {
    return `Times Clicked: ${this.timesClicked}`;
  }
}
```

相关的 HTML 会看起来像这样：

```js
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <title>Connected Callback Example</title>

    <!--
      Notice how we use type="module"
    -->
    <script type="module">

      /// importing the first custom element
      import CustomButton from './CustomButton.js';

      customElements.define('custom-button', CustomButton);
    </script>

  </head>
  <body>

    <custom-button></custom-button>

  </body>
</html>
```

注意到在 `connectedCallback()` 方法中绑定了一个事件监听器。我们将在第五章 [0330b4ac-5fed-441c-8747-ef9f14f91418.xhtml]，*管理状态和属性* 中详细讨论事件监听器，但现在；我们可以用代码作为例子。前面的代码确保只有在 DOM 可用后，才将点击事件绑定到按钮上。这可以防止我们创建与事件相关的错误，我相信这肯定发生在我们每个人身上。

# disconnectedCallback()

就像在将 Web 组件添加到 DOM 时需要执行某些操作一样，在组件从 DOM 中移除后也需要执行某些操作。这种情况最常见的例子又是事件处理器。事件处理器消耗内存，当与它们关联的 DOM 被移除时，事件处理器仍然在页面上监听事件，仍然消耗内存。回调函数 `disconnectedCallback()` 为 Web 组件提供了一种编写代码来处理这些情况的方法。

让我们来看看 `<custom-button>` 元素，以及我们如何使用 `disconnectedCallback()` 来移除附加的事件：

```js
// CustomButton.js

export default class CustomButton extends HTMLElement {
  constructor() {
    super();

    // Initializing an initial state
    this.timesClicked = 0;

    let template = `
      <button>Click Me</button>
      <span>${this.getTimesClicked()}</span>
    `;

    this.innerHTML = template;
  }

  connectedCallback() {

    // adding event handler to the button
    this.querySelector('button')
      .addEventListener('click', this.handleClick.bind(this));
  }

  disconnectedCallback() {
    console.log('We are inside disconnectedCallback');

    // adding event handler to the button
    this.querySelector('button')
      .removeEventListener('click', this.handleClick);
  }

  handleClick() {
    // updating the state
    this.timesClicked++;

    this.querySelector('span')
      .innerText = this.getTimesClicked();
  }

  getTimesClicked() {
    return `Times Clicked: ${this.timesClicked}`;
  }
}
```

如果你查看 `disconnectedCallback()` 方法，我们会看到一个 `console.log` 语句和移除事件的代码。当你在一个页面上运行这个 Web 组件时，你可以手动移除组件，并看到 `disconnectedCallback()` 会自动被调用。我更喜欢去开发者控制台，输入以下代码来观察这个过程：

```js
document.querySelector('custom-button').remove();
```

这将从页面上移除第一个 `<custom-button>` 实例，从而触发 `disconnectedCallback()` 方法。

移除事件处理器只是其中一种用途。在从 DOM 中移除 Web 组件之前，可能需要执行任何数量的用例。

# adoptedCallback()

当 Web 组件从一个父元素移动到另一个父元素时，这个回调会被触发。

就像我们有 `connectedCallback` 和 `disconnectedCallback` 一样，我们可以这样编写 `adoptedCallback`：

```js
adoptedCallback() {
  console.log('I am adopted');
}
```

# attributeChangedCallback()

由于所有自定义元素都像任何其他 HTML 元素一样行动和表现，它们也有能力在其内部拥有属性。我们将在下一章讨论属性，但现在，让我们假设我们有一个名为 `<my-name>` 的自定义元素，其目的是显示文本“Hello, my name is John Doe”。

因此，这个 Web 组件的定义可能如下所示：

```js
// MyName.js

export default class MyName extends HTMLElement {
  constructor() {
    super();

    this.innerText = 'Hello, my name is John Doe';
  }
}
```

现在，假设我们想要一个不同的名称。对于每个不同的名称，我们都需要为自定义元素定义一个不同的定义，从而创建一个完全不同的 Web 组件。为了解决这个问题，我们可以使用属性。我们可以在该元素的 HTML 标签内的属性中传递名称，使其看起来像这样：

```js
<my-name fullname="John Doe"></my-name>
```

但是，为了使 Web 组件使用提供的属性，我们首先要求它观察某些属性，我们可以像这样提供一个数组：

```js
static get observedAttributes() {
  return ['fullname'];
}
```

在这里，我们只是将要观察 `fullname`。你可以根据需求添加更多。我们将在接下来的章节中深入探讨属性。

一旦我们开始观察这些属性，我们就可以使用 `attributeChangedCallback()` 方法根据需求对自定义元素进行必要的更改。我只是在下面的回调中更新了名称：

```js
attributeChangedCallback(name, oldValue, newValue) {
  if (name == 'fullname') {
    this.innerText = 'Hello, my name is ' + newValue;
  }
}
```

如你所见，`attributeChangedCallback()` 接收三个参数：`name`，即更改的属性的名称，以及 `oldValue` 和 `newValue`，分别是更改前后的值。

在前面的代码中，我们只是检查属性的名称是否为 `fullname`，并将文本更新为更新的名称。

完整的组件代码如下所示：

```js
// MyName.js

export default class MyName extends HTMLElement {
  constructor() {
    super();

   this.innerText = 'Hello, my name is NO NAME';
  }

  static get observedAttributes() {
    return ['fullname'];
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (name == 'fullname') {
      this.innerText = 'Hello, my name is ' + newValue;
    }
  }
}
```

与其相关的 `index.html` 文件如下所示：

```js
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <title>Attribute Changed Callback Example</title>

    <!--
      Notice how we use type="module"
    -->
    <script type="module">

      /// importing the first custom element
      import MyName from './MyName.js';

      customElements.define('my-name', MyName);
    </script>

  </head>
  <body>

    <my-name fullname="John Doe"></my-name>

  </body>
</html>
```

如你所见，我们并没有在前面章节中做任何不同的事情。

# 摘要

在本章中，我们讨论了生命周期回调方法是什么以及它们的作用。我们讨论了 `connectedCallback()`、`disconnectedCallback()`、`adoptedCallback()` 和 `attributeChangedCallback()`。我们探讨了如何使用这些回调及其实际用途的多种示例。

在下一章中，我们将探讨如何使用 CSS 来美化我们的 Web 组件，然后我们将讨论黄金标准清单及其目的。
