# 第二章. 增加复杂性的挑战

程序的本质是可能分支的组合以及基于某些条件的自动化选择。当我们编写程序时，我们定义了分支中的内容，以及这个分支将在什么条件下被执行。

在项目的演变过程中，分支的数量通常会迅速增长，以及确定分支是否将被执行的条件的数量。

这对人类来说很危险，因为人类的脑容量有限。

在本章中，我们将实现一个数据同步服务。从实现一些非常基本的功能开始，我们将继续添加内容，看看事情会如何发展。

以下内容将涵盖：

+   设计多设备同步策略

+   相关的有用 JavaScript 和 TypeScript 技术和提示，包括对象作为映射和字符串字面量类型

+   策略模式如何帮助项目

# 实现基本功能

在我们开始编写实际代码之前，我们需要定义这种同步策略将是什么样的。为了使实现免受不必要的干扰，客户端将通过函数调用直接与服务器通信，而不是使用 HTTP 请求或套接字。此外，我们将在客户端和服务器端使用**内存存储**，即变量，来存储数据。

由于我们没有将客户端和服务器分开成两个实际的应用程序，并且我们没有真正使用后端技术，因此不需要太多的 Node.js 经验来理解本章。

然而，请注意，尽管我们省略了网络和数据库请求，但我们希望最终实现的逻辑核心可以在不进行太多修改的情况下应用于实际环境。因此，当涉及到性能问题时，我们仍然需要假设有限的网络资源，特别是对于通过服务器和客户端传输的数据，尽管实现将是同步的而不是异步的。在实际情况中，这不应该发生，但涉及异步操作将引入更多的代码，以及许多需要考虑的情况。但在接下来的章节中，我们将介绍一些关于异步编程的有用模式，如果你尝试实现本章中同步逻辑的异步版本，这将非常有帮助。

如果客户端没有修改已同步的内容，它将存储服务器上所有可用数据的副本，而我们需要做的是提供一组 API，使客户端能够保持其数据副本的同步。

因此，一开始真的很简单：比较最后修改的时间戳。如果客户端的时间戳比服务器上的旧，那么就更新数据副本以及新的时间戳。

## 创建代码库

首先，让我们创建`server.ts`和`client.ts`文件，分别包含`Server`类和`Client`类：

```js
export class Server { 
    // ... 
} 

export class Client { 
    // ... 
} 

```

我更喜欢创建一个`index.ts`文件作为包的入口，它内部处理要导出的内容。在这种情况下，让我们导出所有内容：

```js
export * from './server'; 
export * from './client'; 

```

要从测试文件（假设`src/test/test.ts`）中导入`Server`和`Client`类，我们可以使用以下代码：

```js
import { Server, Client } from '../'; 

```

## 定义要同步的数据的初始结构

由于我们需要比较客户端和服务器的时间戳，我们需要在数据结构上有一个`timestamp`属性。我希望同步的数据是一个字符串，所以让我们在`server.ts`文件中添加一个带有`timestamp`属性的`DataStore`接口：

```js
export interface DataStore { 
  timestamp: number; 
  data: string; 
} 

```

## 通过比较时间戳来获取数据

目前，同步策略是单向的，从服务器到客户端。所以我们需要做的就是简单比较时间戳；如果服务器有更新的，它将响应数据和服务器端的时间戳；否则，它将响应`undefined`：

```js
class Server { 
  store: DataStore = { 
    timestamp: 0, 
    data: '' 
  }; 

  getData(clientTimestamp: number): DataStore { 
    if (clientTimestamp < this.store.timestamp) { 
      return this.store; 
    } else { 
      return undefined; 
    } 
  } 
} 

```

现在我们已经为客户端提供了一个简单的 API，现在是时候实现客户端了：

```js
import { Server, DataStore } from './'; 

export class Client { 
  store: DataStore = { 
    timestamp: 0, 
    data: undefined 
  }; 

  constructor( 
    public server: Server 
  ) { } 
} 

```

### 小贴士

在构造函数参数前加上访问修饰符（包括`public`、`private`和`protected`）将创建一个具有相同名称和相应可访问性的属性。当调用构造函数时，它还会自动分配值。

现在我们需要在`Client`类中添加一个`synchronize`方法来完成这项工作：

```js
synchronize(): void { 
  let updatedStore = this.server.getData(this.store.timestamp); 

  if (updatedStore) { 
    this.store = updatedStore; 
  } 
} 

```

这很容易做到。然而，你是否已经开始觉得我们写的东西有些尴尬？

## 双向同步

通常，当我们谈论同步时，我们会从服务器获取更新，并将更改推送到服务器。现在我们将进行第二部分，如果客户端有更新的数据，则推送更改。

但首先，我们需要给客户端添加更新其数据的能力，通过在`Client`类中添加一个`update`方法：

```js
update(data: string): void { 
  this.store.data = data; 
  this.store.timestamp = Date.now(); 
} 

```

我们还需要服务器能够从客户端接收数据。因此，我们将`Server`类的`getData`方法重命名为`synchronize`，并使其满足新的任务：

```js
synchronize(clientDataStore: DataStore): DataStore { 
  if (clientDataStore.timestamp > this.store.timestamp) { 
      this.store = clientDataStore; 
      return undefined; 
  } else if (clientDataStore.timestamp < this.store.timestamp) { 
      return this.store; 
  } else { 
      return undefined; 
  } 
} 

```

现在我们已经有了同步服务的初步实现。以后，我们将继续添加新功能，使其能够处理各种场景。

## 在实现基本功能时出现的问题

目前，我们写的代码太简单了，以至于可能出错。但仍然存在一些语义问题。

### 从服务器传递数据存储给客户端是没有意义的

我们在`Server`类的`synchronize`方法上使用了`DataStore`作为返回类型。但实际上我们传递的不是数据存储，而是涉及数据和其时间戳的信息。这个信息对象恰好在这个时间点具有与数据存储相同的属性。

此外，这也会误导后来阅读你代码的人（包括未来的你自己）。大多数时候，我们试图消除冗余。但这并不意味着所有看起来相同的东西。所以让我们将其定义为两个接口：

```js
interface DataStore { 
  timestamp: number; 
  data: string; 
} 

interface DataSyncingInfo { 
  timestamp: number; 
  data: string; 
} 

```

我甚至更愿意创建另一个实例，而不是直接返回 `this.store`：

```js
return { 
  timestamp: this.store.timestamp, 
  data: this.store.data 
}; 

```

然而，如果两段代码在代码本身的角度来看具有不同的语义意义，但执行相同的功能，你可能考虑将这部分提取为一个实用工具。

### 使关系清晰

现在我们有两个分离的接口，`DataStore` 和 `DataSyncingInfo`，在 `server.ts` 中。显然，`DataSyncingInfo` 应该是服务器和客户端之间的共享接口，而 `DataStore` 在两边都是相同的，但实际上并没有共享。

因此，我们将创建一个单独的 `shared.d.ts`（如果它包含的不仅仅是 `typings`，也可以是 `shared.ts`），导出 `DataSyncingInfo` 并在 `client.ts` 中添加另一个 `DataStore`。

### 注意

不要盲目跟随。有时这是为了服务器和客户端具有完全相同的存储而设计的。如果是这种情况，接口应该是共享的。

# 特征增长

我们到目前为止所做的一切基本上都是无用的。但是，从现在开始，我们将开始添加功能，使其能够满足实际需求，包括与多个客户端同步多个数据项的能力，以及合并冲突。

## 同步多个项目

理想情况下，我们需要同步的数据将包含大量项目。如果这些项目数量非常有限，直接将 `data` 的类型更改为数组是可行的。

### 简单地用数组替换数据类型

现在让我们将 `DataStore` 和 `DataSyncingInfo` 接口中的 `data` 属性类型更改为 `string[]`。借助 TypeScript，你会得到由于这种更改导致的类型不匹配的错误。通过注释正确的类型来修复它们。

但显然，这远非一个高效的解决方案。

### 以服务器为中心的同步

如果数据存储包含大量数据，理想的方法是只更新那些未更新的项目。

例如，我们可以为每个单独的项目创建一个时间戳，并将这些时间戳发送到服务器，然后让服务器决定特定的数据项是否是最新的。这在某些场景中是一个可行的方案，例如检查软件扩展的更新。在快速网络上偶尔发送带有项目 ID 的数百个时间戳是可以的，但我们将为不同的场景使用另一种方法，否则我将没有太多可写的内容。

在手机上处理离线应用的用户数据同步是我们将要处理的，这意味着我们需要尽力避免浪费网络资源。

### 注意

这里有一个有趣的问题。用户数据同步和检查扩展更新之间有什么区别？考虑数据的大小、多设备的问题等等。

我们考虑发送所有项目的时间戳的原因是让服务器确定某些项目是否需要更新。然而，有必要在客户端存储所有数据项的时间戳吗？

如果我们选择不存储数据更改的时间戳，而是存储与服务器同步的数据的时间戳，那么我们只需发送最后一次成功同步的时间戳就可以获取所有最新的数据。然后，服务器将比较这个时间戳与所有数据项的最后修改时间戳，并决定如何响应。

如本部分标题所示，该过程以服务器为中心，依赖于服务器生成时间戳（尽管它不必这样做，实际上也不应该这样做）。

### 备注

如果你对这些时间戳的工作方式感到困惑，让我们再试一次。服务器将存储上次同步项的时间戳，而客户端将存储与服务器最后一次成功同步的时间戳。因此，如果服务器上没有项的时间戳晚于客户端，那么在该时间戳之后，服务器数据存储没有变化。但如果有一些变化，通过比较客户端的时间戳与服务器项的时间戳，我们将知道哪些项是较新的。

#### 从服务器到客户端的同步

现在似乎有很多东西需要更改。首先，让我们处理从服务器到客户端的数据同步。

这是服务器端期望发生的情况：

+   为服务器上的每个数据项添加时间戳和标识

+   将客户端时间戳与服务器上的每个数据项进行比较

### 备注

如果这些项具有排序索引，我们实际上不需要将客户端的时间戳与服务器上的每个项进行比较。使用具有排序索引的数据库，性能是可以接受的。

+   返回比客户端拥有的更新的项以及一个新的时间戳。

在客户端期望发生的情况：

+   与服务器发送的最后一个时间戳同步

+   使用服务器响应的新数据更新本地存储

+   如果同步完成且没有错误，则更新本地最后同步的时间戳

**更新接口**

首先，我们现在在双方都有更新的数据存储。从服务器开始，数据存储现在包含一个数据项数组。因此，让我们定义 `ServerDataItem` 接口并更新 `ServerDataStore`：

```js
export interface ServerDataItem { 
  id: string; 
  timestamp: number; 
  value: string; 
} 

export interface ServerDataStore { 
  items: { 
    [id: string]: ServerDataItem; 
  }; 
} 

```

### 备注

`{ [id: string]: ServerDataItem }` 类型描述了一个对象，它以 `id` 类型为 `string` 的键，并具有 `ServerDataItem` 类型的值。因此，可以通过 `items['the-id']` 访问 `ServerDataItem` 类型的项。

对于客户端，我们现在有不同的数据项和不同的存储。响应只包含所有数据项的一个子集，因此我们需要 ID 和一个以 ID 为索引的映射来存储数据：

```js
export interface ClientDataItem { 
  id: string; 
  value: string; 
} 

export interface ClientDataStore { 
  timestamp: number; 
  items: { 
    [id: string]: ClientDataItem; 
  }; 
} 

```

以前，客户端和服务器共享相同的 `DataSyncingInfo`，但这种情况将要改变。由于我们首先处理服务器到客户端的同步，所以我们现在只关心同步请求中的时间戳：

```js
export interface SyncingRequest { 
  timestamp: number; 
} 

```

至于服务器的响应，它应该有一个更新的时间戳，与请求时间戳相比，数据项已发生变化：

```js
export interface SyncingResponse { 
  timestamp: number; 
    changes: { 
      [id: string]: string; 
    }; 
} 

```

我用*Server*和*Client*前缀那些接口，以便更好地区分。但如果你不是从`server.ts`和`client.ts`（在`index.ts`中）导出所有内容，这不是必要的。

**更新服务器端**

使用定义良好的数据结构，应该很容易实现我们期望的结果。首先，我们有`synchronize`方法，它接受一个`SyncingRequest`并返回一个`SyncingResponse`；并且我们需要有更新的时间戳：

```js
synchronize(request: SyncingRequest): SyncingResponse { 
  let lastTimestamp = request.timestamp; 
  let now = Date.now(); 

  let serverChanges: ServerChangeMap = Object.create(null); 

  return { 
    timestamp: now, 
    changes: serverChanges 
  }; 
} 

```

### 提示

对于`serverChanges`对象，`{}`（一个对象字面量）可能是首先想到的（如果不是 ES6 Map）。但这样做并不绝对安全，因为它会拒绝`__proto__`作为键。更好的选择是`Object.create(null)`，它接受所有字符串作为其键。

现在我们将添加比客户端更新的项到`serverChanges`：

```js
let items = this.store.items; 

for (let id of Object.keys(items)) { 
  let item = items[id]; 

  if (item.timestamp > lastTimestamp) { 
    serverChanges[id] = item.value; 
  } 
} 

```

**更新客户端**

由于我们将`ClientDataStore`下的`items`的类型更改为映射，我们需要修复初始值：

```js
store: ClientDataStore = { 
  timestamp: 0, 
  items: Object.create(null) 
}; 

```

现在让我们更新`synchronize`方法。首先，客户端将发送一个带有时间戳的请求并从服务器获取响应：

```js
synchronize(): void { 
  let store = this.store; 

  let response = this.server.synchronize({ 
    timestamp: store.timestamp 
  }); 
} 

```

然后，我们将保存更新的数据项到存储中：

```js
let clientItems = store.items;  
let serverChanges = response.changes; 

for (let id of Object.keys(serverChanges)) { 
  clientItems[id] = { 
    id, 
    value: serverChanges[id] 
  }; 
} 

```

最后，更新最后一次成功同步的时间戳：

```js
clientStore.timestamp = response.timestamp; 

```

### 注意

更新同步时间戳应该是完整同步过程中最后要做的事情。确保它不早于数据项存储，否则如果在未来的同步过程中出现任何错误或中断，可能会有一个损坏的离线副本。

### 注意

为了确保它按预期工作，具有相同更改信息的操作即使在多次应用时也应给出相同的结果。

#### 从客户端到服务器的同步

对于以服务器为中心的同步过程，大多数更改都是通过客户端进行的。因此，我们需要在将它们发送到服务器之前组织这些更改。

单个客户端只关心它自己的数据副本。与从服务器同步数据到客户端的过程相比，这会有什么不同呢？好吧，想想我们最初为什么需要服务器上每个数据项的时间戳。我们需要它们是因为我们想知道与特定客户端相比哪些项是新的。

现在，对于客户端的更改：如果它们发生，它们需要同步到服务器，而不需要比较特定的时间戳。

然而，我们可能有多个需要同步更改的客户，这意味着后来时间做出的更改实际上可能会更早地同步，因此我们需要解决冲突。为了实现这一点，我们需要将最后修改时间添加到服务器上的每个数据项和客户端的更改项上。

我已经提到，服务器上存储的时间戳，用于确定需要同步到客户端的内容，不需要（而且最好不是）实际时间点的实际戳记。例如，它可以是所有客户端与服务器之间发生的同步次数。

**更新客户端**

为了高效地处理这个问题，我们可以在 `ClientDataStore` 中创建一个单独的映射，以数据项的 ID 作为键，最后修改时间作为值：

```js
export interface ClientDataStore { 
  timestamp: number; 
  items: { 
    [id: string]: ClientDataItem; 
  }; 
  changed: { 
    [id: string]: number; 
  }; 
} 

```

你可能还想将其值初始化为 `Object.create(null)`。

现在我们更新客户端存储中的项时，我们也将最后修改时间添加到 `changed` 映射中：

```js
update(id: string, value: string): void { 
  let store = this.store; 

  store.items[id] = { 
    id, 
    value 
  }; 

  store.changed[id] = Date.now(); 
} 

```

`SyncingRequest` 中的单个时间戳肯定不能再胜任这项工作了；我们需要为更改的数据添加一个位置，一个以数据项 ID 作为索引，更改信息作为值的映射：

```js
export interface ClientChange { 
  lastModifiedTime: number; 
  value: string; 
} 

export interface SyncingRequest { 
  timestamp: number; 
  changes: { 
    [id: string]: ClientChange; 
  }; 
} 

```

这里又出现了一个问题。如果一个客户端数据项的更改是在离线状态下进行的，系统时钟时间不正确，怎么办？显然，我们需要一些时间校准机制。然而，没有方法可以做到完美的校准。我们将做一些假设，这样我们就不需要为时间校准开启另一个章节：

+   客户端的系统时钟可能比服务器晚或早，但它以正常速度运行，不会在时间之间跳跃。

+   从客户端发送的请求在相对较短的时间内到达服务器。

基于这些假设，我们可以将这些构建块添加到客户端的 `synchronize` 方法中：

1.  在将同步请求（当然，在发送到服务器之前）发送到服务器之前，添加客户端更改：

    ```js
          let clientItems = store.items; 
          let clientChanges: ClientChangeMap = Object.create(null); 

          let changedTimes = store.changed; 

          for (let id of Object.keys(changedTimes)) { 
            clientChanges[id] = { 
              lastModifiedTime: changedTimes[id], 
              value: clientItems[id].value 
            }; 
          } 

    ```

1.  将服务器上的更改与客户端时钟的当前时间同步：

    ```js
          let response = this.server.synchronize({ 
            timestamp: store.timestamp, 
            clientTime: Date.now(), 
            changes: clientChanges 
          }); 

    ```

1.  在成功同步后清理更改：

    ```js
          store.changed = Object.create(null); 

    ```

**更新服务器端**

如果客户端按预期工作，它应该发送包含更改的同步请求。现在是时候让服务器能够处理来自客户端的这些更改了。

服务器端同步过程将分为两个步骤：

1.  将客户端更改应用到服务器数据存储。

1.  准备需要同步到客户端的更改。

首先，我们需要像之前提到的那样，将 `lastModifiedTime` 添加到服务器端数据项中：

```js
export interface ServerDataItem { 
    id: string; 
    timestamp: number; 
    lastModifiedTime: number; 
    value: string; 
} 

```

我们还需要更新 `synchronize` 方法：

```js
let clientChanges = request.changes; 
let now = Date.now(); 

for (let id of Object.keys(clientChanges)) { 
  let clientChange = clientChanges[id]; 

  if ( 
    hasOwnProperty.call(items, id) &&  
    items[id].lastModifiedTime > clientChange.lastModifiedTime 
  ) { 
    continue; 
  } 

  items[id] = { 
    id, 
    timestamp: now, 
    lastModifiedTime, 
    value: clientChange.value 
  }; 
} 

```

### 注意

实际上，我们可以在这里使用 `in` 操作符而不是 `hasOwnProperty`，因为 `items` 对象是以 `null` 作为其原型的。但是，如果你使用的是通过对象字面量创建的对象，或者以其他方式，如 maps，那么对 `hasOwnProperty` 的引用将是你的朋友。

我们已经讨论了通过比较最后修改时间来解决冲突。同时，我们已经做出了假设，这样我们就可以通过在同步时传递客户端时间到服务器，轻松地校准客户端的最后修改时间。

我们将要做的校准是计算客户端时间与服务器时间的偏移量。这就是为什么我们做出了第二个假设：请求需要相对较短时间内轻松到达服务器。为了计算偏移量，我们可以简单地从服务器时间中减去客户端时间：

```js
let clientTimeOffset = now - request.clientTime; 

```

### 注意

为了使时间校准更准确，我们希望记录最早的时间戳，即请求击中服务器后的时间戳。在实践中，你可能会在开始处理一切之前记录请求击中服务器的时间戳。例如，对于 HTTP 请求，你可以在 TCP 连接建立后记录时间戳。

现在，客户端更改的校准时间是原始时间和偏移量的总和。我们可以通过比较校准的最后修改时间来决定是否保留或忽略客户端的更改。校准时间可能大于服务器时间；你可以选择使用服务器时间作为最大值或接受一点小误差。在这里，我们将采取简单的方法：

```js
let lastModifiedTime = Math.min( 
  clientChange.lastModifiedTime + clientTimeOffset, 
  now 
); 

if ( 
  hasOwnProperty.call(items, id) &&  
  items[id].lastModifiedTime > lastModifiedTime 
) { 
  continue; 
} 

```

要使这真正工作，我们还需要排除与客户端更改冲突的服务器更改。为此，我们需要知道在冲突解决过程中幸存下来的更改是什么。一种简单的方法是排除时间戳等于 `now` 的项：

```js
for (let id of Object.keys(items)) { 
  let item = items[id]; 

  if ( 
    item.timestamp > lastTimestamp && 
    item.timestamp !== now 
  ) { 
    serverChanges[id] = item.value; 
  } 
} 

```

因此，我们现在已经实现了一个完整的同步逻辑，能够处理实践中简单的冲突。

## 同步多种类型的数据

在这一点上，我们已将数据硬编码为 `string` 类型。但通常我们还需要存储各种数据，例如数字、布尔值、对象等。

如果我们正在编写 JavaScript，实际上我们不需要做任何改变，因为实现与某些数据类型无关。在 TypeScript 中，我们也不需要做太多：只需将每个相关 `value` 的类型更改为 `any`。但这意味着你正在失去类型安全，如果你对此感到满意，那当然是可以接受的。

但根据我的个人偏好，我希望如果可能的话，每个变量、参数和属性都应该有类型。因此，我们可能仍然有一个 `value` 类型为 `any` 的数据项：

```js
export interface ClientDataItem { 
  id: string; 
  value: any; 
} 

```

我们还可以为特定数据类型有派生接口：

```js
export interface ClientStringDataItem extends ClientDataItem { 
  value: string; 
} 

export interface ClientNumberDataItem extends ClientDataItem { 
  value: number; 
} 

```

但这似乎还不够好。幸运的是，TypeScript 提供了 *泛型*，因此我们可以将前面的代码重写如下：

```js
export interface ClientDataItem<T> { 
  id: string; 
  value: T; 
} 

```

假设我们有一个接受多种类型数据项的存储 - 例如，数字和字符串 - 我们可以使用 `union` 类型声明如下：

```js
export interface ClientDataStore { 
  items: { 
    [id: string]: ClientDataItem<number | string>; 
  }; 
} 

```

如果你记得我们正在为离线移动应用做些事情，你可能会对更改中长属性名，如 `lastModifiedTime` 提出疑问。这是一个合理的问题，一个简单的解决方案是使用 `tuple` 类型，也许还可以与 `enums` 一起使用：

```js
const enum ClientChangeIndex { 
  lastModifiedType, 
  value 
} 

type ClientChange<T> = [number, T]; 

let change: ClientChange<string> = [0, 'foo']; 
let value = change[ClientChangeIndex.value]; 

```

您可以根据自己的喜好应用更多或更少的类型化内容。如果您还不熟悉它们，可以在此处了解更多信息：[`www.typescriptlang.org/handbook`](http://www.typescriptlang.org/handbook)。

## 支持具有增量数据的多个客户端

使类型系统满足多种数据类型很容易。但在现实世界中，我们不会通过简单地比较最后修改时间来解决所有数据类型的冲突。一个例子是跨设备计算用户的每日活跃时间。

很明显，我们需要将多台设备上每天的所有活跃时间加起来。这就是我们将如何实现这一点：

1.  在客户端同步之间累积活跃时长。

1.  在与服务器同步之前，为每一段时间添加一个 UID（唯一标识符）。

1.  如果 UID 尚不存在，则增加服务器端值，然后将 UID 添加到该数据项中。

但在我们实际着手这些步骤之前，我们需要一种方法来区分增量数据项和普通数据项，例如，通过添加一个`type`属性。

由于我们的同步策略是服务器中心的，因此只需要为同步请求和冲突合并提供相关信息。同步响应不需要包括变化的细节，只需合并后的值即可。

### 注意

我将停止逐步说明如何更新每个接口，因为我们正在接近最终结构。但如果您在这方面有任何问题，您可以查看完整的代码包以获取灵感。

### 更新客户端

首先，我们需要客户端支持增量变化。如果您已经考虑过这个问题，您可能已经对放置额外信息的位置，例如 UID，感到困惑。

这是因为我们将概念*变化*（名词）与*值*混淆了。在此之前这并不是问题，因为除了最后修改时间外，值就是变化的内容。我们使用一个简单的映射来存储最后修改时间，并保持存储的清洁，这在那种情况下平衡得很好。

但现在我们需要区分这两个概念：

+   **值**：值以静态方式描述数据项是什么

+   **变化**：变化描述了可能将数据项的值从一种转换为另一种的信息

我们需要有一种通用的变化类型，以及一种新的数据结构来处理带有数值的增量变化：

```js
type DataType = 'value' | 'increment'; 

interface ClientChange { 
  type: DataType; 
} 

interface ClientValueChange<T> extends ClientChange { 
  type: 'value'; 
  lastModifiedTime: number; 
  value: T; 
} 

interface ClientIncrementChange extends ClientChange { 
  type: 'increment'; 
  uid: string; 
  increment: number; 
} 

```

### 注意

我们在这里使用的是`string literal`类型，这是在 TypeScript 1.8 中引入的。要了解更多信息，请参阅我们之前提到的 TypeScript 手册。

应对数据存储结构进行类似的变化。并且当我们在客户端更新一个项目时，我们需要根据不同的数据类型应用正确的操作：

```js
update(id: string, type: 'increment', increment: number): void; 
update<T>(id: string, type: 'value', value: T): void; 
update<T>(id: string, type: DataType, value: T): void; 
update<T>(id: string, type: DataType, value: T): void { 
  let store = this.store; 

  let items = store.items; 
  let storedChanges = store.changes; 

  if (type === 'value') { 
    // ... 
  } else if (type === 'increment') { 
    // ... 
  } else { 
    throw new TypeError('Invalid data type'); 
  } 
} 

```

使用以下代码进行常规变化（当`type`等于`'value'`时）：

```js
let change: ClientValueChange<T> = { 
  type: 'value', 
  lastModifiedTime: Date.now(), 
  value 
}; 

storedChanges[id] = change; 

if (hasOwnProperty.call(items, id)) { 
  items[id].value = value; 
} else { 
  items[id] = { 
    id, 
    type, 
    value 
  }; 
} 

```

对于增量变化，需要更多几行代码：

```js
let storedChange = storedChanges[id] as ClientIncrementChange; 

if (storedChange) { 
  storedChange.increment += <any>value as number; 
} else { 
  storedChange = { 
    type: 'increment', 
    uid: Date.now().toString(), 
    increment: <any>value as number 
  }; 

  storedChanges[id] = storedChange; 
} 

```

### 注意

我个人偏好使用`<T>`进行`any`类型转换，以及使用`as T`进行非`any`类型转换。尽管它已在像 C#这样的语言中使用，但 TypeScript 中的`as`操作符最初是为了与 JSX 中的 XML 标签兼容而引入的。如果你愿意，你也可以在这里写`<number><any>value`或`value as any as number`。

不要忘记更新存储的值。只需将比较更新正常数据项时的`=`改为`+=`即可：

```js
if (hasOwnProperty.call(items, id)) { 
  items[id].value += value; 
} else { 
  items[id] = { 
    id, 
    type, 
    value 
  }; 
} 

```

这一点也不难。但嘿，我们看到了分支。

我们一直在编写分支，但`if (type === 'foo') { ... }`和`if (item.timestamp > lastTimestamp) { ... }`这样的分支之间有什么区别呢？让我们记住这个问题，然后继续前进。

通过`update`方法添加必要的信息后，我们现在可以更新客户端的`synchronize`方法。但在实际场景中存在一个缺陷：同步请求成功发送到服务器，但客户端未能接收到服务器的响应。在这种情况下，当在失败的同步之后调用`update`时，增量会被添加到可能已同步的变更（通过其 UID 识别），在未来的同步中将被服务器忽略。为了解决这个问题，我们需要为所有已经开始同步过程的增量变更添加标记，并避免累积这些变更。因此，我们需要为相同的数据项创建另一个变更。

这实际上是一个很好的提示：因为变更涉及的信息是将一个值从一种形式转换为另一种形式，所以几个待同步的变更最终可能会应用到单个数据项上：

```js
interface ClientChangeList<T extends ClientChange> { 
  type: DataType; 
  changes: T[]; 
} 

interface SyncingRequest { 
  timestamp: number; 
  changeLists: { 
    [id: string]: ClientChangeList<ClientChange>; 
  }; 
} 

interface ClientIncrementChange extends ClientChange { 
  type: 'increment'; 
  synced: boolean; 
  uid: string; 
  increment: number; 
} 

```

现在我们尝试更新一个增量数据项时，我们需要从变更列表（如果有）中获取其最后变更，并查看它是否曾经参与过同步。如果它曾经参与过同步，我们将创建一个新的变更实例。否则，我们只需在客户端累积最后变更的`increment`属性值：

```js
let changeList = storedChangeLists[id]; 
let changes = changeList.changes; 
let lastChange = 
  changes[changes.length - 1] as ClientIncrementChange; 

if (lastChange.synced) { 
  changes.push({ 
    synced: false, 
    uid: Date.now().toString(), 
    increment: <any>value as number 
  } as ClientIncrementChange); 
} else { 
  lastChange.increment += <any>value as number; 
} 

```

或者，如果变更列表尚不存在，我们需要将其设置起来：

```js
let changeList = { 
  type: 'increment', 
  changes: [ 
    { 
      synced: false, 
      uid: Date.now().toString(), 
      increment: <any>value as number 
    } as ClientIncrementChange 
  ] 
}; 

store.changeLists[id] = changeList; 

```

我们还需要更新`synchronize`方法，在开始与服务器同步之前将增量变更标记为`synced`。但具体的实现需要你自己来完成。

### 更新服务器端

在我们添加处理增量变更的逻辑之前，我们需要让服务器端代码适应新的数据结构：

```js
for (let id of Object.keys(clientChangeLists)) { 
  let clientChangeList = clientChangeLists[id]; 

  let type = clientChangeList.type; 
  let clientChanges = clientChangeList.changes; 

  if (type === 'value') { 
    // ... 
  } else if (type === 'increment') { 
    // ... 
  } else { 
    throw new TypeError('Invalid data type'); 
  } 
} 

```

正常数据项的变更列表将始终只包含一个变更。因此我们可以轻松迁移我们已编写的代码：

```js
let clientChange = changes[0] as ClientValueChange<any>; 

```

现在对于增量变更，我们需要将可能多个变更累积应用到单个变更列表中的数据项上：

```js
let item = items[id]; 

for ( 
  let clientChange 
  of clientChanges as ClientIncrementChange[] 
) { 
  let { 
    uid, 
    increment 
  } = clientChange; 

  if (item.uids.indexOf(uid) < 0) { 
    item.value += increment; 
    item.uids.push(uid); 
  } 
} 

```

但请记住要处理时间戳或不存在指定 ID 的项目的情况：

```js
let item: ServerDataItem<any>; 

if (hasOwnProperty.call(items, id)) { 
  item = items[id]; 
  item.timestamp = now; 
} else { 
  item = items[id] = { 
    id, 
    type, 
    timestamp: now, 
    uids: [], 
    value: 0 
  }; 
} 

```

如果不知道客户端增量数据项的当前值，我们无法保证该值是最新的。之前，我们通过比较时间戳与当前同步的时间戳来决定是否响应新值，但这种方法对于增量更改不再适用。

通过删除`clientChangeLists`中仍需要同步到客户端的键，可以简单地使这一过程工作。在准备响应时，它可以跳过`clientChangeLists`中仍存在的 ID：

```js
if ( 
  item.timestamp > lastTimestamp && 
  !hasOwnProperty.call(clientChangeLists, id) 
) { 
  serverChanges[id] = item.value; 
} 

```

记得为在解决冲突中未存活的普通数据项添加`delete clientChangeLists[id];`。

现在我们已经实现了一种同步逻辑，可以为离线应用程序完成很多工作。早些时候，我提出了关于增加分支的问题，这些分支看起来并不好。但如果你知道你的功能将在这里结束，或者至少有有限的变化，这并不是一个糟糕的实现，尽管我们很快就会越过平衡点，因为满足 80%的需求不会让我们感到足够满意。

## 支持更多冲突合并

虽然我们已经满足了 80%的需求，但我们仍然有很大可能想要一些额外的功能。例如，我们想要用户标记为当前月份可用的天数比例，并且用户应该能够从列表中添加或删除天数。我们可以用不同的方式实现这一点，但我们将选择一种简单的方式，就像往常一样。

我们将支持通过添加和删除等操作同步集合，并在客户端计算比率。

### 新的数据结构

为了描述集合更改，我们需要一个新的`ClientChange`类型。当我们向集合中添加或删除一个元素时，我们只关心对同一元素的最后操作。这意味着以下内容：

1.  如果对同一元素进行了多个操作，我们只需要保留最后一个。

1.  需要一个`time`属性来解决问题。

因此，这里有新的类型：

```js
enum SetOperation { 
  add, 
  remove 
} 

interface ClientSetChange extends ClientChange { 
  element: number; 
  time: number; 
  operation: SetOperation; 
} 

```

服务器端存储的集合数据将略有不同。我们将有一个以元素（以`string`形式）为键的映射，以及具有`operation`和`time`属性的结构作为值：

```js
interface ServerSetElementOperationInfo { 
  operation: SetOperation; 
  time: number; 
} 

```

现在我们有足够的信息来从多个客户端解决冲突。并且我们可以通过键生成集合，这需要借助对元素进行的最后操作：

### 更新客户端

现在，客户端的`update`方法得到了一份兼职工作：保存集合更改，就像值和增量更改一样。我们需要为此新工作更新方法签名（不要忘记将`'set'`添加到`DataType`）：

```js
update( 
  id: string, 
  type: 'set', 
  element: number, 
  operation: SetOperation 
): void; 
update<T>( 
  id: string, 
  type: DataType, 
  value: T, 
  operation?: SetOperation 
): void; 

```

我们还需要添加另一个`else if`：

```js
else if (type === 'set') { 
  let element = <any>value as number; 

  if (hasOwnProperty.call(storedChangeLists, id)) { 
    // ... 
  } else { 
    // ... 
  } 
} 

```

如果已经对这个集合进行了操作，我们需要找到并删除对目标元素的最后操作（如果有）。然后附加一个包含最新操作的新更改：

```js
let changeList = storedChangeLists[id]; 
let changes = changeList.changes as ClientSetChange[]; 

for (let i = 0; i < changes.length; i++) { 
  let change = changes[i]; 

  if (change.element === element) { 
    changes.splice(i, 1); 
    break; 
  } 
} 

changes.push({ 
  element, 
  time: Date.now(), 
  operation 
}); 

```

如果自上次成功同步以来没有进行任何更改，我们需要为最新的操作创建一个新的更改列表：

```js
let changeList: ClientChangeList<ClientSetChange> = { 
  type: 'set', 
  changes: [ 
    { 
      element, 
      time: Date.now(), 
      operation 
    } 
  ] 
}; 

storedChangeLists[id] = changeList; 

```

再次提醒，不要忘记更新存储的值。这不仅仅是分配或累积值，但它仍然应该相当容易实现。

### 更新服务器端

就像我们对客户端所做的那样，我们需要添加一个相应的`else if`分支来合并类型为`'set'`的更改。我们还在`clientChangeLists`中删除 ID，无论是否有更新的更改，以实现更简单的实现：

```js
else if (type === 'set') { 
  let item: ServerDataItem<{ 
    [element: string]: ServerSetElementOperationInfo; 
  }>; 

  delete clientChangeLists[id]; 
} 

```

冲突解决逻辑与我们对正常值冲突的处理相当相似。我们只需要对每个元素进行比较，并只保留最后一个操作。

当准备同步到客户端的响应时，我们可以通过将具有`add`作为最后操作的元素组合起来来生成集合：

```js
if (item.type === 'set') { 
  let operationInfos: { 
    [element: string]: ServerSetElementOperationInfo; 
  } = item.value; 

  serverChanges[id] = Object 
    .keys(operationInfos) 
    .filter(element => 
      operationInfos[element].operation === 
        SetOperation.add 
    ) 
    .map(element => Number(element)); 
} else { 
  serverChanges[id] = item.value; 
} 

```

最后，我们有一个正在工作的混乱（如果它实际上能工作的话）。干杯！

## 在实现一切时出现的问题

当我们开始添加功能时，事情实际上很好，如果你不是对设计感追求得过于执着。然后我们感觉到代码有点尴尬，因为我们看到了越来越多的嵌套分支。

因此，现在是时候回答问题了，我们编写的两种分支之间有什么区别？我对为什么我对`if (type === 'foo') { ... }`分支感到尴尬的理解是，它与上下文没有很强的关联。另一方面，比较时间戳是同步过程的一个更自然的部分。

再次提醒，我并不是说这是不好的。但这给我们一个提示，当我们开始失去控制（由于我们有限的脑容量，这只是复杂性的问题）时，我们可能从哪里开始我们的手术。

### 堆积类似但平行的过程

本章的大部分代码是用来处理客户端和服务器之间同步数据的过程。为了适应新功能，我们只是不断地在方法中添加新内容，例如`update`和`synchronize`。

你可能已经发现，大多数逻辑的轮廓都可以，并且应该，在多个数据类型之间共享。但我们没有这样做。

如果我们仔细观察，从代码文本的角度来看，重复实际上是很小的。以客户端的`update`方法为例，每个分支的逻辑似乎都不同。如果你还没有形成内置的抽象反应，你可能会就此停止。或者如果你不喜欢长函数，你可能会通过将其拆分为相同类的小函数来重构代码。这可能会让事情变得更好，但远远不够。

### 数据存储被极大地简化了

在实现中，我们与理想的*内存*存储进行了大量的直接操作。如果能有一个包装器，并使其真正的存储可互换，那会很好。

对于这个实现来说，这可能不是情况，因为它基于极其理想和简化的假设和要求。但添加一个包装器可能是一种提供有用辅助工具的方法。

# 确保正确

因此，让我们摆脱逐字符比较代码的错觉，并尝试找到一个可以应用于更新所有这些数据类型的抽象。这个抽象的两个关键点已经在上一节中提到：

+   一个`change`包含将一个项目的值从一种转换为另一种所需的信息

+   在单次同步过程中，可能对单个数据项生成或应用多个变化

现在，从变化开始，让我们思考当客户端的`update`方法被调用时会发生什么。

## 寻找抽象

仔细看看客户端的`update`方法：

+   对于`'value'`类型的数据，首先我们创建变化，包括一个新值，然后更新变化列表，使新创建的变化成为唯一的变化。之后，我们更新数据项的值。

+   对于`'increment'`类型的数据，我们在变化列表中添加一个包含增量变化的变化；或者如果已经存在尚未同步的变化，则更新现有变化中的增量。然后，我们更新数据项的值。

+   最后，对于`'set'`类型的数据，我们创建一个反映最新操作的变化。在将新的变化添加到变化列表后，我们也移除不再必要的变化。然后我们更新数据项的值。

事情越来越清晰。以下是当调用`update`时这些数据类型所发生的情况：

1.  创建新的变化。

1.  将新的变化合并到变化列表中。

1.  将新的变化应用于数据项。

现在甚至更好。对于不同的数据类型，每一步都是不同的，但不同的步骤具有相同的轮廓；我们需要做的是为不同的数据类型实现不同的策略。

## 实施策略

使用单个`update`函数执行所有类型的更改可能会令人困惑。在我们继续之前，让我们将其拆分为三个不同的方法：`update`用于普通值，`increase`用于增量值，以及`addTo`/`removeFrom`用于集合。

然后，我们将创建一个新的私有方法`applyChange`，它将接受由其他方法创建的变化，并继续执行步骤 2 和步骤 3。它接受一个具有两个方法`append`和`apply`的策略对象：

```js
interface ClientChangeStrategy<T extends ClientChange> { 
  append(list: ClientChangeList<T>, change: T): void; 
  apply(item: ClientDataItem<any>, change: T): void; 
} 

```

对于一个普通的数据项，策略对象可能如下所示：

```js
let strategy: ClientChangeStrategy<ClientValueChange<any>> = {  
  append(list, change) { 
    list.changes = [change]; 
  }, 
  apply(item, change) { 
    item.value = change.value; 
  } 
}; 

```

对于增量数据项，需要更多几行代码。首先，`append`方法：

```js
let changes = list.changes; 
let lastChange = changes[changes.length]; 

if (!lastChange || lastChange.synced) { 
  changes.push(change); 
} else { 
  lastChange.increment += change.increment; 
} 

```

`append`方法之后跟随`apply`方法：

```js
if (item.value === undefined) { 
  item.value = change.increment; 
} else { 
  item.value += change.increment; 
} 

```

现在在`applyChange`方法中，我们需要注意非现有项目和变化列表的创建，并根据不同的数据类型调用不同的`append`和`apply`方法。

同样的技术可以应用于其他过程。尽管适用于客户端和服务器端的详细过程不同，但我们仍然可以将它们一起作为模块编写。

## 包装存储

我们将在具有读写能力的普通内存存储对象周围创建一个轻量级包装器，以服务器端存储为例：

```js
export class ServerStore { 
  private items: { 
    [id: string]: ServerDataItem<any>; 
  } = Object.create(null); 
} 

export class Server { 
  constructor( 
    public store: ServerStore 
  ) { } 
} 

```

为了满足我们的需求，我们需要为`ServerStore`实现`get`、`set`和`getAll`方法（或者更好的是，一个带有条件的`find`方法）：

```js
get<T, TExtra extends ServerDataItemExtra>(id: string): 
  ServerDataItem<T> & TExtra { 
  return hasOwnProperty.call(this.items, id) ? 
    this.items[id] as ServerDataItem<T> & TExtra : undefined; 
} 

set<T, TExtra extends ServerDataItemExtra>( 
  id: string, 
  item: ServerDataItem<T> & Textra 
): void { 
  this.items[id] = item; 
} 

getAll<T, TExtra extends ServerDataItemExtra>(): 
  (ServerDataItem<T> & TExtra)[] { 
  let items = this.items; 
  return Object 
    .keys(items) 
    .map(id => items[id] as ServerDataItem<T> & TExtra); 
} 

```

你可能已经注意到，从接口和泛型中，我还将`ServerDataItem`拆分成了公共部分和额外部分的交叉类型。

# 摘要

在本章中，我们参与了这样一个简化但与现实相关的项目的演变过程。从一个简单的代码库开始，这个代码库不可能出错，我们添加了许多功能，并经历了将可接受的变化组合起来并使整个项目变得混乱的过程。

我们总是试图通过优雅的命名或添加必要的语义冗余来编写可读的代码，但随着复杂性的增加，这并不会有多大帮助。

在这个过程中，我们学习了离线同步的工作原理。借助最常见的设计模式，例如策略模式（Strategy Pattern），我们成功地将项目拆分为小而可控的部分。

在接下来的章节中，我们将使用 TypeScript 中的代码示例来列出更多有用的设计模式，并尝试将这些设计模式应用到具体问题上。
