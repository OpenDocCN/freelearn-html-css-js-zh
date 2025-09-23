# 附录 A. JavaScript 快速参考表

这是一个常用 JavaScript 方法和属性的迷你参考表。对于完整列表和文档，请参阅可在 [`developer.mozilla.org/en/docs/JavaScript`](http://developer.mozilla.org/en/docs/JavaScript) 找到的 Mozilla 开发者网络网站。

# getElementById() 方法

`getElementById()` 方法通过其 ID 返回元素的引用：

```js
var pic =document.getElementById("profilePic");
```

# getElementsByTagName() 方法

`getElementsByTagName()` 方法返回具有给定标签名的元素集合。搜索包括根节点在内的完整文档：

```js
var allImages =document.getElementsByTagName("img");
```

# getElementsByName() 方法

`getElementsByName()` 方法通过其名称返回元素的引用：

```js
var names =document.getElementsByName("name");
```

# alert 方法

`alert()` 方法显示一个包含可选指定内容和 OK 按钮的警告对话框：

```js
alert("This is a sample alert");
```

# toString() 方法

`toString()` 方法返回表示函数源代码的字符串：

```js
var countries = ["U.S.A", "U.K", "India", "France"];
alert(countries.toString());
```

# parseInt() 方法

`parseInt()` 方法接受一个字符串并返回其数值。它可以用来将字符串类型转换为数值类型。对于浮点值，可以使用 `parseFloat()`：

```js
var stringOne = "1";
var intOne = parseInt(stringOne);
```

# getDate() 方法

`getDate()` 方法根据本地时间返回指定日期的月份中的天数：

```js
var day = new Date();
alert(day.getDate())
```

# onclick 事件

当在单个元素上按下并释放指向设备按钮（通常是鼠标按钮）时，会触发 `onclick` 事件：

```js
<button onclick="callFunction()">Click</button>

<script>
function callFunction() {
   alert("Button Clicked");
}
</script>
```

# ondblclick 事件

当在单个元素上双击指向设备按钮（通常是鼠标按钮）时，会触发 `ondblclick` 事件：

```js
<button ondblclick="callFunction()">Click</button>
<script>
function callFunction() {
   alert("Button Double Clicked");
}
</script>
```

# window.location 对象

`window.location` 对象返回一个包含有关当前文档位置信息的 `Location` 对象。也可以不使用 `window` 前缀。

`window.location.href` 对象返回当前页面的 URL：

```js
alert(window.location.href);
```

# 使用 jQuery 的选择器

如果你在项目中使用 jQuery，你必须知道使用 jQuery 选择器是多么容易。它提供了方便的方法来选择元素，而不是使用多个 JavaScript 函数：

+   `$("p")`：这通过标签名选择元素

+   `$("#myID")`：这根据 `id` 属性选择元素

+   `$(".myClass")`：这根据 `class` 属性选择元素

更多关于高级 jQuery 选择器的详细信息，请访问 [`www.w3schools.com/jquery/jquery_selectors.asp`](http://www.w3schools.com/jquery/jquery_selectors.asp)。
