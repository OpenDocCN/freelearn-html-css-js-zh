# 第四章：实践中的正则表达式

在前两章中，我们深入讨论了正则表达式的语法，并且在这一点上，我们已经具备了构建真实项目所需的所有要素，这将是本章的目标。

了解正则表达式的语法允许您对文本模式进行建模，但有时提出一个好的可靠模式可能更困难，因此查看一些实际用例可以帮助您学习一些常见的设计模式。

因此，在本章中，我们将开发一个表单，并探讨以下主题：

+   验证姓名

+   验证电子邮件

+   验证 Twitter 用户名

+   验证密码

+   验证 URL

+   操作文本

# 正则表达式和表单验证

到目前为止，在前端上，正则表达式最常见的用途之一是与用户提交的表单一起使用，因此这就是我们将要构建的内容。我们将构建的表单将包含所有常见字段，如姓名、电子邮件、网站等，但除了所有验证之外，我们还将尝试一些文本处理。

在实际应用中，通常不会手动实现解析和验证代码。您可以创建一个正则表达式，并依赖于一些 JavaScript 库，例如：

+   **jQuery 验证**：参见[`jqueryvalidation.org/`](http://jqueryvalidation.org/)

+   **Parsely.js**：参见[`parsleyjs.org/`](http://parsleyjs.org/)

### 注意

即使是最流行的框架也支持使用正则表达式进行本地验证引擎，例如**AngularJS**（参见[`www.ng-newsletter.com/posts/validations.html`](http://www.ng-newsletter.com/posts/validations.html)）。

## 设置表单

这个演示将是一个允许用户创建在线简介的网站，因此包含不同类型的字段。然而，在我们进入这个之前（因为我们不会构建一个处理表单的后端），我们将设置一些 HTML 和 JavaScript 代码来捕获表单提交并提取/验证其中输入的数据。

为了保持代码整洁，我们将创建一个包含所有验证函数的数组，以及一个数据对象，其中将保存所有最终数据。

以下是我们开始添加字段的 HTML 代码的基本大纲：

```js
<!DOCTYPE HTML>
<html>
    <head>
        <title>Personal Bio Demo</title>
    </head>
    <body>
        <form id="main_form">
            <input type="submit" value="Process" />
        </form>

        <script>
            // js goes here
        </script>
    </body>
</html>
```

接下来，我们需要编写一些 JavaScript 来捕获表单并运行我们将要编写的函数列表。如果一个函数返回 false，这意味着验证未通过，我们将停止处理表单。如果我们通过整个函数列表并且没有出现问题，我们将在控制台和包含我们提取的所有字段的数据对象中注销：

```js
<script>
    var fns = [];
    var data = {};

    var form = document.getElementById("main_form");

    form.onsubmit = function(e) {
        e.preventDefault();

        data = {};

        for (var i = 0; i < fns.length; i++) {
            if (fns[i]() == false) {
                return;
            }
        }

        console.log("Verified Data: ", data);
    }
</script>
```

JavaScript 首先创建了我之前提到的两个变量，然后从 DOM 中提取表单对象并设置提交处理程序。`submit`处理程序首先通过阻止页面实际提交（因为在这个例子中我们没有任何后端代码），然后我们逐个运行函数列表中的函数。

# 验证字段

在本节中，我们将探讨如何手动验证不同类型的字段，如姓名、电子邮件、网站 URL 等。

## 匹配完整姓名

让我们先从一个简单的姓名字段开始。这是我们之前简要介绍过的内容，所以它应该给您一个关于我们的系统将如何工作的想法。以下代码放在脚本标签内，但是在我们迄今为止写的所有内容之后：

```js
function process_name() {
    var field = document.getElementById("name_field");
    var name = field.value;

    var name_pattern = /^(\S+) (\S*) ?\b(\S+)$/;

    if (name_pattern.test(name) === false) {
        alert("Name field is invalid");
        return false;
    }

    var res = name_pattern.exec(name);
    data.first_name = res[1];
    data.last_name = res[3];

    if (res[2].length > 0) {
        data.middle_name = res[2];
    }

    return true;
}

fns.push(process_name);
```

我们以类似于获取表单的方式获取姓名字段，然后提取值并根据模式匹配完整姓名。如果姓名不匹配模式，我们只是警告用户并返回`false`，以让表单处理程序知道验证失败。如果姓名字段格式正确，我们在数据对象上设置相应的字段（请记住，中间名在这里是可选的）。最后一行只是将此函数添加到函数数组中，因此在提交表单时将调用它。

使其正常工作所需的最后一件事是为此表单字段添加 HTML，因此在表单标签内（提交按钮右边），您可以添加此文本输入：

```js
Name: <input type="text" id="name_field" /><br />
```

在浏览器中打开此页面，您应该能够通过在**名称**框中输入不同的值来测试它。如果您输入一个有效的名称，您应该能够打印出具有正确参数的数据对象，否则您应该能够看到此警报消息：

![匹配完整名称](img/2258OS_04_01.jpg)

### 理解完整名称正则表达式

让我们回到用于匹配用户输入的名称的正则表达式：

```js
/^(\S+) (\S*) ?\b(\S+)$/
```

以下是对正则表达式的简要解释：

+   `^` 字符断言其位置在字符串的开头

+   第一个捕获组`(\S+)`

+   `\S+` 匹配一个非空格字符 [`^\r\n\t\f`]

+   `+` 量词一次或多次

+   第二个捕获组`(\S*)`

+   `\S*` 匹配任何非空白字符 [`^\r\n\t\f`]

+   `*` 量词零次或多次

+   `" ?"` 匹配空格字符

+   `?` 量词零次或一次

+   `\b` 断言其位置在（`^\w|\w$|\W\w|\w\W`）单词边界

+   第三个捕获组`(\S+)`

+   `\S+` 匹配一个非空白字符 [`^\r\n\t\f`]

+   `+` 量词一次或多次

+   `$` 断言其位置在字符串的末尾

## 使用正则表达式匹配电子邮件

我们可能想要添加的下一个字段类型是电子邮件字段。电子邮件乍看起来可能很简单，但实际上有各种各样的电子邮件。你可能只想创建一个`word@word.word`的模式，但第一部分除了字母之外还可以包含许多其他字符，域可以是子域，或者后缀可以有多个部分（比如`.co.uk`代表英国）。

我们的模式将简单地寻找一组不是空格或在第一部分中使用了`@`符号的字符。然后我们希望有一个`@`符号，后面跟着另一组至少有一个句点的字符，然后是后缀，后缀本身可能包含另一个后缀。因此，可以以以下方式完成：

```js
/[^\s@]+@[^\s@.]+\.[^\s@]+/
```

### 注意

我们的示例模式非常简单，不会匹配每个有效的电子邮件地址。有一个官方标准用于电子邮件地址的正则表达式，称为**RFC 5322**。有关更多信息，请阅读[`www.regular-expressions.info/email.html`](http://www.regular-expressions.info/email.html)。

因此，让我们将该字段添加到我们的页面上：

```js
Email: <input type="text" id="email_field" /><br />
```

然后我们可以添加这个函数来验证它：

```js
function process_email() {
    var field = document.getElementById("email_field");
    var email = field.value;

    var email_pattern = /^[^\s@]+@[^\s@.]+\.[^\s@]+$/;

    if (email_pattern.test(email) === false) {
        alert("Email is invalid");
        return false;
    }

    data.email = email;
    return true;
}

fns.push(process_email);
```

### 注意

有一个专门设计用于电子邮件的 HTML5 字段类型，但这里我们正在手动验证，因为这是一本正则表达式书。有关更多信息，请参阅[`www.w3.org/TR/html-markup/input.email.html`](http://www.w3.org/TR/html-markup/input.email.html)。

### 理解电子邮件正则表达式

让我们回到用于匹配用户输入的名称的正则表达式：

```js
/^[^\s@]+@[^\s@.]+\.[^\s@]+$/
```

以下是对正则表达式的简要解释：

+   `^` 断言其位置在字符串的开头

+   `[^\s@]+` 匹配不在以下列表中的单个字符：

+   `+` 量词一次或多次

+   `\s` 匹配任何空白字符 [`\r\n\t\f`]

+   `@` 匹配 `@` 字面字符

+   `[^\s@.]+` 匹配不在以下列表中的单个字符：

+   `+` 量词一次或多次

+   `\s` 匹配一个 [`\r\n\t\f`] 空白字符

+   `@.` 是 `@.` 列表中的单个字符

+   `\.` 字符匹配`.`字符

+   `[^\s@]+` 匹配不在以下列表中的单个字符：

+   `+` 量词一次或多次

+   `\s` 匹配 [`\r\n\t\f`] 空白字符

+   `@` 是 `@` 字面字符

+   `$` 断言其位置在字符串的末尾

## 匹配 Twitter 名称

我们要添加的下一个字段是 Twitter 用户名的字段。对于不熟悉的人来说，Twitter 用户名是以 `@username` 格式的，但当人们输入时，他们有时会包括前置的 `@` 符号，而在其他情况下，他们只会写用户名本身。显然，内部我们希望一切都以统一的方式存储，所以我们需要提取用户名，无论 `@` 符号是否存在，然后手动添加一个，所以无论它是否存在，最终结果看起来都是一样的。

因此，让我们再次为此添加一个字段：

```js
Twitter: <input type="text" id="twitter_field" /><br />
```

现在，让我们编写处理它的函数：

```js
function process_twitter() {
    var field = document.getElementById("twitter_field");
    var username = field.value;

    var twitter_pattern = /^@?(\w+)$/;

    if (twitter_pattern.test(username) === false) {
        alert("Twitter username is invalid");
        return false;
    }

    var res = twitter_pattern.exec(username);
    data.twitter = "@" + res[1];
    return true;
}

fns.push(process_twitter);
```

如果用户输入 `@` 符号，它将被忽略，因为在检查用户名后，我们将手动添加它。

### 理解 Twitter 用户名的正则表达式

让我们回到用于匹配用户输入的名称的正则表达式：

```js
/^@?(\w+)$/
```

这是正则表达式的简要解释：

+   `^` 断言它的位置在字符串的开头

+   `@?` 匹配 `@` 字符，字面上

+   `?` 量词表示出现零次或一次

+   第一个捕获组 `(\w+)`

+   `\w+` 匹配一个 [`a-zA-Z0-9_`] 的单词字符

+   `+` 量词表示出现一次或多次

+   `$` 断言它的位置在字符串的末尾

## 匹配密码

另一个常见的字段，可能有一些独特的约束条件，是密码字段。现在，并不是每个密码字段都有趣；你可能只允许几乎任何东西作为密码，只要字段不为空就可以。然而，有些网站需要至少包含一个大写字母、一个小写字母、一个数字和至少一个其他字符。考虑到这些可以组合的方式，创建一个可以验证这一点的模式可能会非常复杂。对于这个问题，一个更好的解决方案，也可以让我们在错误消息上更加详细，是创建四个单独的模式，并确保密码与每个模式匹配。

对于输入，它几乎是相同的：

```js
Password: <input type="password" id="password_field" /><br />
```

`process_password` 函数与前面的例子并没有太大的不同，我们可以看到它的代码如下：

```js
function process_password() {
    var field = document.getElementById("password_field");
    var password = field.value;

    var contains_lowercase = /[a-z]/;
    var contains_uppercase = /[A-Z]/;
    var contains_number = /[0-9]/;
    var contains_other = /[^a-zA-Z0-9]/;

    if (contains_lowercase.test(password) === false) {
        alert("Password must include a lowercase letter");
        return false;
    }

    if (contains_uppercase.test(password) === false) {
        alert("Password must include an uppercase letter");
        return false;
    }

    if (contains_number.test(password) === false) {
        alert("Password must include a number");
        return false;
    }

    if (contains_other.test(password) === false) {
        alert("Password must include a non-alphanumeric character");
        return false;
    }

    data.password = password;
    return true;
}

fns.push(process_password);
```

总的来说，你可能会说这是一个非常基本的验证，而且我们已经涵盖过了，但我认为这是一个很好的例子，展示了工作聪明而不是努力。当然，我们可能可以创建一个长模式，一次性检查所有内容，但那样会更不清晰，也更不灵活。因此，通过将其分解为更小、更易管理的验证，我们能够创建清晰的模式，并同时通过更有帮助的警报消息来提高其可用性。

## 匹配 URL

接下来，让我们为用户的网站创建一个字段；这个字段的 HTML 如下：

```js
Website: <input type="text" id="website_field" /><br />
```

一个 URL 可以有许多不同的协议，但在这个例子中，让我们将其限制为只有 http 或 https 链接。接下来，我们有一个带有可选子域的域名，我们需要以后缀结束。后缀本身可以是一个单词，比如 .com，也可以有多个段，比如 .co.uk。

总的来说，我们的模式看起来类似于这样：

```js
/^(?:https?:\/\/)?\w+(?:\.\w+)?(?:\.[A-Z]{2,3})+$/i
```

在这里，我们使用了多个非捕获组，用于可选的部分和重复一个段的情况。你可能还注意到，我们在正则表达式的末尾使用了不区分大小写的标志 (`/i`)，因为链接可以以小写或大写字母写入。

现在，我们将实现实际的函数：

```js
function process_website() {
    var field = document.getElementById("website_field");
    var website = field.value;

    var pattern = /^(?:https?:\/\/)?\w+(?:\.\w+)?(?:\.[A-Z]{2,3})+$/i

    if (pattern.test(website) === false) {
        alert("Website is invalid");
        return false;
    }

    data.website = website;
    return true;
}

fns.push(process_website);
```

到这一点，你应该对向我们的表单添加字段和添加函数来验证它们的过程非常熟悉了。因此，对于我们剩下的例子，让我们把重点从验证输入转移到操作数据上。

### 理解 URL 的正则表达式

让我们回到用于匹配用户输入的名称的正则表达式：

```js
/^(?:https?:\/\/)?\w+(?:\.\w+)?(?:\.[A-Z]{2,3})+$/i
```

这是正则表达式的简要解释：

+   `^` 断言它的位置在字符串的开头

+   `(?:https?:\/\/)?` 是一个非捕获组

+   `?` 量词表示出现零次或一次

+   `http` 字符串匹配 http 字符（不区分大小写）

+   `s?` 匹配 `s` 字符（不区分大小写）

+   `?` 量词匹配零次或一次

+   `:` 字符与 `:` 匹配

+   `\/` 字符与 `/` 匹配

+   `\/` 字符与 `/` 匹配

+   `\w+` 匹配一个 [`a-zA-Z0-9_]` 单词字符

+   `+` 量词匹配一次或多次

+   `(?:\.\w+)?` 是一个非捕获组

+   `?` 量词匹配零次或一次

+   `\.` 字符与 `.` 匹配

+   `\w+` 匹配一个 [`a-zA-Z0-9_]` 单词字符

+   `+` 量词匹配一次或多次

+   `(?:\.[A-Z]{2,3})+` 是一个非捕获组

+   `+` 量词匹配一次或多次

+   `\.` 字符与 `.` 匹配

+   `[A-Z]{2,3}` 匹配列表中的单个字符

+   `{2,3}` 量词匹配`2`到`3`次

+   `A-Z` 是在 A 和 Z 之间的单个字符（不区分大小写）

+   `$` 断言它的位置在字符串的末尾

+   `i` 修饰符：不区分大小写。不区分大小写的字母，意味着它将匹配 a-z 和 A-Z。

# 操作数据

我们将在我们的表单中再添加一个输入，用于用户的描述。在描述中，我们将解析一些东西，比如电子邮件，然后创建用户描述的纯文本和 HTML 版本。

这个表单的 HTML 非常简单；我们将使用一个标准的文本框，并给它一个适当的字段：

```js
Description: <br />
<textarea id="description_field"></textarea><br />
```

接下来，让我们从处理表单数据所需的基本结构开始：

```js
function process_description() {
    var field = document.getElementById("description_field");
    var description = field.value;

    data.text_description = description;

    // More Processing Here

    data.html_description = "<p>" + description + "</p>";

    return true;
}

fns.push(process_description);
```

这段代码从页面上的文本框中获取文本，然后保存它的纯文本版本和 HTML 版本。在这个阶段，HTML 版本只是简单地将纯文本版本包裹在一对段落标签之间，但这是我们现在要处理的内容。我想要做的第一件事是在段落之间分割，在文本区域中，用户可能有不同的分割方式——行和段落。对于我们的例子，假设用户只输入了一个换行符，那么我们将添加一个 `<br />` 标签，如果有多于一个字符，我们将使用 `<p>` 标签创建一个新的段落。

## 使用 String.replace 方法

我们将在字符串对象上使用 JavaScript 的 replace 方法。这个函数可以接受一个正则表达式模式作为它的第一个参数，以及一个函数作为它的第二个参数；每当它找到模式时，它将调用该函数，函数返回的任何东西都将被插入到匹配的文本的位置。

所以，对于我们的例子，我们将寻找换行符，并在函数中决定是否要用换行标签替换换行，或者根据它能够捕获到多少个换行符来创建一个实际的新段落：

```js
var line_pattern = /\n+/g;
description = description.replace(line_pattern, function(match) {
    if (match == "\n") {
        return "<br />";
    } else {
        return "</p><p>";
    }
});
```

你可能注意到的第一件事是，我们需要在模式中使用 `g` 标志，这样它将寻找所有可能的匹配，而不仅仅是第一个。除此之外，其余的都很简单。考虑这个表单：

![使用 String.replace 方法](img/2258OS_04_02.jpg)

如果你看一下前面代码的控制台输出，你应该会得到类似这样的东西：

![使用 String.replace 方法](img/2258OS_04_03.jpg)

## 匹配一个描述字段

接下来我们需要做的是尝试从文本中提取电子邮件，并自动将它们包装在链接标签中。我们已经介绍了一个正则表达式模式来捕获电子邮件，但我们需要稍微修改它，因为我们先前的模式期望电子邮件是文本中唯一存在的东西。在这种情况下，我们对包含在大段文本中的所有电子邮件感兴趣。

如果您只是在寻找一个单词，您可以使用 `\b` 匹配器，它匹配任何边界（可以是单词的结尾/句子的结尾），所以我们不再使用之前用来表示字符串结尾的美元符号，而是将边界字符放在那里，以表示单词的结尾。然而，在我们的情况下，这还不够好，因为有一些边界字符是有效的电子邮件字符，例如句号字符是有效的。为了解决这个问题，我们可以将边界字符与一个先行断言组合使用，并且说我们希望它以一个单词边界结束，但只有在后面跟着一个空格或句子/字符串的结尾时才会结束。这将确保我们不会截断子域或域的一部分，如果地址中间有一些无效的信息。

现在，我们不再创建一个会尝试解析电子邮件的东西，无论用户如何输入；创建验证器和模式的目的是强制用户输入一些逻辑的内容。也就是说，我们假设如果用户写了一个电子邮件地址，然后是一个句号，那么他/她并没有输入一个无效的地址，而是输入了一个地址，然后结束了一个句子（句号不是地址的一部分）。

在我们的代码中，我们假设在地址的末尾，用户要么会在后面加上一个空格，比如某种标点符号，要么会结束字符串/行。我们不再需要处理行，因为我们已经将它们转换为 HTML，但我们确实需要担心我们的模式在过程中不会捕获到 HTML 标签。

在这之后，我们的模式将类似于这样：

```js
/\b[^\s<>@]+@[^\s<>@.]+\.[^\s<>@]+\b(?=.?(?:\s|<|$))/g
```

我们从一个单词边界开始，然后寻找之前的模式。我将大于号 (`>`) 和小于号 (`<`) 字符都添加到不允许的字符组中，这样就不会捕获任何 HTML 标签。在模式的末尾，您可以看到我们希望在单词边界结束，但只有在后面跟着一个空格、一个 HTML 标签或字符串的结尾时才会结束。完成所有匹配的完整函数如下：

```js
function process_description() {
    var field = document.getElementById("description_field");
    var description = field.value;

    data.text_description = description;

    var line_pattern = /\n+/g;
    description = description.replace(line_pattern, function(match) {
        if (match == "\n") {
            return "<br />";
        } else {
            return "</p><p>";
        }
    });

    var email_pattern = /\b[^\s<>@]+@[^\s<>@.]+\.[^\s<>@]+\b(?=.?(?:\s|<|$))/g;
    description = description.replace(email_pattern, function(match){
        return "<a href='mailto:" + match + "'>" + match + "</a>";
    });

    data.html_description = "<p>" + description + "</p>";

    return true;
}
```

我们可以继续添加字段，但我认为重点已经被理解了。您有一个匹配您想要的模式，并且使用提取的数据，您能够提取和操作数据以满足您可能需要的任何格式。

### 理解描述正则表达式

让我们回到用于匹配用户输入的名称的正则表达式：

```js
/\b[^\s<>@]+@[^\s<>@.]+\.[^\s<>@]+\b(?=.?(?:\s|<|$))/g
```

这是正则表达式的简要解释：

+   `\b` 断言其位置在 (`^\w|\w$|\W\w|\w\W`) 单词边界处

+   `[^\s<>@]+` 匹配不在列表中的单个字符：

+   `+` 量词，出现一次或无限次

+   `\s` 匹配一个 [`\r\n\t\f` ] 空白字符

+   `<>@` 是 `<>@` 列表中的一个字符（区分大小写）

+   `@` 字符串匹配字符 `@`

+   `[^\s<>@.]+` 匹配不在此列表中的单个字符：

+   `+` 量词，出现一次或无限次

+   `\s` 匹配任何 [`\r\n\t\f`] 空白字符

+   `<>@.` 是 `<>@.` 列表中的一个字符（区分大小写）

+   `\.` 字符串匹配字符 `.` 

+   `[^\s<>@]+` 匹配不在此列表中的单个字符：

+   `+` 量词，出现一次或无限次

+   `\s` 匹配一个 [`\r\n\t\f` ] 空白字符

+   `<>@` 是 `<>@` 列表中的一个字符（区分大小写）

+   `\b` 断言其位置在 (`^\w|\w$|\W\w|\w\W`) 单词边界处

+   `(?=.?(?:\s|<|$))` 正向先行断言 - 断言正则表达式可以匹配下面的内容

+   `.?` 匹配任何字符（除了换行符）

+   `?` 量词，出现零次或一次

+   `(?:\s|<|$)` 是一个非捕获组：

+   第一种选择：`\s` 匹配任何空白字符 [`\r\n\t\f`]

+   第二种选择：`<` 字符串匹配字符 `<`

+   第三种选择：`$` 断言字符串的结束位置

+   `g` 修饰符：全局匹配。返回正则表达式的所有匹配项，而不仅仅是第一个

### 解释一个 Markdown 示例

更多正则表达式的示例可以在流行的**Markdown**语法中看到（参考[`en.wikipedia.org/wiki/Markdown`](http://en.wikipedia.org/wiki/Markdown)）。这是一个用户被迫以自定义格式编写东西的情况，尽管它仍然是一个格式，可以节省输入并且更容易理解。例如，在 Markdown 中创建一个链接，你会输入类似于这样的内容：

```js
[Click Me](http://gabrielmanricks.com)
```

然后会转换为：

```js
<a href="http://gabrielmanricks.com">Click Me</a>
```

忽略 URL 本身的任何验证，可以很容易地通过以下模式实现：

```js
/\[([^\]]*)\]\(([^(]*)\)/g
```

它看起来有点复杂，因为方括号和括号都是需要转义的特殊字符。基本上，我们想要一个开放的方括号，直到闭合的方括号，然后我们想要一个开放的括号，再次，直到闭合的括号。

### 提示

一个很好的网站来写 markdown 文档是[`dillinger.io/`](http://dillinger.io/)。

由于我们将每个部分都包装到自己的捕获组中，我们可以编写这个函数：

```js
text.replace(/\[([^\]]*)\]\(([^(]*)\)/g, function(match, text, link){
    return "<a href='" + link + "'>" + text + "</a>";
});
```

在我们的操作示例中，我们没有使用捕获组，但如果使用它们，那么回调的第一个参数将是整个匹配（类似于我们一直在使用的那些），然后所有单独的组将作为后续参数传递，按照它们在模式中出现的顺序。

# 总结

在本章中，我们涵盖了一些示例，向我们展示了如何验证用户输入以及如何操作它们。我们还看了一些常见的设计模式，并且看到有时候简化问题而不是在一个模式中使用蛮力来创建验证更好。

在下一章中，我们将继续通过使用**Node.js**开发一个应用程序来探索一些真实世界的问题，该应用程序可以用于读取文件并提取其信息，以更加用户友好的方式显示出来。
