# 第三章：使用预处理器和提供者扩展 Knockout

在上一章中，我们探讨了向 Knockout 添加自定义绑定处理器以添加功能和将它们与第三方工具集成的技术。这项功能是 Knockout 首次发布时就有的，它允许对 Knockout 的功能进行强大的扩展。在本章中，我们将探讨一些更高级的扩展或甚至改变 Knockout 绑定行为的技术。你将学习如何创建：

+   绑定处理器预处理程序

+   节点预处理程序

+   绑定提供者

在我们介绍完这些之后，我们将看看 Knockout Punches 库，这是一个由 Knockout 开发者 Michael Best 提供的预处理器和扩展集合。

# 绑定处理器预处理

到目前为止，我们已经探讨了绑定处理器的两个属性：`init`和`update`函数。绑定处理器还有一个可选的函数，即`preprocess`，它在`init`函数之前运行。预处理器的目的是在 Knockout 确定要应用哪些绑定之前修改`data-binding`属性。

预处理程序不处理元素或绑定上下文；它们只处理绑定将评估的字符串。例如，如果我们有一个将所有文本绑定转换为大写的预处理程序，那么以下`span`元素将被处理：

```js
<span data-bind="text: 'That Guy'"></span>
```

这个`span`元素将被处理成这样：

```js
<span data-bind="text: 'That Guy'.toUpperCase()"></span>
```

如果你在这个之后检查 HTML，你仍然会看到原始的`data-bind`属性。这是因为预处理程序实际上并不处理元素；它们只是在正常绑定处理器应用之前修改绑定字符串。

## 创建预处理程序

添加一个预处理器的操作就像给绑定处理器添加一个`preprocess`属性一样简单，就像我们添加了`init`和`update`函数：

```js
ko.bindingHandlers.thing.preprocess = function(value, name, addBinding) {
    //Do stuff
}
```

`preprocess`函数的三个参数如下：

+   `value`: 这是绑定处理器给出的表达式。例如，在`text: name`中，值是`name`；在`text: title() + '. ' + name()`中，值是`"title() + '. ' + name()"`。这个值始终是一个字符串。

+   `name`: 这是绑定处理器的名称，例如，`text`或`click`。这在多个绑定处理器使用单个`preprocess`函数的情况下非常有用。

+   `addBinding`: 这是一个回调函数，它接受`name`和`value`字符串参数，就像之前的那些一样。它将这个对作为绑定添加到元素上。

预处理程序的返回值将是整个绑定使用的新值。

让我们看看几个例子。

## 大写预处理程序

Knockout 文档提供了一个关于这个预处理器的示例，在撰写本文时，它返回`value + ".toUpperCase()"`。完整的预处理程序将如下所示：

```js
ko.bindingHandlers.text.preprocess = function(value) {
  return value + '.toUpperCase()';
};
```

上述代码在例如本节开头直接取字符串时将有效。

```js
<span data-bind="text: 'That Guy'"></span>
```

我们预处理器的结果将是 `text: 'That Guy'.toUpperCase()`，并且文本绑定将无错误地处理这个结果。不幸的是，这将在绑定到可观察属性的正常情况下失败：

```js
<span data-bind="text: firstName"></span>
```

Knockout 的正常绑定过程会展开它得到的表达式，这样可观察的属性就不需要括号。另一方面，预处理器只是输出直接由绑定处理器消费的字符串。我们的大写绑定将在这里产生一个非法的结果：

```js
<span data-bind="text: firstName.toUpperCase()"></span>
```

这将失败，因为 `firstName` 是一个可观察的属性，而不是一个字符串，并且可观察的属性没有 `toUpperCase` 方法。

幸运的是，这个解决方案很简单。我们的预处理器可以通过应用一个 `unwrap` 函数安全地处理所有值表达式：

```js
ko.bindingHandlers.text.preprocess = function(value) {
    return 'ko.unwrap(' + value + ').toUpperCase()';
};
```

这将确保任何值——无论是原始类型、可观察的属性还是内联表达式——都能被绑定处理器正确评估。

你可以在 `cp3-uppercase` 分支中看到这个预处理器的示例。

## 包装现有绑定

由于 Knockout 为大多数标准场景提供了默认绑定，因此通常希望自定义绑定能够建立在它们之上。预处理器使得包装其他绑定变得非常容易。

假设我们想要一个绑定，当属性更新时使元素闪烁，同时在它上面提供 `value` 绑定。通常，你可能会想要将这些分成两个单独的绑定，但如果你经常这样做，一个单独的绑定将节省时间和按键。

由于 `value` 绑定已经存在，我们只需使用预处理器通过 `addBinding` 回调添加绑定即可：

```js
ko.bindingHandlers.valueFlash = {
  preprocess: function(value, name, addBinding) {
      addBinding('value', value);
      return value;
  },
  update: function(element, valueAccessor) {
        ko.unwrap(valueAccessor());  //unwrap to get dependency
        $(element).css({opacity: 0}).animate({opacity: 1}, 500);
    }
};
```

`addBinding` 回调负责生成 `value` 绑定，就像它被正常应用一样，这包括为新绑定（如果有的话）运行预处理器。

在添加 `value` 绑定后，我们仍然需要返回原始值是很重要的。如果 `preprocess` 函数没有返回任何内容，那么原始绑定将被移除。之后，绑定处理器的其余部分就像往常一样：添加所需的 `init` 和 `update` 函数，并编写你的自定义行为。在 `cp3-wrap` 分支中有一个这种绑定的示例。

那就是创建绑定处理器预处理器所需要做的全部。由于它们允许扩展性，它们的使用简单直接。当我们在本章的最后部分查看 `Knockout.Punches` 时，我们将探讨一些更多关于绑定预处理器在现实世界中的可能性。

# 节点预处理器

绑定处理器的预处理器附加到单个绑定处理器上，通过修改绑定字符串来工作。它们仅适用于各自处理器的节点。

相反，节点预处理器在每一个 DOM 节点上都会被调用。它们在 UI 首次绑定时运行，以及当 UI 被绑定如 `foreach` 或 `template` 这样的绑定修改时运行。

节点预处理器的目的是在数据绑定发生之前修改 DOM，与仅修改 `data-bind` 属性的绑定预处理器相反。节点预处理器是通过向绑定提供者添加一个 `preprocessNode` 函数来定义的：

```js
ko.bindingProvider.instance.preprocessNode = function(node) {
  /* DOM code */
}
```

预处理器为每个节点调用一次。如果不需要进行任何更改，它应该返回空值。否则，它可以使用标准的 DOM API 插入新节点或删除当前节点：

+   应该使用以下方式在当前节点之前插入新节点：

    ```js
    node.parentNode.insertBefore(newNode, node);
    ```

+   替换可以通过以下方式完成：

    ```js
    node.parentnode.replaceChild(newNode, node);
    ```

+   可以使用以下方式删除：

    ```js
    node.parentNode.removeChild(node);
    ```

添加的任何节点都需要从 `preprocessNode` 返回；否则，Knockout 不会将绑定应用于它们。由于在 `preprocessNode` 中没有绑定上下文（你只有当前节点），因此无法自行应用绑定，除非它们应用于常量或全局值。然而，这并不推荐，因为它会在当前上下文层次结构之外创建一个新的绑定上下文。

## 关闭虚拟模板节点

Knockout 文档提供了一个方便的节点预处理器，它可以自动关闭虚拟模板绑定。通常，在编写无容器的模板绑定时，你需要两个注释节点：

```js
<!-- template: 'some-template' --><!-- /ko -->
```

由于模板绑定在引用外部模板时永远不会包含内容，因此闭合注释节点看起来是不必要的。一个 `preprocess` 函数将允许你使用模板而不需要闭合标签，这样你可以将绑定写成这样：

```js
<!-- template: 'some-template' -->
```

Knockout 需要一个闭合的注释标签，即 `<!-- /ko -->`，用于虚拟绑定。我们可以通过预处理器自动提供这个注释节点：

```js
ko.bindingProvider.instance.preprocessNode = function(node) {
   if (node.nodeType == node.COMMENT_NODE) {
      var match = node.nodeValue.match(/^\s*(template\s*:[\s\S]+)/);
      if (match) {
         // Create a pair of comments to replace the single comment
         var c1 = document.createComment("ko " + match[1]),
            c2 = document.createComment("/ko");
         node.parentNode.insertBefore(c1, node);
         node.parentNode.replaceChild(c2, node);

         // Tell Knockout about the new nodes so that it can apply bindings to them
         return [c1, c2];
      }
   }
};
```

此示例使用正则表达式来识别模板注释并从绑定中提取表达式。然后，它用虚拟模板绑定的标准开/闭注释对替换原始注释。最后，它返回新的注释节点，允许 Knockout 将它们绑定；这将把模板应用到由注释节点创建的虚拟容器中。

## 支持不同的语法

前面的例子应该已经让你了解了节点预处理器的运作方式。然而，节点预处理器的真正威力在于它让我们能够扩展数据绑定语法本身。

看到一系列文本绑定，如这个例子，并不罕见：

```js
First Name: <!-- text: firstName --><!-- /ko -->
Last Name: <!-- text: lastName  --><!-- /ko -->
Birth Date: <!-- text: birthDate --><!-- /ko -->
```

我们想列出几个属性，但这些虚拟元素相当冗长。除了属性名外，它们还增加了 29 个字符，包括空格。当然，我们也可以使用 `span` 元素，但考虑到它们还需要 `data-bind` 属性以及绑定名称，它们的大小几乎相同。

如果你曾经使用过 AngularJS 或 Handlebars，你可能会欣赏使用花括号以字符串形式访问值的最低要求。前面的例子将看起来像这样：

```js
First Name: {{ firstName }}
Last Name: {{ lastName }}
Birth Date: {{ birthDate }}
```

看看这有多简短，阅读起来有多容易！这些 Handlebars 的人有正确的方法。我相信你知道我们接下来要做什么。一个节点预处理器将允许我们用第一个示例中的 HTML 替换相同的 HTML。

这个示例很长，所以我们将把它分成几部分：

```js
var expressionRegex = /{{([\s\S]+?)}}/g;
ko.bindingProvider.instance.preprocessNode = function(node) {
    if (node.nodeType === 3 && node.nodeValue) {
        var newNodes = //Collect new nodes by scanning "node"

        // Insert the resulting nodes into the DOM
        // remove the original unprocessed node
        if (newNodes) {
            for (var i = 0; i < newNodes.length; i++) {
                node.parentNode.insertBefore(newNodes[i], node);
            }
            node.parentNode.removeChild(node);
            return newNodes;
        }
    }
};
```

首先，我们有一个正则表达式模式，它可以找到这些双大括号块。由于文本节点将包含任何内容，直到它们遇到第一个真实元素，所以单个节点中可能有多个大括号块，因此它需要全局匹配。然后，`preprocess` 函数首先检查文本节点类型。

我现在省略了实际扫描节点以创建新节点的部分；我们稍后会回到这一点。

如果有任何需要添加的节点，它们将被插入，然后删除原始节点。最后，返回我们插入的节点，以便 Knockout 可以绑定它们。

这几乎是节点预处理器的大纲代码，并且是一个非常好的模式。检查类型，创建任何新节点，如果有，替换原始节点，并返回新节点。如果您正在创建节点预处理器，这是一个很好的模板开始。

好吧，让我们进入正题。为了分配 `newNodes`，我们需要检查节点中的正则表达式模式并为每个匹配项构建一对虚拟文本绑定：

```js
var newNodes = replaceExpressionsInText(node.nodeValue, expressionRegex, function(expressionText) {
    return [
        document.createComment("ko text:" + expressionText),
        document.createComment("/ko")
    ];
});
```

在这里，我们调用 `replaceExpressionsInText` 并传递节点内容、我们的正则表达式模式和回调函数，该回调函数使用我们通过正则表达式找到的表达式构建正确的替换项。然后，我们只需要实际的搜索：

```js
function replaceExpressionsInText(text, expressionRegex, callback) {
    var prevIndex = expressionRegex.lastIndex = 0,
        resultNodes = null,
        match;

    while (match = expressionRegex.exec(text)) {
        var leadingText = text.substring(prevIndex, match.index);
        prevIndex = expressionRegex.lastIndex;
        resultNodes = resultNodes || [];

        // Preserve leading text
        if (leadingText) {
            resultNodes.push(document.createTextNode(leadingText));
        }

        resultNodes.push.apply(resultNodes, callback(match[1]));
    }

    // Preserve trailing text
    var trailingText = text.substring(prevIndex);
    if (resultNodes && trailingText) {
        resultNodes.push(document.createTextNode(trailingText));
    }

    return resultNodes;
}
```

搜索函数在正则表达式模式上循环，提取第一个匹配项。它将匹配项发送到回调函数，并保留结果，包括任何前导或尾随空格。当完成匹配后，它返回它们。

就这样。现在，我们的 Handlebars 代码将被转换为虚拟文本绑定。您可以在 `cp3-interpolate` 分支中查看这个示例。

### 注意

这段代码是从 [`blog.stevensanderson.com/2013/07/09/`](http://blog.stevensanderson.com/2013/07/09/) 上的 `StringInterpolatingBindingProvider` 示例改编的。

### 多种语法

如果我们想把这个示例做得更深入，我们可以支持额外的插值语法。`replaceExpressionsInText` 已经设置为接受正则表达式输入，并且因为它使用回调，我们可以为不同的正则表达式模式构造不同的节点。

让我们添加嵌入式 Ruby 语法插值，它使用 `<%= expression %>`：

```js
// Replace <%= expr %> with data bound span's
var erbNodes = replaceExpressionsInText(node.nodeValue, /\<\%=([\s\S]+?)\%\>/g, function(expressionText) {
    var span = document.createElement('span');
    span.setAttribute('data-bind', 'text: ' + expressionText);
    return [span];
});
```

这次，我们正在替换一个 `span` 元素而不是虚拟文本元素，这样我们就可以区分生成的 HTML。因为这个预处理器可以支持两种语法，所以您可以对混合语法的模板进行绑定：

```js
First Name: {{ firstName }}
Last Name: <%= lastName %>
Birth Date: {{ birthDate }}
```

生成的 HTML 将看起来像这样：

```js
First Name: <!--ko text: firstName --><!--/ko-->
Last Name: <span data-bind="text: lastName"></span>
Birth Date: <!--ko text: birthDate --><!--/ko-->
```

您可以在 `cp3-interpolate2` 分支中查看这个示例。

# 绑定提供者

使用绑定预处理程序，我们可以访问绑定表达式并在绑定评估之前对其进行修改。使用节点预处理程序，我们可以访问节点并在绑定应用之前修改 DOM。这两者都只是将事物转换成正常的 Knockout 语法。它们也仅限于在 DOM 上操作，并且无法访问绑定上下文。

Knockout 绑定提供程序是对象，它们接收 DOM 节点和绑定上下文，并确定将应用哪些绑定处理程序以及这些绑定接收哪些`valueAccessor`属性。

预期绑定提供程序提供以下函数：

+   `nodeHasBindings(node)`: 此函数应返回一个布尔值，指示节点是否定义了任何绑定。

+   `getBindingAccessors(node, bindingContext)`: 此函数应返回一个对象，其中每个要应用的绑定都有一个属性，其值是一个评估绑定表达式的函数。此函数用作绑定处理程序中的`valueAccessor`属性。

### 注意

如果你针对的是 2.x 版本，你需要支持`getBindings`函数，该函数返回一个对象，其属性值是最终的绑定值。这个函数在 Knockout 3.0 中被弃用。

默认绑定提供程序通过在元素或以`ko`开头的注释节点上查找`data-bind`属性来操作。如果存在，`nodeHasBindings`将返回`true`。当调用`getBindingAccessors`时，它通过评估`data-bind`属性并从绑定上下文中获取`valueAccessors`属性来返回绑定。

## 自定义绑定提供程序

我们已经看到了如何使用预处理程序允许使用不同的语法进行数据绑定。因此，为了更好地理解绑定提供程序的能力，我们将查看预处理程序无法做到的事情：根据数据类型选择绑定。

Knockout 插件`Knockout.BindingConventions`([`github.com/AndersMalmgren/Knockout.BindingConventions`](https://github.com/AndersMalmgren/Knockout.BindingConventions))创建了一个绑定提供程序，该程序通过查看绑定上下文以获取要使用的绑定线索，在`data-name`属性上提供绑定，因此它是一个自定义提供程序的绝佳示例。由于这与 Knockout 的工作方式有很大不同，让我们将其与标准视图模型和绑定设置进行比较：

```js
var BindingSample = function() {
   var self = this;

   self.name = ko.observable('Timothy');
   self.locations = ['Portland', 'Seattle', 'New York City'];
   self.selectedLocation = ko.observable();
   self.isAdmin = ko.observable(true););
}; 
```

使用标准 Knockout 绑定来绑定此内容可能看起来像这样：

```js
<label>Name
  <input data-bind="value: name" />
</label>
<label>LocationLocationName
  <select data-bind="options: locations, value: selectedLocation"></select>
</label>
<label>Admin
  <input data-bind="checked: isAdmin" type="checkbox" />
</label>
```

我们有三个绑定元素和四个绑定。第一个输入是一个绑定到`name`的`value`，`select`元素绑定到`locations`上的`options`和`selectedLocation`上的`value`，最后一个输入将`checked`绑定到`isAdmin`。在需要指定输入绑定是一个值的情况下，这是一个简单的例子；在大多数情况下，输入将绑定到值，或者在这种情况下，复选框将绑定到`checked`。

“约定优于配置”的哲学旨在消除在常规场景中指定发生什么的需求。换句话说，除非另有说明，否则执行标准操作。以下是使用`BindingConventions`插件查看之前 DOM 的方式：

```js
<label>Name
  <input data-name="name" />
</label>
<label>LocationNameLocation
  <select data-name="locations"></select>
</label>
<label>Admin
  <input data-name="isAdmin" />
</label> 
```

在这里，`BindingConventions`正在做所有确定绑定的工作。`name`输入是我们 viewmodel 上的一个字符串可观察对象，它位于一个输入节点上，因此它获得`value`绑定。`isAdmin`输入是我们 viewmodel 上的布尔可观察对象，因此输入节点被转换成复选框，并接收`checked`绑定。`locations`属性是我们 viewmodel 上的一个数组，因此`select`元素获得一个`options`绑定。然而，这还不是全部！我们的 viewmodel 有一个`selectedLocations`可观察对象，`BindingConventions`确定它应该为`select`元素获得一个`value`绑定，因为将数组名称单数化并在前面添加`selected`是一个绑定约定。

最后一个可能看起来像是魔法，我个人认为它有点不明显，但它确实有一定的吸引力。如果你遵循约定，你可以真正简化你的绑定。你可以在`cp3-provider`分支中看到这个示例的实际效果。

现在你已经看到了这个绑定提供者正在做什么，让我们看看它是如何工作的。

### 注意

我们将在`BindingConventions`插件中查看绑定提供者的简化版本。真正的提供者支持更多的约定，并允许添加自定义约定。这个示例只是为了说明类型检测概念和创建自定义提供者的基础知识。

在创建自定义绑定提供者时，需要决定的第一件事是您是否需要扩展默认绑定提供者或替换它。`BindingConventions`提供者将支持`data-name`属性。在这种情况下，扩展默认提供者是有意义的，因为它们之间没有冲突，并且我们将需要标准的`data-bind`支持来处理非常规场景（例如，将我们的选择值绑定到`favoriteLocation`属性）。

做这件事的最简单方法是存储对原始绑定提供者的引用，并在我们的自定义提供者中调用它：

```js
ko.bindingConventions = {};
ko.bindingConventions.ConventionBindingProvider = function () {
     this.orgBindingProvider = ko.bindingProvider.instance || new ko.bindingProvider();
 };

var nodeHasBindings = function(node) { /* check node */ };
var conventionBindings = function(node, bindingContext) { /* check node */ };

 ko.bindingConventions.ConventionBindingProvider.prototype = {
     nodeHasBindings: function (node) {
         return this.orgBindingProvider.nodeHasBindings(node) || nodeHasBindings(node);
     },
     getBindingAccessors: function (node, bindingContext) {
         return this.orgBindingProvider.getBindingAccessors(node, bindingContext)
            || conventionBindings(node, bindingContext);
     }
 };
 ko.bindingProvider.instance = new ko.bindingConventions.ConventionBindingProvider();
```

这基本上是一个扩展默认提供者的绑定提供者的样板代码。它存储原始提供者，并通过首先调用默认提供者来实现`nodeHasBindings`和`getBindingAccessors`函数，如果默认提供者返回空值，则调用其自己的实现。如果您希望您的提供者在默认提供者之前检查绑定，您可以交换调用顺序。最后，您可以通过将绑定处理程序附加到默认提供者的结果来组合两者。

在设置所需的函数之后，`ko.bindingProvider.instance`被替换为新的自定义提供者。重要的是要注意，所有这些都必须在调用`ko.applyBindings`之前完成，因为绑定提供者只为根绑定上下文构建一次。

从这里，我们只需要提供检查绑定并创建它们的方法。检查绑定只需要检查`data-name`属性：

```js
var getNameAttribute = function (node) {
   var name = null;
   if (node.nodeType === 1) {
      name = node.getAttribute("data-name");
   }
   return name;
};

var nodeHasBindings = function(node) { 
   return getNameAttribute(node) !== null; 
};
```

从绑定上下文获取值需要做更多的工作。Knockout 在`ko.expressionRewriting`对象下提供了实用方法，可以解析任何支持的 Knockout 绑定语法。`BindingConventions`插件不支持除属性引用之外的内容，但它支持深层次引用，例如`person.firstName`。为了简单起见，我将不涉及这一点，但如果你对这个感兴趣，可以查看插件源代码中的`getDataFromComplexObjectQuery`。现在，我们假设所有`data-name`属性都直接引用一个属性：

```js
var conventionBindings = function(node, bindingContext) {
   var bindings = {};
   var name = getNameAttribute(node);
   if (name === null) {
      return null;
}

   var data = bindingContext[name] ? bindingContext[name] : bindingContext.$data[name];

   if (data === undefined) {
      throw "Can't resolve member: " + name;
   }

   var unwrapped = ko.utils.peekObservable(data);
   var type = typeof unwrapped;

   //Loop through convention handlers to construct bindings

   return bindings;
};
```

首先，我们从`data-name`属性中获取 viewmodel 属性的名称，然后我们执行一个合理性检查以确保它存在以进行绑定。然后，我们使用`ko.utils.peekObservable`获取数据并检查其类型。所有可观察对象都有一个`peek`函数，它返回底层值而不触发依赖检测。`peekObservable`函数如果第一个参数是可观察的，将调用`peek`；否则，它将只返回第一个参数。这是一个类似于`ko.uwrap`的安全实用工具。

在我们有了这两条信息之后，我们可以构建返回所需的绑定对象。记住，这个绑定对象应该有一个以要应用的绑定命名的属性，其值是绑定对应的`valueAccessor`对象。绑定被返回到绑定提供者的`getBindingAccessors`函数。为了构建绑定，我们将遍历一组约定：

```js
for (var index in ko.bindingConventions.conventionBinders) {
   if (ko.bindingConventions.conventionBinders[index].rules !== undefined) {
      var convention = ko.bindingConventions.conventionBinders[index];
      var shouldApply = true;

      convention.rules.forEach(function (rule) {
         shouldApply = shouldApply && rule(name, node, bindings, unwrapped, type, data, bindingContext);
      });

      if (shouldApply) {
         convention.apply(name, node, bindings, unwrapped, type, function() { return data }, bindingContext);
         break;
      }
   }
}
```

这将遍历`conventionBinders`数组，并按顺序检查每个元素的规则，以找到当前节点、数据和数据类型的匹配项。如果一个约定处理程序的规则都通过了，那么我们调用该约定的`apply`函数并停止检查——每个节点应该只应用一个约定。`apply`函数获取我们迄今为止收集的所有信息，以及一个可以用于绑定的`valueAccessor`属性。

我们的例子只使用了两种约定，即`options`和`input`：

```js
ko.bindingConventions.conventionBinders.options = {
  rules: [function (name, element, bindings, unwrapped) { return element.tagName === 'SELECT' && unwrapped.push; } ],
  apply: function (name, element, bindings, unwrapped, type, valueAccessor, bindingContext) {
      bindings.options = valueAccessor;
      singularize(name, function (singularized) {
         var selectedName = 'selected' + getPascalCased(singularized);
         if (bindingContext.$data[selectedName] !== undefined) {
            bindings['value'] = function() {
               return bindingContext.$data[selectedName];
            };
         }
      });
  }
};
```

选项只有一个规则：元素必须是一个`select`元素，并且数据需要是一个数组（通过查找`push`函数来检查）。

`apply` 函数直接将选项绑定设置到 `valueAccessor` 属性。然后，它尝试在上下文中找到一个匹配 `'selected' + getPascalCased(singularized)` 约定的属性。`singularize` 和 `getPascalCased` 函数在此处未包含，但您可以在以下代码示例分支中看到它们。不出所料，它们找到一个单词的单数形式并将其首字母大写。如果找到匹配项，则将 `value` 绑定添加到传入的 `bindings` 对象中。

`input` 处理器要简单得多：

```js
ko.bindingConventions.conventionBinders.input = {
  rules: [function (name, element) { return element.tagName === 'INPUT' || element.tagName === 'TEXTAREA'; } ],
  apply: function (name, element, bindings, unwrapped, type, valueAccessor, bindingContext) {
      var bindingName = null;
      if (type === 'boolean') {
          element.setAttribute('type', 'checkbox');
          bindingName = 'checked';
      } else {
          bindingName = 'value';
      }
      bindings[bindingName] = valueAccessor;
  }
};
```

`input` 处理器的规则不检查数据类型；它只是节点是 `input` 或 `textarea`。如果类型不是 `Boolean`，则 `apply` 函数将使用 `value` 绑定；否则，它将节点上的 `checkbox` 属性设置为 `checkbox` 并使用 `checked` 绑定。

就这些。这个绑定提供程序将允许使用 `data-name` 属性进行绑定，只需要一个视图模型属性作为值，并且它智能地设置了常规场景的绑定。如果我们需要更多控制，仍然可以使用常规的 `data-bind` 属性来应用绑定。

这个 `BindingConventions` 绑定提供程序的简化实现可以在 `cp3-provider2` 分支中看到。该分支的 `client/app` 目录包含这里讨论的简化实现以及从插件中获取的完整实现。

没有绑定或节点预处理器可以实现这一点，因为它依赖于绑定上下文中的数据类型。希望这能给您一个关于使用自定义绑定提供程序和整体绑定系统灵活性的良好概念。

# Knockout 拳击

现在您已经熟悉了用于修改绑定语法和预处理器的通用使用的技术，我们将探讨流行的 Knockout 插件 `Knockout.Punches (get it?)`。Punches 由 Michael Best 编写，他是 Knockout 开发者，也是 Knockout 预处理器功能以及一些最佳预处理器的实际用例的创造者。我们将探讨其中的一些，并深入探讨它们是如何工作的。本节不会涵盖 Knockout Punches 的所有内容；如果您想了解更多，可以查看在线文档。

### 注意

`Knockout.Punches` 的文档可以在 [`mbest.github.io/knockout.punches`](http://mbest.github.io/knockout.punches) 找到，其中包含 API 参考和源代码。

## 内嵌文本绑定

内嵌文本绑定提供了与我们在 *支持替代语法* 部分中创建的预处理器的相同语法——将花括号转换为虚拟文本节点：

```js
<div>Hello {{ name }}.</div>
```

之前的命令变为以下内容：

```js
<div>Hello <!--ko text:name--><!--/ko-->.</div>
```

Knockout Punches 使用的方法比我们之前看过的更高效，但它仍然提供了我们使用的相同可定制性。如果你想使用除了虚拟文本节点之外的东西作为插值替换，你可以提供一个返回`node-array`的函数作为以下内容的替换：

```js
ko.punches.utils.interpolationMarkup.wrapExpression(expressionText)
```

## 命名空间绑定

Knockout Punches 提供了一个简写绑定语法，将`x.y: value`展开为`x : { y: value }`。默认情况下，这种命名空间语法对`event`、`attr`、`style`和`css`绑定可用。在`style`绑定中使用它将导致以下内容展开：

```js
<div data-bind="style.color: textColor"></div>
```

这将展开为以下内容：

```js
<div data-bind="style: { color: textColor }"></div>
```

这通过覆盖标准的`ko.getBindingHandler`函数来实现，该函数通常只返回绑定处理程序。它被替换为一个查找绑定名称中具有匹配`getNamespacedHandler`属性的点的函数，并返回该函数。

### 动态命名空间绑定

由于`ko.getBindingHandler`被这样覆盖，因此可以通过向绑定处理程序添加`getNamespacedHandler`属性来创建自己的绑定命名空间：

```js
ko.bindingHandlers.customNamespace = {
    getNamespacedHandler: function(binding) {
        return {
           init: function(element, valueAccessor) { },
           update: function(element, valueAccessor) { }
        };
    }
};
```

`binding`参数是绑定的名称；对于`style.color`，它将是`color`。该函数返回要使用的绑定处理程序。这允许你为命名空间中的所有绑定提供一个单一的动态处理程序。

假设我们想要为 Twitter Bootstrap 提示插件创建一个绑定命名空间。我们需要提供提示文本内容和方向。通常，我们可能会编写一个绑定，将这些作为选项：

```js
ko.bindingHandlers.tooltip = {
    update: function(element, valueAccessor) {
      //Cleanup previous tooltips
      if (element.attributes['data-original-title']) {
        $(element).tooltip('destroy');
    }
      var options = valueAccessor();
        $(element).tooltip({ 
          placement: options.placement || 'left', 
          title: ko.unwrap(options.title || 'sample') 
        });
    }
};
```

然后，我们可以用对象绑定到它上：

```js
data-bind="tooltip: { placement: 'top', title: title}"
```

这工作得很好，但我们可以使用命名空间绑定处理程序重写这个，以便获得放置的点语法：

```js
ko.bindingHandlers.tooltip = {
    getNamespacedHandler: function(binding) {
        return {            
            update: function(element, valueAccessor) {
              //Cleanup previous tooltips
              if (element.attributes['data-original-title']) {
                $(element).tooltip('destroy');
      }
                $(element).tooltip({ 
                  placement: binding, 
                  title: ko.unwrap(valueAccessor()) 
                });
            }
        };
    }
};
```

这会产生一个更短的绑定属性，我认为这更容易阅读：

```js
data-bind="tooltip.top: title"
```

这个例子可以在`cp3-namespace`分支中看到。

## 绑定过滤器

在视模型属性上执行过滤操作是很常见的。通常的做法是在视模型上有一个计算属性执行过滤，但这可能会变得冗长，尤其是如果你有多个不同的过滤属性。Knockout Punches 提供了在绑定内应用过滤表达式的语法：

```js
<span data-bind="text: name | fit:20 | uppercase"></span>
```

过滤器由管道分隔，多个参数由冒号分隔。例如，`fit`最多接受三个参数，可以用`fit:20:'…':'middle'`指定。

应该注意的是，在先前的例子中，`name`不包括可观察的括号。虽然带有过滤器的整个绑定是一个单独的表达式，通常需要括号，但 Knockout Punches 通过对其调用`ko.unwrap`来智能地处理每个部分。这意味着绑定值和每个过滤器都被视为它们自己的表达式。

过滤是通过一个绑定预处理器完成的，该预处理器解析表达式并将管道部分递归展开为对过滤器的调用。前面的示例最终将从`preprocess`函数返回以下内容：

```js
ko.filters'uppercase')
```

### 编写自定义过滤器

添加自己的过滤器与添加绑定处理器非常相似。只需向`ko.filters`对象添加一个函数，该函数接受一个值和任意数量的参数，并返回一个修改后的值：

```js
ko.filters.translate = function(value, language) {
    return SomeLanguageLibrary.translate(value, language);
}
```

第一个参数是要处理的当前值。所有其他参数都是在绑定表达式中传递给过滤器的参数。

过滤器可以有零个参数——就像`uppercase`示例中那样——或者可选参数——就像`fit`示例中那样。过滤器预处理器不会检查传递给它的参数数量是否合理；它只是使用绑定表达式中的所有内容调用过滤器。

过滤器预处理器很容易扩展，并且提供了相当大的功能。我认为它是绑定预处理器潜力的最佳示例之一。

### 其他绑定的过滤器

默认情况下，`text`、`attr`和`html`绑定启用了过滤器，但额外的绑定可以通过调用`ko.punches.textFilter.enableForBinding(<binding>)`来使用过滤器。如果你想在自定义绑定上利用过滤器，这可能很有用。

过滤器不能用于双向绑定，如绑定值，因为它们总是产生内联表达式。

## 添加额外的预处理器

Knockout Punches 提供了两个实用方法，用于添加额外的绑定和节点预处理器，这些方法可以通过`ko.punches.utils`访问：

+   `addBindingPreprocessor(bindingKeyOrHandler, preprocessFn)`

+   `addNodePreprocessor(preprocessFn)`

如果你多次调用这些函数中的任何一个，相应的预处理器将被链在一起，每个新的预处理器都在链的末尾被调用。

绑定预处理器将一直运行，直到其中一个移除绑定或直到链的末尾。这阻止了链尝试处理不再存在的绑定。

节点预处理器将一直运行，直到其中一个返回要添加的新节点或直到链的末尾。这阻止了链尝试处理已经修改的节点。新节点不会被节点预处理器遍历，因此它们应该被添加到 DOM 中，并准备好进行数据绑定。

# 摘要

本章主要介绍了如何扩展 Knockout 的绑定过程并修改其语法。我们介绍了三种实现方式：

+   **绑定预处理器**：用于在绑定处理器运行之前修改绑定字符串

+   **节点预处理器**：用于在绑定开始之前修改 DOM

+   **绑定提供者**：用于控制应用于每个 DOM 节点的绑定

最后，我们研究了`Knockout.Punches`插件，以了解一些实际的 Knockout 扩展。

在下一章中，我们将介绍 Knockout 的 Web 组件功能，这些功能允许您将视图和 ViewModel 绑定在一起，形成可重用的控件。
