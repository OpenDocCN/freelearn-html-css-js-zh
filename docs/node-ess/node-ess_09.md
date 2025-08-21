# 第九章：单元测试

我们已经走了这么远，但还没有做任何测试！这不太好，是吗？通常，如果不是总是，测试是软件开发中的一个主要关注点。在本章中，我们将介绍 Node 的单元测试概念。

Node.js 有许多测试框架，在本章中我们将介绍 Mocha。

# 安装 mocha

为了确保`mocha`在所有地方都安装了，我们需要全局安装它。这可以使用`npm install`的`-g`标志来完成：

```js
[~/examples/example-24]$ npm install -g mocha

```

现在，我们可以通过终端控制台使用 Mocha。

通常，我们将所有测试代码放在项目的`test`子目录中。我们只需要运行`mocha`，假设我们首先编写了一些测试，就可以运行我们的代码。

与许多（如果不是所有）单元测试框架一样，Mocha 使用断言来确保测试正确运行。如果抛出错误并且没有处理，那么测试被认为是失败的。断言库的作用是在传递意外值时抛出错误，因此这很有效。

Node.js 提供了一个简单的断言模块，让我们来看一下：

```js
[~/examples/example-24]$ node
> assert = require( 'assert' )
> expected = 1
> actual = 1
> assert.equal( actual, expected )
> actual = 1
> assert.equal( actual, expected )
AssertionError: 2 == 1

```

正如我们所看到的，如果断言不通过，就会抛出错误。但是，提供的错误消息并不是很方便；为了解决这个问题，我们也可以传递错误消息：

```js
> assert.equal( actual, expected, 'Expected 1' )
AssertionError: Expected 1

```

有了这个，我们就可以创建一个测试。

Mocha 提供了许多创建测试的方法，这些方法称为*接口*，默认的称为 BDD。

您可以在[`mochajs.org/#interfaces`](http://mochajs.org/#interfaces)上查看所有接口。

**BDD**（行为驱动开发）接口可以与 Gherkin 进行比较，其中我们指定一个功能和一组场景。它提供了帮助定义这些集合的方法，`describe`或`context`用于定义一个功能，`it`或`specify`函数用于定义一个场景。

例如，如果我们有一个函数，用于连接某人的名和姓，测试可能看起来像下面这样：

```js
var GetFullName = require( '../lib/get-full-name' ),
    assert = require( 'assert' );

describe( 'Fetch full name', function( ) {

    it( 'should return both a first and last name', function( ) {
        var result = GetFullName( { first: 'Node', last: 'JS' } )
        assert.equal( result, 'Node JS' );
    })
})
```

我们还可以为此添加一些其他测试；例如，如果没有传递对象，则会引发错误：

```js
it( 'should throw an error when an object was not passed', function( ) {
    assert.throws(
        function( ) {
            GetFullName( null );
        },
        /Object expected/
    )
})
```

您可以在[`mochajs.org/`](http://mochajs.org/)上探索更多 mocha 特定的功能。

# Chai

除了许多测试框架之外，还有许多断言框架，其中之一称为**Chai**。完整的文档可以在[`chaijs.com/`](http://chaijs.com/)找到。

不要使用 Node.js 提供的内置断言模块，我们可能想要使用 Chai 等模块来扩展我们的可能性。

Chai 有三组接口，should，expect 和 assert。在本章中，我们将介绍 expect。

使用 expect 时，您使用自然语言描述您想要的内容；例如，如果您想要某物存在，可以说`expect( x ).to.exist`而不是`assert( !!x )`：

```js
var Expect = require( 'chai' ).expect
var Assert = require( 'assert' )

var value = 1

Expect( value ).to.exist
assert( !!value )
```

使用自然语言使得阅读您的测试变得更加清晰。

这种语言可以链接在一起；我们有`to`，`be`，`been`，`is`，`that`，`which`，`and`，`has`，`have`，`with`，`at`，`of`和`same`，这些可以帮助我们构建句子，比如：

```js
Expect( value ).to.be.ok.and.to.equal( 1 )
```

但是，这些词只是用于可靠性，它们不会修改结果。还有很多其他词可以用来断言事物，比如`not`，`exists`，`ok`等等。您可以在[`chaijs.com/api/bdd/`](http://chaijs.com/api/bdd/)上查看它们。

chai 的一些用法示例包括：

```js
Expect( true ).to.be.ok
Expect( false ).to.not.be.ok
Expect( 1 ).to.exists
Expect( [ ] ).to.be.empty
Expect( 'hi' ).to.equal( 'hi' )
Expect( 4 ).to.be.below( 5 )
Expect( 5 ).to.be.above( 4 )
Expect( function() {} ).to.be.instanceOf( Function )
```

# 存根方法

*如果它看起来像一只鸭子，游泳像一只鸭子，嘎嘎叫像一只鸭子，那么它可能就是一只鸭子*。

在编写测试时，您只想测试代码的“单元”。通常这将是一个方法，为其提供一些输入，并期望得到某种输出，或者如果它是一个`void`函数，则期望不返回任何内容。

有了这个想法，你必须把你的应用程序看作处于沙盒状态，不能与外部世界交流。例如，它可能无法与数据库通信或进行任何外部请求。如果你要（通常应该）实现持续集成和部署，这种假设是很好的。这也意味着在测试的机器上除了 Node.js 和测试框架之外，没有外部要求，这些可能只是你的软件包的一部分。

除非你要测试的方法非常简单，没有任何外部依赖，否则你可能会想要`mock`你知道它将执行的方法。一个很好的模块就是 Sinon.js；它允许你创建`stubs`和`spies`，以确保正确的数据从其他方法返回，并确保它们首先被调用。

`sinon`提供了许多辅助功能，如前所述，其中之一就是**spy**。spy 主要用于包装一个函数，以查看其输入和输出。一旦 spy 被应用到一个函数上，对外界来说，它的行为完全相同。

```js
var Sinon = require( 'sinon' );

var returnOriginal = function( value ) {
    return value;
}

var spy = Sinon.spy( returnOriginal );

result = spy( 1 );
console.log( result ); // Logs 1
```

我们可以使用 spy 来检查函数是否被调用：

```js
assert( spy.called )
```

或者每次调用时传递了什么参数：

```js
assert.equal( spy.args[ 0 ][ 0 ], 1 )
```

如果我们用一个对象和一个要替换的方法提供了`spy`，那么在完成后我们可以恢复原始的方法。我们通常会在测试的`tear down`中这样做：

```js
var object = {
    spyOnMe: function( value ) {
        return value;
    }
}
Sinon.spy( object, 'spyOnMe' )

var result = object.spyOnMe( 1 )
assert( result.called )
assert.equal( result.args[ 0 ][ 0 ], 1 )

object.spyOnMe.restore( )
```

我们还有一个`stub`函数，它继承了`spy`的所有功能，但是完全替换了原始函数，而不是调用它。

这样我们就可以定义行为，例如，它返回什么：

```js
var stub = Sinon.stub( ).returns( 42 )
console.log( stub( ) ) // logs 42
```

我们还可以为一组传递的参数定义返回值：

```js
var stub = Sinon.stub( )
stub.withArgs( 1, 2, 3 ).returns( 42 )
stub.withArgs( 3, 4, 5 ).returns( 43 )

console.log( stub( 1, 2, 3 ) ) // logs 42
console.log( stub( 3, 4, 5 ) ) // logs 43
```

假设我们有这组方法：

```js
function Users( ) {

}
Users.prototype.getUser = function( id ) {
    return Database.findUser( id );
}
Users.prototype.getNameForUser = function( id ) {
    var user = this.getUser( id );
    return user.name;
}
module.exports = Users
```

现在，我们只关心用户被返回的情况，因为如果找不到用户，`getUser`函数将抛出错误。知道这一点，我们只想测试当找到用户时它返回他们的名字。

这是一个完美的例子，我们想要`stub`一个方法的时候：

```js
var Sinon = require( 'sinon' );
var Users = require( '../lib/users' );
var Assert = require( 'assert' );

it( 'should return a users name', function( ) {

    var name = 'NodeJS';
    var user = { name: name };

    var stub = Sinon.stub( ).returns( user );

    var users = new Users( );
    users.getUser = stub;

    var result = users.getNameForUser( 1 );

    assert.equal( result, name, 'Name not returned' );
});
```

我们可以通过作用域传递函数，而不是替换函数，用传递的对象替换 this；两种方式都可以。

```js
var result = users.getNameForUser.call(
    {
        getUser: stub
    },
    1
);
```

# 摘要

我们现在可以轻松创建一个 Node.js 应用所需的一切。测试只是其中一个对于任何成功的软件都是必不可少的事情。我们介绍了使用 mocha 作为测试框架和 chai 作为断言框架。

在下一章中，我们将介绍如何在 Node.js 中使用另一种语言，CoffeeScript！

为 Bentham Chang 准备，Safari ID bentham@gmail.com 用户编号：2843974 © 2015 Safari Books Online，LLC。此下载文件仅供个人使用，并受到服务条款的约束。任何其他用途均需版权所有者的事先书面同意。未经授权的使用、复制和/或分发严格禁止并违反适用法律。保留所有权利。
