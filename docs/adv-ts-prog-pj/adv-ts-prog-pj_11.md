# 第十一章：评估

# 第一章

1.  使用联合类型，我们可以编写一个接受`FahrenheitToCelsius`类或`CelsiusToFahrenheit`类的方法：

```ts
class Converter {
    Convert(temperature : number, converter : FahrenheitToCelsius | CelsiusToFahrenheit) : number {
        return converter.Convert(temperature);
    }
}

let converter = new Converter();
console.log(converter.Convert(32, new CelsiusToFahrenheit()));
```

1.  要接受键/值对，我们需要使用映射。将我们的记录添加到其中看起来像这样：

```ts
class Commands {
    private commands = new Map<string, Command>();
    public Add(...commands : Command[]) {
        commands.forEach(command => {
            this.Add(command);
        })
    }
    public Add(command : Command) {
        this.commands.set(command.Name, command);
    }
}

let command = new Commands();
command.Add(new Command("Command1", new Function()), new Command("Command2", new Function()));
```

我们实际上在这里添加了两种方法。如果我们想一次添加多个命令，我们可以使用 REST 参数来接受命令数组。

1.  我们可以使用装饰器来在调用我们的`Add`方法时自动记录。例如，我们的`log`方法可能如下所示：

```ts
function Log(target : any, propertyKey : string | symbol, descriptor : PropertyDescriptor) {
    let originalMethod = descriptor.value;
    descriptor.value = function() {
        console.log(`Added a command`);
        originalMethod.apply(this, arguments);
    }
    return descriptor;
}
```

我们只会将这个添加到以下的`Add`方法中，因为接受 REST 参数的`Add`方法无论如何都会调用这个方法：

```ts
@Log
public Add(command : Command) {
    this.commands.set(command.Name, command);
}
```

不要忘记我们使用`@`符号来表示这是一个装饰器。

1.  要添加一个具有相等大小的六个中等列的行，我们使用六个`div`语句，并将类设置为`col-md-2`，就像这样：

```ts
<div class="row">
  <div class="col-md-2">
  </div>
  <div class="col-md-2">
  </div>
  <div class="col-md-2">
  </div>
  <div class="col-md-2">
  </div>
  <div class="col-md-2">
  </div>
  <div class="col-md-2">
  </div>
</div>
```

请记住，根据我们在 Bootstrap 中的讨论，一行中的列数应该等于 12。

# 第三章

1.  React 为我们提供了特殊的文件类型，`.jsx`（用于 JavaScript）或`.tsx`（用于 TypeScript），以创建一个可以*转译*为 JavaScript 的文件，因此 React 将类似 HTML 的元素呈现为 JavaScript。

1.  `class`和`for`都是 JavaScript 中的保留关键字。由于`.tsx`文件似乎在同一个方法中混合了 JavaScript 和 HTML，我们需要别名来指定 CSS 类和`label`关联的控件。React 提供了`className`来指定应该应用于 HTML 元素的类，以及`htmlFor`来指定标签关联的控件。

1.  当我们创建验证器时，我们正在创建可重复使用的代码片段，可以用来执行特定类型的验证；例如，检查字符串是否达到最小长度。由于这些被设计为可重复使用，我们必须将它们与实际应用验证的验证代码分开。

1.  通过用`\d`替换`[0-9]`，我们将`^(?:\\((?:[0-9]{3})\\)|(?:[0-9]{3}))[-. ]?(?:[0-9]{3})[-. ]?(?:[0-9]{4})$`转换为以下表达式：`^(?:\\((?:\d{3})\\)|(?:\d{3}))[-. ]?(?:\d{3})[-. ]?(?:\d{4})$`

1.  使用硬删除，我们从数据库中删除物理记录。使用软删除，我们保留记录，但对其应用一个标记，表示该记录不再处于活动状态。

# 第四章

1.  MEAN 堆栈由四个主要组件组成：

+   **MongoDB**：MongoDB 是一个 NoSQL 数据库，已成为在 Node 中构建数据库支持的事实标准。还有其他数据库选项可用，但 MongoDB 是一个非常受欢迎的选择。

+   **Express**：Express 封装了在 Node 下处理服务器端代码的许多复杂性，并使其更易于使用。例如，如果我们想处理 HTTP 请求，Express 使这变得微不足道，而不是编写等效的 Node 代码。

+   **Angular**：Angular 是一个客户端框架，使得创建强大的 Web 前端更容易。

+   **Node**：Node（或 Node.js）是服务器上应用程序的运行时环境。

1.  我们提供一个前缀使得我们的组件唯一。假设我们有一个组件，我们想要称为`label`；显然，这将与内置的 HTML 标签冲突。为了避免这种冲突，我们的组件选择器将是`atp-label`。由于 HTML 控件从不使用连字符，我们保证不会与现有的控件选择器*冲突*。

1.  要启动我们的 Angular 应用程序，我们在顶层 Angular 文件夹中运行以下命令：

```ts
ng serve --open
```

1.  与我们自己的语言被分解和结构化为单词和标点符号一样，我们可以将视觉元素分解为结构，例如颜色和深度。例如，语言告诉我们颜色的含义，因此，如果我们在应用程序中的一个屏幕上看到一个带有一个颜色的按钮，它应该在我们应用程序的其他屏幕上具有相同的基础用法；我们不会在一个对话框上使用绿色按钮来表示确定，然后在另一个对话框上使用取消。设计语言背后的理念是元素应该是一致的。因此，如果我们将我们的应用程序创建为一个 Material 应用程序，那么对于使用 Gmail 的人来说，它应该是熟悉的（例如）。

1.  我们使用以下命令创建服务：

```ts
ng generate service <<servicename>>
```

这可以缩短为以下内容：

```ts
ng g s <<servicename>>
```

1.  每当请求进入我们的服务器时，我们需要确定如何处理最好的请求，这意味着我们必须将其路由到处理请求的适当功能部分。Express 路由是我们用来实现这一点的机制。

1.  RxJS 实现了观察者模式。这种模式有一个对象（称为**subject**），它跟踪一系列依赖项（称为**observers**），并通知它们*有趣*的行为，例如状态更改。

1.  **CORS**代表**跨域请求共享**。使用 CORS，我们允许*已知*的外部位置访问我们站点上的受限操作。在我们的代码中，由于 Angular 是从与我们的 Web 服务器不同的站点运行的（`localhost:4200`，而不是`localhost:3000`），我们需要启用 CORS 支持来进行发布，否则当我们从 Angular 发出请求时，我们将不会返回任何内容。

# 第五章

1.  GraphQL 并不打算完全取代 REST 客户端。它可以作为一种合作技术，因此它很可能会自己消耗多个 REST API 来生成图。

1.  变异是一种旨在以某种方式更改图中数据的操作。我们可能想要向图中添加新项目，更新项目或删除项目。重要的是要记住，变异只是改变了图 - 如果更改必须持久保存到图从中获取信息的地方，那么图就有责任调用底层服务来进行这些更改。

1.  为了将值传递给子组件，我们需要使用`@Input()`来公开一个字段，以便从父级进行绑定。在我们的代码示例中，我们设置了一个`Todo`项目，如下所示：

```ts
@Input() Todo: ITodoItem;
```

1.  使用 GraphQL，解析器代表了如何将操作转换为数据的指令；它们被组织为与字段的一对一映射。另一方面，模式代表了多个解析器。

1.  要创建一个单例，我们需要做的第一件事是创建一个带有私有构造函数的类。私有构造函数意味着我们可以实例化我们的类的唯一位置是从类本身内部：

```ts
export class Prefill {
  private constructor() {}
}
```

接下来我们需要做的是添加一个字段来保存对类实例的引用，然后提供一个公共静态属性来访问该实例。公共属性将负责实例化类（如果尚未可用），以便我们始终能够访问它：

```ts
private static prefill: Prefill;
public static get Instance(): Prefill {
  return this.prefill || (this.prefill = new this());
}
```

# 第六章

1.  使用`io.emit`，我们可以向所有连接的客户端发送消息。

1.  如果我们想要向特定房间中的所有用户发送消息，我们将使用类似以下的内容，其中我们说我们要向哪个房间发送消息，然后使用`emit`来设置`event`和`message`：

```ts
io.to('room').emit('event', 'message');
```

1.  要将消息发送给除发送方之外的所有用户，我们需要进行广播：

```ts
socket.broadcast.emit('broadcast', 'my message');
```

1.  有一些事件名称，我们不能用作消息，因为它们由于具有对 Socket.IO 具有特殊含义而受到限制。这些是`error`，`connect`，`disconnect`，`disconnecting`，`newListener`，`removeListener`，`ping`和`pong`。

1.  Socket.IO 由许多不同的协作技术组成，其中之一称为 Engine.IO。这提供了底层传输机制。它在连接时采用的第一种连接类型是 HTTP 长轮询，这是一种快速高效的传输机制。在空闲期间，Socket.IO 会尝试确定传输是否可以切换到套接字，如果可以使用套接字，它会无缝地升级传输以使用套接字。对于客户端来说，它们连接迅速，消息可靠，因为 Engine.IO 部分建立连接，即使存在防火墙和负载均衡器。

# 第七章

1.  在`@Component`定义中，我们使用`host`将我们要处理的主机事件映射到相关的 Angular 方法。例如，在我们的`MapViewComponent`中，我们使用以下组件定义将`window load`事件映射到`Loaded`方法：

```ts
@Component({
  selector: 'atp-map-view',
  templateUrl: './map-view.component.html',
  styleUrls: ['./map-view.component.scss'],
  host: {
    '(window:load)' : 'Loaded()'
  }
})
```

1.  纬度和经度是用于确定地球上某个位置的地理术语。纬度告诉我们某物距赤道有多远，赤道为 0；正数表示我们在赤道以北，负数表示我们在赤道以南。经度告诉我们我们距离地球的中心线（按照惯例，通过伦敦的格林威治）有多远。同样，如果我们向东移动，数字是正数，而向西移动意味着数字是负数。

1.  将经度和纬度表示的位置转换为地址的行为称为反向地理编码。

1.  我们使用 Firestore 数据库，这是 Google 的 Firebase 云服务的一部分，用来保存我们的数据。

# 第八章

1.  容器是一个运行实例，它接收运行应用程序所需的各种软件。这是我们的起点；容器是从镜像构建的，您可以自己构建或从中央 Docker 数据库下载。容器可以向其他容器打开，例如主机操作系统，甚至可以使用端口和卷向更广泛的世界打开。容器的一个重要卖点是它易于设置和创建，并且可以快速停止和启动。

1.  当我们启动 Docker 容器时，我们讨论了两种实现方法。第一种方法涉及使用`docker build`和`docker run`的组合来启动服务：

```ts
docker build -t ohanlon/addresses .
docker run -p 17171:3000 -d ohanlon/addresses
```

使用`-d`表示它不会阻塞控制台，因为它会在后台分离并静默运行。这使我们能够一起运行一组这些命令。在下载中，您会找到一个我创建的批处理文件，用于在 Windows 上启动它们。

第二种方法，也是我推荐的方法，使用 Docker 组合。在我们的示例中，我们创建了一个`docker-compose.yml`文件，用于将我们的微服务组合在一起。要运行我们的组合文件，我们需要使用以下命令：

```ts
docker-compose up
```

1.  如果我们使用`docker run`来启动容器，我们可以使用`-p`开关在其中指定端口。以下示例将端口`3000`重新映射到`17171`：

```ts
docker run -p 17171:3000 -d ohanlon/addresses
```

当我们使用 Docker 组合时，我们在`docker-compose.yml`文件中指定端口重映射。

1.  Swagger 为我们提供了许多有用的功能。我们可以用它来创建 API 文档，原型化 API，并用它来自动生成我们的代码，以及进行 API 测试。

1.  当 React 方法无法看到状态时，我们有两个选择。我们可以将其更改为使用`=>`，以便自动捕获`this`上下文，或者我们可以使用 JavaScript 的`bind`功能来绑定到正确的上下文。

# 第九章

1.  虽然 TensorFlow 现在支持 TypeScript/JavaScript，但最初是作为 Python 库发布的。TensorFlow 的后端是使用高性能 C++编写的。

1.  监督式机器学习利用先前的学习，并利用这些来处理新数据。它使用标记的示例来学习正确的答案。在这背后，有训练数据集，监督算法会根据这些数据集来完善它们的知识。

1.  MobileNet 是一种专门的**卷积神经网络**（**CNN**），除其他外，它提供了预先训练的图像分类模型。

1.  MobileNet 的`classify`方法默认返回包含分类名称和概率的三个分类。这可以通过指定要返回的分类数量来覆盖。

1.  当我们想要创建 Vue 应用程序时，我们使用以下命令：

```ts
vue create <<applicationname>>
```

由于我们想创建 TypeScript 应用程序，我们选择手动选择功能，并在功能屏幕上确保选择 TypeScript 作为我们的选项。

1.  当我们在`.vue`文件中创建一个类时，我们使用`@Component`来标记它为一个可以在 Vue 中注册的组件。

# 第十章

1.  JavaScript 和 C#都可以追溯到 C 语言的语法根源，因此它们在很大程度上遵循类似的语言范式，比如使用`{}`来表示操作的范围。由于所有的 JavaScript 都是有效的 TypeScript，这意味着 TypeScript 在这方面完全相同。

1.  启动我们程序的方法是`static Main`方法。它看起来像这样：

```ts
public static void Main(string[] args)
{
  CreateWebHostBuilder(args).Build().Run();
}
```

1.  ASP.NET Core 使用了重写的.NET 版本，去除了它只能在 Windows 平台上运行的限制。这意味着 ASP.NET 的覆盖范围大大增加，因为它现在可以在 Linux 平台上运行，也可以在 Windows 上运行。

1.  Discog 限制了单个 IP 发出的请求数量。对于经过身份验证的请求，Discog 将请求速率限制为每分钟 60 次。对于未经身份验证的请求，在大多数情况下，可以发送的请求数量为每分钟 25 次。请求的数量使用移动窗口进行监控。
