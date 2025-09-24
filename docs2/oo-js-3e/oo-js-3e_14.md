# 附录 A. 保留字

本附录提供了两个保留关键字列表，这些关键字是在 ECMAScript 5（ES5）中定义的。第一个是当前单词列表，第二个是为未来实现保留的单词列表。

也有一个不再保留但曾经是 ES3 中保留字的单词列表。

你不能将保留字用作变量名：

```js
    var break = 1; // syntax error 

```

如果你将这些词用作对象属性，你必须引用它们：

```js
    var o = {break: 1};   // OK in many browsers, error in IE 
    var o = {"break": 1}; // Always OK 
    alert(o.break);       // error in IE 
    alert(o["break"]);    // OK 

```

# 关键字

当前在 ES5 中保留的单词列表如下：

+   `break`

+   `case`

+   `catch`

+   `continue`

+   `调试器`

+   `默认`

+   `删除`

+   `do`

+   `else`

+   `finally`

+   `for`

+   `函数`

+   `if`

+   `in`

+   `instanceof`

+   `new`

+   `return`

+   `switch`

+   `this`

+   `抛出异常`

+   `try`

+   `typeof`

+   `var`

+   `void`

+   `while`

+   `with`

# ES6 保留字

以下关键字在 ES6 中被保留：

+   `class`

+   `常量`

+   `枚举`

+   `导出`

+   `扩展`

+   `实现`

+   `导入`

+   `接口`

+   `let`

+   `包`

+   `私有`

+   `受保护的`

+   `public`

+   `static`

+   `super`

+   `yield`

## 未来保留字

这些关键字目前未使用，但为未来的版本保留：

+   `枚举`

+   `await`

# 以前保留的单词

从 ES5 开始，以下单词不再保留，但为了兼容旧浏览器，最好避免使用它们：

+   `抽象`

+   `布尔`

+   `字节`

+   `char`

+   `double`

+   `最终`

+   `float`

+   `goto`

+   `int`

+   `long`

+   `本地`

+   `short`

+   `同步的`

+   `throws`

+   `transient`

+   `易失的`
