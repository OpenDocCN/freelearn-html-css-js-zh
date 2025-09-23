# 第七章. 与数据库一起工作

数据库查询也可以直接用 Opa 编写。Opa 目前（Opa 1.1.1）支持 NoSQL 数据库 MongoDB 和 CouchDB 以及 SQL 数据库 Postgres。Postgres 仍在开发中，未来版本计划支持更多数据库。Opa 提供了许多独特的先进操作符，并自动执行数据库查询以实现最大生产力。在本章中，我们将简要介绍如何与 MongoDB 一起工作。

# MongoDB 快速入门

首先，我们需要下载([`www.mongodb.org/downloads`](http://www.mongodb.org/downloads))，安装，并运行([`docs.mongodb.org/manual/installation/`](http://docs.mongodb.org/manual/installation/)) MongoDB 服务器。在 MongoDB 正确安装后，让我们从一个简单的例子开始：

```js
database int /counter = 0;
function page(){
    <h1 id="text">Hello {/counter}</h1>
    <input type="button" value="click" onclick={function(_){
          /counter++
          #text = "Hello {/counter}"	
    }}/>}
Server.start(Server.http, {title:"Opa Packt", ~page})
```

在上述代码的第一行中，我们定义了一个包含整数的`/counter`数据库路径。数据库路径描述了数据库中的位置，我们可以通过数据库路径读取、写入、更新和删除数据。请注意，路径的数据类型不能省略。

上述数据库是无名的；我们可以给数据库起一个名字，例如：

```js
database testdb {
    int /counter = 0;
}
```

这样，我们应该从路径`/testdb/counter`读取数据。现在，让我们编译并运行代码：

```js
$ opa 701.opa --

```

当应用程序启动时，如果服务器尚未运行，它将尝试启动 MongoDB 服务器，并将数据存储在默认位置`~/.opa/mongo/data`。如果服务器已经运行，应用程序将尝试连接到服务器。然而，我们也可以使用`--db-local`和`--db-remote`选项让程序连接到我们想要的特定数据库：

+   `--db-local:dbname [/path/to/db]`: 这使用在文件系统中指定位置的本地数据库

+   `--db-remote:dbname [username:password@]hostname[:port]`: 这使用在给定远程位置可访问的远程数据库

例如：

```js
$./701.js --db-local:testdb
$./701.js --db-local:testdb ~/data/mongo
$./701.js --db-remote:testdb localhost:27017
$./701.js --db-remote:testdb admin:admin@localhost:27017
```

# 数据库操作

我们可以通过数据库路径来操作数据。以下代码段声明了一个`testdb`数据库并定义了几个路径：

```js
type Student = {int id, string name, int age}
database testdb {
    int               /basic/i     //Basic type int
    float             /basic/f     //Basic type float
    string            /basic/s     //Basic type string
    Student           /stu         //Record
    list(string)      /lst         //List
    intmap(Student)   /stumap      //Map
    Student           /stuset[{id}]     //Set
}

```

我们定义了自己的学生类型。除了这个类型，我们的例子还涵盖了在数据库中最常使用的数据类型。

每个数据库路径都有一个默认值。每次我们尝试读取一个不存在的值（无论是由于它从未初始化还是已经被删除），都会返回默认值。以下列表显示了不同类型的默认值：

+   整数（int）的默认值是 0

+   浮点数（float）的默认值是 0.0

+   字符串的默认值是""

+   记录的默认值是默认值记录

+   枚举类型的默认值是最好地类似于空情况的值（例如，选项的{none}，列表的{nil}等）

我们可以在声明路径时分配一个值来定义一个特定应用程序的默认值，例如：

```js
    database testdb {
        int      /basic/i = 10
        string   /basic/s = "default"
        Student  /stu = {id: 0, name: "unknown", age: 25}
    }
```

要从数据库中读取数据，只需使用数据库路径，例如：

```js
int i = /testdb/basic/i
Student stu = /testdb/stu
```

我们可以在路径前加上问号（`?`），然后给路径赋予两个选项之一，其中`{some: x}`表示该路径的值是`x`，而`{none}`表示该路径尚未写入，例如：

```js
match (?/testdb/basic/i) {
case {none}: println("unknown");
case {some: x}: println("{x}");
}
```

上述示例如果路径`/testdb/basic/i`尚未写入或已被删除，则打印**unknown**，否则打印该路径的值。

要写入或更新数据库路径，使用操作符`=`。我们还可以使用`<-`来赋值，它与`=`相同。例如：

```js
/testdb/basic/i = 10
/testdb/basic/i <- 10 //the same as above
/testdb/basic/s = "my new string"
/testdb/stu = {id: 1, name: "Li", age: 28}In addition, you can also use the following shortcuts to update integers in database:
/testdb/basic/i++;     //add the integer i by 1
/testdb/basic/i += 5;  //add the integer i by 5
/testdb/basic/i -= 10;	 //minus the integer i by 10
```

要删除存储在路径中的数据，使用`Db.remove(@path)`函数，其中`@path`是路径的引用。我们可以通过在路径前添加`@`符号来获取路径引用，例如：

```js
Db.remove(@/testdb/basic/i)
Db.remove(@/testdb/stu)
```

## 记录

使用记录，我们可以像基本类型一样进行完整的读取和更新：

```js
stu = /testdb/stu;                         //read record
/testdb/stu = {id: 1, name: "Li", age: 28} //update record
```

有时，我们需要强制记录只能整体修改。这被称为**完全修改**。如果一个记录被声明为可以完全修改，那么在执行修改时必须一次性更新所有字段。我们通过在数据库路径后添加`full`关键字来表示该路径可以完全修改。如果给定的路径没有声明为完全修改，我们可以跨越记录边界，通过在路径中包含它们来访问或更新所选字段。考虑以下示例：

```js
type Student = {int id, string name, int age}
database testdb {
    Student /stu1
    Student /stu2
    /stu2 full           //declare /stu2 as full modification
}
/testdb/stu1/name = "Li" //OK
/testdb/stu2 = {id:1, name: "Li", age: 28} //OK
/testdb/stu2/name = "Li" //error: will not compile
```

我们通过添加`/stu2 full`语句声明了`/stu2`以进行完全修改。因此，编译器报告了前一段代码中最后一行（`/testdb/stu2/name = "Li"`）的错误，其中我们尝试修改记录的单个字段，即`name`字段。

## 列表

如同在第二章的*列表*部分所述，*基本语法*，Opa 中的列表只是递归记录。我们可以以与记录相同的方式操作列表。然而，由于列表数据类型使用频率很高，Opa 提供了针对列表的特定快捷方式：

```js
/testdb/lst = ["I", "Love", "Opa", "!"] //Update an entire list
/testdb/lst pop             // Removes first element of a list
/testdb/lst shift           // Removes last element of a list
/testdb/lst <+ "element"    // Append an element
/testdb/lst <++ ["How", "about", "you"] // Append several elements
/testdb/lst <--* "element"              // Remove an element
/testdb/lst <-- ["How", "about", "you"] // Remove several elements
```

## 集合和映射

我们可以像更新列表一样更新集合和映射，然而，访问元素的方式是不同的。我们可以通过引用其主键从给定的集合或映射中获取单个值，例如：

```js
stu = /testdb/stuimap[1] //find element whose key is 1
stu = /testdb/stuset[1]  //find element whose primary key is 1
stu = /testdb/stuset[{id:1}] //the same as above
/testdb/stuset[{id:1}] = {name: "Li"}  //update the chosen item
```

此外，我们可以在方括号内添加查询条件来查询一组值，例如：

```js
/testdb/stuset[id < 10] <- {age: 25}
/testdb/stuset[age >= 25] <- {age++}
```

我们将在下一节中更详细地探讨查询。

# 查询数据

正如我们在上一节中提到的，数据库集合和映射是两种允许在数据库中组织多个数据实例的集合类型。我们可以使用以下运算符查询一组值：

```js
== expr: equals expr
!= expr: not equals expr
< expr:  lesser than expr
<= expr: lesser than or equals expr
> expr:  greater than expr
>= expr: greater than or equals expr
in expr: "belongs to" expr, where expr is a list
q1 or q2: satisfy query q1 or q2
q1 and q2: satisfy both queries, q1 and q2
not q: does not satisfy q
{f1 q1, f2 q2, ...}: the database field f1 satisfies q1, field f2 satisfies q2 etc.
```

此外，我们可以指定一些查询选项如下：

+   `skip n`：这里`expr`应该是一个类型为`int`的表达式，并且它将跳过前*n*个结果。

+   `limit n`：这里`expr`应该是一个类型为`int`的表达式，返回最多*n*个结果。

+   `order fld (, fld)+`: 这里 `fld` 指定了一个排序。`fld` 可以是一个单独的标识符或一个标识符列表，指定了排序应基于的字段。标识符可以可选地以 `+` 或 `-` 前缀来指定升序或降序。最后，可以使用 `<ident>=<expr>` 来动态选择排序，其中 `<expr>` 应该是 `{up}` 或 `{down}` 类型。

以下代码片段获取了年龄在 20 岁以上和 45 岁以下的学生接下来的 50 个结果，并且将它们按年龄（升序）排序，然后按 `id`（降序）排序：

```js
dbset(Student, _) stus = /testdb/stuset[age >= 20 and age <= 45; skip 50; limit 50; order +age, -id]
```

我们可以通过组合查询表达式来创建更复杂的查询条件。查询操作返回一个 `dbset`。`dbset` 是一个包含查询结果的集合。因此，我们可以遍历 `dbset`。考虑以下代码片段。它查询所有名为 `Li` 的学生，并将它们打印出来：

```js
dbset(Student,_) lis = /testdb/stuset[name == "Li"]
iter it = DbSet.iterator(lis)
Iter.iter(function(li){
println("{li}")
},it) 
```

# 摘要

在本章中，我们介绍了与数据库一起工作的基本技术。我们首先给出了一个非常简单的例子。然后，我们讨论了如何操作数据，包括从数据库中检索数据、写入或更新数据以及删除数据。我们涵盖了基本类型和复杂类型，如记录、列表、映射和集合。最后，我们讨论了如何从集合和映射中查询数据。
