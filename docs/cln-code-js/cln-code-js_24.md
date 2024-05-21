# 第十九章：案例研究

在这本书中，我们已经讨论了一系列原则，几乎涵盖了 JavaScript 语言的方方面面，并且长篇大论地讨论了什么是*干净的代码*。这一切都是为了最终能够撰写美丽而干净的 JavaScript 代码，解决真实而具有挑战性的问题领域。然而，追求干净的代码永远不会完成；新的挑战总会出现，让我们以新的、颠覆性的方式思考我们所写的代码。

在本章中，我们将走过 JavaScript 中创建新功能的过程。这将涉及客户端和服务器端的部分，并且将迫使我们应用我们在整本书中积累的许多原则和知识。我们将要解决的具体问题是从我负责的一个真实项目中改编的，虽然我们不会涉及其实施的每一个细节，但我们将涵盖最重要的部分。完成的项目可以在以下链接的 GitHub 上查看：[`github.com/PacktPublishing/Clean-Code-in-JavaScript`](https://github.com/PacktPublishing/Clean-Code-in-JavaScript)。

在本章中，我们将涵盖以下主题：

+   **问题**：我们将定义并探讨问题

+   **设计**：我们将设计一个解决问题的用户体验和架构

+   **实施**：我们将实施我们的设计

# 问题

我们将要解决的问题涉及到我们网页应用程序用户体验的核心部分。我们将要处理的网页应用程序是一个大型植物数据库的前端，其中有成千上万种不同的植物。除了其他功能外，它允许用户找到特定的植物并将它们添加到集合中，以便他们可以跟踪他们的异国温室和植物研究清单。如下图所示：

![](img/e254a325-cd9f-4839-bbfe-f4930bc8f91c.png)

目前，当用户希望找到一种植物时，他们必须使用一个涉及将植物名称（*全拉丁名称*）输入到文本字段中，点击搜索，并收到一组结果的搜索设施，如下截图所示：

![](img/e74e911d-7b93-4c8b-9eb0-e8d3de98270d.png)

对于我们的案例研究，植物名称只存在于它们的全拉丁名称中，其中包括一个科（例如，Acanthaceae）、一个属（例如，Acanthus）和一个种（例如，Carduaceus）。这突显了满足复杂问题领域所涉及的挑战。

这样做已经足够好了，但在一些用户焦点小组和在线反馈之后，我们决定需要为用户提供更好的用户体验，让他们更快地找到他们感兴趣的植物。提出的具体问题如下：

+   有时候我觉得查找物种很麻烦而且慢。我希望它更即时和灵活，这样我就不必一直回去修改我的查询，特别是如果我拼错了的话。

+   通常，当我知道植物的种或属的名称时，我仍然会稍微出错，没有得到结果。然后我就不得不回去调整我的拼写或在其他地方进行搜索。

+   我希望我在输入时能看到种和属的出现。这样我就可以更快地找到适当的植物，不浪费时间。

这里提出了一些可用性问题。我们可以将它们归纳为以下三个主题：

+   **性能**：当前的搜索功能使用起来很慢且笨拙

+   **错误** **更正**：不得不纠正打字错误的过程令人讨厌和繁琐

+   **反馈**：在输入时获得有关现有*属*/*种*的反馈将是有用的

任务现在变得更加清晰。我们需要改进用户体验，使用户能够以更快的方式查询植物数据库，提供更即时的反馈，并让他们在途中防止或纠正输入错误。

# 设计

经过一些头脑风暴，我们决定可以以相当传统的方式解决我们的问题；我们可以简单地将输入字段转换为提供自动建议下拉框的字段。这是一个模拟：

![](img/8e2c2309-6bc8-406d-86fd-f1bfa89b9a68.png)

这个自动建议下拉框将具有以下特点：

+   当输入一个术语时，它将显示一个按优先级排序的植物名称列表，这些名称包含该术语作为前缀，例如，搜索`car`将产生结果`carnea`，但不会产生`encarea`

+   当通过点击、箭头（上/下）或*Enter*键选择一个术语时，它将运行指定的函数（以后可能用于将选定的项目添加到用户的集合中）

+   当找不到匹配的植物名称时，用户将收到通知，例如“没有该名称的植物存在”

这些是我们组件的核心行为，为了实现它们，我们需要考虑客户端和服务器端的部分。我们的客户端将不得不向用户呈现`<input>`，并且当他们输入时，它将不得不动态调整建议列表。服务器将不得不为每个潜在的查询提供给客户端一个建议列表，同时考虑到结果需要快速交付。任何显著的延迟都将严重降低我们试图创建的用户体验的好处。

# 实施

恰好这个新的植物选择组件将是我们网页应用中第一个重要的客户端代码片段，因此，重要的是要注意我们的设计决策不仅会影响到这个特定的组件，还会影响到我们将来考虑构建的任何其他组件。

为了帮助我们实施，并考虑到将来可能的其他潜在添加，我们决定采用 JavaScript 库来辅助 DOM 的操作，并采用支持工具集，使我们能够迅速高质量地工作。在这种情况下，我们决定在客户端使用 React，使用 webpack 和 Babel 进行编译和打包，并在服务器端使用 Express 进行 HTTP 路由。

# 植物选择应用

正如讨论的那样，我们决定将*植物选择*功能构建为一个独立的应用，包括客户端（React 组件）和服务器（植物数据 API）。这种隔离程度使我们能够纯粹地专注于选择植物的问题，但这并不意味着这不能在以后集成到更大的代码库中。

我们的目录结构大致如下：

```js
EveryPlantSelectionApp/
├── server/
│   ├── package.json
|   ├── babel.config.js
│   ├── index.js
|   └── plantData/
│       ├── plantData.js
│       ├── plantData.test.js
|       └── data.json
└── client/
    ├── package.json
    ├── webpack.config.js
    ├── babel.config.js
    ├── app/
    |   ├── index.jsx
    |   └── components/
    |       └── PlantSelectionInput/
    └── dist/
        ├── main.js (bundling target)
        └── index.html
```

除了为我们（程序员）减少复杂性之外，服务器和客户端的分离意味着服务器端应用（即植物选择 API）可以在必要时在自己独立的服务器上运行，而客户端可以从 CDN 静态提供，只需要服务器端的地址即可访问其 REST API。

# 创建 REST API

`EveryPlantSelectionApp`的服务器负责检索植物名称（植物*科*、*属*和*种*），并通过简单的 REST API 使它们可用于我们的客户端代码。为此，我们可以使用`express` Node.js 库，它使我们能够将 HTTP 请求路由到特定的函数，轻松地向客户端提供 JSON。

这是我们服务器实现的基本框架：

```js
import express from 'express';

const app = express();
const port = process.env.PORT || 3000;

app.get('/plants/:query', (req, res) => {
  req.params.query; // => The query
  res.json({
    fakeData: 'We can later place some real data here...'
  });
});

app.listen(
  port,
  () => console.log(`App listening on port ${port}!`)
);
```

正如您所看到的，我们只实现了一个路由（`/plants/:query`）。每当用户在`<input/>`中输入部分植物名称时，客户端将请求这个路由，因此用户输入`Carduaceus`可能会产生以下一系列请求到服务器：

```js
GET /plants/c
GET /plants/ca
GET /plants/car
GET /plants/card
GET /plants/cardu
GET /plants/cardua
...
```

你可以想象这可能导致更多昂贵且可能冗余的请求，特别是如果用户输入速度很快。有可能用户在任何之前的请求完成之前就输入了`cardua`。因此，当我们开始实现客户端时，我们需要使用某种请求节流（或请求防抖）来确保我们只发出合理数量的请求。

**请求节流**是通过只允许在指定时间间隔内执行新请求来减少总请求量的行为，这意味着在五秒内跨度为 100 个请求，节流到一个间隔为一秒，将只产生五个请求。**请求防抖**类似，不过它不是在每个间隔上执行单个请求，而是在实际请求之前等待一定的时间，以便停止产生请求。因此，在五秒内进行 100 个请求，通过五秒的防抖，将在五秒标记处只产生一个最终请求。

为了实现`/plants/`端点，我们需要考虑通过超过*300,000*不同植物物种的名称进行匹配的最佳搜索方式。为了实现这一点，我们将使用一种特殊的内存数据结构，称为**trie**。这也被称为*前缀树*，在需要自动建议或自动完成的情况下非常常见。

Trie 是一种类似树状结构的数据结构，它将相邻的字母块存储为一系列通过分支连接的节点。它比描述起来更容易可视化，所以让我们想象一下，我们需要基于以下数据创建一个 trie：

```js
['APPLE', 'ACORN', 'APP', 'APPLICATION']
```

利用这些数据，生成的 trie 可能看起来像这样：

![](img/0fd9a5f9-ff9b-4291-b9b3-10a7c13975d9.png)

如您所见，我们的四个单词的数据集已被表示为类似树状结构的结构，其中第一个共同的字母`"A"`作为根。 `"CORN"`后缀从这里分支出来。此外， `"PP"`分支（形成`"APP"`）分支出来，然后最后的`"P"`分支到`"L"`，然后分支到`"E"`（形成`"APPLE"`）和`"ICATION"`（形成`"APPLICATION"`）。

这可能看起来很复杂，但是鉴于这种 trie 结构，我们可以根据用户输入的初始前缀（如`"APPL"`）轻松地通过树的节点找到所有匹配的单词（`"APPLE"`和`"APPLICATION"`）。这比任何线性搜索算法都要高效得多。对于我们的目的，给定植物名称的前缀，我们希望能够高效地显示前缀可能导致的每个植物名称。

我们的特定数据集将包括超过 300,000 种不同的植物物种，但在本案例研究中，我们将只使用`Acanthaceae`科的物种，大约有 8,000 种。这些可以以 JSON 的形式使用，如下所示：

```js
[
  { id: 105,
    family: 'Acanthaceae',
    genus: 'Andrographis',
    species: 'alata' },
  { id: 106,
    family: 'Acanthaceae',
    genus: 'Justicia',
    species: 'alata' },
  { id: 107,
    family: 'Acanthaceae',
    genus: 'Pararuellia',
    species: 'alata' },
  { id: 108,
    family: 'Acanthaceae',
    genus: 'Thunbergia',
    species: 'alata' },
  // ...
]
```

我们将把这些数据输入到一个名为**trie-search**的第三方 trie 实现中。选择这个包是因为它满足我们的要求，并且似乎是一个经过充分测试和维护良好的库。

为了使 trie 按我们的期望运行，我们需要将每种植物的*科*、*属*和*种*连接成一个字符串。这使得 trie 可以包括完全合格的植物名称（例如`"Acanthaceae Pararuellia alata"`）和分割名称（`["Acanthaceae", "Pararuellia", "alata"]`）。*分割*名称是由我们使用的 trie 实现自动生成的（意思是它通过正则表达式`/\s/g`在空格上分割字符串）：

```js
const trie = new TrieSearch(['name'], {
  ignoreCase: true // Make it case-insensitive
});

trie.addAll(
  data.map(({ family, genus, species, id }) => {
    return { name: family + ' ' + genus + ' ' + species, id };
  })
);
```

前面的代码将我们的数据集输入到 trie 中。之后，可以通过简单地将前缀字符串传递给其`get(...)`方法来查询它：

```js
trie.get('laxi');
```

这样的查询（例如前缀`laxi`）将从我们的数据集中返回以下内容：

```js
[
  { id: 203,
    name: 'Acanthaceae Acanthopale laxiflora' },
  { id: 809,
    name: 'Acanthaceae Andrographis laxiflora' },
  { id: 390,
    name: 'Acanthaceae Isoglossa laxiflora' },
  //... (many more)
]
```

关于我们的 REST 端点`/photos/:query`，它所需要做的就是返回一个 JSON 有效负载，其中包含我们从`trie.get(query)`获取的内容：

```js
app.get('/plants/:query', (req, res) => {
  const queryString = req.params.query;
  if (queryString.length < 3) {
    return res.json([]);
  }
  res.json(
    trie.get(queryString)
  );
});
```

为了更好地分离我们的关注点，并确保我们没有混合太多不同的抽象层（可能违反了迪米特法则），我们可以将我们的 trie 数据结构和植物数据抽象到自己的模块中。我们可以称之为`plantData`，以传达它封装和提供对植物数据的访问的事实。它的工作方式，恰好是通过内存 trie 数据结构，不需要为其使用者所知：

```js
// server/plantData.js

import TrieSearch from 'trie-search';
import plantData from './data.json';

const MIN_QUERY_LENGTH = 3;

const trie = new TrieSearch(['fullyQualifiedName'], {
  ignoreCase: true
});

trie.addAll(
  plantData.map(plant => {
    return {
      ...plant,
      fullyQualifiedName:
        `${plant.family} ${plant.genus} ${plant.species}`
    };
  })
);

export default {
  query(partialString) {
    if (partialString.length < MIN_QUERY_LENGTH) {
      return [];
    }
    return trie.get(partialString);
  }
};
```

如您所见，此模块返回一个接口，提供一个`query()`方法，我们的主 HTTP 路由代码可以利用它来为`/plants/:query`提供 JSON 结果：

```js
//...
import plantData from './plantData';
//...
app.get('/plants/:query', (req, res) => {
  const query = req.params.query;
  res.json( plantData.query(partial) );
});
```

因为我们已经隔离和包含了植物查询功能，现在更容易对其进行断言。编写一些针对`plantData`抽象的测试将使我们对我们的 HTTP 层使用可靠的抽象具有高度的信心，最大程度地减少了 HTTP 层内部可能出现的潜在错误。

此时，由于这是我们为项目编写的第一组测试，我们将安装 Jest（`npm install jest --save-dev`）。有许多可用的测试框架，风格各异，但对于我们的目的来说，Jest 是合适的。

我们可以在与之直观地位于一起并命名为`plantData.test.js`的文件中为我们的`plantData`模块编写测试。

```js
import plantData from './plantData';

describe('plantData', () => {

  describe('Family+Genus name search (Acanthaceae Thunbergia)', () => {
    it('Returns plants with family and genus of "Acanthaceae Thunbergia"', () =>{
      const results = plantData.query('Acanthaceae Thunbergia');
      expect(results.length).toBeGreaterThan(0);
      expect(
        results.filter(plant =>
          plant.family === 'Acanthaceae' &&
          plant.genus === 'Thunbergia'
        )
      ).toHaveLength(results.length);
    });
  });

});
```

`plantData.test.js`中有许多测试未在此处包含，以保持简洁；但是，您可以在 GitHub 存储库中查看它们：[`github.com/PacktPublishing/Clean-Code-in-JavaScript`](https://github.com/PacktPublishing/Clean-Code-in-JavaScript)。

如您所见，此测试断言`Acanthaceae Thunbergia`查询是否直观地返回包含这些术语的完全限定名称的植物。在我们的数据集中，这将仅包括具有`Acanthaceae`家族和`Thunbergia`属的植物，因此我们可以简单地确认结果是否符合预期。我们还可以检查部分搜索，例如`Acantu Thun`，是否也直观地返回任何以`Acantu`或`Thun`开头的*家族*、*属*或*种*名称的植物：

```js
describe('Partial family & genus name search (Acantu Thun)', () => {
  it('Returns plants that have a fully-qualified name containing both "Acantu" and "Thunbe"', () => {
    const results = plantData.query('Acant Thun');
    expect(results.length).toBeGreaterThan(0);
    expect(
      results.filter(plant =>
        /\bAcant/i.test(plant.fullyQualifiedName) &&
        /\bThun/i.test(plant.fullyQualifiedName)
      )
    ).toHaveLength(results.length);
  });
});
```

我们通过断言每个返回结果的`fullyQualifiedName`是否与正则表达式`/\bAcant/i`和`/\bThun/i`匹配来确认我们的期望。`/i`表达式表示区分大小写。这里的`\b`表达式表示单词边界，以便我们可以确保`Acant`和`Thun`子字符串出现在单词的开头，并且不嵌入在单词中。例如，想象一个名为`Luathunder`的植物。我们不希望我们的自动建议机制匹配这样的实例。我们只希望它匹配前缀，因为用户将输入植物*家族*、*属*或*种*到`<input />`中（从每个单词的开头）。

现在我们有了经过充分测试和隔离的服务器端架构，我们可以开始转向客户端，我们将在用户输入时呈现由`/plants/:query`提供的植物名称。

# 创建客户端构建过程

我们在客户端的第一步是引入*React*和一个支持开发的工具集。在以前的网页开发时代，可以在没有复杂工具和构建步骤的情况下构建东西，这在某种程度上仍然是可能的。在过去，我们可以简单地创建一个 HTML 页面，内联包含任何第三方依赖项，然后开始编写我们的 JavaScript，而不必担心其他任何事情：

```js
<body>
  ... Content
  <script src="//example.org/libraryFoo.js"></script>
  <script src="//example.org/libraryBaz.js"></script>
  <script>
    // Our JavaScript code...
  </script>
</body>
```

从技术上讲，我们仍然可以这样做。即使使用现代前端框架如 React，我们也可以选择将其作为`<script>`依赖项包含在内，然后内联编写原始 JavaScript。然而，通过这样做，我们将无法获得以下优势：

+   **更新的 JavaScript 语法**（ES 2019 及更高版本）：能够使用现代 JavaScript 语法，并将其编译为在所有环境/浏览器中安全使用的 JavaScript。

+   **自定义语法和语言扩展**：能够使用语言扩展（如 JSX 或 FlowJS）或编译为 JavaScript 的其他语言（如 TypeScript 或 CoffeeScript）。

+   **依赖树管理**：能够轻松指定您的依赖关系（例如使用`import`语句），并将这些自动协调和合并成一个捆绑，而无需手动操作`<script>`标签和版本化噩梦。

+   **性能改进**：智能编译和打包可以通过减少 JavaScript 和 CSS 的整体占用空间，从而提供有意义的 HTTP 和运行时性能增益。

+   **检查器和分析器**：能够在您的 JavaScript（以及您的 CSS 和 HTML）上使用检查器和其他形式的分析，让我们深入了解代码质量和潜在的错误。

从根本上说，现在 Web 应用的性质更加复杂，特别是在前端。为了创建一个自动建议组件，我们需要确保我们有一套良好的工具和构建步骤基础，以便持续开发可以无缝和简单。在设置这些东西时可能会带来一些麻烦，但从长远来看是值得的。

为了编译我们的 JavaScript（包括 React 的 JSX），我们将使用*Babel*，它可以将我们的 JavaScript 转换为广泛支持的常规 JavaScript 语法。要在`EveryPlantSelectionApp/client`中添加 Babel 作为依赖项，我们可以使用`npm`来安装它及其各种预设配置：

```js
# Install babel's core dependencies:
npm install --save-dev @babel/core @babel/cli

# Install some smart presets for Babel, allowing us to not have
# to worry about which specific JS syntax we're using:
npm install --save-dev @babel/preset-env

# Install a smart preset for React (i.e. JSX) usage:
npm install --save-dev @babel/preset-react
```

Babel 将管理我们的 JavaScript 的编译，使其成为广泛支持的语法。但是，为了使这些文件准备好交付给浏览器，我们需要将它们捆绑成一个单一的文件，可以像这样在我们的 HTML 中单独交付：

```js
<script src="./ourBundledJavaScript.js"></script>
```

为了实现这一点，我们需要使用一个打包工具，比如 webpack。Webpack 可以为我们执行以下任务：

+   它可以通过 Babel 编译 JavaScript

+   然后它可以协调每个模块，包括任何依赖项

+   它可以生成一个包含所有依赖关系的单一捆绑的 JavaScript 文件

为了使用 webpack，我们需要安装几个相关的依赖项：

```js
# Install Webpack and its CLI:
npm install --save-dev webpack webpack-cli

# Install Webpack's development server, which enables us to more easily
# develop without having to keep re-running the build process:
npm install --save-dev webpack-dev-server

# Install a couple of helpful packages that make it easier for
# Webpack to make use of Babel:
npm install --save-dev babel-loader babel-preset-react
```

Webpack 还需要自己的配置文件，名为`webpack.config.js`。在这个文件中，我们必须告诉它如何打包我们的代码，以及我们希望打包的代码输出到项目中的哪个位置：

```js
const path = require('path');

module.exports = {
  entry: './app/index.jsx',
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/react']
          }
        }
      }
    ]
  },
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
    compress: true,
    port: 9000
  },
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist'),
  }
};
```

这个配置基本上告诉 webpack 以下内容：

+   请从`EveryPlantSelectionApp/client/app/index.jsx`开始

+   请使用 Babel 来编译此模块及其以`.jsx`或`.js`结尾的所有依赖项

+   请将编译和捆绑文件输出到`EveryPlantSelectionApp/client/dist/`

最后，我们需要安装 React，这样我们就可以准备创建我们的植物选择组件：

```js
npm install --save react react-dom
```

看起来好像这样做很多工作只是为了渲染一个基本的 UI 组件，但实际上我们已经创建了一个基础，可以容纳许多新功能，并且我们已经创建了一个构建流水线，可以更容易地将我们的开发代码库交付到生产环境。

# 创建组件

我们组件的工作是显示一个增强的`<input>`元素，当聚焦时，会根据用户输入的内容呈现一个下拉式的可用选项列表，用户可以从中选择。

作为一个原始的大纲，我们可以想象组件包含`<div>`，用户可以在其中输入的`<input>`，以及显示建议的`<ol>`：

```js
const PlantSelectionInput = () => {
  return (
    <div className="PlantSelectionInput">
      <input
        autoComplete="off"
        aria-autocomplete="inline"
        role="combobox" />
      <ol>
        <li>A plant name...</li>
        <li>A plant name...</li>
        <li>A plant name...</li>
      </ol>
    </div>
  );
};
```

`<input>`上的`role`和`aria-autocomplete`属性用于指示浏览器（以及任何屏幕阅读器），用户在键入时将提供一组预定义的选择。这对于可访问性至关重要。`autoComplete`属性用于简单地启用或禁用浏览器的默认自动完成行为。在我们的情况下，我们希望禁用它，因为我们提供了自己的自定义自动完成/建议功能。

我们只希望在`<input>`聚焦时显示`<ol>`。为了实现这一点，我们需要绑定到`<input>`的焦点和失焦事件，然后创建一个可以跟踪我们是否应该考虑组件打开或关闭的独立状态。我们可以称这个状态为`isOpen`，并根据它的布尔值有条件地渲染或不渲染`<ol>`：

```js
const PlantSelectionInput = () => {
  const [isOpen, setIsOpen] = useState(false);
  return (
    <div className="PlantSelectionInput">
      <input
        onFocus={() => setIsOpen(true)}
        onBlur={() => setIsOpen(false)}
        autoComplete="off"
        aria-autocomplete="inline"
        role="combobox" />
      {
        isOpen &&
          <ol>
             <li>A plant name...</li>
             <li>A plant name...</li>
             <li>A plant name...</li>
          </ol>
      }
    </div>
  );
};
```

React 有自己关于状态管理的约定，如果你之前没有接触过，可能看起来很奇怪。`const [foo, setFoo] = useState(null)`代码创建了一个状态（称为`foo`），我们可以根据某些事件来改变它。每当这个状态改变时，React 就会知道触发相关组件的重新渲染。回到第十二章*，真实世界的挑战*，看看*DOM 绑定和协调*部分，以便重新了解这个主题。

接下来，我们需要绑定到`<input>`的`change`事件，以便获取用户输入的内容，并触发对我们的`/plants/:query`端点的请求，以便确定向用户显示什么建议。然而，首先，我们希望创建一个机制，通过它可以发出请求。在 React 世界中，它建议将这种功能建模为自己的*Hook*。记住，按照约定，Hook 通常以*use*动词为前缀，我们可以将其称为`usePlantLike`。作为它的唯一参数，它可以接受一个`query`字段（用户键入的字符串），它可以返回一个带有`loading`字段（指示当前加载状态）和`plants`字段（包含建议）的对象：

```js
// Example of calling usePlantsLike:
const {loading, plants} = usePlantsLike('Acantha');
```

我们的`usePlantsLike`的实现非常简单：

```js
// usePlantLike.js

import {useState, useEffect} from 'react';

export default (query) => {
  const [loading, setLoading] = useState(false);
  const [plants, setPlants] = useState([]);

  useEffect(() => {
    setLoading(true);
    fetch(`/plants/${query}`)
      .then(response => response.json())
      .then(data => {
        setLoading(false);
        setPlants(data);
      });
  }, [query]);

  return { loading, plants };
};
```

在这里，我们使用另一种*React*状态管理模式`useEffect()`，以便在`query`参数发生变化时运行特定的函数。因此，如果`usePlantLike`接收到一个新的`query`参数，例如`Acantha`，那么加载状态将被设置为`true`，并且将启动一个新的`fetch()`，其结果将填充`plants`状态。这可能很难理解，但就案例研究而言，我们真正需要理解的是`usePlantsLike`这种抽象封装了向服务器发出`/plants/:query`请求的复杂性。

**将渲染逻辑与数据逻辑分开是明智的**。这样做可以确保良好的抽象层次和关注点分离，并将每个模块作为*单一责任*的领域。传统的 MVC 和 MVVM 框架有助于强制进行这种分离，而更现代的渲染库（如 React）则给了你更多的选择。因此，在这里，我们选择将数据和服务器通信逻辑隔离在一个 React Hook 中，然后由我们的组件使用。

现在，每当用户在`<input>`中键入内容时，我们可以使用我们的新 React Hook。为了做到这一点，我们可以绑定到它的`change`事件，每次触发时，获取它的`value`，然后将其作为`query`参数传递给`usePlantsLike`，以便为用户派生一组新的建议。然后可以在我们的`<ol>`容器中呈现这些建议：

```js
const PlantSelectionInput = ({ isInitiallyOpen, value }) => {

  const inputRef = useRef();
  const [isOpen, setIsOpen] = useState(isInitiallyOpen || false);
  const [query, setQuery] = useState(value);
  const {loading, plants} = usePlantsLike(query);

  return (
    <div className="PlantSelectionInput">
      <input
        ref={inputRef}
        onFocus={() => setIsOpen(true)}
        onBlur={() => setIsOpen(false)}
        onChange={() => setQuery(inputRef.current.value)}
        autoComplete="off"
        aria-autocomplete="inline"
        role="combobox"
        value={value} />
      {
        isOpen &&
          <ol>{
            plants.map(plant =>
              <li key={plant.id}>{plant.fullyQualifiedName}</li>
            )
          }</ol>
      }
    </div>
  );
};
```

在这里，我们添加了一个新的状态`query`，通过`<input>`的`onChange`处理程序设置它。然后，这个`query`变化将导致`usePlantsLike`从服务器发出一个新的请求，并用多个`<li>`元素填充`<ol>`，每个元素代表一个单独的植物名称建议。

有了这一点，我们已经完成了组件的基本实现。为了使用它，我们可以在我们的`client/index.jsx`入口点中呈现它：

```js
import ReactDOM from 'react-dom';
import React from 'react';
import PlantSelectionInput from './components/PlantSelectionInput';

ReactDOM.render(
  <PlantSelectionInput />,
  document.getElementById('root')
);
```

此代码尝试将`<PlantSelectionInput/>`呈现到具有`"root"`ID 的元素上。如前所述，我们的捆绑工具 webpack 将自动将编译后的 JavaScript 捆绑成一个名为`main.js`的文件，并将其放置在`dist/`（即分发）目录中。这将与我们的`index.html`文件并排，后者将作为用户界面的入口到我们的应用程序。对于我们的目的，这只需要是一个简单的页面，用于演示`PlantSelectionInput`：

```js
<!DOCTYPE html>
<html>
<head>
  <title>EveryPlant Selection App</title>
  <style>
    /* our styles... */
  </style>
</head>
<body>
  <div id="root"></div>
  <script src="./main.js"></script>
</body>
</html>
```

我们可以在`index.html`中的`<style>`标签中放置任何相关的 CSS：

```js
<style>
.PlantSelectionInput {
  width: 100%;
  display: flex;
  position: relative;
}
.PlantSelectionInput input {
  background: #fff;
  font-size: 1em;
  flex: 1 1;
  padding: .5em;
  outline: none;
}
/* ... more styles here ... */
</style>
```

在较大的项目中，最好提出一个可以很好地与许多不同组件配合使用的缩放 CSS 解决方案。在*React*中运行良好的示例包括*CSS 模块*或*styled components*，两者都允许您定义仅针对单个组件的 CSS，避免了全局 CSS 的麻烦。

我们组件的样式并不特别具有挑战性，因为它只是一个文本项的列表。主要挑战在于确保当组件处于完全打开状态时，建议列表出现在页面上的任何其他内容之上。这可以通过相对定位`<input>`容器，然后绝对定位`<ol>`来实现，如下所示：

![](img/f6a48501-2aea-4bf9-80ea-22f15d752833.png)

这结束了我们组件的实现，但我们还应该实现基本级别的测试（至少）。为了实现这一点，我们将使用 Jest，一个测试库，以及它的快照匹配功能。这将使我们能够确认我们的 React 组件生成了预期的 DOM 元素层次结构，并将保护我们免受未来的回归：

```js
// PlantSelectionInput.test.jsx

import React from 'react';
import renderer from 'react-test-renderer';
import PlantSelectionInput from './';

describe('PlantSelectionInput', () => {

  it('Should render deterministically to its snapshot', () => {
    expect(
      renderer
        .create(<PlantSelectionInput />)
        .toJSON()
    ).toMatchSnapshot();
  });

  describe('With configured isInitiallyOpen & value properties', () => {
    it('Should render deterministically to its snapshot', () => {
      expect(
        renderer
          .create(
            <PlantSelectionInput
              isInitiallyOpen={true}
              value="Example..."
            />
          )
          .toJSON()
      ).toMatchSnapshot();
    });
  });

});

```

Jest 会将生成的快照保存到`__snapshots__`目录中，然后将任何将来执行的测试与这些保存的快照进行比较。除了这些测试，我们还可以实现常规的功能性测试，甚至是 E2E 测试，可以编码期望，比如*当用户输入时，建议列表相应更新*。

这结束了我们对组件和案例研究的构建。如果您查看我们的 GitHub 存储库，您可以看到已完成的项目，与组件一起玩耍，自己运行测试，并且您还可以分叉存储库以进行自己的更改。

这是 GitHub 存储库的链接：[`github.com/PacktPublishing/Clean-Code-in-JavaScript`](https://github.com/PacktPublishing/Clean-Code-in-JavaScript)。

# 总结

在这最后一章中，我们通过书中积累的原则和经验，探讨了一个真实世界的问题。我们提出了用户遇到的问题，然后设计和实现了一个干净的用户体验来解决他们的问题。这包括了服务器端和客户端的部分，使我们能够从头到尾看到一个独立的 JavaScript 项目可能是什么样子。虽然我们无法涵盖每一个细节，但我希望这一章对于巩固*干净代码*背后的核心思想有所帮助，现在您应该更有信心编写干净的 JavaScript 代码来解决各种问题领域。我希望您能带走的一个核心原则就是：**专注于用户**。
