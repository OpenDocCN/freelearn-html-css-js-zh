# 第八章 应用程序模式

到目前为止，我们已经花了大量时间研究用于解决局部问题的模式，也就是说，仅涉及少数类而不是整个应用程序的问题。这些模式的范围很窄。它们通常只涉及两三个类，并且在任何给定的应用程序中可能只使用一次。您可以想象一下，还有更大规模的模式适用于整个应用程序。您可以将“工具栏”视为一个通用模式，它在应用程序中的许多地方使用。而且，它是在许多应用程序中使用的模式，以赋予它们相似的外观和感觉。模式可以帮助指导整个应用程序的组装方式。在本章中，我们将研究一系列我称之为 MV*家族的模式。这个家族包括 MVC、MVVM、MVP，甚至 PAC。就像它们的名字一样，这些模式本身非常相似。本章将涵盖这些模式，并展示如何或者是否可以将它们应用到 JavaScript 中。我们还将特别关注这些模式之间的区别。在本章结束时，您应该能够在鸡尾酒会上以 MVP 与 MVC 的微妙差异为题向客人讲解。涉及的主题将如下：+   模型视图模式的历史+   模型视图控制器+   模型视图呈现器+   模型视图视图模型# 首先，一些历史在应用程序内部分离关注点是一个非常重要的想法。我们生活在一个复杂且不断变化的世界中。这意味着不仅几乎不可能制定一个完全符合用户需求的计算机程序，而且用户需求是一个不断变化的迷宫。再加上一个事实，即用户 A 的理想程序与用户 B 的理想程序完全不同，我们肯定会陷入混乱。我们的应用程序需要像我们换袜子一样频繁地更改：至少每年一次。分层应用程序和维护模块化可以减少变更的影响。每个层次对其他层次了解得越少越好。在层次之间保持简单的接口可以减少一个层次的变更对另一个层次的影响的机会。如果您曾经仔细观察过高质量的尼龙制品（如热气球、降落伞或昂贵的夹克），您可能会注意到这种织物似乎形成了微小的方块。这是因为每隔几毫米，都会添加一根粗的加强线到织物中，形成交叉网格图案。如果织物被撕裂，那么撕裂将会被加强线停止或至少减缓。这限制了损坏的范围，并防止其扩散。应用程序中的层次和模块与限制变更的影响传播是完全相同的。在本书的前几章中，我们谈到了开创性的语言 Smalltalk。它是使类著名的语言。像许多这些模式一样，原始的 MV*模式，**模型 视图 控制器**（**MVC**），在被识别之前就已经被使用。尽管很难证明，但 MVC 最初是在 20 世纪 70 年代末由挪威计算机科学家 Trygve Reenskaug 在访问传奇般的施乐 PARC 期间提出的。在 20 世纪 80 年代，该模式在 Smalltalk 应用程序中得到了广泛使用。然而，直到 1988 年，该模式才在一篇名为《使用模型-视图-控制器用户界面范式的烹饪书》的文章中得到更正式的记录，作者是 Glenn E. Krasner 和 Stephen T. Pope。# 模型 视图 控制器 MVC 是一种用于创建丰富、交互式用户界面的模式：这正是在网络上变得越来越受欢迎的界面类型。敏锐的读者已经发现，该模式由三个主要组件组成：模型、视图和控制器。您可以在这个插图中看到信息在这些组件之间的流动：![模型 视图 控制器](img/Image00035.jpg)

前面的图表显示了 MVC 中三个组件之间的关系。

模型包含程序的状态。在许多应用程序中，该模型以某种形式包含在数据库中。模型可以从数据库等持久存储重新生成，也可以是瞬态的。理想情况下，模型是该模式中唯一可变的部分。视图和控制器都不与任何状态相关联。

对于一个简单的登录屏幕，模型可能如下所示：

```js
class LoginModel{
  UserName: string;
  Password: string;
  RememberMe: bool;
  LoginSuccessful: bool;
  LoginErrorMessage: string;
}
```

您会注意到，我们不仅为用户显示的输入字段设置了字段，还为登录状态设置了字段。用户可能不会注意到这一点，但它仍然是应用程序状态的一部分。

模型通常被建模为信息的简单容器。通常，模型中没有真正的功能。它只包含数据字段，也可能包含验证。在一些 MVC 模式的实现中，模型还包含有关字段的元数据，如验证规则。

### 注

裸对象模式是典型 MVC 模式的偏离。它通过大量的业务信息以及有关数据显示和编辑的提示来增强模型。甚至包含了将模型持久化到存储的方法。

裸对象模式中的视图是从这些模型自动生成的。控制器也是通过检查模型自动生成的。这将逻辑集中在显示和操作应用程序状态上，并使开发人员无需编写自己的视图和控制器。因此，虽然视图和控制器仍然存在，但它们不是实际的对象，而是从模型动态创建的。

已经成功地使用了该模式部署了几个系统。一些批评涌现，主要是关于如何从仅仅模型生成吸引人的用户界面以及如何正确协调多个视图。

在 Reenskaug 的博士论文《呈现裸对象》的序言中，他建议裸对象模式实际上比野生 MVC 的大多数派生更接近他最初对 MVC 的愿景。

当模型发生变化时，更新会传达给视图。通常通过使用观察者模式来实现。模型通常不知道控制器或视图。第一个只是告诉它要改变的东西，第二个只通过观察者模式更新，所以模型并不直接知道它。

视图基本上做您所期望的事情：将模型状态传达给目标。我不敢说视图必须是模型的视觉或图形表示，因为视图经常被传达到另一台计算机，并且可能以 XML、JSON 或其他数据格式的形式存在。在大多数情况下，特别是与 JavaScript 相关的情况，视图将是一个图形对象。在 Web 环境中，这通常是由浏览器呈现的 HTML。JavaScript 在手机和桌面上也越来越受欢迎，因此视图也可以是手机或桌面上的屏幕。

上述段落中的模型的视图可能如下图所示：

![模型视图控制器](img/Image00036.jpg)

在没有使用观察者模式的情况下，视图可能会定期轮询模型以查找更改。在这种情况下，视图可能必须保持状态的表示，或者至少一个版本号。重要的是，视图不要单方面更新此状态，而不将更新传递给控制器，否则模型和视图中的副本将不同步。

最后，模型的状态由控制器更新。控制器通常包含更新模型字段的所有逻辑和业务规则。我们登录页面的简单控制器可能如下所示：

```js
class LoginController {
  constructor(model) {
    this.model = model;
  }
  Login(userName, password, rememberMe) {
    this.model.UserName = userName;
    this.model.Password = password;
    this.model.RememberMe = rememberMe;
    if (this.checkPassword(userName, password))
      this.model.LoginSuccessful;
    else {
      this.model.LoginSuccessful = false;
      this.model.LoginErrorMessage = "Incorrect username or password";
    }
  }
};
```

控制器知道模型的存在，并且通常也知道视图的存在。它协调两者。控制器可能负责初始化多个视图。例如，单个控制器可以提供模型所有实例的列表视图，以及仅提供详细信息的视图。在许多系统中，控制器将对模型进行创建、读取、更新和删除（CRUD）操作。控制器负责选择正确的视图，并建立模型和视图之间的通信。

当需要对应用程序进行更改时，代码的位置应立即显而易见。例如：

| 更改 | 位置 |
| --- | --- |
| 屏幕上的元素间距不合适，更改间距。 | 视图 |
| 由于密码验证中的逻辑错误，没有用户能够登录。 | 控制器 |
| 要添加的新字段。 | 所有层 |

### 注意

**Presentation-Abstraction-Control**（**PAC**）是另一种利用三个组件的模式。在这种情况下，它的目标是描述一组封装的三元组的层次结构，更符合我们对世界的思考方式。控制，类似于 MVC 控制器，将交互传递到封装组件的层次结构中，从而允许信息在组件之间流动。抽象类似于模型，但可能只代表对于特定 PAC 而言重要的一些字段，而不是整个模型。最后，演示文稿实际上与视图相同。

PAC 的分层性质允许对组件进行并行处理，这意味着它可以成为当今多处理器系统中强大的工具。

您可能会注意到最后一个需要对应用程序的所有层进行更改。这种责任的多重位置是裸对象模式试图通过动态创建视图和控制器来解决的。MVC 模式通过根据用户交互中的角色将代码分割成位置。这意味着单个数据字段存在于所有层中，如图所示：

![模型视图控制器](img/Image00037.jpg)

有些人可能会称这为横切关注点，但实际上它并没有涵盖足够多的应用程序部分才能被称为这样。数据访问和日志记录是横切关注点，因为它们是普遍存在的，难以集中化。这种领域在不同层之间的普遍存在并不是一个主要问题。但是，如果这让你感到困扰，那么你可能是使用裸对象模式的理想候选人。

让我们开始构建一些 JavaScript 中表示 MVC 的代码。

## MVC 代码

让我们从一个简单的场景开始，我们可以应用 MVC。不幸的是，维斯特洛几乎没有计算机，可能是由于缺乏电力。因此，使用维斯特洛作为示例应用结构模式是困难的。遗憾的是，我们将不得不退一步，谈论一个控制维斯特洛的应用程序。让我们假设它是一个 Web 应用程序，并在客户端实现整个 MVC。

可以通过在客户端和服务器之间分割模型、视图和控制器来实现 MVC。通常，控制器会位于服务器上，并提供视图所知的 API。模型作为通信方法，既用于驻留在 Web 浏览器上的视图，也用于数据存储，可能是某种形式的数据库。在需要在客户端上进行一些额外控制的情况下，通常也会将控制器在服务器和客户端之间分割。

在我们的例子中，我们想创建一个控制城堡属性的屏幕。幸运的是，你很幸运，这不是一本关于使用 HTML 设计用户界面的书，因为我肯定会失败。我们将用图片代替 HTML：

![MVC 代码](img/Image00038.jpg)

在大多数情况下，视图只是为最终用户提供一组控件和数据。在这个例子中，视图需要知道如何调用控制器上的保存函数。让我们来设置一下：

```js
class CreateCastleView {
  constructor(document, controller, model, validationResult) {
    this.document = document;
    this.controller = controller;
    this.model = model;
    this.validationResult = validationResult;
    this.document.getElementById("saveButton").addEventListener("click", () => this.saveCastle());
    this.document.getElementById("castleName").value = model.name;
    this.document.getElementById("description").value = model.description;
    this.document.getElementById("outerWallThickness").value = model.outerWallThickness;
    this.document.getElementById("numberOfTowers").value = model.numberOfTowers;
    this.document.getElementById("moat").value = model.moat;
  }
  saveCastle() {
    var data = {
      name: this.document.getElementById("castleName").value,
      description: this.document.getElementById("description").value,
      outerWallThickness: this.document.getElementById("outerWallThickness").value,
      numberOfTowers: this.document.getElementById("numberOfTowers").value,
      moat: this.document.getElementById("moat").value
    };
    this.controller.saveCastle(data);
  }
}
```

您会注意到这个视图的构造函数包含对文档和控制器的引用。文档包含 HTML 和样式，由 CSS 提供。我们可以不传递对文档的引用，但以这种方式注入文档可以更容易地进行可测试性。我们将在后面的章节中更多地讨论可测试性。它还允许在单个页面上多次重用视图，而不必担心两个实例之间的冲突。

构造函数还包含对模型的引用，用于根据需要向页面字段添加数据。最后，构造函数还引用了一个错误集合。这允许将控制器的验证错误传递回视图进行处理。我们已经设置了验证结果为一个包装集合，看起来像下面这样：

```js
class ValidationResult{
  public IsValid: boolean;
  public Errors: Array<String>;
  public constructor(){
    this.Errors = new Array<String>();
  }
}
```

这里的唯一功能是按钮的`onclick`方法绑定到调用控制器上的保存。我们不是将大量参数传递给控制器上的`saveCastle`函数，而是构建一个轻量级对象并传递进去。这使得代码更易读，特别是在一些参数是可选的情况下。视图中没有真正的工作，所有输入直接传递给控制器。

控制器包含应用程序的真正功能：

```js
class Controller {
  constructor(document) {
    this.document = document;
  }
  createCastle() {
    this.setView(new CreateCastleView(this.document, this));
  }
  saveCastle(data) {
    var validationResult = this.validate(data);
    if (validationResult.IsValid) {
      //save castle to storage
      this.saveCastleSuccess(data);
    }
    else {
      this.setView(new CreateCastleView(this.document, this, data, validationResult));
    }
  }
  saveCastleSuccess(data) {
    this.setView(new CreateCastleSuccess(this.document, this, data));
  }
  setView(view) {
    //send the view to the browser
  }
  validate(model) {
    var validationResult = new validationResult();
    if (!model.name || model.name === "") {
      validationResult.IsValid = false;
      validationResult.Errors.push("Name is required");
    }
    return;
  }
}
```

这里的控制器做了很多事情。首先，它有一个`setView`函数，指示浏览器将给定的视图设置为当前视图。这可能是通过使用模板来完成的。这个工作的机制对于模式来说并不重要，所以我会留给你的想象力。

接下来，控制器实现了一个`validate`方法。该方法检查模型是否有效。一些验证可能在客户端执行，比如测试邮政编码的格式，但其他验证需要与服务器通信。如果用户名必须是唯一的，那么在客户端没有合理的方法在不与服务器通信的情况下进行测试。在某些情况下，验证功能可能存在于模型中而不是控制器中。

设置各种不同视图的方法也在控制器中找到。在这种情况下，我们有一个创建城堡的视图，然后成功和失败的视图。失败情况只返回相同的视图，并附加一系列验证错误。成功情况返回一个全新的视图。

将模型保存到某种持久存储的逻辑也位于控制器中。再次强调，实现这一点不如看到与存储系统通信的逻辑位于控制器中重要。

MVC 中的最后一个字母是模型。在这种情况下，它是一个非常轻量级的模型：

```js
class Model {
  constructor(name, description, outerWallThickness, numberOfTowers, moat) {
    this.name = name;
    this.description = description;
    this.outerWallThickness = outerWallThickness;
    this.numberOfTowers = numberOfTowers;
    this.moat = moat;
  }
}
```

正如你所看到的，它只是跟踪构成应用程序状态的变量。

这种模式中的关注点得到了很好的分离，使得相对容易进行更改。

# 模型视图 Presenter

**模型视图** **Presenter**（**MVP**）模式与 MVC 非常相似。这是在微软世界中相当知名的模式，通常用于构建 WPF 和 Silverlight 应用程序。它也可以在纯 JavaScript 中使用。关键区别在于系统不同部分的交互方式以及它们的责任范围。

第一个区别是，使用 Presenter 时，Presenter 和 View 之间存在一对一的映射关系。这意味着在 MVC 模式中存在于控制器中的逻辑，用于选择正确的视图进行渲染，不存在。或者说它存在于模式关注范围之外的更高级别。选择正确的 Presenter 可能由路由工具处理。这样的路由器将检查参数并提供最佳的 Presenter 选择。MVP 模式中的信息流可以在这里看到：

![Model View Presenter](img/Image00039.jpg)

Presenter 既知道 View 又知道 Model，但 View 不知道 Model，Model 也不知道 View。所有通信都通过 Presenter 传递。

Presenter 模式通常以大量的双向分发为特征。点击将在 Presenter 中触发，然后 Presenter 将更新模型并更新视图。前面的图表表明输入首先通过视图传递。在 MVP 模式的被动版本中，视图与消息几乎没有交互，因为它们被传递到 Presenter。然而，还有一种称为活动 MVP 的变体，允许视图包含一些额外的逻辑。

这种活跃版本的 MVP 对于 Web 情况可能更有用。它允许在视图中添加验证和其他简单逻辑。这减少了需要从客户端传递回 Web 服务器的请求数量。

让我们更新现有的代码示例，使用 MVP 而不是 MVC。

## MVP 代码

让我们从视图开始：

```js
class CreateCastleView {
  constructor(document, presenter) {
    this.document = document;
    this.presenter = presenter;
    this.document.getElementById("saveButton").addEventListener("click", this.saveCastle);
  }
  setCastleName(name) {
    this.document.getElementById("castleName").value = name;
  }
  getCastleName() {
    return this.document.getElementById("castleName").value;
  }
  setDescription(description) {
    this.document.getElementById("description").value = description;
  }
  getDescription() {
    return this.document.getElementById("description").value;
  }
  setOuterWallThickness(outerWallThickness) {
    this.document.getElementById("outerWallThickness").value = outerWallThickness;
  }
  getOuterWallThickness() {
    return this.document.getElementById("outerWallThickness").value;
  }
  setNumberOfTowers(numberOfTowers) {
    this.document.getElementById("numberOfTowers").value = numberOfTowers;
  }
  getNumberOfTowers() {
    return parseInt(this.document.getElementById("numberOfTowers").value);
  }
  setMoat(moat) {
    this.document.getElementById("moat").value = moat;
  }
  getMoat() {
    return this.document.getElementById("moat").value;
  }
  setValid(validationResult) {
  }
  saveCastle() {
    this.presenter.saveCastle();
  }
}
```

如你所见，视图的构造函数不再接受对模型的引用。这是因为 MVP 中的视图不知道使用的是哪个模型。这些信息由 Presenter 抽象掉。保留对 Presenter 的引用仍然在构造函数中，以便向 Presenter 发送消息。

没有模型的情况下，公共 setter 和 getter 方法的数量增加了。这些 setter 允许 Presenter 更新视图的状态。getter 提供了一个抽象，隐藏了视图存储状态的方式，并为 Presenter 提供了获取信息的途径。`saveCastle`函数不再向 Presenter 传递任何值。

Presenter 的代码如下：

```js
class CreateCastlePresenter {
  constructor(document) {
    this.document = document;
    this.model = new CreateCastleModel();
    this.view = new CreateCastleView(document, this);
  }
  saveCastle() {
    var data = {
      name: this.view.getCastleName(),
      description: this.view.getDescription(),
      outerWallThickness: this.view.getOuterWallThickness(),
      numberOfTowers: this.view.getNumberOfTowers(),
      moat: this.view.getMoat()
    };
    var validationResult = this.validate(data);
    if (validationResult.IsValid) {
      //write to the model
      this.saveCastleSuccess(data);
    }
    else {
      this.view.setValid(validationResult);
    }
  }
  saveCastleSuccess(data) {
    //redirect to different presenter
  }
  validate(model) {
    var validationResult = new validationResult();
    if (!model.name || model.name === "") {
      validationResult.IsValid = false;
      validationResult.Errors.push("Name is required");
    }
    return;
  }
}
```

你可以看到，视图现在以持久的方式在 Presenter 中被引用。`saveCastle`方法调用视图以获取其数值。然而，Presenter 确保使用视图的公共方法而不是直接引用文档。`saveCastle`方法更新了模型。如果存在验证错误，它将回调视图以更新`IsValid`标志。这是我之前提到的双重分发的一个例子。

最后，模型与以前一样保持不变。我们将验证逻辑保留在 Presenter 中。在哪个级别进行验证，模型还是 Presenter，不如在整个应用程序中一致地进行验证更重要。

MVP 模式再次是构建用户界面的一个相当有用的模式。视图和模型之间更大的分离创建了一个更严格的 API，允许更好地适应变化。然而，这是以更多的代码为代价的。更多的代码意味着更多的错误机会。

# 模型视图视图模型

我们将在本章中看到的最后一种模式是模型视图视图模型模式，更常被称为 MVVM。到现在为止，这种模式应该已经相当熟悉了。再次，你可以在这个插图中看到组件之间的信息流：

![模型视图视图模型](img/Image00040.jpg)

你可以看到这里许多相同的构造已经回来了，但它们之间的通信有些不同。

在这种变化中，以前的控制器和 Presenter 现在是视图模型。就像 MVC 和 MVP 一样，大部分逻辑都在中心组件中，这里是视图模型。MVVM 中的模型本身实际上非常简单。通常，它只是一个简单地保存数据的信封。验证是在视图模型中完成的。

就像 MVP 一样，视图完全不知道模型的存在。不同之处在于，在 MVP 中，视图知道自己在与某个中间类交谈。它调用方法而不是简单地设置值。在 MVVM 中，视图认为视图模型就是它的视图。它不是调用像`saveCastle`这样的操作并传递数据，或者等待数据被请求，而是在字段发生变化时更新视图模型上的字段。实际上，视图上的字段与视图模型绑定。视图模型可以通过将这些值代理到模型，或者等到调用保存等类似操作时传递数据。

同样地，对视图模型的更改应立即在视图中反映出来。一个视图可能有多个视图模型。这些视图模型中的每一个可能会向视图推送更新，或者通过视图向其推送更改。

让我们来看一个真正基本的实现，然后我们将讨论如何使其更好。

## MVVM 代码

天真的视图实现，坦率地说，是一团糟：

```js
var CreateCastleView = (function () {
  function CreateCastleView(document, viewModel) {
    this.document = document;
    this.viewModel = viewModel;
    var _this = this;
    this.document.getElementById("saveButton").addEventListener("click", function () {
    return _this.saveCastle();
  });
  this.document.getElementById("name").addEventListener("change", this.nameChangedInView);
  this.document.getElementById("description").addEventListener("change", this.descriptionChangedInView);
  this.document.getElementById("outerWallThickness").addEventListener("change", this.outerWallThicknessChangedInView);
  this.document.getElementById("numberOfTowers").addEventListener("change", this.numberOfTowersChangedInView);
  this.document.getElementById("moat").addEventListener("change",this.moatChangedInView);
}
CreateCastleView.prototype.nameChangedInView = function (name) {
  this.viewModel.nameChangedInView(name);
};

CreateCastleView.prototype.nameChangedInViewModel = function (name) {
  this.document.getElementById("name").value = name;
};
//snipped more of the same
CreateCastleView.prototype.isValidChangedInViewModel = function (validationResult) {
  this.document.getElementById("validationWarning").innerHtml = validationResult.Errors;
  this.document.getElementById("validationWarning").className = "visible";
};
CreateCastleView.prototype.saveCastle = function () {
  this.viewModel.saveCastle();
};
return CreateCastleView;
})();
CastleDesign.CreateCastleView = CreateCastleView;
```

这是高度重复的，每个属性都必须被代理回`ViewModel`。我截断了大部分代码，但总共有大约 70 行。视图模型内部的代码同样糟糕：

```js
var CreateCastleViewModel = (function () {
  function CreateCastleViewModel(document) {
    this.document = document;
    this.model = new CreateCastleModel();
    this.view = new CreateCastleView(document, this);
  }
  CreateCastleViewModel.prototype.nameChangedInView = function (name) {
    this.name = name;
  };

  CreateCastleViewModel.prototype.nameChangedInViewModel = function (name) {
    this.view.nameChangedInViewModel(name);
  };
  //snip
  CreateCastleViewModel.prototype.saveCastle = function () {
    var validationResult = this.validate();
    if (validationResult.IsValid) {
      //write to the model
      this.saveCastleSuccess();
    } else {
      this.view.isValidChangedInViewModel(validationResult);
    }
  };

  CreateCastleViewModel.prototype.saveCastleSuccess = function () {
    //do whatever is needed when save is successful.
    //Possibly update the view model
  };

  CreateCastleViewModel.prototype.validate = function () {
    var validationResult = new validationResult();
    if (!this.name || this.name === "") {
      validationResult.IsValid = false;
        validationResult.Errors.push("Name is required");
    }
    return;
  };
  return CreateCastleViewModel;
})();
```

看一眼这段代码就会让你望而却步。它的设置方式会鼓励复制粘贴编程：这是引入代码错误的绝佳方式。我真希望有更好的方法来在模型和视图之间传递变化。

## 在模型和视图之间传递变化的更好方法

你可能已经注意到，有许多 MVVM 风格的 JavaScript 框架。显然，如果它们遵循我们在前一节中描述的方法，它们就不会被广泛采用。相反，它们遵循两种不同的方法之一。

第一种方法被称为脏检查。在这种方法中，与视图模型的每次交互之后，我们都会遍历其所有属性，寻找变化。当发现变化时，视图中的相关值将被更新为新值。对于视图中值的更改，操作都附加到所有控件上。然后这些操作会触发对视图模型的更新。

对于大型模型来说，这种方法可能会很慢，因为遍历大型模型的所有属性是昂贵的。可以导致模型变化的事情很多，而且没有真正的方法来判断是否通过更改另一个字段来改变模型中的一个远程字段，而不去验证它。但好处是，脏检查允许你使用普通的 JavaScript 对象。不需要像以前那样编写代码。而另一种方法——容器对象则不然。

使用容器对象提供了一个特殊的接口，用于包装现有对象，以便直接观察对象的变化。基本上这是观察者模式的应用，但是动态应用，因此底层对象并不知道自己正在被观察。间谍模式，也许？

这里举个例子可能会有所帮助。假设我们现在使用的是之前的模型对象：

```js
var CreateCastleModel = (function () {
  function CreateCastleModel(name, description, outerWallThickness, numberOfTowers, moat) {
    this.name = name;
    this.description = description;
    this.outerWallThickness = outerWallThickness;
    this.numberOfTowers = numberOfTowers;
    this.moat = moat;
  }
  return CreateCastleModel;
})();
```

然后，`model.name`不再是一个简单的字符串，我们会在其周围包装一些函数。在 Knockout 库的情况下，代码如下所示：

```js
var CreateCastleModel = (function () {
  function CreateCastleModel(name, description, outerWallThickness, numberOfTowers, moat) {
    this.name = ko.observable(name);
    this.description = ko.observable(description);
    this.outerWallThickness = ko.observable(outerWallThickness);
    this.numberOfTowers = ko.observable(numberOfTowers);
    this.moat = ko.observable(moat);
  }
  return CreateCastleModel;
})();
```

在上面的代码中，模型的各种属性都被包装成可观察的。这意味着它们现在必须以不同的方式访问：

```js
var model = new CreateCastleModel();
model.name("Winterfell"); //set
model.name(); //get
```

这种方法显然给你的代码增加了一些摩擦，并且使得更改框架相当复杂。

当前的 MVVM 框架在处理容器对象与脏检查方面存在分歧。AngularJS 使用脏检查，而 Backbone、Ember 和 Knockout 都使用容器对象。目前还没有明显的赢家。

## 观察视图变化

幸运的是，Web 上 MV*模式的普及以及观察模型变化的困难并没有被忽视。你可能期待我说这将在 ECMAScript-2015 中得到解决，因为这是我的正常做法。奇怪的是，解决所有这些问题的`Object.observe`，是 ECMAScript-2016 讨论中的一个功能。然而，在撰写本文时，至少有一个主要浏览器已经支持它。

可以像下面这样使用：

```js
var model = { };
Object.observe(model, function(changes){
  changes.forEach(function(change) {
    console.log("A " + change.type + " occured on " +  change.name + ".");
    if(change.type=="update")
      console.log("\tOld value was " + change.oldValue );
  });
});
model.item = 7;
model.item = 8;
delete model.item;
```

通过这个简单的接口来监视对象的变化，可以消除大型 MV*框架提供的大部分逻辑。为 MV*编写自己的功能将会更容易，实际上可能根本不需要使用外部框架。

# 提示和技巧

各种 MV*模式的不同层不一定都在浏览器上，也不一定都需要用 JavaScript 编写。许多流行的框架允许在服务器上维护模型，并使用 JSON 进行通信。

`Object.observe`可能还没有在所有浏览器上可用，但有一些 polyfill 可以用来创建类似的接口。性能不如原生实现好，但仍然可用。

# 总结

将关注点分离到多个层次可以确保应用程序的变化像防撕裂一样被隔离。各种 MV*模式允许在图形应用程序中分离关注点。各种模式之间的差异在于责任如何分离以及信息如何通信。

在下一章中，我们将探讨一些模式和技术，以改善开发和部署 JavaScript 到 Web 的体验。

