# 附录 D. 正则表达式

当你使用正则表达式（在第四章 Chapter 4 中讨论），你可以匹配字面字符串，例如：

```js
    > "some text".match(/me/); 
    ["me"] 

```

然而，正则表达式的真正力量来自于匹配模式，而不是字面字符串。下表描述了您可以在模式中使用的不同语法，并提供了一些使用示例：

| **模式** | **描述** |
| --- | --- |

| `[abc]` | 匹配字符类：

```js
    > "some text".match(/[otx]/g);   
    ["o", "t", "x",   "t"]   

```

|

| `[a-z]` | 定义为范围的字符类。例如，[a-d] 等同于 [abcd]，[a-z] 匹配所有小写字母，[a-zA-Z0-9_] 匹配所有字符、数字和下划线字符：

```js
    > "Some Text".match(/[a-z]/g);   
    ["o", "m", "e",   "e", "x", "t"]   
    > "Some Text".match(/[a-zA-Z]/g);   
    ["S", "o", "m",   "e", "T", "e", "x", "t"]   

```

|

| `[^abc]` | 匹配除字符类之外的任何内容：

```js
    > "Some Text".match(/[^a-z]/g);   
    ["S", " ", "T"]   

```

|

| `a&#124;b` | 匹配 a 或 b。管道字符表示 OR，并且可以多次使用：

```js
    > "Some Text".match(/t&#124;T/g);
    ["T", "t"]
    > "Some Text".match(/t&#124;T&#124;Some/g);
    ["Some", "T",   "t"]

```

|

| `a(?=b)` | 仅当 a 后跟 b 时才匹配 a：

```js
    > "Some Text".match(/Some(?=Tex)/g);   
    null   
    > "Some Text".match(/Some(?=Tex)/g);   
    ["Some"]   

```

|

| `a(?!b)` | 仅当 a 后不跟 b 时才匹配 a：

```js
    > "Some Text".match(/Some(?!Tex)/g);   
    null   
    > "Some Text".match(/Some(?!Tex)/g);   
    ["Some"]   

```

|

| `\` | 转义字符，用于帮助您将模式中使用的特殊字符作为字面字符进行匹配：

```js
    > "R2-D2".match(/[2-3]/g);   
    ["2", "2"]   
    > "R2-D2".match(/[2\-3]/g);   
    ["2", "-", "2"]   

```

|

| `\n``\r``\f``\t``\v` | 换行符回车符换页符制表符垂直制表符 |
| --- | --- |

| `\s` | 空白字符，或前五个转义序列中的任何一个：

```js
    > "R2\n D2".match(/\s/g);   
    ["\n", " "]   

```

|

| `\S` | 上述内容的相反；匹配除了空白字符之外的所有内容。等同于 [^\s]：

```js
    > "R2\n D2".match(/\S/g);   
    ["R", "2", "D",   "2"]   

```

|

| `\w` | 任何字母、数字或下划线。等同于 [A-Za-z0-9_]：

```js
    > "S0m3 text!".match(/\w/g);   
    ["S", "0", "m",   "3", "t", "e", "x", "t"]   

```

|

| `\W` | `\w` 的相反：

```js
    > "S0m3 text!".match(/\W/g);   
    [" ", "!"]   

```

|

| `\d` | 匹配一个数字，等同于 [0-9]：

```js
    > "R2-D2 and C-3PO".match(/\d/g);   
    ["2", "2", "3"]   

```

|

| `\D` | `\d` 的相反；匹配非数字，等同于 [⁰-9] 或 [^\d]：

```js
    > "R2-D2 and C-3PO".match(/\D/g);   
    ["R", "-", "D",   " ", "a", "n", "d",
      " ", "C",   "-", "P", "O"]   

```

|

| `\b` | 匹配单词边界，例如空格或标点符号。匹配 R 或 D 后跟 2：

```js
    > "R2D2 and C-3PO".match(/[RD]2/g);   
    ["R2", "D2"]   

```

相同的，但仅在单词的结尾：

```js
    > "R2D2 and C-3PO".match(/[RD]2\b/g);   
    ["D2"]   

```

相同的模式，但输入包含一个连字符，它也是单词的结尾：

```js
    > "R2-D2 and C-3PO".match(/[RD]2\b/g);   
    ["R2", "D2"]   

```

|

| `\B` | `\b` 的相反：

```js
    > "R2-D2 and C-3PO".match(/[RD]2\B/g);   
    null   
    > "R2D2 and C-3PO".match(/[RD]2\B/g);   
    ["R2"]   

```

|

| `[\b]` | 匹配退格字符。 |
| --- | --- |
| `\0` | 空字符。 |

| `\u0000` | 匹配一个由四位十六进制数字表示的 Unicode 字符：

```js
    > "стоян".match(/\u0441\u0442\u043E/);   
    ["сто"]   

```

|

| `\x00` | 匹配由两位十六进制数字表示的字符代码：

```js
    > "\x64";   
     "d"   
    > "dude".match(/\x64/g);   
    ["d", "d"]   

```

|

| `^` | 要匹配的字符串的开始。如果您设置了 `m` 修改符（多行），它将匹配每行的开始：

```js
   > "regular\nregular\nexpression".match(/r/g);   
    ["r", "r", "r",   "r", "r"]   
    > "regular\nregular\nexpression".match(/^r/g);   
    ["r"]   
   > "regular\nregular\nexpression".match(/^r/mg);   
    ["r", "r"]   

```

|

| `$` | 匹配输入的结尾，或者当使用多行修改符时，匹配每行的结尾：

```js
    > "regular\nregular\nexpression".match(/r$/g);   
    null   
    > "regular\nregular\nexpression".match(/r$/mg);   
    ["r", "r"]   

```

|

| `.` | 匹配任何单个字符（除了换行符和回车符）：

```js
    > "regular".match(/r./g);   
    ["re"]   
    > "regular".match(/r.../g);   
    ["regu"]   

```

|

| `*` | 匹配前面的模式，如果它出现零次或多次。例如，`/.*/` 将匹配任何内容，包括空内容（空输入）：

```js
    > "".match(/.*/);   
    [""]   
    > "anything".match(/.*/);   
    ["anything"]   
    > "anything".match(/n.*h/);   
    ["nyth"]   

```

请记住，模式是“贪婪”的，这意味着它会尽可能多地匹配：

```js
    > "anything within".match(/n.*h/g);   
    ["nything with"]   

```

|

| `?` | 如果前面的模式出现零次或一次，则匹配该模式：

```js
    > "anything".match(/ny?/g);   
    ["ny", "n"]   

```

|

| `+` | 如果前面的模式至少出现一次（或更多次），则匹配该模式：

```js
    > "anything".match(/ny+/g);   
    ["ny"]   
    > "R2-D2 and C-3PO".match(/[a-z]/gi);   
    ["R", "D", "a",   "n", "d", "C", "P", "O"]   
    > "R2-D2 and C-3PO".match(/[a-z]+/gi);   
    ["R", "D", "and",   "C", "PO"]   

```

|

| `{n}` | 如果前面的模式恰好出现 n 次，则匹配该模式：

```js
    > "regular expression".match(/s/g);   
    ["s", "s"]   
    > "regular expression".match(/s{2}/g);   
    ["ss"]   
    > "regular expression".match(/\b\w{3}/g);   
    ["reg", "exp"]   

```

|

| `{min,max}` | 匹配前一个模式，如果它出现于 min 和 max 次数之间。你可以省略 max，这意味着没有最大次数，但只有最小次数。你不能省略 min。以下是一个输入为“doodle”且“o”重复 10 次的例子：

```js
    > "doooooooooodle".match(/o/g);   
    ["o", "o", "o",   "o", "o", 
    "o", "o", "o", "o",   "o"]   
    > "doooooooooodle".match(/o/g).length;   
    10   
    > "doooooooooodle".match(/o{2}/g);   
    ["oo", "oo", "oo",   "oo", "oo"]   
    > "doooooooooodle".match(/o{2,}/g);   
    ["oooooooooo"]   
    > "doooooooooodle".match(/o{2,6}/g);   
    ["oooooo", "oooo"]   

```

|

| `(pattern)` | 当模式在括号内时，它会被记住以便用于替换。这些也被称为捕获模式。捕获的匹配可用作$1, $2,... $9。匹配所有“r”出现并重复它们：

```js
    > "regular expression".replace(/(r)/g, '$1$1');   
    "rregularr exprression"   

```

匹配“re”并将其转换为“er”：

```js
    > "regular expression".replace(/(r)(e)/g, '$2$1');   
    "ergular experssion"   

```

|

| `(?:pattern)` | 非捕获模式，不会被记住且在$1, $2...中不可用。以下是一个如何匹配“re”的例子，但“r”不会被记住，第二个模式变为$1：

```js
   > "regular expression".replace(/(?:r)(e)/g, '$1$1');   
   "eegular expeession"   

```

|

确保你在特殊字符可能有两种含义时注意，例如`^`，`?`和`\b`。
