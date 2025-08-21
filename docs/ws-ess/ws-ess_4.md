# 第四章：在真实场景中使用 WebSockets

我们在上一章中看到了如何创建一个实时演示共享应用程序。我们了解了实时数据传输的工作原理以及如何设置服务器。现在我们将进入下一步，看看我们需要添加哪些元素来加强我们的应用程序的结构。在这一章中，我们将看到创建应用程序的不同步骤。

# 真实场景

这里的问题是什么是真实场景？我们已经看到了一个真实世界的场景应用程序，但是这里我们指的是什么？一个结构良好的应用程序如果没有框架支持是不完整的。在上一个应用程序中，我们使用了 JavaScript 服务器和 JavaScript 库，进行了集成并构建了我们的应用程序。但是你认为应用程序的结构足够好以支持可扩展性或可重用性吗？答案是否定的，因为我们没有使用任何框架来为我们的应用程序提供更好的结构。在这一章中，让我们谈谈实际场景，我们在应用程序中实现不同的结构或框架。

# JavaScript 框架

随着 HTML5 的发展，JavaScript 框架开始进入视野。情况是我们有很多选择。一些常用的框架包括 AngularJS、Ember.js、Knockout.js、Backbone.js 等等。我们将在下一个示例中使用 AngularJS。AngularJS 是由谷歌开发的，是一个功能强大的框架，具有许多必要的功能。

## AngularJS

AngularJS 是由谷歌开发的开源框架。它基于一个非常著名的设计模式：**Model-View-ViewModel**（**MVVM**）。除此之外，它还提供了与 HTML5 无缝配合的功能，如指令、绑定和控制器。它主要处理单页应用程序的问题，提供了实现动态视图和路由机制的功能，以简化页面之间的导航，而无需加载完整的网页。这个特性使得这个框架对开发者非常有益。它不仅解决了开发中的问题，还使得测试变得非常容易。

关于 AngularJS 框架在网上有很多详细信息。

# 学以致用

学以致用是学习的最佳方式之一。有时候你会学习一些东西，然后实施它。但是因为你已经阅读了场景，你可以很容易地实施它。最好的方法之一是开始行动，当你面临问题时，尝试找到解决方案。这将提高你的解决问题的能力，并帮助你探索更多。在类似的情况下，让我们从一个应用程序开始，看看我们在哪里遇到问题，以及我们在哪里可以看到需要一个框架。

# 协作绘图应用程序

让我们构建一个绘图应用程序，用户可以在画布上绘图，其他用户也可以同时进行相同的操作。基本上，我们正在创建一个协作绘图应用程序。在构建应用程序之前，让我们收集要求并进行一些分析，这是构建应用程序必需的。

## 要求

在这里，我们的主要要求是需要制作一个提供协作绘图的应用程序。所以我们需要一个客户端应用程序，它连接到服务器，并实时地将数据从一个用户传递到另一个用户。除此之外，我们需要使用 HTML 来绘制一个机制。我们可以使用一个很好的现成库，它为我们提供了绘图所需的功能，而不是花费大量时间编写绘图功能的代码。

因此，如果我们列出构建应用程序所需的项目，它将如下所示：

+   客户端应用程序

+   服务器

+   绘图库

+   实时数据传输的实现

现在我们知道要创建什么。下一步是为应用程序划分任务。

### 绘图库

我们选择使用库而不是编写整个东西。有一些库可用，但其中最好的是**Fabric**.**js**库。你可以从[`fabricjs.com/`](http://fabricjs.com/)下载该库。你甚至可以构建一个自定义的库文件，选择其中的功能以使其更轻量级。这个库提供了许多功能，你可以在上述网站上看到所有这些功能。让我们看一下 Fabric.js 库的演示代码：

```js
<!DOCTYPE html>
<html>
<head>
</head>
<body>
    <canvas id="canvas" width="300" height="300">
....</canvas>

    <script src="img/fabric.js">
....</script>

    <script>
        var canvas = new fabric.Canvas('canvas');
        var rect = new fabric.Rect({
            top : 100,
            left : 100,
            width : 60,
            height : 70,
            fill : 'red'
        });
        canvas.add(rect);
    </script>
</body>
</html>
```

我们可以看到这段代码中这个库是多么简单。你只需要添加画布标签并开始向其中添加对象，它就会在应用程序中显示。这个库非常容易实现，这将帮助我们很多，因为我们已经在这里处理了很多不同的事情。尝试一下这段代码，看看输出结果，并尝试使用这个库来熟悉它。

## 客户端应用程序

第一步是制作一个客户端应用程序。以下是客户端的代码：

```js
<!DOCTYPE html>

<html>

<head>

</head>

<body>

<button id="addCircle">Add Circle</button>

<button id="addRectangle">Add Rectangle</button>

<button id="addTriangle">Add Triangle</button>

<button id="pencil" toggle>Pencil</button>

<button id="selection" toggle>Selection</button>

    <canvas id="canvas" width="1024" height="768"></canvas>

    <script src="img/fabric.js"></script>

    <script>

//creating canvas instance
        var canvas = new fabric.Canvas('canvas');

//setting some properties for canvas

        canvas.freeDrawingBrush.color = 'green';

        canvas.freeDrawingBrush.lineWidth = 10;
        canvas.selectable = false;

        canvas.on('path:created',function(e){

            console.log(JSON.stringify(e));

        })

//main initialize method
        function init()

        {

            pencil.addEventListener('click', pencilHandler);

            addCircle.addEventListener('click', addCircleHandler);

            addRectangle.addEventListener('click', addRectangleHandler);

            addTriangle.addEventListener('click', addTriangleHandler);

            selection.addEventListener('click', function(){

                canvas.isDrawingMode = false;

            })

        }

//changing the drawing mode to free drawing
        function pencilHandler()

        {

            canvas.isDrawingMode = true;

        }

//adding circle to the canvas
        function addCircleHandler()

        {

            var circle = new fabric.Circle({

              radius: 20,

              fill: 'green',

              left: 100,

              top: 100

            });

        canvas.add(circle);

        }

//adding rectangle to the canvas
        function addRectangleHandler()

        {

            var rect = new fabric.Rect({

                top : 100,

                left : 100,

                width : 60,

                height : 70,

                fill : 'red'

            });

            canvas.add(rect);

        }

//adding triangle to the canvas
        function addTriangleHandler()

        {

            var triangle = new fabric.Triangle({

                width: 20,

                height: 30,

                fill: 'blue',

                left: 50,

                top: 50

            });

            canvas.add(triangle);

        }

//adding window load event
        window.addEventListener("load", init, false);

    </script>

</body>

</html>
```

在这段代码中，我们创建了一个画布，并制作了一些按钮来添加不同的形状到画布上。添加的一个重要功能是自由绘制。复制并粘贴代码到`index.html`文件中并尝试运行它。如果你阅读了 Fabric.js 库，你将了解它是如何工作的。不要忘记下载库文件并在代码中包含库。

## 与服务器集成

由于我们已经为基本客户端功能编写了代码，现在我们需要使用 WebSocket 将应用程序与服务器集成。为此，我们首先需要找出需要发送到服务器的数据。为了协作，我们必须发送关于我们需要在其他用户画布上创建的形状的数据。让我们列出我们需要在客户端和服务器端执行的一系列操作。

客户端：

+   捕获添加形状按钮的事件

+   将对象发送到服务器

+   创建与服务器的 WebSocket 连接

+   捕获服务器数据

+   处理并将从服务器接收的对象添加到画布上

服务器：

+   创建一个 WebSocket 服务器

+   接收数据

+   将数据传递给所有连接的客户端

根据前面列出的项目清单，我们已经对服务器端和客户端代码进行了一些更改。

### 客户端代码

现在，我们将根据列出的服务器通信项目在我们的客户端代码中实现代码，这将有代码与服务器通信。以下是根据客户端项目更改的客户端代码：

```js
<!DOCTYPE html>
<html>
  <head>
  </head>
  <body>
    <button id="addCircle">Add Circle</button>
    <button id="addRectangle">Add Rectangle</button>
    <button id="addTriangle">Add Triangle</button>
    <button id="pencil" toggle>Pencil</button>
    <button id="selection" toggle>Selection</button>
    <canvas id="canvas" width="1024" height="768"></canvas>
    <script src="img/fabric.js"></script>
    <script>
      //creating canvas instance
              var canvas = new fabric.Canvas('canvas');

      //setting some properties for canvas
              canvas.freeDrawingBrush.color = 'green';

              canvas.freeDrawingBrush.lineWidth = 10;

              canvas.selectable = false;

              canvas.on('path:created',function(e){

                  console.log(JSON.stringify(e));

              }) 

      //main initialize method
              function init()

              {

                  initServer();

                  pencil.addEventListener('click', pencilHandler);

                  addCircle.addEventListener('click', addCircleHandler);

                  addRectangle.addEventListener('click', 
      addRectangleHandler);

                  addTriangle.addEventListener('click', 
      addTriangleHandler);

                  selection.addEventListener('click', function(){

                      canvas.isDrawingMode = false;

                  })

              }

      //changing the drawing mode to free drawing
              function pencilHandler()

              {

                  canvas.isDrawingMode = true;

              }
      //add circle to the canvas
              function addCircleHandler()

              {

                  var obj = {

                    radius: 20, 

                    fill: 'green',

                    left: 100, 

                    top: 100

                  };

                  // var circle = new fabric.Circle(obj);

                  // canvas.add(circle);
      //sending the circle object to the server
                  sendObject('Circle',obj);

              }

      //add rectangle to the canvas
              function addRectangleHandler()

              {

                  var obj = {

                      top : 100,

                      left : 100,

                      width : 60,

                      height : 70,

                      fill : 'red'

                  };

                  var rect = new fabric.Rect(obj);

                  // canvas.add(rect);
      //sending the rectangle object to the server
                  sendObject('Rectangle',obj);

              }

      //add triangle to the canvas       
       function addTriangleHandler()

              {

                  var obj = {

                      width: 20,

                      height: 30,

                      fill: 'blue',

                      left: 50,

                      top: 50

                  };

                  var triangle = new fabric.Triangle(obj);

                  // canvas.add(triangle);
      //sending the object to server 
                  sendObject('Rectangle',obj);

              }

      //generic method to add object to the canvas
              function addObject(type,obj)

              {

                  var shape;

                  if(type == 'Triangle')

                  {

                      shape = new fabric.Triangle(obj);

                  }

                  else if(type == 'Rectangle')

                  {

                      shape = new fabric.Rect(obj);

                  }

                   else if(type == 'Circle')

                  {

                      shape = new fabric.Circle(obj);

                  }

                  canvas.add(shape);

              }

      //check for JSON string
              function isJson(str)

              {

                  try

                  {

                      JSON.parse(str);

                  }

                  catch (e)

                  {
                      return false;

                  }

                  return true;

              }

              var ws;

      //method to send object to the server
              function sendObject(type,obj)

              {

                  ws.send(JSON.stringify({'type': type,'data' : obj}));

              }

              function connectionOpen()

              {

                   ws.send('connection open');

              }
      //method handler when message is received from server
              function onMessageFromServer(message)

              {

                  console.log('received: '+ message);

                  if(isJson(message.data))

                  {

                      var obj = JSON.parse(message.data)

                      console.log("got data from server");

                      addObject(obj.type,obj.data)

                  }
              }

      //initialize server method
              function initServer()

              {

                  ws = new WebSocket('ws://localhost:9001');

                  ws.onopen = connectionOpen;

                  ws.onmessage = onMessageFromServer;

              }

              window.addEventListener("load", init, false);

    </script>
  </body>
</html>
```

#### 代码解释

我们已经在上一章中看到了如何从 WebSocket 服务器发送和接收数据。这里我们编写了一个发送数据的方法`sendObject`，它将对象的类型和属性发送到服务器：

```js
 function sendObject(type,obj)

        {
            ws.send(JSON.stringify({'type': type,'data' : obj}));
        }
```

这里的一个主要方法是`addObject`。一旦我们从服务器获取数据，我们就会得到两个属性：一个是类型，另一个是具有属性值的对象。这些是我们发送到服务器的值，然后检查对象的类型并使用相应的方法将其添加到画布中：

```js
function addObject(type,obj)

        {

            var shape;

            if(type == 'Triangle')

            {

                shape = new fabric.Triangle(obj);

            }

            else if(type == 'Rectangle')

            {

                shape = new fabric.Rect(obj);

            }

             else if(type == 'Circle')

            {

                shape = new fabric.Circle(obj);

            } 

            canvas.add(shape);

        }
```

其余的代码非常简单和直接。

### 服务器代码

现在让我们看看在服务器端需要做什么。以下代码将展示我们需要在服务器端编写什么：

```js
var WebSocketServer = require('ws').Server
    wss = new WebSocketServer({

        port: 9001

    });

//method to broadcast message to all the users
wss.broadcast = function broadcast(data, sentBy)

{

    for (var i in this.clients)

    {

        this.clients[i].send(data);

    }

};

function isJson(str)

{

    try

    {

        JSON.parse(str);

    } 

    catch (e)

    {

        return false;

    }

    return true;

}

//client connection open method
wss.on('connection', function connection(ws)

{
//client message receive method
    ws.on('message', function incoming(message)

    {
        if (isJson(message))

        {
//broadcasting message to all users
            wss.broadcast(message, this);

            console.log('broadcasting data');

        }

        console.log('received: %s', message);

    });

    console.log('sending initial Data');

});
```

#### 代码解释

在服务器端，我们并没有做太多编码。它几乎与上一章的服务器代码相同。我们接收数据并将其广播给所有连接的用户。

# 自己动手

这个应用程序是一个非常简单和易于构建的应用程序。我们已经看到了如何创建一个具有一些有限功能的简单应用程序。可以添加许多功能来使这个应用程序更加强大。让我们给你一些关于你可以开发的功能的提示和信息。

## 用户注册

用户打开 URL 时，将打开一个登录/注册对话框。用户的姓名等详细信息将显示在屏幕左上角。

### 提示

这种情况将需要一个数据库连接。有一些数据库可以很容易地连接到我们的 Node.js 服务器，比如**MongoDB**。我会把它的实现方法留给你。要了解如何连接 Node.js 和 MongoDB，请访问[`mongodb.github.io/node-mongodb-native/`](http://mongodb.github.io/node-mongodb-native/)。

## 用户列表

创建一个按钮，点击后会显示当前在线用户的列表。这种情况需要在客户端和服务器端都进行代码更改。让我列出一些你需要实现这个功能的关键点：

+   一旦你开发了用户注册功能，我们已经在数据库中保存了用户列表。我们可以维护一个所有在线用户的列表，或者我们可以只在服务器上保留列表。将数据持久化在服务器上的问题是，一旦服务器重新启动，数据将被擦除。

+   从服务器获取用户列表，一旦我们加入服务器就立即发送。这可以通过发送特定的消息来实现，比如`getOnlineUsers`，并在消息事件处理程序中添加另一个条目，返回用户列表。

+   在屏幕上显示用户列表，这样你就可以看到一个合适的在线用户列表。这需要在客户端进行更改。

## 与特定用户分享

由于我们已经实现了用户列表，现在我们可以实现基于用户的绘图共享。在这种情况下，我们只能与一些特定的用户分享我们的绘图。

### 提示

这可以通过向我们发送到服务器的对象添加另一个参数来实现：目标用户 ID。这个用户 ID 对于用户是唯一的，用于识别用户。这将帮助我们仅向特定用户发送数据。

## 保存绘图

一旦我们完成了绘图，我们可以保存它，并使其在将来可用。

### 提示

我们必须将我们的应用程序连接到一个可以保存我们在先前场景中已经实现的值的数据库。现在我们需要在数据库中添加另一个表，只用于存储绘图。Fabric.js 给我们提供了我们绘制的所有绘图元素的对象，我们可以制作一个 JSON 字符串并将其存储在数据库中以供将来使用。

# 应用程序结构

重构应用程序是一个非常重要的部分。如果我们看一下我们写的代码，我们会发现它没有一个良好的结构。结构必须是这样的，以便将来如果我们想要添加一些功能，那么应该很容易做到。代码应该以一种易于维护的方式编写。为了实现这一点，我们需要使用某种结构，这就是所谓的框架。框架旨在为应用程序提供一种结构感。

# 重构应用程序

现在我们知道了框架，让我们使用 AngularJS 框架重构我们的应用程序。让我们看看我们可以在这里重构什么；我们将把一切分成模型、视图、控制器和服务层。让我们看看这些术语是什么，它们在我们的应用程序中的作用。

## 模型

在我们的应用程序中，我们还没有看到存储数据的需求，但如果我们想扩展我们的应用程序并添加更多功能，那么就需要一个**模型**。正如我们在一些场景中所看到的，我们有用户和绘图列表，我们需要模型在客户端存储数据，以便很容易地访问。AngularJS 提供了很好的功能来存储数据，绑定有助于在 UI 中非常容易地显示列表数据。

## 视图

一个应用程序通常被分成不同的视图，但在我们的应用程序中，我们只有一个视图。正如我们在场景中所看到的，我们需要一个用户登录界面。在这种情况下，我们需要设置一个不同的视图，这时视图就出现了。AngularJS 为我们提供了一种非常简单的方式来维护我们的视图。AngularJS 的路由机制也帮助我们在不同的视图之间导航，提供浏览器历史以及维护单页面应用程序。

## 控制器

由于应用程序被分成不同的视图，我们还需要不同的控制器，这些控制器基本上控制 UI 行为，并帮助与服务进行通信。AngularJS 控制器非常强大，并实现了**依赖注入**（**DI**），这有助于将服务、模型等注入到控制器中，以在视图中进行操作。

## 服务

当我们有一个连接到服务器的应用程序时，服务非常重要。将服务器通信集中在一个地方是一个很好的方法，因为它在应用程序中创建了不同的层，可以在不影响应用程序的其他层的情况下进行操作。

当我们阅读并了解使用 AngularJS 框架构建应用程序的不同模式时，我强烈建议您开始使用 AngularJS 实现相同的应用程序。这是一个非常优秀的框架，可以满足开发者的所有需求，是一个功能齐全的框架。

# 总结

在本章中，我们已经看到了如何利用基于 HTML5 的 JavaScript 库。我们将 WebSockets 与 Fabric.js 库结合在一起，用于协作应用程序。我们还看到了如何将应用程序分成部分并进行创建。我们了解了开发流程，并学习了应用程序的结构。

在下一章中，我们将看到 WebSocket 的行为及其在移动设备和平板电脑上的实现。
