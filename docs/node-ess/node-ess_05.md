# 第五章：配置

随着我们的应用程序变得越来越大，我们开始失去对配置做什么的视野；我们可能也会陷入这样一种情况：我们的代码在 12 个不同的地方运行，每个地方都需要一些代码来做一些其他事情，例如连接到不同的数据库。然后，对于这 12 个环境，我们有三个版本：生产、暂存和开发。突然间，情况变得非常复杂。这就是为什么我们需要能够从更高层次配置我们的代码，这样我们就不会在这个过程中破坏任何东西。

# JSON 文件

有几种方法可以配置我们的应用程序。我们将首先看一种简单的 JSON 文件。

如果我们查看默认支持的扩展名，我们可以看到我们可以将 JSON 直接导入到我们的代码中，如下所示：

```js
[~/examples/example-16]$ node
> require.extensions
{ '.js': [Function],
'.json': [Function],
'.node': [Function: dlopen] }

```

让我们创建一个简单的服务器，使用配置文件而不是硬编码文件：

首先，我们必须创建配置文件：

```js
{
    "host": "localhost",
    "port": 8000
}
```

有了这个，我们现在可以创建我们的服务器了：

```js
var Config = require('./config.json'),
    Http = require('http');
Http.createServer(function(request, response) {

}).listen(Config.port, Config.host, function() {
    console.log('Listening on port', Config.port, 'and host', Config.host);
});
```

现在，我们只需要更改`config`文件，而不是更改代码来更改服务器运行的端口。

但是我们的`config`文件有点太通用了；我们不知道主机或端口是什么，以及它们与什么相关。

在配置时，键需要更具体，这样我们才知道它们被用于什么，除非应用程序直接给出了上下文。例如，如果应用程序只提供纯静态内容，那么使用更通用的键可能是可以接受的。

为了使这些配置键更具体，我们可以将它们全部包装在一个服务器对象中：

```js
{
    "server": {
        "host": "localhost",
        "port": 8000
    }
}
```

现在，为了了解服务器的端口，我们需要使用以下代码：

```js
Config.server.port
```

一个可能有用的例子是连接到数据库的服务器，因为它们可以接受端口和主机作为参数：

```js
{
    "server": {
        "host": "localhost",
        "port": 8000
    },
    "database": {
        "host": "db1.example.com",
        "port": 27017
    }
}
```

# 环境变量

我们可以通过使用环境变量来配置我们的应用程序的另一种方式。

这些可以由你运行应用程序的环境或使用的命令来定义。

在 Node.js 中，你可以使用`process.env`来访问环境变量。使用`env`时，你不希望过多地污染这个空间，所以最好是给键加上与你自己相关的前缀——你的程序或公司。例如，`Config.server.host`变成了`process.env.NAME_SERVER_HOST`；原因是我们可以清楚地看到与你的程序相关的内容和不相关的内容。

使用环境变量来配置我们的服务器，我们的代码将如下所示：

```js
var Http = require('http'),
    server_port,
    server_host;

server_port = parseInt(process.env.FOO_SERVER_PORT, 10);
server_host = process.env.FOO_SERVER_HOST;

Http.createServer(function(request, response) {

}).listen(server_port, server_host, function() {
    console.log('Listening on port', server_port, 'and host', server_host);
});
```

为了使用我们的变量运行这段代码，我们将使用：

```js
[~/examples/example-17]$ FOO_SERVER_PORT=8001 \
FOO_SERVER_HOST=localhost node server.js
Listening on port 8001 and host localhost

```

你可能注意到我不得不对`FOO_SERVER_PORT`使用`parseInt`；这是因为以这种方式传递的所有变量本质上都是字符串。我们可以通过执行`typeof process.env.FOO_ENV`来看到这一点：

```js
[~/examples/example-17]$ FOO_ENV=1234 node
> typeof process.env.FOO_ENV
'string'
> typeof parseInt( process.env.FOO_ENV, 10 )
'number'

```

尽管这种配置非常简单易于创建和使用，但可能不是最佳方法，因为如果变量很多，很难跟踪它们，并且它们很容易被遗漏。

# 参数

配置可以通过作为进程启动时传递给 Node.js 的参数来完成，你可以使用`process.argv`来访问这些参数，`argv`代表参数向量。

`process.argv`返回的数组始终会在索引`0`处有一个`node`。例如，如果你运行`node server.js`，那么`process.argv`的值将是`[ 'node', '/example/server.js' ]`。

如果你向 Node.js 传递一个参数，它将被添加到`process.argv`的末尾。

如果你运行`node server.js --port=8001`，`process.argv`将包含`[ 'node', '/example/server.js', '--port=8001' ]`，非常简单，对吧？

尽管我们可以有所有这些配置，但我们应该始终记住，配置可以被简单地排除，即使这种情况发生，我们仍希望我们的应用程序能够运行。通常情况下，当你有配置选项时，你应该提供默认的硬编码值作为备份。

密码和私钥等参数永远不应该有默认值，但通常标准的链接和选项应该有默认值。在 Node.js 中很容易给出默认值，你只需要使用 `OR` 运算符。

```js
value = value || 'default';
```

基本上，这样做的作用是检查值是否为`falsy`；如果是，则使用默认值。你需要注意那些你知道可能是`falsy`的值，布尔值和数字肯定属于这个范畴。

在这些情况下，你可以使用一个检查 `null` 值的 `if` 语句，如下所示：

```js
if ( value == null ) value = 1
```

# 总结

配置就介绍到这里。在本章中，你学会了三种创建动态应用程序的方法。我们学到了应该以一种可以识别值的变化和它们对应用程序的影响的方式命名配置键。我们还学会了如何使用环境变量和 `argv` 将简单参数传递给我们的应用程序。

有了这些信息，我们可以继续在下一章中连接和利用数据库。

为 Bentham Chang 准备，Safari ID bentham@gmail.com 用户编号：2843974 © 2015 Safari Books Online，LLC。此下载文件仅供个人使用，并受到服务条款的约束。任何其他使用都需要版权所有者的事先书面同意。未经授权的使用、复制和/或分发严格禁止并违反适用法律。保留所有权利。
