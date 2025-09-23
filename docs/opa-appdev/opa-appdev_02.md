# 第二章：基本语法

在本章中，我们将介绍 Opa 的基本语法。本章不会涵盖 Opa 的每一个细节，但它是一些你应该了解的知识点。同时假设你具备一些计算机编程的基础知识。

# 基本数据类型

数据类型是应用程序操作的数据的形状。Opa 使用数据类型对你的应用程序进行合理性和安全性检查。Opa 还使用数据类型执行多项优化。Opa 有三种基本数据类型：整数、浮点数和字符串。此外，你可以使用 `type` 关键字定义自己的类型：

```js
type Student = {string name, int age, float score}
Student stu = { name:"li", age:28, score:80.0}
```

实际上，多亏了类型推断机制，Opa 在大多数情况下即使你没有提供任何类型信息也能工作。例如：

```js
x = 10        // the same as: int x = 10
x = {a:1,b:2} // the type of x is: {a:int, b:int}
```

因此，在本章的其余部分，我们不会在变量之前讨论类型信息，但你应该在心里知道它的类型。在实际编码中，一个最佳实践是提供我们主要函数的数据类型，并让推断引擎获取所有局部变量和次要函数的数据类型。

## 整数

编写整型字面量相当简单；有几种方式可以做到这一点：

```js
x = 10     // 10 in base 10
x = 0xA    // 10 in base 16, any case works (0Xa, 0XA, Oxa)
x = 0o12   // 10 in base 8
x = 0b1010 // 10 in base 2
```

### 注意

在 Opa 中，尾随的分号是可选的；如果你想的话可以添加它。

Opa 提供了 `Int` 模块（[`doc.opalang.org/module/stdlib.core/Int`](http://doc.opalang.org/module/stdlib.core/Int)）来操作整数。以下是最常用的函数：

```js
i1 = Int.abs(-10)        // i1 = 10
i2 = Int.max(10,8)       // i2 = 10
```

在 `float`、`int` 和 `String` 之间没有自动类型转换。因此，请使用以下函数在 `int`、`float` 和 `String` 之间进行转换。

```js
i3 = Int.of_float(10.6)      // i3 = 10
i4 = Int.of_string("0xA")    // i4 = 10, 0xA is 10 in dec
f1 = Int.to_float(10) 	       // f1 = 10.0, f1 is a float
s1 = Int.to_string(10)       // s1 = "10", s1 is a string
```

## 浮点数

定义浮点数也很容易。它们可以写成以下方式：

```js
x = 12.21   // the normal one
x = .12     // omitting the leading zero
x = 12\.     // to indicate this is a float, not an integer
x = 12.5e10 // scientific notation
```

Opa 提供了 `Float` 模块（[`doc.opalang.org/module/stdlib.core/Float`](http://doc.opalang.org/module/stdlib.core/Float)）来操作浮点数。以下是最常用的函数：

```js
f1 = Float.abs(-10.0)        //f1 = 10.0
f2 = Float.ceil(10.5)        //f2 = 11.0
f3 = Float.floor(10.5)	       //f3 = 10.0
f4 = Float.round(10.5)       //f4 = 11.0
f5 = Float.of_int(10)        //f5 = 10.0
f6 = Float.of_string("10.5") //f6 = 10.5
i1 = Float.to_int(10.5)      //i1 = 10, i1 is an integer
s1 = Float.to_string(10.5)   //s1 = "10.5", s1 is a string
```

## 字符串

在 Opa 中，文本由不可变的 utf8 编码字符字符串表示。字符串字面量遵循类似于 C 语言、Java 或 JavaScript 中使用的语法。请注意，你必须使用反斜杠转义特殊字符。

```js
x = "hello!"
x = "\"" // special characters can be escaped with backslashes
```

Opa 有一个称为字符串插入的功能，可以将任意表达式放入字符串中。你可以通过在字符串中嵌入花括号内的表达式来实现这一点。例如：

```js
x = "1 + 2 = {1+2}"   //will produce "1 + 2 = 3"
lang = "Opa"
y = "I love {lang}!"  //will produce "I love Opa!"
```

Opa 提供了 `String` 模块（[`doc.opalang.org/module/stdlib.core/String`](http://doc.opalang.org/module/stdlib.core/String)）来操作字符串。以下是最常用的功能：

```js
s = "I love Opa! "              //Note there is a space at the end.
len = String.length(s)          //get length, len = 12
isEmpty = String.is_empty(s)    //test if a string is empty, false
isBlank = String.is_blank(s)    //test if a string is blank, false
cont = String.contains(s,"Opa") //check if a string contains a 
                                 //substring,true
idx1 = String.index("love",s)   //found, idx1 = {some:2}
idx2 = String.index("loving",s) //not found, idx2 = {none}
ch = String.get(0,s)            //get nth char, ch = 'I'
s2 = String.trim(s)             //do trim, s2 = "I love Opa!"
s3 = String.replace("I","We",s2)//s3 = "We love Opa!"
```

## 和

一个值有一个类型 `t` 或 `u` 的和类型，这意味着这个类型的值可以是两种变体之一，即类型 `t` 的值或类型 `u` 的值。

和类型的一个好例子是布尔值，其定义如下：

```js
type bool = {true} or {false}
```

因此，类型为 `bool` 的变量可以是 `{true}` 或 `{false}`。另一个常用的和类型是 `option` 类型，其定义如下：

```js
type option('a) = {none} or {'a some}
```

`option(`a`)` 值要么是 `none`，要么是 `some`（类型为 `a` 的值 x）。类型 `a` 表示任何类型。这是一种类型安全地处理可能不存在值的方法。`option` 类型被广泛使用；以 `String.index` 为例：

```js
idx1 = String.index("love","I love Opa!")   //idx1 = {some:2}
idx2 = String.index("loving","I love Opa!") //idx2 = {none}
```

`String.index` 的返回类型是选项（`int`），这意味着如果出现子串，它将返回一个 `{some:int}` 记录；如果没有出现，则返回一个 `{none}` 记录。

注意，求和数据类型不仅限于两种情况；它们可以有数十种情况。

# 函数

Opa 是一种函数式语言。其特性之一是函数是常规值，这意味着函数可以作为参数传递或作为结果返回。因此，它们遵循与其他任何值相同的命名规则。

```js
function f(x,y){      // function f with the two parameters x and y
  x + y + 1
}
function int f(x,y){  // explicitly indicates the return type
  x + y + 1
}
```

## 最后的表达式返回

你可能会注意到函数体中没有返回语句。这是因为 Opa 使用最后表达式返回，这意味着函数的最后表达式是返回值。例如：

```js
function max(x,y){
  if(x >= y) x else y
}
```

如果 `x` 大于或等于 `y`，则 `x` 是最后一个表达式，并将返回 `x`；如果 `y` 大于 `x`，则 `y` 是最后一个表达式，并将返回 `y`。

## 模块

功能通常被重新组合到模块中；例如：

```js
module  M {
  x = 1
  y = x
  function test(){ jlog("testing") }
}
```

我们可以通过使用点操作符（`.`）来访问模块的内容；例如，`M.x`、`M.y` 和 `M.test`。实际上，模块的内容不是字段定义，而是绑定。在这个例子中，我们将整数 `1` 绑定到变量 `x`，并将变量 `x` 的值绑定到变量 `y`。

# 数据结构

在 Opa 中构建数据结构的唯一方法是使用记录，我们将在稍后讨论。所有其他数据结构，如元组和列表，都是基于记录的。Opa 提供了不同的模块来帮助用户操作列表和映射。让我们首先看看记录。

## 记录

简而言之，记录是一组数据的集合。以下是构建记录的方法：

```js
x = {} // the empty record
x = {a:2,b:3} //a record with field "a" and "b"
```

空记录 `{}` 有一个同义词 `void`，意味着相同的意思。有几种语法简写可用于简洁地编写记录。首先，如果你给出字段名而没有字段值，这意味着该字段的值是 `void`：

```js
x = {a}      // means {a:void}
x = {a, b:2} // means {a:void b:2}
```

我们经常使用的第二个简写是符号 `~`。它表示如果字段值留空，则将其分配给与字段名相同的变量：

```js
x = {~a, b:2}    // means {a:a, b:2}
x = ~{a, b}      // means {a:a, b:b}
x = ~{a, b, c:4} // means {a:a, b:b, c:4}
x = ~{a:{b}, c}  // means {a:{b:void}, c:c}, NOT {a:{b:b}, c:c}
//Consider this more meaningful example
name = "Li"; sex  = "Male"; age  = 28; 
person = ~{name, sex, age} //means {name:"Li", sex:"Male", age: 28}
```

我们还可以使用关键字 `with` 从现有记录构建记录：

```js
x = {a:1,b:2,c:3}
y = {x with a:"1",b:5} // y = {a:"1", b:5, c:3}
```

注意，你可以重新定义任意多的字段。在我们刚才看到的例子中，`y` 中的字段 `a` 是一个字符串，但 `x` 中的字段 `a` 是一个整数。这里有一些关于派生的更多例子：

```js
X = {a:1, b:{c:"2", d:3.0}}
// you can update fields deep in the record
y = {x with b.c:"200"}  // y = {a:1, b:{c:"200", d:3.0}}
// you can use the same syntactic shortcuts as used before
y = {x with a}          // means {x with a:void}
y = {x with ~a}         // means {x with a:a}
y = ~{x with a, b:{e}}  // means {x with a:a, b:{e}}
```

### 元组

N-元组是一个由 *n* 个元素组成的序列，其中 N 是一个正整数。在 Opa 中，N-元组只是一个具有字段 `f1` 到 `fN` 的记录：

```js
x = (1,)          // a tuple of size 1, the same as {f1:1}
x = (1,"2",{a:3}) // a size 3 tuple, the same as {f1:1, f2:"2", f3:{a:3}} 
y = {x with f1:2} // we can manipulate a tuple the same way as a 
                    //record
// y = {f1:2, f2:"2", f3:{a:3}}
```

注意第一个情况中的尾随逗号；它区分了 1-元组和大括号表达式。尾随逗号适用于任何其他元组，尽管在这种情况下写或不写都没有区别。

## 列表

在 Opa 中，一个列表（链表）是一个不可变的数据结构，旨在包含有限或无限个相同类型的元素集合。实际上，列表只是一个具有特殊结构的记录，定义为：

```js
type list('a) = {nil} or {'a hd, list('a) tl}
```

这是构建列表的方法：

```js
x = []      // the empty list, equals to {nil}
x = [2,3,4] // a three element list, the same as a record: 
             // {hd:2, tl:{hd:3, tl:{hd:4, tl:{nil}}}}
y = [0,1|x] // this will put 0,1 on top of x: [0,1,2,3,4]
```

Opa 中的列表与 C 语言和 Java 中的数组非常相似。但也有一些区别。首先，Opa 中的列表是不可变的，这意味着列表的元素不能通过赋值来改变。其次，我们操作列表的方式不同。我们使用`List`模块（[`doc.opalang.org/module/stdlib.core/List`](http://doc.opalang.org/module/stdlib.core/List)）在 Opa 中管理列表。以下是在列表上最常用的操作（将在后续章节中解释）：

```js
l = [1,2,3]
len = List.length(l)       // return the length of a list
isEmpty = List.is_empty(l) // test if a list is empty
head = List.head(l)        // get the first element, will fail if 
                             // the list is empty
element = List.get(0,l)    // get nth element, return option('a)
l1 = List.add(4,l)         // adding an element at the head of a 
                             //list
l2 = 4 +> l                // a shortcut for List.add
l3 = List.remove(3,l)      // removing an element
l4 = List.drop(2,l)        // drop the first n elements
```

### 遍历列表

在 C 语言或 Java 中，我们使用`for`或`while`循环来遍历列表或数组。它们看起来像这样：

```js
int[] numbers = [1,2,3,4,5]
for(int i=0; i<numbers.length; i++){ //do something }
```

但在 Opa 中，情况完全不同。要遍历一个列表，我们使用`List.fold`或`List.foldi`。`List.fold`是一个强大的函数，你可以用它来对列表执行几乎任何操作。以下是一个获取列表长度的简单示例：

```js
len = List.fold(function(_,i){ i+1 }, ["a","b","c"], 0)
```

`List.fold`接受三个参数。第一个是一个函数，第二个是列表，第三个是初始值。它遍历列表并对每个元素应用该函数。所以，如果我们命名这个函数为`f`，它的执行方式如下：

```js
len = f("c", f("b", f("a",0)))
```

首先，`f("a",0)`将被执行并返回 1，这里的`0`是初始值，`a`是第一个元素。然后`f("b",1)`将返回 2，最后`f("c",2)`将返回 3。以下是一个更复杂的示例：

```js
//find the max natural number in the list
max = List.fold(function(x,a){ 
  if(x > a) x else a
},[1,4,3,2,7,8,5],0)
```

### 查找元素

我们有多种方法在列表中查找元素。`List.index`搜索元素的第一次出现并返回其索引。`List.index_p`搜索匹配给定函数的任何元素的第一次出现并返回其索引。`List.find`与`List.index_p`相同，但返回元素本身而不是其索引。例如：

```js
l = ["a","b","c"]
r1 = List.index("b",l)                      // r1 = {some:1}
r2 = List.index("x",l)                      // r2 = {none}
r3 = List.index_p(function(x){ x == "b"},l) // r3 = {some:1}
r4 = List.find(function(x){ x == "b"},l)    // r4 = {some:"b"}
```

### 列表转换

如果你想要将元素投影到一个新的列表中，例如将列表中的数字加倍或选择奇数，你可以使用`List.map`和`List.filter`来实现。以下是一些示例：

```js
l1 = [1,2,3,4,5]
l2 = List.map(function(x){ 2*x }, l1); //l2 = [2,4,6,8,10]
l3 = List.filter(function(x){mod(x,2) == 0},l1); // l3 = [2,4]
```

### 排序列表

调用`List.sort`函数以通常的顺序对列表进行排序。通常的顺序是指默认的顺序，即从小到大排序数字和按字母顺序排序的字符串。考虑以下代码：

```js
l1 = List.sort(["by","as","of","At"]) //l1 = ["At","as","by","of"]
l2 = List.sort([1,3,4,6,2])           //l2 = [1,2,3,4,6]
```

`List.sort_by`使用通常的顺序，但它将元素投影出来，例如在比较之前将字符串转换为小写。`List.sort_with`允许我们使用自己的比较函数。

为了更清楚地说明，假设有三个点，P1 (1, 3)，P2 (3, 2)，和 P3 (2, 1)，我们想要以两种不同的方式对它们进行排序：

+   通过它们的 Y 坐标

+   通过坐标原点（0, 0）的距离

让我们看看在 Opa 中如何做到这一点：

```js
p = [{x:1,y:3},{x:3,y:2},{x:2,y:1}]
l1 = List.sort_with(function(p1,p2){  // sort by Y corordination
  if(p1.y >= p2.y) {gt} else {lt}
},p)
l2 = List.sort_by(function(p){        //sort by distance
  p.x*p.x + p.y*p.y    
},p)
```

## 映射

映射是一个重要的数据结构，就像列表一样。在 Opa 中，映射最常见的用例是 `stringmap` 和 `intmap`。`stringmap` 是从字符串到某种类型值的映射，而 `intmap` 是从数字到某种类型值的映射。

我们操作映射的方式几乎与列表相同，再次重复是不明智的。以下是一些最常用的操作：

```js
m1 = Map.empty                 // create an empty map
m2 = StringMap.empty           // create an empty stringmap
m3 = IntMap.empty              // create an empty intmap
m4 = Map.add("key1","val1",m1) // adding a key-val pair
v1 = Map.get("key1",m4)        // getting a value
m5 = Map.remove("key1",m4)     // removing a key
```

# 模式匹配

模式匹配是 C 语言或 Java 的 `switch` 语句的泛化。在 C 语言和 Java 中，`switch` 语句仅允许你根据整数（包括 `char`）或 `enum` 值从许多语句中选择。而在 Opa 中，模式匹配比这更强大。模式匹配的更通用语法如下：

```js
match(<expr>){
case <case_1>: <expression_1>
case <case_2>: <expression_2>
case <case_n>: < expression_n>
}
```

当模式执行时，`<expr>` 被评估为一个值，然后按顺序与每个模式匹配，直到找到匹配的案例。你可以这样想：

```js
if (case_1 matched) expression_1 else {
  if (case_2 matched) expression_2 else {
    ...
         if (case_n matched) expression_n else no_matches
         ...
  }
}
```

模式匹配的规则很简单，如下所示：

+   **规则 1**：任何值都匹配模式 `_`

+   **规则 2**：任何值都匹配变量模式 `x`，并且该值被绑定到标识符 `x`

+   **规则 3**：当整数/浮点数/字符串相等时，它们匹配整数/浮点数/字符串模式

+   **规则 4**：当两个记录具有相同的字段且字段值按组件匹配模式时，记录（包括元组和列表）匹配封闭记录模式

+   **规则 5**：当值具有模式的所有字段（但可以更多）且公共字段的值按组件匹配模式时，记录（包括元组和列表）匹配开放记录模式

+   **规则 6**：当值匹配模式时，它匹配为 `x` 模式，并且此外它将 `x` 绑定到该值

+   **规则 7**：如果一个值匹配两个子模式中的一个，则该值匹配 `OR` 模式

+   **规则 8**：在所有其他情况下，匹配失败

前三条规则和最后三条规则（规则 1、2、3、6、7、8）很容易理解。让我们来看看它们：

```js
match(y){
case 0:       //if y == 0, match [rule 3]
case 1 as x:  //if y == 1, match and 1 is bound to x [rule 6]
case 2 | 3 :  //if y is 2 or 3, match [rule 7]
case x:       //any value will match and the value is bound
                //to x [rule 2]
case _:       //match, we do not care about the value.
}
```

### 注意

这段代码无法编译，我们只是用它来展示规则。

规则 4 和规则 5 稍微复杂一些。一个紧密记录模式是一个具有固定字段的记录。一个开放记录模式是一个以 `…` 结尾的记录，表示它可能包含我们不关心的其他字段。以下示例可能使这一点更清晰：

```js
x = {a:1, b:2, c:3}
match(x){
case {a:1,b:2}:     //a close record pattern, but will not match //cause they do not have the same fields [rule 4]
case {a:1,b:2,c:2}: //a close record pattern, still will not match //cause c is not equal [rule 4]
case {a:1,b:2,...}: //An open record pattern, matches [rule 5]
}
```

我们也可以匹配元组和列表（因为元组和列表是特殊的记录，它们并不难理解）。例如：

```js
t = (1,"2",3.0)
match(t){         //matching a tuple
case (1,"2",3.1): //not match, 3.1 != 3.0
case (1,"2",_):   //match, _ matches anything
case (1,"2",x):   //match, now x = 3.0
case {f1:1 ...}:  //match, remember tuples are just records
}
y = [1,2,3]
match(y){         //matching a list
case [1,2]:       //not match
case [1,2,_]:     //match, _ matches anything
case [1,2,x]:     //match, now x = 3
case [2,|_]:      //not match, '|_' means the rest of the list
case [1,|_]:      //match 
case [1,2,|_]:    //match
case [1,x|_]:     //match, now x = 2
}
```

# 文本解析器

解析是网络应用程序需要经常执行的操作。Opa 提供了一种内置的语法来构建文本解析器，这些解析器就像函数一样是一等值。解析器基于解析表达式语法([`en.wikipedia.org/wiki/Parsing_expression_grammar`](http://en.wikipedia.org/wiki/Parsing_expression_grammar))，一开始可能看起来像正则表达式，但它们的行为与正则表达式截然不同。文本解析器相对于正则表达式的一个主要优点是你可以轻松地组合解析器。一个很好的例子是解析 URL。让我们立即开始我们的第一个 Opa 解析器：

```js
first_parser = parser { 
case "Opa"  : 1 
}
```

对于`first_parser`，表达式只是字面字符串，这意味着这个解析器只有在用字符串`"Opa"`喂入时才会成功。那么如何使用这个解析器呢？`Parser`模块([`doc.opalang.org/module/stdlib.core.parser/Parser`](http://doc.opalang.org/module/stdlib.core.parser/Parser))提供了一系列处理解析器的函数。其中最重要的一项是：

```js
Parser.try_parse : Parser.general_parser('a), string -> option('a)
```

它接受一个解析器和字符串作为参数，并产生某种类型的可选值。让我们看看如何使用这个函数：

```js
x = Parser.try_parse(parser1,"Opa")  //x = {some: 1}
y = Parser.try_parse(parser1,"Java") //y = {none}
```

现在让我们考虑以下解析器：

```js
digit1 = parser { case x=[0-9]+: x }
digit2 = parser { case x=([0-9]+): x }
```

`digit1`和`digit2`都接受像`"5"`、`"100"`这样的数字字符串，并且两者都会将值赋给标识符`x`。如果我们用字符串`"100"`来喂`digit1`解析器，`x`将是字符串的解析结果：字符列表`['1','0','0']`。如果我们用字符串`"100"`来喂`digit2`解析器，`x`将是输入字符串：`100`。因此，如果我们想获取输入字符串，我们需要将表达式放在括号内。

让我们更进一步；考虑以下解析器：

```js
abs_parser = parser{
  case x=("+"?[0-9]+): Int.of_string("{x}")
  case x=("-"[0-9]+) : 0 – Int.of_string("{x}")
}
x = Parser.try_parse(abs_parser,"-100") // x = {some: 100}
```

此解析器接受一个整数字符串并返回其绝对值。你可以利用之前的知识来理解它是如何工作的。请注意，尽管 PEG 的表达式看起来像正则表达式，但它们是不同的。

# 摘要

本章向您介绍了 Opa 编程语言的基本语法，包括数据类型、函数、记录、元组、列表、映射、模式和解析器。这是我们编写良好的 Opa 程序所应掌握的基本知识。在具备这些知识的基础上，我们将看到如何在下一章中开发一个网络应用程序。
