# 第八章：*附录*

## 关于

本节旨在帮助学生执行书中的活动。它包括详细的步骤，学生需要执行这些步骤以实现活动的目标。

## 第一章：介绍 ECMAScript 6

### 活动 1 - 实现生成器

您被要求构建一个简单的应用程序，根据请求生成斐波那契数列中的数字。该应用程序为每个请求生成序列中的下一个数字，并在给定输入时重置序列。使用生成器生成斐波那契数列。如果将一个值传递给生成器，则重置序列。为了简单起见，您可以从 n=1 开始斐波那契数列。

为了突出生成器如何用于构建迭代数据集，请按照以下步骤进行：

1.  查找斐波那契数列并了解下一个值是如何计算的。

1.  为斐波那契数列创建一个生成器。

1.  在生成器内，使用变量`n2`和`n1`设置`current`和`next`（0, 1）的默认值。

1.  创建一个无限的`while`循环。

1.  在`while`循环内，使用`yield`关键字提供序列中的当前值，并将 yield 语句的返回值保存到名为`input`的变量中。

1.  如果输入包含一个值，则将变量`n2`和`n1`重置为它们的起始值。

1.  在`while`循环内，从`current` + `next`计算出新的下一个值，并将其保存到变量 next 中。

1.  否则，将`n2`更新为包含`n1`（`next`值）的值，并将`n1`设置为我们在`while`循环顶部计算的`next`值。

**代码：**

##### **index.js**

```php
function* fibonacci () {
 let n2 = 0;x
 let n1 = 1;
 while ( true ) {
   let input = yield n2;
   if ( input ) {
     n1 = 1;
     n2 = 0;
   } else {
     let next = n1 + n2;
     [ n1, n2 ] = [ next, n1 ];
   }
 }
}
let gen = fibonacci();
```

###### 片段 1.87：实现生成器

https://bit.ly/2CV4KAi

**结果：**

![图 1.19：带有生成器的斐波那契数列](img/Figure_1.19.jpg)

###### 。

###### 图 1.19：带有生成器的斐波那契数列

您已成功演示了如何使用生成器基于斐波那契数列构建迭代数据集。

## 第二章：异步 JavaScript

### 活动 2 - 使用 Async/Await

您被要求构建一个与数据库交互的服务器。您必须编写一些代码来查找数据库中的集合和基本用户对象。导入`simple_db.js`文件。使用`async/await`语法编写以下程序，使用`get`和`insert`命令：

+   查找名为`john`的键，名为`sam`的键，以及您的名字作为数据库键。

+   如果数据库条目存在，则记录结果对象的`age`字段。

+   如果您的名字在数据库中不存在，请插入您的名字并关联一个包含您的名字、姓氏和年龄的对象。查找新的数据关联并记录年龄。

对于任何失败的`db.get`操作，将键保存到数组中。在程序结束时，打印失败的键。

**DB API：**

`db.get（index）：`

这需要一个索引并返回一个 promise。如果索引不存在，数据库查找失败，或者未指定键参数，则 promise 将被拒绝并返回错误。

`db.insert（index，insertData）：`

这需要一个索引和一些数据，并返回一个 promise。如果操作完成，promise 将被插入的键满足。如果操作失败，或者没有指定键或插入的数据，则 promise 将被拒绝并返回错误。

要利用 promise 和 async/await 语法构建程序，请按照以下步骤进行：

1.  使用`require（'./simple_db'）`导入数据库 API，并将其保存到变量`db`中。

1.  编写一个异步主函数。所有操作都将在这里进行。

1.  创建一个数组来跟踪导致`db`错误的键。将其保存到变量`missingKeys`中。

1.  创建一个 try-catch 块。

在 try 部分内，使用 async/await 和`db.get（）`函数从数据库中查找键`john`。

将值保存到变量`user`中。

记录我们查找的用户的年龄。

在 catch 部分，将键`john`推送到`missingKeys`数组中。

1.  创建第二个 try-catch 块。

在 try 部分中，使用 async/await 和`db.get()`函数查找数据库中的键`sam`

将值保存到变量`user`中。

记录查找到的用户的年龄。

在 catch 部分，将键`sam`推送到`missingKeys`数组中。

1.  创建第三个 try-catch 块。

在 try 部分中，使用 async/await 和`db.get()`函数查找数据库中的您的名字的键。

将值保存到变量`user`中。

记录查找到的用户的年龄。

在 catch 部分，将键推送到`missingKeys`数组中。

在 catch 部分，使用 await 和`db.insert()`将您的`user`对象插入`db`中。

在 catch 部分，在`catch`块内创建一个新的 try-catch 块。在新的 try 部分中，使用 async/await 查找刚刚添加到 db 中的用户。将找到的用户保存到`variable` `user`中。记录查找到的用户的年龄。在 catch 部分，将键推送到`missingKeys`数组中。

1.  在所有的 try-catch 块之外，在主函数的末尾，返回`missingKeys`数组。

1.  调用`main`函数，并为返回的承诺附加一个`then()`和`catch()`处理程序。

1.  `then()`处理程序应传递一个记录承诺解析值的函数。

1.  `catch()`处理程序应传递一个记录错误消息字段的函数。

**代码**：

##### **index.js**

```php
const db = require( './simple_db' );
async function main() {
 const missingKeys = [];
 try { const user = await db.get( 'john' ); } 
 catch ( err ) { missingKeys.push( 'john' ); }
 try { const user = await db.get( 'sam' ); }
 catch ( err ) { missingKeys.push( 'sam' ); }
 try { const user = await db.get( 'zach' ); }
 catch ( err ) {
   missingKeys.push( 'zach' );
   await db.insert('zach', { first: 'zach', last: 'smith', age: 25 });
   try { const user = await db.get( 'zach' ); }
   catch ( err ) { missingKeys.push( 'zach' ); }
 }
 return missingKeys;
}
main().then( console.log ).catch( console.log );
```

###### 片段 2.43：使用 async/await

https://bit.ly/2FvhPo2

**结果**：

![图 2.12：显示的姓名和年龄](img/Figure_2.12.jpg)

###### 图 2.12：显示的姓名和年龄

您已成功实现了文件跟踪命令并浏览了存储库的历史记录。

## 第三章：DOM 操作和事件处理

### 活动 3 - 实现 jQuery

您想要制作一个控制家中智能 LED 灯系统的 Web 应用程序。您有三个 LED 灯，可以单独打开或关闭，或者一起切换。您必须构建一个简单的 HTML 和 jQuery 界面，显示灯的开启状态。它还必须有控制灯的按钮。

要利用 jQuery 构建一个功能应用程序，请按照以下步骤进行：

1.  为该活动创建一个目录，并在该目录中，在命令提示符中运行`npm run init`以设置`package.json`。

1.  运行`npm install jquery -s`以安装 jQuery。

1.  为该活动创建一个 HTML 文件，并给 HTML 块添加一个 body。

1.  添加一个样式块。

1.  添加一个 div 来容纳所有的按钮和灯。

1.  添加一个带有`jQuery`文件源的`script`标签。

```php
<script src="./node_modules/jquery/dist/jquery.js"></script>
```

1.  添加一个`script`标签来保存主 JavaScript 代码。

1.  在 CSS 样式表中为`light`类添加以下设置：

宽度和高度：`100px`

背景颜色：`白色`

1.  在 div 中添加一个切换按钮，使用`id=toggle`。

1.  添加一个 div 来容纳带有 id`lights`的灯。

1.  在这个 div 内添加三个 div。

#### 注意

每个 div 应该有一个带有`light`类的 div 和一个带有`lightButton`类的按钮。

1.  在代码脚本中，设置一个函数在 DOM 加载时运行：

`$(() => { ... })`

1.  选择所有`lightButton`类按钮，并添加一个点击处理程序，执行以下操作：

停止事件传播并选择元素目标，并通过遍历 DOM 获取`light` div。

检查`lit`属性。如果已点亮，则取消`lit`属性。否则，使用`jQuery.attr()`设置它

将`background-color css`样式更改为反映`lit`属性的`jQuery.css()`。

1.  通过 ID 选择`toggle`按钮，并添加一个点击处理程序，执行以下操作：

停止事件传播并通过 CSS 类选择所有的灯按钮，并在它们上触发点击事件：

**代码**：

##### **activity.html**

以下是精简的代码。完整的解决方案可以在`activities/activity3/index.html`中找到。

```php
$( '.lightButton' ).on( 'click', e => {
  e.stopPropagation();
  const element = $( e.target ).prev();
  if ( element.attr( 'lit' ) !== 'true' ) {
    element.attr( 'lit', 'true' );
    element.css( 'background-color', 'black' );
  } else {
    element.attr( 'lit', 'false' );
    element.css( 'background-color', 'white' );
  }
} );
$( '#toggle' ).on( 'click', e => {
  e.stopPropagation();
  $( '.lightButton' ).trigger( 'click' );
} );
```

###### 片段 3.32：jQuery 函数应用

https://bit.ly/2VV9DlB

**结果**：

![图 3.14：在每个 div 后添加按钮](img/Figure_3.14.jpg)

###### 图 3.14：在每个 div 后添加按钮

![图 3.15：添加切换按钮](img/Figure_3.21.jpg)

###### 图 3.15：添加切换按钮

您已成功利用 jQuery 构建了一个功能应用程序。

## 第四章：测试 JavaScript

### Activity 4 – 利用测试环境

您的任务是将斐波那契数列测试代码升级为使用 Mocha 测试框架。将您为 Activity 1 创建的斐波那契数列代码和测试代码升级为使用 Mocha 测试框架来测试代码。您应该为 n=0 条件编写测试，然后实现 n=0 条件，然后为 n=1 条件编写测试并实现，然后为 n=2 条件编写测试并实现，最后为 n=5、n=7 和 n=9 条件做同样的操作。如果 `it()` 测试通过，调用 `done` 回调而不带参数。否则，使用错误或其他真值调用 `test done` 回调。

要利用 Mocha 测试框架编写和运行测试，请按照以下步骤进行：

1.  使用 `npm run init` 设置项目目录。使用 `npm install mocha -s -g` 全局安装 mocha。

1.  创建 `index.js` 来保存代码，`test.js` 保存测试。

1.  将测试脚本添加到 `package.json` 的 `scripts` 字段中。测试应该调用 `mocha` 模块并传入 `test.js` 文件。

1.  将递归斐波那契数列代码添加到 `index.js`。您可以使用 `Exercise 24` 中构建的代码。

1.  导出函数与 `module.exports = { fibonacci }.`

1.  使用以下命令将斐波那契函数导入测试文件：`const { fibonacci } = require( './index.js' )`;

1.  为测试编写一个 `describe` 块。传入字符串 `fibonacci` 和一个回调函数

1.  通过手工计算每个斐波那契数列中的预期值（您也可以在 Google 上查找该数列，以避免手工计算太多数学）。

1.  对于每个测试条件（n=0、n=1、n=2、n=5、n=7 和 n=9），执行以下操作：

使用 `it()` 函数创建一个 mocha 测试，并将测试描述作为第一个参数传入。

将回调作为第二个参数。回调函数应该接受一个参数 `done`。

在回调函数中调用斐波那契数列，并使用不等于比较将其结果与预期值进行比较。

调用 `done()` 函数并传入测试比较结果。

如果测试失败，返回 `done( error )`。否则，返回 `done(null)` 或 `done(false)`。

1.  使用 `npm run test` 从命令行运行测试。

**代码：**

##### **test.js**

```php
'use strict';
const { fibonacci } = require( './index.js' );
describe( 'fibonacci', () => {
 it( 'n=0 should equal 0', ( done ) => {
   done( fibonacci( 0 ) !== 0 );
 } );
 it( 'n=1 should equal 1', ( done ) => {
   done( fibonacci( 1 ) !== 1 );
 } );
 it( 'n=2 should equal 1', ( done ) => {
   done( fibonacci( 2 ) !== 1 );
 } );
 it( 'n=5 should equal 5', ( done ) => {
   done( fibonacci( 5 ) !== 5 );
 } );
 it( 'n=7 should equal 13, ( done ) => {
   done( fibonacci( 7 ) !== 13 );
 } );
 it( 'n=9 should equal 34, ( done ) => {
   done( fibonacci( 9 ) !== 34 );
 } );
} );
```

###### Snippet 4.9：利用测试环境

https://bit.ly/2CcDpJE

查看下面的输出截图：

![图 4.8：显示斐波那契数列](img/Figure_4.71.jpg)

###### 图 4.8：显示斐波那契数列

**结果**

您已成功利用 Mocha 测试框架编写和运行测试。

## 第五章：函数式编程

### Activity 1 – 递归不可变性

您正在使用 JavaScript 构建一个应用程序，并且您的团队被告知出于安全原因不能使用任何第三方库。现在，您必须为该应用程序使用函数式编程（FP）原则，并且您需要一个算法来创建不可变的对象和数组。创建一个递归函数，使用 `Object.freeze()` 强制对象和数组在所有嵌套级别上的不可变性。为简单起见，您可以假设对象中没有嵌套的 null 或类。在 `activities/activity5/activity-test.js` 中编写您的函数。该文件包含测试您的实现的代码。

为了演示在对象中强制不可变性，请按照以下步骤进行：

1.  打开 `activities/activity5/activity-test.js` 中的活动测试文件。

1.  创建一个名为 `immutable` 的函数，它接受一个参数 `data`。

1.  检查 `data` 是否不是 `object` 类型。如果不是，则返回。

1.  冻结 `data` 对象。您不需要冻结非对象。

1.  使用 `object.values` 和 `forEach()` 循环遍历对象值。对每个值递归调用不可变函数。

1.  运行测试文件中包含的代码。如果有任何测试失败，修复错误并重新运行测试。

**代码：**

##### **activity-solution.js**

```php
function immutable( data ) {
 if ( typeof data !== 'object' ) {
   return;
 }
 Object.freeze( data );
 Object.values( data ).forEach( immutable );
}
```

###### 片段 5.11：递归不可变性

https://bit.ly/2H56ah1

查看下面的输出截图：

![图 5.7：通过测试的输出显示](img/Figure_5.11.jpg)

###### 图 5.7：通过测试的输出显示

**结果：**

您已成功地展示了在对象中强制使用不可变性。

## 第六章：JavaScript 生态系统

### 活动 6 - 使用 React 构建前端

从练习 35 中负责笔记应用的前端团队意外退出。您必须使用 React 构建此应用程序的前端。您的前端应该有两个视图：一个 Home 视图和一个`Edit`视图。为每个视图创建一个 React 组件。`Home`视图应该有一个按钮，可以将视图切换到 Edit 视图。Edit 视图应该有一个按钮，可以切换回 Home 视图，一个包含笔记文本的文本输入，一个调用 API 加载路由的加载按钮，以及一个调用 API 保存路由的保存按钮。已提供一个 Node.js 服务器。在`activities/activity6/activity/src/index.js`中编写您的 React 代码。当您准备测试您的代码时，在启动服务器之前运行构建脚本（在`package.json`中定义）。您可以参考练习 35 中的`index.html`文件，了解如何调用 API 路由的提示。

要构建一个可工作的 React 前端并将其与 Node.js express 服务器集成，按照以下步骤进行：

1.  开始在"activity/activity6/activity"中工作。运行`npm install`来安装所需的依赖项。

1.  在 src/index.js 中，创建名为`Home`和`Editor`的 React 组件。

1.  向 App react 组件添加一个构造函数。在构造函数中：

接受`props`变量。调用`super`并将`props`传递给`super`。

在`this`作用域中设置`state`对象。它必须具有一个名为`view`且值为`home`的属性。

1.  向应用程序添加一个`changeView`方法。

`changeView`方法应该接受一个名为`view`的参数。

使用`setState`更新状态，并将`state`的`view`属性设置为提供的参数`view`。

在构造函数中，添加一行代码，将`this.changeView`设置为`this.changeView.bind(this)`。

1.  在 App 的`render`函数中，根据`this.state.view`的值创建一个条件渲染，执行以下操作：

如果`state.view`等于`home`，则显示 Home 视图。否则，显示编辑器视图。

将`changeView`函数作为参数传递给两个视图，即`<Editor changeView={this.changeView}/>`和`<Home changeView={this.changeView}/>`.

1.  在`Home`组件中，添加一个`goEdit()`函数，调用通过`props`传入的`changeView`函数（`this.props.changeView`），并将字符串`editor`传递给`changeView`函数。

1.  在`Home`组件中创建`render`函数：

返回一个 JSX 表达式。

在返回的 JSX 表达式中，添加一个包含应用程序`Note Editor App`标题的`div`，并添加一个按钮，将视图更改为`Edit`视图。

按钮应在单击时调用`goEdit`函数。确保将`this`状态正确绑定到`goEdit`函数。

1.  向`Editor`组件添加一个构造函数：

接受`props`参数

调用`super`并将`props`变量传递给它。

在`this`作用域中将`state`变量设置为`{value: ‘’}`对象。

1.  在`Editor`中添加一个名为`handleChage`的函数：

接受一个名为`e`的参数，表示事件对象。

使用`setState`更新状态，将状态属性`value`设置为事件目标的值：

```php
this.setState( { value: e.target.value } );
```

1.  在`Editor`中创建一个名为`save`的函数，向 API 的 save 路由发出 HTTP 请求。

创建一个新的 XHR 请求，并将其保存到`xhttp`变量中。

将`xhttp`属性`onreadystatechange`设置为一个函数，检查`this.readyState`是否等于`4`。如果不是，则返回。还要检查`this.status`是否等于`200`。如果不是，则抛出错误。

通过在`xhttp`上调用`open`函数打开`xhr`请求。传入参数`POST`，`/save`和`true`。

通过在`xhttp`对象上调用`setRequestHeader`将请求头`Content-Type`设置为`application/json;charset=UTF-8`。传入指定的值。

通过`xhttp.send`发送文本输入的 JSON 数据。

必须发送的值存储在`this.state`中。在发送之前对值进行字符串化。

1.  在`Editor`中创建一个`load`函数，向 API 加载路由发出 HTTP 请求。

创建一个新的 XHR 请求并将其保存到`xhttp`变量中。

将`this`范围保存到名为`that`的变量中，以便在`xhr`请求内部使用。

将`xhttp`属性`onreadystatechange`设置为一个函数，检查`this.readyState`是否等于 4。如果不是，则返回。然后检查`this.status`是否等于 200。如果不是，则抛出错误。它在 React 组件的范围上调用`setState`函数，该函数保存在`that`变量中。

传入一个对象，其中键`value`等于请求的解析响应。使用`JSON.parse`函数从`this.response`变量中解析 HTTP 响应值。

通过在`xhttp`变量上调用`open`函数打开 HTTP 请求。传入参数`GET`、`/load`和`true`。

通过在`xhttp`对象上调用`send()`方法发送 XHR 请求。

1.  在`Editor`中创建一个`goHome`函数。

调用通过 React 元素属性对象传入的`changeView`函数（`this.props.changeView()`）。

传入字符串“主页”。

1.  在`Editor`中创建`render`函数。

添加一个按钮，点击后调用包含文本“返回主页”的`goHome`函数。在点击事件上调用`goHome`函数。确保将`this`范围正确绑定到函数。

添加一个包含笔记文本的“文本”输入。文本输入从`state.value`字段加载其值。“文本”字段在更改事件上调用`handleChange`函数。确保将`this`范围正确绑定到`handleChange`函数。

添加一个按钮，从包含文本“加载”的服务器中加载笔记数据。在点击事件上调用`load`函数。确保将`this`范围正确绑定到 load 函数调用。

添加一个按钮，将包含文本“保存”的笔记数据保存到服务器。在点击事件上调用`save`函数。确保将`this`范围正确绑定到 save 函数调用。

确保将`this`状态正确绑定到所有监听器。

1.  准备好后通过以下方式测试代码：

在根项目文件夹中运行`npm run build`以将代码从 JSX 转换。

运行`npm start`以启动服务器。

在服务器启动时加载指定的 URL（`127.0.0.1:PORT`）。

通过单击“编辑”和“返回主页”按钮测试视图更改。

通过在“文本”输入框中输入文本，保存并检查创建的文件夹中的文本文件，测试“保存”功能。

通过在文本文件中输入文本，加载它，并确保“文本”输入中的值与文本文件中的值匹配，测试“加载”功能。

以下提供了一个简化的片段。有关完整解决方案代码，请参阅`activities/activity6/solution/src/index.js`。

##### Index.js

```php
class Editor extends React.Component {
 constructor( props ) { ... }
 handleChange( e ) { ... }
 save() { ... }
 load() { ... }
 goHome() { ... }
 render() {
   return (
     <div>
       <button type="button" onClick={this.goHome.bind( this )}>Back to home</button>
       <input type="text" name="Note Text" value={this.state.value} onChange={this.handleChange.bind( this )}/>
       <button type="button" onClick={this.load.bind( this )}>Load</button>
       <button type="button" onClick={this.save.bind( this )}>Save</button>
     </div>
   );
 }
}
class Home extends React.Component {
 goEdit() { ... }
 render() {
   return (
     <div>
       <h1>Note Editor App</h1>
       <button type="button" onClick={this.goEdit.bind( this )}>Edit Note</button>
     </div>
   );
 }
}
//{…}
```

###### 片段 6.42：React 组件

https://bit.ly/2RzxKI2

**结果：**

查看这里的输出截图：

![图 6.13：编辑视图](img/Figure_6.13.jpg)

###### 图 6.13：编辑视图

![图 6.14：服务器视图](img/Figure_6.21.jpg)

###### 图 6.14：服务器视图

![图 6.15：运行服务器以测试代码](img/Figure_6.31.jpg)

###### 图 6.15：运行服务器以测试代码

您已成功构建了一个可工作的 React 前端，并将其与 Node.js express 服务器集成。
