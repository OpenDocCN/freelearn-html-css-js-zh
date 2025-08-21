# 第十章：Nest.js 中的路由和请求处理

Nest.js 中的路由和请求处理由控制器层处理。Nest.js 将请求路由到定义在控制器类内部的处理程序方法。在控制器的方法中添加路由装饰器，如`@Get()`，告诉 Nest.js 为此路由路径创建一个端点，并将每个相应的请求路由到此处理程序。

在本章中，我们将使用我们的博客应用程序中的 EntryController 作为一些示例的基础，来介绍 Nest.js 中路由和请求处理的各个方面。我们将看看您可以使用的不同方法来编写请求处理程序，因此并非所有示例都与我们的博客应用程序中的代码匹配。

# 请求处理程序

在 EntryController 中注册的`/entries`路由的基本 GET 请求处理程序可能如下所示：

```js
import { Controller, Get } from '@nestjs/common';

@Controller('entries')
export class EntryController {
    @Get()
    index(): Entry[] {
        const entries: Entry[] = this.entriesService.findAll();
        return entries;
    }

```

`@Controller('entries')`装饰器告诉 Nest.js 在类中注册的所有路由添加一个`entries`前缀。此前缀是可选的。设置此路由的等效方式如下：

```js
import { Controller, Get } from '@nestjs/common';

@Controller()
export class EntryController {
    @Get('entries')
    index(): Entry[] {
        const entries: Entry[] = this.entriesService.findAll();
        return entries;
    }

```

在这里，我们不在`@Controller()`装饰器中指定前缀，而是在`@Get('entries')`装饰器中使用完整的路由路径。

在这两种情况下，Nest.js 将所有 GET 请求路由到此控制器中的`index()`方法。从处理程序返回的条目数组将**自动**序列化为 JSON 并作为响应主体发送，并且响应状态码将为 200。这是 Nest.js 中生成响应的标准方法。

Nest.js 还提供了`@Put()`、`@Delete()`、`@Patch()`、`@Options()`和`@Head()`装饰器，用于创建其他 HTTP 方法的处理程序。`@All()`装饰器告诉 Nest.js 将给定路由路径的所有 HTTP 方法路由到处理程序。

# 生成响应

Nest.js 提供了两种生成响应的方法。

## 标准方法

使用自 Nest.js 4 以来可用的标准和推荐方法，Nest.js 将**自动**将从处理程序方法返回的 JavaScript 对象或数组序列化为 JSON 并将其发送到响应主体中。如果返回一个字符串，Nest.js 将只发送该字符串，而不将其序列化为 JSON。

默认的响应状态码为 200，除了 POST 请求使用 201。可以通过使用`@HttpCode(...)`装饰器轻松地更改处理程序方法的响应代码。例如：

```js
@HttpCode(204)
@Post()
create() {
  // This handler will return a 204 status response
}

```

## Express 方法

在 Nest.js 中生成响应的另一种方法是直接使用响应对象。您可以要求 Nest.js 将响应对象注入到处理程序方法中，使用`@Res()`装饰器。Nest.js 使用[express 响应对象](http://expressjs.com/en/api.html#res)。

您可以使用响应对象重写先前看到的响应处理程序，如下所示。

```js
import { Controller, Get, Res } from '@nestjs/common';
import { Response } from 'express';

@Controller('entries')
export class EntryController {
    @Get()
    index(@Res() res: Response) {
        const entries: Entry[] = this.entriesService.findAll();
        return res.status(HttpStatus.OK).json(entries);
    }
}

```

直接使用 express 响应对象将条目数组序列化为 JSON 并发送 200 状态码响应。

`Response`对象的类型来自 express。在`package.json`中的`devDependencies`中添加`@types/express`包以使用这些类型。

# 路由参数

Nest.js 使得从路由路径接受参数变得容易。为此，您只需在路由的路径中指定路由参数，如下所示。

```js
import { Controller, Get, Param } from '@nestjs/common';

@Controller('entries')
export class EntryController {
    @Get(':entryId')
    show(@Param() params) {
        const entry: Entry = this.entriesService.find(params.entryId);
        return entry;
    }
}

```

上述处理程序方法的路由路径为`/entries/:entryId`，其中`entries`部分来自控制器路由前缀，而由冒号表示的`:entryId`参数。使用`@Param()`装饰器注入 params 对象，其中包含参数值。

或者，您可以使用`@Param()`装饰器注入单个参数值，如下所示指定参数名称。

```js
import { Controller, Get, Param } from '@nestjs/common';

@Controller('entries')
export class EntryController {
    @Get(':entryId')
    show(@Param('entryId') entryId) {
        const entry: Entry = this.entriesService.findOne(entryId);
        return entry;
    }
}

```

# 请求体

要访问请求的主体，请使用`@Body()`装饰器。

```js
import { Body, Controller, Post } from '@nestjs/common';

@Controller('entries')
export class EntryController {
    @Post()
    create(@Body() body: Entry) {
        this.entryService.create(body);
    }
}

```

# 请求对象

要访问客户端请求的详细信息，您可以要求 Nest.js 使用`@Req()`装饰器将请求对象注入到处理程序中。Nest.js 使用[express 请求对象](http://expressjs.com/en/api.html#req)。

例如，

```js
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('entries')
export class EntryController {
    @Get()
    index(@Req() req: Request): Entry[] {
        const entries: Entry[] = this.entriesService.findAll();
        return entries;
    }

```

`Request`对象的类型来自 express。在`package.json`的`devDependencies`中添加`@types/express`包以使用这些类型。

# 异步处理程序

到目前为止，在本章中展示的所有示例都假设处理程序是同步的。在实际应用中，许多处理程序将需要是异步的。

Nest.js 提供了许多方法来编写异步请求处理程序。

## 异步/等待

Nest.js 支持异步请求处理程序函数。

在我们的示例应用程序中，`entriesService.findAll()`函数实际上返回一个`Promise<Entry[]>`。使用 async 和 await，这个函数可以这样写。

```js
import { Controller, Get } from '@nestjs/common';

@Controller('entries')
export class EntryController {
    @Get()
    async index(): Promise<Entry[]> {
        const entries: Entry[] = await this.entryService.findAll();
        return entries;
    }

```

异步函数必须返回 promises，但是在现代 JavaScript 中使用 async/await 模式，处理程序函数可以看起来是同步的。接下来，我们将解决返回的 promise 并生成响应。

## Promise

同样，您也可以直接从处理程序函数返回一个 promise，而不使用 async/await。

```js
import { Controller, Get } from '@nestjs/common';

@Controller('entries')
export class EntryController {
    @Get()
    index(): Promise<Entry[]> {
        const entriesPromise: Promise<Entry[]> = this.entryService.findAll();
        return entriesPromise;
    }

```

## Observables

Nest.js 请求处理程序也可以返回 RxJS Observables。

例如，如果`entryService.findAll()`返回的是 Observable 而不是 Promise，那么以下内容将是完全有效的。

```js
import { Controller, Get } from '@nestjs/common';

@Controller('entries')
export class EntryController {
    @Get()
    index(): Observable<Entry[]> {
        const entriesPromise: Observable<Entry[]> = this.entryService.findAll();
        return entriesPromise;
    }

```

没有推荐的方法来编写异步请求处理程序。使用您最熟悉的任何方法。

# 错误响应

Nest.js 有一个异常层，负责捕获来自请求处理程序的未处理异常，并向客户端返回适当的响应。

全局异常过滤器处理从请求处理程序抛出的所有异常。

## HttpException

如果从请求处理程序抛出的异常是`HttpException`，全局异常过滤器将把它转换为 JSON 响应。

例如，您可以从`create()`处理程序函数中抛出`HttpException`，如果 body 无效则如此。

```js
import { Body, Controller, HttpException, HttpStatus, Post } from '@nestjs/common';

@Controller('entries')
export class EntryController {
    @Post()
    create(@Body() entry: Entry) {
        if (!entry) throw new HttpException('Bad request', HttpStatus.BAD_REQUEST);
        this.entryService.create(entry);
    }
}

```

如果抛出此异常，响应将如下所示：

```js
{
    "statusCode": 400,
    "message": "Bad request"
}

```

您还可以通过将对象传递给`HttpException`构造函数来完全覆盖响应体，如下所示。

```js
import { Body, Controller, HttpException, HttpStatus, Post } from '@nestjs/common';

@Controller('entries')
export class EntryController {
    @Post()
    create(@Body() entry: Entry) {
        if (!entry) throw new HttpException({ status: HttpStatus.BAD_REQUEST, error: 'Entry required' });
        this.entryService.create(entry);
    }
}

```

如果抛出此异常，响应将如下所示：

```js
{
    "statusCode": 400,
    "error": "Entry required"
}

```

## 未识别的异常

如果异常未被识别，意味着它不是`HttpException`或继承自`HttpException`的类，则客户端将收到下面的 JSON 响应。

```js
{
    "statusCode": 500,
    "message": "Internal server error"
}

```

# 总结

借助于我们示例博客应用程序中的 EntryController，本章涵盖了 Nest.js 中的路由和请求处理的方面。您现在应该了解各种方法，可以用来编写请求处理程序。

在下一章中，我们将详细介绍 OpenAPI 规范，这是一个 JSON 模式，可用于构建一组 restful API 的 JSON 或 YAML 定义。
