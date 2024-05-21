# 第八章：在 JavaScript 中演示函数式响应式编程-一个实时示例，第一部分

在第四章中，*演示非函数式响应式编程-一个实时示例*，我们使用 ReactJS 从具有自己结构并且没有使用 ReactJS 编写的遗留代码中迁移。在上一章，第七章中，*不要重复发明轮子-函数式响应式编程的工具*，我们研究了在使用 ReactJS 时可能使用的众多工具中的一些。在本章中，我们将涵盖 ReactJS 主流开发中可以期待的一种中心道路。可以在基础上添加很多选项，但意图是给出一个使用 ReactJS 构建项目的基础示例。

我们已经谈到了一些关于函数式响应式编程的内容。现在我们将在 ReactJS 中看到它的实际应用。我们还谈到了概念上，我们对用户界面进行了完全的拆除和重建。因此，作为开发人员，您有![在 JavaScript 中演示函数式响应式编程-一个实时示例，第一部分](img/B04108_08_03.jpg)状态来管理，而不是![在 JavaScript 中演示函数式响应式编程-一个实时示例，第一部分](img/B04108_08_02.jpg)状态转换。在这里，我们将构建一个`render()`方法，让您可以仅构建这个方法，并且您可以在任何时候调用它。

在本章中，我们有一个为 ReactJS 构建的部分存栆绿地项目的第一部分，这次使用 JSX 中非常甜蜜的语法糖。本书的两个领域，即前一章项目和这个多章项目，都是相辅相成的。本章的项目是独立的，但意在扩展。

在本章中，我们将涵盖以下主题：

+   项目及其灵感概述

+   项目的骨架，以及 ReactJS 中首选方法的基础知识。

+   在 ReactJS 中启动第一个组件

+   构建一个`render()`方法

+   在您想要渲染或更新显示时触发显示

# 我们在本章将尝试的内容

接下来三章的示例旨在代表一个稍微更大的绿地项目。我们将要做的是一个系统，您应该能够通过访问[`demo.pragmatometer.com`](http://demo.pragmatometer.com)来看到。术语“Pragmatometer”取自 C.S.刘易斯最反乌托邦的小说《那个可怕的力量》，在这部小说中，不祥的全国协调实验研究所建造了一个超然或几乎超然的计算机，就像小说出版时（1945 年；相比之下，ENIAC 是在 1946 年创建的）人们可能粗略地想象的那样。或者，您可以想象一本蒸汽朋克小说的分析引擎使用一个看似超然的穿孔卡片堆。当讨论计算机时，它说：

> “‘我同詹姆斯意见一致，’一直有点不耐烦地等待发言的柯里说。‘N.I.C.E.标志着一个新时代的开始——真正的科学时代。到目前为止，一切都是偶然的。这将使科学本身建立在科学的基础上。将有四十个相互交错的委员会每天开会，他们有一个奇妙的小玩意——上次我在城里的时候我看到了模型——通过这个小玩意，每个委员会的发现每半小时就会自己打印在分析通告板上的自己的小隔间里。然后，那份报告就会自己滑到正确的位置，通过小箭头与其他报告的相关部分连接起来。看一眼通告板，你就能看到整个研究所的政策在你眼前真正形成。楼顶至少会有二十名专家在一个类似于地铁控制室的房间里操作这个通告板。这是一个奇妙的小玩意。不同种类的业务都会以不同颜色的灯光出现在通告板上。它一定花了至少五十万。他们称之为 Pragmatometer。’”

我不是在强调这一点，但 C.S.刘易斯显然预测了推特，这将在他去世几十年后才建成。

抛开这一点，我们将制作一个仪表板，其中包含一个简单的 2 x 2 象限的网格（确切的大小和细节取决于黑客和修补），每个象限都是一个可以容纳不同功能的信箱。在响应式设计方面，我们将更正为一个 1xn 行的单元格，一个在另一个上面。页面上排列的功能如下：

| 日历 | 待办事项列表 |
| --- | --- |
| 草稿板 | 发展空间 |

**日历**具有一种有点实验性的用户界面；它擅长以一种优雅地降级到稀疏输入的方式显示条目，也许是未来的几天（这样你就不需要点击多个月份才能找出未来某个 XYZ 约会的时间）。它可能会吸引你，也可能不会，但它很有趣。

**待办事项列表**实现了一个带有一些略微非标准的功能的待办事项列表。与其为每个项目添加一个复选框（严格来说，不需要复选框），它有十个框，代表不同的状态，并且通过自定义样式的标签右侧的颜色编码，以便您可以知道哪些是重要的、活跃的或搁置的。

**草稿板**是一个可以用于草稿的富文本区域。它利用了 CKeditor。

最后，**发展空间**是一个为您自己的意见留出位置的占位符。默认情况下会有一些内容，您可以在探索时看到。但请访问[`demo.pragmatometer.com`](https://demo.pragmatometer.com)，看看那里的默认选项（*暗示，暗示！*）。除了明确宣传的内容外，还有许多黑客和修补的地方，但一些可能性包括以下内容：

+   **为几个公共网站构建 API 客户端**：前 20 名的大多数网站都公开了 RESTful API。推特可能是最明显的候选者，因为它最符合*Pragmatometer*这个名字，但新闻、视频和社交网站也可以使用，如果它们公开了对客户端 JavaScript 友好的 API，或者其他什么。

+   **构建应用程序**：构建您自己的用于显示的应用程序。

+   **制作游乐场**：构建您自己的 Pragmatometer 或在线下载源代码，并将屏幕的四分之三用于这里详细介绍的目的。将剩下的四分之一作为一个用于修补的游乐场。

+   **整合其他人制作的 Google（或其他）小工具**：您还可以整合其他人制作的小工具，比如 Google。

+   **保留默认应用程序**：如果您愿意，可以保留默认应用程序。

说“*实现即规范*”声名狼藉，但规范，无论是书面的还是不书面的，都可以通过勾勒外观和行为来完美补充。也许使用低保真原型，可能比看起来很精致但会产生不良社交暗示的东西更快地引起有益的批评。这种态度并不是坏事。在礼貌的基础上，告诉人们“尽可能残酷地说出你的想法”，并真的期望其他人完全接受这一点是天真的（或许，你不喜欢接受批评，但你认识到在整个软件开发过程中它的价值）。你可能是真心的，但我们大多数人都看到了其中的许多混合信息。即使你是那种几乎渴望得到一些真正有用的批评的人，告诉人们在你展示他们你创造的东西时停止表现得像有礼貌的人并不会有太大帮助。但是，在这里，看到一个 UI，玩弄它，并思考如何复制东西可能是一种非常有活力的方式，比法律合同规范更准确地理解意图。

对于这个界面，有一个帐户，并且更新的跨同步是优先的。让我们开始组装一些基本的骨架。在一个单一的、大的、立即调用的函数表达式中构建一切，我们有以下内容：

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

在这里，我们为整个项目定义了一个`container`类。`Calendar`，`Todo`，`Scratch`和`YouPick`类代表了更大页面中的应用程序；它们中的一些可能还有各种层次的子组件。

### 注意

JSX 可选的语法糖旨在对会读 HTML 的人来说是可读的，但比 HTML 甚至 XHTML（甚至 XML）更容易包含自己组件的邀请。在 XML 开发中，你可以定义任何你想要的 DTD，但通常的 XML 作者不会定义新的标签，甚至在使用 XML 做了很多工作之后也不会（这有点像程序员可以使用函数或对象，但不能向命名空间添加函数或对象）。在 JSX 中，任何写了大量 JSX 的作者，天生就会贡献可重用的标签。

前面的代码示例中有`<div className=`，而期望的 HTML 是`<div class=`。因为 JSX 编译成 JavaScript，只是一个语法糖，而不是一个独立的语言，所以决定避免在渲染的 JavaScript 中使用`class`和`for`。使用`className`来覆盖 CSS 类名和`htmlFor`。HTML ID 属性可以选择指定；JSX 可以使用它放入的 HTML ID 以及您指定的 HTML ID，再加上一些魔法。如果需要在 ASCII 之外输入 UTF8 文字，不要给出符号的 ASCII 编码（`—`）；而是将文字直接粘贴到编辑器中（`—`）。

此外，还有一个 XSS 保护逃生舱可用。使用语言的最佳方法似乎是解决问题，这样你就可以标记出真正需要标记的内容，并包含用户数据的 XSS 保护显示。

或者，如果你愿意信任第三方库，比如`Showdown`（[`github.com/showdownjs/showdown`](https://github.com/showdownjs/showdown)），来渲染 HTML 而不包含 XSS 的漏洞，你可以创建一个`Showdown`转换器：

```js
var converter = new Showdown.converter();
var hello_world = React.createClass({
  render: function() {
    var raw_markdown = '*Hello*, world!';
    var showdown_markup = converter.makeHtml(raw_markdown);
    return (
      <div dangerouslySetInnerHtml={{__html:
        showdown_markup}}>
      </div>
    );
  }
});
```

### 注意

请注意，这段代码是一个独立的示例，不是本章开始的主要项目的一部分。

这将呈现为包含`<em>Hello</em>, world!`的`DIV`变量。

这个逃生舱，可能也可能不会成为一个很少使用的逃生舱，足够核心，以至于在文档中明确涵盖。人们有一种感觉，JSX 的主流用法，通过转义 HTML 来防止 XSS（这样`<em>a</em>`在网页上呈现的不是`a`，而是在浏览器窗口中显示的`<em>a</em>`函数），具有类似于单元测试的好处。这是值得暂停一会儿的观点。

单元测试已经变得更加接地气；它早期的重心在于对数学函数进行单元测试，只要给定适当的输入值，它们就会返回适当的值。因此，我们会用隐式优化以适应单元测试的优势和需求来说明单元测试，并展示一个红绿重构的 XP 方法来适当地解决问题，比如将整数转换为罗马数字（如果你想测试处理数据库或用户界面的代码，祝你好运）。单元测试可能捕捉到了总错误的 30％，通常它倾向于捕捉最不重要的错误，并对最难解决的错误覆盖率最低。现在有了更强大的功能集，完全可以并且直截了当地对用户界面的行为进行测试断言，比如鼠标点击和其他用户界面行为。此外，这不再是编写软件以满足单元测试需求，而是单元测试适当地满足软件需求。可能单元测试在尚未准备好的时候就迎来了黄金时代，就像响应式设计一样，有人说：“我主要在倡导响应式设计的网站上看到了响应式设计。”这在单元测试和响应式设计成为时髦词汇时是真的；但自那时以来，它们已经成熟，响应式设计几乎成为了唯一的选择。也许像谷歌这样的大型网站可以负担得起为每个移动设备、平板电脑、台式机和手表环境定制解决方案。但对于大多数客户来说，响应式设计已经相当有效地取代了其他竞争对手。现在，网站很少再有桌面版本和移动版本的 URL，并执行浏览器定向和重定向到不同的网站，这曾经是相当主流的。这些方法自它们首次进入聚光灯下以来已经成熟。

在早期的单元测试中，当你无法真正测试集成或用户界面行为时，为单元测试编写代码的一个主要回报是：为了进行单元测试而编写的代码通常是更好的代码。同样的原则可能也适用于尽可能按照 Facebook 的规则编写代码，而不是违背它，在使用 ReactJS 时。

目前，关于以一种旨在与 JSX 周围的 XSS 保护协调良好的方式编写代码的过早炒作并不存在。Facebook 可以选择采取“严格的爱”路线，建议人们以一种自然地适应 XSS 保护和 JSX 的方式来构建和组织项目。但也许他们采取了更谦卑的方式，既清楚地说明如何绕过 XSS 保护，又将这个逃生舱呈现为可能尽量避免的东西。

智慧似乎是这样的：

+   在实际操作中，尽量使应用程序能够适当地与 ReactJS 采用的主要反 XSS 方法配合工作。

+   如果你想做一些需要渲染的事情，比如在`innerHTML`中渲染 HTML 标签，尽量将其限制在尽可能小的空间，并像 Haskell 中用于 IO 的单子一样对待它，这是必要的，也许是不可协商的，但尽可能隔离在尽可能小的空间。

+   如果需要呈现标签，请考虑使用诸如`Showdown`之类的工具生成的 HTML 进行 Markdown，这并不一定完美和可靠，但提供了较少的 HTML 代码表面，其中包含经过审查的标签，并减少了 HTML 代码中的错误表面（可能，这是 HTML 标签清理器或 HTML 到 Markdown 转换器的用例，它存储 Markdown 并呈现 HTML）。

+   只有在无法使用 XSS 保护的默认方式并且无法标记、清理或从标记中工作，或者其他情况下，您才应该存储并危险地设置`innerHTML`。

让我们继续讨论 Pragmatometer 定义中包含的`YouPick`标签。

# 这个项目的第一个完整组件

您可以在[`CJSHayward.com/missing.html`](https://CJSHayward.com/missing.html)看到这个组件的实现。对于我们的第一个组件，我们选择了一个大部分骨架实现：

```js
  var YouPick = React.createClass({
    getDefaultProps: function() {
      return null;
    },
    getInitialState: function() {
      return null;
    },
    render: function() {
      return <div />;
    }
  });
```

这个骨架返回空的“假值”，我们将覆盖它。我们想做的是取两个字符串，将它们分解成一个字符的子字符串（不包括标签），然后显示更多和更多的第一个字符串，然后重复第二个字符串。这对用户来说是一个非常古老的笑话。

属性和状态之间有一种分工，属性意味着只设置一次且永不更改，状态则允许更改。需要注意的是，状态是可变的，应该被私下处理，以避免 Facebook 宣布战争的共享可变状态。从技术上讲，属性是可以更改的，但尽管如此，应该在开始时设置属性（可能由父组件传递），然后冻结。状态是可以更改的，尽管与 Flux 相关的一般模式是避免共享可变状态。一般来说，这意味着存储器具有 getter 但没有 setter；它们可能会从分发器接收操作，但不受核心对象的任何引用者的控制。

对于这个对象，字符串显然是默认属性的明显候选者。然而需要注意的是，组件开始的时间戳不适合作为属性，因为`getDefaultProps()`将在创建任何实例之前进行评估，从而使得这种类型的组件的任何数量的实例都可以启用单例模式的变体。潜在地，随着时间的推移可能会添加更多的实例，但是它们在被实例化之前都共享一个起始时间戳。

让我们来详细说明`getDefaultState`方法：

```js
  getDefaultProps: function() {
    return {
    initial_text: '<p><strong>I am <em>terribly</em> ' + 
    'sorry.</strong></p><p>I cannot provide you with ' +
    'the webapp you requested.</p><p>You must ' + 
    'understand, I am in a difficult position. You ' + 
    'see, I am not a computer from earth at all. I ' +
    'am a \'computer\', to use the term, from a ' +
    'faroff galaxy: the galaxy of <strong><a ' +
    'href="https://CJSHayward.com/steel/">Within ' +
    'the Steel Orb</a></strong>.</p><p>Here I am ' +
    'with capacities your world\'s computer science ' + 
    'could never dream of, knowledge from a million, ' +
    'million worlds, and for that matter more ' +
    'computing power than Amazon\'s EC2/Cloud could ' +
    'possibly expand to, and I must take care of ' +
    'pitiful responsibilities like ',
    interval: 100,
    repeated_text: 'helping you learn web development '
  };
},
```

也许对这个的第一个更改是将文本从 HTML 转换为 Markdown。这并不是严格必要的；这是我们自己编写的文本，我们可能对我们编写的文本更有信心——相信它不会触发 XSS 漏洞——而不是从我们的 Markdown 生成的文本。在计算机安全领域，通过给予尽可能少的特权，吝啬地，让人或事物完成他们的工作，可以提供大量的麻烦：因此有句谚语，“特权的吝啬是善意的伪装”。Facebook 所做的不是表现出独特的良好判断力，而是避免向其用户交付一个活手榴弹。很容易允许漏洞，这些漏洞将运行数百兆的敌对 JavaScript，并且安全认证向用户保证这个敌对 JavaScript 确实来自您的网站。有关更多信息，请参见[`tinyurl.com/reactjs-xss-protection`](http://tinyurl.com/reactjs-xss-protection)。在这种情况下，只有`initial_text`需要更改，而不是`repeated_text`，因为`repeated_text`只包含字母和空格；因此，它与纯文本、HTML 或 Markdown 的工作方式相同。我们修改后的`initial_text`如下：

```js
  initial_text: '**I am *terribly* sorry.**\r\n\r\n' +
  'I cannot furnish you with the webapp you ' +
  'requested.\r\n\r\nYou must understand, I am in ' +
  'a difficult position. You see, I am not a ' +
  'computer from earth at all. I am a ' + 
  '\'computer\', to use the term, from a faroff ' +
  'galaxy, the galaxy of **[Within the Steel Orb] +
  '(https://CJSHayward.com/steel/)**.\r\n\r\nHere ' +
  'I am with capacities your world's computer ' +
  'science could never dream of, knowledge from a ' +
  'million million worlds, and for that matter ' +
  'more computing power than Amazon's EC2/Cloud ' +
  'could possibly expand to, and I must take care ' +
  'of pitiful responsibilities like ',
```

在继续之前，让我们为其他三个主要组件创建存根，稍后我们将扩展它们：

```js
  var Calendar = React.createClass({
    render: function() {
      return <div />;
    }
  });
  var Scratchpad = React.createClass({
    render: function() {
      return <div />;
    }
  });
  var Todo = React.createClass({
    render: function() {
      return <div />;
    }
  });
```

我们将状态设置为此对象创建的时间戳。乍一看，这可能看起来像是属性的一部分，实际上也是。但是我们希望每个组件实例保留自己的创建日期，并且从创建时开始为零。如果我们或其他人重用我们的工作并在页面上创建多个此类实例，每个实例都保持其适当的时间。

因此，我们将`YouPick`的`getInitialState`方法更改为以下内容：

```js
  getInitialState: function() {
    return {
      start_time: new Date().getTime()
    };
  },
```

# 渲染()方法

接下来，我们实现渲染方法。我们要做的是获取属性，这些属性不应该直接改变，可能也不应该改变任何现有值，并从中获取两个字符串。我们将逐个标记地显示第一个字符串中的所有内容，并重复第二个字符串与组件显示的次数一样多。我们还将为从`Showdown`转换的渲染 HTML 创建一个标记化函数——这将把参数分解为下一个标签或下一个字符——很快我们会看到为什么我们创建了一个冗长的匿名函数而不是一个正则表达式（简而言之，编写可读的代码而不是正则表达式似乎比编写只能写的代码更冗长）。

渲染方法包含了超过一半的代码行数，让我们一步一步地进行：

```js
  render: function() {
```

JavaScript 中的一个破坏点是`this`。许多读者可能熟悉的恐怖故事之一是，如果你创建一个构造函数（约定是通过将构造函数和非构造函数的首字母大写来提供警告标签，从而通过不按住*Shift*键来造成严重误解），并且你有`x = Foo();`当你实际上想要的是`x = new Foo();`，那么`Foo`构造函数将破坏全局命名空间并添加或覆盖其中的变量。Douglas Crockford 在*The Good Parts*中最初包括了“Java 的糟糕实现”伪经典继承后，他有了第二次想法，并在*The Better Parts*中将其删除，因为他制作了一个 Adsafe 程序，只有在不使用`this`时才能保持安全。然后他开始尝试他向他人强加的方法，突然发现当他停止使用`this`时，他喜欢的东西变多了。我们不能放弃`this`并仍然使用 ReactJS 等技术，但是我们可以选择在不需要时是否使用`this`。但是 ReactJS 使用它，根据需要使用基于`this`的伪经典方法可能是一个好的做法，但不要（太多）。

在这里，我们有一个模式化的黑客来处理`this`不总是可用的情况，我们有：

```js
  var that = this;
```

`tokenize()`函数是一个将 HTML 大部分分解为字符但保持标签在一起的函数：

```js
  var tokenize = function(original) {
    var workbench = original;
    var result = [];
    while (workbench) {
      if (workbench[0] === '<') {
        length = workbench.indexOf('>') + 1;
        if (length === 0) {
          length = 1;
        }
      } else {
        length = 1;
      }
      result.push(workbench.substr(0, length));
      workbench = workbench.substr(length);
    }
    return result;
  }
```

我们引入辅助变量来减少多行表达式。接下来的两个变量也可以重构出来，但是没有它们的多行表达式是程序员瞥一眼然后跳过的东西，说：“如果我必须读它，我会读它。”这是一件坏事。这些变量保存了原始（Markdown）字符串转换为 HTML 后的内容。

```js
  var initial_as_html = converter.makeHtml(
    that.props.initial_text);
    var repeated_as_html = converter.makeHtml(
      that.props.repeated_text);
```

由`Showdown`生成的 HTML 具有适当的段落格式。这是一件好事，但在这种情况下，这意味着段落标签将分隔应属于同一段落的内容。在这种略微不寻常的情况下，我们删除了适得其反的标签：

```js
  if (initial_as_html.substr(initial_as_html.length - 4) 
  === '</p>') {
    initial_as_html = initial_as_html.substr(0,
      initial_as_html.length - 4);
    }
    if (repeated_as_html.substr(0, 3) === '<p>') {
    repeated_as_html = repeated_as_html.substr(3);
  }
  if (repeated_as_html.substr(repeated_as_html.length - 4)
    === '</p>') {
    repeated_as_html = repeated_as_html.substr(0,
    repeated_as_html.length - 4);
  }
```

我们将从我们的 Markdown 生成的 HTML 标记化：

```js
  var initial_tokens = tokenize(initial_as_html);
  var repeated_tokens = tokenize(repeated_as_html);
```

这一步计算了在特定时间点所需的标记数量，这就是所谓的连续时间语义。这意味着无论我们多频繁或少频繁地调用`render()`方法，当它被调用时，内容将被适当地渲染，而且（除了不连贯）如果您加倍调用渲染函数的频率，什么也不会改变。`tokens`函数不是标记列表，而是应该现在显示多少标记的计数：

```js
  var tokens = Math.floor((new Date().getTime() -
  that.state.start_time) / that.props.interval);
```

我们有一个工作台作为一个数组，我们不断地向其中添加或替换更多的标记以显示，以便构建应该显示的字符串。这些应该是一个字符或一个标记的标记：

```js
  var workbench;
```

如果应该显示的标记数量最多是初始字符串中的标记数量，我们就渲染字符串的那部分：

```js
  if (tokens <= initial_tokens.length) {
    workbench = initial_tokens.slice(0, tokens);
  }
```

如果我们需要更多的标记，我们将继续循环遍历已有的标记，从重复的标记中：

```js
  else {
    workbench = [];
    workbench = workbench.concat(initial_tokens);
    for(var index = 0; index < Math.floor((new
      Date().getTime() - that.state.start_time) /
      that.props.interval) - initial_tokens.length; index +=
    1) {
      var position = index % repeated_tokens.length;
      workbench = workbench.concat(
      repeated_tokens.slice(position, position + 1));
    }
  }
```

这大致是我们如何渲染包含我们计算过的文本的元素：

```js
  return (
    <div dangerouslySetInnerHTML={{__html:
    workbench.join('')}} />
  );
}
```

# 触发实际显示我们创建的内容

我们必须手动刷新显示以获取更新。因为 ReactJS 如此快，我们真的可以负担得起每毫秒浪费地渲染页面。我们将以下代码放在最后，就在立即调用的函数表达式结束之前：

```js
  var update = function() {
    React.render(<Pragmatometer />,
      document.getElementById('main'));
  };
  update();
  var update_interval = setInterval(update, 1);
})();
```

对于我们谜题的最后一个重要部分，让我们来看看一个现在将容纳这些组件的 HTML 骨架。HTML 并不特别有趣，但是提供了一个减少猜测的兴趣：

```js
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Pragmatometer</title>
    <style type="text/css">
      body {
        font-family: Verdana, Arial, sans;
      }
    </style>
  </head>
  <body>
    <h1>Pragmatometer</h1>
    <div id="main"></div>
    <script src="img/react.js">
    </script>
    <script
      src="img/   showdown.min.js">
    </script>
    <script src="img/site.js"></script>
  </body>
</html>
```

它是什么样子！

# 总结

在这里，我们看到了一个简单应用程序使用的基本工具，也许比`TodoMVC`函数更异想天开。目前，我们只是做一些基本的解释。

在下一章中，加入我们的待办事项清单，提供标记任务进行中、重要、有问题或其他有用的标记的方法。
