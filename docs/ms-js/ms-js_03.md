# 第三章．数据结构与操作

在编程中你花费大部分时间做的事情是操作数据。你处理数据的属性，根据数据得出结论，改变数据的本性。在本章中，我们将详细介绍 JavaScript 中的各种数据结构和数据操作技术。正确使用这些表达式结构，你的程序将会是正确的、简洁的、易于阅读的，并且很有可能是更快的。这将在以下主题帮助下解释：

+   正则表达式

+   精确匹配

+   从字符类中匹配

+   重复出现

+   开始和结束

+   反向引用

+   贪婪与懒惰量词

+   数组

+   映射

+   集合

+   风格问题

# 正则表达式

如果你不熟悉正则表达式，我建议你花时间去学习它们。有效地学习和使用正则表达式是你会获得的最有价值的技能之一。在大多数代码审查会议中，我首先评论的是如何将一段代码转换成单个正则表达式（或 RegEx）的行。如果你研究流行的 JavaScript 库，你会惊讶地看到正则表达式的普遍性。大多数经验丰富的工程师主要依赖正则表达式，因为一旦你知道如何使用它们，它们就是简洁且易于测试的。然而，学习正则表达式将需要大量的精力和时间。正则表达式是表达匹配文本字符串的模式的方法。表达式本身由术语和操作符组成，使我们能够定义这些模式。我们很快就会看到这些术语和操作符由什么组成。

在 JavaScript 中，创建正则表达式有两种方法：通过正则表达式字面量和使用`RegExp`对象实例化。

例如，如果我们想要创建一个正好匹配字符串 test 的正则表达式，我们可以使用以下正则表达式字面量：

```js
var pattern = /test/;
```

正则表达式字面量使用斜杠分隔。或者，我们可以构造一个`RegExp`实例，将正则表达式作为字符串传递：

```js
var pattern = new RegExp("test");
```

这两种格式都会在变量 pattern 中创建相同的正则表达式。除了表达式本身，还有三个标志可以与正则表达式关联：

+   `i`：这使正则表达式不区分大小写，所以`/test/i`不仅匹配`test`，还匹配`Test`、`TEST`、`tEsT`等。

+   `g`：这与默认的局部匹配相反，后者只匹配第一个出现。稍后会有更多介绍。

+   `m`：这允许跨多行匹配，这可能来自`textarea`元素的值。

这些标志在字面量末尾附加（例如，`/test/ig`）或作为字符串传递给`RegExp`构造器的第二个参数（`new RegExp("test", "ig")`）。

以下示例说明了各种标志以及它们如何影响模式匹配：

```js
var pattern = /orange/;
console.log(pattern.test("orange")); // true

var patternIgnoreCase = /orange/i;
console.log(patternIgnoreCase.test("Orange")); // true

var patternGlobal = /orange/ig;
console.log(patternGlobal.test("Orange Juice")); // true
```

如果我们只能测试模式是否与一个字符串匹配，那就没什么意思了。让我们看看如何表达更复杂的模式。

# 精确匹配

任何不是特殊正则字符或运算符的连续字符都代表一个字符字面量：

```js
var pattern = /orange/;
```

我们的意思是`o`后面跟着`r`，后面跟着`a`，后面跟着`n`，后面跟着……—你应该明白了。当我们使用正则表达式时，我们很少使用精确匹配，因为那就像是比较两个字符串。精确匹配模式有时被称为简单模式。

# 从一类字符中匹配

如果你想匹配一组字符，你可以在`[]`里放置这一组字符。例如，`[abc]`就意味着任何字符`a`、`b`或`c`：

```js
var pattern = /[abc]/;
console.log(pattern.test('a')); //true
console.log(pattern.test('d')); //false
```

你可以指定想匹配除模式以外的任何内容，通过在模式的开头添加一个`^`（感叹号）来实现：

```js
var pattern = /[^abc]/;
console.log(pattern.test('a')); //false
console.log(pattern.test('d')); //true
```

这个模式的一个关键变体是值的范围。如果我们想匹配一系列连续的字符或数字，我们可以使用以下的模式：

```js
var pattern = /[0-5]/;
console.log(pattern.test(3)); //true
console.log(pattern.test(12345)); //true
console.log(pattern.test(9)); //false
console.log(pattern.test(6789)); //false
console.log(/[0123456789]/.test("This is year 2015")); //true
```

特殊字符，比如`$`和`.`，要么代表与自身以外的匹配，要么是修饰前面项的运算符。实际上，我们已经看到了`[`, `]`, `-`, 和`^`字符如何用来表示它们字面值以外的含义。

那么我们如何指定想匹配一个字面`[`或`$`或`^`或其他特殊字符呢？在正则表达式中，反斜杠字符转义它后面的任何字符，使其成为一个字面匹配项。所以`\[`指定了一个对`[`字符的精确匹配，而不是字符类表达式的开始。双反斜杠（`\\`）匹配一个单反斜杠。

在前面的例子中，我们看到了`test()`方法，它基于匹配到的模式返回`true`或`false`。有时你想访问特定模式的各个出现。在这种情况下，`exec()`方法就派上用场了。

`exec()`方法接收一个字符串作为参数，返回一个包含所有匹配项的数组。考虑以下例子：

```js
var strToMatch = 'A Toyota! Race fast, safe car! A Toyota!'; 
var regExAt = /Toy/;
var arrMatches = regExAt.exec(strToMatch); 
console.log(arrMatches);
```

```js
['Toy']; if you want all the instances of the pattern Toy, you can use the g (global) flag as follows:
```

```js
var strToMatch = 'A Toyota! Race fast, safe car! A Toyota!'; 
var regExAt = /Toy/g;
var arrMatches = regExAt.exec(strToMatch); 
console.log(arrMatches);
```

这将返回原文中所有单词`oyo`的出现。String 对象包含`match()`方法，其功能与`exec()`方法类似。在 String 对象上调用`match()`方法，把正则表达式作为参数传给它。考虑以下例子：

```js
var strToMatch = 'A Toyota! Race fast, safe car! A Toyota!'; 
var regExAt = /Toy/;
var arrMatches = strToMatch.match(regExAt);
console.log(arrMatches);
```

在这个例子中，我们在 String 对象上调用`match()`方法。我们把正则表达式作为参数传给`match()`方法。这两种情况的结果是一样的。

另一个 String 对象的方法是`replace()`。它用一个不同的字符串替换所有子字符串的出现：

```js
var strToMatch = 'Blue is your favorite color ?'; 
var regExAt = /Blue/;
console.log(strToMatch.replace(regExAt, "Red"));
//Output- "Red is your favorite color ?"
```

你可以把一个函数作为`replace()`方法的第二个参数。`replace()`函数把匹配到的文本作为参数，并返回用作替换的文本：

```js
var strToMatch = 'Blue is your favorite color ?'; 
var regExAt = /Blue/;
console.log(strToMatch.replace(regExAt, function(matchingText){
  return 'Red';
}));
//Output- "Red is your favorite color ?"
```

字符串对象的`split()`方法也接受一个正则表达式参数，并返回一个包含在原字符串分割后生成的所有子字符串的数组：

```js
var sColor = 'sun,moon,stars';
var reComma = /\,/;
console.log(sColor.split(reComma));
//Output - ["sun", "moon", "stars"]
```

我们需要在逗号之前加上反斜杠，因为正则表达式中逗号有特殊含义，如果我们想直接使用它，就需要转义它。

使用简单的字符类，你可以匹配多个模式。例如，如果你想匹配`cat`、`bat`和`fat`，以下片段展示了如何使用简单的字符类：

```js
var strToMatch = 'wooden bat, smelly Cat,a fat cat';
var re = /[bcf]at/gi;
var arrMatches = strToMatch.match(re);
console.log(arrMatches);
//["bat", "Cat", "fat", "cat"]
```

正如你所看到的，这种变化打开了编写简洁正则表达式模式的可能性。看下面的例子：

```js
var strToMatch = 'i1,i2,i3,i4,i5,i6,i7,i8,i9';
var re = /i[0-5]/gi;
var arrMatches = strToMatch.match(re);
console.log(arrMatches);
//["i1", "i2", "i3", "i4", "i5"]
```

在这个例子中，我们匹配匹配字符的数字部分，范围为`[0-5]`，因此我们从`i0`得到匹配到`i5`。您还可以使用否定类`^`过滤其余的匹配：

```js
var strToMatch = 'i1,i2,i3,i4,i5,i6,i7,i8,i9';
var re = /i[⁰-5]/gi;
var arrMatches = strToMatch.match(re);
console.log(arrMatches);
//["i6", "i7", "i8", "i9"]
```

注意我们是如何只否定范围子句而不是整个表达式的。

几个字符组有快捷方式。例如，快捷方式`\d`与`[0-9]`相同：

| 表示法 | 意义 |
| --- | --- |
| `\d` | 任何数字字符 |
| `\w` | 字母数字字符（单词字符） |
| `\s` | 任何空白字符（空格、制表符、换行符等） |
| `\D` | 非数字字符 |
| `\W` | 非字母数字字符 |
| `\S` | 非空白字符 |
| `.` | 除换行符外的任何字符 |

这些快捷方式在编写简洁的正则表达式中很有价值。考虑这个例子：

```js
var strToMatch = '123-456-7890';
var re = /[0-9][0-9][0-9]-[0-9][0-9][0-9]/;
var arrMatches = strToMatch.match(re);
console.log(arrMatches);
//["123-456"]
```

这个表达式看起来确实有点奇怪。我们可以用`\d`替换`[0-9]`，使这变得更易读：

```js
var strToMatch = '123-456-7890';
var re = /\d\d\d-\d\d\d/;
var arrMatches = strToMatch.match(re);
console.log(arrMatches);
//["123-456"]
```

然而，你很快就会看到还有更好的方法来这样做。

# 重复出现

到目前为止，我们看到了如何匹配固定字符或数字模式。大多数时候，你希望处理模式的某些重复特性。例如，如果我想要匹配 4 个`a`，我可以写`/aaaa/`，但如果我想指定一个可以匹配任意数量`a`的模式呢？

正则表达式为您提供了各种重复量词。重复量词让我们指定特定模式可以出现的次数。我们可以指定固定值（字符应出现 *n* 次）和变量值（字符可以出现至少 *n* 次，直到它们出现 *m* 次）。以下表格列出了各种重复量词：

+   `?`: 要么出现 0 次要么出现 1 次（将出现标记为可选）

+   `*`: 0 或多个出现

+   `+`: 1 或多个出现

+   `{n}`: 正好 `n` 次出现

+   `{n,m}`: 在 `n` 和 `m` 之间的出现

+   `{n,}`: 至少出现 `n` 次

+   `{,n}`: 0 到 `n` 次出现

在以下示例中，我们创建一个字符`u`可选（出现 0 或 1 次）的模式：

```js
var str = /behaviou?r/;
console.log(str.test("behaviour"));
// true
console.log(str.test("behavior"));
// true
```

把`/behaviou?r/`表达式看作是 0 或 1 次字符`u`的出现有助于阅读。重复量词 succeeds 了我们想要重复的字符。让我们尝试一些更多例子：

```js
console.log(/'\d+'/.test("'123'")); // true
```

你应该读取并解释`\d+`表达式，就像`'`是字面字符匹配，`\d`匹配字符`[0-9]`，`+`量词将允许一个或多个出现，而`'`是字面字符匹配。

您还可以使用`()`对字符表达式进行分组。观察以下示例：

```js
var heartyLaugh = /Ha+(Ha+)+/i;
console.log(heartyLaugh.test("HaHaHaHaHaHaHaaaaaaaaaaa"));
//true
```

让我们把前面的表达式分解成更小的块，以了解这里发生了什么：

+   `H`：字面字符匹配

+   `a+`：字符`a`的一个或多个出现

+   `(`：表达式组的开始

+   `H`：字面字符匹配

+   `a+`：字符`a`的一个或多个出现

+   `)`：表达式组的结束

+   `+`：表达式组（`Ha+`）的一个或多个出现

现在更容易看出分组是如何进行的。如果我们必须解释表达式，有时读出表达式是有帮助的，如前例所示。

通常，你想匹配一组字母或数字本身，而不仅仅是作为子字符串。当你匹配的词不是其他任何词的一部分时，这是一个相当常见的用例。我们可以通过使用`\b`模式来指定单词边界。`\b`的单词边界匹配一侧是单词字符（字母、数字或下划线）而另一侧不是的位置。考虑以下示例。

以下是一个简单的字面匹配。如果`cat`是子字符串的一部分，这个匹配也会成功：

```js
console.log(/cat/.test('a black cat')); //true
```

然而，在下面的示例中，我们通过在单词`cat`前标示`\b`来定义一个单词边界——这意味着我们只想匹配`cat`作为一个单词而不是一个子字符串。边界是在`cat`之前建立的，因此在文本`a black cat`中找到了匹配项：

```js
console.log(/\bcat/.test('a black cat')); //true
```

当我们对单词`tomcat`使用相同的边界时，我们得到一个失败的匹配，因为在单词`tomcat`中`cat`之前没有单词边界：

```js
console.log(/\bcat/.test('tomcat')); //false
```

在单词`tomcat`中，`cat`之后有一个单词边界，因此以下是一个成功的匹配：

```js
console.log(/cat\b/.test('tomcat')); //true
```

在以下示例中，我们在单词`cat`的前后都定义了单词边界，以表示我们想要`cat`作为一个有前后边界的独立单词：

```js
console.log(/\bcat\b/.test('a black cat')); //true
```

基于相同逻辑，以下匹配失败，因为在单词`concatenate`中`cat`前后的边界不存在：

```js
console.log(/\bcat\b/.test("concatenate")); //false
```

`exec()`方法在获取关于找到匹配的信息方面很有用，因为它返回一个包含关于匹配的信息的对象。`exec()`返回的对象有一个`index`属性，告诉我们成功匹配在字符串中的开始位置。这在许多方面都是有用的：

```js
var match = /\d+/.exec("There are 100 ways to do this");
console.log(match);
// ["100"]
console.log(match.index);
// 10
```

## 替代方案——或

使用`|`（管道）字符可以表示替代方案。例如，`/a|b/`匹配`a`或`b`字符，而`/(ab)+|(cd)+/`匹配`ab`或`cd`的一个或多个出现。

# 开始和结束

经常，我们可能希望确保模式在字符串的开始处或 perhaps 在字符串的结束处匹配。当正则表达式的第一个字符是井号时（`^`），它将匹配固定在字符串的开始处，例如`/^test/`仅当`test`子字符串出现在要匹配的字符串的开头时才匹配。同样，美元符号（`$`）表示模式必须出现在字符串的末尾：`/test$/`。

使用`^`和`$`指示指定的模式必须包含整个候选字符串：`/^test$/`。

# 反向引用

在表达式计算之后，每个组都存储起来以供以后使用。这些值称为反向引用。反向引用通过从左到右遇到左括号字符的顺序创建并编号。你可以将反向引用视为与正则表达式中的项成功匹配的字符串的部分。

引用后缀的表示方法是一个反斜杠，后面跟着要引用的捕获组的编号，从 1 开始，例如`\1`、`\2`等等。

一个例子可能是`/^([XYZ])a\1/`，它匹配一个以`X`、`Y`或`Z`中的任何一个字符开头，后面跟着一个`a`，再后面跟着与第一个捕获组匹配的任何字符的字符串。这与`/[XYZ] a[XYZ]/`非常不同。`a`后面的字符不能是`X`、`Y`或`Z`中的任何一个，而必须是触发第一个字符匹配的那个。反向引用用于字符串的`replace()`方法，使用特殊字符序列`$1`、`$2`等等。假设你想把`1234 5678`字符串改为`5678 1234`。以下代码实现此功能：

```js
var orig = "1234 5678";
var re = /(\d{4}) (\d{4})/;
var modifiedStr = orig.replace(re, "$2 $1"); 
console.log(modifiedStr); //outputs "5678 1234" 
```

在这个例子中，正则表达式有两个组，每个组都有四个数字。在`replace()`方法的第二个参数中，`$2`等于`5678`，`$1`等于`1234`，对应于它们在表达式中出现的顺序。

# 贪婪与懒惰量词

我们迄今为止讨论的所有量词都是贪婪的。一个贪婪的量词从整个字符串开始寻找匹配。如果没有找到匹配，它会删除字符串中的最后一个字符并重新尝试匹配。如果没有再次找到匹配，它将再次删除最后一个字符，并重复这个过程，直到找到匹配或者字符串剩下没有字符。

例如，`\d+`模式将匹配一个或多个数字。例如，如果你的字符串是`123`，贪婪匹配将匹配`1`、`12`和`123`。贪婪模式`h`.`+l`将在字符串`hello`中匹配`hell`—这是可能的最长字符串匹配。由于`\d+`是贪婪的，它会尽可能多地匹配数字，因此匹配将是`123`。

与贪婪量词相比，懒惰量词尽可能少地匹配量词化的令牌。你可以在正则表达式中添加一个问号（`?`）使其变得懒惰。一个懒惰的模式`h.?l`将在字符串`hello`中匹配`hel`—这是可能的最短字符串。

`\w*?X`模式将匹配零个或多个单词，然后匹配一个`X`。然而，在`*`后面的问号表示应该尽可能少地匹配字符。对于字符串`abcXXX`，匹配可以是`abcX`、`abcXX`或`abcXXX`。哪一个应该被匹配？由于`*?`是懒惰的，尽可能少地匹配字符，因此匹配是`abcX`。

有了这些必要的信息，让我们尝试使用正则表达式解决一些常见问题。

从字符串的开始和结束去除多余的空格是一个非常常见的用例。由于字符串对象直到最近才有一个`trim()`方法，因此一些 JavaScript 库为没有`String.trim()`方法的旧浏览器提供并使用了字符串截取的实现。最常用的方法看起来像下面的代码：

```js
function trim(str) {
  return (str || "").replace(/^\s+|\s+$/g, "");
}
console.log("--"+trim("   test    ")+"--");
//"--test--"
```

如果我们想用一个空格替换重复的空格怎么办？

```js
re=/\s+/g;
console.log('There are    a lot      of spaces'.replace(re,' '));
//"There are a lot of spaces"
```

```js
As you can see, regular expressions can prove to be a Swiss army knife in your JavaScript arsenal. Careful study and practice will be extremely rewarding for you in the long run.
```

# 数组

数组是一个有序的值集合。你可以用一个名字和索引来引用数组元素。以下是 JavaScript 中创建数组的三个方法：

```js
var arr = new Array(1,2,3);
var arr = Array(1,2,3);
var arr = [1,2,3];
```

当这些值被指定时，数组初始化为这些值作为数组的元素。数组的`length`属性等于参数的数量。方括号语法称为数组字面量。这是一种更简短且更推荐的方式来初始化数组。

如果你想初始化一个只有一个元素且该元素碰巧是数字的数组，你必须使用数组字面量语法。如果你将一个单一的数字值传递给`Array()`构造函数或函数，JavaScript 将这个参数视为数组的长度，而不是单个元素：

```js
var arr = [10];
var arr = Array(10); // Creates an array with no element, but with arr.length set to 10
// The above code is equivalent to
var arr = [];
arr.length = 10;
```

JavaScript 没有显式的数组数据类型。然而，你可以使用预定义的`Array`对象及其方法来处理应用程序中的数组。`Array`对象有各种方式操作数组的方法，如连接、反转和排序它们。它有一个属性来确定数组长度和其他用于正则表达式的属性。

你可以通过给它的元素赋值来填充一个数组：

```js
var days = [];
days[0] = "Sunday";
days[1] = "Monday";
```

你也可以在创建数组时填充它：

```js
var arr_generic = new Array("A String", myCustomValue, 3.14);
var fruits = ["Mango", "Apple", "Orange"]
```

在大多数语言中，数组的元素都必须是同一类型。JavaScript 允许数组包含任何类型的值：

```js
var arr = [
  'string', 42.0, true, false, null, undefined,
  ['sub', 'array'], {object: true}, NaN
]; 
```

你可以使用元素的索引号码来引用`Array`的一个元素。例如，假设你定义了以下数组：

```js
var days = ["Sunday", "Monday", "Tuesday"]
```

然后你将数组的第一个元素称为`colors[0]`，第二个元素称为`colors[1]`。元素的索引从`0`开始。

JavaScript 内部将数组元素作为标准对象属性存储，使用数组索引作为属性名。`length`属性是不同的。`length`属性总是返回最后一个元素索引加一。正如我们讨论的，JavaScript 数组索引是基于 0 的：它们从`0`开始，而不是`1`。这意味着`length`属性将是数组中存储的最高索引加一：

```js
var colors = [];
colors[30] = ['Green'];
console.log(colors.length); // 31
```

你还可以赋值给`length`属性。如果写入的值比存储的项目数少，数组就会被截断；写入`0`则会清空它：

```js
var colors = ['Red', 'Blue', 'Yellow'];
console.log(colors.length); // 3
colors.length = 2;
console.log(colors); // ["Red","Blue"] - Yellow has been removed
colors.length = 0;
console.log(colors); // [] the colors array is empty
colors.length = 3;
console.log(colors); // [undefined, undefined, undefined]
```

如果你查询一个不存在的数组索引，你会得到`undefined`。

一个常见的操作是遍历数组的值，以某种方式处理每一个值。这样做最简单的方式如下：

```js
var colors = ['red', 'green', 'blue']; 
for (var i = 0; i < colors.length; i++) { 
  console.log(colors[i]); 
}
```

`forEach()` 方法提供了另一种遍历数组的方式：

```js
var colors = ['red', 'green', 'blue'];
colors.forEach(function(color) {
  console.log(color);
});
```

传递给 `forEach()` 的函数对数组中的每个项目执行一次，将数组项目作为函数的参数传递。在 `forEach()` 循环中不会遍历未赋值的值。

`Array` 对象有一组实用的方法。这些方法允许操作数组中存储的数据。

`concat()` 方法将两个数组合并成一个新数组：

```js
var myArray = new Array("33", "44", "55");
myArray = myArray.concat("3", "2", "1"); 
console.log(myArray);
// ["33", "44", "55", "3", "2", "1"]
```

`join()` 方法将数组的所有元素合并成一个字符串。这在处理列表时可能很有用。默认的分隔符是逗号 (`,`)：

```js
var myArray = new Array('Red','Blue','Yellow');
var list = myArray.join(" ~ "); 
console.log(list);
//"Red ~ Blue ~ Yellow"
```

`pop()` 方法从数组中移除最后一个元素，并返回该元素。这与栈的 `pop()` 方法类似：

```js
var myArray = new Array("1", "2", "3");
var last = myArray.pop(); 
// myArray = ["1", "2"], last = "3"
```

`push()` 方法向数组的末尾添加一个或多个元素，并返回数组的结果长度：

```js
var myArray = new Array("1", "2");
myArray.push("3"); 
// myArray = ["1", "2", "3"]
```

`shift()` 方法从数组中移除第一个元素，并返回该元素：

```js
var myArray = new Array ("1", "2", "3");
var first = myArray.shift(); 
// myArray = ["2", "3"], first = "1"
```

`unshift()` 方法向数组的开头添加一个或多个元素，并返回数组的新长度：

```js
var myArray = new Array ("1", "2", "3");
myArray.unshift("4", "5"); 
// myArray = ["4", "5", "1", "2", "3"]
```

`reverse()` 方法反转或转置数组的元素——第一个数组元素变为最后一个，最后一个变为第一个：

```js
var myArray = new Array ("1", "2", "3");
myArray.reverse(); 
// transposes the array so that myArray = [ "3", "2", "1" ]
```

`sort()` 方法对数组的元素进行排序：

```js
var myArray = new Array("A", "C", "B");
myArray.sort(); 
// sorts the array so that myArray = [ "A","B","c" ]
```

`sort()` 方法可以接受一个回调函数作为可选参数，以定义元素如何进行比较。该函数比较两个值并返回三个值之一。让我们研究以下函数：

+   `indexOf(searchElement[, fromIndex])`：此方法在数组中搜索 `searchElement` 并返回第一个匹配项的索引：

    ```js
    var a = ['a', 'b', 'a', 'b', 'a','c','a'];
    console.log(a.indexOf('b')); // 1
    // Now try again, starting from after the last match
    console.log(a.indexOf('b', 2)); // 3
    console.log(a.indexOf('1')); // -1, 'q' is not found
    ```

+   `lastIndexOf(searchElement[, fromIndex])`：此方法类似于 `indexOf()`，但只从后向前搜索：

    ```js
    var a = ['a', 'b', 'c', 'd', 'a', 'b'];
    console.log(a.lastIndexOf('b')); //  5
    // Now try again, starting from before the last match
    console.log(a.lastIndexOf('b', 4)); //  1
    console.log(a.lastIndexOf('z')); //  -1
    ```

既然我们已经深入讲解了 JavaScript 数组，那么让我向您介绍一个名为 **Underscore.js** 的绝佳库（[`underscorejs.org/`](http://underscorejs.org/)）。Underscore.js 提供了一系列极其有用的函数编程助手，使您的代码更加清晰和功能化。

我们假设您熟悉**Node.js**；在这种情况下，通过 npm 安装 Underscore.js：

```js
npm install underscore
```

由于我们正在将 Underscore 作为 Node 模块进行安装，因此我们将通过在 Node.js 上运行 `.js` 文件来输入所有示例。您也可以使用 **Bower** 安装 Underscore。

类似于 jQuery 的 `$` 模块，Underscore 带有一个 `_` 模块的定义。您将使用这个模块引用调用所有函数。

将以下代码输入文本文件并命名为 `test_.js`：

```js
var _ = require('underscore');
function print(n){
  console.log(n);
}
_.each([1, 2, 3], print);
//prints 1 2 3
```

以下是不使用 underscore 库中的 `each()` 函数的写法：

```js
var myArray = [1,2,3];
var arrayLength = myArray.length;
for (var i = 0; i < arrayLength; i++) {
  console.log(myArray[i]);
}
```

这里所展示的是一个强大的功能性结构，使代码更加优雅和简洁。你可以明显看出传统方法是冗长的。像 Java 这样的许多语言都受到这种冗长的影响。它们正在逐渐接受函数式编程范式。作为 JavaScript 程序员，我们尽可能地将这些思想融入到我们的代码中是非常重要的。

前面例子中看到的`each()`函数遍历元素列表，依次将每个元素传递给迭代函数。每次迭代函数调用时，都会传入三个参数（元素、索引和列表）。在前面的例子中，`each()`函数遍历数组`[1,2,3]`，对于数组中的每个元素，`print`函数都会被调用，并传入数组元素作为参数。这是访问数组中所有元素的方便方法，代替传统的循环机制。

`range()`函数创建整数列表。如果省略起始值，默认为`0`，步长默认为`1`。如果你想要一个负范围，使用负步长：

```js
var _ = require('underscore');
console.log(_.range(10));
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
console.log(_.range(1, 11));
//[ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ]
console.log(_.range(0, 30, 5));
//[ 0, 5, 10, 15, 20, 25 ]
console.log(_.range(0, -10, -1));
//[ 0, -1, -2, -3, -4, -5, -6, -7, -8, -9 ]
console.log(_.range(0));
//[]
```

默认情况下，`range()`用整数填充数组，但用一点小技巧，你也可以用其他数据类型填充：

```js
console.log(_.range(3).map(function () { return 'a' }) );
[ 'a', 'a', 'a' ]
```

这是一种快速方便的方法来创建和初始化一个带有值的数组。我们经常通过传统循环来做这件事。

`map()`函数通过映射每个列表中的值到一个转换函数，生成一个新的值数组。考虑以下示例：

```js
var _ = require('underscore');
console.log(_.map([1, 2, 3], function(num){ return num * 3; }));
//[3,6,9]
```

`reduce()`函数将一个值列表减少到一个单一的值。初始状态由迭代函数传递，每个连续步骤由迭代函数返回。以下示例展示了使用方法：

```js
var _ = require('underscore');
var sum = _.reduce([1, 2, 3], function(memo, num){console.log(memo,num);return memo + num; }, 0);
console.log(sum);
```

在这个例子中，`console.log(memo,num);`这行代码只是为了更清楚地说明想法。输出结果如下：

```js
0 1
1 2
3 3
6
```

最终输出是`*1+2+3=6*`的和。正如你所见，两个值被传递到迭代函数中。在第一次迭代中，我们调用迭代函数并传入两个值`(0,1)`——`memo`在调用`reduce()`函数时的默认值是`0`，`1`是列表的第一个元素。在函数中，我们计算`memo`和`num`的和并返回中间的`sum`，这个`sum`将被`iterate()`函数作为`memo`参数使用——最终，`memo`将累积`sum`。理解这个概念对于了解如何使用中间状态来计算最终结果很重要。

`filter()`函数遍历整个列表，返回满足条件的所有元素的数组。看看下面的例子：

```js
var _ = require('underscore');
var evens = _.filter([1, 2, 3, 4, 5, 6], function(num){ return num % 2 == 0; });
console.log(evens);
```

`filter()`函数的迭代函数应该返回一个真值。结果的`evens`数组包含所有满足真值测试的元素。

`filter()`函数的反义词是`reject()`。正如名字 suggest，它遍历列表并忽略满足真值测试的元素：

```js
var _ = require('underscore');
var odds = _.reject([1, 2, 3, 4, 5, 6], function(num){ return num % 2 == 0; });
console.log(odds);
//[ 1, 3, 5 ]
```

我们使用了与上一个例子相同的代码，但这次用`reject()`方法而不是`filter()`——结果正好相反。

`contains()`函数是一个有用的小函数，如果值在列表中，就返回`true`；否则，返回`false`：

```js
var _ = require('underscore');
console.log(_.contains([1, 2, 3], 3));
//true
```

一个非常实用的函数，我已经喜欢上了，就是 `invoke()`。它在列表中的每个元素上调用一个特定的函数。我无法告诉你自从偶然发现它以来我使用了多少次。让我们研究以下示例：

```js
var _ = require('underscore');
console.log(_.invoke([[5, 1, 7], [3, 2, 1]], 'sort'));
//[ [ 1, 5, 7 ], [ 1, 2, 3 ] ]
```

在这个例子中，`Array` 对象的 `sort()` 方法被应用于数组中的每个元素。注意这将失败：

```js
var _ = require('underscore');
console.log(_.invoke(["new","old","cat"], 'sort'));
//[ undefined, undefined, undefined ]
```

这是因为 `sort` 方法不是字符串对象的一部分。然而，这完全有效：

```js
var _ = require('underscore');
console.log(_.invoke(["new","old","cat"], 'toUpperCase'));
//[ 'NEW', 'OLD', 'CAT' ]
```

这是因为 `toUpperCase()` 是字符串对象的方法，列表中的所有元素都是字符串类型。

`uniq()` 函数返回去除原始数组所有重复项后的数组：

```js
var _ = require('underscore');
var uniqArray = _.uniq([1,1,2,2,3]);
console.log(uniqArray);
//[1,2,3]
```

`partition()` 函数将数组分成两部分；一部分是满足谓词的元素，另一部分是不满足谓词的元素：

```js
var _ = require('underscore');
function isOdd(n){
  return n%2==0;
}
console.log(_.partition([0, 1, 2, 3, 4, 5], isOdd));
//[ [ 0, 2, 4 ], [ 1, 3, 5 ] ]
```

```js
[1,2,3]—this is a helpful method to eliminate any value from a list that can cause runtime exceptions.
```

`without()` 函数返回一个删除特定值所有实例的数组副本：

```js
var _ = require('underscore');
console.log(_.without([1,2,3,4,5,6,7,8,9,0,1,2,0,0,1,1],0,1,2));
//[ 3, 4, 5, 6, 7, 8, 9 ]
```

# 映射（Maps）

```js
Map type and their usage:
```

```js
var founders = new Map();
founders.set("facebook", "mark");
founders.set("google", "larry");
founders.size; // 2
founders.get("twitter"); // undefined
founders.has("yahoo"); // false

for (var [key, value] of founders) {
  console.log(key + " founded by " + value);
}
// "facebook founded by mark"
// "google founded by larry"
```

# 集合

ECMAScript 6 引入了集合。集合是值的集合，并且可以按照它们的元素插入顺序进行迭代。关于集合的一个重要特征是，集合中的值只能出现一次。

以下代码片段展示了集合的一些基本操作：

```js
var mySet = new Set();
mySet.add(1);
mySet.add("Howdy");
mySet.add("foo");

mySet.has(1); // true
mySet.delete("foo");
mySet.size; // 2

for (let item of mySet) console.log(item);
// 1
// "Howdy"
```

我们简要讨论过，JavaScript 数组并不是真正意义上的数组。在 JavaScript 中，数组是具有以下特征的对象：

+   `length` 属性

+   继承自 `Array.prototype` 的函数（我们将在下一章讨论这个）

+   对数字键的特殊处理

当我们写数组索引作为数字时，它们会被转换为字符串——`arr[0]` 内部变成了 `arr["0"]`。由于这一点，当我们使用 JavaScript 数组时，我们需要注意一些事情：

+   通过索引访问数组元素并不是一个常数时间操作，比如在 C 语言中。因为数组实际上是键值映射，访问将取决于映射的布局和其他因素（冲突等）。

+   JavaScript 数组是稀疏的（大多数元素都有默认值），这意味着数组中可能会有间隙。为了理解这一点，看看以下代码片段：

    ```js
    var testArr=new Array(3);
    console.log(testArr); 
    ```

    你会看到输出是 `[undefined, undefined, undefined]`——`undefined` 是数组元素存储的默认值。

考虑以下示例：

```js
var testArr=[];
testArr[3] = 10;
testArr[10] = 3;
console.log(testArr);
// [undefined, undefined, undefined, 10, undefined, undefined, undefined, undefined, undefined, undefined, 3]
```

你可以看到这个数组中有间隙。只有两个元素有值，其余的都是使用默认值填充的间隙。了解这一点可以帮助你避免一些问题。使用 `for...in` 循环迭代数组可能会导致意外的结果。考虑以下示例：

```js
var a = [];
a[5] = 5;
for (var i=0; i<a.length; i++) {
  console.log(a[i]);
}
// Iterates over numeric indexes from 0 to 5
// [undefined,undefined,undefined,undefined,undefined,5]

for (var x in a) {
  console.log(x);
}
// Shows only the explicitly set index of "5", and ignores 0-4
```

# 风格问题

和前面章节一样，我们将花些时间讨论创建数组时的风格考虑。

+   使用字面量语法创建数组：

    ```js
    // bad
    const items = new Array();
    // good
    const items = [];
    ```

+   使用 `Array#push` 而不是直接赋值来向数组中添加项目：

    ```js
    const stack = [];
    // bad
    stack[stack.length] = 'pushme';
    // good
    stack.push('pushme');
    ```

# 总结

随着 JavaScript 作为一种语言的成熟，其工具链也变得更加健壮和有效。经验丰富的程序员很少会避开像 Underscore.js 这样的库。随着我们看到更多高级主题，我们将继续探索更多这样的多功能库，这些库可以使你的代码更加紧凑、易读且性能更优。我们研究了正则表达式——它们在 JavaScript 中是第一类对象。一旦你开始理解`RegExp`，你很快就会发现自己更多地使用它们来使你的代码更加简洁。在下一章，我们将探讨 JavaScript 对象表示法以及 JavaScript 原型继承是如何为面向对象编程提供一种新的视角。
