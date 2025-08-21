# 第六章：数据验证

在本章中，我们将涵盖以下配方：

+   通过长度验证文本

+   通过范围验证数字

+   使用内置模式验证

+   内置约束和自定义验证的高级用法

+   计算密码强度

+   验证美国邮政编码

+   使用异步服务器端验证

+   结合客户端和服务器端验证

# 介绍

表单通常希望用户以某种方式行为，并按要求插入数据。这就是数据验证的作用。服务器端验证始终是必须要做的，应该考虑在客户端进行表单验证。

验证使应用程序用户友好，节省时间和带宽。客户端和服务器端验证相辅相成，应始终使用。在本章中，我们将介绍一些主要用于 HTML5 客户端检查的新机制，以及如何解决一些常见问题。

# 通过长度验证文本

在客户端进行的最基本检查之一是插入或提交表单的文本长度。这经常被忽略，但这是必须要做的检查之一，不仅仅是在客户端。想象一下，如果我们的输入没有任何限制，一些大文本可能会轻松地使服务器超载。

## 如何做...

让我们创建一个简单的 HTML 表单，其中包含一些我们将应用一些约束的不同输入：

1.  页面头部是标准的，所以我们将直接进入创建表单，首先添加`name`输入，限制为`20`个字符，如下所示：

```html
<form>
    <div>
      <label>
        Name <input id="name" type="text" name="name" maxlength="20" title="Text is limited to 20 chars"placeholder="Firstname Lastname" >
      </label>
    </div>
```

1.  在那之后，我们将添加另一个`input`字段，最初具有无效值，长于指定的测试目的，如下所示：

```html
    <div>
      <label>
        Initially invalid <input value="Some way to long value" maxlength="4" name="testValue" title="You should not have more than 4 characters">
      </label>
    </div>
```

1.  此外，我们将添加`textarea`标签，其中将添加`spellcheck`属性，如下所示：

```html
    <div>
      <label>
        Comment <textarea spellcheck="true" name="comment" placeholder="Your comment here"> </textarea>
      </label>
    </div>
```

1.  在那之后，我们将添加两个按钮，一个用于提交表单，另一个用于启用 JavaScript 备用验证，如下所示：

```html
    <button type="submit">Save</button>
    <button id="enable" type="button">Enable JS validation</button>
```

1.  由于我们将使用 jQuery Validate 插件测试备用版本，因此我们将添加这两个库的依赖项，并包括我们自己的`formValidate.js`文件，稍后将对其进行定义：

```html
    <script src="img/jquery.min.js"></script>
    <script src="img/jquery.validate.min.js"></script>
    <script src="img/formValidate.js" ></script>
```

1.  我们还需要选择要提交的表单，并在单击启用备用按钮时使用插件添加基于 JavaScript 的验证：

```html
    $("#enable").click(function(){
      $("#userForm").validate({
        rules: {
          name : "required",
          comment: {
            required: true,
            minlength: 50
          }
        },
        messages: {
          name: "Please enter your name",
          comment: {
            required: "Please enter a comment",
            minlength: "Your comment must be at least 50 chars long"
          }
        }
      });
    });
```

### 注意

请注意，我们还添加了将显示在验证错误上的消息。

启用 JavaScript 的按钮仅用于演示目的，在实际应用中，您可能会将其作为备用或作为唯一方法。由于我们只检查最大长度，除非我们先前使用不正确的值呈现了 HTML，否则验证不应该是一个问题。至于验证消息，在撰写本文时，所有现代浏览器和 IE 10 都支持，但尚未有移动浏览器添加支持。我们可以首先检查浏览器是否支持拼写检查属性，然后相应地采取行动：

```html
if ('spellcheck' in document.createElement('textarea')){
    // spellcheck is supported
} else {
   //spellchek is not supported
}
```

## 它是如何工作的...

最初，我们将查看`maxlength`属性。正如您所期望的那样，浏览器不允许用户输入违反此类型的约束，它们通常在插入最大值后停止输入。

因此，问题是如何违反此约束？

嗯，如果渲染的 HTML 一开始就是无效的，或者如果数据是以编程方式更改的，那么表单将在没有验证的情况下提交。这实际上是指定的行为；有一个脏标志，指示输入是否来自用户。在我们的情况下，只要我们不在标记为**最初无效**的输入中更改任何内容，表单就会成功提交。另一方面，当用户更改该元素中的一些数据时，表单将因验证错误而失败，如下面的屏幕截图所示：

![它是如何工作的...](img/9282OT_06_01.jpg)

在 Chrome Version 28 开发版本上显示的验证弹出窗口

在验证错误弹出窗口中显示的旁边的错误消息将具有`title`属性的内容，这意味着除了标准提示之外，此属性还有另一个用途。这个消息框在不同的浏览器上看起来不同。

尽管在浏览器上启用语法和拼写检查的主要控制权在用户手中，但有一个名为`spellcheck`的属性，可以添加以提示浏览器进行拼写检查。在我们的例子中，注释将如下屏幕截图所示：

![它是如何工作的...](img/9282OT_06_02.jpg)

这个属性是可继承的，并且可以与`lang`属性结合使用。例如，如果我们有以下片段：

```html
<html lang="en">
<body spellcheck="true">
  <textarea></textarea>
  <div lang="fr">
    <textarea></textarea>
     <input type="text">
  </div>
</body>
</html>
```

在我们的例子中使用了多种不同的语言。由于页面设置了`lang="en"`属性，因此嵌套在其中的所有元素都将使用英语词典。而因为`div`元素具有`lang="fr"`属性，所有嵌套的元素（`textarea`和`input type = text`）将根据法语词典进行检查。

### 注意

有关拼写检查的更多信息可以在 WHATWG 页面上找到[`www.whatwg.org/specs/web-apps/current-work/multipage/editing.html#spelling-and-grammar-checking`](http://www.whatwg.org/specs/web-apps/current-work/multipage/editing.html#spelling-and-grammar-checking)。还有一件事要注意的是，过去`spellcheck`属性必须设置为`true`或`false`，但是最新更改后可以留空[`www.w3.org/TR/html-markup/global-attributes.html#common.attrs.spellcheck`](http://www.w3.org/TR/html-markup/global-attributes.html#common.attrs.spellcheck)。

为什么我们说用户有完全控制权呢？嗯，如果用户一直在浏览器中选择拼写检查或从未检查过，那么该选项将覆盖此标签的行为。此属性可以应用于文本输入相关元素，以及其内容已被编辑的元素。

备用或以不同的方式是使用 JavaScript 来验证文本长度。因为 HTML5 中没有`minlength`属性，所以没有标准的最小长度验证方式。因此，我们将使用 jQuery 验证插件。还有一种方法可以使用`pattern`属性和正则表达式来做到这一点，但我们将在本章的*使用内置模式验证*中详细讨论。

要启用验证，我们选择表单并通过指定验证规则来设置规则，其中键是表单参数名称，值是应用的验证检查：

```html
 $("#userForm").validate({
        rules: {
          name : "required",
          comment: {
            required: true,
            minlength: 50
          }
        },
```

之后，我们为每个单独的检查添加消息，其中键再次是表单参数名称，如下所示：

```html
        messages: {
          name: "Please enter your name",
          comment: {
            required: "Please enter a comment",
            minlength: "Your comment must be at least 50 chars long"
          }
```

验证规则还将包括添加到表单元素的原始属性。在我们的例子中，标记为**最初无效**的输入具有`maxlength`属性，这将作为 JavaScript 配置的其他规则的一部分添加。此外，JavaScript 中定义的规则可以移动为适当的表单元素的一部分。最后，JavaScript 版本的结果应该看起来像以下的屏幕截图：

![它是如何工作的...](img/9282OT_06_03.jpg)

## 还有更多...

在我们的例子中，jQuery 验证插件显示的侧边文本的样式与标签相同。当存在验证错误时，会添加一个名为`.error`的简单 CSS 类，还有一个选项在发生验证问题时或被移除时执行函数。这可以在配置验证元素时完成，如下所示：

```html
       highlight: function(currentElement) {
          console.log("error on " +currentElement );
        }, unhighlight: function(currentElement) {
          console.log("no more error on " +currentElement );
        }
```

就样式验证消息和元素而言，它们将在本章后面讨论。

# 按范围验证数字

当涉及到表单中的数字时，基本验证是检查数字是否在给定范围内。为了实现这一点，应该将`min`和`max`属性应用于数字、范围和日期/时间相关的输入。

## 操作步骤...

我们将创建一个包含几个输入元素的表单，这些元素需要进行范围限制，如下所示：

1.  首先，我们将通过创建年龄为`18`的数字`input`字段来开始，如下面的代码片段所示：

```html
<form>
  <div>
    <label>
      Age <input id="age" type="number" name="name" min="18" max="140" />
    </label>
  </div>
```

1.  我们将添加`range`输入，用于用户将下注的**Bet**值，如下所示：

```html
  <div>
    <label>
      Bet <input id="deposit" value="1000" type="range" name="deposit" min="0" max="2000" />
       <output id="depositDisplay">1000</output>
    </label>
  </div>
```

1.  我们还包括以下限制为`min`、`max`和`step`的输入：

```html
  <div>
    <label>
      Doubles <input value="4" type="number" name="doubles" min="0" step="5" max="10" title="The value should be multiple of 5"/>
    </label>
  </div>
<div>
    <label>
      Awesomeness <input id="awesomeness" value="11" type="range" name="awesomeness" min="0" step="3" max="50" />
      <output id="awesomenessDisplay">10</output>
    </label>
  </div>
```

1.  之后，我们将添加 jQuery 的依赖项，我们的`example.js`，以及一个输入`submit`，如下所示：

```html
<input type="submit" />
</form>
  <script src="img/jquery.min.js"></script>
  <script src="img/example.js"> </script>
```

1.  我们还将在`example.js`脚本中简单地将范围输入与输出字段链接起来，以便进行简单的显示，如下所示：

```html
$(function() {
  $('#deposit').change(function() {
    $('#depositDisplay').html(this.value);
  });

  $('#awesomeness').change(function() {
    $('#awesomenessDisplay').html(this.value);
  });
 });
```

## 工作原理...

正如您所期望的那样，用户期望年龄在`18`到`140`之间，如果输入不在该范围内，我们将得到一个下溢约束违规，显示适当的（**值必须大于或等于{min}**）消息。同样，我们得到一个溢出约束违规，显示消息，**值必须小于或等于{max}**。

对于输入类型`range`，用户无法超出范围，甚至无法触发步骤不匹配的验证错误。步骤不匹配错误应该只在初始值不在`min`值内以及`step`属性的倍数时触发：

```html
<input value="11" type="range" name="awesomeness" min="0" step="3" max="50" />
```

这里`11`不应该是有效的，因为`step`属性的值是`3`，并且没有办法使用滑块到达`11`，但是初始值就是这样，所以我们应该得到一个验证错误，但这取决于浏览器。大多数当前版本的浏览器在渲染时只是纠正了最初选择的值。

如果我们尝试提交**双打**输入的表单，我们应该会收到一个验证消息，如下面的截图所示：

![工作原理...](img/9282OT_06_06.jpg)

在这里，我们收到消息，因为值是`4`，但约束是`min="0" step="5" max="10"`，这意味着输入的值必须是`5`的倍数。

用户无法使用输入类型`range`获得验证消息，但是使用输入类型`number`可以，因为用户可以在此手动插入数据。

# 使用内置的模式验证

为了创建更复杂的验证，我们需要使用 JavaScript。为了简化开发，引入了`input`字段的`pattern`属性。这使我们能够使用正则表达式进行验证检查，在本教程中，我们将看一些可以在其中使用的元素。

## 操作步骤...

在这个例子中，我们将使用简单的 HTML 创建一个表单，如下所示：

1.  首先，我们将直接在`body`部分添加表单，从**用户名**字段开始：

```html
  <div>
    <label>
      Username: <input type="text" title="only letters allowed" name="username" pattern="^[a-zA-Z]+$" />
    </label>
  </div>
```

1.  然后，我们将添加**电话**，如下所示：

```html
  <div>
    <label>
      Phone <input type="tel" name="phone" pattern="[\+]?[1-9]+" />
    </label>
  </div>
```

1.  我们将包括**网页**的`url`，如下所示：

```html
  <div>
    <label>
      Webpage <input type="url" name="webpage" />
    </label>
  </div>
```

1.  我们将添加**电子邮件**和**Gmail**输入，如下所示：

```html
  <div>
    <label>
      Emails <input type="email" name="emails" multiple required />
    </label>
  </div>
  <div>
    <label>
    Gmail <input type="email" name="emails" pattern="[a-z]+@gmail.com" maxlength="14"/>
    </label>
  </div>
```

## 工作原理...

如果指定了`pattern`属性，则使用早期版本的 JavaScript 正则表达式。整个文本必须与给定的表达式匹配。对于我们的例子，我们使用了宽松的验证，例如对于输入类型`tel`，我们允许数字和可选的前导`+`，由模式`[\+]?[1-9]+`指定。

一些其他输入类型，如`URL`和`email`，使用它们内置的验证。所有邮件必顶匹配以下正则表达式：

```html
/^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/
```

现在这是非常宽容的，所以我们可以添加额外的验证，就像我们在标记为**Gmail**的输入中添加的那样。约束可以组合，或者如果某个属性接受多个条目，所有这些条目都将根据约束进行验证，就像我们在以下电子邮件示例中所做的那样：

![工作原理...](img/9282OT_06_07.jpg)

还要记住，我们需要使用 title 或 placeholder 或其他方式添加提示，因为用户将默认收到**请匹配请求的格式**消息，并且不知道自己做错了什么。

## 还有更多...

有一个名为[`html5pattern.com/`](http://html5pattern.com/)的网站，旨在作为常用输入模式的来源。这绝对是一个很好的资源，我们鼓励您访问它。

# 内置约束和自定义验证的高级用法

到目前为止，我们已经使用了一些内置的验证机制。现在我们将更深入地研究其中一些，并了解如何添加自定义内容。当我们创建一个具有大多数这些功能的表单时，我们还将更改样式并应用一些更高级的检查，以及看到如何在某些元素上禁用验证。

### 注意

表单验证的当前工作草案版本可以在[`www.whatwg.org/specs/web-apps/current-work/multipage/forms.html#client-side-form-validation`](http://www.whatwg.org/specs/web-apps/current-work/multipage/forms.html#client-side-form-validation)找到。

## 如何做...

我们将创建一个表单，其中将使用 CSS 样式的错误消息，并使用 HTML 和 JavaScript 进行自定义验证，如下所示：

1.  我们将首先创建头部部分，在其中包括`example.css`，其中 CSS 文件将包含具有有效、无效、可选和必需状态的`input`元素的选择器：

```html
  <head>
    <title>Built In Validation</title>
    <link rel="stylesheet" href="example.css">
  </head>
```

1.  下一步是创建`example.css`文件。`valid.png`图像可以在源示例中找到。在现实生活中，您可能不会使用所有这些状态来设计表单的外观，但我们在这里添加它是为了展示可以做什么：

```html
input:invalid {
    background-color: red;
    outline: 0;
}
input:valid {
    background-color: green;
    background: url(valid.png) no-repeat right;
    background-size: 20px 15px;
    outline: 0;
}
input:required{
  box-shadow: inset 0 0 0.6em black;
}
input:optional{
  box-shadow: inset 0 0 0.6em green;
}
```

### 注意

CSS `box-shadow`在旧版浏览器中并不完全支持，例如 IE 8。`box-shadow`的规范可以在[`www.w3.org/TR/css3-background/#box-shadow`](http://www.w3.org/TR/css3-background/#box-shadow)找到。

1.  在`head`部分之后，我们将开始在`body`部分中添加表单元素。首先，我们将添加`name`和`nickname`字段，使它们成为`required`，稍后我们将确保它们的值不相同：

```html
  <div>
    <label>
      Name <input required name="name" x-moz-errormessage="We need this."/>
    </label>
  </div>
  <div>
    <label>
    Nickname <input required name="nickname"/>
    </label>
  </div>
```

1.  我们还可以包括两个与日期/时间相关的输入，一个用于`week`，另一个用于`month`，我们将限制周数从 2013 年第二周到 2014 年第二周，并允许选择其他每个月：

```html
       <div>
      <label>
        Start week <input type="week" name="week" min="2013-W02" max="2014-W02" required />
      </label>
    </div>
    <div>
      <label>
        Best months <input value="2013-01" type="month" step="2" name="month" />
      </label>
    </div>
```

1.  此外，我们将添加三个按钮：一个用于提交表单，另一个用于使用 JavaScript 检查有效性，另一个用于在没有验证和约束检查的情况下提交：

```html
    <button type="submit">Save</button>
    <button type="submit" formnovalidate>Save but don't validate</button>
    <button type="button">Check Validity</button>
```

1.  在表单之外，我们将添加一个`div`元素来显示一些日志信息：

```html
  <div id="validLog"></div>
```

1.  至于 JavaScript，我们添加了 jQuery 的依赖项并包括`example.js`：

```html
    <script src="img/jquery.min.js"></script>
    <script src="img/example.js"></script>
```

1.  在`example.js`文件中，我们将为检查有效性按钮添加一个事件，在其中我们将打印每个表单元素的验证错误的`ValidityState`值到`validLog`：

```html
    $(function() {
      var attemptNumber = 1;
      $("button[type=button]").click(function(){
        var message = (attemptNumber++)+"#<br/>";
        var isValid = $('form')[0].checkValidity();
        if(isValid){
          message += "Form is valid";
        }else{
          $("input").each(function( index ) {
            var validityState = $(this)[0].validity;
            var errors = "";
            If(!validityState.valid){
              message += "Invalid field <b> " + $(this).attr("name")+"</b>: ";
              for(key in validityState){
                if(validityState[key]){
                errors += key+" ";
            }
          }
          message += "  " + errors + " <br />";
        }
        });
      }
      message += "<hr />";
      $("#validLog").prepend(message);
    });
```

1.  要添加自定义验证，我们将使用`.setCustomValidity()`方法，因此它将检查`name`和`nickname`的值是否相同，如果是，我们将添加验证错误，如果不是，我们将删除自定义检查：

```html
    $("input[name='nickname']").change(function(){
      if($(this).val() === $("input[name='name']").val()){
        $(this)[0].setCustomValidity("You must have an awesome nickname so nickname and name should not match");
      }else{
      $(this)[0].setCustomValidity("");

    });
    $("input[name='name']").change(function(){
      if($(this).val() === $("input[name='nickname']").val()){
      $(this)[0].setCustomValidity("Nickname and name should not match");
      }else{
      $(this)[0].setCustomValidity("");
      }
    });
```

## 它是如何工作的...

`required`属性标记了表单内的 HTML 元素，要求在提交表单之前必须有一个值。第一个没有值的字段将在提交时获得焦点，并向用户显示带有消息的提示：

![它是如何工作的...](img/9282OT_06_04.jpg)

在 Firefox 上，有几种自定义显示给用户的消息的方法；我们可以使用`x-moz-errormessage`属性。在我们的情况下，这是`x-moz-errormessage="We need this."`，但这只在那里起作用。在 Chrome 上，`title`属性还会在标准消息旁边额外显示，但原始消息仍然存在。更改消息的另一种方法是使用 JavaScript 设置值：

```html
 <input type="text" required="required" oninvalid="this.setCustomValidity('Please put some data here')">
```

至于样式化表单元素，有 CSS 伪类选择器`:required`和`:optional`。

### 注意

在 WebKit 中，有特定于浏览器的 CSS 选择器，可以用来设置提示框的样式，如下所示：

`::-webkit-validation-bubble {…}`

`::-webkit-validation-bubble-message {…}`

但是因为它们是特定于浏览器的，它们在实际使用中并不是非常有用。

`min`、`max`和`step`属性可以用于与日期相关的输入类型，而不仅仅是数字。日期类型的默认步长是一天，周类型是一周，依此类推。如果我们设置一个与默认步长不同的步长，例如在月份输入上，如果我们将步长设置为 2，用户将无法从日期选择器控件中选择每隔一个月，但仍然可以在文本中输入错误的日期，触发`stepMismatch`。

因为验证是在提交表单之前触发的，如果输入无效，`submit`事件将不会被调用。如果我们需要在不进行验证的情况下提交数据，可以使用`formnovalidate`属性，如下所示：

```html
<button type="submit" formnovalidate>Save but don't validate</button>
```

有时我们可能需要从 JavaScript 中访问元素的`validityState`值；为此，可以在表单和表单中的输入元素上执行`checkValidity()`方法。顾名思义，它检查元素的状态，当它在表单上调用时，所有子元素都会被检查验证，此外，我们还可以在每个单独的元素上调用该方法，例如`input`、`select`或`textarea`。在我们的情况下，对于表单，它是这样的：

```html
$('form')[0].checkValidity();
```

`$('form')[0]`元素给我们提供了所选 jQuery 对象的包装 DOM 元素，也可以通过在所选元素上调用`.get()`来完成。每个元素都有一个我们可以读取的`validitystate`值，如下所示：

```html
       $("input").each(function(index) {
          var validityState = $(this)[0].validity;
       …
```

在这一点上，我们可以访问`validityState`对象的几个内置检查，例如：`valueMissing`、`typeMismatch`、`patternMismatch`、`tooLong`、`rangeUnderflow`、`rangeOverflow`、`stepMismatch`、`badInput`和`customError`。如果存在这样的约束违规，每个都将返回`true`。在本示例代码中，我们只是将约束违规的名称打印到日志中。

如果我们有相互依赖的字段，或者需要实现一些自定义验证逻辑，会发生什么？没问题，我们可以在每个依赖字段上使用`setCustomValidity()`方法。在我们的情况下，我们希望`name`和`nickname`变量的输入不同。因此，我们添加了更改监听器，如果它们相同，我们只需使用`customValidity("your message here")`设置消息，当我们需要移除违规时，我们将消息设置为空字符串：

```html
    $("input[name='nickname']").change(function(){
      if($(this).val() === $("input[name='name']").val()){
      $(this)[0].setCustomValidity("You must have an awesome nickname so nickname and name should not be the same");
      }else{
      $(this)[0].setCustomValidity("");
      }
    });
```

此外，还有两个 CSS 伪选择器`:valid`和`:invalid`，我们将用它们来根据它们的`validityState`值来设置元素的样式。

### 注意

客户端表单验证的规范可以在以下网址找到：[`www.whatwg.org/specs/web-apps/current-work/multipage/forms.html#client-side-form-validation`](http://www.whatwg.org/specs/web-apps/current-work/multipage/forms.html#client-side-form-validation)。至于约束 API，更多信息可以在[`www.whatwg.org/specs/web-apps/current-work/#the-constraint-validation-api`](http://www.whatwg.org/specs/web-apps/current-work/#the-constraint-validation-api)找到。

需要注意的一点是，并非所有浏览器都完全支持所有功能。例如，IE 9 没有对任何约束或新输入类型的支持。有关当前浏览器支持的更多信息，请访问[`caniuse.com/#search=form%20vali`](http://caniuse.com/#search=form%20vali)和[`www.quirksmode.org/compatibility.html`](http://www.quirksmode.org/compatibility.html)。

## 还有更多...

如果我们想使用一些属性来禁用整个表单的验证，我们可以设置表单属性称为`novalidate`。例如，这将禁用检查，但允许使用输入类型范围的`min`和`max`。

还有另一种方法可以禁用标准浏览器提示框并创建自定义提示框：

```html
    $('form').each(function(){
      $(this)[0].addEventListener('invalid', function(e) {
        e.preventDefault();
        console.log("custom popup");
      },true);
    });
```

在使用内置约束之前，应该考虑几个问题：

+   我们需要知道用户何时点击了**提交**按钮吗？

+   我们是否需要为不支持表单验证 API 的浏览器提供客户端验证？

如果我们需要知道用户何时尝试提交表单，我们可以附加点击事件监听器，而不是提交。至于旧版浏览器，我们可以选择依赖必须存在的服务器端验证，但如果我们不想在客户端失去功能，可以通过添加 webshim 的方式来做到这一点，[`afarkas.github.com/webshim/demos/index.html`](http://afarkas.github.com/webshim/demos/index.html)。

![更多内容...](img/9282OT_06_07.jpg)

# 计算密码强度

许多网站在其注册表单上显示用户选择的密码强度。这种做法的目标是帮助用户选择一个更好、更强的密码，这样就不容易被猜测或暴力破解。

在这个示例中，我们将制作一个密码强度计算器。它将通过计算潜在攻击者必须在猜测密码之前进行的暴力破解尝试的数量来确定密码强度。它还会警告用户，如果他的密码在 500 个常用密码列表中。

## 做好准备

在开始之前，重要的是要看一下我们将如何计算攻击者必须进行的暴力破解尝试的数量。我们将看两个因素：密码的长度和用户使用的字符集的大小。

字符集的大小可以通过以下方式确定：

+   如果用户在密码中添加小写字母表字母，则字符集的大小将增加 26（字母表中的字母数）

+   如果用户在密码中的任何位置使用大写字母，将添加额外的 26 个字符

+   如果用户添加一个数字，则添加 10 个字符

+   如果用户添加特殊字符，如句号、逗号、括号、和等，则添加 24 个字符

+   如果用户使用其他表中找不到的 Unicode 字符，则添加 20 个字符

## 如何做...

让我们编写 HTML 和 JavaScript 代码：

1.  创建一个简单的 HTML 页面，其中包含一个`password`输入，然后添加一个`div`元素，我们将使用密码强度结果进行更新。常见密码将通过名为`common-passwords.js`的脚本包含：

```html
<!DOCTYPE HTML>
<html>
    <head>
        <title>Password strength calculator</title>
   </head>
   <body>
   <input type="password" id="pass" value="" />
   <div id="strength">0 (very poor)</div>
   <script src="img/jquery.min.js"></script>
   <script type="text/javascript" src="img/common-passwords.js"></script>
   <script type="text/javascript" src="img/example.js"></script>
   </body>
</html>
```

`common-passwords.js`脚本没有包含在这里，但可以在附加代码中找到。

1.  检查逻辑的代码在`example.js`中：

```html
$(function() {
    function isCommon(pass) {
      return ~window.commonPasswords.indexOf(pass);
    }

    function bruteMagnitude(pass) {
      var sets = [
      { regex: /\d/g, size: 10 },
      { regex: /[a-z]/g, size: 26 },
      { regex: /[A-Z]/g, size: 26 },
      { regex: /[!-/:-?\[-`{-}]/g, size: 24 },
      ];
      var passlen = pass.length,
      szSet = 0;

      sets.forEach(function(set) {
        if (set.regex.test(pass)) {
          szSet += set.size;
          pass = pass.replace(set.regex, '');
        }
        });
        // other (unicode) characters
        if (pass.length) szSet += 20;
        return passlen * Math.log(szSet) / Math.LN10;
    }

    var strengths = ['very poor', 'poor', 'passing', 'fair',
        'good', 'very good', 'excellent'];

    function strength(pass) {
        if (isCommon(pass) || !pass.length) return 0;
        var str = bruteMagnitude(pass);
        return str < 7  ? 0 // very poor
             : str < 9  ? 1 // poor      - 10 million - 1 billion
             : str < 11 ? 2 // passing   - 1 billion - 100 billion
             : str < 13 ? 3 // fair      - 100 billion - 10 trillion
             : str < 15 ? 4 // good      - 10 trillion - 1 quadrillion
             : str < 17 ? 5 // very good - 1-100 quadrillion
             : 6;           // excellent - over 100 quadrillion
    }
```

1.  在`password`字段中按下键时，更新指示密码强度的`div`元素：

```html
    $('#pass').on('keyup keypress', function() {
      var pstrength = strength($(this).val());
      $("#strength").text(pstrength + ' (' + strengths[pstrength] + ')');
    });
});
```

## 它是如何工作的...

计算分为两部分：检查密码的普遍性和密码的复杂性。

我们通过检查密码是否可以在`commonPasswords`数组中找到来检查密码是否常见，该数组由`common-password.js`提供。如果未找到条目，`Array#indexOf`返回 1。按位非运算符`~`将该值转换为零，这等于 false。所有大于或等于 0 的其他数字将具有负值，这是 true 值。因此，如果在数组中找到密码，则整个表达式返回 true。

在`bruteMagnitude`函数中，我们使用`passwordLength`和字符`setsize`方法计算密码的暴力破解数量级：

```html
magnitude = log10(setSize passwordLength) = passwordLength * log10(setSize)
```

这是暴力破解密码攻击者必须尝试猜测密码的密码数量级的近似值。

基于这些信息，我们现在可以给出实际的密码强度。如果密码属于前 500 个常见密码之一，它将被分类为弱密码。否则，它将根据其暴力破解强度按照以下表进行分类：

| 数量级 | 密码数量 | 评级 |
| --- | --- | --- |
| 少于 7 | 少于 1000 万 | 非常差 |
| 7 到 8 | 1000 万到 10 亿 | 差 |
| 9 到 10 | 10 亿到 1000 亿 | 通过 |
| 11 到 12 | 1000 亿到 10 万亿 | 公平 |
| 13 到 14 | 10 万亿到 1 千万亿 | 良好 |
| 15 到 17 | 1 到 100 万亿 | 非常好 |
| 大于 17 | 大于 100 万亿 | 优秀 |

分类，以及描述性文本将在每次按键时更新，并显示给用户在密码字段下方。

# 验证美国邮政编码

在网页上具有地址表单的情况下，在客户端验证邮政编码可能会很有用。

输入数字是一个容易出错的过程。如果我们能够提供一些基本的即时验证来告知用户可能在其数据输入中出现的错误，那对用户来说将会很好。

另一方面，一个令人满意的完整邮政编码数据库具有非平凡的大小。在客户端加载完整数据库可能会很困难且不太理想。

在这个示例中，我们将编写一个客户端邮政编码验证函数。在这个过程中，我们将学习如何将一个非平凡的邮政编码数据库转换为可以在客户端加载的较小表示。

## 准备工作

首先让我们下载邮政编码数据库文件。[unitedstateszipcode.org](http://unitedstateszipcode.org)网站提供了一个 CSV 格式的免费邮政编码数据库（[`www.unitedstateszipcodes.org/zip-code-database/`](http://www.unitedstateszipcodes.org/zip-code-database/)）。

我们将从该文件中提取一个较小的数据库，该数据库可以在客户端加载。为此，我们将编写一个 Node.js 脚本，请确保已安装 Node.js。从[`nodejs.org/`](http://nodejs.org/)下载 Node.js，在附录 A 中有解释，*安装 Node.js 和使用 npm*。

### 注意

`Node.js`是建立在 Chrome 的 V8 JavaScript 引擎之上的平台，用于编写快速的异步网络应用程序。它配备了一个名为`npm`的出色模块管理器，以及一个包含数以万计模块库的注册表。

## 如何做...

在与`zip_code_database.csv`相同的目录中，我们将创建一个新的 Node.js 脚本。为了处理 CSV 文件，我们将使用一个 CSV 解析库。

1.  从相同目录的命令提示符中，让我们通过运行以下命令来安装 node 模块 CSV：

```html
npm install csv

```

1.  然后我们将创建`createdb.js`，它将解析 CSV 文件并从中提取最少量的数据，即美国州和邮政编码：

```html
var csv = require('csv');
var zips = {};
csv().from.path('zip_code_database.csv').on('record', function(zc) {
    // column 0 is zipcode; column 5 is state
    // column 12 is country, 13 is decomissioned (0/1)
    // filter non-decmissioned US zipcodes
    if (zc[12] == 'US' && !parseInt(zc[13])) {
      zips[zc[5]] = zips[zc[5]] || [];
      zips[zc[5]].push(parseInt(zc[0].trim(), 10));
    }
}).on('end', function() {
```

1.  此时，我们有一个可用的邮政编码数组。但是，如果我们直接写出所有这些邮政编码，将会得到一个相当大的 400 KB 的 JSON 数组，在使用 GZIP 压缩后为 150 KB。许多有效的邮政编码数字是连续的。我们可以利用这一点，将它们表示为范围。通过应用这种技术，我们得到一个 115 KB 的文件，在压缩后为 45 KB。这个大小看起来更加可接受：

```html
    var zipCodeDB = [];
    function rangify(arr) {
      var ranges = [], first = 0, last = 0;
      for (var k = 0; k < arr.length; ++k) {
        var first = arr[k];
        while (arr[k] + 1 >= arr[k + 1] && k < arr.length - 1) ++k;
        var last = arr[k];
        ranges.push(first != last? [first, last]:[first]);
        first = last = 0;
      }
      return ranges;
    }
```

1.  最终表示将是一个按`state`名称排序的 JSON 数组。数组中的每个元素表示一个州，包含两个属性：州名和表示为数字的有效邮政编码列表，或表示为二维数组的邮政编码范围：

```html
    var list = [];
    for (var state in zips) if (state != 'undefined') {
        list.push({state: state, codes: rangify(zips[state])});
    }
    list = list.sort(function(s1, s2) {
        return s1.state < s2.state ? -1
             : s1.state > s2.state ?  1
             :0;
    });
    console.log('window.zipCodeDB =', JSON.stringify(list));
    }
```

1.  在相同目录中从命令行中运行此脚本`node createdb.js > zipcodedb.js`将生成`zipcodedb.js`文件，其中包含数据库。以下是数据库 JSON 的示例：

```html
window.zipCodeDB = [{
    "state": "AA",
    "codes": [34004, 34006, 34008, 34011, [34020, 34025],
        [34030, 34039], 3404, ...]
},
...
]
```

1.  现在我们可以使用这个数据库来创建基本的验证器，将其包含在我们的`index.html`页面中。页面将包含一个简单的州选择下拉菜单和一个邮政编码字段。在邮政编码字段下方将是验证消息：

```html
<!DOCTYPE HTML>
<html>
    <head>
        <title>Zip code validation</title>
   </head>
   <body>
   <p>State: <select id="state"></select></p>
   <p>Zipcode: <input type="text" id="zipcode" value="" /></p>
   <div id="validate">Invalid zipcode</div>
   <script src="img/jquery.min.js"></script>
   <script type="text/javascript" src="img/zipcodedb.js"></script>
   <script type="text/javascript" src="img/example.js"></script>
   </body>
</html>
```

1.  最后，我们将编写一个`lookup`函数来检查给定的邮政编码是否在我们的数据库中，我们将在用户输入时用它来验证用户输入。我们将使用相同的数据库填充州下拉菜单：

```html
$(function() {

    function lookup(zipcode) {
      function within(zipcode, ranges) {
        for (var k = 0; k < ranges.length; ++k)
        if (zipcode == ranges[k]
        || (ranges[k].length > 1
        && ranges[k][0] <= zipcode
        && zipcode <= ranges[k][1])) return k;
        return -1;
        }
        for (var k = 0; k < window.zipCodeDB.length; ++k) {
          var state = window.zipCodeDB[k],
          check = within(zipcode, state.codes);
          if (~check) return state.state;
        }
        return null;
    }

    window.zipCodeDB.forEach(function(state) {
      $('<option />').attr('value', state.state)
      .text(state.state).appendTo('#state');
    });

    $("#zipcode").on('keypress keyup change', function() {
      var state = lookup($(this).val());
      if (state == $("#state").val())
      $('#validate').text('Valid zipcode for ' + state);
      else $('#validate').text('Invalid zipcode');
    });
});
```

## 工作原理...

为了在客户端验证邮政编码，我们首先需要将数据库转换为更小的大小。

下载的数据库包含了许多额外的数据，例如城市到邮政编码的映射，邮政编码类型，时区和地理坐标，以及已废弃的邮政编码。我们删除了额外的数据，只留下仍在使用中的美国邮政编码及其所在州。

为了进一步减少数据库，我们将更长的有效邮政编码范围表示为包含范围内第一个和最后一个数字的数组。这有助于将数据库大小进一步减小到一个合理的大小，与中等网站图像的大小相比。

为了使用数据库，我们编写了一个简单的`lookup`函数，检查邮政编码是否在任何州的`zipcode`和`ranges`的值列表中，并在找到时返回州名。

在用户输入邮政编码时，验证信息会自动更新。

# 使用异步服务器端验证

许多验证检查只能在服务器端执行。以下是示例：

+   在验证用户注册表单时，我们需要检查输入的用户名是否可用

+   当用户输入邮政地址时，我们可能需要向外部服务询问地址是否正确

服务器端验证检查的问题在于它们需要是异步的。因此，它们不能被写成 JavaScript 函数，返回验证结果。

为了解决这个问题，在这个示例中，我们将制作一个使用续传风格的验证器。示例中有一个用户名输入字段，用于与服务器进行验证。服务器会检查用户名是否可用于注册或已被其他用户占用。

## 准备工作

我们将简要介绍续传风格。这是大多数 JavaScript 库用于异步操作的风格，例如服务器通信。例如，在 jQuery 中，我们不是写以下代码：

```html
data = $.getJSON('/api/call');
doThingsWith(data);
```

我们写如下：

```html
$.getJSON('/api/call', function(data) {
    doThingsWith(data);
});
```

我们可以将相同的转换应用于验证函数，如下所示：

```html
var errors = validate(input)
if (errors.length) display(errors);
```

这将变成：

```html
validate(input, function(errors) {
    if (errors.length) display(errors);
});
```

这意味着我们还需要更改`validate`函数。例如，如果我们有如下所示：

```html
function validate(input) {
    if (condition(input))
      return [{message: "Input does not satisfy condition"}];
    else return [];
}
```

将其转换为续传风格后，我们将会有：

```html
function validate(input, callback) {
    condition(input, function(result) {
      if (result) callback([{message: "Input does not satisfy condition"}]);
      else callback([]);
    });
}
```

这使我们能够在验证函数中使用服务器端调用，例如`$.getJSON`，如下所示：

```html
function validate(input, callback) {
    $.getJSON('/api/validate/condition', function(result)
    if (result) callback([{message: "Input does not satisfy condition"}]);
    else callback([]);
    });
}
```

现在我们可以从浏览器中使用我们的服务器端验证器。

## 如何做...

我们将编写包含要验证的表单和实现验证的 JavaScript 代码的 HTML 页面。

1.  让我们从 HTML 页面开始。它必须包含一个带有用户名输入和默认情况下隐藏的红色文本验证结果的表单：

```html
<!DOCTYPE HTML>
<html>
<head>
<title>Async validation</title>
<style type="text/css">
p[data-validation-error] {
    display:none;
    color:red;
}
</style>
</head>
<body>
<form>
    <p>Username:</p>
    <p><input name="user" id="user" value="" /></p>
    <p data-validation-error="user"></p>
</form>
<script src="img/jquery.min.js"></script>
<script type="text/javascript" src="img/example.js"></script>
</body>
</html>
```

1.  验证代码将在`example.js`中 - 它包含一个模拟`async`服务器调用的函数，一个用于延迟执行`async`服务器调用以防止多次调用的函数，以及一个显示验证结果的函数：

```html
$(function() {
    function validate(name, callback) {
      // Simulate an async server call
      setTimeout(function() {
        callback(~['user', 'example'].indexOf(name) ?
        'Username is already in use' : null);
        },500);
    }
    function createDelayed(ms) {
      var t = null;
      return function(fn) {
        if (t) clearTimeout(t);
        t = setTimeout(fn, ms);
        };
    };
    var delayed = createDelayed(1500);

    var user = $('input[name="user"]'),
    form = user.parents('form');
    user.on('keyup keypress', function() {
      delayed(validate.bind(null, $(this).val(), function callback(err) {
        var validationError = form.find('p[data-validation-error="user"]');
        console.log(validationError);
        if (err) validationError.text(err).show();
        else validationError.hide();
      }));
    });
});
```

## 工作原理...

`example.js`中的`validate`函数中的代码通过使用`setTimeout`函数模拟了服务器调用。可以轻松地用真实的服务器验证 API 调用替换这段代码，类似于`jQuery.getJSON`来获取验证结果。

`createDelayed`函数创建一个`delayer`对象。`delayer`对象包装要延迟的函数。它不同于`setInterval`，因为如果在延迟到期之前再次调用`delayer`对象，先前的超时将被取消并重新启动。这有助于我们避免在每次按键时向服务器发送请求，而是在用户停止输入后`1500ms`发送请求。

我们在每次用户按键时调用`delayer`对象，将"`this`"绑定到`null`，将第一个参数绑定到输入字段的当前值，将`callback`函数绑定到一个函数，如果存在返回的验证错误，则显示它。

# 结合客户端和服务器端验证

在处理真实的 Web 表单时，我们通常需要对多个字段进行各种验证。一些字段可能只需要在客户端执行的检查，而有些可能还需要服务器端验证。

在这个示例中，我们将设计和实现自己的验证插件，支持异步验证。它将类似于 jQuery Validate 的工作。我们还将实现一些基本的验证方法，如`required`、`minLength`和`remote`。

我们将在一个简单的用户注册表单上使用这些方法，直到用户在所有字段中输入有效数据为止，该表单将被阻止提交。

## 准备工作

我们设计过程的第一步是设计将在验证器中使用的数据结构。我们将创建一个类似于 jQuery Validate 的 API，它以配置对象作为参数。但是，我们将选择更现代的 HTML5 方法，其中验证规则嵌入到 HTML 中，如下所示：

```html
  <form data-avalidate>
    <input name="field"
        data-v-ruleName="ruleParam" name="user" value="" />
    <span data-v-error="ruleName">{parameterized} rule error</span>
  </form>
```

为了支持这种规则和消息结构，Validate 将利用验证插件。

每个插件都将有一个唯一的名称。

该插件将是一个接受三个参数的函数：正在进行验证的元素、规则参数对象和在验证完成时调用的`callback`函数。`callback`函数将有两个参数：第一个参数将指示字段是否有效，第二个参数将包含消息参数。

该插件将阻止表单提交，除非验证所有字段的有效性。

## 如何做...

让我们编写 HTML 和 JavaScript 代码。

1.  `index.html`页面将包含嵌入其验证规则的表单。请注意，我们还可以混合使用标准的 HTML 表单验证，例如通过`required`属性，如下所示：

```html
<!DOCTYPE HTML>
<html>
<head>
<title>Async validation</title>
<style type="text/css">
[data-v-error] { display:none; color:red; }
label { width: 10em; display:inline-block; text-align: right; }
</style>
</head>
<body>
  <form data-avalidate>
    <p>
    <label for="user">Username:</label>
    <input name="user"
      required
      data-v-minlen="6"
      data-v-server="/api/validate/unique"
      value="" />
    <span data-v-error="minlen">Must be at least {minlen} characters long</span>
    <span data-v-error="server">{username} is already in use</span>
    </p>

    <p>
    <label for="email">Email:</label>
    <input name="email" type="email"
      required
      data-v-minlen="6"
      data-v-server="/api/validate/email"
      value="" />
    <span data-v-error="server">{email} is already in use</span>
    </p>

    <p><label for="pass">Password:</label>
    <input name="pass" type="password"
      required
      data-v-minlen="8"
      data-v-strength="3"
      value="" />
    <span data-v-error="minlen">Must be at least {minlen} characters long</span>
    <span data-v-error="strength">Strength is {strength}</span>
    </p>

    <p><label for="pass2">Password (again):</label>
    <input name="pass2" type="password"
      required
      data-v-equals="pass"
      value="" />
    <span data-v-error="equals">Must be equal to the other password</span>
    </p>

    <input type="submit">

</form>
<script src="img/jquery.min.js"></script>
<script type="text/javascript" src="img/avalidate.js"></script>
<script type="text/javascript" src="img/avalidate-plugins.js"></script>
</body>
</html>
```

### 注意

这个 HTML 文件的有趣之处在于除了`avalidate.js`和`avalidate-plugins.js`之外，没有包含其他脚本，但它们为这个表单提供了完整的验证。

1.  让我们看看需要添加到`avalidate.js`的代码：

```html
;(function($) {
```

为了正确执行异步验证，我们需要能够延迟请求，直到用户停止输入。为此，我们使用`createDelayed` - 它创建超时，在每次调用时重置自身：

```html
    function createDelayed(ms) {
      var t = null;
      return function(fn) {
        if (t) clearTimeout(t);
        t = setTimeout(fn, ms);
      };
    }
```

`showError`在表单旁边显示适当的错误，并用模板文本填充它。第一次运行时，它将模板移出`error`元素的内部文本，并添加到一个新的属性中：

```html
    function showError(error, strings) {
      var tmpl;
      if (!error.attr('data-v-template')) {
      tmpl = error.text().toString();
      error.attr('data-v-template', tmpl);
    } else tmpl = error.attr('data-v-template');
      for (var key in strings)
      tmpl = tmpl.replace('{'+key+'}', strings[key]);
      error.text(tmpl).show();
    }
```

`elementVerifier`在一个元素上执行。它查找由`data-v-pluginName`属性指定的所有验证器插件，从属性中读取插件选项，然后运行异步插件。

1.  当所有插件完成验证时，如果没有找到错误，它将标记元素为有效。否则，它会显示错误，就像它们出现的那样：

```html
    function elementVerifier() {
      var isValid = true, waiting = 0, field = this;
      $.each(this.attributes, function(i, attr) {
        if (!attr.name.match(/data-v-/)) return;
        var plugin = attr.name.toString().replace('data-v-',''),
        options = attr.value;

        ++waiting;
        $.avalidate[plugin].call(field, options, function (valid, strings) {
          var error = $(field).parent().find('[data-v-error="'+plugin+'"]');
          if (!valid) {
            showError(error, strings);
            isValid = false;
          }
          else error.hide();
          if (!--waiting && isValid)
          $(field).attr('data-valid', 1);
        });
      });
    }
```

1.  `setupFormVerifier`通过绑定到其字段中发生的所有更改、键盘和鼠标事件，启用了对特定表单的验证过程。它为每个元素创建一个单独的`delayer`变量，并使用该`delayer`运行`elementVerifier`对象。最后，它禁止表单提交，除非`elementVerifier`对象标记所有字段为有效：

```html
    function setupFormVerifier(form) {
      form.on('change keyup mouseup', 'input,textarea,select', function() {
        var $this = $(this)
        var delayer = $this.data('avalidate');
        if (!delayer) {
          delayer = createDelayed(800);
          $this.data('avalidate', delayer);
        }
        $this.attr('data-valid', 0);
        delayer(elementVerifier.bind(this));
        }).on('submit', function(e) {
            var all = $(this).find('input,textarea,select').filter('[type!="submit"]'),
          valid = all.filter('[data-valid="1"]');
          if (all.length != valid.length)
          e.preventDefault();
        });
    }
```

1.  以下是使一切无需手动干预的部分。我们监听文档`body`对象上到达的所有事件，如果一个事件到达一个应该启用验证但没有启用的表单，我们就会在其上运行`setupFormVerifier`（一次）：

```html
    $(function() {
      $('body').on('submit change keyup mouseup', 'form[data-avalidate]', function() {
        if (!$(this).attr('data-avalidate-enabled')) {
          setupFormVerifier($(this));
          $(this).attr('data-avalidate-enabled', 1)
        }
      });
    });
}(jQuery));
```

1.  插件更容易编写。这是`avalidate-plugins.js`。请注意，服务器插件是用`setTimeout`模拟的。在进行 AJAX 调用时，同样的原则也适用：

```html
;(function($) {

    $.avalidate = {};
    $.avalidate.equals = function(name, callback) {
      var other = $(this).parents('form').find('[name="'+name+'"]').val();
      callback($(this).val() === other, {});
    };
    $.avalidate.minlen = function(len, callback) {
       callback($(this).val().length >= len || $(this).text().length >= len, {minlen: len});
    };
    $.avalidate.server = function(param, cb) {
      setTimeout(function() {
        var val = $(this).val();
        if (~param.indexOf('mail'))
        cb('test@test.com' != val, {email: val });
        else
        cb('username' != val, { username: val });
      }.bind(this), 500);
    };
    $.avalidate.strength = function(minimum, cb) {
        cb($(this).val().length > minimum, {strength: 'Low'});
    };

}(jQuery));
```

## 它是如何工作的...

这个验证器利用了新的 HTML5 数据属性功能。HTML5 通过添加输入元素属性和类型确实包含了一些很棒的新验证选项，但这还不够。为了解决这个问题，我们遵循 HTML5 模型，并为验证方法和验证错误消息添加了自己的数据属性。

为了使这些新的数据属性起作用，我们需要加载 JavaScript 代码。JavaScript 初始化元素的一个缺点是，每当我们在页面上添加新元素时，都需要调用初始化函数。这个插件成功地避免了这个缺点，它使用了新的 jQuery 绑定 API。与直接绑定到表单不同，监听器附加到了文档`body`对象上。因此，它可以与所有表单元素一起工作，包括新添加的元素。

灵活的插件使验证器能够轻松扩展，而无需修改核心。添加新的验证规则就像添加一个新函数一样简单。

最后，我们的错误消息可以包含由验证器提供的可选消息字符串填充的用户友好模板。

### 注意

您可能已经注意到，JavaScript 文件以分号字符（`;`）开头。这使它们更安全，可以进行连接和缩小。如果我们在另一个用括号括起来的脚本之前添加一个以值结尾的脚本（在没有分号的情况下将整个脚本内容视为函数调用），那么该值将被视为该函数调用的参数。为了避免这种情况，我们在括号之前添加一个分号，终止可能缺少分号的任何先前语句。
