# 第三章：一个 React Bootstrap 个人联系人管理器

在本章中，我们将学习如何使用 React 构建个人联系人管理器，它是一个用于构建**用户界面**（**UI**）的小组件库。通过学习 React，您将获得使用当前最流行的库之一的能力，并开始了解何时以及如何使用绑定的力量来简化您的代码。

探索 React 将帮助我们了解如何为客户端编写现代应用程序，并研究其要求。

为了帮助我们开发应用程序，本章将涵盖以下主题：

+   创建一个模拟布局来检查我们的布局

+   创建我们的 React 应用程序

+   使用`tslint`分析和格式化代码

+   添加 Bootstrap 支持

+   在 React 中使用 tsx 组件

+   React 中的`App`组件

+   展示我们的个人详细信息 UI

+   使用绑定简化我们的更新

+   创建验证器并将它们应用为验证

+   在 React 组件中应用验证

+   创建并将数据发送到 IndexedDB 数据库

# 技术要求

由于我们使用 IndexedDB 数据库来存储数据，将需要一个现代的网络浏览器，如 Chrome（11 版或更高版本）或 Firefox（4 版或更高版本）。完成的项目可以从[`github.com/PacktPublishing/Advanced-TypeScript-3-Programming-Projects/tree/master/chapter03`](https://github.com/PacktPublishing/Advanced-TypeScript-3-Programming-Projects/tree/master/chapter03)下载。下载项目后，您将需要使用`npm install`安装软件包要求。

# 了解项目概述

我们将使用 React 构建一个个人联系人管理器数据库。数据存储在客户端上，使用标准的 IndexedDB 数据库。完成后，我们的应用程序将如下所示：

![](img/d5353a60-f978-4180-bf0b-a86ad7ce205c.png)

您应该能够在本章中完成这些步骤，与 GitHub 存储库中的代码一起工作，大约需要两个小时。

# 开始使用组件

本章依赖于 Node.js，可在[`nodejs.org/`](https://nodejs.org/)上获得。随着我们在本章中的进展，我们将安装以下组件：

+   `@types/bootstrap`（4.1.2 或更高版本）

+   `@types/reactstrap`（6.4.3 或更高版本）

+   `bootstrap`（4.1.3 或更高版本）

+   `react`（16.6.3 或更高版本）

+   `react-dom`（16.6.3 或更高版本）

+   `react-script-ts`（3.1.0 或更高版本）

+   `reactstrap`（6.5.0 或更高版本）

+   `create-react-app`（2.1.2 或更高版本）

# 创建一个带有 TypeScript 支持的 React Bootstrap 项目

正如我们在第二章中讨论的*使用 TypeScript 创建 Markdown 编辑器*，最好的方法是首先收集我们将要编写的应用程序的需求。以下是本章的要求：

+   用户将能够创建一个人的新详细信息或编辑它们

+   这些详细信息将保存到客户端数据库

+   用户将能够加载所有人的列表

+   用户将能够删除一个人的个人详细信息

+   个人详细信息将包括名字和姓氏、地址（由两个地址行、城镇、县和邮政编码组成）、电话号码和出生日期。

+   个人详细信息将保存到数据库中

+   名字至少为一个字符，姓氏至少为两个字符

+   地址行 1、城镇和县至少为五个字符

+   邮政编码将符合大多数邮政编码的美国标准

+   电话号码将符合标准的美国电话格式

+   用户可以通过点击按钮清除详细信息

# 创建我们的模拟布局

一旦我们有了我们的要求，通常最好草拟一些我们认为应用程序布局应该是什么样的草图。我们想做的是创建一个布局，显示我们正在使用网页浏览器布局的草图格式。我们希望它看起来像是草绘的，因为我们与客户互动的方式。我们希望他们能够了解我们应用程序的大致布局，而不会陷入诸如特定按钮有多宽等细节中。

特别有用的是使用诸如[`ninjamock.com`](https://ninjamock.com)这样的工具来创建我们界面的线框草图。这些草图可以在线与客户或其他团队成员共享，并直接添加评论。以下草图示意了我们完成后希望我们的界面看起来的样子：

![](img/22334fc2-a57c-47ec-88b6-7ea1a5f928cc.png)

# 创建我们的应用程序

在我们开始编写代码之前，我们需要安装 React。虽然可以手动创建我们需要的 React 基础设施，但大多数人使用`create-react-app`命令来创建 React 应用程序。我们不会做任何不同的事情，所以我们也将使用`create-react-app`命令。React 默认不使用 TypeScript，因此我们将在用于创建应用程序的命令中添加一些额外的内容，以为我们提供所有需要的 TypeScript 功能。我们使用`create-react-app`，给它我们应用程序的名称和一个额外的`scripts-version`参数，为我们挂接 TypeScript：

```ts
npx create-react-app chapter03 --scripts-version=react-scripts-ts
```

如果您以前安装过 Node.js 包，您可能会认为在前面的命令中有一个错误，并且我们应该使用`npm`来安装`create-react-app`。但是，我们使用`npx`代替`npm`，因为`npx`是**Node Package Manager**（**NPM**）的增强版本。使用`npx`，我们省去了运行`npm install create-react-app`来安装`create-react-app`包，然后手动运行`create-react-app`来启动进程的步骤。使用`npx`确实有助于加快我们的开发工作流程。

创建完我们的应用程序后，我们打开`Chapter03`目录并运行以下命令：

```ts
npm start
```

假设我们已经设置了默认浏览器，它应该打开到`http://localhost:3000`，这是该应用程序的默认网页。这将提供一个包含默认 React 示例的标准网页。现在我们要做的是编辑`public/index.html`文件并为其设置一个标题。我们将把我们的标题设置为`Advanced TypeScript - Personal Contacts Manager`。虽然这个文件的内容看起来很少，但它包含了我们在 HTML 方面所需要的一切，即一个名为`root`的`div`元素。这是我们的 React 代码将依附的挂钩，我们稍后会讨论。我们可以实时编辑我们的应用程序，以便我们所做的任何更改都将被编译并自动返回到浏览器：

```ts
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="theme-color" content="#000000">
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json">
    <link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico">
    <title>Advanced TypeScript - Personal Contacts Manager</title>
  </head>
  <body>
    <noscript>
      You need to enable JavaScript to run this app.
    </noscript>
    <div id="root"></div>
  </body>
</html>
```

# 使用 tslint 格式化我们的代码

一旦我们创建了我们的应用程序，我们使用了一个叫做`tslint`的东西，它通过查找潜在问题来分析我们的代码。请注意，当我们创建我们的应用程序时，对此的支持已经自动添加。运行的`tslint`版本应用了一套非常激进的规则，我们检查我们的代码是否符合这些规则。我在我的代码库中使用了完整的`tslint`规则集；但是，如果您想放松规则，只需将`tslint.json`文件更改为以下内容：

```ts
{
  "extends": [],
  "defaultSeverity" : "warning",
  "linterOptions": {
    "exclude": [
      "config/**/*.js",
      "node_modules/**/*.ts",
      "coverage/lcov-report/*.js"
    ]
  }
}
```

# 添加 Bootstrap 支持

我们的应用程序需要做的一件事是引入对 Bootstrap 的支持。这不是 React 默认提供的功能，因此我们需要使用其他包添加这个功能：

1.  安装 Bootstrap 如下：

```ts
npm install --save bootstrap
```

1.  有了这个，我们现在可以自由地使用一个 React-ready 的 Bootstrap 组件。我们将使用`reactstrap`包，因为这个包以 React 友好的方式针对 Bootstrap 4：

```ts
npm install --save reactstrap react react-dom
```

1.  `reactstrap`不是一个 TypeScript 组件，所以我们需要安装这个和 Bootstrap 的`DefinitelyTyped`定义：

```ts
npm install --save @types/reactstrap
npm install --save @types/bootstrap
```

1.  有了这个，我们现在可以添加 Bootstrap CSS 文件。为了做到这一点，我们将通过在`index.tsx`文件中添加对我们本地安装的 Bootstrap CSS 文件的引用，添加以下`import`到文件的顶部：

```ts
import "bootstrap/dist/css/bootstrap.min.css";
```

在这里，我们使用本地的 Bootstrap 文件是为了方便。正如我们在第一章中讨论的*高级 TypeScript 特性*，我们希望将其更改为在生产版本中使用 CDN 源。

1.  为了整理一下，从`src/index.tsx`中删除以下行，然后从磁盘中删除匹配的`.css`文件：

```ts
import './index.css'
```

# React 使用 tsx 组件

你现在可能会问一个问题，为什么索引文件有不同的扩展名？也就是说，为什么是`.tsx`而不是`.ts`？要回答这些问题，我们必须稍微改变我们对扩展的心智形象，并谈谈为什么 React 使用`.jsx`文件而不是`.js`（`.tsx`版本是`.jsx`的 TypeScript 等价物）。

这些 JSX 文件是 JavaScript 的扩展，会被转译成 JavaScript。如果你试图在 JavaScript 中直接运行它们，那么如果它们包含任何这些扩展，你将会得到运行时错误。在传统的 React 中，有一个转译阶段，它会将 JSX 文件转换为 JavaScript，通过将代码扩展为标准的 JavaScript。实际上，这是一种我们从 TypeScript 中得到的编译阶段。使用 TypeScript React，我们得到了相同的结果，TSX 文件最终会成为 JavaScript 文件。

那么，现在的问题是为什么我们实际上需要这些扩展？为了回答这个问题，我们将分析`index.tsx`文件。这是我们添加了 Bootstrap CSS 文件后文件的样子：

```ts
import "bootstrap/dist/css/bootstrap.min.css";
import * as React from 'react';
import * as ReactDOM from 'react-dom';
import App from './App';

import registerServiceWorker from './registerServiceWorker';

ReactDOM.render(
  <App />,
  document.getElementById('root') as HTMLElement
);
registerServiceWorker();
```

`import`语句现在应该对我们来说很熟悉，`registerServiceWorker`是添加到代码中的行为，通过从缓存中提供资产，而不是一次又一次地重新加载它们，来提供更快的生产应用程序。React 的一个关键原则是它应该尽可能快，这就是`ReactDOM.render`的作用所在。如果我们阅读这段代码，事情应该变得清晰。它正在寻找在我们提供的 HTML 页面中标记为根的元素——我们在`index.html`文件中看到了这一点。我们在这里使用`as HTMLElement`语法的原因是我们想让 TypeScript 知道这是什么类型（这个参数要么派生自一个元素，要么为空——是的，这意味着底层是一个联合类型）。

现在，我们需要一个特殊的扩展的原因是因为代码中有一个说`<App />`的部分。我们在这里所做的是将一段 XML 代码嵌入到我们的语句中。在这个特定的例子中，我们告诉我们的`render`方法渲染一个名为`App`的组件，这个组件在`App.tsx`文件中被定义。

# React 如何使用虚拟 DOM 来提高响应性

我略过了为什么使用`render`方法，现在是时候解释一下 React 的秘密武器，也就是虚拟**文档对象模型**（**DOM**）。如果你已经开发了一段时间的 Web 应用程序，你可能已经了解了 DOM。如果你从未遇到过这个，DOM 是一个描述网页将会是什么样子的实体。Web 浏览器非常依赖 DOM，并且随着多年的发展，它可能变得相当笨重。浏览器制造商只能尽力加快 DOM 的速度。如果他们想要能够提供旧的网页，那么他们必须支持完整的 DOM。

虚拟 DOM 是标准 DOM 的轻量级副本。它之所以轻量级是因为它缺少标准 DOM 的一个重要特性；也就是说，它不必呈现到屏幕上。当 React 运行`render`方法时，它遍历每个`.tsx`（或 JavaScript 中的`.jsx`）文件并在那里执行渲染代码。然后将此渲染代码与上次运行的渲染的副本进行比较，以确定确切发生了什么变化。只有那些发生变化的元素才会在屏幕上更新。这个比较阶段是我们必须使用虚拟 DOM 的原因。使用这种方法更快地告诉哪些元素需要更新，只有那些发生变化的元素才需要更新。

# 我们的 React App 组件

我们已经提到了 React 中组件的使用。默认情况下，我们将始终有一个`App`组件。这是将呈现到我们 HTML 根元素的组件。我们的组件源自`React.Component`，因此我们的`App`组件的开头看起来像下面这样：

```ts
import * as React from 'react';
import './App.css';

export default class App extends React.Component {

}
```

当然，我们的组件需要一个常用的方法来触发组件的渲染。毫不奇怪，这个方法被称为`render`。由于我们正在使用 Bootstrap 来显示我们的 UI，我们希望呈现一个与我们的`Container` div 相关的组件。为此，我们将使用`reactstrap`中的`Container`组件（并引入我们将用于显示界面的核心组件）：

```ts
import * as React from 'react';
import './App.css';
import Container from 'reactstrap/lib/Container';
import PersonalDetails from './PersonalDetails';
export default class App extends React.Component {
  public render() {
    return (
      <Container>
        <PersonalDetails />
      </Container>
    );
  }
}
```

# 显示个人详细信息界面

我们将创建一个名为`PersonalDetails`的类。这个类将在`render`方法中呈现出我们界面的核心。同样，我们使用`reactstrap`来布置界面的各个部分。在我们分解`render`方法的复杂性之前，让我们先看看这一切是什么样子：

```ts
import * as React from 'react';
import Button from 'reactstrap/lib/Button';
import Col from 'reactstrap/lib/Col';
import Row from 'reactstrap/lib/Row';

export default class PersonalDetails extends React.Component {

  public render() {
    return (
      <Row>
        <Col lg="8">
          <Row>
            <Col><h4 className="mb-3">Personal details</h4></Col>
          </Row>
          <Row>
            <Col><label htmlFor="firstName">First name</label></Col>
            <Col><label htmlFor="lastName">Last name</label></Col>
          </Row>
          <Row>
            <Col>
              <input type="text" id="firstName" className="form-control" placeholder="First name" />
            </Col>
            <Col><input type="text" id="lastName" className="form-control" placeholder="Last name" /></Col>
          </Row>
... Code omitted for brevity
        <Col>
          <Col>
            <Row>
              <Col lg="6"><Button size="lg" color="success">Load</Button></Col>
              <Col lg="6"><Button size="lg" color="info">New Person</Button></Col>
            </Row>
          </Col>
        </Col>
      </Row>
    );
  }
}
```

正如您所看到的，这个方法中有很多事情要做；然而，其中绝大部分是重复的代码，用于复制行和列的 Bootstrap 元素。例如，如果我们看一下`postcode`和`phoneNumber`元素的布局，我们会发现我们正在布置两行，每行有两个显式的列。在 Bootstrap 术语中，其中一个`Col`元素是三个大尺寸，另一个是四个大尺寸（我们将留给 Bootstrap 来考虑剩下的空列）：

```ts
<Row>
  <Col lg="3"><label htmlFor="postcode">Postal/ZipCode</label></Col>
  <Col lg="4"><label htmlFor="phoneNumber">Phone number</label></Col>
</Row>
<Row>
  <Col lg="3"><input type="text" id="postcode" className="form-control" /></Col>
  <Col lg="4"><input type="text" id="phoneNumber" className="form-control" /></Col>
</Row>
```

看着标签和输入元素，我们可以看到有两个不熟悉的元素。当然，在标签中正确的键是`for`，我们应该在输入中使用`class`来引用 CSS 类？我们在这里使用替代键的原因是`for`和`class`是 JavaScript 关键字。由于 React 允许我们在渲染中混合代码和标记语言，React 必须使用不同的关键字。这意味着我们使用`htmlFor`来替换`for`，使用`className`来替换`class`。回到我们谈论虚拟 DOM 时，这给了我们一个重要的提示，即这些 HTML 元素是用于类似目的的副本，而不是元素本身。

# 使用绑定简化更新值

许多现代框架的一个特性是使用绑定来消除手动更新输入或触发事件的需要。使用绑定的想法是，框架在 UI 元素和代码之间建立连接，例如属性，监视基础值的变化，然后在检测到变化时触发更新。正确使用时，这可以减少我们编写代码的繁琐工作，更重要的是有助于减少错误。

# 提供要绑定的状态

使用 React 进行绑定的想法是我们有一个需要绑定的状态。对于创建要在屏幕上显示的数据，我们的状态可以简单地是描述我们要使用的属性的接口。对于单个联系人，这将转化为我们的状态看起来像这样：

```ts
export interface IPersonState {
  FirstName: string,
  LastName: string,
  Address1: string,
  Address2: StringOrNull,
  Town: string,
  County: string,
  PhoneNumber: string;
  Postcode: string,
  DateOfBirth: StringOrNull,
  PersonId : string
}
```

请注意，我们创建了一个名为`StringOrNull`的联合类型作为一种便利。我们将把它放在一个名为`Types.tsx`的文件中，使其看起来像这样：

```ts
export type StringOrNull = string | null;
```

现在我们要做的是告诉我们的组件它将使用什么状态。首先要做的是更新我们的类定义，使其看起来像这样：

```ts
export default class PersonalDetails extends React.Component<IProps, IPersonState>
```

这遵循了一个惯例，即属性从父级传递到我们的类中，而状态来自我们的本地组件。这种属性和状态的分离对我们很重要，因为它为父组件与子组件之间的通信提供了一种方式（以及子组件与父组件之间的回传），同时仍然能够管理组件作为状态所需的数据和行为。

在这里，我们的属性在一个名为`IProps`的接口中定义。现在我们已经告诉 React 我们的状态的*形状*将会是什么，React 和 TypeScript 会用这个信息创建一个`ReadOnly<IPersonState>`属性。因此，确保我们使用正确的状态是很重要的。如果我们对状态使用了错误的类型，TypeScript 会通知我们这一点。

请注意，前面的陈述有一个警告。如果我们有两个完全相同形状的接口，那么 TypeScript 会将它们视为等价的。因此，即使 TypeScript 期望`IState`，如果我们提供了一个名为`IMyOtherState`的东西，它具有完全相同的属性，那么 TypeScript 也会乐意让我们使用它。当然，问题是，为什么我们要首先复制接口呢？我想不出很多情况下我们会这样做，所以使用正确的状态的想法几乎适用于我们可能遇到的所有情况。

我们的`app.tsx`文件将会为状态创建一个默认值，并将其作为属性传递给我们的组件。默认状态是当用户按下清除按钮清除当前编辑的条目，或者按下新建人员按钮开始添加新人员时将会应用的状态。我们的`IProps`接口看起来是这样的：

```ts
interface IProps {
  DefaultState : IPersonState
}
```

一开始可能会有些令人困惑的是，我之前的陈述和属性和状态是不同的这个想法之间存在潜在的矛盾——状态是组件本地的东西，但我们将状态作为属性的一部分传递下去。我故意在名称中使用状态的一部分来强调这一点。我们传递的值可以是任何东西。它们不必代表任何状态；它们可以只是组件调用以触发父级响应的函数。我们的组件将接收这个属性，并且它将负责将其需要的任何部分转换为状态。

有了这个，我们就可以准备改变我们的`App.tsx`文件，创建我们的默认状态，并将其传递给我们的`PersonalDetails`组件。正如我们在下面的代码中所看到的，`IProps`接口中的属性成为了`<PersonalDetails ..`行中的一个参数。我们向我们的属性接口添加更多的项目，我们就需要在这一行中添加更多的参数：

```ts
import * as React from 'react';
import Container from 'reactstrap/lib/Container';
import './App.css';
import PersonalDetails from './PersonalDetails';
import { IPersonState } from "./State";

export default class App extends React.Component {
  private defaultPerson : IPersonState = {
    Address1: "",
    Address2: null,
    County: "",
    DateOfBirth : new Date().toISOString().substring(0,10),
    FirstName: "",
    LastName: "",
    PersonId : "",
    PhoneNumber: "",
    Postcode: "",
    Town: ""
  }
  public render() {
    return (
      <Container>
        <PersonalDetails DefaultState={this.defaultPerson} />
      </Container>
    );
  }
}
```

当我们想要将日期挂接到日期选择器组件时，使用 JavaScript 处理日期可能会让人望而却步。日期选择器期望以 YYYY-MM-DD 的格式接收日期。因此，我们使用`new Date().toISOString().substring(0,10)`语法来获取今天的日期，其中包括一个时间组件，并且只从中检索 YYYY-MM-DD 部分。尽管日期选择器期望日期以这种格式呈现，但它并没有规定这是屏幕上显示的格式。屏幕上的格式应该遵守用户的本地设置。

有趣的是，我们对支持传递属性所做的更改已经在这里看到了绑定的作用。在`render`方法中，我们设置`Default={this.defaultPerson}`时，我们正在使用绑定。在这里使用`{}`，我们告诉 React 我们想要绑定到某些东西，无论是属性还是事件。我们在 React 中会经常遇到绑定。

现在我们将在`PersonalDetails.tsx`中添加一个构造函数，以支持从`App.tsx`传入的属性：

```ts
private defaultState: Readonly<IPersonState>;
constructor(props: IProps) {
  super(props);
  this.defaultState = props.DefaultState;
  this.state = props.DefaultState;
}
```

我们在这里做两件事。首先，我们正在设置一个默认状态，以便在需要时返回到我们从父级那里收到的状态；其次，我们正在为此页面设置状态。我们不必在我们的代码中创建一个状态属性，因为这是由`React.Component`为我们提供的。这是学习如何将我们的属性从父级绑定到状态的最后一部分。

对状态的更改不会反映在父级 props 中。如果我们想要明确地将一个值设置回父组件，这将要求我们触发对`props.DefaultState`的更改。如果可能的话，我建议不要直接这样做。

好的。让我们设置我们的名字和姓氏元素，使其与我们状态的绑定一起工作。这里的想法是，如果我们在代码中更新名字或姓氏的状态，这将自动在我们的 UI 中更新。因此，让我们根据需要更改条目：

```ts
<Row>
  <Col><input type="text" id="firstName" className="form-control" value={this.state.FirstName} placeholder="First name" /></Col>
  <Col><input type="text" id="lastName" className="form-control" value={this.state.LastName} placeholder="Last name" /></Col>
</Row>
```

现在，如果我们运行我们的应用程序，我们会发现条目已绑定到底层状态。然而，这段代码存在一个问题。如果我们尝试在任一文本框中输入，我们会发现没有任何反应。实际的文本输入被拒绝了。这并不意味着我们做错了什么，而是我们只是在这里看到了整体图片的一部分。我们需要理解的是，React 为我们提供了一个只读版本的状态。如果我们希望我们的 UI 更新我们的状态，我们必须通过对变化做出反应，然后适当地设置状态来明确地选择这一点。首先，我们将编写一个事件处理程序来处理文本更改时的状态设置：

```ts
private updateBinding = (event: any) => {
  switch (event.target.id) {
    case `firstName`:
      this.setState({ FirstName: event.target.value });
      break;
    case `lastName`:
      this.setState({ LastName: event.target.value });
      break;
  }
}
```

有了这个设置，我们现在可以使用`onChange`属性更新我们的输入以触发此更新。同样，我们将使用绑定将`onChange`事件与作为结果触发的代码匹配：

```ts
<Row>
  <Col>
    <input type="text" id="firstName" className="form-control" value={this.state.FirstName} onChange={this.updateBinding} placeholder="First name" />
  </Col>
  <Col><input type="text" id="lastName" className="form-control" value={this.state.LastName} onChange={this.updateBinding} placeholder="Last name" /></Col>
</Row>
```

从这段代码中，我们可以清楚地看到`this.state`为我们提供了对我们在组件中设置的底层状态的访问，并且我们需要使用`this.setState`来更改它。`this.setState`的语法应该看起来很熟悉，因为它与我们在 TypeScript 中多次遇到的键值匹配。在这个阶段，我们现在可以更新我们的其余输入组件以支持这种双向绑定。首先，我们将扩展我们的`updateBinding`代码如下：

```ts
private updateBinding = (event: any) => {
  switch (event.target.id) {
    case `firstName`:
      this.setState({ FirstName: event.target.value });
      break;
    case `lastName`:
      this.setState({ LastName: event.target.value });
      break;
    case `addr1`:
      this.setState({ Address1: event.target.value });
      break;
    case `addr2`:
      this.setState({ Address2: event.target.value });
      break;
    case `town`:
      this.setState({ Town: event.target.value });
      break;
    case `county`:
      this.setState({ County: event.target.value });
      break;
    case `postcode`:
      this.setState({ Postcode: event.target.value });
      break;
    case `phoneNumber`:
      this.setState({ PhoneNumber: event.target.value });
      break;
    case `dateOfBirth`:
      this.setState({ DateOfBirth: event.target.value });
      break;
  }
}
```

我们不打算将我们需要对实际输入进行的所有更改都进行代码转储。我们只需要更新每个输入以将值与相应的状态元素匹配，并在每种情况下添加相同的`onChange`处理程序。

由于`Address2`可能为空，我们在绑定上使用`!`运算符，使其看起来略有不同：`value={this.state.Address2!}`。

# 验证用户输入和验证器的使用

在这个阶段，我们真的应该考虑验证用户的输入。我们将在我们的代码中引入两种类型的验证。第一种是最小长度验证。换句话说，我们将确保一些条目在被视为有效之前必须具有最少数量的条目。第二种验证类型使用称为正则表达式的东西来验证它。这意味着它接受输入并将其与一组规则进行比较，以查看是否有匹配；如果您对正则表达式不熟悉，这些表达式可能看起来有点奇怪，因此我们将对它们进行分解，以确切了解我们正在应用的规则。

我们将把我们的验证分解为三个部分：

1.  提供检查功能的类，比如应用正则表达式。我们将称这些为验证器。

1.  将验证项目应用到状态的不同部分的类。我们将称这些类为验证。

1.  将调用验证项目并使用失败验证的详细信息更新 UI 的组件。这将是一个名为`FormValidation.tsx`的新组件。

我们将首先创建一个名为`IValidator`的接口。这个接口将接受一个通用参数，以便我们可以将它应用到几乎任何我们想要的东西上。由于验证将告诉我们输入是否有效，它将有一个名为`IsValid`的单一方法，该方法接受相关输入，然后返回一个`boolean`值：

```ts
interface IValidator<T> {
  IsValid(input : T) : boolean;
}
```

我们要编写的第一个验证器是检查字符串是否具有最小数量的字符，我们将通过构造函数设置。我们还将防范用户未提供输入的情况，通过在输入为 null 时从`IsValid`返回`false`：

```ts
export class MinLengthValidator implements IValidator<StringOrNull> {
  private minLength : number;
  constructor(minLength : number) {
    this.minLength = minLength;
  }
  public IsValid(input : StringOrNull) : boolean {
    if (!input) {
      return false;
    }
    return input.length >= this.minLength;
  }
}
```

我们要创建的另一个验证器稍微复杂一些。这个验证器接受一个字符串，用它来创建一个叫做正则表达式的东西。正则表达式实际上是一种提供一组规则来测试我们的输入字符串的迷你语言。在这种情况下，构成我们正则表达式的规则被传递到我们的构造函数中。构造函数将实例化 JavaScript 正则表达式引擎（`RegExp`）的一个实例。与最小长度验证类似，我们确保如果没有输入则返回`false`。如果有输入，我们返回我们正则表达式测试的结果：

```ts
import { StringOrNull } from 'src/Types';

export class RegularExpressionValidator implements IValidator<StringOrNull> {
  private regex : RegExp;
  constructor(expression : string) {
    this.regex = new RegExp(expression);
  }
  public IsValid (input : StringOrNull) : boolean {
    if (!input) {
      return false;
    }
    return this.regex.test(input);
  } 
}
```

现在我们有了验证器，我们将研究如何应用它们。也许不会让人感到意外的是，我们要做的第一件事是定义一个接口，形成我们希望验证做的*合同*。我们的`Validate`方法将接受来自我们组件的`IPersonState`状态，验证其中的项目，然后返回一个验证失败的数组。

```ts
export interface IValidation {
  Validate(state : IPersonState, errors : string[]) : void;
}
```

我决定将验证分解为以下三个领域：

1.  验证地址

1.  验证姓名

1.  验证电话号码

# 验证地址

我们的地址验证将使用`MinLengthValidator`和`RegularExpressionValidator`验证器：

```ts
export class AddressValidation implements IValidation {
  private readonly minLengthValidator : MinLengthValidator = new MinLengthValidator(5);
  private readonly zipCodeValidator : RegularExpressionValidator 
    = new RegularExpressionValidator("^[0-9]{5}(?:-[0-9]{4})?$");
}
```

最小长度验证足够简单，但如果你以前从未见过这种类型的语法，正则表达式可能会让人望而生畏。在查看我们的验证代码之前，我们将分解正则表达式的工作。

第一个字符`^`告诉我们验证将从字符串的开头开始。如果我们省略这个字符，那么意味着我们的匹配可以出现在文本的任何地方。使用`[0-9]`告诉正则表达式引擎我们要匹配一个数字。严格来说，由于美国邮政编码以五个数字开头，我们需要告诉验证器我们要匹配五个数字，我们通过告诉引擎我们需要多少个来做到这一点：`[0-9]{5}`。如果我们只想匹配主要区号，比如 10023，我们几乎可以在这里结束我们的表达式。然而，邮政编码还有一个可选的四位数字部分，它与主要部分由一个连字符分隔。因此，我们必须告诉正则表达式引擎我们有一个可选的部分要应用。

我们知道邮政编码可选部分的格式是一个连字符和四位数字。这意味着正则表达式的下一部分必须将测试视为一个测试。这意味着我们不能测试连字符，然后分别测试数字；我们要么有-1234 格式，要么什么都没有。这告诉我们我们想要将要测试的项目分组。在正则表达式中将事物分组的方法是将表达式放在括号内。因此，如果我们应用之前的逻辑，我们可能会认为验证的这部分是 `(-[0-9]{4})`。首次尝试，这与我们想要的非常接近。这里的规则是将其视为一个组，其中第一个字符必须是连字符，然后必须有四个数字。这个表达式的一部分有两件事情需要解决。第一件事是目前这个测试是不可选的。换句话说，输入 10012-1234 是有效的，而 10012 不再有效。第二个问题是我们在表达式中创建了一个捕获组，而我们并不需要。 

捕获组是一个编号组，代表匹配的次数。如果我们想在文档的多个地方匹配相同的文本，这可能很有用；然而，由于我们只想要一个匹配，这是可以避免的。

我们现在将解决验证的可选部分的两个问题。我们要做的第一件事是删除捕获组。这是通过使用 `?:` 运算符来完成的，告诉引擎这个组是一个非捕获组。接下来我们要处理的是应用 `?` 运算符，表示我们希望此匹配发生零次或一次。换句话说，我们已经将其设置为可选测试。此时，我们可以成功测试 10012 和 10012-1234，但我们还有一件事需要处理。我们需要确保输入只匹配此输入。换句话说，我们不希望在结尾允许任何杂乱的字符；否则，用户可以输入 10012-12345，引擎会认为我们有一个有效的输入。我们需要做的是在表达式的结尾添加 `$` 运算符，表示表达式在那一点处期望行的结束。此时，我们的正则表达式是 `^[0-9]{5}(?:-[0-9]{4})?$`，它匹配我们期望应用于邮政编码的验证。

我选择明确指定数字表示为 `[0-9]`，因为这对于新接触正则表达式的人来说是一个清晰的指示，表示 0 到 9 之间的数字。有一个等效的速记可以用来表示单个数字，那就是使用 `\d` 代替。有了这个，我们可以将这个规则重写为 `^\d{5}(?:-\d{4})?$`。在这里使用 `\d` 代表一个**美国信息交换标准代码**（**ASCII**）数字。

回到我们的地址验证，实际验证本身非常简单，因为我们花时间编写了为我们做了艰苦工作的验证器。我们所需要做的就是对地址的第一行、城镇和县区应用最小长度验证器，对邮政编码应用正则表达式验证器。每个失败的验证项目都会添加到错误列表中：

```ts
public Validate(state: IPersonState, errors: string[]): void {
  if (!this.minLengthValidator.IsValid(state.Address1)) {
    errors.push("Address line 1 must be greater than 5 characters");
  }
  if (!this.minLengthValidator.IsValid(state.Town)) {
    errors.push("Town must be greater than 5 characters");
  }
  if (!this.minLengthValidator.IsValid(state.County)) {
    errors.push("County must be greater than 5 characters");
  }
  if (!this.zipCodeValidator.IsValid(state.Postcode)) {
    errors.push("The postal/zip code is invalid");
  }
}
```

# 验证姓名

姓名验证是我们将要编写的最简单的验证部分。此验证假定我们的名字至少有一个字母，姓氏至少有两个字母：

```ts
export class PersonValidation implements IValidation {
  private readonly firstNameValidator : MinLengthValidator = new MinLengthValidator(1);
  private readonly lastNameValidator : MinLengthValidator = new MinLengthValidator(2);
  public Validate(state: IPersonState, errors: string[]): void {
    if (!this.firstNameValidator.IsValid(state.FirstName)) {
      errors.push("The first name is a minimum of 1 character");
    }
    if (!this.lastNameValidator.IsValid(state.FirstName)) {
      errors.push("The last name is a minimum of 2 characters");
    }
  }
}
```

# 验证电话号码

电话号码验证将分为两部分。首先，我们验证电话号码是否有输入。然后，我们验证以正确格式输入，使用正则表达式。在分析正则表达式之前，让我们看看这个验证类是什么样子的：

```ts
export class PhoneValidation implements IValidation {

  private readonly regexValidator : RegularExpressionValidator = new RegularExpressionValidator(`^(?:\\((?:[0-9]{3})\\)|(?:[0-9]{3}))[-. ]?(?:[0-9]{3})[-. ]?(?:[0-9]{4})$`);
  private readonly minLengthValidator : MinLengthValidator = new MinLengthValidator(1);

  public Validate(state : IPersonState, errors : string[]) : void {
    if (!this.minLengthValidator.IsValid(state.PhoneNumber)) {
      errors.push("You must enter a phone number")
    } else if (!this.regexValidator.IsValid(state.PhoneNumber)) {
      errors.push("The phone number format is invalid");
    }
  }
}
```

最初，正则表达式看起来比邮政编码验证更复杂；然而，一旦我们将其分解，我们会发现它有很多熟悉的元素。它使用`^`从行的开头捕获，使用`$`捕获到行的末尾，并使用`?:`创建非捕获组。我们还看到我们设置了数字匹配，比如`[0-9]{3}`表示三个数字。如果我们逐段分解，我们会发现这确实是一个简单的验证部分。

我们的电话号码的第一部分要么采用(555)或 555 的格式，后面可能跟着一个连字符、句号或空格。乍一看，`(?:\\((?:[0-9]{3})\\)|(?:[0-9]{3}))[-. ]?`是表达式中最令人生畏的部分。正如我们所知，第一部分要么是(555)这样的东西，要么是 555；这意味着我们要么测试*这个表达式*，要么测试*这个表达式*。我们已经看到`(`和`)`对正则表达式引擎来说意味着特殊的东西，所以我们必须有一些机制可用来表明我们正在看实际的括号，而不是括号代表的表达式。这就是表达式中`\\`的意思。

在正则表达式中使用`\`来转义下一个字符，使其被当作字面量处理，而不是作为一个规则形成表达式来匹配。另外，由于 TypeScript 已经将`\`视为转义字符，我们必须对转义字符进行转义，以便表达式引擎看到正确的值。

当我们想要一个正则表达式表示一个值必须是这样或那样时，我们将表达式分组，然后使用`|`来分隔它。看看我们的表达式，我们首先看到我们首先寻找(*nnn*)部分，如果没有匹配，我们会转而寻找*nnn*部分。

我们还说这个值可以后面跟着一个连字符、句号或空格。我们使用`[-. ]`来匹配列表中的单个字符。为了使这个测试是可选的，我们在末尾加上`?`。

有了这个知识，我们看到正则表达式的下一部分，`(?:[0-9]{3})[-. ]?`，正在寻找三个数字，后面可能跟着一个连字符、句号或空格。最后一部分，`(?:[0-9]{4})`，表示数字必须以四位数字结尾。我们现在知道我们可以匹配像(555) 123-4567，123.456.7890 和(555) 543 9876 这样的数字。

对于我们的目的，像这样的简单邮政编码和电话号码验证非常完美。在大型应用程序中，我们不希望依赖这些验证。这些只是测试看起来是否符合特定格式的数据；它们实际上并不检查它们是否属于真实地址或电话。如果我们的应用程序达到了一个阶段，我们实际上想要验证这些是否存在，我们将不得不连接到执行这些检查的服务。

# 在 React 组件中应用验证

在我们的模拟布局中，我们确定我们希望我们的验证出现在`保存`和`清除`按钮下方。虽然我们可以在主组件内部完成这个操作，但我们将把我们的验证分离到一个单独的验证组件中。该组件将接收我们主组件的当前状态，在状态改变时应用验证，并返回我们是否可以保存我们的数据。

与我们创建`PersonalDetails`组件的方式类似，我们将创建属性传递到我们的组件中：

```ts
interface IValidationProps {
  CurrentState : IPersonState;
  CanSave : (canSave : boolean) => void;
}
```

我们将在`FormValidation.tsx`中创建一个组件，它将应用我们刚刚创建的不同的`IValidation`类。构造函数只是将不同的验证器添加到一个数组中，我们很快将对其进行迭代并应用验证：

```ts
export default class FormValidation extends React.Component<IValidationProps> {
  private failures : string[];
  private validation : IValidation[];

  constructor(props : IValidationProps) {
    super(props);
    this.validation = new Array<IValidation>();
    this.validation.push(new PersonValidation());
    this.validation.push(new AddressValidation());
    this.validation.push(new PhoneValidation());
  }

  private Validate() {
    this.failures = new Array<string>();
    this.validation.forEach(validation => {
      validation.Validate(this.props.CurrentState, this.failures);
    });

    this.props.CanSave(this.failures.length === 0);
  }
}
```

在`Validate`方法中，我们在调用我们的属性的`CanSave`方法之前，对每个验证部分都进行验证。

在我们添加`render`方法之前，我们将重新访问`PersonalDetails`并添加我们的`FormValidation`组件：

```ts
<Row><FormValidation CurrentState={this.state} CanSave={this.userCanSave} /></Row>
```

`userCanSave`方法看起来像这样：

```ts
private userCanSave = (hasErrors : boolean) => {
  this.canSave = hasErrors;
}
```

因此，每当验证更新时，我们的`Validate`方法回调`userCanSave`，这已经作为属性传递进来。

让我们运行验证的最后一件事是从`render`方法中调用`Validate`方法。我们这样做是因为每当父级的状态改变时，渲染周期都会被调用。当我们有一系列验证失败时，我们需要将它们添加到我们的 DOM 中作为我们想要渲染回接口的元素。一个简单的方法是创建所有失败的映射，并提供一个迭代器作为一个函数，它将循环遍历每个失败并将其写回作为一个行到接口：

```ts
public render() {
  this.Validate();
  const errors = this.failures.map(function it(failure) {
    return (<Row key={failure}><Col><label>{failure}</label></Col></Row>);
  });
  return (<Col>{errors}</Col>)
}
```

在这一点上，每当我们在应用程序内部改变状态时，我们的验证将自动触发，并且任何失败都将被写入浏览器作为`label`标签。

# 创建并发送数据到 IndexedDB 数据库

如果我们不能保存细节以便下次回到应用程序时使用，那将会是非常糟糕的体验。幸运的是，较新的 Web 浏览器提供了对一种称为 IndexedDB 的东西的支持，这是一个基于 Web 浏览器的数据库。使用这个作为我们的数据存储意味着当我们重新打开页面时，这些细节将可用。

当我们使用数据库时，我们需要牢记两个不同的领域。我们需要代码来构建数据库表，我们需要代码来保存数据库中的记录。在我们开始编写数据库表之前，我们将添加描述我们的数据库外观的能力，这将用于构建数据库。

接下来，我们将创建一个流畅的接口来添加`ITable`公开的信息：

```ts
export interface ITableBuilder {
  WithDatabase(databaseName : string) : ITableBuilder;
  WithVersion(version : number) : ITableBuilder;
  WithTableName(tableName : string) : ITableBuilder;
  WithPrimaryField(primaryField : string) : ITableBuilder;
  WithIndexName(indexName : string) : ITableBuilder;
}
```

流畅接口的理念是它们允许我们将方法链接在一起，以便更容易地阅读。它们鼓励将方法操作放在一起，使得更容易阅读实例发生了什么，因为操作都是分组在一起的。这个接口是流畅的，因为这些方法返回`ITableBuilder`。这些方法的实现使用`return this;`来允许将操作链接在一起。

使用流畅的接口，不是所有的方法都需要是流畅的。如果你在接口上创建一个非流畅的方法，那就成为了调用链的终点。这有时用于需要设置一些属性然后构建具有这些属性的类的实例的类。

构建表的另一方面是从构建器获取值的能力。由于我们希望保持我们的流畅接口纯粹处理添加细节，我们将编写一个单独的接口来检索这些值并构建我们的 IndexedDB 数据库：

```ts
export interface ITable {
  Database() : string;
  Version() : number;
  TableName() : string;
  IndexName() : string;
  Build(database : IDBDatabase) : void;
}
```

虽然这两个接口有不同的目的，并且将以不同的方式被类使用，但它们都指向相同的基础代码。当我们编写公开这些接口的类时，我们将在同一个类中实现这两个接口。这样做的原因是我们可以根据调用代码看到的接口来分隔它们的行为。我们的表构建类定义如下：

```ts
export class TableBuilder implements ITableBuilder, ITable {
}
```

当然，如果我们现在尝试构建这个，它会失败，因为我们还没有实现我们的任何一个接口。这个类的`ITableBuilder`部分的代码如下：

```ts
private database : StringOrNull;
private tableName : StringOrNull;
private primaryField : StringOrNull;
private indexName : StringOrNull;
private version : number = 1;
public WithDatabase(databaseName : string) : ITableBuilder {
  this.database = databaseName;
  return this;
}
public WithVersion(versionNumber : number) : ITableBuilder {
  this.version = versionNumber;
  return this;
}
public WithTableName(tableName : string) : ITableBuilder {
  this.tableName = tableName;
  return this;
}
public WithPrimaryField(primaryField : string) : ITableBuild
  this.primaryField = primaryField;
  return this;
}
public WithIndexName(indexName : string) : ITableBuilder {
  this.indexName = indexName;
  return this;
}
```

在大多数情况下，这是简单的代码。我们已经定义了一些成员变量来保存细节，每个方法负责填充一个单一的值。代码变得有趣的地方在于`return`语句。通过返回`this`，我们有能力将每个方法链接在一起。在我们添加`ITable`支持之前，让我们通过创建一个类来添加个人详细信息表定义来探索如何使用这个流畅的接口：

```ts
export class PersonalDetailsTableBuilder {
  public Build() : TableBuilder {
    const tableBuilder : TableBuilder = new TableBuilder();
    tableBuilder
      .WithDatabase("packt-advanced-typescript-ch3")
      .WithTableName("People")
      .WithPrimaryField("PersonId")
      .WithIndexName("personId")
      .WithVersion(1);
    return tableBuilder;
  }
}
```

这段代码的作用是创建一个将数据库名称设置为`packt-advanced-typescript-ch3`并向其中添加`People`表的表格构建器，将主字段设置为`PersonId`并在其中创建一个名为`personId`的索引。

现在我们已经看到了流畅接口的运行方式，我们需要通过添加缺失的`ITable`方法来完成`TableBuilder`类：

```ts
public Database() : string {
  return this.database;
}

public Version() : number {
  return this.version;
}

public TableName() : string {
  return this.tableName;
}

public IndexName() : string {
  return this.indexName;
}

public Build(database : IDBDatabase) : void {
  const parameters : IDBObjectStoreParameters = { keyPath : this.primaryField };
  const objectStore = database.createObjectStore(this.tableName, parameters);
  objectStore!.createIndex(this.indexName, this.primaryField);
}
```

`Build`方法是代码中最有趣的部分。这是我们使用底层 IndexedDB 数据库的方法来物理创建表格的地方。`IDBDatabase`是实际 IndexedDB 数据库的连接，我们将在开始编写核心数据库功能时检索到它。我们使用它来创建我们将用来存储人员记录的对象存储。设置`keyPath`允许我们给对象存储一个我们想要搜索的字段，因此它将匹配字段的名称。当我们添加索引时，我们可以告诉对象存储我们想要能够搜索的字段。

# 向我们的状态添加活动记录支持

在查看我们的实际数据库代码之前，我们需要介绍最后一部分拼图——我们将要存储的对象。虽然我们一直在处理状态，但我们一直在使用`IPersonState`来表示一个人的状态，并且就`PersonalDetails`组件而言，这已经足够了。在处理数据库时，我们希望扩展这个状态。我们将引入一个新的`IsActive`参数，用于确定一个人是否显示在屏幕上。我们不需要更改`IPersonState`的实现来添加这个功能；我们将使用交集类型来处理这个问题。我们首先要做的是添加一个具有这个活动标志的类，然后创建我们的交集类型：

```ts
export interface IRecordState {
  IsActive : boolean;
}

export class RecordState implements IRecordState {
  public IsActive: boolean;
}

export type PersonRecord = RecordState & IPersonState;
```

# 使用数据库

既然我们有了构建表格和保存到表格中的状态表示的能力，我们可以把注意力转向连接数据库并实际操作其中的数据。我们要做的第一件事是将我们的类定义为一个通用类型，可以与我们刚刚实现的`RecordState`类扩展的任何类型一起工作：

```ts
export class Database<T extends RecordState> {

}
```

我们需要在这个类中指定我们接受的类型的原因是，其中大多数方法要么接受该类型的实例作为参数，要么返回该类型的实例供调用代码使用。

随着 IndexedDB 成为标准的客户端数据库，它已经成为可以直接从 window 对象访问的内容。TypeScript 提供了强大的接口来支持数据库，因此它被公开为`IDBFactory`类型。这对我们很重要，因为它使我们能够访问打开数据库等操作。实际上，这是我们的代码开始操作数据的起点。

每当我们想要打开数据库时，我们都会给它一个名称和版本。如果数据库名称不存在，或者我们试图打开一个更新版本，那么我们的应用程序代码需要升级数据库。这就是`TableBuilder`代码发挥作用的地方。由于我们已经指定`TableBuilder`实现了`ITable`接口以提供读取值和构建底层数据库表的能力，我们将使用它（表实例将在不久后传递到构造函数中）。

最初，使用 IndexedDB 可能会有些奇怪，因为它强调了大量使用事件处理程序。例如，当我们尝试打开数据库时，如果代码决定需要升级，它会触发`upgradeneeded`事件，我们使用`onupgradeneeded`来处理。这种事件的使用允许我们的代码异步地执行，因为执行会继续而不必等待操作完成。然后，当事件处理程序被触发时，它接管处理。当我们向这个类添加数据方法时，我们将会看到很多这样的情况。

有了这些信息，我们可以编写我们的`OpenDatabase`方法来使用`Version`方法的值打开数据库。第一次我们执行这段代码时，我们需要写入数据库表。即使这是一个新表，它也被视为升级，因此会触发`upgradeneeded`事件。再次，我们可以看到在`PersonalDetailsTableBuilder`类中具有构建数据库的能力的好处，因为我们的数据库代码不需要知道如何构建表。通过这样做，如果需要，我们可以重用这个类来将其他类型写入数据库。当数据库打开时，将触发`onsuccess`处理程序，我们将设置一个实例级别的`database`成员，以便以后使用：

```ts
private OpenDatabase(): void {
    const open = this.indexDb.open(this.table.Database(), this.table.Version());
    open.onupgradeneeded = (e: any) => {
        this.UpgradeDatabase(e.target.result);
    }
    open.onsuccess = (e: any) => {
        this.database = e.target.result;
    }
}

private UpgradeDatabase(database: IDBDatabase) {
    this.database = database;
    this.table.Build(this.database);
}
```

现在我们有了构建和打开表的能力，我们将编写一个接受`ITable`实例的构造函数，我们将用它来构建表：

```ts
private readonly indexDb: IDBFactory;
private database: IDBDatabase | null = null;
private readonly table: ITable;

constructor(table: ITable) {
    this.indexDb = window.indexedDB;
    this.table = table;
    this.OpenDatabase();
}
```

在开始编写处理数据的代码之前，我们还需要为这个类编写最后一个辅助方法。为了将数据写入数据库，我们必须创建一个事务并从中检索对象存储的实例。实际上，对象存储代表数据库中的一个表。基本上，如果我们想要读取或写入数据，我们需要一个对象存储。由于这是如此常见，我们创建了一个`GetObjectStore`方法来返回对象存储。为了方便起见，我们将允许我们的事务将每个操作都视为读取或写入，这是我们在调用事务时指定的：

```ts
private GetObjectStore(): IDBObjectStore | null {
    try {
        const transaction: IDBTransaction = this.database!.transaction(this.table.TableName(), "readwrite");
        const dbStore: IDBObjectStore = transaction.objectStore(this.table.TableName());
        return dbStore;
    } catch (Error) {
        return null;
    }
}
```

当我们阅读代码时，您会看到我选择将方法命名为`Create`、`Read`、`Update`和`Delete`。通常将前两个方法命名为`Load`和`Save`是相当常见的；然而，我故意选择了这些方法名，因为在与数据库中的数据工作时，我们经常使用*CRUD 操作*这个术语，其中**CRUD**指的是**Create**、**Read**、**Update**和**Delete**。通过采用这种命名约定，我希望这能够巩固这种联系。

我们要添加的第一个（也是最简单的）方法将允许我们将记录保存到数据库中。`Create`方法接受一个单独的记录，获取对象存储，并将记录添加到数据库中：

```ts
public Create(state: T): void {
    const dbStore = this.GetObjectStore();
    dbStore!.add(state);
}
```

当我最初编写本章的代码时，我编写了`Read`和`Write`方法来使用回调方法。回调方法背后的想法很简单，就是接受一个函数，我们的方法可以在`success`事件处理程序触发时*回调*到它。当我们看很多 IndexedDB 示例时，我们可以看到它们倾向于采用这种类型的约定。在我们看最终版本之前，让我们看一下`Read`方法最初的样子：

```ts
public Read(callback: (value: T[]) => void) {
    const dbStore = this.GetObjectStore();
        const items : T[] = new Array<T>();
        const request: IDBRequest = dbStore!.openCursor();
        request.onsuccess = (e: any) => {
            const cursor: IDBCursorWithValue = e.target.result;
            if (cursor) {
                const result: T = cursor.value;
                if (result.IsActive) {
                    items.push(result);
                }
                cursor.continue();
            } else {
                // When cursor is null, that is the point that we want to 
                // return back to our calling code. 
                callback(items);
            }
    }
}
```

该方法通过获取对象存储并使用它来打开一个称为游标的东西来打开。游标为我们提供了读取记录并移动到下一个记录的能力；因此，当游标被打开时，成功事件被触发，这意味着我们进入了`onsuccess`事件处理程序。由于这是异步发生的，`Read`方法完成，因此我们将依赖回调将实际值传回调用它的类。看起来相当奇怪的`callback: (value: T[]) => void`是我们将用来将`T`项数组返回给调用代码的实际回调。

在`success`事件处理程序内部，我们从事件中获取结果，这将是一个光标。假设光标不为空，我们从光标中获取结果，并且如果我们的记录状态是活动的，我们将记录添加到我们的数组中；这就是为什么我们对我们的类应用了通用约束——这样我们就可以访问`IsActive`属性。然后我们在光标上调用`continue`，它会移动到下一条记录。调用`continue`方法会再次触发`success`，这意味着我们重新进入`onsuccess`处理程序，导致下一条记录发生相同的代码。当没有更多记录时，光标将为空，因此代码将使用项目数组回调到调用代码。

我提到这是这段代码的初始实现。虽然回调很有用，但它们并没有真正充分利用 TypeScript 给我们带来的力量。这意味着我们将在返回给调用代码之前将所有记录聚集在一起。这意味着我们的`success`处理程序内部的逻辑将有一些细微的结构差异：

```ts
public Read() : Promise<T[]> {
    return new Promise((response) => {
        const dbStore = this.GetObjectStore();
        const items : T[] = new Array<T>();
        const request: IDBRequest = dbStore!.openCursor();
        request.onsuccess = (e: any) => {
            const cursor: IDBCursorWithValue = e.target.result;
            if (cursor) {
                const result: T = cursor.value;
                if (result.IsActive) {
                    items.push(result);
                }
                cursor.continue();
            } else {
                // When cursor is null, that is the point that we want to 
                // return back to our calling code. 
                response(items);
            }
        }
    });
}
```

由于这是返回一个承诺，我们从方法签名中删除回调，并返回一个`T`数组的承诺。我们必须注意的一件事是，我们将用于存储结果的数组的范围必须在`success`事件处理程序之外；否则，每次我们命中`onsuccess`时都会重新分配它。这段代码有趣的地方在于它与回调版本有多么相似。我们所做的只是改变返回类型，同时从方法签名中删除回调。我们承诺的响应部分充当回调的位置。

一般来说，如果我们的代码接受回调，我们可以通过返回一个将回调从方法签名中移动到承诺本身的承诺来将其转换为承诺。

我们的光标逻辑与我们依赖光标检查的逻辑相同，以查看我们是否有一个值，如果有，我们就将其推送到我们的数组上。当没有更多记录时，我们调用承诺上的响应，以便调用代码可以在承诺的`then`部分中处理它。为了说明这一点，让我们来看看`PersonalDetails`中的`loadPeople`代码：

```ts
private loadPeople = () => {
  this.people = new Array<PersonRecord>();
  this.dataLayer.Read().then(people => {
    this.people = people;
    this.setState(this.state);
  });
}
```

`Read`方法是我们的 CRUD 操作中最复杂的部分。我们接下来要编写的方法是`Update`方法。当记录已更新时，我们希望重新加载列表中的记录，以便屏幕上的名字更改得到更新。更新我们的记录的对象存储操作是`put`。如果成功完成，它会触发成功事件，这会导致我们的代码调用承诺上的`resolve`属性。由于我们返回的是`Promise<void>`类型，因此在调用时可以使用`async`/`await`语法：

```ts
public Update(state: T) : Promise<void> {
    return new Promise((resolve) =>
    {
        const dbStore = this.GetObjectStore();
        const innerRequest : IDBRequest = dbStore!.put(state);
        innerRequest.onsuccess = () => {
          resolve();
        } 
    });
}
```

我们的最终数据库方法是`Delete`方法。`Delete`方法的语法与`Update`方法非常相似——唯一的真正区别是它只接受索引，告诉它在数据库中要“删除”哪一行：

```ts
public Delete(idx: number | string) : Promise<void> {
    return new Promise((resolve) =>
    {
        const dbStore = this.GetObjectStore();
        const innerRequest : IDBRequest = dbStore!.delete(idx.toString());
        innerRequest.onsuccess = () => {
          resolve();
        } 
    });
}
```

# 从 PersonalDetails 访问数据库

我们现在可以为我们的`PersonalDetails`类添加数据库支持。我们要做的第一件事是更新成员变量和构造函数，引入数据库支持并存储我们想要显示的人员列表：

1.  首先，我们添加成员：

```ts
private readonly dataLayer: Database<PersonRecord>;
private people: IPersonState[];
```

1.  接下来，我们更新构造函数，连接到数据库并使用`PersonalDetailsTableBuilder`创建`TableBuilder`：

```ts
const tableBuilder : PersonalDetailsTableBuilder = new PersonalDetailsTableBuilder();
this.dataLayer = new Database(tableBuilder.Build());
```

1.  我们还需要做的一件事是在我们的`render`方法中添加显示人员的能力。类似于使用`map`显示验证失败的方式，我们将`map`应用于`people`数组：

```ts
let people = null;
if (this.people) {
  const copyThis = this;
  people = this.people.map(function it(p) {
  return (<Row key={p.PersonId}><Col lg="6"><label >{p.FirstName} {p.LastName}</label></Col>
  <Col lg="3">
    <Button value={p.PersonId} color="link" onClick={copyThis.setActive}>Edit</Button>
  </Col>
  <Col lg="3">
    <Button value={p.PersonId} color="link" onClick={copyThis.delete}>Delete</Button>
  </Col></Row>)
  }, this);
}
```

1.  然后用以下方式呈现出来：

```ts
<Col>
  <Col>
  <Row>
    <Col>{people}</Col>
  </Row>
  <Row>
    <Col lg="6"><Button size="lg" color="success" onClick={this.loadPeople}>Load</Button></Col>
    <Col lg="6"><Button size="lg" color="info" onClick={this.clear}>New Person</Button></Col>
  </Row>
  </Col>
</Col>
```

“Load”按钮是在这个类中从`loadPeople`方法调用的许多地方之一。当我们更新然后删除记录时，我们将看到它的使用。

在处理数据库代码时，通常会遇到情况，其中删除记录不应从数据库中物理删除。我们可能不希望物理删除它，因为另一条记录指向该记录，因此删除它将破坏其他记录。或者，我们可能需要出于审计目的保留它。在这些情况下，通常会执行一种称为软删除的操作（硬删除是从数据库中删除记录的操作）。使用软删除，记录上会有一个指示记录是否活动的标志。虽然`IPersonState`没有提供此标志，但`PersonRecord`类型有，因为它是`IPersonState`和`RecordState`的交集。我们的`delete`方法将把`IsActive`更改为`false`并使用该值更新数据库。加载人员的代码已经理解，它正在检索`IsActive`为`true`的记录，因此这些已删除的记录将在重新加载列表时消失。这意味着，虽然我们在数据库代码中编写了一个删除方法，但我们实际上不会使用它。它作为一个方便的参考，您可能希望更改代码以执行硬删除，但这对我们的目的并不是必要的。

删除按钮将触发删除操作。由于此列表中可能有多个项目，并且我们不能假设用户在删除之前会选择一个人，因此我们需要在尝试删除之前从人员列表中找到该人。回顾渲染人员的代码，我们可以看到人员的 ID 被传递到事件处理程序。在编写事件处理程序之前，我们将编写一个异步从数据库中删除人员的方法。在此方法中，我们要做的第一件事是使用`find`数组方法找到该人：

```ts
private async DeletePerson(person : string) {
  const foundPerson = this.people.find((element : IPersonState) => {
    return element.PersonId === person;
  });
  if (!foundPerson) {
    return;
  }
}
```

假设我们从数组中找到了这个人，我们需要将这个人置于一个状态，以便我们可以将`IsActive`设置为`false`。我们首先创建一个`RecordState`的新实例，如下所示：

```ts
  const personState : IRecordState = new RecordState();
  personState.IsActive = false;
```

我们有一个交集类型，`PersonRecord`，由人和记录状态的交集组成。我们将展开`foundPerson`和`personState`以获得我们的`PersonRecord`类型。有了这个，我们将调用我们的`Update`数据库方法。当更新完成后，我们想要重新加载人员列表并清除编辑器中当前的项目——以防它是我们刚刚删除的项目；我们不希望用户能够简单地再次保存并将`IsActive`设置为`true`来恢复记录。我们将利用我们可以在写成`promise`的代码上使用`await`来等待记录更新完成后再继续处理：

```ts
  const state : PersonRecord = {...foundPerson, ...personState};
  await this.dataLayer.Update(state);
  this.loadPeople();
  this.clear();
```

`clear`方法只是将状态更改回我们的默认状态。这是我们将其传递到此组件的整个原因，这样我们就可以轻松地将值清除回其默认状态：

```ts
private clear = () => {
  this.setState(this.defaultState);
}
```

使用我们的`delete`事件处理程序，完整的代码如下：

```ts
private delete = (event : any) => {
  const person : string = event.target.value;
  this.DeletePerson(person);
}

private async DeletePerson(person : string) {
  const foundPerson = this.people.find((element : IPersonState) => {
    return element.PersonId === person;
  });
  if (!foundPerson) {
    return;
  }
  const personState : IRecordState = new RecordState();
  personState.IsActive = false;
  const state : PersonRecord = {...foundPerson, ...personState};
  await this.dataLayer.Update(state);
  this.loadPeople();
  this.clear();
}
```

我们需要连接的最后一个数据库操作是从保存按钮触发的。保存的操作取决于我们之前是否保存了记录，这可以通过`PersonId`是否为空来确定。在尝试保存记录之前，我们必须确定它是否可以保存。这取决于检查验证是否允许我们保存。如果存在未解决的验证失败，我们将通知用户他们无法保存记录：

```ts
private savePerson = () => {
  if (!this.canSave) {
    alert(`Cannot save this record with missing or incorrect items`);
    return;
  }
}
```

类似于我们使用删除技术的方式，我们将通过将状态与`RecordState`结合来创建我们的`PersonRecord`类型。这次，我们将`IsActive`设置为`true`，以便它被视为活动记录。

```ts
const personState : IRecordState = new RecordState();
personState.IsActive = true;
const state : PersonRecord = {...this.state, ...personState};
```

当我们插入记录时，我们需要为`PersonId`分配一个唯一值。为简单起见，我们将使用当前日期和时间。当我们将人员添加到数据库时，我们重新加载人员列表，并从编辑器中清除当前记录，以便用户不能通过再次点击“保存”来插入重复记录：

```ts
  if (state.PersonId === "") {
    state.PersonId = Date.now().toString();
    this.dataLayer.Create(state);
    this.loadPeople();
    this.clear();
  }
```

更新人员的代码利用了 promise 的特性，以便在保存完成后立即更新人员列表。在这种情况下，我们不需要清除当前记录，因为如果用户再次点击“保存”，我们不可能创建一个新记录，而只是更新当前记录：

```ts
  else {
    this.dataLayer.Update(state).then(rsn => this.loadPeople());
  }
```

保存的完成方法如下：

```ts
private savePerson = () => {
  if (!this.canSave) {
    alert(`Cannot save this record with missing or incorrect items`);
    return;
  }
  if (state.PersonId === "") {
    state.PersonId = Date.now().toString();
    this.dataLayer.Create(state);
    this.loadPeople();
    this.clear();
  }
  else {
    this.dataLayer.Update(state).then(rsn => this.loadPeople());
  }
}
```

我们还需要涵盖一个最后的方法。您可能已经注意到，当我们点击“编辑”按钮时，我们没有办法选择并在文本框中显示用户。逻辑推断，按下按钮应该触发一个事件，将`PersonId`传递给事件处理程序，我们可以使用它从列表中找到相关的人；当使用删除按钮时，我们已经看到了这种行为类型，因此我们对代码的选择部分有了一个很好的想法。一旦我们有了这个人，我们调用`setState`来更新状态，这将通过绑定的力量更新显示：

```ts
private setActive = (event : any) => {
  const person : string = event.target.value;
  const state = this.people.find((element : IPersonState) => {
    return element.PersonId === person;
  });
  if (state) {
    this.setState(state);
  }
}
```

现在我们已经拥有了构建 React 联系人管理器所需的所有代码。我们满足了本章开头设定的要求，并且我们的显示看起来与我们的模拟布局非常接近。

# 增强

`Create`方法存在一个潜在问题，即它假设立即成功。它没有处理操作的`success`事件。此外，还有一个进一步的问题，即`add`操作具有`complete`事件，因为`success`事件可能在记录成功写入磁盘之前触发，如果事务失败，则不会引发`complete`事件。您可以将`Create`方法转换为使用 promise，并在引发`success`事件时恢复处理。然后，更新组件的插入部分，以便在完成后重新加载。

删除会重置状态，即使用户没有编辑被删除的记录。因此，增强删除代码，只有在被编辑的记录与被删除的记录相同时才重置状态。

# 总结

本章向我们介绍了流行的 React 框架，并讨论了如何使用 TypeScript 来构建现代客户端应用程序以添加联系信息。我们首先定义了需求，并在创建基本实现之前，创建了我们应用程序的模拟布局，使用`create-react-app`和`react-scripts-ts`脚本版本。为了以 React 友好的方式利用 Bootstrap 4，我们添加了`reactstrap`包。

在讨论了 React 如何使用特殊的 JSX 和 TSX 格式来控制渲染方式之后，我们开始定制`App`组件，并添加了自定义的 TSX 组件。通过这些组件，我们学习了如何传递属性和设置状态，然后使用它们创建双向绑定。通过这些绑定，我们讨论了如何通过创建可重用的验证器来验证用户输入，然后将其应用于验证类。作为验证的一部分，我们添加了两个正则表达式，并对其进行了分析以了解其构造方式。

最后，我们研究了如何将个人信息保存在 IndexedDB 数据库中。这一部分首先是了解如何使用表构建器构建数据库和表，然后是如何操作数据库。我们学习了如何将基于回调的方法转换为使用 promises API 以提供异步支持，以及软删除和硬删除数据之间的区别。

在下一章中，我们将继续使用 Angular 与 MongoDB、Express 和 Node.js，它们合称为 MEAN 堆栈，来构建一个照片库应用程序。

# 问题

1.  是什么赋予了 React 在`render`方法中混合视觉元素和代码的能力？

1.  为什么 React 使用`className`和`htmlFor`？

1.  我们看到电话号码可以使用正则表达式`^(?:\\((?:[0-9]{3})\\)|(?:[0-9]{3}))[-. ]?(?:[0-9]{3})[-. ]?(?:[0-9]{4})$`进行验证。我们还讨论了表示单个数字的另一种方式。我们如何将这个表达式转换为使用另一种表示方式得到完全相同的结果？

1.  为什么我们要将验证器与验证代码分开创建？

1.  软删除和硬删除之间有什么区别？

# 进一步阅读

+   React 是一个大的话题。为了更多地了解其中的思想，我推荐*React and React Native* –* Second Edition *（[`www.packtpub.com/application-development/react-and-react-native-second-edition`](https://www.packtpub.com/application-development/react-and-react-native-second-edition)）。

+   有关在 React 中使用 TypeScript 的更多信息，我推荐 Carl Rippon 的*Learn React with TypeScript 3*（[`www.packtpub.com/web-development/learn-react-typescript-3`](https://www.packtpub.com/web-development/learn-react-typescript-3)）。

+   Packt 还出版了 Loiane Groner 和 Gabriel Manricks 的优秀书籍*JavaScript Regular Expressions*（[`www.packtpub.com/web-development/javascript-regular-expressions`](https://www.packtpub.com/web-development/javascript-regular-expressions)），如果你想提升你的正则表达式知识。
