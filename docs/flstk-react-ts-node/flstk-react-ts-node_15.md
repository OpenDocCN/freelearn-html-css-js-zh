# 第十二章：为我们的在线论坛应用构建 React 客户端

我们已经走了很长的路。在本章中，我们将开始编写我们的应用程序，从 React 客户端开始。我们将利用前几章学到的一切，使用新的 Hooks API 构建我们的 React 应用。我们还将使用响应式技术构建一个移动客户端，该客户端将适应其视图以处理移动设备和桌面设备。

# 技术要求

现在，你应该对使用 React、Node、Express 和 GraphQL 进行 Web 开发有很好的理解。你也应该熟悉 CSS。我们将再次使用 Node 和 Visual Studio Code 来编写我们的代码。

本书的 GitHub 存储库可以在[`github.com/PacktPublishing/Full-Stack-React-TypeScript-and-Node`](https://github.com/PacktPublishing/Full-Stack-React-TypeScript-and-Node)找到。使用`Chap12`文件夹中的代码。

要设置*第十二章*代码文件夹，转到你的`HandsOnTypescript`文件夹并创建一个名为`Chap12`的新文件夹。

# 创建我们的 React 应用的初始版本

在本节中，我们将构建我们的 React 客户端。我们将无法完全完成客户端，因为它将需要我们的后端功能，比如我们的 GraphQL API、认证能力、发布主题等等。但是，我们将开始创建我们的主要屏幕并设置 Redux 和 React Router。

本节将包含大量的代码。请经常休息并控制自己的节奏。代码将在我们的构建过程中不断演变、迭代和重构多次。有时，这将是为了更好的代码重用。有时，这将是为了改进我们的设计和可读性。因此，如果你遇到困难，请参考源代码。这将是本书迄今为止最具挑战性的部分。

注意

我们不会展示每一行代码，因为那将是多余的。请下载并在你的编辑器中打开源代码以便跟随。

在本节中，我们将涵盖以下主题：

+   React 项目设置和依赖配置

+   样式和布局

+   核心组件和功能创建

提示

从一开始就让一切编译和工作实际上对你的学习没有任何好处。不要专注于让东西简单地编译和运行。相反，尝试实验和进行更改。换句话说，打破代码，使其无法编译，然后修复它。这是确保你理解自己在做什么的唯一方法。

让我们从使用`create-react-app`创建我们的基本项目开始。然后，我们将添加 Redux 和 React Router：

1.  转到终端中的`Chap12`文件夹并运行以下命令：

```ts
create-react-app super-forum-client --template typescript
```

1.  接下来，进入新的`super-forum-client`文件夹并运行`start`命令以确保它正常工作：

```ts
npm start
```

1.  现在，让我们安装 Redux 和 React Router：

```ts
package-lock.json file and the node_modules folder. Then, do a clean install using npm install.
```

所以，现在，我们已经安装了我们的核心包。在我们开始编码之前，我们需要讨论如何布置我们的应用程序。在我们的情况下，我们希望我们的应用程序能在移动设备和桌面上运行。这样，我们就可以拥有一个单一的应用程序，可以在手机、桌面和笔记本电脑上运行。

有多种方法可以实现这个目标。我们可以使用诸如**Bootstrap**这样的库或**Ionic**这样的 UI 框架来帮助我们构建 UI 和布局。这些框架非常好用，但它们也隐藏了一些关于 Web 上布局和样式工作的细节。使用框架时，你也可能失去一些控制，最终得到的网站看起来与使用相同框架的其他网站类似。

## CSS Grid

对于我们的应用程序，我们将使用响应式网页设计。响应式网页设计只是我们的网页应用程序适应不同的设备和屏幕尺寸的意图。在使用 Web 技术时，有许多方法可以做到这一点。其中之一就是 CSS Grid。通过这个系统，我们可以构建我们的应用程序屏幕，充分利用桌面空间，同时自动重新配置为移动设备。因此，我们将使用 CSS Grid 以及其他 Web 技术来创建我们的布局。

CSS Grid 给我们提供了大部分像 Bootstrap 这样的程序所能实现的功能。但是，CSS Grid 是 CSS 网络标准的一部分，而不是第三方库的一部分。因此，我们知道我们的布局将始终与网络一起工作，永远不会突然不受支持。

那么，什么是 CSS Grid？CSS Grid 是内置标准 CSS 的布局方法，允许我们使用行和列创建灵活的布局。它被创建来取代表格布局的使用。CSS Grid 非常强大，有许多做同样事情的方法。为了保持简单，我将向您展示一种特定的方法来做到这一点，尽管如果您认为这将有用，您可以稍后探索更多选项。让我们开始使用 CSS Grid：

1.  首先，回到我们的项目，打开`App.tsx`，并删除`App`对象的内容。做以下事情：

```ts
import React from "react";
import "./App.css";
function App() {
  return (
    <div className="App">
      <nav className="navigation">Nav</nav>
      <div className="sidebar">Sidebar</div>
      <div className="leftmenu">Left Menu</div>
      <main className="content">Main</main>
      <div className="rightmenu">Right Menu</div>
    </div>
  );
}
export default App;
```

正如您所看到的，我们已经摆脱了大部分内容，并用布局占位符替换了它。当然，我们最终会将这些元素制作成组件，但现在，我们将专注于使我们的`Grid`布局工作。

1.  现在，让我们替换`App.css`文件的内容，就像这样：

```ts
:root {
  --min-screen-height: 1000px;
}
```

首先，有一个`:root`伪类，我们将使用它作为我们应用程序主题的 CSS 变量的容器。为了使样式和主题更一致和更容易，我们将使用变量而不是硬编码的值。随着我们构建应用程序，您将看到这里添加了越来越多的变量：

```ts
.App {
margin: 0 auto;
```

以下的边距设置使我们的布局居中：

```ts
  max-width: 1200px;
  display: grid;
  grid-template-columns: 0.7fr 0.9fr 1.5fr 0.9fr;
  grid-template-rows: 2.75rem 3fr;
  grid-template-areas:
    "nav nav nav nav" 
    "sidebar leftmenu content rightmenu";
  gap: 0.75rem 0.4rem;
}
```

以下是与 Grid 相关的属性的概述：

+   `display`：在这里，我们声明我们的元素将是`grid`类型。

+   `grid-template-columns`：这个属性告诉我们的应用程序列的宽度是相对的。在我们的设置中，它表示我们有四列。`fr`值表示应该给列的可用宽度的一部分。因此，例如，在我们的情况下，我们有四列，所以如果每列的可用宽度完全相等，每列的值将是`1fr`。但在我们的情况下，每列将使用不同数量的宽度，比均匀分配的宽度小或大，这就是为什么我们有不同的值。可能的值可以是具体的，比如`100px`或`2rem`，百分比，比如例子`20%`，或者是隐式的，比如`.25fr`。

+   `grid-template-rows`：指示行的数量和大小。可能的值与列相同。

+   `grid-template-areas`：每个 Grid 都可以有称为区域的标记部分。正如这个例子所示，您只需在网格形式的列和行中为每个区域添加标签。因此，在我们的情况下，`"nav nav nav nav"`代表我们的两行中的第一行，有四列，而`"sidebar leftmenu content rightmenu"`代表我们的第二行及其每一列。

+   `gap`：这是在列和行之间添加填充的一种方法。第一个条目表示行，第二个表示列。

1.  现在我们已经解释了 CSS Grid 的基本特性，让我们来看看与 Grid 相关部分的样式。剩下的样式是用于 Grid 内容区域的：

```ts
.navigation {
  grid-area: nav;
}
.sidebar {
  min-height: var(--min-screen-height);
  grid-area: sidebar;
  background-color: aliceblue;
}
.leftmenu {
  grid-area: leftmenu;
  background-color: skyblue;
}
.content {
  min-height: var(--min-screen-height);
  grid-area: content;
  background-color: blanchedalmond;
}
.rightmenu {
  grid-area: rightmenu;
  background-color: coral;
}
```

如您所见，它们有一个`grid-area`属性，指示元素属于网格的哪个区域。`nav`区域将用于导航。`sidebar`将显示用户特定设置的菜单，并且仅在桌面和笔记本电脑上显示；它将在移动设备上隐藏。`leftmenu`将用于存储我们的主题类别列表。`content`将容纳我们按类别筛选的主题列表。最后，`rightmenu`将显示一些热门或相关的主题列表。

注意

我暂时使用这些尴尬的`background-color`设置，只是为了清楚地区分每个区域。最终，我们会将它们删除。

现在，我们的应用程序在桌面和笔记本设备上都有一个基本布局。但是，我们如何使其自动重新配置以适应较小的屏幕，比如手机和平板电脑？有一种名为**媒体查询**的 CSS 技术可以在这种情况下提供帮助。然而，对于我们的需求来说，它单独是不够的。

我们正在动态构建我们的应用程序，使用由状态更改驱动的 React。这意味着在较小的设备上，某些屏幕组件不应该被绘制，如果它们不需要或者不能在较小的设备上显示。因此，尽管我们可以使用媒体查询在检测到较小的屏幕时隐藏元素，但让 React 渲染永远不会被用户看到或直接使用的东西将是资源的低效使用。

相反，让我们看看我们可以在代码中如何使用事件处理和 React Hooks 来解决这个问题：

1.  首先，我们要做的是将我们的元素转换为 React 组件。让我们在`src`文件夹内创建一个名为`components`的新文件夹。

1.  然后，在该文件夹内，为我们`App`组件的根`div`中的每个元素创建一个容器组件。现在，您的`src`文件夹和`App.tsx`文件应该是这样的：

![图 12.1 – 重构后的 App.tsx 文件](img/Figure_12.01_B15508.jpg)

图 12.1 – 重构后的 App.tsx 文件

由于篇幅限制，我不会审查我们需要在这里创建的每个文件，因为这是高度重复的代码，但这是更新后的`Main`组件的一个示例（当然，源代码将包含所有组件的完整应用程序代码）：

```ts
import React from "react";
const Main = () => {
  return <main className="content">Main</main>;
};
export default Main;
```

如您所见，我们只是将我们的代码从`App.tsx`移动到组件的`Main.tsx`文件中。这意味着您需要创建剩下的组件；也就是说，`Nav`、`SideBar`、`LeftMenu`和`RightMenu`。这是 React Developer Tools 屏幕的截图，显示了我们目前的组件层次结构。React Developer Tools 在*第六章，使用 create-react-app 设置我们的项目并使用 Jest 进行测试*中进行了讨论：

![图 12.2 – 组件层次结构视图](img/Figure_12.02_B15508.jpg)

图 12.2 – 组件层次结构视图

请注意，这里有**Nav**、**SideBar**、**LeftMenu**、**Main**和**RightMenu**组件。每个组件代表我们网站根目录上的应用程序区域。请注意，随着我们构建应用程序，我们将有更多的屏幕。

无论如何，我们必须进行这种组件化，因为我们正在构建一个 React 应用程序。但是这如何帮助我们实现我们的愿望，使我们的 Web 应用程序响应，以便自动配置到不同的设备屏幕？通过将网格的每个区域分离为自己的组件，我们可以允许每个组件使用一个 React Hook 来查找屏幕尺寸信息。因此，如果组件不适合某个屏幕尺寸，它将不会渲染或以不同的方式渲染。

为了使这个响应式系统工作，我们需要两个主要功能。首先，我们需要一些额外的 CSS 样式，使用媒体查询在检测到较小设备时以不同的方式布局我们的网格。此外，我们需要让我们的组件在使用特定屏幕尺寸时变得意识到，并且要么不渲染组件，要么以不同的方式渲染它。让我们看看代码是什么样子的。

首先，让我们为移动设备创建媒体查询。打开您的`App.css`文件，并在文件底部添加以下媒体查询：

```ts
@media screen and (orientation: portrait) and (max-width: 768px) {
  .App {
    grid-template-columns: 1fr;
    grid-template-areas:
      "nav"
      "content";
  }
}
```

在这里，当设备的`orientation`处于`portrait`模式且分辨率为`768px`或更低时，我们覆盖了原始的`App`类定义。如果您在 iPhone X 上使用 Chrome 开发者工具以移动模式运行应用程序，您应该会看到这个：

![图 12.3 - Chrome 开发者工具中我们应用程序的移动模式视图](img/Figure_12.03_B15508.jpg)

图 12.3 - Chrome 开发者工具中我们应用程序的移动模式视图

应用程序右侧是白色的，因为我们仍在渲染原始桌面模式中存在的元素。我们很快会解决这个问题。现在，让我们创建我们的**Hook**，它有助于处理基于设备尺寸的渲染：

1.  在`src`文件夹内创建一个名为`hooks`的文件夹。然后，添加一个名为`useWindowDimensions.ts`的文件。请注意，它不是一个组件，因为它有一个`ts`扩展名。从本书的 GitHub 存储库中复制源代码，然后我们来看一下。

首先，我们创建一个名为`WindowDimension`的接口，以便我们可以为我们的 Hook 返回的内容进行类型化，这种情况下是浏览器的`window`对象尺寸。

然后，在*第 8 行*，我们命名我们的`useWindowDimensions` Hook。然后，在下一行，我们创建一个名为`dimension`的状态对象，并为`height`和`width`赋值为`0`。

1.  接下来，我们创建我们的处理函数`handleResize`，它将使用状态更新方法`setDimension`来设置我们的尺寸值。我们浏览器的`window`对象提供了尺寸值。

1.  最后，从*第 21 行*开始，我们使用`useEffect` Hook 来处理窗口的`resize`事件。请注意，空数组`[]`表示这将仅在首次加载时运行。还要注意，当我们添加事件处理程序时，我们还必须返回一个事件移除器（这可以防止内存泄漏和冗余事件处理程序被添加）。

1.  现在，我们需要更新我们的`SideBar`、`LeftMenu`和`RightMenu`组件，以便它们将使用我们的`useWindowDimensions` Hook，并且知道在设备宽度小于或等于`768`时不进行渲染（与我们的媒体查询相同）。使用 Hook 的代码在这些组件中是相同的，所以我只会在这里展示`SideBar`组件。请以类似的方式自行更新其他组件：

```ts
import React from "react";
import { useWindowDimensions } from "../hooks/useWindowDimensions";
const SideBar = () => {
  const { width } = useWindowDimensions();
  if (width <= 768) {
    return null;
  }
  return <div className="sidebar">Sidebar</div>;
};
export default SideBar;
```

如您所见，我们使用`useWindowDimensions` Hook 来获取`width`维度。然后我们检查它是否为`768`或更低，如果是，我们返回`null`；否则，我们返回正常的 JSX。其他组件将使用相同的代码来使用`useWindowDimensions` Hook。

如果您运行应用程序，您会看到白色间隙现在已经消失，并且这些组件不会在 HTML 中进行渲染。请注意，为了节省时间，我们只会支持 iPhone X 的桌面和移动纵向模式。支持每种可能的设备配置超出了本书的范围。这是一个关于支持多个设备屏幕的主题的好链接：[`developers.google.com/web/fundamentals/codelabs/your-first-multi-screen-site`](https://developers.google.com/web/fundamentals/codelabs/your-first-multi-screen-site)。

在我们继续之前，让我们完善我们的客户端基本配置，如 Redux 和 React Router。

1.  更新您的`index.tsx`文件，以便包含 Redux 和 React Router。我们在*第七章**，学习 Redux 和 React Router*中涵盖了 Redux 和 React Router。如果遇到困难，源代码始终可用。

1.  现在，让我们在`src`文件夹内创建一个名为`store`的文件夹，并在其中添加我们的 Redux 文件。创建`AppState.ts`和`configureStore.ts`文件，并输入源文件中显示的代码。我们现在还没有准备好`UserProfileReducer`，所以您可以暂时将其排除在外。我们不会使用 Redux 中间件，因为我在*第七章**，学习 Redux 和 React Router*中展示了这一点。

现在，在我们继续并开始创建组件之前，让我们向我们的应用程序添加一个新的 React 功能，这将帮助我们增加更多的亮点。

## 错误边界

错误边界很像是 React 组件的异常处理。在大型应用程序中，通常无法防止所有可能发生的错误。因此，通过在我们的组件中使用错误边界，我们可以“捕获”意外错误，并为用户提供更好的用户体验。发生错误时，我们将显示一个预先创建的错误屏幕，而不是一些看起来可怕的技术错误消息。让我们开始吧：

1.  首先，让我们创建我们的错误边界文件。在`components`文件夹内，创建一个名为`ErrorBoundary.tsx`的文件，并将此书的 GitHub 存储库中的源代码添加到其中。请注意，错误边界仍然使用旧的类样式，因为我们需要`getDerivedStateFromError`和`componentDidCatch`生命周期事件处理程序来捕获错误。React 团队确实计划最终添加 Hooks 等效功能。

在文件顶部，请注意我们还有一个匹配的 CSS 样式文件。这很琐碎，所以我不会在这里展示，但您可以在源代码中找到它。

首先，我们将为我们的错误边界的 props 创建一个类型，称为`ErrorBoundaryProps`。

接下来，我们必须为我们的错误边界的本地状态创建另一种类型，称为`ErrorBoundaryState`。在`ErrorBoundary`类定义的开头，我们将看到一些用于设置状态的构造函数的样板。紧接着，我们将使用`getDerivedStateFromError`函数告诉 React 如果`hasError`为 true，则显示错误 UI。

在*第 31 行*，在我们的`componentDidCatch`函数中，我们的组件意识到发生了某种错误，并将我们的`hasError`状态变量设置为 true。我们还可以在这里运行我们自己的代码来记录错误并在需要时通知支持。

最后，如果`hasError`为 true，我们会呈现我们的消息，以便用户不必看到可能令人困惑的奇怪技术消息。当然，您可以编写自己的自定义消息。

警告

错误边界不会捕获发生在事件处理程序、异步代码或服务器端渲染的 React 中的错误，以及错误边界本身抛出的错误。通常情况下，您必须自己处理这些错误，使用`try catch`。

1.  现在，让我们通过在其中一个组件中抛出错误来测试我们的错误边界。更新`Main.tsx`文件的`Main`函数，如下所示：

```ts
const Main = () => {
  const test = true;
  if (test) throw new Error("Main fail");
  else {
    return <main className="content">Main</main>;
  }
};
```

如您所见，我们故意抛出了一个`Error`。

1.  现在尝试运行应用程序。您应该会看到我们试图避免的屏幕类型。为什么会发生这种情况？这是因为我们目前处于开发模式，React 故意在此模式下显示所有错误。如果我们处于生产模式，通过运行`npm run build`，我们将看到错误边界消息。

然而，即使在开发模式下，我们仍然可以在 Chrome 浏览器右上角点击**x**按钮来查看我们的错误边界屏幕。如果您这样做，您应该会看到以下消息：

![图 12.4 – 错误边界消息](img/Figure_12.04_B15508.jpg)

图 12.4 – 错误边界消息

如您所见，我们现在显示了正常的错误消息。再次，随意根据您的喜好设置此消息的样式。为了节省时间，我们将保持原样。

## 数据服务层

在我们的应用程序中，我们将调用 GraphQL API 或 Web API，或者获取网络调用。但是，这些后端服务都还没有准备好。现在，我们将创建一个文件，其中包含模拟真实后端的假网络调用。一旦我们的真实后端到位，我们将删除此功能：

1.  首先，在`src`内创建一个名为`services`的文件夹，然后在其中创建`DataService.ts`文件。由于这是我们很快会丢弃的代码，我不会在这里展示，但你可以从源文件中获取代码。请注意，此服务中将包含对模型类型的一些引用，因此您需要添加这些引用，并且在本章的进展中我们将对其进行讨论。

1.  现在我们有了获取数据的方法，让我们更新我们的`LeftMenu`组件，以便使用它。但首先，我们需要创建我们的`Category`类型，因为我们使用的是 TypeScript。在`src`内创建一个名为`model`的新文件夹。然后，创建`Category.ts`文件并添加源代码。

1.  现在，更新`LeftMenu.tsx`文件。首先，我们将通过添加名为`Category`的模型类型和`LeftMenu.css`文件来更新导入。我们稍后会在我们的代码中使用它们。

1.  然后，在*line 9*上，创建我们的状态对象`categories`，其中包含我们的类别列表。在我们加载`Category`数据之前，我们需要一些默认文本，`Left Menu`。

1.  然后，在*line 13*上，我们有`useEffect`，在其中我们调用我们的`getCategories`函数并获取我们的`Categories`。然后，我们使用 ES6 的`map`函数将我们的对象转换为 JSX。

1.  最后，在返回的 JSX 中，我们在 UI 中使用`Categories`状态对象。

如果您重新加载浏览器，您将看到由于我们假的`DataService`中的定时器而出现 2 秒的延迟，然后显示类别列表，就像这样：

![图 12.5 - 加载的类别](img/Figure_12.05_B15508.jpg)

图 12.5 - 加载的类别

一旦我们的真实服务器调用准备就绪，我们将删除`DataService`。

## 导航菜单

现在我们有了基本配置和布局，我们可以开始创建我们的侧边栏菜单。我们的侧边栏菜单项的有趣之处在于它们将同时用于侧边栏和作为移动设备的下拉模态框。这样，我们可以通过只有一个组件来减少代码量。

现在，为了创建具有正确链接集的侧边栏，我们需要知道用户是否已登录。如果他们没有登录，我们将显示登录和注册菜单。如果他们已登录，我们将显示注销和用户配置文件菜单。用户配置文件菜单屏幕将显示用户的设置，以及他们发布的帖子列表。由于我们的用户的登录状态将在整个应用程序中共享，让我们将这些数据放入我们的 Redux 存储中：

1.  我们将使用`UserProfile`对象实例的存在或不存在作为用户已登录的指示。首先，让我们向当前空的 reducers 集合中添加一个新的 reducer。在`store`内创建一个名为`user`的新文件夹。现在，创建一个名为`Reducer.ts`的文件并添加所需的源代码。

1.  然后，创建一个名为`UserProfileSetType`的操作类型，以便我们的`UserProfileReducer`可以与其他 reducer 区分开。

1.  接下来，我们必须创建一个名为`UserProfilePayload`的有效负载类型。这是我们的操作在稍后分派时将包含的数据。

1.  然后，我们必须创建`UserProfileAction`接口，它是`action`类型的。这用于区分用户配置文件的操作与其他操作类型。

1.  最后，我们有我们的实际 reducer，`UserProfileReducer`，它根据我们期望的`UserProfileSetType`执行过滤。再次强调，Redux 在*第七章**中已经涵盖了，学习 Redux 和 React Router。*

1.  为了帮助我们样式化我们的组件，我们需要使用图标来提供更好的视觉呈现。让我们安装 Font Awesome，因为它是免费的，并提供了一个吸引人的样式和图标套件，非常受欢迎的网页开发。运行以下命令：

```ts
npm i @fortawesome/fontawesome-svg-core @fortawesome/free-solid-svg-icons @fortawesome/react-fontawesome  
```

1.  现在我们已经添加了我们的图标，让我们在`src/components`文件夹内创建一个名为`sidebar`的新文件夹，并将现有的`SideBar.tsx`文件移动到其中。现在，创建一个名为`SideBarMenus.tsx`的新文件，并将以下代码添加到其中。确保你已经添加了必要的导入：

```ts
const SideBarMenus = () => {
  const user = useSelector((state: AppState) => state.   user);
const dispatch = useDispatch();
```

我们使用`useSelector`和`useDispatch` Hook 来访问 Redux 的功能：

```ts
useEffect(() => { 
    dispatch({
      type: UserProfileSetType,
      payload: {
        id: 1,
        userName: "testUser",
      },
    });
  }, [dispatch]);
```

然后，我们使用`useEffect` Hook 来调用、分发和更新我们的`UserProfile`对象。注意它现在是硬编码的，但是当我们的后端准备好时，我们将使用 GraphQL 调用：

```ts
  return (
    <React.Fragment>
      <ul>
        <FontAwesomeIcon icon={faUser} />
          <span className="menu-name">{user?.userName}           </span>
      </ul>
    </React.Fragment>
  );
};
```

接下来，我们必须为 UserProfile 添加一个`FontAwesome`字体，然后显示当前的`用户名`。这个菜单项最终将是可点击的，这样我们用户的个人资料屏幕就会出现：

```ts
export default SideBarMenus;
```

在构建我们的登录、登出、注册等屏幕时，我们将把这些菜单项添加到这个 JSX 中。

我个人觉得项目符号很分散注意力，所以让我们通过在`index.css`文件中添加以下样式来移除应用程序所有无序列表的项目符号：

```ts
ul {
    list-style-type: none 
}
```

1.  现在，我们需要更新`SideBar.tsx`，使其使用`SideBarMenus.tsx`。像这样更新`SideBar`。首先，添加适当的导入，比如`SideBarMenus`，首先：

```ts
const SideBar = () => {
  const { width } = useWindowDimensions();
  if (width <= 768) {
    return null;
  }
  return (
    <div className="sidebar">
      <SideBarMenus />
    </div>
  );
};
```

现在，我们可以更新 JSX 来包含它。

请注意，我们最终会编写一些代码，以便`UserProfile`图标和`userName`只在用户实际登录时出现。我们还将使其可点击，以便点击它会打开用户的 UserProfile 屏幕。然而，没有我们的后端，我们无法做到这一点。现在，我们将其作为一个占位符。

1.  让我们继续并重用我们的`SideBarMenus`组件来进行移动显示。在`components`文件夹内更新`Nav.tsx`文件。添加适当的导入：

```ts
const Nav = () => {
  const { width } = useWindowDimensions();
  const getMobileMenu = () => {
    if (width <= 768) {
      return (
        <FontAwesomeIcon icon={faBars} size="lg"          className="nav-mobile-menu" />
      );
    }
    return null;
  };
```

同样，我们使用我们的`useWindowDimensions` Hook 来确定我们是否在移动设备上。然而，这一次，我们创建了一个名为`getMobileMenu`的函数来处理决定返回什么 JSX 的逻辑。如果我们不是在移动设备上运行，它不返回任何东西；否则，它返回汉堡菜单的`FontAwesome`图标：

```ts
  return (
    <nav className="navigation">
      {getMobileMenu()}
      <strong>SuperForum</strong>
    </nav>
  );
};
export default Nav;
```

当在移动设备上查看屏幕时，应该是这样的：

![图 12.6 - 移动模式下的导航菜单](img/Figure_12.06_B15508.jpg)

图 12.6 - 移动模式下的导航菜单

1.  在构建我们的应用程序时，我们需要能够显示模态框。因此，在继续之前，我们需要安装`react-modal`。这个包将允许我们创建一些组件模态弹出框。这使得它们在何时显示上更加灵活。像这样安装`react-modal`：

```ts
npm i react-modal
npm i @types/react-modal -D
```

1.  为了使用这个模态框并使其响应并适应不同的设备屏幕，我们需要更新我们的样式。在我们的`App.css`文件中，你会看到一个名为`modal-menu`的类被应用到了所有的模态框上。

这是我们的非移动设备将获得的模态框的默认样式。这里需要注意的主要是模态框从屏幕的 50%位置开始。然后，我们使用`transform`将其拉回一半（自身的 50%）。这样可以使我们的模态框居中，使其位于屏幕的中间。请注意，`z-index`设置得很高，以确保这个模态框始终显示在顶部。

对于移动设备，我们使用`App.css`文件中现有的媒体查询来保存一个更新后的`modal-menu`。基本上，我们正在覆盖桌面样式中的相同属性，使用移动媒体查询的样式。在这种情况下，我们使用`left`、`right`和`top`来将模态框拉伸到可用屏幕的两端。这就是为什么我们的 transform 现在是 0，因为它不再需要。

1.  接下来，我们将为汉堡图标添加点击处理程序，然后在图标被点击时显示我们的`SideBarMenus`组件。因此，我们需要再次更新我们的`Nav.tsx`文件，以便包含我们的模态框，其中显示`SideBarMenus`。让我们更新`Nav.tsx`。首先添加适当的导入。然后，添加源代码中的代码。

1.  如果我们从*第 10 行*开始查看，我们会看到一个名为`showMenu`的新本地状态。我们将使用它来控制我们是显示还是隐藏我们的模态菜单。

1.  `onClickToggle`处理程序在`FontAwesomeIcon`中使用，在`getMobileMenu`函数内，用于切换`showMenu`本地状态，从而显示或隐藏模态框。

1.  在`ReactModal`中，当任何关闭请求进入组件时，我们需要设置控制显示的状态，以便可以明确地将其设置为 false；否则，模态框将不会消失。这就是`onRequestClose`的作用。`shouldCloseOnOverlayClick`属性允许我们关闭模态框，即使我们在外部任何地方点击也可以。这是用户通常期望的行为，所以最好有。

1.  最后，JSX 已经更新，以便我们可以添加我们的`ReactModal`，其中包括我们的`SideBarMenus`组件。

正如您所看到的，模态框被称为`ReactModal`，在其属性中，有一个名为`isOpen`的属性。这决定了模态框是否显示。

1.  如果您运行代码，然后点击汉堡图标，您将看到这个：

![图 12.7 – 带有 SideBarMenus 的 ReactModel](img/Figure_12.07_B15508.jpg)

图 12.7 – 带有 SideBarMenus 的 ReactModel

再次，随着我们添加更多功能，我们将扩展此菜单。

## 身份验证组件

现在我们已经设置好了我们的 SideBar，让我们开始构建我们的身份验证组件。我们将首先构建我们的注册、登录和注销屏幕：

1.  让我们首先创建注册模态框。为了做到这一点，我们需要在`SideBarMenus`组件内部添加一个注册链接。打开`SideBarMenus.tsx`文件并像这样更新它：

```ts
li to the returned JSX and included the new icon and label for the register.
```

1.  现在，在创建注册组件之前，让我们创建一个辅助服务，用于验证我们的密码。我们希望确保用户输入足够长且复杂的密码，因此我们需要一个名为`common`的`src`，然后另一个名为`validators`的文件夹。在`validators`文件夹中，创建一个名为`PasswordValidator.ts`的文件，并将以下代码添加到其中。代码非常简单，所以我不会在这里展示全部，但请注意密码强度和正则表达式。正则表达式只是在字符串中搜索模式的编程方式：

```ts
const strongPassword = new RegExp(
    "^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#$%^&*])      (?=.{8,})"
  );
  if (!strongPassword.test(password)) {
    passwordTestResult.message =
      "Password must contain at least 1 special       character, 1 cap letter, and 1 number";
    passwordTestResult.isValid = false;
  }
```

在这里，我们使用了正则表达式来检查适当的复杂性，确保我们的密码中既有字母、数字又有符号。括号表示一组相关表达式。因此，首先是小写字母，然后是大写字母，然后是数字，然后是符号，最后是预期长度：

```ts
  return passwordTestResult;
};
```

这段代码并不特别复杂，但由于我们将需要在多个组件中使用，比如注册和服务器端，将其放在一个单独的文件中更有利于代码重用。

注意

在 SPA 网页开发中，通常会对验证进行两次 – 一次在客户端，一次在服务器端。这样做可能看起来多余，但对于增加的安全性是必要的。一旦我们开始构建服务器端代码，我们将学习如何在项目之间共享这样的依赖。

1.  由于我们正在创建多个与身份验证相关的组件，让我们在`components`内部创建一个名为`auth`的文件夹，然后将我们的身份验证相关文件放在其中。一旦创建了`auth`文件夹，就在其中添加一个名为`Registration.tsx`的文件。将以下代码添加到文件中。如果您查看源代码，您将能够看到必要的导入。确保您的`App.css`文件也已更新。请注意，最终，我们将把其中一些代码移到共享位置，但现在，我们将直接在我们的`Registration`组件中使用它：

```ts
const userReducer = (state: any, action: any) => {
  switch (action.type) {
    case "userName":
      return { ...state, userName: action.payload };
    case "password":
      return { ...state, password: action.payload };
    case "passwordConfirm":
      return { ...state, passwordConfirm: action.payload        };
    case "email":
      return { ...state, email: action.payload };
    case "resultMsg":
      return { ...state, resultMsg: action.payload };
    default:
      return { ...state, resultMsg: "A failure has        occurred." };
  }
};
```

在这里，我们正在创建我们的 reducer，其中有许多相关字段：

```ts
export interface RegistrationProps {
  isOpen: boolean;
  onClickToggle: (
    e: React.MouseEvent<Element, MouseEvent> | React.    KeyboardEvent<Element>
  ) => void;
}
```

由于这是一个模态组件，我们允许父组件通过传递 props 来控制此组件的显示方式。`isOpen`prop 控制模态的显示方式，而`onClickToggle`函数控制模态的隐藏和显示：

```ts
const Registration: FC<RegistrationProps> = ({ isOpen, onClickToggle }) => {
  const [isRegisterDisabled, setRegisterDisabled] =    useState(true);
  const [
    { userName, password, email, passwordConfirm, resultMsg },
    dispatch,
  ] = useReducer(userReducer, {
    userName: "davec",
    password: "",
    email: "admin@dzhaven.com",
    passwordConfirm: "",
    resultMsg: "",
  });
```

在这里，我们有`isRegisterDisabled`本地状态值，如果给定值不正确，则禁用注册按钮，当然还有我们的本地 reducer，`userReducer`：

```ts
  const allowRegister = (msg: string, setDisabled:     boolean) => {
    setRegisterDisabled(setDisabled);
    dispatch({ payload: msg, type: "resultMsg" });
  };
```

`allowRegister`只是一个用于设置注册按钮为禁用并在需要时显示消息的辅助函数。

1.  接下来，我们有一系列`onChange`事件处理程序，用于每个字段，比如`userName`字段。它们根据需要进行验证，并更新输入的文本：

```ts
const onChangeUserName = (e: React.ChangeEvent<HTMLInputElement>) => {
    dispatch({ payload: e.target.value, type: "userName" });
    if (!e.target.value) allowRegister("Username cannot     be empty", true);
    else allowRegister("", false);
};
```

`onChangeUserName`函数用于设置`userName`并验证是否允许继续注册：

```ts
const onChangeEmail = (e: React.ChangeEvent<HTMLInputElement>) => {
dispatch({ payload: e.target.value, type: "email" });
if (!e.target.value) allowRegister("Email cannot be empty", true);
else allowRegister("", false);
};
```

`onChangeEmail`函数用于设置电子邮件并验证是否允许继续注册：

```ts
const onChangePassword = (e: React.ChangeEvent<HTMLInputElement>) => {
dispatch({ payload: e.target.value, type: "password" });
const passwordCheck: PasswordTestResult = isPasswordValid(e.target.value);
if (!passwordCheck.isValid) {
allowRegister(passwordCheck.message, true);
return;
}
passwordsSame(passwordConfirm, e.target.value);
};
```

`onChangePassword`函数用于设置密码并验证是否允许继续注册：

```ts
const onChangePasswordConfirm = (e: React.ChangeEvent<HTMLInputElement>) => {
    dispatch({ payload: e.target.value, type:     "passwordConfirm" });
    passwordsSame(password, e.target.value);
};
```

`onChangedPasswordConfirm`函数用于设置`passwordConfirm`并验证是否允许继续注册：

```ts
const passwordsSame = (passwordVal: string, passwordConfirmVal: string) => {
if (passwordVal !== passwordConfirmVal) {
allowRegister("Passwords do not match", true);
return false;
} else {
allowRegister("", false);
return true;
}
};
```

最后，由于这是一个注册组件，我们使用`passwordsSame`来检查密码和确认密码是否相等。

1.  接下来，我们有`onClickRegister`和`onClickCancel`。`onClickRegister`按钮点击处理程序将提交尝试的注册。目前，由于我们没有后端，它不会进行实际提交，但一旦服务器启动，我们将填写它。另一方面，`onClickCancel`处理程序退出`Registration`组件：

```ts
const onClickRegister = (
e: React.MouseEvent<HTMLButtonElement, MouseEvent>
) => {
e.preventDefault();
onClickToggle(e);
};
const onClickCancel = (
e: React.MouseEvent<HTMLButtonElement, MouseEvent>
) => {
onClickToggle(e);
};
```

请注意，`e.preventDefault`函数只是阻止了标准行为，这取决于上下文而有所不同。在表单的情况下，我们的`onClickRegister`处理程序与表单标签内的按钮相关联，因此默认行为是提交并导致页面刷新。页面刷新是`preventDefault`。

1.  现在事件处理程序已经设置好，我们返回与这些处理程序相关联的 JSX。首先，我们从`ReactModal`包装组件开始：

```ts
return (
    <ReactModal
        className="modal-menu"
        isOpen={isOpen}
        onRequestClose={onClickToggle}
        shouldCloseOnOverlayClick={true}
    >
    <form>
        <div className="reg-inputs">
            <div>
                <label>username</label>
                <input type="text" value={userName}                onChange={onChangeUserName} />
            </div>
```

同样，我们的模态是由父组件通过`isOpen`和`onClickToggle`props 来外部控制的。

```ts
<div>
        <label>email</label>
        <input type="text" value={email}          onChange={onChangeEmail} />
      </div>
```

在这里，我们有我们的电子邮件字段。

```ts
      <div>
        <label>password</label>
        <input
          type="password"
          placeholder="Password"
          value={password}
          onChange={onChangePassword}
        />
    </div>
```

这是我们的密码字段。

```ts
    <div>
        <label>password confirmation</label>
        <input
            type="password"
            placeholder="Password Confirmation"
            value={passwordConfirm}
            onChange={onChangePasswordConfirm}
            />
        </div>
    </div>
```

这是我们的密码确认字段。

```ts
    <div className="reg-buttons">
        <div className="reg-btn-left">
            <button
                style={{ marginLeft: ".5em" }}
                className="action-btn"
                disabled={isRegisterDisabled}
                onClick={onClickRegister}
            >
            Register
            </button>
```

这是我们的注册按钮。

```ts
            <button
                style={{ marginLeft: ".5em" }}
                className="cancel-btn"
                onClick={onClickCancel}
            >
            Close
            </button>
```

这是我们的取消按钮。

```ts
        </div>
            <span className="reg-btn-right">
                <strong>{resultMsg}</strong>
            </span>
        </div>
        </form>
        </ReactModal>
    );
};
export default Registration;
```

最后，请注意，我们有一个消息部分，它使用`resultMsg` reducer 字段。如果出现问题，它将显示错误。

1.  现在，如果您在桌面模式下运行应用程序，您应该会看到类似这样的东西：![图 12.8-桌面注册模态视图](img/Figure_12.08_B15508.jpg)

图 12.8-桌面注册模态视图

如果您运行 Chrome 调试器并切换到移动模式，然后单击汉堡图标，然后单击注册标签，您将看到以下屏幕：

![图 12.9-移动注册模态视图](img/Figure_12.09_B15508.jpg)

图 12.9-移动注册模态视图

正如您所看到的，我们能够通过使用 CSS 响应能力有效地获得两个屏幕，而只使用一个组件。

1.  现在，让我们继续登录模态。如果我们看一下现有的`Registration`组件，我们会发现它包含一些代码，我们也可以在`Login`组件中使用。我们真的应该重构代码，以便可以重用它。例如，`Registration`，`Login`和`Logout`都将使用`ReactModal`，因此接收控制模态显示的 props。因此，让我们看看我们可以做些什么来重用我们现有的代码。首先，让我们从`Registration.tsx`文件中提取`RegistrationProps`接口，并将其放在自己的文件中。在`components`内创建一个名为`types`的文件夹。然后，创建一个名为`ModalProps.ts`的文件，并添加`RegistrationProps`接口。将其重命名为`ModalProps`。

如您所见，它与`RegistrationProps`相同，只是名称更改。现在，打开`Registration.tsx`文件，删除`RegistrationProps`，并导入`ModalProps`。然后，用`ModalProps`替换`RegistrationProps`。检查一下是否一切正常运行。

1.  我们重构了`ModalProps`，以便它可以在组件之间重复使用。现在，让我们拿出`UserReducer`，因为`Login`使用了它的一些字段。在现有的`auth`文件夹内创建一个名为`common`的新文件夹，并创建`UserReducer.ts`文件。将以下代码放入其中：

```ts
const userReducer = (state: any, action: any) => {
  switch (action.type) {
    case "userName":
      return { ...state, userName: action.payload };
    case "password":
      return { ...state, password: action.payload };
    case "passwordConfirm":
      return { ...state, passwordConfirm: action.payload       };
    case "email":
      return { ...state, email: action.payload };
    case "resultMsg":
      return { ...state, resultMsg: action.payload };
    case "isSubmitDisabled. This field will replace the existing isRegisterDisabled so that it can be used to disable buttons across any authentication screens.Now, remove `userReducer` from the `Registration.tsx` file and import it from the new `UserReducer.ts` file. Also, replace `isRegisterDisabled` with `isSubmitDisabled` and include `isSubmitDisabled` in your `destructured` object, as well as the state initializer of the `useReducer` Hook call.
```

1.  现在，让我们进行另一个重构。`Registration`中的`allowRegister`函数禁用了一个按钮并更新了状态消息。这也可以清楚地被重用。让我们在 common 文件夹内创建一个名为`Helpers.ts`的新文件，并将以下代码放入其中：

```ts
import { Dispatch } from "react";
export const allowSubmit = (
  dispatch: Dispatch<any>,
  msg: string,
  setDisabled: boolean
) => {
  dispatch({ type: "isSubmitDisabled", payload: setDisabled });
  dispatch({ payload: msg, type: "resultMsg" });
};
```

如您所见，我们将函数名称更改为`allowSubmit`，现在将`dispatch`作为参数。现在，从`Registration`中删除`allowRegister`，并导入新的`allowSubmit`函数，并将`allowRegister`调用更新为`allowSubmit`调用。检查一下您的`Registration.tsx`文件的代码是否与源代码一致。

我们将保持两个`onClick`调用不变，即使`Login`也将有类似的调用，因为一旦我们的后端准备好，我们可能还需要为这些调用做一些特定于组件的事情。

现在，您应该能够运行此代码。

1.  现在，我们可以在新的`Login`组件中使用新提取的代码。在`auth`文件夹中，创建一个名为`Login.tsx`的新文件，并从源代码中添加相关代码。我将在这里突出显示一些项目：

```ts
  const [
    { userName, password, resultMsg, isSubmitDisabled },
    dispatch,
  ] = useReducer(userReducer, {
    userName: "",
    password: "",
    resultMsg: "",
    isSubmitDisabled: true,
  });
```

由于我们的`Login`组件与我们的`Registration`组件有不同的需求，我们只使用了`userReducer`中的一部分字段，通过对象解构来实现。

在 JSX 中，注意我们已经更新了一些 CSS 类，以便更好地对齐按钮。这些新类在`App.css`文件中。

1.  最后，我们需要添加一个登录链接。更新`SideBarMenu.tsx`文件，如源代码所示。

由于`Logout`非常相似，我已经添加了该组件，但不会在这里进行介绍。随着后端的进一步完善，我们将添加代码来控制显示哪些菜单链接取决于用户的登录状态。我们还将添加额外的验证。但在此之前，我们还有很多工作要做，所以让我们继续。

## 路由和屏幕

现在，让我们继续创建应用程序需要的路由。到目前为止，我们的应用程序只有一个 URL。根 URL 是`http://localhost:3000`。现在，我们希望将我们的应用程序划分为具有特定部分的不同路由。我们将从获取现有代码开始，修改它，并将其作为我们的第一个根 React 路由。让我们开始吧：

1.  首先，让我们将我们的网格区域相关组件移入不同的文件夹。首先，在`components`文件夹内创建一个名为`areas`的文件夹。然后，将`Nav.tsx`、`Nav.css`、`RightMenu.tsx`、`Main.tsx`、`LeftMenu.tsx`和`LeftMenu.css`文件，以及整个`sidebar`文件夹，移入新的`areas`文件夹。您的文件路径导入将需要更新，包括`App.tsx`文件。查看源代码，了解如何操作。

1.  完成后，在`areas`内创建一个名为`main`的新文件夹，并将`Main.tsx`文件移入其中。确保更新您的路径。我们将把所有与主要区域相关的组件添加到此文件夹中。

1.  我们将在此文件夹中创建的第一个新组件是`MainHeader`组件。顾名思义，它将用作主区域的标题。它将显示我们当前正在查看的主题项目的类别。在`main`文件夹内创建`MainHeader.tsx`文件，并将源代码中的代码添加到其中。

此控件的唯一目的是显示当前的`Category`名称。

再次注意，我们在`MainHeader.css`和`App.css`文件中有一些新的 CSS 类。

## 主屏幕

在继续之前，让我们为我们的新路由执行一些基本设置。在这里，我们将创建我们的新屏幕组件`Home`，并更新任何相关文件，例如`App.tsx`：

1.  当我们第一次创建`App.tsx`文件时，我们假设我们的应用程序只有一个屏幕。显然，这是不正确的。现在我们已经完善了我们的布局，让我们开始添加我们不同的屏幕和路由。打开`App.tsx`文件并像这样更新它。

在这里，我们添加了一个名为`Home`的新导入，代表了主页路由。我们稍后会构建这个：

```ts
import Home from "./components/routes/Home";function App() {
const renderHome = (props: any) => <Home {...props} />;
```

我们在这里定义一个函数，以发送到我们路由的`render`属性。这个函数允许所有路由的 props，以及我们想要发送的任何自定义 props，在初始化我们的`Home`组件时包含在内：

```ts
  return (
    <Switch>
      <Route exact={true} path="/" render={renderHome} />
      <Route
        path="/categorythreads/:categoryId"
        render={renderHome}
      />
    </Switch>
  );
}
```

因此，以前显示我们的网格区域的代码现在将在`Home`组件中，而这个组件我们稍后会构建。

如*第七章**所示，学习 Redux 和 React Router*，我们的`Switch`组件允许 React Router 根据提供的 URL 更改路由屏幕的渲染。目前，我们将有两个指向相同`Home`屏幕的路由，但稍后我们将添加更多。根路径将显示默认类别的线程，而`categorythreads`路由将显示特定类别的线程。

1.  在创建新的`Home`组件之前，让我们稍微重构一下我们的 CSS，并使其更具可重用性。首先，通过在`App`类之前添加以下类来更新`App.css`文件：

```ts
.screen-root-container {
  margin: 0 auto;
  max-width: 1200px;
  margin-bottom: 2em;
  border: var(--border);
  border-radius: 0.3em;
}
```

这将成为我们应用程序中代表路由屏幕的任何组件的根类。

1.  接下来，在`components/routes`文件夹内创建一个名为`Home.css`的新文件。现在，从`App.css`中剪切整个 CSS 样式集：

```ts
.Home.css file. Once they've been copied over, change the name of the App class to home-container. We're changing the name so that the class' purpose is clearer. Now, let's create our new Home screen component and learn how to use these CSS classes.
```

1.  在`components`文件夹内创建一个名为`routes`的文件夹，并在其中添加一个名为`Home.tsx`的新文件。代码很简短简单，所以您可以直接从源代码中复制。这主要是来自以前版本的`App.tsx`的旧代码。

我们已经更新了我们的根 CSS`App`类，现在它是`screen-root-container home-container`。在一个类属性中使用两个类，意味着首先应用第一个类的样式，然后应用下一个类的样式，这将覆盖之前的任何设置。此外，我们现在将能够在其他屏幕中使用`screen-root-container`。

我们已经成功地将原始的`App.tsx`代码移到了`Home.tsx`文件中。请注意，我们还将我们的`Nav`组件放在了一个`div`标签内。我们这样做是为了以后可以在其他屏幕中重用`Nav`组件。您现在应该从`Nav.tsx`组件文件中删除`className="navigation"`属性。

1.  现在我们已经更新了我们的`Home`屏幕，我们需要更新我们的`Main`组件，以便列出给定类别内的线程。为了做到这一点，我们实际上需要做一些更新。首先，我们需要创建两个新模型，名为`Thread`和`ThreadItem`。`Thread`是初始帖子，而`ThreadItem`是一个回复。让我们从我们的模型开始。

首先，在`models`文件夹中创建`Thread.ts`，如源代码所示。

这里没有太多需要解释的，因为这是相当明显的。但是，请注意，`points`表示点赞的总数。

接下来，让我们做`ThreadItem.ts`。创建所需的文件并将源代码添加到其中。这与`Thread`非常相似。

1.  现在，我们将创建线程卡文件组件。这个组件将代表一个单独的线程记录，并显示诸如标题、正文和点数等内容。在`components/areas/main`文件夹内创建一个名为`ThreadCard.tsx`的文件。然后，添加代码：

```ts
import React, { FC } from "react";
import "./ThreadCard.css";
import Thread from "../../../models/Thread";
import { Link, useHistory } from "react-router-dom";
import { faEye, faHeart, faReplyAll } from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
import { useWindowDimensions } from "../../../hooks/useWindowDimensions";
```

首先，我们有各种导入，包括`Link`对象和来自 React Router 的`useHistory` Hook：

```ts
interface ThreadCardProps {
  Thread object as our parameter. We will use this object and its members as we render our ThreadCard UI:

```

const ThreadCard: FC<ThreadCardProps> = ({ thread }) => {

const history = useHistory();

const { width } = useWindowDimensions();

const onClickShowThread = (e: React.  MouseEvent<HTMLDivElement>) => {

history.push("/thread/" + thread.id);

};

```ts

Here, we are using the React Router `useHistory` Hook to get the `history` object. When someone clicks on our Thread, we use the `history` object to redirect the app to a new URL by `pushing` the new URL on top of the `history` object. We will build our thread route and component later:

```

const getPoints = (thread: Thread) => {

如果（宽度 <= 768）{

return (

<label

style={{

marginRight: ".75em",

marginTop: ".25em",

}}

{线程.点数 || 0}

<FontAwesomeIcon

icon={faHeart}

className="points-icon"

style={{

marginLeft: ".2em",

}}

/>

</label>

);

}

return null;

};

```ts

The `getPoints` function creates the UI for displaying "likes" on our post. However, since our UI is responsive, it does not appear in desktop mode when we check the screen's `width` property:

```

const getResponses = (thread: Thread) => {

如果（宽度 <= 768）{

return (

<label

style={{

marginRight: ".5em",

}}

{线程 && 线程.线程项目 && 线程.线程项目.length}

```ts

This function shows the response count, as indicated by the `thread.threadItems.length` property:

```

<FontAwesomeIcon

icon={faReplyAll}

className="points-icon"

style={{

marginLeft: ".25em",

marginTop: "-.25em",

}}

/>

</label>

);

}

return null;

};

```ts

The `getResponses` function shows how many `ThreadItems` responses there are for this `Thread`. However, since our UI is responsive, it does not appear in desktop mode when we check the screen's `width` property:

```

const getPointsNonMobile = () => {

如果（宽度 > 768）{

return (

<div className="threadcard-points">

<div className="threadcard-points-item">

{线程.点数 || 0}

<br />

<FontAwesomeIcon icon={faHeart}              className="points-icon" />

</div>

<div

className="threadcard-points-item"

style={{ marginBottom: ".75em" }}

{线程 && 线程.线程项目 && 线程.线程项目.length}

```ts

This function is getting the likes count, as indicated by the `thread.threadItems.length` property:

```

<br />

<FontAwesomeIcon icon={faReplyAll}              className="points-icon" />

</div>

</div>

);

}

return null;

};

```ts

The `getPointsNonMobile` function returns the points column on the right of `ThreadCard`, but only renders it if the device is a desktop or laptop with a screen width bigger than 768 pixels.Remember that every React component that may be used multiple times on the same screen must have a unique `key` value. So, later, when we use this component, you will see that each instance has been given a unique `key` value. The following JSX is returning the `Category` name as a `Link` so that when it's clicked, the user will be sent to the screen showing the Threads for that `Category`:

```

return (

<section className="panel threadcard-container">

<div className="threadcard-txt-container">

<div className="content-header">

<Link

to={`/categorythreads/${thread.category.id}`}

className="link-txt"

<strong>{thread.category.name}</strong>

</Link>

```ts

`Link` is a React Router component that renders a URL anchor (HTTP link). Notice that `categorythreads` is the second route we created earlier and that it takes `categoryId` as a parameter:

```

<span className="username-header" style={{            marginLeft: ".5em" }}>

{thread.userName}

</span>

</div>

<div className="question">

<div

onClick={onClickShowThread}

data-thread-id={thread.id}

style={{ marginBottom: ".4em" }}

<strong>{thread.title}</strong>

</div>

<div

className="threadcard-body"

onClick={onClickShowThread}

data-thread-id={thread.id}

<div>{thread.body}</div>

</div>

```ts

As you can see, we use the `thread` prop extensively while rendering our UI.Here, we are using the `getPoints` and `getResponses` functions to render a subset of our UI so that it shows points and responses:

```

<div className="threadcard-footer">

<span

样式={{

marginRight: ".5em",

}}

<label>

{thread.views}

<FontAwesomeIcon icon={faEye}                  className="icon-lg" />

</label>

</span>

<span>

{getPoints(thread)}

{getResponses(thread)}

</span>          </div>

</div>

</div>

```ts

Here, we are using `getPointsNonMobile` to show our response count and likes:

```

{getPointsNonMobile()}    </section>

);

};

export default ThreadCard;

```ts

Notice that we have referenced many CSS classes in this component, all of which can be found in the `ThreadCard.css` and `App.css` files in the source code. I won't go over every single CSS class here, but if you look at the `ThreadCard.css` file, you'll notice that there is a reference to something called `flex`. Flexbox is another method of creating a layout in CSS similar to Grids. However, Flexbox is intended to be used for single-row or single-column layouts; for example:

```

.threadcard-txt-container {

display: flex;

flex-direction: column;

width: 92%;

margin: 0.75em 1em 0.75em 1.2em;

border-right: solid 1px var(--border-color);

}

```ts

In this CSS, the display method is indicated as `flex` and `flex-direction` is column. This means that the layout of all the elements inside `threadcard-txt-container` will be in a single stacked column. So, even if we had elements such as labels or buttons, which are normally set in a horizontal line, if they live inside a column-based flex container, they will be laid out vertically. If we had used the row attribute, then the layout would be horizontal.
```

1.  现在我们已经创建了我们的 Thread 容器，`ThreadCard`，让我们更新我们的`Main.tsx`文件，以便我们可以使用它。添加源代码中的代码。

如果您查看*第 8 行*，您将看到`useParams`函数被使用。之前，我们在`App.tsx`文件中为 React Router 创建了两个路由。其中一个路由`categorythreads`接受了一个 URL 参数。通过使用`useParams` Hook，我们可以获取路由参数 - 在这种情况下是`categoryId` - 以便我们可以使用它们。

然后，在*第 9 行*，我们有`category`状态。一旦我们从线程列表中检索到我们的类别，我们将更新此状态。

在*第 10 行*，我们有一个状态对象，它是我们的`ThreadCards`列表，称为`threadCards`。

然后，在`useEffect`中，如果我们获得新的`categoryId`，我们将更新我们的`ThreadCards`列表。当我们获得有效的`categoryId`时，我们使用我们的`DataService`来查询特定类别的线程列表，然后构建出`ThreadCards`列表。我们还取第一个线程来获取类别的名称，因为它们都属于同一类别。

最后，我们返回我们的 UI。

注意

有时，您会看到有关`useEffect` Hook 数组中缺少依赖项的警告。这些是我认为是有意见的警告，通过经验，您将能够判断哪些可以安全地忽略。例如，在`Main.tsx`的`useEffect`中，我故意忽略了关于`category`状态对象的警告，因为将对象包含在数组中会触发`useEffect`的不必要的双重运行（因为`useEffect`在其数组列表中的某些内容发生变化时运行），可能会导致双重渲染。

1.  现在，让我们尝试在桌面模式下运行。转到`http://localhost:3000/categorythreads/1`。你应该看到以下内容：

![图 12.10 - 类别帖子 URL 的桌面视图](img/Figure_12.10_B15508.jpg)

图 12.10 - 类别帖子 URL 的桌面视图

在移动设备上它是这个样子的：

![图 12.11 - 类别帖子 URL 的移动视图](img/Figure_12.11_B15508.jpg)

图 12.11 - 类别帖子 URL 的移动视图

正如你所看到的，在移动模式下，我们没有右侧的积分栏。相反，这些积分在主文本部分的底部。图标显示，对于第一篇帖子，有两个人看过它。有 55 个人喜欢它，还有一个人回复了。

哇 - 我们刚刚经历了很多代码！但是，我们还没有完成！让我们继续构建我们的`RightMenu`组件。

在我们的`RightMenu`上，我们想要显示一个帖子数量最多的三个类别的列表。在每个类别中，我们将显示最多人浏览的帖子。让我们开始吧：

1.  首先，在`areas`文件夹内创建一个名为`rightMenu`的`RightMenu`文件夹。

1.  现在，在那个文件夹里创建一个名为`TopCategory.tsx`的新文件。这个组件将代表一个单独的顶级类别及其帖子。

1.  创建一个新的模型，代表从服务器传来的数据。让我们称之为`CategoryThread`。在`models`文件夹内创建一个名为`CategoryThread.ts`的文件，并输入源代码。

1.  现在，我们需要更新我们现有的`RightMenu`组件，并创建一个新的组件来显示我们的`CategoryThread`项目。为了对我们的`CategoryThread`项目进行分组和组织，我们需要使用一个叫做 Lodash 的工具来帮助我们。

Lodash 是一个提供了大量 JavaScript 辅助函数库的依赖项。在这里不可能涵盖它的所有功能。然而，Lodash 特别适用于管理数组和集合。你会发现它非常容易使用，但如果你想要更多细节，这里是他们的文档链接：`https://lodash.com/docs/`。像这样安装 Lodash：

```ts
import _ from "lodash". You will add an enormous amount of code to your project by doing so. Only import the specific call using import groupBy from "lodash/groupBy".Now, we can update our `RightMenu.tsx` file as shown in the source code.First, notice that in addition to Lodash, we also imported a new `RightMenu.css` file, along with some minor styling. We also imported the `TopCategory` component, which we'll build after.Next, we have a new state object called `topCategories` that we will use to store our array of top categories.Then, in `useEffect`, we have our top categories from the `getTopCategories` function. Then, we group the results by category and create our array of `TopCategory` elements. The `TopCategory` component elements will display our data. Notice that the `TopCategory` component receives each group of top categories through the `topCategories` prop.The component then returns the `topCategories` elements.
```

1.  现在，我们需要构建我们的`TopCategory`组件。在与`RightMenu`相同的文件夹中创建一个名为`TopCategory.tsx`的文件，并添加相关的源代码。

在顶部，请注意我们有一个补充的 CSS 文件叫做`TopCategory.css`。

接下来，我们有一个名为`TopCategoryProps`的新接口用于接收 props。在*line 10*上，当准备好时，帖子状态对象将存储我们的 JSX 元素。

然后，在*line 12*上，我们有`useEffect`，我们将使用它基于传入的 prop 来构建我们的 UI 元素；也就是说，`topCategories`。

返回的 JSX 有一个`strong`标题，这是找到的第一个类别元素的名称，因为顶级类别的数组总是来自一个类别。然后，我们包括了我们的帖子列表。

1.  由于这个`RightMenu`在移动设备上不渲染，让我们看看在桌面上它是什么样子的：

![图 12.12 - 带有顶级类别的 RightMenu](img/Figure_12.12_B15508.jpg)

图 12.12 - 带有顶级类别的 RightMenu

好的 - 我们快要完成了！我们已经完成了大部分我们主屏幕所需的内容，但现在，我们需要我们的应用程序显示单独的**帖子**。

## 帖子屏幕

这个屏幕将是多用途的。使用这个屏幕，我们将能够创建一个新的帖子或显示一个现有的帖子。我们还将在同一个屏幕上显示帖子的回复。让我们开始吧：

1.  首先，我们需要创建我们的新路由组件。我们将称之为`Thread.tsx`，并将其放在一个名为`thread`的新文件夹中，该文件夹应放在`routes`文件夹内。然而，我们的`Thread`组件将会很复杂，所以我们应该将其拆分为称为子组件的模块化部分。在这种情况下，这样做不会给我们带来代码重用的好处。然而，这将使代码更容易阅读和重构，因为它将被分发成块，而不是一个非常庞大的单体。让我们创建一个名为`ThreadHeader.tsx`的新组件文件，并将源代码添加到其中。

首先，注意我们正在导入的新函数`getTimePastIfLessThanDay`。这个函数将查看传入的日期并适当地格式化它，以便易于阅读。

这个组件将接受字段作为参数，而不具有自己的状态。`ThreadHeader`充当一个仅用于显示的组件。它显示了主题的`title`、`userName`和`lastModifiedOn`时间。

1.  现在，创建`Thread.tsx`文件并将源代码添加到其中。

请注意，我们正在导入一个新的`Thread.css`文件和我们的新的`ThreadHeader`组件。还要注意，由于我们的组件也被称为`Thread`，就像我们的模型一样，我将我们的模型导入为`ThreadModel`。这种问题在大型项目中可能会经常发生，所以你应该知道你可以以这种方式导入。

接下来，我们必须创建我们的本地`thread`状态对象，它是`ThreadModel`类型。然后，我们必须再次使用`useParams` Hook 来获取路由参数的`id`，这是这条线程的 ID。

在`useEffect`中，如果`id`路由参数存在且大于`0`，我们尝试获取我们的`thread`。稍后，一旦我们的后端准备好了，我们将编写一些代码，以便可以插入新的线程。

最后，我们返回我们的 UI，其中包括`ThreadHeader`。请注意，`lastModifiedOn`字段是非空的，所以我们使用三元检查来检查`thread`是否为空，如果为空则返回当前日期。

1.  现在，我们需要为我们的`Thread`屏幕组件创建一个新的路由。再次打开`App.tsx`并更新代码，就像这样：

```ts
function App() {
  const renderHome = (props: any) => <Home {...props} />;
  const renderThread = (props: any) => <Thread {...props} />;
```

在这里，我们为我们的`Thread`组件添加了`renderThread`函数：

```ts
  return (
    <Switch>
      <Route exact={true} path="/" render={renderHome} />
      <Route
        path="/categorythreads/:categoryId"
        render={renderHome}
      />
      <Route
        path="/thread/:id"
        render={renderThread}
      />
    </Switch>
  );
}
```

请注意，我们的`Thread`的路由是`"/thread/:id"`，这意味着在线程路径之后，它期望一个参数。在内部，React Router 将其标记为`id`。

1.  现在，我们将添加我们线程屏幕的下一部分。在这个屏幕上，我们将通过下拉菜单显示线程的类别。然而，由于 HTML 中标准的下拉菜单，称为`select`元素，外观丑陋且与 React 集成不佳，我们将使用一个名为`react-dropdown`的 NPM 包来帮助我们获得一个更具吸引力和 React 集成的控件。

像这样安装`react-dropdown`：

```ts
ThreadCategory.tsx in the thread folder and add the source code to it.Once you've set up the imports, create the `ThreadCategoryProps` interface, which will represent our prop type.Next, we start creating our `ThreadCategory` component and set up a constant variable, `catOptions`, that contains the items that will appear as selectable options in our dropdown. Again, we are only temporarily hardcoding values until our backend is ready.Finally, we are returning the JSX with an initialized `DropDown` control.
```

1.  现在，让我们创建我们的`Title`组件。我们将它称为`ThreadTitle`。在`thread`文件夹中创建一个名为`ThreadTitle.tsx`的文件，并将源代码添加到其中。

这只是一个简单的渲染器，所以我不会在这里解释它。然而，请注意，目前我们的`onChangeTitle`处理程序是空的。同样，一旦我们的后端准备好了，我们将区分读取和写入状态，并实现`onChangeTitle`函数。

1.  现在，让我们更新我们的`Thread.tsx`文件，看看我们到目前为止做了什么。像这样更新`Thread.tsx`。请注意，随着我们添加这些与 Thread 相关的组件，我们一直在更新`Thread.css`文件，所以也要保持你的 CSS 文件更新。

状态和`useEffect`代码基本上是一样的，所以我不会在这里展示它：

```ts
  return (
    <div className="screen-root-container">
      <div className="thread-nav-container">
        <Nav />
      </div>
      <div className="thread-content-container">
        <ThreadHeader
          userName={thread?.userName}
          lastModifiedOn={thread ? thread.lastModifiedOn : new Date()}
          title={thread?.title}
        />
        <ThreadCategory categoryName={thread?.category?.name} />
        <ThreadTitle title={thread?.title} />
      </div>
    </div>
  );
};
```

在这里，我们已经将新的组件添加到了返回的 JSX 中。正如你所看到的，我们的代码比在`Thread.tsx`文件中拥有单独的元素和事件处理程序时要简短和易读得多。

如果你通过`http://localhost:3000/thread/1`运行应用程序，你应该会看到这个：

![图 12.13 - 线程屏幕](img/Figure_12.13_B15508.jpg)

图 12.13 - 线程屏幕

请注意右侧的空白处是我们将为线程添加喜欢和回复计数信息的地方。

现在，我们不会在这里审查每个 CSS 文件，因为我们想专注于代码，但由于这是一个重要的屏幕和路由目的地，让我们审查一下 CSS，看看我们是如何布局的。到目前为止，我们就是这样。更新你的`Thread.css`文件，这样我们就可以一起看一下。

就像我们之前在主屏幕上做的那样，我们将我们的导航控件放在自己的名为`thread-nav-container`的 div 容器中。

`thread-content-container`类是实际的 Thread 内容布局。正如您所见，布局是一个具有两列和不确定数量行的网格。

其余内容使用`grid-column`属性添加到第一列。稍后我们将添加第二列来保存我们的 Thread 的点（赞）。

1.  现在，我们需要为我们的 Thread 帖子的正文添加一个部分。正文条目更复杂，因为我们需要添加一个富文本条目格式化程序。这个控件将允许用户格式化他们的文本并进行更复杂的编辑。

为了创建我们的正文，让我们安装一个名为 Slate.js 的 NPM 包。这将是我们的富文本编辑器和格式化程序。我们还需要安装几个依赖项，包括一个叫做 Emotion 的东西。Emotion 是一个允许我们直接在 JavaScript 中使用 CSS 的库：

```ts
editor inside our components folder and create a new file called RichTextControls.tsx. This file contains the controls that we will be using in our editor. The source code I am using is from the Slate.js project at https://github.com/ianstormtaylor/slate/blob/master/site/components.tsx. This code is fairly large, so I'll show and explain the relevant code as we use each control. 
```

1.  接下来，我们需要在相同的`editor`文件夹中创建`RichEditor.tsx`文件，并将此代码添加到其中。

在我们的导入部分的顶部，我们可以看到通常的与 React 相关的导入，但也有两个 Slate.js 的导入。这些是帮助我们创建编辑器 UI 的。我稍后会更详细地解释这些。

`isHotKey`导入是一个帮助我们为编辑器构建键盘快捷键的工具。

`withHistory`导入允许编辑器保存已发生的编辑，以正确的顺序，以便在需要时可以撤消。

`Button`和`Toolbar`是可以用来构建我们的编辑器 UI 的控件。我们稍后将创建`RichTextControls`文件。

现在，我们可以导入我们的图标和 CSS 样式表。

`HOTKEYS`变量是一个包含各种快捷键到格式配对的字典。左边的`[keyName: string]`表示字典键；右边显示值。

在*第 26 行*，我们有`initialValue`变量。我们的编辑器使用对象作为其值，而不是字符串。因此，`initialValue`变量表示编辑器的起始值对象。类型是来自 Slate.js 编辑器的`Node`数组。在 Slate.js 中，文本被表示为节点的分层树。这是为了确保文本的结构保持完整，同时也允许格式信息与文本一起存在。您可以将其视为文本和元数据一起。

`LIST_TYPES`数组用于区分条目是段落还是文本列表。

在*第 38 行*，我们开始创建我们的`RichEditor`组件。正如我们之前提到的，在 Slate.js 中，编辑器内文本的值或内容不是普通文本。它是一个 JSON 对象，其根类型是`Node`。因此，我们的主文本值，称为`value`，是`Node`数组类型的状态对象。

接下来，我们有`renderElement`函数，它在内部用于呈现较大的文本片段。`Element`是一组多行文本。我们稍后将构建`Element`组件。

然后，我们有`renderLeaf`函数，用于呈现较小的文本片段。`Leaf`是一小段文本。我们稍后将创建这个组件。

请注意，我们在*第五章**，使用 Hooks 进行 React 开发*中介绍了`useCallback`和`useMemo`等 Hooks。

然后我们有编辑器变量。编辑器是接受和显示文本的 React 组件，而不是`Slate`、`Toolbar`和`Editable`组件，它们作为编辑器周围的包装器，并为其注入或修改文本格式。

`useEffect`函数用于获取`existingBody`属性并将其作为本地状态值，假设传入了`existingBody`。再次强调，`existingBody`仅在查看模式下传入，而不是创建模式。

`onChangeEditorValue`事件处理程序在 UI 中更改时设置本地`value`状态。再次注意，值类型不是文本，而是`Node`数组。

从*第 59 行*开始，我们开始定义我们的 JSX。我们使用我们的`editor`实例、本地`value`状态和`onChange`事件初始化我们的 Slate 包装组件。

接下来，`Toolbar`来自`RichTextControls.tsx`文件，表示一个布局容器，并包含我们的格式化按钮。它们看起来像这样。我稍后会解释`MarkButton`和`BlockButton`：

![图 12.14 – Slate.js 工具栏按钮](img/Figure_12.14_B15508.jpg)

图 12.14 – Slate.js 工具栏按钮

可编辑控件包含了我们编辑器的主要格式化程序、快捷键和基本设置。

请注意，为了可读性，我已经将大部分函数移到主组件之外。

在*第 92 行*，我们有我们的`MarkButton`控件。`MarkButton`是一个生成按钮 UI 并关联实际格式化程序的函数，当特定按钮被点击时触发。通常，标记用于单词或字符，而不是通常是多行语句的块。`Button`来自我们的`RichTextControls.tsx`文件。它表示我们工具栏上的样式化按钮。

接下来，我们有`isMarkActive`函数。`isMarkActive`函数确定格式化程序是否已经应用。

接下来，`toggleMark`函数将根据是否已应用格式来切换格式。它将编辑器与格式关联起来。

`BlockButton`设置文本块的格式并创建其按钮。通常，一个块包含多个`Nodes`。

`isBlockActive`函数确定是否应用了格式。

`ToggleBlock`切换应用的格式。

接下来，`Element`组件确定要使用哪种类型的 HTML。`Elements`在 Slate.js 中经常使用。

我们使用`Leafs`来确定要返回的较小的 HTML。`Leafs`在 Slate.js 中经常使用。

现在我们有了一个可重用的富文本编辑器。我们肯定会在我们的线程显示中使用这个组件。现在，因为它是自己的组件，我们可以在任何地方重用这段代码。

1.  现在，我们需要将我们的新`RichEditor`添加到我们的`ThreadBody.tsx`文件中。它是一个小组件，所以只需从源代码中添加代码。

1.  最后，我们需要从我们的`Thread`组件中引用我们的`ThreadBody`，就像这样。确保您有所有必要的导入。然后，在 JSX 中，在`ThreadTitle`的下面，添加以下代码：

```ts
        <ThreadBody body={thread?.body} />
```

再次注意，现在将它放入组件中，阅读和理解这个 JSX 是多么容易。

现在，让我们看看这是什么样子的：

![图 12.15 – 线程输入屏幕及其编辑器](img/Figure_12.15_B15508.jpg)

图 12.15 – 线程输入屏幕及其编辑器

我们的富文本编辑器提供以下选项：加粗、斜体、下划线、显示为代码、制作标题、引用、编号列表和项目符号列表。正如您所看到的，我们所有的格式化工作都很好。

在使用 Slate.js 时，您可能会想知道为什么会出现项目符号，即使我们之前已经在我们的`index.css`文件中添加了删除`ul`样式的 CSS。为了在我们的编辑器中获得适当的样式，我更新了那个样式，就像这样：

```ts
ul:not([data-slate-node="element"]) {
  list-style-type: none;
}
```

这是一个 CSS 选择器，表示“如果元素上有一个名为`data-slate-node`的自定义属性，则不应用此样式”。这是 Slate.js 用来区分自己的元素和其他标准 HTML 的方法。

哇，那是很多代码！但是，我们还没有完成。我们仍然需要在右侧创建我们的积分列，添加我们的响应功能，并允许添加`ThreadItems`。让我们稍后再处理积分列，先处理我们的响应系统：

1.  我们要做的第一件事是一些重构。在我们的`ThreadHeader`组件中，我们显示了`userName`和`lastModifiedOn`来让用户知道谁创建了帖子以及何时创建的。我们也可以将这个显示用于我们的回复。所以，让我们将这一部分代码提取出来，放到一个单独的组件中，这样我们就可以重用它。在`routes/thread`文件夹中创建一个名为`UserNameAndTime.tsx`的文件，并添加源代码。由于我们基本上是复制了`ThreadHeader`的代码，我就不在这里进行审查了。

1.  现在，我们可以通过更新我们的`ThreadHeader`组件代码来使用它。通过用以下代码替换`title`下的`h3`标签下的 JSX 来更新它。不要忘记添加导入语句：

```ts
      <UserNameAndTime userName={userName} lastModifiedOn={lastModifiedOn} />    
```

太棒了！现在，我们可以开始构建我们的`ThreadItems`组件。但这一次，我们会有点不同。在主题回复的情况下，可能会有多个回复。因此，这种情况在编程设计中通常需要使用一种称为工厂模式的东西。

所以，我们要做的实际上是两个组件。一个组件将充当工厂“构建”主题回复。另一个组件将定义回复实际上是什么样子的。因此，这两个组件一起可以产生任意数量的回复。请注意，我们没有使用工厂的正式设计模式，只是一个粗略的概念模型。让我们开始吧：

1.  首先，我们需要创建我们的`ThreadResponse`组件，它将定义我们的`ThreadItem` UI 和行为是什么样子的。在`routes`/`thread`文件夹中创建一个`ThreadResponse.tsx`文件，并添加相关的源代码。

首先，注意我们正在导入和重用我们之前创建的`RichEditor`和`UserNameAndTime`组件。你能想象如果我们没有将它们组件化，要重新创建它们需要多少工作吗？谢天谢地，我们把它们放到了它们自己的组件中！

接下来，我们有我们的`ThreadResponseProps`接口。请注意，我们所有的 props 都是可选的。这是为了准备当我们重构这个组件并使其能够创建新的回复条目时。

最后，我们有了返回的 JSX。这是一个非常简单的 UI - 我们只显示我们的`UserNameAndTime`和`RichEditor`。

1.  现在，让我们创建我们的`ThreadResponse`工厂。在同一个文件夹中创建一个名为`ThreadResponseBuilder.tsx`的文件，并添加相关的源代码。

首先，我们有`ThreadResponsesBuilderProps`接口。这个组件将接收一个包含`ThreadItems`列表的`props`。我们将不得不更新我们的`Thread`父组件，以便它将列表传递下去。

从*第 12 行*开始，因为我们的构建器正在生产多个回复，我们唯一的状态`responseElements`是一个用于包含它们的 JSX 元素。

接下来，我们使用`useEffect`来创建我们的回复元素列表。每个`ThreadResponse`实例都有一个唯一的键，这可以防止渲染问题。每当我们的`threadItems` props 改变时，我们将创建一个`ThreadResponses`的`ul`。

最后，我们返回我们的 JSX，这是一个`TheadResponse`元素的列表。

1.  我们快要完成了。让我们更新我们的`Thread.tsx`文件，使其现在使用我们的`ThreadResponsesBuilder`组件。请注意，样式已经在`App.css`和`Thread.css`文件中更新。

在`ThreadBody`下面的 JSX 中，添加以下代码中显示的突出显示的标签：

```ts
  return (
    <div className="screen-root-container">
      <div className="thread-nav-container">
        <Nav />
      </div>
      <div className="thread-content-container">
        <ThreadHeader
          userName={thread?.userName}
          lastModifiedOn={thread ? thread.lastModifiedOn : new Date()}
          title={thread?.title}
        />
        <ThreadCategory categoryName={thread?.category?.name} />
        <ThreadTitle title={thread?.title} />
        <ThreadBody body={thread?.body} />
        hr, to separate out the Thread post from any responses.Our screen should now look like this:
```

![图 12.16 - 一个主题及其回复](img/Figure_12.16_B15508.jpg)

图 12.16 - 一个主题及其回复

我们现在几乎有了一个完整的`Thread`发布和查看 UI。但是，我们还没有完成。我们仍然需要构建我们的点查看器，并启用`Thread`和`ThreadItem`的发布。我们将在这里构建点查看组件，但是让我们把发布能力留到以后的章节，当我们的后端准备好时再将其联系在一起。此外，当我们的后端准备好时，为什么我们在这里做某些事情将变得更加清晰。

对于我们的`categorythreads`路由，你会发现我们有一个垂直条显示我们的点赞和回复计数。如果你看一下我们是如何创建这个部分的，你会发现我们将那段代码放入了一个名为`getPointsNonMobile`的函数中。我们可以将这个功能提取到自己的 React 组件中。显然，这将允许我们在`ThreadCard`组件和`Thread`组件以及以后可能需要的任何其他地方使用它。让我们开始吧：

1.  创建一个名为`ThreadPointsBar.tsx`的新文件，并将其放在`components`文件夹的根目录中。我们将从`ThreadCard`组件中获取`getPointsNonMobile`函数，并将其添加到这个新组件中。

第 6 行，我们使用`ThreadPointsBarProps`作为我们的 props 类型。你可能会想为什么我不直接传入整个 Thread 对象。只添加需要的成员数据可以更好地保持关注点的分离。如果我们传递整个 Thread，不仅会告诉`ThreadPointsBar`我们正在处理哪种模型类型，还会给它一些实际上并没有使用或需要的信息。

接下来，返回的 JSX 基本上与原始函数相同，因为它执行相同的操作。现在，尝试更新`ThreadCard`组件，以便删除`getPointsNonMobile`函数。我们将在其位置添加我们的新`ThreadPointsBar`组件。请注意，我们稍微更新了`ThreadCard.css`文件，因此您应该刷新它。屏幕应该与我们的原始屏幕相同，因为我们只是移动了一些东西。

1.  现在，让我们将新的`ThreadPointsBar`组件添加到我们的`Thread`路由组件中。JSX 的变化很小但很重要，所以让我们在这里进行解释，然后再看看我们更新后的`Thread.css`文件：

```ts
  return (
    <div className="screen-root-container">
      <div className="thread-nav-container">
        <Nav />
      </div>
      <div className="thread-content-container">
        <div className="thread-content-post-container">
```

在这里，我们改变了一些元素的顺序。现在，主要的 Thread 帖子相关元素位于`thread-content-post-container`类的`div`下面：

```ts
          <ThreadHeader
            userName={thread?.userName}
            lastModifiedOn={thread ? thread.lastModifiedOn : new Date()}
            title={thread?.title}
          />
          <ThreadCategory categoryName={thread?.category?.name} />
          <ThreadTitle title={thread?.title} />
          <ThreadBody body={thread?.body} />
        </div>
        <div className="thread-content-points-container">
```

在这里，我们有一个全新的带有`thread-content-points-container`类的`div`，其中包含我们的新`ThreadPointsBar`组件：

```ts
          <thread-content-response-container:

```

</div>

);

};

```ts

Let's look at our refreshed CSS `Thread.css` file to see what's going on.Near the top of the file, I've explicitly given a definition for `grid-template-rows`. The Grid now has two rows: one for posts and one for responses. Posts take up one part of available space, but responses can take up as much space as needed, which is what `auto` means, since it could have 0 or more responses.We now have this new class, `thread-content-points-container`. We need this to change the layout of our `ThreadPointsBar`, which is now different from the main screen. Notice that it puts itself into the second column start index and first Grid row. The `> div` element on the second definition means to give the `div` elements inside `ThreadPointsBar` and `threadcard-points` a specific height of all available.Now, our main Thread post items, such as `ThreadTitle` and `ThreadBody`, live inside this `thread-content-post-container`.Our responses – mainly `ThreadResponsesBuilder` – live inside this `thread-content-response-container`. Notice that `grid-row` is set to 2.After the `thread-content-response-container` class, you'll notice that all the section-related classes no longer need references to any Grid column or Grid since they all live inside `thread-content-post-container`.
```

1.  现在，我们想为我们的回复给出点数总计。但是，因为我们可能会有很多回复，为每个回复显示 20 或 30 个小的垂直点条可能看起来不太好。为了使事情看起来更整洁，让我们把这些点放在与我们的`userName`和`createdOn`日期相同的行上。幸运的是，我们已经在`ThreadCard`组件中使用`getPoints`函数创建了大部分显示这些点的代码。所以，让我们也将其转换为一个组件。

创建一个名为`ThreadPointsInline.tsx`的新文件，并将相关源代码添加到其中。我们基本上只是将我们的`getPoints`代码复制粘贴到这里，所以没有太多解释。但是，请注意我们从`ThreadPointsBar`组件中重用了`ThreadPointsBarProps`接口。因此，我们需要将这种类型导出。

我假设你知道如何更新`ThreadCard.tsx`文件，因为我们之前使用`ThreadPointsBar`做过这个。现在，让我们更新`ThreadResponse.tsx`文件，以便使用我们的新`ThreadPointsInline`组件。尝试自己做这个；只有在卡住时才查看代码。现在我们有：

![图 12.17 - 显示主题点数](img/Figure_12.17_B15508.jpg)

图 12.17 - 显示主题点数

正如你所看到的，我们的两个点系统都可以看到。现在，我们需要实现一个最后的小技巧，以便在移动设备上正确显示这个屏幕。

1.  打开`Thread.css`文件，确保它包含与源代码相同的媒体查询。

现在，打开`Thread`组件的代码，这样我们就可以浏览它。

在*第 32 行*，你会看到我们的线程帖子相关的项目都位于`thread-content-container`内。通过媒体查询设置了 CSS 类，以便它只有一个`ThreadPointsBar`组件，从该区域，我们不会得到一个空白空间，因为之前有两列。

接下来，我们可以看到我们的`ThreadPointsBar`实际上位于`thread-content-points-container`内。在媒体查询中，我们使该元素不可见。这仍然是有效的，因为你可能还记得，内部，`ThreadPointsBar`正在使用我们的`useWindowDimensions` Hook 来确定它是否应该渲染自身。它不会为移动设备执行此操作。

太棒了！现在让我们在移动设备上查看我们的屏幕：

![图 12.18 – 线程屏幕的移动视图](img/Figure_12.18_B15508.jpg)

图 12.18 – 线程屏幕的移动视图

太棒了！现在，我们有一个代码库和两个屏幕。

在本章的最后一个项目中，我们将构建`UserProfile`屏幕。我们想在这个屏幕上做一些事情：

+   允许用户重置他们的密码。

+   显示所有用户生成的线程帖子。

+   显示所有用户生成的回复（`ThreadItems`）。

让我们开始吧：

1.  我们实际上要做的第一件事是对`SideBarMenus`组件进行更改。我们需要移出`useEffect`调用，以便将我们的用户发送到 Redux，然后发送到`Login`组件。我们这样做是为了当用户成功登录时，新的用户对象将被发送到 Redux。到目前为止，你应该已经习惯了进行这种改变。所以，继续将这段代码从`SideBarMenu`中移除，并添加到`Login`中。

提示

确保将代码放入`Login`时，将`dispatch`的名称更改为其他名称，因为`Login`组件中已经有一个`dispatch`。

1.  这个新屏幕将包括密码重置功能，但你可能还记得我们已经有很多代码来进行密码确认在我们的`Register`组件中。让我们尝试将该代码提取到自己的组件中，以便我们可以在`Register`组件和我们的新`UserProfile`组件中重用它。

在`components/auth/common`文件夹内创建一个名为`PasswordComparison.tsx`的文件。将相关的源代码添加到其中。

这是一个相当简单的复制粘贴，但有一些要注意的地方。请注意，此组件不使用`userReducer`，而是使用其值的 props。特别要注意的是其中一个是`dispatch`函数。`dispatch`调用属于父级。其他一切基本上都是复制和粘贴。

尝试自己从原始的`Register`组件中删除这段代码。确保删除所有不必要的导入。

1.  现在，让我们在`routes`文件夹内创建一个新的`userProfile`文件夹，这样我们就可以创建我们的新的`UserProfile.tsx`文件，并将相关的源代码添加到其中。

从*第 14 行*开始，我们使用我们的`userReducer`，因为我们需要一些它的属性，比如`userName`。我们还获取 Redux 用户 reducer 并为用户的 Threads 和`ThreadItems`设置一些本地状态。

在*第 28 行*，`useEffect`函数使用`DataService`的`getUserThreads`函数，获取用户的 Threads。我们不需要另一个调用来获取`ThreadItems`，因为 Threads 包含相关的`ThreadItems`。但是，我更新了`ThreadItem`类，以便它包括其父`ThreadId`。查看这些文件以获取该代码。

接下来，从*第 38 行*开始，我们将查询结果中的每个线程映射到一个`li`。我们还将所有`ThreadItems`添加到一个数组中，以便以后使用。

然后，从*第 53 行*开始，我们将我们的`ThreadItems`映射到一组`li`。

在*第 77 行*，我们使用了之前创建的`PasswordComparison`组件。

在*第 82 行*，请注意我们的按钮使用了`isSubmitDisabled`。你能猜到这种禁用是如何工作的吗，即使`UserProfile`并不包含任何改变它的代码？没错——`PasswordComparison`在内部使用我们 UserProfile 的`dispatch`函数来做到这一点。

最后，我们的 Threads 和`ThreadItems`是从我们的本地状态对象渲染出来的。

1.  对于最后的更改，让我们更新我们的`App.tsx`文件，以便包括我们的新路由`UserProfile`。请注意，我们还需要临时添加`userName` Redux 调用，直到`Login.tsx`中的相同调用完全工作（一旦我们的后端准备好，我们将在`Login.tsx`中完成调用）。这是因为当我们加载我们的`UserProfile`时，不能保证用户已经加载了他们的`Login`屏幕。但是，我们知道如果他们已经加载了应用程序中的任何屏幕，他们必须已经加载了`App.tsx`组件。从源代码中更新`App.tsx`。

首先，我们有一个`useEffect`，其中有一个硬编码的`userName`被发送到 Redux 存储。同样，这只是临时的，直到我们的后端准备好为止。

在*第 26 行*，`renderUserProfile`是返回我们的`UserProfile`组件的函数。然后在*第 33 行*将该函数用作新路由的目的地；即`"/userprofile/:id"`。

我们还需要做一个微小的改变。在我们的`SideBarMenus`组件中，让我们更新我们的`userName`标签，使其成为指向我们新的`UserProfile`屏幕的链接。您可以在`SideBarMenus.tsx`文件中找到这个 JSX：

```ts
<span className="menu-name">{user?.userName}</span>
```

然后，用这个替换它：

```ts
<span className="menu-name">
            <Link to={`/userprofile/${user?.id}`}>{user?.userName}</Link>
          </span>
```

现在，如果您运行应用程序，您将看到以下内容：

![图 12.19 – 用户资料屏幕](img/Figure_12.19_B15508.jpg)

图 12.19 – 用户资料屏幕

如果您点击任何一个 Thread 链接，您会发现它们会带我们到 thread 路由。

太棒了！在这一章中，我们经历了大量的 React 代码。我们学习了布局、文件夹结构、组件创建、代码重用、代码重构、样式等。特别是代码重构可能非常耗时甚至令人紧张。然而，现实是，大多数时候，我们不会写新代码，而是重构现有代码。因此，这是建立我们技能的一个很好的方式。

在接下来的几章中，我们将构建我们的后端，并将其与我们的客户端连接起来。现在你应该感到非常自信——你已经在这一复杂章节中付出了巨大的努力。

# 总结

在这一章中，我们开始了构建全栈应用程序的旅程，首先创建了 React 客户端。我们使用 Hooks 创建了组件，实现了组件层次结构，并使用 CSS Grid 设计了布局。然后我们重构了大量的代码，并尽量重用了尽可能多的代码。尽管我们还没有完成，但我们已经构建了最终应用程序的一个重要部分。

在下一章中，我们将学习关于后端服务器上的会话状态，会话状态是什么，如何使用它，以及创建和管理会话数据的最流行工具：Redis。
