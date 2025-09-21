

# 第八章：内置 JavaScript 方法

我们刚刚覆盖了 JavaScript 中的大多数基本构建块。现在，是时候看看一些将使您的生活更轻松的强大内置方法了，这些方法我们还没有看到。内置方法是我们在使用 JavaScript 时直接获得的功能。我们可以使用这些方法而无需先编写代码。这是我们已经做了很多的事情，例如`console.log()`和`prompt()`。

许多内置方法也属于内置类。这些类及其方法可以随时使用，因为 JavaScript 已经定义了它们。这些类是为了我们的方便而存在的，因为它们是非常常见的需求，例如`Date`、`Array`和`Object`类。

利用已经内置到 JavaScript 中的功能可以提高代码的有效性，节省时间，并符合开发解决方案的各种最佳实践。我们将讨论一些此类函数的常见用途，例如操作文本、数学计算、处理日期和时间值、交互以及支持健壮的代码。以下是本章涵盖的主题：

+   全局 JavaScript 方法

+   字符串方法

+   数学方法

+   日期方法

+   数组方法

+   数字方法

注意：练习、项目和自我检查测验的答案可以在*附录*中找到。

# 内置 JavaScript 方法简介

我们已经看到了许多内置的 JavaScript 方法。任何我们没有自己定义的方法都是内置方法。一些例子包括`console.log()`、`Math.random()`、`prompt()`等等——例如，考虑数组上的方法。方法和函数之间的区别在于，函数可以在脚本中的任何地方定义，而方法是在类内部定义的。因此，方法基本上是类和实例上的函数。

方法通常也可以链式调用；这仅适用于返回结果的方法。下一个方法将在结果上执行。例如：

```js
let s = "Hello";
console.log(
  s.concat(" there!")
  .toUpperCase()
  .replace("THERE", "you")
  .concat(" You're amazing!")
); 
```

我们创建了一个变量`s`，并在第一行将其中的`Hello`存储进去。然后我们想要记录一些内容。这段代码被分成不同的行以提高可读性，但实际上它是一个语句。我们首先在我们的`s`变量上执行一个`concat()`方法，该方法将字符串附加到我们的字符串上。因此，在第一次操作之后，值变为`Hello there!`。然后我们使用下一个方法将其转换为大写。此时，值变为`HELLO THERE!`。然后我们继续将`THERE`替换为`you`。之后，值变为`HELLO you!`。然后我们再次向其附加一个字符串，最终将值记录下来：

```js
HELLO you! You're amazing! 
```

在这个例子中，我们需要记录或存储输出，因为仅调用字符串上的方法并不会更新原始字符串值。

# 全局方法

全局 JavaScript 方法可以在不引用它们所属的内置对象的情况下使用。这意味着我们可以直接使用方法名，就像它是一个在我们当前作用域内定义的函数一样，前面不需要“对象”。例如，而不是写：

```js
let x = 7;
console.log(Number.isNaN(x)); 
```

你也可以这样写：

```js
console.log(isNaN(x)); 
```

因此，`Number`可以省略，因为`isNaN`在不需要引用它所属的类（在这种情况下是`Number`类）的情况下被全局提供。在这种情况下，这两个`console.log`语句都将记录`false`（它们正在做完全相同的事情），因为当它不是数字时，`isNaN`返回`true`。而`7`是一个数字，所以它将记录`false`。

JavaScript 被构建为可以直接使用这些方法，因此为了实现这一点，在表面之下进行了一些魔法操作。JavaScript 的创造者选择了他们认为最常用的方法。因此，为什么一些方法是作为全局方法提供的，而其他方法则不是，可能看起来有些随意。这只是某些非常聪明的开发者在一个特定时间点的选择。

我们将在下面讨论最常见的全局方法。我们首先从解码和编码 URI（包括转义和未转义的）开始，然后解析数字，最后进行评估。

## URI 的解码和编码

有时你需要对字符串进行编码或解码。编码简单地说就是将一种形状转换为另一种形状。在这种情况下，我们将处理百分编码，也称为 URL 编码。在我们开始之前，可能会有一些关于 URI 和 URL 含义的混淆。**URI**（**统一资源标识符**）是某种资源的标识符。**URL**（**统一资源定位符**）是 URI 的一个子类别，它不仅是一个标识符，而且还包含有关如何访问它的信息（位置）。

让我们讨论如何对这些 URI（以及 URL，因为它们是子集）进行编码和解码。你需要这样做的一个例子是在表单中使用`get`方法通过 URL 发送变量。你通过 URL 发送的这些变量被称为查询参数。

如果某个内容包含空格，这将进行解码，因为你的 URL 中不能使用空格。它们将被转换为`%20`。URL 可能看起来像这样：

`www.example.com/submit?name=maaike%20van%20putten&love=coding`

所有字符都可以转换为以`%`开头的格式。然而，在大多数情况下，这并不是必要的。URI 可以包含一定数量的字母数字字符。特殊字符需要编码。一个例子，在编码之前是：

`https://www.example.com/submit?name=maaike van putten`

编码后的相同 URL 是：

`https://www.example.com/submit?name=maaike%20van%20putten`

有两种编码和解码方法。我们将在下面讨论它们及其用例。由于 URI 不能包含空格，因此使用包含空格的变量时，使用这些方法是至关重要的。

### decodeUri() 和 encodeUri()

`decodeUri()` 和 `encodeUri()` 实际上并不是真正的编码和解码，它们更多的是修复损坏的 URI。就像之前的例子中的空格一样。这对方法非常擅长修复损坏的 URI 并将它们解码回字符串。这里你可以看到它们的作用：

```js
let uri = "https://www.example.com/submit?name=maaike van putten";
let encoded_uri = encodeURI(uri);
console.log("Encoded:", encoded_uri);
let decoded_uri = decodeURI(encoded_uri);
console.log("Decoded:", decoded_uri); 
```

下面是输出结果：

```js
Encoded: https://www.example.com/submit?name=maaike%20van%20putten
Decoded: https://www.example.com/submit?name=maaike van putten 
```

如你所见，它已经替换了编码版本中的空格，并在解码版本中再次删除它们。所有其他字符都保持不变——这个编码和解码不关注特殊字符，因此将它们留在 URI 中。冒号、问号、等号、斜杠和和号是可以预期的。

这对于修复损坏的 URI 非常棒，但实际上，当你需要编码包含这些字符的字符串时（`/` `,` `?` `:` `@` `&` `=` `+` `$` `#`），实际上是无用的。这些可以用作 URI 的一部分，因此被跳过。这就是下一个两个内置方法派上用场的地方。

### decodeUriComponent() 和 encodeUriComponent()

因此，`decodeURI()` 和 `encodeURI()` 方法可以非常有助于修复损坏的 URI，但当你只想编码或解码包含特殊字符（如 `=` 或 `&`）的字符串时，它们就毫无用处。以下是一个例子：

`https://www.example.com/submit?name=this&that=some thing&code=love`

奇怪的价值，我们可以同意这一点，但它将展示我们的问题。使用 encodeURI 对此进行编码后，我们将得到：

`https://www.example.com/submit?name=this&that=some%20thing&code=love`

根据 URI 标准，这里实际上有 3 个变量：

+   `name`（值是 `this`）

+   `that`（值是 `some` `thing`）

+   `code`（值是 `love`）

虽然我们打算发送一个变量，`name`，其值为 `this&that=some thing&code=love`。

在这种情况下，你需要 `decodeUriComponent()` 和 `encodeUriComponent()`，因为变量部分中的 `=` 和 `&` 也需要编码。目前，这种情况并不存在，它实际上会在解释查询参数（`?` 后的变量）时引起问题。我们只想发送一个参数：`name`。但结果我们发送了三个。

让我们看看另一个例子。这是前一个章节中示例使用组件编码的结果：

```js
let uri = "https://www.example.com/submit?name=maaike van putten";
let encoded_uri = encodeURIComponent(uri);
console.log("Encoded:", encoded_uri);
let decoded_uri = decodeURIComponent(encoded_uri);
console.log("Decoded:", decoded_uri); 
```

最终的输出如下：

```js
Encoded: https%3A%2F%2Fwww.example.com%2Fsubmit%3Fname%3Dmaaike%20van%20putten
Decoded: https://www.example.com/submit?name=maaike van putten 
```

显然，你不想你的 URI 是这样的，但组件方法对于编码，例如 URL 变量很有用。如果 URL 变量包含特殊字符，如 `=` 和 `&`，这些字符如果没有被编码，就会改变意义并破坏 URI。

### 使用 escape() 和 unescape() 进行编码

这些仍然是全局方法，可用于执行类似于编码（escape）和解码（unescape）的操作。这两种方法强烈建议不要使用，并且它们实际上可能从未来的 JavaScript 版本中消失，或者浏览器可能出于良好原因不支持它们。

### 练习 8.1

将字符串`How's%20it%20going%3F`的`decodeURIComponent()`输出到控制台。也将字符串`How's it going?`编码并输出到控制台。创建一个网络 URL 并编码 URI：

1.  将字符串作为变量添加到 JavaScript 代码中

1.  使用`encodeURIComponent()`和`decodeURIComponent()`将结果输出到控制台

1.  创建一个带有请求参数`http://www.basescripts.com?=Hello World";`的网络 URI

1.  将网络 URI 编码并输出到控制台

## 解析数字

解析字符串到数字有不同的方法。在许多情况下，你将不得不将字符串转换为数字，例如从 HTML 网页中读取输入框。你不能用字符串进行计算，但可以用数字。根据你确切需要做什么，你可能需要这些方法中的任何一个。

### 使用`parseInt()`创建整数

使用`parseInt()`方法将字符串转换为整数。这个方法是`Number`类的一部分，但它全局可用，你可以不用在前面加上`Number`来使用它。这里你可以看到它是如何工作的：

```js
let str_int = "6";
let int_int = parseInt(str_int);
console.log("Type of ", int_int, "is", typeof int_int); 
```

我们从一个包含`6`的字符串开始。然后我们使用`parseInt`方法将这个字符串转换为整数，当我们记录结果时，我们将在控制台得到：

```js
Type of 6 is number 
```

你可以看到类型已经从`string`变成了`number`。在这个时候，你可能想知道如果`parseInt()`尝试解析其他类型的数字，比如浮点数的字符串版本或二进制数字，会发生什么。你认为当我们这样做时会发生什么？

```js
let str_float = "7.6";
let int_float = parseInt(str_float);
console.log("Type of", int_float, "is", typeof int_float);
let str_binary = "0b101";
let int_binary = parseInt(str_binary);
console.log("Type of", int_binary, "is", typeof int_binary); 
```

这将记录：

```js
Type of 7 is number
Type of 0 is number 
```

你能理解这里的逻辑吗？首先，JavaScript 不喜欢通过崩溃或使用错误作为退出方式，所以它正在尽力让它尽可能正常工作。`parseInt()`方法在遇到非数字字符时会停止解析。这是指定的行为，你在使用`parseInt()`时需要记住这一点。在第一种情况下，它一遇到点就停止解析，所以结果是`7`。在二进制数字的情况下，它一遇到`b`就停止解析，结果是`0`。到现在你可能已经能猜出这是做什么的：

```js
let str_nan = "hello!";
let int_nan = parseInt(str_nan);
console.log("Type of", int_nan, "is", typeof int_nan); 
```

由于第一个字符是非数字的，JavaScript 将这个字符串转换为`NaN`。这是你将在控制台得到的结果：

```js
Type of NaN is number 
```

所以`parseInt()`可能有点古怪，但它非常有价值。在现实世界中，它被大量用于通过网页结合用户输入和计算。

### 使用`parseFloat()`创建浮点数

同样，我们可以使用`parseFloat()`将字符串解析为浮点数。它的工作方式完全相同，但它也可以理解小数，并且它不会一遇到第一个点就停止解析：

```js
let str_float = "7.6";
let float_float = parseFloat(str_float);
console.log("Type of", float_float, "is", typeof float_float); 
```

这将记录：

```js
Type of 7.6 is number 
```

使用`parseInt()`，这个值变成了`7`，因为它一遇到非数字字符就会停止解析。然而，`parseFloat()`可以处理数字中的一个点，并且将之后的数字解释为小数。你能猜出当它遇到第二个点时会发生什么吗？

```js
let str_version_nr = "2.3.4";
let float_version_nr = parseFloat(str_version_nr);
console.log("Type of", float_version_nr, "is", typeof float_version_nr); 
```

这将记录：

```js
Type of 2.3 is number 
```

策略与 `parseInt()` 函数类似。一旦它找到一个无法解释的字符，在这个例子中是第二个点，它将停止解析并只返回到目前为止的结果。然后要注意的另一件事是，它不会在整数后面添加 `.0`，所以 `6` 不会变成 `6.0`。这个例子：

```js
let str_int = "6";
let float_int = parseFloat(str_int);
console.log("Type of", float_int, "is", typeof float_int); 
```

将记录：

```js
Type of 6 is number 
```

最后，二进制数和字符串的行为是相同的。它会在遇到无法解释的字符时停止解析：

```js
let str_binary = "0b101";
let float_binary = parseFloat(str_binary);
console.log("Type of", float_binary, "is", typeof float_binary);
let str_nan = "hello!";
let float_nan = parseFloat(str_nan);
console.log("Type of", float_nan, "is", typeof float_nan); 
```

这将记录：

```js
Type of 0 is number
Type of NaN is number 
```

你会在需要十进制数时使用 `parseFloat()`。然而，它不能与二进制、十六进制和八进制值一起工作，所以当你真正需要处理这些值或整数时，你必须使用 `parseInt()`。

## 使用 eval() 执行 JavaScript

这个全局方法将参数作为 JavaScript 语句执行。这意味着它将执行插入其中的任何 JavaScript，就像那个 JavaScript 是直接在那个地方写的一样，而不是 `eval()`。这对于处理注入的 JavaScript 可能很方便，但注入的代码伴随着巨大的风险。我们稍后会处理这些风险；让我们首先探索一个例子。这是一个非常棒的网站：

```js
<html>
  <body>
    <input onchange="go(this)"></input>
    <script>
      function go(e) {
        eval(e.value);
      }
    </script>
  </body>
</html> 
```

这是一个带有输入框的基本 HTML 网页。

你将在 *第九章*，*文档对象模型* 中了解更多关于 HTML 的内容。

你在输入框中插入的内容将被执行。如果我们把这个写在输入框里：

```js
document.body.style.backgroundColor = "pink"; 
```

网站的背景会变成粉色。这看起来很有趣，对吧？然而，我们无法强调使用 `eval()` 时应该多么小心。根据许多开发者的说法，他们甚至可能把它称为 *邪恶的*。你能解释为什么这可能是吗？

答案是安全性！是的，这可能是大多数情况下在安全性方面最糟糕的事情。你将执行外部代码。这段代码可能是恶意的。这是一种支持代码注入的方法。备受尊敬的 **OWASP** （**开放网络应用安全项目**） 基金会每三年发布一次安全威胁的前 10 名。代码注入自他们第一次发布的前 10 名以来一直存在，并且现在它仍然是 OWASP 前 10 名安全威胁之一。在服务器端运行它可能导致你的服务器崩溃，你的网站关闭，或者更糟。几乎总是有比使用 `eval()` 更好的解决方案。除了安全风险之外，它在性能方面也很糟糕。所以仅仅因为这个原因，你可能想要避免使用它。

好吧，关于这个的最后一点。如果你知道你在做什么，你可能想在非常具体的情况下使用它。尽管它被认为是“邪恶的”，但它拥有很大的力量。在某些情况下使用它是可以接受的，例如当你创建模板引擎、你自己的解释器和所有其他 JavaScript 核心工具时。只是要小心危险，并仔细控制对这个方法的访问。还有一个最后的技巧，当你觉得你真的必须使用 eval 时，在网上快速搜索一下。很可能你会找到一个更好的方法。

# 数组方法

我们已经看到了数组——它们可以包含多个项目。我们也看到了数组上的一些内置方法，如 `shift()` 和 `push()`。让我们在接下来的几节中看看更多。

## 对每个项目执行特定操作

我们之所以从这种方法开始，是有原因的。您可能此时正在考虑循环，但有一个内置的方法可以用来为数组中的每个元素执行一个函数。这就是 `forEach()` 方法。我们在 *第六章*，*函数* 中简要提到了它，但让我们更详细地考虑它。它接受需要为每个元素执行的函数作为输入。这里您可以看到一个示例：

```js
let arr = ["grapefruit", 4, "hello", 5.6, true];
function printStuff(element, index) {
  console.log("Printing stuff:", element, "on array position:", index);
}
arr.forEach(printStuff); 
```

这段代码将写入控制台：

```js
Printing stuff: grapefruit on array position: 0
Printing stuff: 4 on array position: 1
Printing stuff: hello on array position: 2
Printing stuff: 5.6 on array position: 3
Printing stuff: true on array position: 4 
```

如您所见，它为数组中的每个元素调用了 `printStuff()` 函数。我们还可以使用索引，它是第二个参数。我们不需要控制循环的流程，我们也不能在某个点上卡住。我们只需要指定需要为每个元素执行哪个函数。这个元素将被输入到这个函数中。这被广泛使用，尤其是在更函数式编程风格的场景中，其中许多方法被链式使用，例如，处理数据。

## 过滤数组

我们可以使用数组上的内置 `filter()` 方法来改变数组中的值。`filter()` 方法接受一个函数作为参数，这个函数应该返回一个布尔值。如果布尔值为 `true`，则元素将最终出现在过滤后的数组中。如果布尔值为 `false`，则元素将被排除在外。您可以看到它是如何工作的：

```js
let arr = ["squirrel", 5, "Tjed", new Date(), true];
function checkString(element, index) {
  return typeof element === "string";
}
let filterArr = arr.filter(checkString);
console.log(filterArr); 
```

这将在控制台输出：

```js
[ 'squirrel', 'Tjed' ] 
```

重要的是要意识到原始数组没有改变，`filter()` 方法返回一个包含通过过滤器的元素的新数组。我们在这里将这个新数组捕获到变量 `filterArr` 中。

## 检查所有元素的条件

您可以使用 `every()` 方法来检查数组中所有元素是否都满足某个条件。如果是这样，`every()` 方法将返回 `true`，否则将返回 `false`。我们在这里使用前一个例子中的 `checkString()` 函数和数组：

```js
console.log(arr.every(checkString)); 
```

这将记录 `false`，因为数组中并非所有元素都是 `string` 类型。

## 用数组的另一部分替换数组的一部分

`copyWithin()` 方法可以用来用数组的另一部分替换数组的一部分。在第一个例子中，我们指定了 3 个参数。第一个参数是目标位置，值将被复制到该位置。第二个参数是要复制到目标位置的起始位置，最后一个参数是要复制到目标位置的序列的结束位置；这个最后一个索引不包括在内。这里我们只是用位置 3 的值覆盖位置 0：

```js
arr = ["grapefruit", 4, "hello", 5.6, true];
arr.copyWithin(0, 3, 4); 
```

`arr` 变成了：

```js
[ 5.6, 4, 'hello', 5.6, true ] 
```

如果我们指定长度为 `2` 的范围，则从起始位置开始的前两个元素将被覆盖：

```js
arr = ["grapefruit", 4, "hello", 5.6, true];
arr.copyWithin(0, 3, 5); 
```

现在 `arr` 变成了：

```js
[ 5.6, true, 'hello', 5.6, true ] 
```

我们也可以完全不指定结束；它将取到字符串的末尾：

```js
let arr = ["grapefruit", 4, "hello", 5.6, true, false];
arr.copyWithin(0, 3);
console.log(arr); 
```

这将记录：

```js
[ 5.6, true, false, 5.6, true, false ] 
```

重要的是要记住，这个函数会改变原始数组的*内容*，但永远不会改变原始数组的*长度*。

## 映射数组的值

有时候你需要改变数组中的所有值。使用数组的`map()`方法，你可以做到这一点。这个方法将返回一个包含所有新值的新数组。你将必须说明如何创建这些新值。这可以通过箭头函数来完成。它将为数组中的每个元素执行箭头函数，例如：

```js
let arr = [1, 2, 3, 4];
let mapped_arr = arr.map(x => x + 1);
console.log(mapped_arr); 
```

这是新映射数组在控制台输出的样子：

```js
[ 2, 3, 4, 5 ] 
```

使用箭头函数，`map()`方法创建了一个新数组，其中每个原始数组值都增加了 1。

## 在数组中查找最后一个出现的位置

我们可以使用`indexOf()`找到出现，就像我们已经看到的那样。要找到最后一个出现，我们可以使用数组上的`lastIndexOf()`方法，就像我们对`string`做的那样。

如果它能找到具有该值的最后一个元素，它将返回该元素的索引：

```js
let bb = ["so", "bye", "bye", "love"];
console.log(bb.lastIndexOf("bye")); 
```

这将记录`2`，因为索引 2 持有最后一个`bye`变量。当你请求一个不存在的东西的最后一个索引时，你认为你会得到什么？

```js
let bb = ["so", "bye", "bye", "love"];
console.log(bb.lastIndexOf("hi")); 
```

对的（希望如此）！它是`-1`。

## 练习 8.2

使用`filter()`和`indexOf()`从数组中删除重复项。起始数组是：

```js
["Laurence", "Mike", "Larry", "Kim", "Joanne", "Laurence", "Mike", "Laurence", "Mike", "Laurence", "Mike"] 
```

使用数组`filter()`方法，这将创建一个新数组，使用通过函数实现的测试条件通过的元素。最终结果将是：

```js
 [ 'Laurence', 'Mike', 'Larry', 'Kim', 'Joanne' ] 
```

采取以下步骤：

1.  创建一个包含人名的数组。确保你包含重复的名字。这个练习将删除重复的名字。

1.  使用`filter()`方法，将数组中每个项目的结果作为匿名函数内的参数。使用值、索引和数组参数，返回过滤后的结果。你可以暂时将返回值设置为`true`，因为这将使用原始数组中的所有结果构建新数组。

1.  在函数内添加一个`console.log`调用，该调用将输出数组中当前项目的索引值。同时添加值，以便你可以看到具有当前索引号的项值以及数组索引值的第一匹配结果。

1.  使用`indexOf()`当前值返回项目索引值，并应用条件检查是否与原始索引值匹配。这个条件只会在第一个结果上为真，所以所有后续的重复项都将为假，并且不会添加到新数组中。`false`不会将值返回到新数组中。由于`indexOf()`只获取数组中的第一个匹配项，所以重复项都将为假。

1.  将新的唯一值数组输出到控制台。

## 练习 8.3

使用数组`map()`方法更新数组的内 容。采取以下步骤：

1.  创建一个数字数组。

1.  使用数组 `map` 方法和一个匿名函数，返回一个更新后的数组，将数组中的所有数字乘以 2。将结果输出到控制台。

1.  作为另一种方法，使用箭头函数格式，通过一行代码使用数组的 `map()` 方法将数组的每个元素乘以 2。

1.  将结果记录到控制台。

# 字符串方法

我们已经处理过字符串了，你很可能已经遇到了一些字符串方法。有一些我们没有特别说明，我们将在本节中讨论它们。

## 字符串的合并

当你想连接字符串时，可以使用 `concat()` 方法。这不会改变原始字符串（们）；它返回作为字符串的合并结果。你必须将结果捕获到新变量中，否则它将丢失：

```js
let s1 = "Hello ";
let s2 = "JavaScript";
let result = s1.concat(s2);
console.log(result); 
```

这将记录：

```js
Hello JavaScript 
```

## 将字符串转换为数组

使用 `split()` 方法我们可以将字符串转换为数组。同样，我们还需要捕获结果；它不会改变原始字符串。让我们使用包含 `Hello JavaScript` 的上一个结果。我们必须告诉 `split` 方法它应该在哪个字符串上分割。每次它遇到该字符串时，它都会创建一个新的数组项：

```js
let result = "Hello JavaScript";
let arr_result = result.split(" ");
console.log(arr_result); 
```

这将记录：

```js
[ 'Hello', 'JavaScript' ] 
```

如你所见，它创建了一个由空格分隔的所有元素的数组。我们可以按任何字符分割，例如逗号：

```js
let favoriteFruits = "strawberry,watermelon,grapefruit";
let arr_fruits = favoriteFruits.split(",");
console.log(arr_fruits); 
```

这将记录：

```js
[ 'strawberry', 'watermelon', 'grapefruit' ] 
```

现在已创建一个包含 3 个元素的数组。你可以根据任何内容进行分割，而你正在分割的字符串将不包括在结果中。

## 将数组转换为字符串

使用 `join()` 方法可以将数组转换为字符串。以下是一个基本示例：

```js
let letters = ["a", "b", "c"];
let x = letters.join();
console.log(x); 
```

这将记录：

```js
a,b,c 
```

`x` 的类型是 `string`。如果你想用其他东西而不是逗号，你可以指定它，就像这样：

```js
let letters = ["a", "b", "c"];
let x = letters.join('-');
console.log(x); 
```

这将使用 `–` 而不是逗号。这是结果：

```js
a-b-c 
```

这可以很好地与我们在上一节中介绍的 `split()` 方法结合使用，它执行相反的操作，将字符串转换为数组。

## 处理索引和位置

能够找出字符串中某个子字符串的索引非常有用。例如，当你需要通过日志文件的用户输入搜索某个特定单词并从该索引开始创建子字符串时。以下是如何找到字符串索引的示例。`indexOf()` 方法返回子字符串的第一个字符的索引，一个单一的数字：

```js
let poem = "Roses are red, violets are blue, if I can do JS, then you             can too!";
let index_re = poem.indexOf("re");
console.log(index_re); 
```

这将在控制台记录 `7`，因为 `re` 在 `are` 中的第一次出现是在索引 `7`。当它找不到索引时，它将返回 `-1`，就像这个例子一样：

```js
let indexNotFound = poem.indexOf("python");
console.log(indexNotFound); 
```

它记录 `-1` 来指示我们正在搜索的字符串在目标字符串中不存在。通常你会在处理结果之前编写一个 `if` 检查来查看它是否为 `-1`。例如：

```js
if(poem.indexOf("python") != -1) {
  // do stuff
} 
```

在字符串中搜索特定子字符串的另一种方法是使用 `search()` 方法：

```js
let searchStr = "When I see my fellow, I say hello";
let pos = searchStr.search("lo");
console.log(pos); 
```

这将输出 `17`，因为这是 `lo` 在 `fellow` 中的索引。与 `indexOf()` 类似，如果找不到，它将返回 `-1`。这个例子就是这样：

```js
let notFound = searchStr.search("JavaScript");
console.log(notFound); 
```

`search()` 方法可以接受正则表达式格式作为输入，而 `indexOf()` 只接受一个字符串。`indexOf()` 比搜索方法更快，所以如果你只需要查找一个字符串，请使用 `indexOf()`。如果你需要查找字符串模式，你必须使用 `search()` 方法。

正则表达式是定义字符串模式的特殊语法，你可以用它来替换所有出现，但我们将在 *第十二章*，*中级 JavaScript* 中处理它。

接下来，`indexOf()` 方法返回第一次出现的索引，但同样，我们还有一个 `lastIndexOf()` 方法。它返回参数字符串最后一次出现的位置。如果找不到，它返回 `-1`。以下是一个示例：

```js
let lastIndex_re = poem.lastIndexOf("re");
console.log(lastIndex_re); 
```

这返回 `24`；这是 `re` 在我们的诗中最后一次出现。它是第二个 `are`。

有时你可能需要做相反的操作；不是寻找字符串出现的索引，而是想知道某个索引位置上的字符是什么。这就是 `charAt(index)` 方法派上用场的地方，其中指定的索引位置作为参数：

```js
let pos1 = poem.charAt(10);
console.log(pos1); 
```

这是在记录 `r`，因为索引 10 的字符是 `red` 中的 `r`。如果你询问的索引位置超出了字符串的范围，它将返回一个空字符串，就像在这个例子中发生的那样：

```js
let pos2 = poem.charAt(1000);
console.log(pos2); 
```

这将在屏幕上输出一个空行，如果你询问 `pos2` 的类型，它将返回 `string`。

## 创建子字符串

使用 `slice(start, end)` 方法我们可以创建子字符串。这不会改变原始字符串，而是返回一个新的包含子字符串的字符串。它接受两个参数，第一个是开始索引的位置，第二个是结束索引。如果你省略第二个索引，它将从开始位置继续到字符串的末尾。结束索引不包括在子字符串中。以下是一个示例：

```js
let str = "Create a substring";
let substr1 = str.slice(5);
let substr2 = str.slice(0,3);
console.log("1:", substr1);
console.log("2:", substr2); 
```

这将输出：

```js
1: e a substring
2: Cre 
```

第一个只有一个参数，所以它从索引 5（包含一个 `e`）开始，并从那里获取字符串的其余部分。第二个有两个参数，`0` 和 `3`。`C` 在索引 0，`a` 在索引 3。由于最后一个索引不包括在子字符串中，它只会返回 `Cre`。

## 替换字符串的一部分

如果你需要替换字符串的一部分，你可以使用 `replace(old, new)` 方法。它接受两个参数，一个是要在字符串中查找的字符串，一个是要替换旧值的新的值。以下是一个示例：

```js
let hi = "Hi buddy";
let new_hi = hi.replace("buddy", "Pascal");
console.log(new_hi); 
```

这将在控制台输出 `Hi Pascal`。如果你没有捕获结果，它就会消失，因为原始字符串不会改变。如果你要替换的字符串在原始字符串中不存在，替换不会发生，原始字符串将被返回：

```js
let new_hi2 = hi.replace("not there", "never there");
console.log(new_hi2); 
```

这记录了 `Hi buddy`。

最后一点需要注意的是，默认情况下它只会改变第一次出现的位置。所以这个例子只会替换新字符串中的第一个 `hello`：

```js
let s3 = "hello hello";
let new_s3 = s3.replace("hello", "oh");
console.log(new_s3); 
```

这会记录 `oh hello`。如果我们想替换所有出现的位置，我们可以使用 `replaceAll()` 方法。这将用指定的新字符串替换所有出现的位置，如下所示：

```js
let s3 = "hello hello";
let new_s3 = s3.replaceAll("hello", "oh");
console.log(new_s3); 
```

这会记录 `oh oh`。

## 大写和小写

我们可以使用 `string` 上的内置方法 `toUpperCase()` 和 `toLowerCase()` 来改变字符串的字母。再次强调，这并不会改变原始字符串，所以我们需要捕获结果：

```js
let low_bye = "bye!";
let up_bye = low_bye.toUpperCase();
console.log(up_bye); 
```

这会记录：

```js
BYE! 
```

它将所有字母转换为大写。我们可以使用 `toLowerCase()` 来做相反的操作：

```js
let caps = "HI HOW ARE YOU?";
let fixed_caps = caps.toLowerCase();
console.log(fixed_caps); 
```

这会记录：

```js
hi how are you? 
```

让我们让它更复杂一些，比如说我们想要句子的第一个字母大写。我们可以通过结合我们现在已经看到的一些方法来实现这一点：

```js
let caps = "HI HOW ARE YOU?";
let fixed_caps = caps.toLowerCase();
let first_capital = fixed_caps.charAt(0).toUpperCase().concat(fixed_caps.slice(1));
console.log(first_capital); 
```

我们在这里正在链式调用方法；我们首先使用 `charAt(0)` 获取 `fixed_caps` 的第一个字符，然后通过调用 `toUpperCase()` 来将其转换为大写。然后我们需要字符串的其余部分，我们可以通过连接 `slice(1)` 来获取它。

## 字符串的开始和结束

有时候你可能会想检查一个字符串的开始或结束部分。你已经猜到了，`string` 上有内置方法可以做到这一点。我们可以想象这一章很难理解，所以这里有一点点鼓励，同时也是一个例子：

```js
let encouragement = "You are doing great, keep up the good work!";
let bool_start = encouragement.startsWith("You");
console.log(bool_start); 
```

这将在控制台输出 `true`，因为句子以 `You` 开头。请注意，这是大小写敏感的。所以下面的例子将输出 `false`：

```js
let bool_start2 = encouragement.startsWith("you");
console.log(bool_start2); 
```

如果你不在乎大小写，你可以在之前讨论的 `toLowerCase()` 方法中使用它，这样它就不会考虑大小写。

```js
let bool_start3 = encouragement.toLowerCase().startsWith("you");
console.log(bool_start3); 
```

我们现在将字符串转换为小写，这样我们就知道我们在这里只处理小写字母。然而，这里的一个重要注意事项是，这将对大字符串的性能产生影响。

再次强调，一个更高效的替代方案是使用正则表达式。对 *第十二章*，*中级 JavaScript* 感到兴奋了吗？

为了结束这一节，我们可以对检查字符串是否以特定字符串结束做同样的事情。你可以在这里看到它的实际应用：

```js
let bool_end = encouragement.endsWith("Something else");
console.log(bool_end); 
```

由于它不以 `Something else` 结尾，所以它将返回 `false`。

## 练习 8.4

使用字符串操作，创建一个函数，该函数将返回一个字符串，其中所有单词的首字母大写，其余字母小写。你应该将句子 `thIs will be capiTalized for each word` 转换为 `This Will Be Capitalized For Each Word`：

1.  创建一个包含不同大小写字母的单词的字符串，包括大写和小写的单词混合。

1.  创建一个函数，该函数接受一个字符串作为参数，这是我们将会操作的值。

1.  在函数中首先将所有内容转换为小写字母。

1.  创建一个空数组，用于存储当我们将单词大写时它们的值。

1.  使用 `split()` 方法将短语转换为单词数组。

1.  遍历新数组中的每个单词，以便你可以独立选择每个单词。你可以使用 `forEach()` 来做这个。

1.  使用 `slice()` 从每个单词中隔离第一个字母，然后将其转换为大写。再次使用 `slice()`，获取不包括第一个字母的单词剩余部分。然后将这两个部分连接起来，形成现在已大写的单词。

1.  将新的大写单词添加到您创建的空白数组中。循环结束时，你应该有一个包含所有单词作为新数组中单独项目的数组。

1.  使用 `join()` 方法将更新后的单词数组转换回一个字符串，单词之间用空格分隔。

1.  返回更新后带有大写单词的新字符串的值，然后可以将其输出到控制台。

## 练习 8.5

使用 `replace()` 字符串方法，通过将字符串中的元音字母替换为数字来完成这个元音替换练习。你可以从以下字符串开始：

```js
I love JavaScript 
```

然后将其转换为以下类似的内容：

```js
2 l3v1 j0v0scr2pt 
```

按以下步骤操作：

1.  创建之前指定的字符串，并将其转换为小写。

1.  创建一个包含元音字母的数组：a, e, i, o, u。

1.  遍历数组中的每个字母，并将当前字母输出到控制台，以便你可以看到哪个字母将被转换。

1.  在循环中，使用 `replaceAll()` 更新每个元音子字符串，使用元音数组中字母的索引值。

    使用 `replace()` 只会替换第一次出现的内容；如果你使用 `replaceAll()`，这将更新所有匹配的结果。

1.  循环完成后，将新字符串的结果输出到控制台。

# 数字方法

让我们继续学习 `Number` 对象的一些内置方法。我们已经见过一些了，它们非常受欢迎，以至于其中一些已经被做成全局方法。

## 检查某个值是否（不是）一个数字

这可以用 `isNaN()` 来做。我们已经在讨论全局方法时见过这个方法，我们可以在它前面不加 `Number` 使用这个方法。通常你想要做的是相反的操作，你可以在函数前面加一个 `!` 来取反：

```js
let x = 34;
console.log(isNaN(x));
console.log(!isNaN(x));
let str = "hi";
console.log(isNaN(str)); 
```

这将在控制台输出：

```js
false
true
true 
```

由于 `x` 是一个数字，`isNaN` 将为 `false`。但这个结果取反后变为 `true`，因为 `x` 是一个数字。字符串 `hi` 不是一个数字，所以它将变为 `false`。那么这个呢？

```js
let str1 = "5";
console.log(isNaN(str1)); 
```

这里正在进行一些有趣的操作，即使 `5` 在引号之间，JavaScript 仍然将其视为数字 5，并将输出 `false`。在这个时候，我相信你希望你的伴侣、家人和同事像 JavaScript 一样理解和宽容。

## 检查某个值是否是有限的

到现在为止，你可能已经能够猜出在 `Number` 上检查某个数是否有限的那个方法的名字。它是一个非常流行的方法，并且已经被做成全局方法，它的名字是 `isFinite()`。对于 `NaN`、`Infinity` 和 `undefined`，它返回 `false`，对于所有其他值返回 `true`：

```js
let x = 3;
let str = "finite";
console.log(isFinite(x));
console.log(isFinite(str));
console.log(isFinite(Infinity));
console.log(isFinite(10 / 0)); 
```

这将记录：

```js
true
false
false
false 
```

在这个列表中，唯一的有限数是 `x`。其他的都不是有限的。字符串不是一个数字，因此不是有限的。`Infinity` 也不是有限的，`10` 除以 `0` 返回 `Infinity`（这不是一个错误）。

## 检查是否为整数

是的，这是用 `isInteger()` 做的。与 `isNaN()` 和 `isFinite()` 不同，`isInteger()` 没有被做成全局方法，我们必须引用 `Number` 对象来使用它。它确实做了你想象中的事情：如果值是整数，它返回 `true`；如果不是，返回 `false`：

```js
let x = 3;
let str = "integer";
console.log(Number.isInteger(x));
console.log(Number.isInteger(str));
console.log(Number.isInteger(Infinity));
console.log(Number.isInteger(2.4)); 
```

这将记录：

```js
true
false
false
false 
```

由于列表中唯一的整数是 `x`。

## 指定小数位数

我们可以使用 `toFixed()` 方法告诉 JavaScript 使用多少位小数。这与 `Math` 中的四舍五入方法不同，因为我们可以在这里指定小数位数。它不会改变原始值，所以我们必须存储结果：

```js
let x = 1.23456;
let newX = x.toFixed(2); 
```

这将只保留两位小数，所以 `newX` 的值将是 `1.23`。它按正常方式四舍五入；当你要求更多一位小数时，你可以看到这一点：

```js
let x = 1.23456;
let newX = x.toFixed(3); 
console.log(x, newX); 
```

这记录了 `1.23456 1.235` 作为输出。

## 指定精度

同样，还有一个方法可以指定精度。这又不同于 `Math` 类中的四舍五入方法，因为我们可以指定要查看的总数。这归结为 JavaScript 查看总数。它也在计算小数点前的数字：

```js
let x = 1.23456;
let newX = x.toPrecision(2); 
```

因此，这里 `newX` 的值将是 `1.2`。同样，这里它也在进行四舍五入：

```js
let x = 1.23456;
let newX = x.toPrecision(4); 
console.log(newX); 
```

这将记录 `1.235`。

现在，让我们继续讨论一些相关的数学方法！

# 数学方法

`Math` 对象有许多方法，我们可以使用它们在数字上进行计算和操作。我们将在下面介绍最重要的几个。当你使用一个在输入时显示建议和选项的编辑器时，你可以看到所有可用的方法。

## 查找最大和最小数

有一个内置的 `max()` 方法可以找到参数中的最大数。你可以在这里看到它：

```js
let highest = Math.max(2, 56, 12, 1, 233, 4);
console.log(highest); 
```

它记录了 `233`，因为这是最大的数。以类似的方式，我们可以找到最小的数：

```js
let lowest = Math.min(2, 56, 12, 1, 233, 4);
console.log(lowest); 
```

这将记录 `1`，因为这是最小的数。如果你尝试用非数字参数做这个操作，你将得到 `NaN` 作为结果：

```js
let highestOfWords = Math.max("hi", 3, "bye");
console.log(highestOfWords); 
```

它没有输出 `3`，因为它不是忽略文本，而是得出结论，它无法确定 `hi` 应该是高于还是低于 `3`。因此，它返回 `NaN`。

## 平方根和幂运算

`sqrt()` 方法用于计算某个数的平方根。这里你可以看到它的实际应用：

```js
let result = Math.sqrt(64);
console.log(result); 
```

这将记录`8`，因为`64`的平方根是`8`。这种方法与你在学校学到的数学一样。为了将一个数提升到某个幂（例如 2³），我们可以使用`pow(base, exponent)`函数。就像这样：

```js
let result2 = Math.pow(5, 3);
console.log(result2); 
```

我们在这里将`5`提升到`3`的幂（5³），所以结果是`125`，这是 5*5*5 的结果。

## 将小数转换为整数

将小数转换为整数的方法有很多。有时你可能需要四舍五入一个数字。你可以使用`round()`方法来做这件事：

```js
let x = 6.78;
let y = 5.34;
console.log("X:", x, "becomes", Math.round(x));
console.log("Y:", y, "becomes", Math.round(y)); 
```

这将记录：

```js
X: 6.78 becomes 7
Y: 5.34 becomes 5 
```

如你所见，这里使用的是正常的四舍五入。也可能你不想向下取整，而是向上取整。例如，如果你需要计算你需要多少块木板，你得出结论你需要`1.1`，`1`将不足以完成工作。你需要`2`。在这种情况下，你可以使用`ceil()`方法（即天花板）：

```js
console.log("X:", x, "becomes", Math.ceil(x));
console.log("Y:", y, "becomes", Math.ceil(y)); 
```

这将记录：

```js
X: 6.78 becomes 7
Y: 5.34 becomes 6 
```

`ceil()`方法总是向上取整到遇到的第一个整数。我们之前在生成随机数时已经使用过这个方法了！注意这里的负数，因为`-5`比`-6`要大。这是它的工作方式，就像你在下面的例子中看到的那样：

```js
let negativeX = -x;
let negativeY = -y;
console.log("negativeX:", negativeX, "becomes", Math.ceil(negativeX));
console.log("negativeY:", negativeY, "becomes", Math.ceil(negativeY)); 
```

这将记录：

```js
negativeX: -6.78 becomes -6
negativeY: -5.34 becomes -5 
```

`floor()`方法与`ceil()`方法正好相反。它向下取整到最接近的整数，就像你在这里看到的那样：

```js
console.log("X:", x, "becomes", Math.floor(x));
console.log("Y:", y, "becomes", Math.floor(y)); 
```

这将记录：

```js
X: 6.78 becomes 6
Y: 5.34 becomes 5 
```

再次提醒，注意这里的负数，因为它可能感觉不太直观：

```js
console.log("negativeX:", negativeX, "becomes", Math.floor(negativeX));
console.log("negativeY:", negativeY, "becomes", Math.floor(negativeY)); 
```

这将记录：

```js
negativeX: -6.78 becomes -7
negativeY: -5.34 becomes -6 
```

然后还有一个最后的方法，`trunc()`。对于正数，它给出的结果与`floor()`相同，但它得到这些结果的方式不同。它不是向下取整，它只是简单地返回整数部分：

```js
console.log("X:", x, "becomes", Math.trunc(x));
console.log("Y:", y, "becomes", Math.trunc(y)); 
```

这将记录：

```js
X: 6.78 becomes 6
Y: 5.34 becomes 5 
```

当我们使用负数进行`trunc()`时，我们可以看到差异：

```js
negativeX: -6.78 becomes -6
negativeY: -5.34 becomes –5 
```

所以，每当你需要向下取整时，你将不得不使用`floor()`，如果你需要数字的整数部分，你需要`trunc()`。

## 指数和对数

指数是基数被提升到的数。在数学中，我们经常使用`e`（欧拉数），这就是 JavaScript 中的`exp()`方法所做的工作。它返回`e`必须提升到的数以得到输入。我们可以使用`Math`的内置`exp()`方法来计算指数，以及`log()`方法来计算自然对数。你可以在这里看到一个例子：

```js
let x = 2;
let exp = Math.exp(x);
console.log("Exp:", exp);
let log = Math.log(exp);
console.log("Log:", log); 
```

这将记录：

```js
Exp: 7.38905609893065
Log: 2 
```

如果你现在在数学上跟不上了，不要担心。当你需要编程时，你会弄明白的。

## 练习题 8.6

使用以下步骤实验`Math`对象：

1.  使用`Math`将`PI`的值输出到控制台。

1.  使用`Math`获取`5.7`的`ceil()`值，获取`5.7`的`floor()`值，获取`5.7`的舍入值，并将其输出到控制台。

1.  将随机值输出到控制台。

1.  使用`Math.floor()`和`Math.random()`从 0 到 10 获取一个数字。

1.  使用`Math.floor()`和`Math.random()`从 1 到 10 获取一个数字。

1.  使用`Math.floor()`和`Math.random()`从 1 到 100 获取一个数字。

1.  创建一个函数，使用 `min` 和 `max` 参数生成随机数。运行该函数 100 次，每次在控制台返回一个 1 到 100 的随机数。

# 日期方法

为了在 JavaScript 中处理日期，我们使用内置的 `Date` 对象。此对象包含许多内置函数，用于处理日期。

## 创建日期

创建日期有不同的方法。创建日期的一种方法是通过使用不同的构造函数。您在这里可以看到一些示例：

```js
let currentDateTime = new Date();
console.log(currentDateTime); 
```

这将记录当前日期和时间，在这种情况下：

```js
2021-06-05T14:21:45.625Z 
```

但是，我们这里不是使用内置方法，而是使用构造函数。有一个内置方法，`now()`，它返回当前日期和时间，类似于无参数构造函数所做的那样：

```js
let now2 = Date.now();
console.log(now2); 
```

这将记录当前时间，以自 1970 年 1 月 1 日以来的秒数表示。这是一个表示 Unix 纪元的任意日期。在这种情况下：

```js
1622902938507 
```

我们可以向 Unix 纪元时间添加 1,000 毫秒：

```js
let milliDate = new Date(1000);
console.log(milliDate); 
```

它将记录：

```js
1970-01-01T00:00:01.000Z 
```

JavaScript 也可以将许多字符串格式转换为日期。始终注意日期格式中日期和月份的顺序以及 JavaScript 的解释器。这可能会根据地区而变化：

```js
let stringDate = new Date("Sat Jun 05 2021 12:40:12 GMT+0200");
console.log(stringDate); 
```

这将记录：

```js
2021-06-05T10:40:12.000Z 
```

最后，您还可以使用构造函数指定一个特定日期：

```js
let specificDate = new Date(2022, 1, 10, 12, 10, 15, 100);
console.log(specificDate); 
```

这将记录：

```js
2022-02-10T12:10:15.100Z 
```

请注意这里的一个非常重要的细节，第二个参数是月份。`0` 代表一月，`11` 代表十二月。

## 获取和设置日期元素的函数

现在我们已经看到了如何创建日期，我们将学习如何获取日期的某些部分。这可以通过许多 `get` 方法之一来完成。您将使用哪个取决于您需要哪个部分：

```js
let d = new Date();
console.log("Day of week:", d.getDay());
console.log("Day of month:", d.getDate());
console.log("Month:", d.getMonth());
console.log("Year:", d.getFullYear());
console.log("Seconds:", d.getSeconds());
console.log("Milliseconds:", d.getMilliseconds());
console.log("Time:", d.getTime()); 
```

这将立即记录：

```js
Day of week: 6
Day of month: 5
Month: 5
Year: 2021
Seconds: 24
Milliseconds: 171
Time: 1622903604171 
```

时间如此之高，因为它是从 1970 年 1 月 1 日以来的毫秒数。您可以使用类似的 `set` 方法更改日期。这里需要注意的是，原始日期对象会通过这些设置方法被更改：

```js
d.setFullYear(2010);
console.log(d); 
```

我们已经将我们的日期对象的年份更改为 2010 年。这将输出：

```js
2010-06-05T14:29:51.481Z 
```

我们还可以更改月份。让我们将以下代码片段添加到我们的年份更改代码中。这将将其更改为十月。请注意，当我这样做的时候，我会反复运行代码，所以当我没有设置这些时，示例中的分钟和更小的时间单位将会变化：

```js
d.setMonth(9);
console.log(d); 
```

它将记录：

```js
2010-10-05T14:30:39.501Z 
```

这是一个奇怪的情况，为了更改日期，我们必须调用 `setDate()` 方法，而不是 `setDay()` 方法。没有 `setDay()` 方法，因为星期几是从特定日期中扣除的。我们不能改变 2021 年 9 月 5 日是星期日的事实。不过，我们可以更改月份中的天数：

```js
d.setDate(10);
console.log(d); 
```

这将记录：

```js
2010-10-10T14:34:25.271Z 
```

我们还可以更改小时：

```js
d.setHours(10);
console.log(d); 
```

现在它将记录：

```js
2010-10-10T10:34:54.518Z 
```

记得 JavaScript 不喜欢崩溃吗？如果您使用大于 24 的数字调用 `setHours()`，它将滚动到下一个日期（每 24 小时一个），并且在使用模运算符后，`hours % 24` 剩余的部分将是小时。同样的过程适用于分钟、秒和毫秒。

`setTime()`实际上用插入的纪元时间覆盖了完整的日期：

```js
d.setTime(1622889770682);
console.log(d); 
```

这将记录：

```js
2021-06-05T10:42:50.682Z 
```

## 解析日期

使用内置的`parse()`方法，我们可以从字符串中解析纪元日期。它接受许多格式，但同样，你需要注意日期和月份的顺序：

```js
let d1 = Date.parse("June 5, 2021");
console.log(d1); 
```

这将记录：

```js
1622851200000 
```

如你所见，它以许多零结尾，因为我们没有在字符串中指定时间或秒。这里还有一个完全不同格式的例子：

```js
let d2 = Date.parse("6/5/2021");
console.log(d2); 
```

这也将记录：

```js
1622851200000 
```

解析的输入是日期的 ISO 格式。许多格式可以解析为字符串，但你需要小心。结果可能取决于确切的实现。确保你知道传入字符串的格式，以免混淆月份和日期，并确保你知道实现的行为了。只有当你知道字符串格式时，才能可靠地完成这项工作。例如，当你需要将来自你自己的数据库或网站日期选择器的数据转换为日期时。

## 将日期转换为字符串

我们也可以将日期转换回字符串。例如，使用这些方法：

```js
console.log(d.toDateString()); 
```

这将记录以文字格式表示的日期：

```js
Sat Jun 05 2021 
```

这是一种不同的转换方法：

```js
console.log(d.toLocaleDateString()); 
```

它将记录：

```js
6/5/2021 
```

## 练习题 8.7

将完整的月份名称输出到控制台中的日期。在转换到或从数组时，请记住它们是零基的：

1.  设置一个日期对象，可以是未来的任何日期或过去的日期。将日期输出到控制台以查看它通常如何作为日期对象输出。

1.  设置一个包含一年中所有月份名称的数组。保持它们的顺序，以便它们与日期月份输出相匹配。

1.  使用`getDate()`从日期对象值中获取日期。

1.  从日期对象值中获取年份，使用`getFullYear()`。

1.  使用`getMonth()`从日期对象值中获取月份。

1.  设置一个变量来存储日期对象中的日期，并使用数组月份名称的数值作为索引输出月份。由于数组是零基的，月份返回的值为 1-12，因此结果需要减去 1。

1.  将结果输出到控制台。

# 章节项目

## 单词打乱器

创建一个函数，该函数返回单词的值，并使用`Math.random()`打乱字母顺序：

1.  创建一个字符串，用于存储你选择的单词值。

1.  创建一个可以接受字符串单词值的参数的函数。

1.  就像数组一样，字符串也有默认的长度。你可以使用这个长度来设置循环的最大值。你需要创建一个单独的变量来存储这个值，因为随着循环的进行，字符串的长度会减小。

1.  创建一个空临时的字符串变量，你可以用它来存储新的打乱后的单词值。

1.  创建一个`for`循环，该循环将从字符串参数中的 0 开始迭代，直到达到字符串的原始长度值。

1.  创建一个变量，使用`Math.floor()`和`Math.random()`乘以当前字符串的长度，通过其索引值随机选择一个字母。

1.  将新字母添加到新字符串中，并从原始字符串中移除它。

1.  使用`console.log()`输出由随机字母组成的新构造字符串，并在循环继续时同时输出原始字符串和新字符串。

1.  通过选择从索引值开始的子字符串并将其添加到索引加一之后的剩余字符串值来更新原始字符串。输出新的原始字符串值，其中已移除字符。

1.  当你遍历内容时，你会看到剩余字母的倒计时、构建中的单词的新打乱版本，以及原始单词中减少的字母。

1.  返回最终结果，并使用原始字符串单词作为参数调用该函数。将此输出到控制台。

## 倒计时计时器

创建一个可以在控制台窗口中执行的倒计时计时器代码，并显示到达目标日期之前剩余的总毫秒数、天数、小时数、分钟数和秒数：

1.  创建一个你想要倒计时的结束日期。在字符串中将其格式化为日期类型格式。

1.  创建一个倒计时函数，该函数将解析`endDate()`并从当前日期减去该结束日期。这将显示以毫秒为单位的总数。使用`Date.parse()`，你可以将日期的字符串表示形式转换为自 1970 年 1 月 1 日 00:00:00 UTC 以来的数值。

1.  一旦你有了总毫秒数，要获取天数、小时数、分钟数和秒数，你可以采取以下步骤：

    +   要获取天数，你可以将日期中的毫秒数除以，并使用`Math.floor()`移除余数。

    +   要获取小时数，可以使用模运算来捕获在总天数被移除后的余数。

    +   要获取分钟数，你可以使用分钟中的毫秒值，并使用模运算捕获余数。

    +   通过将数字除以毫秒中的秒数并获取余数，以同样的方式获取秒数。如果你使用`Math.floor()`，你可以向下取整，移除任何将在较低值中显示的剩余小数位。

1.  返回一个对象中的所有值，其属性名表示值所引用的时间单位。

1.  创建一个函数，使用`setTimeout()`每秒（1,000 毫秒）运行`update()`函数。`update()`函数将创建一个变量，可以暂时保存`countdown()`的对象返回值，并创建一个空变量，将用于创建输出值。

1.  在同一函数中，使用`for`循环获取`temp`对象变量的所有属性和值。当你遍历对象时，更新输出字符串以包含属性名和属性值。

1.  使用`console.log()`将输出结果字符串打印到控制台。

# 自我检查测验

1.  以下哪种方法可以解码以下内容？

    ```js
    var c = "http://www.google.com?id=1000&n=500";
    var e = encodeURIComponent(c); 
    ```

    1.  `decodeURIComponent(e)`

    1.  `e.decodeUriComponent()`

    1.  `decoderURIComponent(c)`

    1.  `decoderURIComponent(e)`

1.  以下语法将在控制台输出什么？

    ```js
    const arr = ["hi","world","hello","hii","hi","hi World","Hi"];
    console.log(arr.lastIndexOf("hi")); 
    ```

1.  以下代码在控制台的结果是什么？

    ```js
    const arr = ["Hi","world","hello","Hii","hi","hi World","Hi"];
    arr.copyWithin(0, 3, 5);
    console.log(arr); 
    ```

1.  以下代码在控制台的结果是什么？

    ```js
    const arr = ["Hi","world","hello","Hii","hi","hi World","Hi"];
    const arr2 = arr.filter((element,index)=>{
        const ele2 = element.substring(0, 2);
        return (ele2 == "hi");
    });
    console.log(arr2); 
    ```

# 摘要

在本章中，我们处理了许多内置方法。这些是 JavaScript 提供给我们的方法，我们可以使用它们来完成我们经常需要做的事情。我们回顾了最常用的全局内置方法，这些方法如此常见，以至于无需使用它们所属的对象作为前缀即可使用。

我们还讨论了数组方法、字符串方法、数字方法、数学方法和日期方法。当你对 JavaScript 更加熟悉时，你会发现自己会大量使用这些方法，并在适当的时候将它们链接起来（只要它们返回结果）。

现在我们已经熟悉了 JavaScript 的许多核心特性，接下来几章我们将深入探讨它是如何与 HTML 和浏览器协同工作，使网页生动起来的！
