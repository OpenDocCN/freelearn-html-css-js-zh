# 附录 B. 内置函数

本附录包含了内置函数（全局对象的方法）的列表，讨论在第三章中，*函数*：

| **函数** | **描述** |
| --- | --- |

| `parseInt()` | 接受两个参数：一个输入对象和基数；然后尝试返回输入的整数表示。不处理输入中的指数。默认基数为`10`（十进制数）。失败时返回`NaN`。省略基数可能会导致意外结果（例如对于`08`这样的输入），因此最好总是指定它：

```js
    > parseInt('10e+3');   
    10   
    > parseInt('FF');   
    NaN   
    > parseInt('FF', 16);   
    255   

```

|

`parseFloat()` | 接受一个参数并尝试返回其浮点数表示。理解输入中的指数：

```js
    > parseFloat('10e+3');   
    10000   
    > parseFloat('123.456test');   
    123.456   

```

|

| `isNaN()` | 缩写自“不是一个数字”。接受一个参数，如果参数不是有效的数字则返回`true`，否则返回`false`。首先尝试将输入转换为数字：

```js
    > isNaN(NaN);   
    true   
    > isNaN(123);   
    false   
    > isNaN(parseInt('FF'));   
    true   
    > isNaN(parseInt('FF', 16));   
    false   

```

|

| `isFinite()` | 如果输入是一个数字（或可以转换为数字），则返回`true`，但如果不是数字，则返回`Infinity`或`-` `Infinity`。对于无穷大或非数值的情况返回`false`：

```js
    > isFinite(1e+1000);   
    false   
    > isFinite(-Infinity);   
    false   
    > isFinite("123");   
    true   

```

|

| `encodeURIComponent()` | 将输入转换为 URL 编码的字符串。有关 URL 编码工作原理的更多详细信息，请参阅维基百科文章[`en.wikipedia.org/wiki/Url_encode`](http://en.wikipedia.org/wiki/Url_encode)：

```js
    > encodeURIComponent   
    ('http://phpied.com/');   
    "http%3A%2F%2Fphpied.com%2F"   
    > encodeURIComponent   
    ('some script?key=v@lue');   
    "some%20script%3Fkey%3Dv%40lue"   

```

|

| `decodeURIComponent()` | 接受一个 URL 编码的字符串并解码它：

```js
    > decodeURIComponent('%20%40%20');   
    " @ "   

```

|

| `encodeURI()` | 对输入进行 URL 编码，但假定给定了完整的 URL，因此通过不对协议（例如`http://`）和主机名（例如`www.phpied.com`）进行编码来返回有效的 URL：

```js
    > encodeURI('http://phpied.com/');   
    "http://phpied.com/"   
    > encodeURI('some   script?key=v@lue');   
    "some%20script?key=v@lue"   

```

|

| `decodeURI()` | `encodeURI()` 的相反操作：

```js
    > decodeURI("some%20script?key=v@lue");   
    "some script?key=v@lue"   

```

|

| `eval()` | 接受一个 JavaScript 代码的字符串并执行它。返回输入字符串中最后一个表达式的结果。尽量避免使用：

```js
    > eval('1 + 2');   
    3   
    > eval('parseInt("123")');   
    123   
    > eval('new Array(1, 2, 3)');   
    [1, 2, 3]   
    > eval('new Array(1, 2, 3); 1 +   2;');   
    3   

```

|
