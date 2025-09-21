# 4

# 逻辑语句

到目前为止，我们的代码相当静态。每次执行它都会做同样的事情。在本章中，这一切都将改变。我们将处理逻辑语句。逻辑语句允许我们在代码中创建多个路径。根据某个表达式的结果，我们将遵循一个代码路径或另一个。

有不同的逻辑语句，我们将在本章中介绍它们。我们将从`if`和`if else`语句开始。然后我们将处理三元运算符，最后我们将处理的是`switch`语句。

在旅途中，我们将涵盖以下主题：

+   `if`和`if else`语句

+   `else if`语句

+   条件三元运算符

+   switch 语句

    注意：练习、项目和自我检查测验的答案可以在*附录*中找到。

# `if`和`if else`语句

我们可以使用`if`和`if else`语句在代码中做出决定。这非常类似于以下模板：

*如果*某个条件为真，则*将发生某个动作*，否则*将发生另一个动作**

例如，*如果*下雨，我会带上我的伞，*否则*我会把伞留在家里。在代码中并没有太大的不同：

```js
let rain = true;
if(rain){
  console.log("** Taking my umbrella when I need to go outside **");
} else {
  console.log("** I can leave my umbrella at home **");
} 
```

在这种情况下，`rain`的值为`true`。因此，它将在控制台记录：

```js
** Taking my umbrella when I need to go outside ** 
```

但让我们先退一步，看看语法。我们以单词"if"开始。之后，我们得到括号内的某个东西。括号之间的一切都将被转换为布尔值。如果这个布尔值的值为`true`，它将执行与`if`关联的代码块。你可以通过大括号来识别这个块。

下一个块是可选的；它是一个`else`块。它以单词"else"开始，并且仅在布尔值为`false`时执行。如果没有`else`块，并且条件评估为`false`，则程序将跳到`if`下面的代码。

这两个块中只有一个将被执行；当表达式为真时执行`if`块，当表达式为假时执行`else`块：

```js
if(expression) {
  // code associated with the if block
  // will only be executed if the expression is true
} else {
  // code associated with the else block
  // we don't need an else block, it is optional
  // this code will only be executed if the expression is false
} 
```

这里有一个另一个例子。如果年龄低于 18 岁，则在控制台记录访问被拒绝，否则在控制台记录该人获准进入：

```js
if(age < 18) {
  console.log("We're very sorry, but you can't get in under 18");
} else {
  console.log("Welcome!");
} 
```

与`if`语句相关的一个常见的编码错误。我在下面的代码片段中犯了这个错误。你能看出这段代码做了什么吗？

```js
let hobby = "dancing";
if(hobby == "coding"){
  console.log("** I love coding too! **");
} else {
  console.log("** Can you teach me that? **");
} 
```

它将记录以下内容：

```js
** I love coding too! ** 
```

这可能会让你感到惊讶。这里的问题是`if`语句中的单个等号。它不是评估条件，而是将`coding`赋值给`hobby`。然后它将`coding`转换为布尔值，由于它不是一个空字符串，所以它将是`true`，因此`if`块将被执行。所以，请始终记住在这种情况下使用双等号。

让我们通过一个练习来测试我们的知识。

## 练习 4.1

1.  创建一个具有布尔值的变量。

1.  将变量的值输出到控制台。

1.  检查变量是否为真，如果是，则使用以下语法在控制台输出一条消息：

    ```js
    if(myVariable){
    //action
    } 
    ```

1.  添加另一个带有变量前缀 `!` 的 `if` 语句来检查条件是否不成立，并创建一个在这种情况下将被打印到控制台的消息。你应该有两个 `if` 语句，一个带有 `!`，另一个不带。你也可以使用一个 `if` 和一个 `else` 语句来代替——尝试一下！

1.  将变量改为相反的值，看看结果如何变化。

# `else if` 语句

`if` 语句的一个变体是带有多个 `else if` 块的 `if` 语句。在某些情况下，这可能会更有效，因为你总是只会执行一个或零个块。如果你有很多堆叠在一起的 `if else` 语句，它们将被评估并可能执行，即使上面的某个语句已经有一个条件评估为真并继续执行相关的代码块。

这里是书面模板：

*如果 *一个值落在某个类别中*，则 *将发生某些操作*，否则如果 *值落在与上一个语句不同的类别中*，则 *将发生某些操作*，否则如果 *值落在与上述任一括号不同的类别中*，则 *将发生某些操作**

例如，考虑以下语句，以确定票价应该是多少。如果一个人年龄小于 3 岁，则免费入场，否则如果一个人年龄大于 3 岁且小于 12 岁，则入场费为 5 美元，否则如果一个人年龄大于 12 岁且小于 65 岁，则入场费为 10 美元，否则如果一个人年龄为 65 岁或以上，则入场费为 7 美元：

```js
let age = 10;
let cost = 0;
let message;
if (age < 3) {
    cost = 0;
    message = "Access is free under three.";
} else if (age >= 3 && age < 12) {
    cost = 5;
    message ="With the child discount, the fee is 5 dollars";
} else if (age >= 12 && age < 65) {
    cost = 10;
    message ="A regular ticket costs 10 dollars.";
} else {
    cost = 7;
    message ="A ticket is 7 dollars.";
}
console.log(message);
console.log("Your Total cost "+cost); 
```

很可能你会认为代码比书面模板更容易阅读。在这种情况下，做得很好！你已经开始像 JavaScript 开发者一样思考了。

代码从上到下执行，并且只有一个块会被执行。一旦遇到一个为真的表达式，其他块将被忽略。这就是为什么我们也可以像这样编写我们的示例：

```js
if(age < 3){
  console.log("Access is free under three.");
} else if(age < 12) {
  console.log("With the child discount, the fee is 5 dollars");
} else if(age < 65) {
  console.log("A regular ticket costs 10 dollars.");
} else if(age >= 65) {
  console.log("A ticket is 7 dollars.");
} 
```

## 练习 4.2

1.  创建一个提示来询问用户的年龄

1.  将提示响应转换为数字

1.  声明一个 `message` 变量，你将使用它来保存用户的控制台消息

1.  如果输入的年龄等于或大于 21 岁，将 `message` 变量设置为确认入场并允许购买酒精

1.  如果输入的年龄等于或大于 19 岁，将 `message` 变量设置为确认入场但拒绝购买酒精

1.  提供一个默认的 `else` 语句，如果没有任何条件成立，则将 `message` 变量设置为拒绝入场

1.  将 `message` 响应变量输出到控制台

# 条件三元运算符

我们实际上没有在我们的 *第二章* *JavaScript Essentials* 中的运算符部分讨论这个非常重要的运算符。这是因为它有助于首先理解 if else 语句。记住，我们有一个被称为一元运算符的一元运算符，因为它只有一个操作数？这就是为什么我们的三元运算符有这个名字；它有三个操作数。以下是它的模板：

```js
operand1 ? operand2 : operand3; 
```

`operand1` 是要评估的表达式。如果表达式的值为 `true`，则执行 `operand2`。如果表达式的值为 `false`，则执行 `operand3`。在这里，你可以将问号读作“then”，将冒号读作“else”：

```js
expression ? statement for true : statement associated with false; 
```

在心里说它的模板应该是这样的：

*if *operand1*, then *operand2*, else *operand3***

让我们看看几个例子：

```js
let access = age < 18 ? "denied" : "allowed"; 
```

```js
If age is lower than 18, *then* it will assign the value denied, *else* it will assign the value allowed. You can also specify an action in a ternary statement, like this:
```

```js
age < 18 ? console.log("denied") : console.log("allowed"); 
```

这种语法一开始可能会让人困惑。在阅读时，心里想说什么的模板真的可以在这里救命。你只能将这些三元运算符用于非常短的操作，因此在这些情况下最好使用三元运算符，因为它使代码更容易阅读。然而，如果逻辑包含多个比较参数，你必须使用常规的 if-else。

## 练习 4.3

1.  为 ID 变量创建一个布尔值

1.  使用三元运算符创建一个消息变量，该变量将检查他们的 ID 是否有效，并允许或不允许一个人进入场所

1.  将响应输出到控制台

# switch 语句

If else 语句非常适合评估布尔条件。你可以用它们做很多事情，但在某些情况下，最好用 switch 语句来替换它们。这尤其适用于评估四个或五个以上的值。

我们将看到 switch 语句如何帮助我们以及它们的外观。首先，看看这个 if else 语句：

```js
if(activity === "Get up") {
  console.log("It is 6:30AM");
} else if(activity === "Breakfast") {
  console.log("It is 7:00AM");
} else if(activity === "Drive to work") {
  console.log("It is 8:00AM");
} else if(activity === "Lunch") {
  console.log("It is 12.00PM");
} else if(activity === "Drive home") {
  console.log("It is 5:00PM")
} else if(activity === "Dinner") {
  console.log("It is 6:30PM");
} 
```

它是根据我们在做什么来确定时间的。最好使用 switch 语句来实现这一点。switch 语句的语法如下所示：

```js
switch(expression) {
  case value1:
    // code to be executed
    break;
  case value2:
    // code to be executed
    break;
  case value-n:
    // code to be executed
    break;
} 
```

你可以在心里这样读：如果表达式等于 `value1`，则执行该 case 指定的任何代码。如果表达式等于 `value2`，则执行该 case 指定的任何代码，依此类推。

这是我们可以使用 switch 语句以更干净的方式重写我们的长 if else 语句的方法：

```js
switch(activity) {
  case "Get up":
    console.log("It is 6:30AM");
    break;
  case "Breakfast":
    console.log("It is 7:00AM");
    break;
  case "Drive to work":
    console.log("It is 8:00AM");
    break;
  case "Lunch":
    console.log("It is 12:00PM");
    break;  
  case "Drive home":
    console.log("It is 5:00PM");
    break;    
  case "Dinner":
    console.log("It is 6:30PM");
    break;
} 
```

如果我们的活动值为 `Lunch`，它将向控制台输出以下内容：

```js
It is 12:00PM 
```

你可能想知道所有这些 breaks 是怎么回事？如果你在 case 的末尾不使用 `break` 命令，它将执行下一个 case。这将从匹配的 case 开始执行，直到 switch 语句的末尾或我们遇到一个 `break` 语句。这是没有 breaks 的 `Lunch` 活动的 switch 语句的输出：

```js
It is 12:00PM
It is 5:00PM
It is 6:30PM 
```

最后一个注意事项。`switch` 使用严格类型检查（三等号策略）来确定相等性，这会检查值和数据类型。

## 默认情况

我们还没有处理开关的一部分，那就是一个特殊的标签，即`default`。这和 if else 语句的 else 部分非常相似。如果它没有与任何 case 匹配，并且存在默认 case，那么它将执行与默认 case 关联的代码。下面是一个带有默认 case 的 switch 语句的模板：

```js
switch(expression) {
  case value1:
    // code to be executed
    break;
  case value2:
    // code to be executed
    break;
  case value-n:
    // code to be executed
    break;
  default:
    // code to be executed when no cases match
    break;
} 
```

习惯上，将默认 case 放在 switch 语句的最后一个 case，但代码即使放在中间或第一个 case 也能正常工作。然而，我们建议你坚持这些约定，并将其放在最后一个 case，因为这是其他开发者（以及可能未来的你）在以后处理你的代码时预期的。

假设我们的长`if`语句有一个看起来像这样的`else`语句相关联：

```js
if(…) {
  // omitted to avoid making this unnecessarily long
} else {
  console.log("I cannot determine the current time.");
} 
```

switch 语句将看起来像这样：

```js
switch(activity) {
  case "Get up":
    console.log("It is 6:30AM");
    break;
  case "Breakfast":
    console.log("It is 7:00AM");
    break;
  case "Drive to work":
    console.log("It is 8:00AM");
    break;
  case "Lunch":
    console.log("It is 12:00PM");
    break;  
  case "Drive home":
    console.log("It is 5:00PM");
    break;    
  case "Dinner":
    console.log("It is 6:30PM");
    break;
  default:
    console.log("I cannot determine the current time.");
    break;
} 
```

如果活动的值要是不指定为 case 的内容，例如，“看 Netflix”，它将在控制台记录以下内容：

```js
I cannot determine the current time. 
```

### 练习 4.4

如*第一章*中所述，*JavaScript 入门*，JavaScript 函数`Math.random()`将返回一个介于 0 到小于 1 之间的随机数，包括 0 但不包括 1。然后你可以通过乘以结果并将其使用`Math.floor()`向下舍入到最接近的整数来将其缩放到所需的范围；例如，要生成一个介于 0 到 9 之间的随机数：

```js
// random number between 0 and 1
let randomNumber = Math.random(); 
// multiply by 10 to obtain a number between 0 and 10
randomNumber = randomNumber * 10; 
// removes digits past decimal place to provide a whole number
RandomNumber = Math.floor(randomNumber); 
```

在这个练习中，我们将创建一个魔法 8 球随机答案生成器：

1.  首先设置一个变量，将其随机值分配给它。这个值是通过生成 0-5 之间的随机数来分配的，有 6 种可能的结果。你可以随着添加更多结果来增加这个数字。

1.  创建一个提示，可以获取用户输入的字符串值，你可以在最终输出中重复它。

1.  使用 switch 语句创建 6 个响应，每个响应分配给随机数生成器的不同值。

1.  创建一个变量来保存最终响应，这应该是一个打印给用户的句子。你可以为每个 case 分配不同的字符串值，根据随机值的返回结果分配新值。

1.  用户输入问题后，将用户的原问题以及随机选择的 case 响应输出到控制台。

## 混合 case

有时，你可能想要为多个 case 执行完全相同的事情。在 if 语句中，你必须指定所有不同的*或*（`||`）子句。在 switch 语句中，你可以通过将它们堆叠在一起来简单地合并它们，如下所示：

```js
switch(grade){
  case "F":
  case "D":
    console.log("You've failed!");
    break;
  case "C":
  case "B":
    console.log("You've passed!");
    break;
  case "A":
    console.log("Nice!");
    break;
  default:
    console.log("I don't know this grade.");
} 
```

对于`F`和`D`的值，发生的情况相同。对于`C`和`B`也是如此。当`grade`的值为`C`或`B`时，它将在控制台记录以下内容：

```js
You've passed! 
```

这比替代的 if-else 语句更易读：

```js
if(grade === "F" || grade === "D") {
  console.log("You've failed!");
} else if(grade === "C" || grade === "B") {
  console.log("You've passed!");
} else if(grade === "A") {
  console.log("Nice!");
} else {
  console.log("I don't know this grade.");
} 
```

### 练习 4.5

1.  创建一个名为`prize`的变量，并使用提示让用户通过选择 0 到 10 之间的数字来设置值。

1.  将提示响应转换为数字数据类型

1.  创建一个用于输出消息的变量，包含值“我的选择：”

1.  使用 switch 语句（和创造力），根据所选数字提供有关奖励的响应

1.  使用 switch break 添加奖励的合并结果

1.  通过连接你的奖励变量字符串和输出消息字符串将消息返回给用户

# 章节项目

## 评估数字游戏

请求用户输入一个数字，并检查它是否大于、等于或小于代码中的动态数值。将结果输出给用户。

## 朋友检查游戏

请求用户输入一个名字，使用 switch 语句返回一个确认，如果所选的名字在 case 语句中已知，则用户是朋友。如果你不知道这个名字，可以添加一个默认响应。将结果输出到控制台。

## 石头、布、剪刀游戏

这是一个玩家和电脑之间的游戏，两者将随机选择石头、布或剪刀（或者你也可以创建一个使用真实玩家输入的版本！）。石头会打败剪刀，布会打败石头，剪刀会打败布。你可以使用 JavaScript 创建你自己的游戏版本，使用 if 语句应用逻辑。由于这个项目有点难，这里有一些建议的步骤：

1.  创建一个包含变量“石头”、“布”和“剪刀”的数组。

1.  设置一个变量，生成一个 0-2 之间的随机数用于玩家，然后为电脑的选择也做同样的事情。这个数字代表数组中三个项目的索引值。

1.  创建一个用于保存响应消息的变量。这可以显示玩家的随机结果，然后也是从数组中匹配项目的电脑结果。

1.  创建一个条件来处理玩家和电脑的选择。如果两者相同，则结果是平局。

1.  使用条件来应用游戏逻辑并返回正确的结果。有几种方法可以使用条件语句来完成，但你可以检查哪个玩家的索引值更大，并据此分配胜利，除了石头打剪刀的情况。

1.  添加一个新的输出消息，显示玩家选择与电脑选择以及游戏结果。

# 自我检查测验

1.  在这种情况下，控制台将输出什么？

    ```js
    const q = '1';
    switch (q) {
        case '1':
            answer = "one";
            break;
        case 1:
            answer = 1;
            break;
        case 2:
            answer = "this is the one";
            break;
        default:
            answer = "not working";
    }
    console.log(answer); 
    ```

1.  在这种情况下，控制台将输出什么？

    ```js
    const q = 1;

    switch (q) {
        case '1':
            answer = "one";
        case 1:
            answer = 1;
        case 2:
            answer = "this is the one";
            break;
        default:
            answer = "not working";
    }
    console.log(answer); 
    ```

1.  在这种情况下，控制台将输出什么？

    ```js
    let login = false;
    let outputHolder = "";
    let userOkay = login ? outputHolder = "logout" : outputHolder = "login";
    console.log(userOkay); 
    ```

1.  在这种情况下，控制台将输出什么？

    ```js
    const userNames = ["Mike", "John", "Larry"];
    const userInput = "John";
    let htmlOutput = "";
    if (userNames.indexOf(userInput) > -1) {
        htmlOutput = "Welcome, that is a user";
    } else {
        htmlOutput = "Denied, not a user ";
    }
    console.log(htmlOutput + ": " + userInput); 
    ```

1.  在这种情况下，控制台将输出什么？

    ```js
    let myTime = 9;
    let output;
    if (myTime >= 8 && myTime < 12) {
        output = "Wake up, it's morning";
    } else if (myTime >= 12 && myTime < 13) {
        output = "Go to lunch";
    } else if (myTime >= 13 && myTime <= 16) {
        output = "Go to work";
    } else if (myTime > 16 && myTime < 20) {
        output = "Dinner time";
    } else if (myTime >= 22) {
        output = "Time to go to sleep";
    } else {
        output = "You should be sleeping";
    }
    console.log(output); 
    ```

1.  在这种情况下，控制台将输出什么？

    ```js
    let a = 5;
    let b = 10;
    let c = 20;
    let d = 30;
    console.log(a > b || b > a);
    console.log(a > b && b > a);
    console.log(d > b || b > a);
    console.log(d > b && b > a); 
    ```

1.  在这种情况下，控制台将输出什么？

    ```js
    let val = 100;
    let message = (val > 100) ? `${val} was greater than 100` : `${val} was LESS or Equal to 100`;
    console.log(message);
    let check = (val % 2) ? `Odd` : `Even`;
    check = `${val} is ${check}`;
    console.log(check); 
    ```

# 摘要

现在，让我们来总结一下。在本章中，我们学习了条件语句。我们从 if else 语句开始。每当与 if 关联的条件为真时，if 块就会被执行。如果条件为假且存在 else 块，那么 else 块将被执行。我们还看到了三元运算符以及它们带来的奇特语法。如果你只需要在每个块中写一个语句，这是一种写 if-else 语句的简短方式。

最后，我们看到了 switch 语句以及它们如何被用来优化我们的条件代码。使用 switch 语句，我们可以将一个条件与许多不同的案例进行比较。当它们相等（值和类型）时，与 case 关联的代码将被执行。

在下一章中，我们将向其中添加循环！这将帮助我们编写更高效的代码和算法。
