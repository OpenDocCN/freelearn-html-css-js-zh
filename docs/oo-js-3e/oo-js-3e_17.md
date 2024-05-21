# 附录 D. 正则表达式

当您使用正则表达式（在第四章中讨论，*对象*）时，您可以匹配文字字符串，例如：

```js
    > "some text".match(/me/); 
    ["me"] 

```

然而，正则表达式的真正威力来自于匹配模式，而不是文字字符串。以下表格描述了您可以在模式中使用的不同语法，并提供了一些使用示例：

| **模式** | **描述** |
| --- | --- |

| `[abc]` | 匹配一类字符：

```js
    > "some text".match(/[otx]/g);   
    ["o", "t", "x",   "t"]   

```

|

| `[a-z]` | 作为范围定义的一类字符。例如，[a-d]与[abcd]相同，[a-z]匹配所有小写字符，[a-zA-Z0-9_]匹配所有字符，数字和下划线字符：

```js
    > "Some Text".match(/[a-z]/g);   
    ["o", "m", "e",   "e", "x", "t"]   
    > "Some Text".match(/[a-zA-Z]/g);   
    ["S", "o", "m",   "e", "T", "e", "x", "t"]   

```

|

| `[^abc]` | 匹配不属于字符类的所有内容：

```js
    > "Some Text".match(/[^a-z]/g);   
    ["S", " ", "T"]   

```

|

| `a&#124;b` | 匹配 a 或 b。管道字符表示或，可以使用多次：

```js
    > "Some Text".match(/t&#124;T/g);
    ["T", "t"]
    > "Some Text".match(/t&#124;T&#124;Some/g);
    ["Some", "T",   "t"]

```

|

| `a(?=b)` | 只有在后面跟着 b 时才匹配 a：

```js
    > "Some Text".match(/Some(?=Tex)/g);   
    null   
    > "Some Text".match(/Some(?=Tex)/g);   
    ["Some"]   

```

|

| `a(?!b)` | 只有在后面不跟着 b 时才匹配 a：

```js
    > "Some Text".match(/Some(?!Tex)/g);   
    null   
    > "Some Text".match(/Some(?!Tex)/g);   
    ["Some"]   

```

|

| `\` | 转义字符，用于帮助您匹配模式中用作文字的特殊字符：

```js
    > "R2-D2".match(/[2-3]/g);   
    ["2", "2"]   
    > "R2-D2".match(/[2\-3]/g);   
    ["2", "-", "2"]   

```

|

| `\n``\r``\f``\t``\v` | 换行符回车换行符制表符垂直制表符 |
| --- | --- |

| `\s` | 空格，或前面五个转义序列中的任何一个：

```js
    > "R2\n D2".match(/\s/g);   
    ["\n", " "]   

```

|

| `\S` | 与上面相反；匹配除空格之外的所有内容。与[^\s]相同：

```js
    > "R2\n D2".match(/\S/g);   
    ["R", "2", "D",   "2"]   

```

|

| `\w` | 任何字母，数字或下划线。与[A-Za-z0-9_]相同：

```js
    > "S0m3 text!".match(/\w/g);   
    ["S", "0", "m",   "3", "t", "e", "x", "t"]   

```

|

| `\W` | `\w`的相反：

```js
    > "S0m3 text!".match(/\W/g);   
    [" ", "!"]   

```

|

| `\d` | 匹配数字，与[0-9]相同：

```js
    > "R2-D2 and C-3PO".match(/\d/g);   
    ["2", "2", "3"]   

```

|

| `\D` | `\d`的相反；匹配非数字，与[⁰-9]或[^\d]相同：

```js
    > "R2-D2 and C-3PO".match(/\D/g);   
    ["R", "-", "D",   " ", "a", "n", "d",
      " ", "C",   "-", "P", "O"]   

```

|

| `\b` | 匹配词边界，如空格或标点符号。匹配 R 或 D 后面跟着 2：

```js
    > "R2D2 and C-3PO".match(/[RD]2/g);   
    ["R2", "D2"]   

```

与上面相同，但只在单词的末尾：

```js
    > "R2D2 and C-3PO".match(/[RD]2\b/g);   
    ["D2"]   

```

相同的模式，但输入中有一个破折号，这也是一个单词的结尾：

```js
    > "R2-D2 and C-3PO".match(/[RD]2\b/g);   
    ["R2", "D2"]   

```

|

| `\B` | `\b`的相反：

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

| `\u0000` | 匹配 Unicode 字符，由四位十六进制数字表示：

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

| `^` | 要匹配的字符串的开头。如果设置了`m`修饰符（多行），则匹配每行的开头：

```js
   > "regular\nregular\nexpression".match(/r/g);   
    ["r", "r", "r",   "r", "r"]   
    > "regular\nregular\nexpression".match(/^r/g);   
    ["r"]   
   > "regular\nregular\nexpression".match(/^r/mg);   
    ["r", "r"]   

```

|

| `$` | 匹配输入的结尾，或者在使用多行修饰符时，匹配每行的结尾：

```js
    > "regular\nregular\nexpression".match(/r$/g);   
    null   
    > "regular\nregular\nexpression".match(/r$/mg);   
    ["r", "r"]   

```

|

| `.` | 匹配除换行符和换行符之外的任何单个字符：

```js
    > "regular".match(/r./g);   
    ["re"]   
    > "regular".match(/r.../g);   
    ["regu"]   

```

|

| `*` | 如果出现零次或多次，则匹配前面的模式。例如，`/.*/`将匹配任何内容，包括空（空输入）：

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

| `?` | 如果出现零次或一次，则匹配前面的模式：

```js
    > "anything".match(/ny?/g);   
    ["ny", "n"]   

```

|

| `+` | 如果至少出现一次（或更多次），则匹配前面的模式：

```js
    > "anything".match(/ny+/g);   
    ["ny"]   
    > "R2-D2 and C-3PO".match(/[a-z]/gi);   
    ["R", "D", "a",   "n", "d", "C", "P", "O"]   
    > "R2-D2 and C-3PO".match(/[a-z]+/gi);   
    ["R", "D", "and",   "C", "PO"]   

```

|

| `{n}` | 如果出现 n 次，则匹配前面的模式：

```js
    > "regular expression".match(/s/g);   
    ["s", "s"]   
    > "regular expression".match(/s{2}/g);   
    ["ss"]   
    > "regular expression".match(/\b\w{3}/g);   
    ["reg", "exp"]   

```

|

| `{min,max}` | 如果出现在 min 和 max 次之间，则匹配前面的模式。您可以省略 max，这将意味着没有最大值，但只有最小值。您不能省略 min。输入为“doodle”，其中“o”重复了 10 次的示例：

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

| `(pattern)` | 当模式在括号中时，它会被记住，以便可以用于替换。这些也被称为捕获模式。捕获的匹配可用作$1，$2，... $9 匹配所有“r”出现并重复它们：

```js
    > "regular expression".replace(/(r)/g, '$1$1');   
    "rregularr exprression"   

```

匹配“re”并将其变为“er”：

```js
    > "regular expression".replace(/(r)(e)/g, '$2$1');   
    "ergular experssion"   

```

|

| `(?:pattern)` | 非捕获模式，不会被记住，也不会在$1，$2...中可用。以下是一个示例，说明如何匹配“re”，但不会记住“r”，第二个模式变为$1：

```js
   > "regular expression".replace(/(?:r)(e)/g, '$1$1');   
   "eegular expeession"   

```

|

当特殊字符有两种含义时，请确保您注意，就像`^`，`?`和`\b`一样。
