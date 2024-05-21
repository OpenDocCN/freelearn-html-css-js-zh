# 第七章：使用 Firebase 进行 Angular 基于云的地图

在过去的几章中，我们花了相当多的时间编写我们自己的后端系统，以返回信息给客户端。在过去的几年里，有一种趋势是使用第三方云系统。云系统可以帮助降低编写应用程序的成本，因为其他公司提供了我们需要使用的所有基础设施，并负责测试、升级等。在本章中，我们将研究如何使用必应地图团队和 Firebase 的云基础设施来提供数据存储。

本章将涵盖以下主题：

+   注册必应地图

+   计费云功能的含义

+   注册 Firebase

+   添加地图组件

+   使用地图搜索功能

+   使用`EventEmitter`来通知父组件子组件事件

+   响应地图事件以添加和删除自己的兴趣点

+   在地图上叠加搜索结果

+   整理事件处理程序

+   将数据保存到 Cloud Firestore

+   配置 Cloud Firestore 身份验证

# 技术要求

完成的项目可以从[`github.com/PacktPublishing/Advanced-TypeScript-3-Programming-Projects/tree/master/Chapter07`](https://github.com/PacktPublishing/Advanced-TypeScript-3-Programming-Projects/tree/master/Chapter07)下载。

下载项目后，您将需要使用`npm install`命令安装软件包要求。

# 现代应用程序和转向云服务

在整本书中，我们一直在专注于编写应用程序，其中我们控制应用程序运行的基础设施以及数据的物理存储位置。在过去的几年里，趋势是摆脱这种类型的应用程序，转向其他公司通过所谓的**基于云的服务**提供这种基础设施的模式。*云服务*已经成为一个用来描述使用其他公司的按需服务的总称营销术语，依赖于它们提供应用程序功能、安全性、扩展性、备份功能等。其背后的想法是，我们可以通过让其他人为我们处理这些功能来减少资本成本，从而使我们能够编写利用这些功能的应用程序。

在本章中，我们将研究如何使用微软和谷歌的基于云的服务，因此我们将研究注册这些服务的过程，使用它们的含义，以及如何在我们最终的 Angular 应用程序中使用它们。

# 项目概述

对于我们最后的 Angular 应用程序，我们将使用必应地图服务来展示我们日常使用的地图类型，以搜索位置。我们将进一步使用微软的本地洞察服务来搜索当前可见地图区域内的特定业务类型。这是我在为这本书制定计划时最激动人心的两个应用程序之一，因为我对基于地图的系统情有独钟。

除了显示地图，我们还可以通过直接点击地图上的点来选择地图上的兴趣点。这些点将由彩色图钉表示。我们将保存这些点的位置和名称，以及它们在谷歌的基于云的数据库中。

这个应用程序应该需要大约一个小时来完成，只要你在 GitHub 上的代码旁边工作。

在本章中，我们将不再提供如何使用`npm`添加软件包，或者如何创建 Angular 应用程序、组件等的详细信息，因为到这个时候你应该已经熟悉如何做这些了。

完成后，应用程序应该看起来像这样（也许不要放大到纽卡斯尔）：

![](img/a0f765d2-db52-41cc-88eb-8488d2516510.png)

# 开始使用 Angular 中的必应地图

这是我们最后一个 Angular 应用程序，所以我们将以与之前章节中创建应用程序相同的方式开始。同样，我们将使用 Bootstrap，而不是 Angular Material。

我们在本章中要专注的包如下：

+   `bootstrap`

+   `bingmaps`

+   `firebase`

+   `guid-typescript`

由于我们将把我们的代码连接到基于云的服务，我们首先必须注册它们。在本节中，我们将看看我们需要做什么来注册。

# 注册必应地图

如果我们想要使用必应地图，我们必须注册必应地图服务。导航到[`www.bingmapsportal.com`](https://www.bingmapsportal.com)并单击“登录”按钮。这需要一个 Windows 帐户，所以如果你没有一个，你需要设置一个。现在，我们假设你有一个 Windows 帐户可用：

![](img/082d18cf-f170-4e4b-b7e7-0efef3da47b2.png)

当我们登录时，我们需要创建一个密钥，我们的应用程序将使用它来向必应地图服务标识自己，以便他们知道我们是谁，并可以跟踪我们的地图使用情况。从“我的帐户”选项中，选择“我的密钥”：

![](img/871b25e0-38ce-4aef-a2e5-975fe4c172d0.png)

当密钥屏幕出现时，你会看到一个名为“点击此处创建新密钥”的链接。点击链接将显示以下屏幕：

![](img/c03c1b67-b41c-45c7-977e-d4058f3f05ff.png)

这个屏幕上的大部分信息都相当容易理解。应用程序名称用于在我们有多个密钥并且需要搜索它们时使用。URL 不需要设置，但如果我部署到不同的 Web 应用程序，我喜欢这样做。这是一个方便的方式来记住哪个密钥与哪个应用程序相关联。由于我们不打算使用付费企业服务，我们唯一可用的密钥类型是基本的。

应用程序类型可能是这里最重要的字段，从我们的角度来看。我们可以选择多种应用程序类型，每种类型都有关于它可以接受的交易数量的限制。我们将坚持使用 Dev/Test，它限制我们在一年的时间内累计的可计费交易次数为 125,000 次。

当我们在本章中使用本地洞察代码时，这将生成可计费的交易。如果你不想承担任何费用的风险，我建议你禁用执行此搜索的代码。

当我们点击“创建”时，我们的地图密钥被创建，并且可以通过点击表中出现的“显示密钥”或“复制密钥”链接来获取。现在我们已经设置好了地图密钥所需的一切，让我们继续注册数据库。

# 注册 Firebase

Firebase 需要一个 Google 帐户。假设我们有一个可用的 Google 帐户，我们可以在[`console.firebase.google.com/`](https://console.firebase.google.com/)上访问 Firebase 的功能。当出现这个屏幕时，点击“添加项目”按钮开始添加 Firebase 支持的过程：

![](img/1d9dd8ac-8931-4642-88ce-77e2c20ff855.png)

为项目选择一个有意义的名称。在我们创建项目之前，我们应该阅读使用 Firebase 的条款和条件，并在同意时勾选复选框。请注意，如果我们选择共享 Google Analytics 的使用统计数据，我们应该阅读适当的条款和条件，并勾选控制器-控制器条款复选框：

![](img/ae904ea1-6b3a-4983-8162-a7d375eb5e19.png)

点击“创建项目”后，我们现在可以访问 Firebase 项目。虽然 Firebase 作为云服务提供商不仅仅是一个数据库，还提供存储、托管等功能，但我们只是使用数据库选项。当我们点击数据库链接时，会出现 Cloud Firestore 屏幕，我们需要点击“创建数据库”来开始创建数据库的过程：

![](img/f550f6f4-904d-429c-b822-81af8bd6ac94.png)

每当我在本章中提到 Firebase 时，我是在简单地说这是 Firebase 云平台的 Firestore 功能。

在创建数据库时，我们需要选择要应用于我们的数据库的安全级别。我们在这里有两个选项。我们可以从数据库被锁定开始，以便禁用读写。然后，通过编写数据库将检查以确定是否允许写入的规则来启用对数据库的访问。

然而，为了我们的目的，我们将以测试模式开始，这允许对数据库进行无限读写：

![](img/78fe0e8c-0642-4993-a5c7-305d5fdbd2d7.png)

与 Bing 地图类似，Firebase 有使用限制和成本影响。我们正在创建一个 Spark 计划数据存储，这是免费的 Firebase 版本。这个版本有硬性限制，比如每月只能存储 1GB 的数据，每天可以读取 50000 次，每天可以写入 20000 次。有关定价和限制的详细信息，请阅读[`firebase.google.com/pricing`](https://firebase.google.com/pricing)/。

一旦我们点击了启用并有一个可用的数据库，我们需要能够访问 Firebase 为我们创建的密钥和项目详细信息。要找到这些信息，请点击菜单上的项目概述链接。按钮弹出一个屏幕，显示我们需要复制到我们的项目的详细信息：

![](img/85b83eb6-5b9f-44b1-a89c-344c0bca061e.png)

我们现在已经设置好了云基础设施，并且有了我们需要的密钥和详细信息。我们现在准备编写我们的应用程序。

# 使用 Angular 和 Firebase 创建 Bing Maps 应用程序

在过去几年中，增长最快的应用程序类型之一是地图应用程序的爆炸，无论是用于您的卫星导航系统还是在手机上运行 Google 地图。在这些应用程序的底层，有由微软或谷歌等公司开发的地图服务。我们将使用 Bing 地图服务来为我们的应用程序添加地图支持。

我们的地图应用程序有以下要求：

+   点击位置将把该位置添加为兴趣点

+   添加兴趣点时，将显示一个信息框，显示有关它的详细信息

+   再次点击兴趣点将删除它

+   兴趣点将被保存到数据库中

+   用户将能够移动兴趣点，更新数据库中的详细信息

+   在可用的情况下，将自动检索并显示商业信息

# 添加地图组件

我们将为这一步创建两个 Angular 组件，一个叫做`MappingcontainerComponent`，另一个叫做`MapViewComponent`。

我将它们分开，因为我想使用`MappingcontainerComponent`来包含引导程序基础设施，而`MapViewComponent`将只包含地图本身。如果你愿意，你可以将它们合并在一起，但是为了清晰地描述每个部分的情况，对我来说在这里创建两个组件更容易。这意味着我们需要在这两个组件之间引入一些协调，这将加强我们在第五章中介绍的`EventEmitter`行为，*Angular ToDo App with GraphQL and Apollo*。

在为这些组件添加任何内容之前，我们需要编写一些模型和服务，以提供我们的地图和数据访问所需的基础设施。

# 兴趣点

每个兴趣点都由一个图钉表示，并且可以表示为纬度和经度坐标，以及它的名称。

纬度和经度是地理术语，用于准确标识地球上的位置。纬度告诉我们某物距赤道有多远，纬度为 0。这意味着正数表示我们在赤道以北，负数表示我们在赤道以南。经度告诉我们我们距离地球的中心线有多远，按照惯例，这条线穿过伦敦的格林威治。同样，如果我们向东移动，数字是正数，而从格林威治线向西移动意味着数字将是负数。

表示此模型如下所示：

```ts
export class PinModel {
  id: string;
  lat: number;
  long: number;
  name: string;
}
```

在本节中，我们将引用图钉和兴趣点。它们都代表同一件事，因此我们将交替使用它们。

当我们创建一个实例时，我们将使用 GUID 来表示它。由于 GUID 是唯一的，我们将其用作查找兴趣点的便捷方式。这并不是我们将在数据库中存储模型的确切表示，因为此标识符旨在用于跟踪地图上的图钉，而不是用于跟踪数据库中的图钉。为此，我们将添加一个单独的模型，用于在数据库中存储模型项：

```ts
export interface PinModelData extends PinModel {
 storageId: string;
}
```

我们将其创建为接口，因为 Firebase 只希望接收数据，而不希望有围绕它的类基础设施。我们也可以将`PinModel`创建为接口，但是实例化它的语法稍微麻烦一些，这就是为什么我们选择将其创建为类的原因。

有了这些模型，我们现在准备连接到 Firebase。我们将使用官方的 Angular Firebase 库`AngularFire`，而不是直接使用 Firebase 的`npm`。这个库的`npm`引用是`@angular/fire`。

当我们设置我们的 Firebase 数据存储时，我们得到了需要创建一个唯一标识连接的设置。我们将把这些设置复制到我们的`environment.ts`和`environment.prod.ts`文件中。当我们将应用程序发布到生产环境时，Angular 会将`environment.prod.ts`重新映射到环境文件，以便我们可以拥有单独的开发和生产设置：

```ts
firebase: {
  apiKey: "AIzaSyC0MzFxTtvt6cCvmTGE94xc5INFRYlXznw",
  authDomain: "advancedtypescript3-mapapp.firebaseapp.com",
  databaseURL: "https://advancedtypescript3-mapapp.firebaseio.com",
  projectId: "advancedtypescript3-mapapp",
  storageBucket: "advancedtypescript3-mapapp.appspot.com",
  messagingSenderId: "6102469443"
}
```

通常不建议在开发和生产系统中使用相同的端点，因此您可以创建一个单独的 Firebase 实例来保存生产映射信息，并将其存储在`environment.prod.ts`中。

在`app.module`中，我们将导入`AngularFire`模块，然后在导入中引用它们。当我们引用`AngularFireModule`时，我们调用静态的`initializeApp`方法，该方法将使用`environment.firebase`设置来建立与 Firebase 的连接。

首先，`import`语句如下：

```ts
import { AngularFireModule } from '@angular/fire';
import { AngularFirestoreModule } from '@angular/fire/firestore';
import { AngularFireStorageModule } from '@angular/fire/storage';
```

接下来，我们设置 Angular 的`imports`：

```ts
imports: [
  BrowserModule,
  HttpClientModule,
  AngularFireModule.initializeApp(environment.firebase),
  AngularFireStorageModule,
  AngularFirestoreModule
],
```

对于 Firebase 的功能，有一个服务作为与数据库交互的单一实现点是有帮助的。这就是为什么我们将创建一个`FirebaseMapPinsService`：

```ts
export class FirebaseMapPinsService {
}
```

在这个类中，我们将使用`AngularFire`的一个功能，称为`AngularFirestoreCollection`。Firebase 公开了`Query`和`CollectionReference`类型，以对数据库中的基础数据执行 CRUD 操作。`AngularFirestoreCollection`将此行为封装成一个方便的流。我们将通用类型设置为`PinModelData`，以说明将保存到数据库中的数据是什么：

```ts
private pins: AngularFirestoreCollection<PinModelData>;
```

我们的服务将提供一个模型，创建一个`PinModelData`数组的可观察对象，连接到`pins`属性。我们将这一切连接在一起的方式在构造函数中，该构造函数接收`AngularFirestore`。通过传递将存储在数据库中的集合名称，`pins`集合与底层集合相关联（将数据保存为 JSON 文档）。我们的`Observable`监听集合上的`valueChanges`，如下所示：

```ts
constructor(private readonly db: AngularFirestore) { 
  this.pins = db.collection<PinModelData>('pins');
  this.model = this.pins.valueChanges();
}
```

在设计这个应用程序时，我做出的一个决定是，从 UI 中删除标记应该导致从数据库中删除相关的兴趣点。由于它没有被任何其他东西引用，我们不需要将其保留为引用数据。删除数据就像使用`doc`从数据存储中获取基础文档记录一样简单，然后将其删除：

```ts
Delete(item: PinModelData) {
  this.pins.doc(item.storageId).delete();
}
```

当用户添加一个兴趣点时，我们希望在数据库中创建相应的条目，但当他们移动标记时，我们希望更新记录。我们可以将逻辑合并到一个方法中，因为我们知道一个具有空`storageId`的记录之前没有保存到数据库中。因此，我们使用 Firebase 的`createId`方法为其提供一个唯一的 ID。如果`storageId`存在，那么我们就要更新它：

```ts
Save(item: PinModelData) {
  if (item.storageId === '') {
    item.storageId = this.db.createId();
    this.pins.doc(item.storageId).set(item);
  }
  else {
    this.pins.doc(item.storageId).update(item);
  }
}
```

# 表示地图标记

我们可以很好地将标记保存到数据库中，但我们还需要一种方法来表示地图上的标记，以便在地图会话期间显示它们并根据需要移动它们。这个类还将作为与数据服务的连接。我们将要编写的类将演示 TypeScript 3 中引入的一个巧妙的小技巧，称为**rest tuples**，并且起始如下：

```ts
export class PinsModel {
  private pins: PinModelData[] = [];
  constructor(private firebaseMapService: FirebaseMapService) { }
}
```

我们要引入的第一个功能涉及在用户点击地图时添加标记的数据。这个方法的签名看起来有点奇怪，所以我们将花一两分钟来解释它是如何工作的。签名看起来像这样：

```ts
public Add(...args: [string, string, ...number[]]);
```

当我们看到`...args`作为最后（或唯一）参数时，我们立刻想到的是我们将使用 REST 参数。如果我们从开始就分解参数列表，我们可以将其看作是这样开始的：

```ts
public Add(arg_1: string, arg_2: string, ...number[]);
```

这几乎看起来是有道理的，但在那里还有另一个 REST 参数。这基本上意味着我们可以在元组的末尾有任意数量的数字。我们必须对此应用`...`，而不仅仅是应用`number[]`，是因为我们需要展开元素。如果我们只使用数组格式，我们将不得不在调用代码中将元素推入这个数组。有了元组中的 REST 参数，我们可以取出数据，保存到数据库中，并将其添加到我们的`pins`数组中，就像这样：

```ts
public Add(...args: [string, string, ...number[]]) {   const data: PinModelData = {   id: args[0],   name: args[1],   lat: args[2],   long: args[3],   storageId: ''   };   this.firebaseMapService.Save(data);   this.pins.push(data);  }
```

使用这样的元组的含义是，调用代码必须确保将值放入正确的位置。

当我们到达调用这个代码的地方时，我们可以看到我们的方法是这样调用的：

```ts
this.pinsModel.Add(guid.toString(), geocode, e.location.latitude, e.location.longitude);
```

当用户在地图上移动标记时，我们将使用类似的技巧来更新其位置。我们所需要做的就是在数组中找到模型并更新其数值。我们甚至需要更新名称，因为移动标记的行为将改变标记的地址。我们在数据服务上调用相同的`Save`方法，就像我们在`Add`方法中所做的那样：

```ts
public Move(...args: [string,string, ...number[]]) {   const pinModel: PinModelData = this.pins.find(x => x.id === args[0]);   if (pinModel) {   pinModel.name = args[1];   pinModel.lat = args[2];   pinModel.long = args[3];  }   this.firebaseMapService.Save(pinModel);  }
```

其他类也需要访问数据库中的数据。我们在这里面临两个选择——我们可以让其他类也使用 Firebase 地图服务，并且可能错过对这个类的调用，或者我们可以使这个类成为地图服务的唯一访问点。我们将依赖这个类成为与`FirebaseMapPinsService`的唯一联系点，这意味着我们需要通过`Load`方法公开`model`：

```ts
public Load(): Observable<PinModelData[]>{   return this.firebaseMapService.model;  }
```

删除兴趣点使用的方法签名比添加或移动兴趣点简单得多。我们只需要记录的客户端端`id`，然后使用它来找到`PinModelData`项目并调用`Delete`从 Firebase 中删除。一旦我们删除了记录，我们就会找到这条记录的本地索引，并通过对数组进行拼接来删除它：

```ts
public Remove(id: string) {
  const pinModel: PinModelData = this.pins.find(x => x.id === id);
  this.firebaseMapService.Delete(pinModel);
  const index: number = this.pins.findIndex(x => x.id === id);
  if (index >= 0) {
    this.pins.splice(index,1);
  }
}
```

# 尝试有趣的地图搜索

当涉及到获取用户放置或移动图钉的位置名称时，我们希望这是自动发生的。我们真的不希望用户在映射时必须手动输入这个值，映射可以自动为我们选择。这意味着我们将不得不使用映射功能来为我们获取这些信息。

必应地图有许多可选模块，我们可以选择使用，这些模块使我们能够进行基于位置的搜索等操作。为了做到这一点，我们将创建一个名为`MapGeocode`的类，它将为我们进行搜索：

```ts
export class MapGeocode {
}
```

您可能注意到，对于我们的一些类，我们是在没有创建服务的情况下创建它们的。这意味着我们将不得不手动实例化这个类。这没问题，因为我们可以手动控制我们类的生命周期。如果你愿意，在重新创建代码时，你可以将`MapGeocode`等类转换为服务并注入它。

由于搜索是一个可选功能，我们需要加载它。为此，我们将传入我们的地图并使用`loadModule`来加载`Microsoft.Maps.Search`模块，传入`SearchManager`的新实例作为选项：

```ts
private searchManager: Microsoft.Maps.Search.SearchManager;
constructor(private map: Microsoft.Maps.Map) {
  Microsoft.Maps.loadModule('Microsoft.Maps.Search', () => {
    this.searchManager = new Microsoft.Maps.Search.SearchManager(this.map);
  });
}
```

我们要做的所有事情就是编写一个执行查找的方法。由于这可能是一个耗时的操作，我们需要将其设置为`Promise`类型，返回将被填充为名称的字符串。在这个`Promise`中，我们创建一个包含位置的请求和一个回调，当`reverseGeocode`方法执行时，将使用位置的名称更新`Promise`中的回调。有了这个，我们调用`searchManager.reverseGeocode`来执行搜索：

```ts
public ReverseGeocode(location: Microsoft.Maps.Location): Promise<string> {
  return new Promise<string>((callback) => {
    const request = {
      location: location,
      callback: function (code) { callback(code.name); }
    };
    if (this.searchManager) {
      this.searchManager.reverseGeocode(request);
    }
  });
}
```

在编码中，名称很重要。在地图制作中，当我们进行地理编码时，我们将物理地址转换为位置。将位置转换为地址的行为称为**反向地理编码**。这就是为什么我们的方法有一个相当繁琐的名字`ReverseGeocode`。

还有另一种类型的搜索需要考虑。我们希望进行一种使用可见地图区域（视口）来识别该区域内的咖啡店的搜索。为此，我们将使用微软的新 Local Insights API 来搜索特定区域内的企业等内容。目前这种实现有一个限制，即 Local Insights 仅适用于美国地址，但计划在其他国家和地区推出此功能。

为了证明我们仍然可以在服务中使用地图，我们将创建一个`PointsOfInterestService`，它接受一个`HttpClient`，我们将使用它来获取 REST 调用的结果：

```ts
export class PointsOfInterestService {
  constructor(private http: HttpClient) {}
}
```

REST 调用端点接受一个查询，告诉我们我们感兴趣的企业类型，用于执行搜索的位置以及地图密钥。同样，我们的搜索功能可能是长时间运行的，所以我们将返回一个`Promise`，这次是一个自定义的`PoiPoint`，返回纬度和经度，以及企业的名称：

```ts
export interface PoiPoint {
  lat: number,
  long: number,
  name: string
}
```

当我们调用 API 时，我们将使用`http.get`，它返回一个 observable。我们将使用`pipe`和`map`来使用`MapData`对结果进行转换。我们将订阅结果并解析结果（注意我们并不真正知道返回类型，所以我们将其留空为`any`）。返回类型可以包含多个`resourceSets`，大多用于一次性进行多种类型的查询，但我们只需要关注初始的`resourceSet`，然后用它来提取资源。以下代码显示了我们从这次搜索中感兴趣的元素的格式。当我们完成解析结果后，我们将取消订阅搜索订阅，并在`Promise`上回调刚刚添加的点：

```ts
public Search(location: location): Promise<PoiPoint[]> {
  const endpoint = `https://dev.virtualearth.net/REST/v1/LocalSearch/?query=coffee&userLocation=${location[0]},${location[1]}&key=${environment.mapKey}`;
  return new Promise<PoiPoint[]>((callback) => {
    const subscription: Subscription = this.http.get(endpoint).pipe(map(this.MapData))
    .subscribe((x: any) => {
      const points: PoiPoint[] = [];
      if (x.resourceSets && x.resourceSets.length > 0 && x.resourceSets[0].resources) {
        x.resourceSets[0].resources.forEach(element => {
          if (element.geocodePoints && element.geocodePoints.length > 0) {
            const poi: PoiPoint = {
              lat: element.geocodePoints[0].coordinates[0],
              long: element.geocodePoints[0].coordinates[1],
              name: element.name
            };
            points.push(poi)
          }
        });
      }
      subscription.unsubscribe();
      callback(points);
    })
  });
}
```

在我们的查询中，我们只是在一个点上搜索——如果需要的话，我们可以很容易地扩展到在我们的视图范围内搜索一个边界框，方法是接受地图边界框并将`userLocation`更改为`userMapView=${boundingBox{0}},${boundingBox{1}},${boundingBox{2}},${boundingBox{3}}`（其中`boundingBox`是一个矩形）。有关扩展搜索的更多细节，请参见[`docs.microsoft.com/en-us/previous-versions/mt832854(v=msdn.10)`](https://docs.microsoft.com/en-us/previous-versions/mt832854(v=msdn.10))。

现在我们已经完成了地图搜索功能和数据库功能，是时候在屏幕上实际放置地图了。让我们现在来处理这个问题。

# 将 Bing 地图添加到屏幕上

就像我们之前讨论的那样，我们将使用两个组件来显示地图。让我们从`MapViewComponent`开始。这个控件的 HTML 模板非常简单：

```ts
<div #myMap style='width: 100%; height: 100%;'> </div> 
```

是的，这确实是我们的 HTML 的全部内容。它背后发生的事情要复杂一些，这就是我们将学习 Angular 如何让我们连接到标准 DOM 事件的地方。我们通常不显示整个`@Component`元素，因为它几乎是样板代码，但在这种情况下，我们将不得不做一些稍微不同的事情。这是我们组件的第一部分：

```ts
@Component({
  selector: 'atp-map-view',
  templateUrl: './map-view.component.html',
  styleUrls: ['./map-view.component.scss'],
  host: {
  '(window:load)' : 'Loaded()'
  } }) export class MapViewComponent implements OnInit {
  @ViewChild('myMap') myMap: { nativeElement: string | HTMLElement; };    constructor() { }    ngOnInit() {  }
}
```

在`@Component`部分，我们将窗口加载事件挂钩到`Loaded`方法。我们很快会添加这个方法，但现在知道这是我们如何将组件挂钩到主机事件的方式很重要。在组件内部，我们使用`@ViewChild`来挂钩到我们模板中的`div`。基本上，这允许我们通过名称引用视图内的元素，以便我们可以以某种任意的方式处理它。

我们添加`Loaded`方法的原因是因为 Bing 地图有一个特别讨厌的习惯，即在 Chrome 或 Firefox 等浏览器中不正常工作，除非我们在`window.load`事件中挂接地图。我们将在模板中添加一个`div`语句来托管地图，使用一系列地图加载选项，包括地图凭据和默认缩放级别：

```ts
Loaded() {   // Bing has a nasty habit of not working properly in browsers like 
  // Chrome if we don't hook the map up 
 // in the window.load event.   const map = new Microsoft.Maps.Map(this.myMap.nativeElement, {   credentials: environment.mapKey,   enableCORS: true,   zoom: 13   });
  this.map.emit(map);
}
```

如果我们想选择特定类型的地图类型来显示，我们可以在地图加载选项中设置如下：

```ts
mapTypeId:Microsoft.Maps.MapTypeId.road
```

我们的`MapViewComponent`将托管在另一个组件内部，因此我们将创建一个`EventEmitter`，我们可以用它来通知父组件。我们已经在我们的`Loaded`方法中添加了发射代码，将刚加载的地图传回给父组件：

```ts
@Output() map = new EventEmitter();
```

现在让我们添加父容器。大部分模板只是用来创建带有行和列的 Bootstrap 容器。在`div`列内，我们将托管刚刚创建的子组件。同样，我们可以看到我们使用了`EventEmitter`，所以当地图被发射时，它触发`MapLoaded`事件：

```ts
<div class="container-fluid h-100">
 <div class="row h-100">
 <div class="col-12">
 <atp-map-view (map)="MapLoaded($event)"></atp-map-view>
 </div>
 </div> </div>
```

大多数映射容器代码现在应该是我们熟悉的领域。我们注入`FirebaseMapPinsService`和`PointsOfInterestService`，我们用它们在`MapLoaded`方法中创建`MapEvents`实例。换句话说，当`atp-map-view`组件触发`window.load`时，填充的 Bing 地图就会回来：

```ts
export class MappingcontainerComponent implements OnInit {   private map: Microsoft.Maps.Map;
  private mapEvents: MapEvents;
  constructor(private readonly firebaseMapPinService: FirebaseMapPinsService, 
private readonly poi: PointsOfInterestService) { }    ngOnInit() {
 }    MapLoaded(map: Microsoft.Maps.Map) {
  this.map = map;
  this.mapEvents = new MapEvents(this.map, new PinsModel(this.firebaseMapPinService), this.poi);
 } }
```

关于显示地图的说明——我们确实需要设置`html`和`body`的高度，以使其延伸到浏览器窗口的全高。在`styles.scss`文件中设置如下：

```ts
html,body {
  height: 100%; }
```

# 地图事件和设置标记

我们有地图，我们有逻辑来将兴趣点保存到数据库并在内存中移动它们。我们唯一没有的是处理用户实际从地图本身创建和管理标记的代码。现在是时候纠正这种情况并添加一个`MapEvents`类来为我们处理这个问题。就像`MapGeocode`、`PinModel`和`PinsModel`类一样，这个类是一个独立的实现。让我们从添加以下代码开始：

```ts
export class MapEvents {
  private readonly geocode: MapGeocode;
  private infoBox: Microsoft.Maps.Infobox;

  constructor(private map: Microsoft.Maps.Map, private pinsModel: PinsModel, private poi: PointsOfInterestService) {

  }
}
```

`Infobox`是在将兴趣点添加到屏幕上时出现的框。我们可以在添加每个兴趣点时添加一个新的，但这将是一种资源浪费。相反，我们将添加一个单独的`Infobox`，并在添加新点时重用它。为此，我们将添加一个辅助方法，检查之前是否已设置`Infobox`。如果之前没有设置，我们将实例化`Infobox`的新实例，输入图钉位置、标题和描述。我们将使用`setMap`来设置此`Infobox`将出现在的地图实例。当我们重用这个`Infobox`时，我们只需要在选项中设置相同的值，然后将可见性设置为`true`：

```ts
private SetInfoBox(title: string, description: string, pin: Microsoft.Maps.Pushpin): void {
  if (!this.infoBox) {
    this.infoBox = new Microsoft.Maps.Infobox(pin.getLocation(), { title: title, description: description });
    this.infoBox.setMap(this.map);
  return;
  }
  this.infoBox.setOptions({
    title: title,
    description: description,
    location: pin.getLocation(),
    visible: true
  });
}
```

在我们添加从地图中选择点的能力之前，我们还需要向这个类添加一些辅助方法。我们要添加的第一个方法是从本地见解搜索中获取兴趣点并将它们添加到地图上。在这里，我们可以看到我们添加图钉的方式是创建一个绿色的`Pushpin`，然后将其添加到我们的 Bing 地图上的正确`Location`。我们还添加了一个事件处理程序，以响应对图钉的点击，并使用我们刚刚添加的方法显示`Infobox`：

```ts
AddPoi(pois: PoiPoint[]): void {
  pois.forEach(poi => {
    const pin: Microsoft.Maps.Pushpin = new Microsoft.Maps.Pushpin(new Microsoft.Maps.Location(poi.lat, poi.long), {
      color: Microsoft.Maps.Color.fromHex('#00ff00')
    });
    this.map.entities.push(pin);
    Microsoft.Maps.Events.addHandler(pin, 'click', (x) => {
      this.SetInfoBox('Point of interest', poi.name, pin);
    });
  })
}
```

下一个辅助方法更复杂，所以我们将分阶段添加它。当用户在地图上单击时，将调用`AddPushPin`代码。签名如下：

```ts
AddPushPin(e: any): void {
}
```

在这个方法中，我们要做的第一件事是创建一个`Guid`，用于在添加`PinsModel`条目时使用，并在点击位置添加一个可拖动的`Pushpin`：

```ts
const guid: Guid = Guid.create();
const pin: Microsoft.Maps.Pushpin = new Microsoft.Maps.Pushpin(e.location, {
  draggable: true
});
```

有了这个方法，我们将调用之前编写的`ReverseGeocode`方法。当我们从中获取结果时，我们将添加我们的`PinsModel`条目，并在显示`Infobox`之前将`Pushpin`推到地图上：

```ts
this.geocode.GeoCode(e.location).then((geocode) => {
  this.pinsModel.Add(guid.toString(), geocode, e.location.latitude, e.location.longitude);
  this.map.entities.push(pin);
  this.SetInfoBox('User location', geocode, pin);
});
```

我们还没有完成这个方法。除了添加一个`Pushpin`，我们还必须能够拖动它，以便用户在拖动图钉时选择一个新的位置。我们将使用`dragend`事件来移动图钉。同样，我们之前付出的辛苦工作得到了回报，因为我们有一个简单的机制来`Move` `PinsModel`并显示我们的`Infobox`：

```ts
const dragHandler = Microsoft.Maps.Events.addHandler(pin, 'dragend', (args: any) => {
  this.geocode.GeoCode(args.location).then((geocode) => {
    this.pinsModel.Move(guid.toString(), geocode, args.location.latitude, args.location.longitude);
    this.SetInfoBox('User location (Moved)', geocode, pin);
  });
});
```

最后，当用户点击图钉时，我们希望从`PinsModel`和地图中删除图钉。当我们为`dragend`和`click`添加事件处理程序时，我们将处理程序保存到变量中，以便我们可以使用它们从地图事件中删除事件处理程序。自我整理是一个好习惯，特别是在处理事件处理程序之类的事情时：

```ts
const handler = Microsoft.Maps.Events.addHandler(pin, 'click', () => {
  this.pinsModel.Remove(guid.toString());
  this.map.entities.remove(pin);

  // Tidy up our stray event handlers.
  Microsoft.Maps.Events.removeHandler(handler);
  Microsoft.Maps.Events.removeHandler(dragHandler);
});
```

好了，我们的辅助方法已经就位。现在我们只需要更新构造函数，以便在地图上单击以设置兴趣点并在用户查看的视口发生变化时搜索本地见解。让我们从响应用户在地图上单击开始：

```ts
this.geocode = new MapGeocode(this.map);
Microsoft.Maps.Events.addHandler(map, 'click', (e: any) => {
  this.AddPushPin(e);
});
```

在这里，我们不需要将处理程序存储为变量，因为我们将其与在浏览器中运行时不会被移除的东西关联起来，即地图本身。

当用户移动地图以便查看其他区域时，我们需要执行本地见解搜索，并根据返回的结果添加兴趣点。我们将事件处理程序附加到地图`viewchangeend`事件以触发此搜索：

```ts
Microsoft.Maps.Events.addHandler(map, 'viewchangeend', () => {
  const center = map.getCenter();
  this.poi.Search([center.latitude, center.longitude]).then(pointsOfInterest => {
    if (pointsOfInterest && pointsOfInterest.length > 0) {
      this.AddPoi(pointsOfInterest);
    }
  })
})
```

我们不断看到事先准备方法可以节省我们很多时间。我们只是利用`PointsOfInterestService.Search`方法来进行本地见解搜索，然后将结果传递给我们的`AddPoi`方法。如果我们不想执行本地见解搜索，我们可以简单地删除此事件处理程序，而无需进行任何搜索。

我们唯一剩下要做的就是处理从数据库加载我们的标记。这里的代码是我们已经看到的用于添加`click`和`dragend`处理程序的代码的变体，但我们不需要执行地理编码，因为我们已经有了每个兴趣点的名称。因此，我们不打算重用`AddPushPin`方法。相反，我们将选择在整个部分内联执行。加载订阅如下所示：

```ts
const subscription = this.pinsModel.Load().subscribe((data: PinModelData[]) => {
  data.forEach(pinData => {
    const pin: Microsoft.Maps.Pushpin = new Microsoft.Maps.Pushpin(new Microsoft.Maps.Location(pinData.lat, pinData.long), {
      draggable: true
    });
    this.map.entities.push(pin);
    const handler = Microsoft.Maps.Events.addHandler(pin, 'click', () => {
      this.pinsModel.Remove(pinData.id);
      this.map.entities.remove(pin);
    Microsoft.Maps.Events.removeHandler(handler);
      Microsoft.Maps.Events.removeHandler(dragHandler);
    });
    const dragHandler = Microsoft.Maps.Events.addHandler(pin, 'dragend', (args: any) => {
      this.geocode.GeoCode(args.location).then((geocode) => {
        this.pinsModel.Move(pinData.id, geocode, args.location.latitude, args.location.longitude);
        this.map.entities.push(pin);
    this.SetInfoBox('User location (moved)', geocode, pin);
      });
    });
  });
  subscription.unsubscribe();
  this.pinsModel.AddFromStore(data);
});
```

需要注意的是，由于我们正在处理订阅，一旦完成订阅，我们就会从中`取消订阅`。订阅应返回一个`PinModelData`项目数组，我们可以遍历并根据需要添加元素。

就是这样。我们现在已经有了一个可用的映射解决方案。这是我最期待写的章节之一，因为我喜欢映射应用程序。我希望你和我一样享受这个过程。然而，在我们离开这一章之前，如果你想防止人们未经授权访问数据，你可以在下一节中应用这些知识。

# 保护数据库

这一部分是提供数据库安全性所需的可选概述。您可能还记得，当我们创建 Firestore 数据库时，我们设置了访问权限，以便任何人都可以完全不受限制地访问。在开发小型测试应用程序时这没问题，但通常不适用于商业应用程序的部署。

我们将更改数据库的配置，以便只有在授权 ID 设置时才允许读/写访问。为此，请在数据库中选择“规则”选项卡，并将`if request.auth.uid != null;`添加到规则列表中。`match /{document=**}`的格式简单地意味着这个规则适用于列表中的任何文档。可以设置只适用于特定文档的规则，但在这样的应用程序环境中并没有太多意义。

请注意，这样做意味着我们必须添加身份验证，就像我们在第六章中所做的那样，*使用 Socket.IO 构建聊天室应用程序*。设置这一点超出了本章的范围，但从上一章复制导航并提供登录功能应该很简单：

![](img/7bd73cd5-8073-493f-aefe-9725e47bee1c.png)

这是一段相当漫长的旅程。我们经历了注册不同在线服务的过程，并将映射功能引入了我们的代码。与此同时，我们还看到了如何使用 TypeScript 支持在 Angular 应用程序中搭建脚手架，而无需生成和注册服务。现在，您应该能够拿起这段代码，并尝试添加您真正想要的映射功能。

# 摘要

在本章中，我们已经完成了使用 Microsoft 和 Google 的云服务引入 Angular 项目的工作，这些云服务以 Bing Maps 和 Firebase 云服务的形式存储数据。我们注册了这些服务，并从中获取了相关信息，以便为客户端访问它们。在编写代码的过程中，我们创建了与 Firestore 数据库一起工作的类，并与 Bing Maps 交互，执行诸如基于用户点击搜索地址、在地图上添加标记以及使用本地洞察力搜索咖啡店等操作。

继续我们的 TypeScript 之旅，我们介绍了 rest 元组。我们还看到如何向 Angular 组件添加代码以响应浏览器主机事件。

在下一章中，我们将重新审视 React。这一次，我们将创建一个使用 Docker 包含各种微服务的有限微服务 CRM。

# 问题

1.  Angular 如何允许我们与主机元素交互？

1.  纬度和经度是什么？

1.  逆地理编码的目的是什么？

1.  我们使用哪项服务来存储我们的数据？
