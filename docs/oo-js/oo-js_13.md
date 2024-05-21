# 第十三章。响应式编程和 React

随着 ES6，一些新的想法正在涌现。这些是强大的想法，可以帮助你用更简洁的代码和设计构建强大的系统。在本章中，我们将向你介绍两种这样的想法-响应式编程和 react。尽管它们听起来相似，但它们是非常不同的。本章不会详细讨论这些想法的实际细节，但会给你必要的信息，让你了解这些想法的潜力。有了这些信息，你可以开始将这些想法和框架融入到你的项目中。我们将讨论响应式编程的基本思想，并更详细地看一下 react。

# 响应式编程

响应式编程最近受到了很多关注。这个想法相对较新，像许多新想法一样，有很多令人困惑的，有时是矛盾的信息在流传。我们在本书的前面讨论了异步编程。JavaScript 通过提供支持异步编程的一流语言构造，将异步编程推向了新的高度。

响应式编程本质上是使用异步事件流进行编程。事件流是随时间发生的事件序列。考虑以下图表：

![Reactive programming](img/image_13_001.jpg)

在上图中，时间从左到右流逝，不同的事件随时间发生。随着事件随时间发生，我们可以向整个序列添加事件监听器。每当事件发生时，我们都可以通过做一些事情来对其做出反应。

JavaScript 中的另一种序列是数组。例如，考虑以下代码行：

```js
    var arr = [1,1,13,'Rx',0,0]; 
    console.log(arr); 
    >>> [1, 1, 13, "Rx", 0, 0] 

```

在这种情况下，整个序列同时存在于内存中。然而，在事件流的情况下，事件随时间发生，此时没有状态。考虑以下代码行：

```js
    var arr = Rx.Observable.interval(500).take(9).map(
      a=>[1,1,13,'Rx',0,0][a]); 
    var result = arr; 
    result.subscribe(x=>console.log(x)); 

```

暂时不要太担心这个例子中发生了什么。在这里，事件是随时间发生的。这里不是有一个固定的数组元素，而是随时间发生的，500 毫秒后。

我们将向`arr`事件流添加一个事件监听器，当事件发生时，我们将在控制台上打印出元素。你可以看到数组和事件流中的方法之间的相似之处。现在，为了扩展这种相似性，假设你想要从这个列表中过滤掉所有的非数字。你可以使用`map`函数来处理这个事件流，就像你在数组上使用它一样，然后你会想要过滤结果，只显示整数。考虑以下代码行：

```js
    var arr = [1,1,13,'Rx',0,0]; 
    var result = arr.map(x => parseInt(x)).filter(x => !isNan(x)); 
    console.log(result); 

```

有趣的是，相同的方法也适用于事件流。看一下以下代码示例：

```js
    var arr = Rx.Observable.interval(500).take(9).map(
      a=>[1,1,13,'Rx',0,0][a]); 
    var result = arr.map(x => parseInt(x)).filter(x => !isNaN(x)); 
    result.subscribe(x=>console.log(x)); 

```

这些只是更简单的例子，只是为了确保你开始看到事件流随时间流动。请暂时不要担心语法和结构。在我们能够看它们之前，我们需要确保我们理解如何在响应式编程中思考。事件流对于响应式编程至关重要；它们允许你在声明时定义值的动态行为（定义来自 Andre Staltz 的博客）。

假设你有一个`a`变量，最初的值是`3`。然后，你有一个`b`变量，它是`10 * a`。如果我们在控制台上输出`b`，我们会看到`30`。考虑以下代码行：

```js
    let a = 3; 
    let b = a * 10; 
    console.log(b); //30 
    a = 4; 
    console.log(b); // Still 30 

```

我们知道结果是非常直接的。当我们将`a`的值更改为`4`时，`b`的值不会改变。这就是静态声明的工作原理。当我们谈论响应式编程和事件流时，这是人们在理解事件流如何流动时遇到困难的地方。理想情况下，我们希望创建一个公式，*b=a*10*，随着时间的推移，无论`a`的值如何变化，变化的值都会反映在公式中。

这就是我们可以通过事件流实现的。假设`a`是一个只有值`3`的事件流。然后，我们有`streamB`，它是`streamA`映射的结果。每个`a`值都将映射为`10 * a`。

如果我们给`streamB`添加一个事件监听器，并且我们控制台记录，我们会看到`b`是`30`。看一下以下的例子：

```js
    var streamA = Rx.Observable.of(3, 4); 
    var streamB = streamA.map(a => 10 * a); 
    streamB.subscribe(b => console.log(b)); 

```

如果我们这样做，我们将得到一个事件流，它只有两个事件。它有事件`3`，然后有事件`4`，当`a`改变时，`b`也会相应地改变。如果我们运行这个，我们会看到`b`是`30`和`40`。

现在我们已经花了一些时间来了解响应式编程的基础知识，你可能会问以下问题。

## 为什么你应该考虑响应式编程？

在我们编写现代网络和移动应用程序时，需要高度响应和交互式的 UI 应用程序，有必要找到一种处理实时事件而不会停止用户在 UI 上交互的方法。当你处理多个 UI 和服务器事件时，你将花费大部分时间编写处理这些事件的代码。这很繁琐。响应式编程为你提供了一个结构化的框架，以最少的代码处理异步事件，同时你可以专注于应用程序的业务逻辑。

响应式编程不仅限于 JavaScript。响应式扩展在许多平台和语言中都有，比如 Java、Scala、Clojure、Ruby、Python 和 Object C/Cocoa。`Rx.js`和`Bacon.js`是流行的提供响应式编程支持的 JavaScript 库。

深入研究`Rx.js`不是本章的目的。我们的目的是向你介绍响应式编程的概念。如果你有兴趣为你的项目采用响应式编程，你应该看看 Andre Staltz 的优秀介绍（[`gist.github.com/staltz/868e7e9bc2a7b8c1f754`](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)）。

# React

React 正在以 JavaScript 世界为风暴。Facebook 创建了 React 框架来解决一个古老的问题-如何有效地处理传统的**模型-视图-控制器**应用程序的视图部分。

React 提供了一种声明式和灵活的构建用户界面的方式。关于 React 最重要的一点是，它只处理一个东西-视图或 UI。React 不处理数据、数据绑定或其他任何东西。有完整的框架，比如 Angular，处理数据、绑定和 UI；React 不是那样的。

React 提供了一个模板语言和一小组函数来渲染 HTML。React 组件可以在内存中存储它们自己的状态。要构建一个完整的应用程序，你还需要其他部分；React 只是处理该应用程序的视图部分。

在编写复杂 UI 时的一个大挑战是在模型改变时管理 UI 元素的状态。React 提供了一个声明式 API，这样你就不必担心每次更新时确切发生了什么变化。这使得编写应用程序变得更加容易。React 使用虚拟 DOM 和差异算法，因此组件更新是可预测的，同时也足够快以用于高性能应用程序。

# 虚拟 DOM

让我们花一点时间来了解什么是虚拟 DOM。我们讨论了**DOM**（文档对象模型），一个网页上 HTML 元素的树结构。DOM 是事实上的，也是网页的主要渲染机制。DOM 的 API，比如`getElementById()`，允许遍历和修改 DOM 树中的元素。DOM 是一棵树，这种结构非常适合遍历和更新元素。然而，DOM 的遍历和更新都不是很快。对于一个大页面，DOM 树可能会相当大。当你想要一个有大量用户交互的复杂 UI 时，更新 DOM 元素可能会很繁琐和缓慢。我们已经尝试过 jQuery 和其他库来减少频繁 DOM 修改的繁琐语法，但 DOM 本身作为一种结构是相当有限的。

如果我们不必一遍又一遍地遍历 DOM 来修改元素呢？如果您只是声明组件应该是什么样子，然后让其他人处理如何渲染该组件的逻辑呢？react 就是这样做的。React 允许您声明您希望 UI 元素看起来像什么，并将低级别的 DOM 操作 API 抽象出来。除了这个非常有用的抽象之外，react 还做了一些相当聪明的事情来解决性能问题。

React 使用一种称为虚拟 DOM 的东西。虚拟 DOM 是 HTML DOM 的轻量级抽象。您可以将其视为 HTML DOM 的本地内存副本。React 使用它来执行呈现 UI 组件状态所需的所有计算。

您可以在[`facebook.github.io/react/docs/reconciliation.html`](https://facebook.github.io/react/docs/reconciliation.html)找到有关此优化的更多详细信息。

然而，React 的主要优势不仅仅是虚拟 DOM。React 是一个很棒的抽象，使得在开发大型应用程序时更容易进行组合、单向数据流和静态建模。

# 安装和运行 react

首先，让我们安装 react。早些时候，在您的计算机上安装和设置 react 需要处理一堆依赖项。但是，我们将使用一个相对更快的方法来让 react 运行起来。我们将使用`create-react-app`，通过它可以安装 react 而无需任何构建配置。安装是通过`npm`完成的，如下所示：

```js
    npm install -g create-react-app 

```

在这里，我们正在全局安装`create-react-app`节点模块。安装了`create-react-app`之后，您可以为应用程序设置目录。考虑以下命令：

```js
    create-react-app react-app 
    cd react-app/ 
    npm start 

```

然后，打开`http://localhost:3000/`来查看您的应用程序。您应该会看到类似以下屏幕截图的内容：

![Installing and running react](img/image_13_002.jpg)

如果您在编辑器中打开目录，您将看到为您创建了几个文件，如下面的屏幕截图所示：

![Installing and running react](img/image_13_003.jpg)

在这个项目中，`node_modules`是运行此项目所需的依赖项，也是 react 本身的依赖项。重要的目录是`src`，其中保存了源代码。对于这个示例，让我们只保留两个文件-`App.js`和`index.js`。`/public/index.html`文件应该只包含根`div`，它将用作我们的 react 组件的目标。考虑以下代码片段：

```js
    <!doctype html> 
    <html lang="en"> 
      <head> 
        <title>React App</title> 
      </head> 
      <body> 
 **<div id="root"></div>** 
      </body> 
    </html> 

```

进行这种更改的时候，您将看到以下错误：

![Installing and running react](img/image_13_004.jpg)

使用 react 进行开发的美妙之处在于代码更改是实时重新加载的，您可以立即得到反馈。

接下来，清空`App.js`的所有内容，并用以下代码替换它：

```js
    import React from 'react'; 
    const App = () => <h1>Hello React</h1> 
    export default App 

```

现在，转到`index.js`并删除`import ./index.css;`行。您无需做任何操作，比如重新启动服务器和刷新浏览器，就可以在浏览器上看到修改后的页面。考虑以下屏幕截图：

![Installing and running react](img/image_13_005.jpg)

在我们创建`HelloWorld` react 组件之前，有一些重要的事情需要注意。

在`App.js`和`index.js`中，我们导入了创建 react 组件所需的两个库。考虑以下代码行：

```js
    import React from 'react'; 
    import ReactDOM from 'react-dom'; 

```

在这里，我们导入了`React`，这是一个允许我们构建 react 组件的库。我们还导入了`ReactDOM`，这是一个允许我们放置我们的组件并在 DOM 的上下文中使用它们的库。然后，我们导入了我们刚刚工作过的组件-App 组件。

我们还在`App.js`中创建了我们的第一个组件。考虑以下代码行：

```js
    const App = () => <h1>Hello React</h1> 

```

这是一个无状态函数组件。创建组件的另一种方法是创建一个类组件。我们可以用以下类组件替换前面的组件：

```js
    class App extends React.Component { 
      render(){ 
        return <h1>Hello World</h1> 
      } 
    } 

```

这里有很多有趣的事情正在发生。首先，我们使用`class`关键字创建一个类组件，它继承自超类`React.Component`。

我们的组件`App`是一个 react 组件类或 react 组件类型。组件接受参数，也称为`props`，并通过`render`函数返回要显示的视图层次结构。

`render`方法返回要渲染的描述，然后 react 接受该描述并将其渲染到屏幕上。特别是，`render`返回一个 react 元素，它是要渲染的轻量级描述。大多数 react 开发人员使用一种称为 JSX 的特殊语法，这使得编写这些结构更容易。`<div />`语法在构建时转换为`React.createElement`(`'div'`)。JSX 表达式`<h1>Hello World</h1>`在构建时转换为以下内容：

```js
    return React.createElement('h1', null, 'Hello World'); 

```

类组件和无状态函数组件之间的区别在于，类组件可以包含状态，而无状态（因此名称为）函数组件不能。

react 组件的`render`方法只允许返回单个节点。如果你做了以下操作：

```js
    return <h1>Hello World</h1><p>React Rocks</p> 

```

你会得到以下错误：

```js
    Error in ./src/App.js 
    Syntax error: Adjacent JSX elements must be wrapped in 
      an enclosing tag (4:31) 

```

这是因为你实质上返回了两个`React.createElement`函数，这不是有效的 JavaScript。虽然这可能看起来像是一个破坏者，但这很容易解决。我们可以将我们的节点包装成一个父节点，并从`render`函数返回该父节点。我们可以创建一个父`div`，并将其他节点包装在其中。考虑以下示例：

```js
    render(){ 
        return ( 
          <div> 
            <h1>Hello World</h1> 
            <p>React Rocks</p> 
          </div> 
          ) 
    } 

```

## 组件和 props

组件在概念上可以被视为 JavaScript 函数。它们像普通函数一样接受任意数量的输入。这些输入被称为 props。为了说明这一点，让我们考虑以下函数：

```js
    function Greet(props) { 
      return <h1>Hello, {props.name}</h1>; 
    } 

```

这是一个普通函数，也是一个有效的 react 组件。它接受一个名为`props`的输入，并返回一个有效的 JSX。我们可以在 JSX 中使用`props`，使用大括号和`name`等属性使用标准对象表示法。现在`Greet`是一个一流的 react 组件，让我们在`render()`函数中使用它，如下所示：

```js
    render(){ 
      return ( 
       return <Greet name="Joe"/> 
      ) 
    } 

```

我们将`Greet()`作为一个普通组件调用，并将`this.props`传递给它。自定义组件必须大写。React 认为以小写字母开头的组件名称是标准 HTML 标签，并期望自定义组件名称以大写字母开头。正如我们之前看到的，我们可以使用 ES6 类创建一个类组件。这个组件是`React.component`的子类。与我们的`Greet`函数等效的组件如下：

```js
    class Greet extends React.Component { 
      render(){ 
          return <h1>Hello, {this.props.name}</h1> 
      } 
    } 

```

就实际目的而言，我们将使用这种创建组件的方法。我们很快就会知道为什么。

一个重要的要点是组件不能修改自己的 props。这可能看起来像是一个限制，因为在几乎所有非平凡的应用程序中，你都希望用户交互在 react 中改变 UI 组件状态，例如，在表单中更新出生日期，`props`是只读的，但有一个更健壮的机制来处理 UI 更新。

## 状态

状态类似于 props，但它是私有的，并且完全由组件控制。正如我们之前看到的，React 中的函数组件和类组件是等效的，一个重要的区别是状态仅在类组件中可用。因此，就实际目的而言，我们将使用类组件。

我们可以改变我们现有的问候示例来使用状态，每当状态改变时，我们将更新我们的`Greet`组件以反映更改的值。

首先，我们将在我们的`App.js`中设置状态，如下所示：

```js
    class Greet extends React.Component { 
 **constructor(props) {**
 **super(props);** 
 **this.state = {** 
**greeting: "this is default greeting text"** 
**}** 
 **}** 
      render(){ 
          return <h1>{this.state.greeting}, {this.props.name} </h1> 
      } 
    } 

```

在这个例子中有一些重要的事情需要注意。首先，我们调用类`constructor`来初始化`this.state`。我们还调用基类构造函数`super()`，并将`props`传递给它。调用`super()`后，我们通过将`this.state`设置为一个对象来初始化我们的默认状态。例如，我们在这里给一个`greeting`属性赋值。在`render`方法中，我们将使用`{this.state.greeting}`来使用这个属性。设置了初始状态后，我们可以添加 UI 元素来更新这个状态。让我们添加一个输入框，当输入框改变时，我们将更新我们的状态和`greeting`元素。考虑以下代码行：

```js
    class Greet extends React.Component { 
      constructor(props) { 
        super(props); 
        this.state = { 
          greeting: "this is default greeting text" 
        } 
      } 
 **updateGreeting(event){** 
**this.setState({** 
**greeting:
      event.target.value,**
 **})**
 **}** 
      render(){ 
          return ( 
          <div>   
 **<input type="text" onChange={this.updateGreeting.bind(this)}/>** 
            <h1>{this.state.greeting}, {this.props.name} </h1> 
           </div>  
          ) 
        } 
    } 

```

在这里，我们添加一个输入框，并在输入框的`onChange`方法被调用时更新组件的状态。我们使用自定义的`updateGreeting()`方法通过调用`this.setState`和更新属性来更新状态。当您运行此示例时，您会注意到当您在文本框上输入内容时，只有`greeting`元素被更新，而`name`没有。看一下下面的截图：

![State](img/image_13_006.jpg)

React 的一个重要特性是，一个 React 组件可以输出或渲染其他 React 组件。我们这里有一个非常简单的组件。它有一个值为文本的状态。它有一个`update`方法，它将从事件中更新文本的值。我们将创建一个新的组件。这将是一个无状态函数组件。我们将它称为 widget。它将接受`props`。我们将在这里返回这个 JSX 输入。考虑以下代码片段：

```js
    render(){ 
        return ( 
          <div>   
 **<Widget update={this.updateGreeting.bind(this)} />** 
 **<Widget update={this.updateGreeting.bind(this)} />** 
 **<Widget update={this.updateGreeting.bind(this)} />** 
          <h1>{this.state.greeting}, {this.props.name} </h1> 
          </div>  
        ) 
      } 
    } 
    const Widget = (props) => <input type="text" 
      onChange={props.update}/> 

```

首先，我们将输入元素提取到一个无状态函数组件中，并将其称为`Widget`。我们将`props`传递给此组件。然后，我们将`onChange`更改为使用`props.update`。现在，在我们的`render`方法中，我们使用`Widget`组件并传递一个绑定`updateGreeting()`方法的 prop `update`。现在`Widget`是一个组件，我们可以在`Greet`组件的任何地方重用它。我们创建了`Widget`的三个实例，当任何一个`Widget`被更新时，问候文本将被更新，如下面的截图所示：

![State](img/image_13_007.jpg)

## 生命周期事件

当您有一堆组件和几个状态更改和事件时，清理工作变得很重要。React 为您提供了几个组件生命周期钩子来处理组件的生命周期事件。了解组件的生命周期将使您能够在创建或销毁组件时执行某些操作。此外，它还为您提供了决定是否应该首先更新组件的机会，并根据`props`或状态更改做出反应。

组件经历三个阶段-挂载、更新和卸载。对于每个阶段，我们都有钩子。看一下下面的图表：

![Life cycle events](img/image_13_008.jpg)

当组件初始渲染时，会调用两个方法`getDefaultProps`和`getInitialState`，正如它们的名称所暗示的，我们可以在这些方法中设置组件的默认`props`和初始状态。

`componentWillMount`在执行`render`方法之前被调用。我们已经知道`render`是我们返回要渲染的组件的地方。一旦`render`方法完成，`componentDidMount`方法就会被调用。您可以在此方法中访问 DOM，并建议在此方法中执行任何 DOM 交互。

状态更改会调用一些方法。`shouldComponentUpdate`方法在`render`方法之前被调用，它让我们决定是否应该允许重新渲染或跳过。这个方法在初始渲染时从未被调用。`componentWillUpdate`方法在`shouldComponentUpdate`方法返回`true`后立即被调用。`componentDidUpdate`方法在`render`完成后被渲染。

对`props`对象的任何更改都会触发类似状态更改的方法。另一个被调用的方法是`componentWillReceiveProps`；它仅在`props`发生变化时被调用，而且不是初始渲染。您可以在此方法中基于新旧 props 更新状态。

当组件从 DOM 中移除时，将调用`componentWillUnmount`。这是一个执行清理的有用方法。

React 的一个很棒的地方是，当您开始使用它时，这个框架对您来说会感觉非常自然。您只需要学习很少的移动部分，抽象程度恰到好处。

# 摘要

本章旨在介绍一些最近备受关注的重要新观念。响应式编程和 React 都可以显著提高程序员的生产力。React 绝对是由 Facebook 和 Netflix 等公司支持的最重要的新兴技术之一。

本章旨在为您介绍这两种技术，并帮助您开始更详细地探索它们。
