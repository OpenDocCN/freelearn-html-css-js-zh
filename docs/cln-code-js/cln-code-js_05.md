# 第四章：SOLID 和其他原则

软件世界充斥着原则和首字母缩略词。关于我们应该如何编写代码有许多坚定和根深蒂固的想法。所有这些原则的数量之多可能令人不知所措，特别难以知道在设计抽象时应该选择哪条道路。JavaScript 能够适应许多不同的范式是它作为一种编程语言的优势之一，但这也可能使我们的工作更加困难。JavaScript 程序员需要实现自己的范式。

在希望使事情变得不那么复杂的这一章中，我们将介绍各种众所周知的原则，并将它们分解，以便我们可以看到它们的基本意图。我们将探讨这些原则如何与我们已经讨论过的清晰代码的原则相关联，使我们能够做出自己的明智决定，以便在追求清晰代码时使用什么方法。

我们将涵盖面向对象和函数式编程原则。通过探索这些原则范围，我们将能够为自己制定一张指导思想的地图，这将使我们能够批判性地思考如何在我们所从事的任何范式中编写清晰的代码。

在本章中，我们将涵盖以下主题：

+   **迪米特法则**（**LoD**）

+   SOLID

+   抽象原则

+   函数式编程原则

# 迪米特法则

在我们深入探讨 SOLID 领域之前，探索一个不太知名的原则是很有用的，即所谓的迪米特法则，或者最少知识原则。这个所谓的法则有三个核心思想：

+   一个单元应该只对其他单元有有限的了解

+   一个单元只应该和它的直接朋友交谈

+   一个单元不应该和陌生人交谈

你可能会想知道一个单元*与陌生人交谈*是什么意思。在这种情况下，一个单元是一个特定的编码抽象：可能是一个函数、一个模块或一个类。这里的*交谈*意味着*接口*，比如调用另一个模块的代码或让另一个模块调用你的代码。

这是一个非常有用且简单的法则，可以应用于我们所有的编程，无论是编写一行代码还是设计整个架构。然而，它经常被遗忘或忽视。

让我们以在商店购物的简单行为为例。我们可以用`顾客`和`店主`的抽象来表达这种互动：

```js
class Customer {}
class Shopkeeper {}
```

我们还可以说`顾客`类有一个钱包，他们在里面存放他们的钱：

```js
class Customer {
  constructor() {
    this.wallet = new CustomerWallet();
  }
}

class CustomerWallet {
  constructor() {
    this.amount = 0;
  }
  addMoney(deposit) {
    this.amount += deposit;
  }
  takeMoney(debit) {
    this.amount -= debit;
  }
}
```

`店主`和`顾客`之间的简化交互版本可能是这样的：

```js
class Shopkeeper {
  processPurchase(product, customer) {
    const price = product.price();
    customer.wallet.takeMoney(price);
    // ...
  }
}
```

这看起来可能没问题，但让我们考虑一下这种互动的现实生活类比。店主从顾客口袋里拿走钱包，然后打开钱包，拿走所需的金额，而不以任何方式直接与顾客互动。

很明显，这在现实生活中永远不会是一种社交上合适的互动，但至关重要的是，店主正在做出超出他们权限范围之外的假设。顾客可能希望使用不同的支付方式，或者甚至可能没有钱包。顾客的支付方式是他们自己的事情。这就是我们所说的*只和朋友交谈*：你只应该与你应该了解的抽象进行接口。这里的店主不应该（也不会）了解顾客的钱包，因此不应该*与之交谈*。

接受这些教训，我们可以按照以下方式编写一个更清晰的抽象：

```js
class Shopkeeper {
  processPurchase(product, customer) {
    const price = product.price();
    customer.requestPayment(price);
    // ...
  }
}
```

现在看起来更合理了。`店主`直接与`顾客`交谈。顾客反过来将*与*他们的`顾客钱包`实例*交谈*，取回所需的金额，然后交给店主。

我们很可能都写过一些违反了迪米特法则的代码。当然，我们编写的代码并不总是像商店老板和顾客之间的互动那样刻意或整洁，但迪米特法则仍然适用。我们可以通过一个典型的 JavaScript 代码来进一步说明这一点，该代码负责通过文档对象模型（DOM）向用户显示消息：

```js
function displayHappyBirthday(name) {
 const container = document.createElement('div');
 container.className = 'message birthday-message';
 container.appendChild(
   document.createTextNode(`Happy Birthday ${name}!`)
 );
 document.body.appendChild(container);
}
```

这是典型和惯用的前端 JavaScript。要在文档中显示`生日`消息，我们首先自己构建字符串，并将其放入文本节点，然后将其附加到具有`message`和`birthday-message`类的`<div>`元素中。然后，我们将这个 DOM 树附加到文档中，以便用户查看。

DOM 是一组 API，使我们能够与解析的 HTML 文档进行交互，通常在浏览器中。DOM 作为一个术语，也用来描述由此解析过程生成的节点树。因此，DOM 树可以从给定的 HTML 文档中派生，但我们也可以构建自己的 DOM 树并自由地操纵它们。

前面的代码是否遵守了迪米特法则？我们的抽象，`displayHappyBirthday`函数，关注的是生日快乐消息的概念，并且直接与 DOM 进行交互。然而，DOM 并不是它的朋友。DOM 是一个实现细节——一个陌生人——在`生日快乐`消息的概念中。`生日快乐`消息的机制不应该需要了解 DOM。因此，适当的做法是构建另一个抽象来连接这两个陌生人：

```js
function displayMessage(message, className) {
  const container = document.createElement('div');
  container.className = `message ${className}`;
  container.appendChild(
    document.createTextNode(message)
  );
  document.body.appendChild(container);
}
```

在这里，我们有一个更通用的`displayMessage`函数，直接与 DOM 进行交互。我们的`displayHappyBirthday`函数可以被改变，使其纯粹与这个`displayMessage`抽象进行交互：

```js
function displayHappyBirthday(name) {
  return displayMessage(
    `Happy Birthday ${name}!`,
    'birthday-message'
  );
}
```

现在可以说这段代码与`displayMessage`的实现更松散地耦合在一起。我们以后可以决定改变我们用来显示消息的确切机制，而不需要改变`displayHappyBirthday`函数。因此，我们增强了代码的可维护性。通过概括一个常见的功能——显示消息，我们还可以使未来的功能更加无缝——例如，显示`新年快乐`的消息：

```js
function displayHappyNewYear(name) {
  return displayMessage(
    `Happy New Year! ${name}`,
    'happy-new-year-message'
  );
}
```

迪米特法则，本质上关注的是我们认为应该与其他抽象进行接口的抽象。它并不提供关于*朋友*或*陌生人*是什么，或者一个单元只能有有限的其他单元知识的指导。这个法则挑战我们为自己定义这些术语，以及我们正在构建的抽象。我们有责任停下来考虑我们的抽象是如何进行接口的，也许我们应该以不同的方式设计它们。

我选择首先写关于这个原则，因为我觉得它是最值得记住和最普遍有用的编写干净代码和干净抽象的工具。

接下来，我们将讨论 SOLID 和其他原则，它们各自以不同的方式补充了迪米特法则。

# SOLID

SOLID 是一组常用的原则，用于构建单个模块或更大的架构。具体来说，它是一个缩写，代表五个特定的面向对象编程设计原则：

+   单一职责原则（SRP）

+   开闭原则

+   里氏替换原则

+   接口隔离原则

+   依赖倒置原则

不必记住这些名字甚至是缩写本身，但这些原则背后的思想是有用的。在本节中，我们将探讨每个原则以及 JavaScript 示例。需要注意的是，虽然 SOLID 主要与面向对象编程有关，但其背后有更深层次的真理，无论你的编程范式如何，都是有用的。

# 单一职责原则

当我们编写代码时，我们不断地构建抽象；在这样做时，我们对构建正确的抽象、以正确的方式划分感兴趣。SRP 通过查看它们的责任来帮助我们弄清楚如何划分这些抽象。

在这种情况下，责任指的是您的抽象所涵盖的目的和关注领域。验证电话号码的函数可以说具有单一责任。然而，同时验证和规范这些带有国家代码的号码的函数可以说具有两个责任。SRP 会告诉我们需要将该抽象拆分为两个独立的抽象。

SRP 的目标是得到高度内聚的代码。内聚性是指抽象的各个部分在某种方式上都功能统一，它们可以说都共同工作以实现抽象的目的。关于识别单一责任的一个有用问题是：*您的抽象设计有多少个原因需要更改*？

我们可以通过一个例子来探讨这个问题。假设我们的任务是构建一个小型日历应用程序。最初，我们可能想象这里有两个不同的抽象：

```js
class Calendar {}
class Event {}
```

“事件”类可以说包含有关事件的时间和元信息，“日历”类可以说包含事件。基本的起始前提是您可以向“日历”实例添加和删除一个或多个“事件”实例。在这里，我们表达了用于向“日历”添加和删除事件的方法：

```js
class Calendar {
  addEvent(event) {...}
  removeEvent(event) {...}
}
```

随着时间的推移，我们必须向我们的“日历”添加各种其他功能，例如检索特定日期内的事件的方法，以及以各种格式导出事件的方法：

```js
class Calendar {

  addEvent(event) {...}
  removeEvent(event) {...}
  getEventsBetween(stateDate, endDate) {...}

  setTimeOfEvent(event, startTime, endTime) {...}
  setTitleOfEvent(event, title) {...}

  exportFilteredEventsToXML(filter) {...}
  exportFilteredEventsToJSON(filter) {...}

}
```

即使没有实现，您也可以看到所有这些方法的添加已经创建了一个更加复杂的类。从技术上讲，所有这些方法都与日历的功能相关，因此有理由让它们保留在一个抽象中，但是如果我们回到我们提出的问题——*我们的抽象设计有多少个原因需要更改？*我们可以看到“日历”类现在有很多可能的原因：

+   事件上定义的时间可能需要更改

+   事件上定义的标题的方式可能需要更改

+   搜索事件的方式可能需要更改

+   XML 模式可能需要更改

+   JSON 模式可能需要更改

鉴于潜在变更的不同原因的数量，将变更分割为更合适的抽象是有意义的。例如，特定事件的时间和标题设置方法（`setTimeOfEvent`，`setTitleOfEvent`）绝对应该在“事件”类本身内部，因为它们与“事件”类的目的高度相关：包含有关特定事件的详细信息。导出到 JSON 和 XML 的方法也应该移动，可能移到一个专门负责导出逻辑的类中。以下代码显示了我们所做的更改：

```js
class Event {
  setTime(startTime, endTime) {...}
  setTitle(title) {...}
}

class Calendar {
  addEvent(event) {...}
  removeEvent(event) {...}
  getEventsBetween(stateDate, endDate) {...}
}

class CalendarExporter {
  exportFilteredEventsToXML(filter) {...}
  exportFilteredEventsToJSON(filter) {...}
}
```

希望你能看到，我们每个抽象都内部紧凑，并且每个抽象都比如果所有功能都仅驻留在“日历”类中要更紧凑地封装其责任。

SRP 不仅仅是关于创建易于使用和维护的抽象，它还允许我们编写更专注于其主要目的的代码。以这种方式更加专注使我们更清晰地优化和测试我们的代码单元，这有利于我们代码库的可靠性和效率。由 SRP 指导的内聚抽象的正确划分可能是您改进代码清晰度的最重要方式之一。

# 开闭原则

**开闭原则**（OCP）陈述如下：

*软件实体（类、模块、函数等）应该对扩展开放，但对修改关闭

-梅耶，伯特兰（1988 年）

在构建抽象时，我们应该使它们能够开放以便其他开发人员可以构建其行为，使抽象适应他们的需求。 在这种情况下，扩展最好被认为是一个广义术语，包括所有类型的适应。 如果一个模块或函数的行为不符合我们的要求，最好能够使其适应我们的需求，而无需修改它或创建我们自己的替代品。

考虑我们的日历应用程序中的`Event`类和`renderNotification`方法：

```js
class Event {

  renderNotification() {
    return `
      You have an event occurring in
      ${this.calcMinutesUntil()} minutes!
    `;
  }

  // ...

}
```

我们可能希望有一种单独的事件类型，它会渲染一个以“紧急！”开头的通知，以确保用户更加关注它。 实现这种适应的最简单方法是通过继承`Event`类：

```js
class ImportantEvent extends Event {
  renderNotification() {
    return `Urgent! ${super.renderNotification()}`;
  }
}
```

我们通过覆盖`renderNotification`方法来给我们紧急消息加上前缀，并调用超类的`renderNotification`来填充通知字符串的其余部分。 在这里，通过继承，我们已经实现了扩展，使`Event`类适应我们的需求。

继承只是实现扩展的一种方式。 我们可以采取各种其他方法。 一种可能性是，在最初实现`Event`时，我们预见到需要自定义通知字符串，并实现了一种配置`renderCustomNotifcation`函数的方法：

```js
class Event {

  renderNotification() {
    const defaultNotification = `
      You have an event occurring in
      ${this.calcMinutesUntil()} minutes!
    `;
    return (
      this.config.renderCustomNotification
        ? this.config.renderCustomNotification(defaultNotification)
        : defaultNotification
    );
  }

  // ...

}
```

这段代码假设有一个`config`对象可用。 我们可以选择调用`renderCustomNotification`并传递默认通知字符串。 如果没有配置，则仍然使用默认通知字符串。 这与继承方法有根本的不同，因为`Event`类本身规定了可能的扩展。

通过配置提供适应性意味着用户无需担心在扩展类时所需的内部实现知识。 适应的路径变得简化：

```js
new Event({
  title: 'Doctor Appointment',
  config: {
    renderCustomNotification: defaultNotification => {
      return `Urgent! ${defaultNotifcation}`;
    }
  }
});
```

这种方法要求您的实现能够预见其最可能的适应性，并且这些适应性可预测地内部化到抽象本身。 但是，不可能预见所有需求，即使我们试图这样做，我们可能最终会创建一个如此复杂和庞大的配置，以至于用户和维护人员会受到影响。 因此，在这里需要取得平衡：适应性是一件好事，但我们也必须平衡它与一个有限目的的专注和连贯的抽象。

# 里斯科夫替换原则

**里斯科夫替换原则**规定类型应能够被其子类型替换，而不会改变程序的可靠性。 这在表面上是一个晦涩的原则，因此值得用现实世界的类比来解释它。

许多现实世界的技术创新都具有替代的特点。 沃尔沃 XC90 是一种汽车，福特福克斯也是。 两者都提供了我们期望的汽车的常见接口。 对于我们作为这些车辆的人类用户，我们可以假设它们各自的操作方式都继承自一种常见的车辆操作模式，比如有方向盘，门，刹车踏板等。

人类的假设是，这两种车型是超类型 c*ar*的子类型，因此作为人类，我可以依赖于它们各自从超类型（汽车）继承的方面。 另一种表达里斯科夫替换原则的方式是：*类型的使用者只应关注操作它所需的最不具体的类型*。 举个例子，如果我们要在代码中编写一个`Driver`抽象，我们希望它通常与所有`Car`抽象进行接口，而不是编写依赖于特定车型（如沃尔沃 XC90）的特定代码。

为了使里氏替换原则更具体一些，让我们回到我们的`Calendar`应用程序示例中。在前一节关于开闭原则的部分，我们探讨了如何通过继承将`Event`类扩展为新的`ImportantEvent`类：

```js
class ImportantEvent extends Event {
  renderNotification() {
    return `Urgent! ${super.renderNotification()}`;
  }
}
```

我们这样做的假设是，我们的`Calendar`类及其实现不会关心事件是`Events`还是`Events`的子类。我们期望它会对待它们一样。例如，`Calendar`类可能有一个`notifyUpcomingEvents`方法，它会遍历所有即将发生的事件，并在每个事件上调用`renderNotification`：

```js
class Calendar {

  getEventsWithinMinutes(minutes) {
    return this.events.filter(event => {
      return event.startsWithinMinutes(minutes);
    });
  }

  notifiyUpcomingEvents() {
    this.getEventsWithinMinutes(10).forEach(event => {
      this.sendNotification(
        event.renderNotification()
      );
    });
  }

  // ...
}
```

关键的是，`Calendar`的实现并不考虑它正在处理的`Event`实例的类型。事实上，前面的代码（并未涵盖整个实现）只规定了事件对象必须具有`startsWithinMinutes`方法和`renderNotification`方法。

与里氏替换原则相关的是我们已经讨论过的一个概念：最少知识原则（LoD），它驱使我们问：这个抽象需要多少信息才能实现其目的？对于`Calendar`来说，它只需要规定事件具有它将使用的方法和属性。没有理由让它做出更多的考虑。`Calendar`的实现现在不仅可以处理事件的子类，还可以处理任何提供所需属性和方法的对象。

# 接口隔离原则

**接口隔离原则**关注的是保持接口高度内聚，只从事一个任务或一组高度相关的任务。它规定*不应该强迫客户端依赖它们不使用的方法*。

这个原则在精神上类似于单一责任原则：它的目标是确保您创建专注且高度内聚的抽象，只关注一个责任领域。但它不是让您考虑责任本身的概念，而是让您看待您正在创建的接口，并考虑它们是否适当地隔离。

考虑一个地方政府办公室。他们有一张纸质表格（让我们称之为 1A 表），用于更改个人信息。这是一张存在了 50 多年的表格。通过这张表格，当地公民可以更改他们的一些信息，包括但不限于以下内容：

+   姓名变更

+   婚姻状况变更

+   地址变更

+   地方税折扣状态变更（学生/老年人）

+   残疾状态变更

正如你可以想象的那样，这是一张非常复杂和密集的表格，有许多独立的部分，公民必须确保填写正确。我们都可能接触过政府文书工作的官僚复杂性，如下所示：

![](img/902a31b4-da7b-4b0f-a33e-d522f264f486.png)

1A 表提供了一组接口，用于地方政府办公室提供的各种更改功能。接口隔离原则要求我们考虑这张表格是否强迫其客户端依赖他们不使用的方法。在这个上下文中，客户端是表格的用户，也就是公民，而方法是表格提供的所有可用功能：注册姓名更改、地址更改等等。

显而易见，1A 表单并没有很好地遵循接口隔离原则。如果我们重新设计它，我们可能会将其服务的各个功能分离成独立的表单。我们可以通过使用我们在本章开头学到的东西来做到这一点：最少信息原则（LoD），它向我们提出一个非常简单的问题：每个抽象（例如，更改地址）需要的最少信息是什么？然后我们可以选择只在每个表单中包含完成其功能所需的内容：

![](img/1d39fbfe-ed00-4fcf-bf89-f089d45f1c06.png)

将纸质表单中必要的字段分离出来，可能看起来很明显，但程序员在其编码抽象中经常忽视有效地执行这一点。接口隔离原则提醒我们正确地将抽象分离成独立和内部一致的接口的重要性。这样做有以下好处：

+   **增强可靠性**：拥有真正解耦的正确隔离接口使得代码更易于测试和验证，从而有助于其长期的可靠性和稳定性。

+   **增强可维护性**：拥有分离的接口意味着对一个接口的更改不需要影响其他接口。正如我们在 1A 表单的布局中所看到的，位置和可用空间严重依赖于表单的每个部分。然而，一旦我们解耦了这些依赖关系，我们就可以自由地维护和更改每一个部分，而不用担心其他部分。

+   **增强可用性**：拥有根据目的和功能分离的接口意味着用户能够以更少的时间和认知努力理解和浏览接口。用户是我们接口的消费者，因此最依赖接口清晰地划分。

# 依赖倒置原则

依赖倒置原则陈述如下：

+   高层模块不应该依赖于低层模块。两者都应该依赖于抽象（即接口）

+   抽象不应该依赖于细节。细节（如具体实现）应该依赖于抽象

第一点可能会让你想起 LoD。它在很大程度上谈论了相同的概念：高级与低级的分离。

我们的抽象应该被分离（解耦），以便我们可以在以后轻松更改低级实现细节，而无需重构所有代码。依赖倒置原则在其第二点中建议我们通过中间抽象来实现这一点，通过这些中间抽象，高级模块可以与低级细节进行接口。这些中间抽象有时被称为适配器，因为它们适应了低级抽象，以便高级抽象可以使用。

为什么叫做依赖倒置？高级模块最初可能依赖于低级模块。在提供面向对象编程概念（如抽象类和接口）的语言中，比如 Java，可以说低级模块最终可能依赖于接口，因为接口提供了低级模块实现的脚手架。高级模块也依赖于这个接口，以便可以利用低级模块。我们可以看到这里依赖关系是倒置的，以便高级和低级模块都依赖于同一个接口。

再次考虑我们的“日历”应用程序，假设我们想要实现一种方法来检索固定位置周围发生的事件。我们可以选择实现一个类似这样的方法：

```js
class Calendar {

  getEventsAtLocation(targetLocation, kilometerRadius) {

    const geocoder = new GeoCoder();
    const distanceCalc = new DistanceCalculator();

    return this.events.filter(event => {

      const eventLocation = event.location.address
        ? geocoder.geocode(event.location.address)
        : event.location.coords;

      return distanceCalc.haversineFormulaDistance(
        eventLocation,
        targetLocation
      ) <= kilometerRadius / 1000;

    });

  }

  // ... 

}
```

`getEventsAtLocation`方法负责检索距离给定位置一定半径（以公里为单位）内的事件。如您所见，它使用`GeoCoder`类和`DistanceCalculator`类来实现其目的。

`Calendar`类是一个高级抽象，涉及日历及其事件的广泛概念。然而，`getEventsAtLocation`方法包含了许多与位置相关的细节，更多地是低级关注。这里的`Calendar`类关注于在`DistanceCalculator`上使用哪个公式以及计算中使用的测量单位。例如，您可以看到`kilometerRadius`参数必须除以`1000`才能得到以米为单位的距离，然后与`haversineFormulaDistance`方法返回的距离进行比较。

所有这些细节都不应该是高级抽象，如`Calendar`类的业务。依赖倒置原则要求我们考虑如何将这些关注点抽象到一个中间抽象中，作为高级和低级之间的桥梁。我们可以通过一个新类`EventLocationCalculator`来实现这一点：

```js
const distanceCalculator = new DistanceCalculator();
const geocoder = new GeoCoder();
const METRES_IN_KM = 1000;

class EventLocationCalculator {
  constructor(event) {
    this.event = event;
  }

  getCoords() {
    return this.event.location.address
      ? geocoder.geocode(this.event.location.address)
      : this.event.location.coords
  }

  calculateDistanceInKilometers(targetLocation) {
    return distanceCalculator.haversineFormulaDistance(
      this.getCoords(),
      targetLocation
    ) / METRES_IN_KM;
  }
}
```

然后，`Event`类可以在其自己的`isEventWithinRadiusOf`方法中利用这个类。以下代码展示了这一示例：

```js
class Event {

  constructor() {
    // ...
    this.locationCalculator = new EventLocationCalculator();
  }

  isEventWithinRadiusOf(targetLocation, kilometerRadius) {
    return locationCalculator.calculateDistanceInKilometers(
      targetLocation
    ) <= kilometerRadius;
  }

  // ...

}
```

因此，`Calendar`类只需要关注`Event`实例具有`isEventWithinRadiusOf`方法这一事实。它不需要任何信息，也不对确定距离的具体实现做出规定；这些细节留给我们的低级`EventLocationCalculator`类：

```js
class Calendar {

  getEventsAtLocation(targetLocation, kilometerRadius) {
    return this.events.filter(event => {
      return event.isEventWithinRadiusOf(
        targetLocation,
        kilometerRadius
      );
    });
  }

  // ...

}
```

依赖倒置原则类似于与抽象界面分离原则相关的其他原则，但它特别关注依赖关系以及这些依赖关系的指向。当我们设计和构建抽象时，我们隐含地建立了一个依赖图。例如，如果我们要绘制出我们得到的实现的依赖关系，那么它看起来会像这样：

![](img/a504d714-ae69-4ef8-a0d2-4c5ffdc9c8c3.png)

绘制此类依赖图非常有用。它们是探索代码真正复杂性的有用方式，通常可以突出可能改进的领域。最重要的是，它们让我们观察到我们的低级实现（细节）在何处，如果有的话，影响了我们的高级抽象。只有当我们看到这种情况时，我们才能加以纠正。因此，在阅读本书及以后的过程中，始终要在脑海中有一个依赖关系图；这将有助于引导您编写更脱耦、更可靠的代码。

依赖倒置原则是 SOLID 首字母缩写中的最后一个，而 SOLID，正如我们所见，主要关注于我们构建抽象的*方式*。我们将要介绍的下一个原则将绑定我们迄今为止所涵盖的大部分内容，因为它就是抽象本身的原则。如果我们从本章中记住的唯一一件事是什么，那么我们至少应该记住**抽象原则**。

# 抽象原则

在第一章中，我们介绍了抽象塔的概念，以及每个抽象都是隐藏复杂性的简化杠杆的想法。编程中的抽象原则陈述如下：

实现应该与接口分离。

实现是抽象的复杂底层：它所隐藏的部分。接口是简化的顶层。这就是为什么我们说抽象是隐藏复杂性的简化杠杆。创建将实现与接口分离到恰到好处的抽象的技艺并不像看起来那么简单。因此，编程世界为我们提供了两个警告：

+   **不要重复自己**（**DRY**）：这是一个警告，告诉我们要避免编写重复我们已经编写的代码。如果你发现自己不得不重复自己，那么这表明你未能抽象出某些东西，或者抽象得不够。

+   **你不会需要它**（**YAGNI**）：也被称为**保持简单，愚蠢！**（**KISS**），这个警告告诉我们要警惕过度抽象不需要被抽象的代码。这与 DRY 截然相反，并提醒我们除非有必要（如果我们开始重复自己，也许），我们不应尝试抽象。

在这两个警告之间，中间某处，就是完美的抽象。设计抽象，使其尽可能简单和有用，是一种平衡。一方面，我们可以说我们有过度抽象（DRY 警告我们要注意这一点），另一方面，我们有过度抽象（YAGNI 警告我们要注意这一点）。

# 过度抽象

过度抽象是指已经移除或替换了太多复杂性，以至于底层复杂性变得难以利用。过度抽象的风险在于，我们要么过度简化以求简单，要么添加新的不必要的复杂性，使抽象的用户感到困惑。

例如，假设我们需要一个画廊抽象，我们希望在网站和各种移动应用程序上都能使用它来显示画廊。根据平台的不同，画廊将使用可用的接口来生成布局。在网页上，它将生成 HTML 和 DOM，但在移动应用程序上，它将使用各种可用的本机 UI SDK。抽象为我们提供了所有这些跨平台复杂性的控制杆。

我们对画廊的最初要求非常简单：

+   能够显示一个或多个图像

+   能够在图像旁显示标题

+   能够控制单个图像的尺寸

外部团队为我们创建了一个画廊组件。我们打开文档，看到它有以下示例代码，向我们展示如何创建一个包含两张图片的画廊：

```js
const gallery = new GalleryComponent(
  [
    new GalleryComponentImage(
      new GalleryComponentImage.PathOfImage('JPEG', '/foo/images/Picture1.jpg'),
      new GalleryComponentImage.Options({
        imageDimensionWidth: { unit: 'px', amount: 200 },
        imageDimensionHeight: { unit: 'px', amount: 150 },
        customStyleStrings: ['border::yellow::1px']
      }),
      [
        new GalleryComponentImage.SubBorderCaptionElementWithText({
          content: { e: 'paragraph', t: 'The caption for this employee' }
        })
      ]
    }),
    new GalleryComponentImage(
      new GalleryComponentImage.PathOfImage('JPEG', '/foo/images/Picture2.jpg'),
      new GalleryComponentImage.Options({
        imageDimensionWidth: { unit: 'px', amount: 200 },
        imageDimensionHeight: { unit: 'px', amount: 150 },
        customStyleStrings: ['border::yellow::1px']
      }),
      [
        new GalleryComponentImage.SubBorderCaptionElementWithText({
          content: { e: 'paragraph', t: 'The caption for this employee' }
        })
      ]
    })
  ]
);
```

这个接口对于只显示几张图片的基本目的来说似乎非常复杂。考虑到我们简单的需求，我们可以说前面的接口是过度抽象的证据：它没有简化底层复杂性，而是引入了一整套我们甚至不需要的新的复杂性和各种功能。它在技术上满足了我们的要求，但我们必须在其复杂的领域中导航才能实现我们想要的东西。

这样的抽象，编码了新的复杂性并规定了自己的特性和命名约定，有可能不仅无法减少复杂性，还可能增加复杂性！抽象不应增加复杂性；这与抽象的整个目的背道而驰。

请记住，适当的抽象级别取决于上下文。对于您的用例来说可能是过度抽象的，对于另一个用例来说可能是不足抽象的。F1 赛车手对引擎的抽象级别与福特福克斯车手不同。抽象，像许多清洁代码概念一样，取决于受众和用户。

过度抽象还可以奇怪地采用过度简化的形式，其中对底层复杂性的控制杆并不向我们开放。我们的`GalleryComponent`接口的过度简化版本可能如下所示：

```js
const gallery = new GalleryComponent(
  '/foo/images/PictureOne.jpg',
  '/foo/images/PictureTwo.jpg'
);
```

这种最小接口可能看起来与以前的代码完全相反，在某些方面是这样，但奇怪的是，它也是过度抽象的一个例子。记住，抽象是指通过接口提供对底层复杂性的杠杆。在这种情况下，这个杠杆太简单了，只提供了非常有限的复杂性，我们希望利用这种复杂性。它不允许我们添加标题或控制图像尺寸；它只允许我们列出一组图像，什么都不能做。

通过前面两个例子，你已经看到过度抽象可以有两种不同的风格：一种是过于复杂，一种是过于简化。这两种都是不受欢迎的。

# 低抽象

如果过度抽象是指*过多*的复杂性已被移除或替换，那么低抽象是指*过少*的复杂性已被移除或替换。这导致用户需要关注底层复杂性。想象一下，你有一辆汽车，你必须在没有方向盘或仪表盘的情况下驾驶它。你必须直接通过引擎来控制它，用你的双手拉动杠杆和转动油腻的齿轮，同时注意路况。我们可以说这辆车有一种低抽象的控制方法。

我们探讨了我们的画廊组件的过度抽象版本，那么让我们看看低抽象版本可能是什么样子：

```js
const gallery = new GalleryComponent({
  web: [
    () => {
      const el = document.createElement('div');
      el.className = 'gallery-container';
      return el;
    },
    {
      data: [
        `<img src="/foo/images/PictureOne.jpg" width=200 height=150 />
         <span>The caption</span>`,
        `<img src="/foo/images/PictureTwo.jpg" width=200 height=150 />
         <span>The caption</span>`
       ]
    }
  ],
  android: [
    (view, galleryPrepData) => {
      view.setHasFixedSize(true);
      view.setLayoutManager(new GridLayoutManager(getApplicationContext(),2));
      return new MyAdapter(getApplicationContext(), galleryPrepData());
    },
    {
      data: [
        ['/foo/images/PictureOne.jpg', 200, 150, 'The Caption']
        ['/foo/images/PictureTwo.jpg', 200, 150, 'The Caption']
      ]
    }
  ]
});
```

这个`GalleryComponent`的版本似乎在强迫我们定义特定于 Web 的 HTML 和特定于 Android 的代码。理想情况下，我们依赖于抽象来隐藏这种复杂性，给我们一个简化的接口来利用它，但它没有做到。在这里，编写特定于平台的代码的复杂性在这里没有得到足够的抽象，因此我们可以说这是一个低抽象的例子。

从前面的代码中，你也可以看到我们被迫重复我们图像的源 URL 和标题文本。这应该提醒我们之前探讨过的一个警告：DRY，它表明我们没有充分抽象出某些东西。

如果我们留意那些被迫重复自己的领域，那么我们可以希望构建更好的抽象。但要注意，低抽象并不总是显而易见。

可以说各种抽象都是*泄漏的抽象*，因为它们通过它们的接口向上*泄漏*它们的一部分复杂性。前面的代码就是一个例子：我们可以说它正在向上泄漏其跨平台复杂性的实现细节。

# 平衡的抽象

根据我们对低抽象和过度抽象的了解，我们可以说平衡的抽象是坐落在这两种不受欢迎的对立面之间的抽象。创建平衡的抽象的技巧既是一门艺术，也是一门科学，需要我们对问题领域和用户的能力和意图有很好的理解。通过运用本章中的许多原则和警告，我们可以希望在我们的代码构建中保持平衡。对于`GalleryComponent`的上一个例子，我们应该再次探索抽象的要求：

+   显示一个或多个图像的能力

+   显示图像旁边的标题的能力

+   控制各个图像的尺寸的能力

这些，我们可以说，是我们必须提供给底层跨平台复杂性的*杠杆*。我们的抽象应该只旨在暴露这些杠杆，而不是其他不必要的复杂性。以下是这种抽象的一个例子：

```js
const gallery = new GalleryComponent([
  {
    src: '/foo/images/PictureOne.jpg',
    caption: 'The Caption',
    width: 200,
    height: 150
  },
  {
    src: '/foo/images/PictureTwo.jpg',
    caption: 'The Caption',
    width: 200,
    height: 150
  },
]);
```

通过这个接口，我们可以定义一个或多个图像，设置它们的尺寸，并为每个图像定义标题。它满足了所有的要求，而不会引入新的复杂性或从底层实现中泄漏复杂性。因此，这是一个平衡的抽象。

# 功能性编程原则

JavaScript 允许我们以多种不同的方式进行编程。到目前为止，在本书中分享的许多示例更倾向于面向对象编程，它主要使用对象来表达问题领域。函数式编程不同之处在于，它主要使用纯函数和不可变数据来表达问题领域。

所有编程范式都广泛关注同一件事：使表达问题领域更容易，作为程序员传达我们的意图，并容纳有用和可用的抽象的创建。我们从一种范式中采纳的最佳原则可能仍然适用于另一种范式，因此要采取开放的态度！

通过探索一个示例，最容易观察和讨论面向对象编程和函数式编程之间的差异。假设我们希望构建一个机制，以便我们可以从服务器获取分页数据。为了以面向对象的方式实现这一点，我们可能会创建一个`PaginatedDataFetcher`类：

```js
// An OOP approach

class PaginatedDataFetcher {

  constructor(endpoint, startingPage) {
    this.endpoint = endpoint;
    this.nextPage = startingPage || 1;
  }

  getNextPage() {
    const response = fetch(
      `/api/${this.endpoint}/${this.nextPage}`
    );
    this.nextPage++;
    return fetched;
  }

}
```

下面是一个使用`PaginatedDataFetcher`类的示例：

```js
const pageFetcher = new PaginatedDataFetcher('account_data', 30);

await pageFetcher.getNextPage(); // => Fetches /api/account_data/30
await pageFetcher.getNextPage(); // => Fetches /api/account_data/31
await pageFetcher.getNextPage(); // => Fetches /api/account_data/32
```

如您所见，每次调用`getNextPage`时，我们都会检索下一页的数据。`getNextPage`方法依赖于其对象的记忆状态，`endpoint`和`nextPage`，以了解下一个要请求的 URL。

**状态**可以被视为任何程序或代码片段的基础记忆数据，其结果或效果是由此派生的。汽车的状态可能意味着它的当前保养情况，燃油和机油水平等。同样，运行程序的状态是它从中派生功能的基础数据。

函数式编程与面向对象编程有所不同，它纯粹关注函数的使用和不可变状态来实现其目标。人们在探索函数式编程时通常遇到的第一个心理障碍与状态有关，引发了诸如“我应该把状态存储在哪里？”和“如何使事物改变而无法改变状态？”等问题。我们可以通过查看分页数据获取器的函数式编程等价物来探讨这个问题。

我们创建了一个函数`getPage`，我们将传递一个`endpoint`和一个`pageNumber`，如下所示：

```js
// A more functional approach

const getPage = async (endpoint, pageNumber = 1) => ({
 endpoint,
 pageNumber,
 response: await fetch(`/api/${endpoint}/${pageNumber}`)
 next: () => getPage(endpoint, pageNumber + 1)
});
```

调用`getPage`函数时，将返回一个包含来自服务器的响应以及使用的`endpoint`和`pageNumber`的对象。除了这些属性之外，该对象还将包含一个名为`next`的函数，如果调用，将通过随后调用`getPage`来触发另一个请求。可以按以下方式使用它：

```js
const page1 = await getPage('account_data');
const page2 = await page1.next();
const page3 = await page2.next();
const page4 = await page3.next();

// Etc.
```

您会注意到，使用这种模式时，我们只需要引用最后检索到的页面，即可进行下一次请求。通过页面 2 返回的`next()`函数检索页面 3。通过页面 3 返回的`next()`函数检索页面 4。

我们的`getPage`函数不会改变任何数据：它只使用传递的数据来派生新数据，因此可以说它采用了不可变性。还可以说它也是一个纯函数，因为对于给定的输入参数（`endpoint`和`pageNumber`），它将始终返回相同的结果。`getPage`返回的`next`函数也是纯的，因为它将始终返回相同的结果：如果我调用`page2.next()`一百万次，它将始终获取`page 3`。

函数纯度和不可变性是最重要的函数式概念之一，而且，值得注意的是，这些原则适用于所有编程范式。我们并不打算在这里深入探讨函数式编程，而只是涵盖其最适用的原则，以增强我们的抽象构建能力。

# 函数纯度

当函数的返回值仅从其输入值派生时（也称为**幂等性**），并且没有副作用时，可以说函数是纯的。这些特征给我们带来了以下好处：

+   **可预测性**：不会对程序的其他部分产生副作用的函数是可以轻松推理的函数。如果一个函数改变了它不拥有的状态，可能会在代码的其他部分创建级联的变化，这可能会非常复杂，从而产生维护和可靠性问题。

+   **可测试性**：纯函数总是在给定相同的输入时返回相同的结果，因此非常容易验证。纯函数可能变得复杂，但如果保持纯净，它们将始终易于测试。

幂等性是指在提供特定输入时总是得出相同结果的特性。因此，幂等函数是高度确定性的。幂等函数可能仍然具有副作用，因此它可能并不总是一个纯函数，但从抽象用户的角度来看，幂等性是非常可取的，因为这意味着我们总是知道可以期望什么。

在面向对象编程中，对象上的方法通常不能说是纯的，因为它们会改变状态（在对象上）或在相同的输入参数下返回不同的结果。例如，考虑以下`Adder`类：

```js
class Adder {
  constructor() {
    this.total = 0;
  }
  add(n) {
    return this.total += n;
  }
}

const adder = new Adder();
adder.add(10); // => 10
adder.add(10); // => 20
adder.add(5);  // => 25
```

`add`方法在这里不是纯的。即使给定相同的参数，它返回不同的结果，并且具有副作用：改变它不拥有的状态（即对象的 total 属性）。我们可以很简单地创建一个功能纯粹的加法抽象：

```js
const add = (a, b) => a + b;

add(10, 10); // => 20
add(10, 20); // => 30
```

这可能看起来有些牵强，但功能纯度背后的概念是，从复杂的需求中得出真正纯粹的基本原语和函数，以构造它所需的。功能纯度在这里教给我们一个一般性的教训：将功能分解为其最原始的部分，直到你拥有一个真正可测试的独立单元。然后我们可以将这些较小的单元组合成更复杂的单元，以执行更复杂的工作。

# 不可变性

这一章主要讨论了我们如何构建和分离抽象，但同样重要的是考虑数据在这些抽象之间传递的期望。

不可变性是指数据不应该发生变化的简单想法。这意味着，当我们初始化一个对象时，例如，我们不应该向其添加新属性或随时间更改现有属性。相反，我们应该派生一个全新的对象，只对我们自己的副本进行更改。不可变性是数据的特征，也是函数式编程的一般原则。语言也可以通过不允许已声明的变量或对象的变异来强制不可变性。JavaScript 的`const`就是这种强制的一个例子：

```js
const name = 'James';
name = 'Samuel L. Jackson';
// => Uncaught TypeError: Assignment to constant variable.
```

知道某物是不可变的意味着我们可以放心地知道它不会改变；我们可以依赖它的特性，而不必担心程序的其他部分可能在我们不知情的情况下改变它。这在 JavaScript 的异步世界中尤为重要，数据以复杂的方式在作用域和接口之间穿梭。

与本章涵盖的许多原则一样，不可变性并不一定要严格遵循才能从中获得好处。在某些领域保持不可变性，在其他领域保持可变性，可能是一种可行的方法。想象一份官方文件在政府大楼中穿梭。每个部门都默认文件没有被意外的人任意修改；特定部门可能选择复制文件，然后对其自己的副本进行各种变更，以满足自己独特的目的。代码库与此并无二致。通过构建抽象并让它们相互依赖，我们有意地使它们能够操纵彼此的数据和功能。

# 总结

在本章中，我们涵盖了大量的理论和实践技能。我们涵盖了 LoD（或最少信息原则）、所有 SOLID 原则、抽象原则，以及函数式编程范式中的一些关键原则。即使你不记得所有的名字，希望你能记住每个原则所包含的基础知识和关键教训。

编程既是一门艺术，也是一门科学。它涉及在追求真正平衡的抽象时平衡所有这些原则。这些原则都不应被视为硬性规则。它们只是指导方针，将在我们的旅程中帮助我们。

在下一章中，我们将继续探讨编程中最具挑战性的一个方面，无论是在 JavaScript 中还是在其他地方：命名事物的问题。
