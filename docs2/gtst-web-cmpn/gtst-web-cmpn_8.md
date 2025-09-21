# 将 Web 组件与 Web 框架集成

在前面的章节中，我们要么从头开始创建 Web 组件，要么使用库来创建 Web 组件。我们甚至只用 Web 组件创建了一个单页 Web 应用程序。但如果我们有一个现有的项目怎么办？如果这是一个单体前端 Web 应用程序，我们需要在这个项目中使用 Web 组件的方法怎么办？如果我们想用一个 Web 组件实现一个快速原型化的功能而不需要太多开销怎么办？这可以在时间和金钱上节省大量的努力。

这一章仅针对此类场景；在这里，我们将探讨我们可以如何使用 Web 组件在现有项目中。

顺便说一句，这一章是为高级用户准备的。

我假设你已经使用过 React、Angular 或 Vue，并且你正在寻找将 Web 组件包含到已经使用这些技术之一或多个的 Web 应用程序中的方法。我也假设你知道如何运行这些 Web 应用程序。然而，为了简化，我们将查看使用这些技术中最简单的组件，以及如何将两个 Vanilla Web 组件包含在这些技术中的任何一种中。

在这一章中，我们将看到以下主题：

+   与现有项目的集成

+   在 React 中集成 Web 组件

+   在 Angular 中集成 Web 组件

+   在 Vue 中集成 Web 组件

# `<header-image>` Web 组件

假设我们有一个名为 `<header-image>` 的 Web 组件，其目的是显示一个图像，并且在鼠标悬停时，它应该能够显示一个文本，显示图像的简要描述。这个 Web 组件的定义可能看起来像这样：

```js
export default class HeaderImage extends HTMLElement {
  constructor() {

    // We are not even going to touch this.
    super();

    // lets create our shadow root
    this.shadowObj = this.attachShadow({mode: 'open'});

    // Then lets render the template
    this.render();
  }

  static get observedAttributes() {
    return ['src', 'alttext'];
  }

  attributeChangedCallback(name, oldValue, newValue) {
    if (name == 'src') {
      this.src = newValue;
      this.render();
    }
    if (name == 'alttext') {
      this.alt = newValue;
      this.render();
    }
  }

  ...
  ...

}
```

如你所见，我们在构造函数中简单地调用了 `super()` 方法。然后我们为组件创建了一个 shadow root，并调用了 `render()` 方法。我们还确保通过属性传入的任何更改都会重新渲染 Web 组件，以反映与这些属性相关的更新。

关于 `render()` 方法，它看起来可能如下所示：

```js
render() {
  this.shadowObj.innerHTML = this.getTemplate();
}

getTemplate() {
  return `
    <img src="img/${this.getAttribute('src')}"
      alt="${this.getAttribute('alt')}"/>
    ${this.handleErrors()}
    <style>
      img {
        width: 400px;;
      }
    </style>
  `;
}
```

在这里，我们在 shadow root 的 HTML 中添加了一个图像。此外，我们还通过 `handleErrors()` 方法启用了错误处理：

```js
handleErrors() {
  if(!this.getAttribute('alt')) {
    return `
      <div class="error">Missing Alt Text</div>
      <style>
        .error {
          color: red;
        }
      </style>
    `;
  }

  return ``;
}
```

这个 `handleErrors()` 方法寻找缺失的属性 `alt`，并输出一个错误消息，要求用户输入 `alttext`。

我们可以使用以下 HTML 使用这个 Web 组件：

```js
<header-image alttext="Blue Clouds"
      src="img/clouds-sky-header-2069-1024x300.jpg"></header-image>
```

现在我们已经知道了我们的 Web 组件的样子，让我们尝试在现有项目中使用它。我们将从一个使用 React 的现有项目开始。

# 在 React 中集成 Web 组件

假设我们有一个 React 应用程序。我将使用 React 提供的启动应用程序。你可以在你的更复杂的应用程序中自由尝试这个功能。进行此操作的步骤将完全相同。

# 设置 React 项目

如果你有自己的应用程序，你不需要通过这一部分。

您可以使用以下链接来设置项目：[`facebook.github.io/create-react-app/`](https://facebook.github.io/create-react-app/)。

设置完成后，您将得到一个可以使用以下命令运行的项目：

```js
npm start
```

# 添加 React 组件

为了简化，我添加了一个 React 组件。这个 React 组件将模拟一个负责包含`<header-image>`网络组件的真实生活场景。让我们把这个 React 组件称为`MainBody`；其定义看起来可能如下：

```js
import React, { Component } from 'react';

export default class MainBody extends Component {
  render() {
    return (
      <div>
        <p>This is the main body</p>
      </div>
    );
  }
}
```

如您所见，它只显示一行文本，没有其他内容。如果您有一个更复杂的组件，步骤将是相同的。至于入门级应用，我们首先在`App`组件中包含这个`MainBody`组件，如下所示：

```js
import React from 'react';
import logo from './logo.svg';
import './App.css';
import MainBody from './main-body/main-body.js';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
        <MainBody />
      </header>
    </div>
  );
}

export default App;
```

在这里，我们只是导入`MainBody`组件，并在`App`组件中直接使用它。

# 在 React 组件中集成 Vanilla Web Component

为了在`MainBody` React 组件中使用`<header-image>`组件，我们将在`MainBody`组件中添加一些内容：

```js
import React, { Component } from 'react';
import HeaderImage from '../web-components/header-image/header-image.js';

export default class MainBody extends Component {

  constructor() {
    super();
    this.state = {
      src: 'https://www.freewebheaders.com/wordpress/wp-content/gallery/clouds-sky/clouds-sky-header-2069-1024x300.jpg',
      altText: 'Blue Clouds'
    }
  }

  componentDidMount() {
    customElements.define('header-image', HeaderImage);
  }

  render() {
    return (
      <div>
        <p>This is the main body</p>
        <header-image alttext={this.state.altText}
          src={this.state.src}>
        </header-image>
      </div>
    );
  }
}
```

在这里，我们从其相应位置导入`<header-image>`网络组件，然后在生命周期回调方法`componentDidMount()`中注册自定义元素。然后，我们尝试通过状态变量将`alt`和`src`发送到`<header-image>`组件。

对于所有试图使用任何 Vanilla Web Component 的 React 组件，步骤都是相同的。现在我们已经了解了如何在 React 项目中使用 Web Component，让我们看看它在一个 Angular 应用程序中的样子。

# 在 Angular 中集成 Web 组件

假设我们有一个已经存在的 Angular 应用程序。这可能是一个完整的项目或入门级应用，我们想在 Angular 组件中使用`<header-image>`网络组件。我们将从设置开始。

# 设置 Angular 项目

假设我们想要从一个入门级应用开始。我们可以按照以下网址提供的步骤来安装和运行入门级应用：[`angular.io/guide/quickstart`](https://angular.io/guide/quickstart)。

Angular 默认不支持 Vanilla Web Components，因此在我们开始使用 Web Components 之前，我们需要告诉 Angular 我们想要使用一个网络组件。我们可以在`app.module.ts`文件中添加以下代码来实现：

```js
...
import { NgModule, CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';
...
...

@NgModule({
  ...
  ...
  schemas: [
   CUSTOM_ELEMENTS_SCHEMA
  ]
})
```

这是告诉 Angular 预期一个不是使用 Angular 构建的自定义元素。

# 与 Angular 集成

现在，假设我们有一个名为`app-main-body`的组件，它是使用 Angular 构建的（文件：`main-body.component.ts`），其外观大致如下：

```js
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-main-body',
  templateUrl: './main-body.component.html',
  styleUrls: ['./main-body.component.css']
})
export class MainBodyComponent implements OnInit {
  src: string ;
  altText: string;

  constructor() {
    this.src = 'https://www.freewebheaders.com/wordpress/wp-content/gallery/clouds-sky/clouds-sky-header-2069-1024x300.jpg';
    this.altText = 'Blue Clouds';
  }

  ngOnInit() {

  }
}
```

如果我们想在这里包含`<header-image>`网络组件，我们可以简单地添加以下代码：

```js
...
import HeaderImage from '../web-components/header-image/header-image.js';

...
export class MainBodyComponent implements OnInit {
  ...
  ...

  ngOnInit() {
    customElements.define('header-image', HeaderImage);
  }

}
```

在这里，我们只是导入组件定义，然后在`ngOnInit()`回调方法内部注册自定义元素。如果我们查看模板文件`main-body.component.html`，Web 组件可以按照以下代码所示包含在内：

```js
<p>
  main-body works!
</p>
<header-image attr.src={{src}} attr.alttext="{{alttext}}"></header-image>
```

在这里，我们将`src`和`altText`作为属性值传递给`<header-image>`组件。这样，我们就可以在 Angular 项目中使用在 Angular 之外构建的 Web Components。

既然我们已经知道了如何在 Angular 项目中使用 Vanilla Web 组件，那么让我们看看如何在 Vue 组件中使用 Web Components。

# 在 Vue 中集成 Web Components

Vue 是那些增长速度极快的库之一，因此我认为如果我们看到如何在 Vue 组件中包含 Web 组件，那将是一件好事。

假设我们有一个看起来像这样的`<main-body>` Vue 组件：

```js
Vue.component('main-body', {
  props: ['src', 'alt'],
  template: `
    <p>This is the main body</p>
    `,
})
```

如你所见，它所做的只是显示文本，就像 Angular 和 React 中的主体组件一样。假设我们想要将`<header-image>` Web 组件包含到这个`<main-body>`组件中。这将使`<main-body>`组件看起来像这样：

```js
import HeaderImage from '../web-components/header-image/header-image.js';

Vue.component('main-body', {
  props: ['src', 'alt'],
  template: `
    <p>This is the main body</p>
    <header-image src="img/{{src}}" alttext="{{alt}}"></header-image>
    `,
  created: function() {
    customElements.define('header-image', HeaderImage);
  }
})
```

在这里，我们只是导入`HeaderImage`组件，并在`created()`回调方法内部注册这个 Web 组件。正如你所见，在 Vue 组件中使用 Web 组件非常简单，可以通过插值将属性值传递给 Web 组件，就像之前代码中展示的那样。

使用本节中提到的过程，我们可以将现有的 Web 组件添加到任何 Vue 项目中。

# 摘要

在本章中，我们探讨了如何将 Web Components 集成到已经使用前端世界中最著名的库/框架的一些现有项目中。我们学习了如何将使用 Vanilla JavaScript 构建的现有 Web 组件添加到 React、Angular 或 Vue 项目中。本章中学到的技术可以用于任何框架或库，以及任何类型的现有项目。在现有项目中包含 Web Components 也是快速原型化功能的好用例，组件一旦完成工作就可以立即移除。

我希望这一章能帮助你创建更好的 Web 应用程序，无论它们是否使用 Web Components。
