# 第十一章：服务工作者-缓存和加速

到目前为止，我们已经看过了专用和共享工作线程，它们帮助将计算密集型任务放入后台。我们甚至创建了一个使用`SharedWorker`的共享缓存。现在，我们将看一下服务工作者，并学习它们如何用于为我们缓存资源（如 HTML、CSS、JavaScript 等）和数据，以便我们不必进行昂贵的往返到服务器。

在本章中，我们将涵盖以下主题：

+   了解 ServiceWorker

+   为离线使用缓存页面和模板

+   保存请求以备后用

到本章结束时，我们将能够为我们的 Web 应用程序创建离线体验。

# 技术要求

对于本章，您将需要以下内容：

+   一个编辑器或 IDE，最好是 VS Code

+   谷歌浏览器

+   可以运行 Node.js 的环境

+   本章的代码可以在[`github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter11`](https://github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter11)找到。

# 了解 ServiceWorker

`ServiceWorker`是一个位于我们的 Web 应用程序和服务器之间的代理。它捕获所做的请求并检查是否有与之匹配的模式。如果有模式匹配，则它将运行与该模式匹配的代码。为`ServiceWorker`编写代码与我们之前查看的`SharedWorker`和`DedicatedWorker`有些不同。最初，我们在一些代码中设置它并下载自身。我们有各种事件告诉我们工作线程所处的阶段。这些按以下顺序运行：

1.  **下载**：`ServiceWorker`正在为其托管的域或子域下载自身。

1.  **安装**：`ServiceWorker`正在附加到其托管的域或子域。

1.  **激活**：`ServiceWorker`已完全附加并加载以拦截请求。

安装事件尤其重要。这是我们可以监听更新的`ServiceWorker`的地方。假设我们想要为我们的`ServiceWorker`推送新代码。如果用户仍在我们决定将该代码推送到服务器时的页面上，他们仍将使用旧的工作线程。有办法终止旧的工作线程并强制它们更新（我们稍后会看到），但它仍将使用旧缓存。

此外，如果我们正在使用缓存来存储被请求的资源，它们将存储在旧缓存中。如果我们要更新这些资源，那么我们要确保清除先前的缓存并开始使用新的缓存。稍后我们将看一个例子，但最好提前了解这一点。

最后，服务工作者将每隔 24 小时更新一次自身，因此如果我们不强制用户更新`ServiceWorker`，他们将在 24 小时时获得这个新副本。这些都是我们在本章示例中要牢记的想法。我们在写出它们时会提醒您。

让我们从一个非常基本的例子开始。按照以下步骤进行：

1.  首先，我们需要一个静态服务器，以便我们可以使用服务工作者。为此，请运行`npm install serve`并将以下代码添加到`app.js`文件：

```js
const handler = require('serve-handler');
const http = require('http');
const server = http.createServer((req, res) => {
    return handler(req, res, {
        public : 'source'
    });
});
server.listen(3000, () => {
    console.log('listening at 3000');
});
```

1.  现在，我们可以从`source`目录中提供所有内容。创建一个基本的 HTML 页面，并让它加载一个名为`BaseServiceWorker.js`的`ServiceWorker`：

```js
<!DOCTYPE html>
<html>
    <head>
        <!-- get some resources -->
    </head>
    <body>
        <script type="text/javascript">
              navigator.serviceWorker.register('./BaseServiceWorker.js', 
             { scope : '/'})
            .then((reg) => {
                console.log('successfully registered worker');
            }).catch((err) => {
                console.error('there seems to be an issue!');
            })
        </script>
    </body>
</html>
```

1.  创建一个基本的`ServiceWorker`，每当发出请求时都会记录到我们的控制台：

```js
self.addEventListener('install', (event) => {
    console.log('we are installed!');
});
self.addEventListener('fetch', (event) => {
    console.log('a request was made!');
    fetch(event.request);
});
```

我们应该在控制台中看到两条消息。一条应该是静态的，说明我们已经正确安装了所有内容，而另一条将说明我们已成功注册了一个工作线程！现在，让我们向我们的 HTML 添加一个 CSS 文件并对其进行服务。

1.  将我们的新 CSS 文件命名为`main.css`并添加以下 CSS：

```js
*, :root {
    margin : 0;
    padding : 0;
    font-size : 12px;
}
```

1.  将此 CSS 文件添加到我们的 HTML 页面的顶部。

有了这个，重新加载页面并查看控制台中显示的内容。注意它没有说明我们已成功发出请求。如果我们不断点击重新加载按钮，可能会在页面重新加载之前看到消息出现。如果我们想看到这条消息，我们可以在 Chrome 中转到以下链接并检查那里的`ServiceWorker`：`chrome://serviceworker-internals`。

我们可能会看到其他服务工作者被加载。很多网站都这样做，这是一种缓存网页的技术。我们将很快更详细地研究这个问题。这就是为什么对于一些应用程序来说，第一次加载可能会很痛苦，而之后它们似乎加载得更快的原因。

页面顶部应该显示一个选项，用于在启动`ServiceWorker`时启动开发工具。请检查此选项。然后，停止/启动工作线程。现在，将打开一个控制台，允许我们调试我们的`ServiceWorker`：

![](img/dcee480a-892e-4485-aca2-9ca2cf3dddd3.png)

虽然这对调试很有用，但如果我们看一下启动此行为的页面，我们会看到一个小窗口，其中显示类似以下内容的信息：

```js
Console: {"lineNumber":2,"message":"we are installed!","message_level":1,"sourceIdentifier":3,"sourceURL":"http://localhost:3000/BaseServiceWorker.js"}
```

每次重新加载页面时都会获取 CSS 文件！如果我们再重新加载几次，应该会有更多这样的消息。这很有趣，但我们肯定可以做得更好。让我们继续缓存我们的`main.css`文件。将以下内容添加到我们的`BaseServiceWorker.js`文件中：

```js
self.addEventListener('install', (event) => {
    event.waitUntil(
        caches.open('v1').then((cache) => {
            return cache.addAll([
                './main.css'
            ]);
        }).then(() => {
            console.log('we are ready!');
        })
    );
});
self.addEventListener('fetch', (event) => {
    event.respondWith(
        caches.match(event.request).then((response) => {
            return response || fetch(event.request);
        })
    )
});
```

有了这个，我们引入了一个缓存。这个缓存将为我们获取各种资源。除了这个缓存，我们还引入了事件的`waitUntil`方法。这允许我们延迟`ServiceWorker`的初始化，直到我们从服务器获取了所有想要的数据。在我们的 fetch 处理程序中，我们现在正在检查我们的缓存中是否有资源。如果有，我们将提供该文件；否则，我们将代表页面发出 fetch 请求。

现在，如果我们加载页面，我们会注意到我们只有`we are ready`消息。尽管我们有新的代码，但页面被 Chrome 缓存了，所以它没有放弃我们的旧服务工作者。为了强制添加新的服务工作者，我们可以进入开发者控制台，转到应用程序选项卡。然后，我们可以转到左侧面板，转到`ServiceWorker`部分。应该有一个时间轴，说明有一个`ServiceWorker`正在等待被激活。如果我们点击旁边的文字，说 skipWaiting，我们可以激活新代码。

请点击此选项。看起来好像没有发生任何事情，但是如果我们返回到`chrome://serviceworker-internals`页面，我们会看到有一条消息。如果我们继续重新加载页面，我们会看到我们只有一条消息。这意味着我们已经加载了我们的新代码！

另一种检查我们是否成功缓存了`main.css`文件的方法是限制应用程序的下载速度（特别是因为我们是在本地托管）。返回开发人员工具，点击网络选项卡。在禁用缓存选项附近应该有一个网络速度的下拉菜单。目前，它应该显示我们在线。请将其切换到离线状态：

![](img/15280a71-55c0-412f-9146-916062274e3b.png)

好吧，我们刚刚丢失了我们的页面！在`BaseServiceWorker.js`中，我们应该添加以下内容：

```js
caches.open('v1').then((cache) => {
    return cache.addAll([
        './main.css',
        '/'
    ]);
})
```

现在，我们可以再次将我们的应用程序上线，并让这个新的`ServiceWorker`添加到页面中。添加完成后，将我们的应用程序切换到离线状态。现在，页面可以离线工作！我们将稍后更详细地探讨这个想法，但这给了我们一个很好的预览。

通过这简单的`ServiceWorker`和缓存机制的观察，让我们把注意力转向缓存页面并在`ServiceWorker`中添加一些模板功能。

# 为离线使用缓存页面和模板

正如我们在本章开头所述，Service Worker 的主要用途之一是缓存页面资源以供将来使用。我们在第一个简单的`ServiceWorker`中看到了这一点，但我们应该设置一个更复杂的页面，其中包含更多资源。按照以下步骤进行：

1.  创建一个名为`CacheServiceWorker.js`的全新`ServiceWorker`，并将以下模板代码添加到其中。这是大多数`ServiceWorker`实例将使用的代码：

```js
self.addEventListener('install', (event) => {
    event.waitUntil(
        caches.open('v1').then((cache) => {
            return cache.addAll([
                // add resources here
            ]);
        }).then(() => {
            console.log('we are ready!');
        })
    );
});
self.addEventListener('fetch', (event) => {
    event.respondWith(
        caches.match(event.request).then((response) => {
            return response || fetch(event.request);
        })
    )
});
```

1.  更新我们的`index.html`文件，以利用这个新的`ServiceWorker`：

```js
navigator.serviceWorker.register('./CacheServiceWorker.js', { scope : '/'})
    .then((reg) => {
        console.log('successfully registered worker');
    }).catch((err) => {
        console.error('there seems to be an issue!', err);
    })
```

1.  现在，让我们在我们的页面上添加一些按钮和表格。我们很快将利用这些：

```js
<button id="addRow">Add</button>
<button id="remove">Remove</button>
<table>
    <thead>
        <tr>
            <th>Id</th>
            <th>Name</th>
            <th>Description</th>
            <th>Points</th>
        </tr>
    </thead>
    <tbody id="tablebody">
    </tbody>
</table>
```

1.  添加一个 JavaScript 文件，用于处理我们与`interactions.js`页面的所有交互：

```js
const add = document.querySelector('#addRow');
const remove = document.querySelector('#remove');
const tableBody = document.querySelector('#tablebody');
add.addEventListener('click', (ev) => {
    fetch('/add').then((res) => res.json()).then((fin) =>
     tableBody.appendChild(fin));
});
remove.addEventListener('click', (ev) => {
    while(tableBody.firstChild) {
        tableBody.removeChild(tableBody.firstChild);
    }
});
```

1.  将 JavaScript 文件添加到我们的`ServiceWorker`作为预加载：

```js
caches.open('v1').then((cache) => {
    return cache.addAll([
        '/',
        './interactions.js',
        './main.css'
    ]);
}).then(() => {
    console.log('we are ready!');
})
```

1.  将 JavaScript 文件添加到我们的`index.html`文件的底部：

```js
<script src="interactions.js" type="text/javascript"></script>
```

现在，如果我们加载我们的页面，我们应该看到一个简单的表格坐在那里，有一个标题行和一些按钮。让我们继续向我们的页面添加一些基本样式，以使它更容易看到。将以下内容添加到我们在处理`BaseServiceWorker`时添加的`main.css`文件中：

```js
table {
    margin: 15px;
    border : 1px solid black;
}
th {
    border : 1px solid black;
    padding : 2px;
}
button {
    border : 1px solid black;
    padding :5px;
    background : #2e2e2e;
    color : #cfcfcf;
    cursor : pointer;
    margin-left : 15px;
    margin-top : 15px;
}
```

这个 CSS 为我们提供了一些基本的样式。现在，如果我们点击“添加”按钮，我们应该看到以下消息：

```js
The FetchEvent for "http://localhost:3000/add" resulted in a network error response: the promise was rejected.
```

由于我们还没有添加任何代码来处理这个问题，让我们继续在我们的`ServiceWorker`中拦截这条消息。按照以下步骤进行：

1.  将以下虚拟代码添加到我们的`ServiceWorker`的`fetch`事件处理程序中：

```js
event.respondWith(
    caches.match(event.request).then((response) => {
        if( response ) {
            return response
        } else {
            if( event.request.url.includes("/add") ) {
                return new Response(new Blob(["Here is some data"], 
                    { type : 'text/plain'}),
                    { status : 200 });
            }
            fetch(event.request);
        }
    })
)
```

1.  点击“添加”按钮。我们应该看到一个新的错误，说明它无法解析 JSON 消息。将`Blob`数据更改为一些 JSON：

```js
return new Response(new Blob([JSON.stringify({test : 'example', stuff : 'other'})], { type : 'application/json'}), { status : 200 });
```

1.  再次点击“添加”按钮。我们应该得到一个声明，说明我们刚刚传递给处理程序的内容不是`Node`类型。解析我们在“添加”按钮的点击处理程序中得到的数据：

```js
fetch('/add').then((res) => res.json()).then((fin) =>  {
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${fin.test}</td>
                    <td>${fin.stuff}</td>
                    <td>other</td>`;
    tableBody.appendChild(tr);
});
```

现在，如果我们尝试运行我们的代码，我们会看到一些有趣的东西：我们的 JavaScript 文件仍然是旧代码。`ServiceWorker`正在使用我们以前的旧缓存。在这里我们可以做两件事。首先，我们可以禁用`ServiceWorker`。或者，我们可以删除旧缓存并用新缓存替换它。我们将执行第二个选项。为此，我们需要在安装监听器中添加以下代码到我们的`ServiceWorker`中：

```js
event.waitUntil(
    caches.delete('v1').then(() => {
        caches.open('v1').then((cache) => {
            return cache.addAll([
                '/',
                './interactions.js',
                './main.css'
            ]);
        }).then(() => {
            console.log('we are ready!');
        });
    })
);
```

现在，我们可以在前端代码中加载模板，但我们将在这里模拟一个服务器端渲染系统。这有一些应用场景，但我想到的主要应用场景是我们在开发中尝试的模板系统。

大多数模板系统需要在我们使用它们之前编译成最终的 HTML 形式。我们可以设置一个*watch*类型的系统，在这个系统中，每当我们更新模板时，这些模板都会被重新加载，但这可能会变得繁琐，特别是当我们只想专注于前端时。另一种方法是将这些模板加载到我们的`ServiceWorker`中，并让它渲染它们。这样，当我们想要进行更新时，我们只需通过`caches.delete`方法删除我们的缓存，然后重新加载它。

让我们设置一个简单的示例，就像前面的示例一样，但模板不是在我们的前端代码中创建的，而是在我们的`ServiceWorker`中。按照以下步骤进行：

1.  创建一个名为`row.template`的模板文件，并用以下代码填充它：

```js
<td>${id}</td>
<td>${name}</td>
<td>${description}</td>
<td>${points}</td>
```

1.  删除我们的`interactions.js`中的模板代码，并用以下代码替换它：

```js
fetch('/add').then((res) => res.text()).then((fin) =>  {
    const row = document.createElement('tr');
    row.innerHTML = fin;
    tableBody.appendChild(row);
});
```

1.  让我们设置一些基本的模板代码。我们不会做任何接近第九章中所做的实际示例-构建静态服务器。相反，我们将循环遍历我们传递的对象，并填写我们的模板的部分，其中我们的键在对象中对应：

```js
const renderTemplate = function(template, obj) {
    const regex = /\${([a-zA-Z0-9]+)\}/;
    const keys = Object.keys(obj);
    let match = null;
    while(match = regex.exec(template)) {
        const key = match[1];
        if( keys.includes(key) ) {
            template = template.replace(match[0], obj[key]);
        } else {
            match = null;
        }
    }
    return template;
}
```

1.  将响应更改为`/add`端点，使用以下代码：

```js
if( event.request.url.includes('/add') ) {
    return fetch('./row.template')
        .then((res) => res.text())
        .then((template) => {
            return new Response(new Blob([renderTemplate(template, 
             add)],{type : 'text/html'}), {status : 200});   
        })
} else if( response ) {
    return response
} else {
    return fetch(event.request);
}
```

现在，我们将从服务器中获取我们想要的模板（在我们的情况下是`row.template`文件），并用我们拥有的任何数据填充它（同样，在我们的情况下，我们将使用存根数据）。现在，我们在`ServiceWorker`中有了模板，并且可以轻松地设置端点以通过这个模板系统。

当我们想要个性化网站的错误页面时，这也可能是有益的。如果我们想要在我们的 404 页面中出现一个随机图像并将其合并到页面中，我们可以在`ServiceWorker`中完成，而不是访问服务器。我们甚至可以在离线状态下这样做。我们只需要实现与此处相同类型的模板化。

有了这些概念，很容易看到我们在拦截请求时的能力以及我们如何使我们的 Web 应用程序在离线时工作。我们将学习的最后一个技术是在离线时存储我们的请求，并在重新联机时运行它们。这种类型的技术可以用于从浏览器中保存或加载文件。让我们来看看。

# 保存请求以便以后使用

到目前为止，我们已经学会了如何拦截请求并从我们的本地系统返回或甚至增强响应。现在，我们将学习如何在离线模式下保存请求，然后在联机时将调用发送到服务器。

让我们继续为此设置一个新的文件夹。按照以下步骤进行：

1.  创建一个名为`offline_storage`的文件夹，并向其中添加以下文件：

+   `index.html`

+   `main.css`

+   `interactions.js`

+   `OfflineServiceWorker.js`

1.  将以下样板代码添加到`index.html`中：

```js
<!DOCTYPE html>
<html>
    <head><!-- add css file --></head>
    <body>
        <h1>Offline Storage</h1>
        <button id="makeRequest">Request</button>
        <table>
            <tbody id="body"></tbody>
        </table>
        <p>Are we online?: <span id="online">No</span>
        <script src="interactions.js"></script>
        <script>
            let online = false;
            const onlineNotification =  
             document.querySelector('#online');
            window.addEventListener('load', function() {
                const changeOnlineNotification = function(status) {
                    onlineNotification.textContent = status ? "Yes" 
                     : "No";
                    online = status;
                }
                changeOnlineNotification(navigator.onLine);
                 navigator.serviceWorker.register('.
                 /OfflineCacheWorker.js', {scope : '/'})
                window.addEventListener('online', () => {
                 changeOnlineNotification(navigator.onLine) });
                window.addEventListener('offline', () => {
                 changeOnlineNotification(navigator.onLine) });
            });
        </script>
    </body>
</html>
```

1.  将以下样板代码添加到`OfflineServiceWorker.js`中：

```js
self.addEventListener('install', (event) => {
    event.waitUntil(   
     // normal cache opening
    );
});
self.addEventListener('fetch', (event) => {
    event.respondWith(
        caches.match(event.request).then((response) => {
            // normal response handling
        })
    )
});
```

1.  最后，将以下样板代码添加到`interactions.js`中：

```js
const requestMaker = document.querySelector('#makeRequest');
const tableBody = document.querySelector('#body');
requestMaker.addEventListener('click', (ev) => {
    fetch('/request').then((res) => res.json()).then((fin) => {
        const row = document.createElement('tr');
        row.innerHTML = `
        <td>${fin.id}</td>
        <td>${fin.name}</td>
        <td>${fin.phone}</td>
        <td><button id=${fin.id}>Delete</button></td>
        `
        row.querySelector('button').addEventListener('click', (ev) 
         => {
            fetch(`/delete/${ev.target.id}`).then(() => {
                tableBody.removeChild(row);
            });
        });
        tableBody.appendChild(row);
    })
})
```

将所有这些代码放在一起后，让我们继续更改我们的 Node.js 服务器，使其指向这个新的文件夹位置。我们将通过停止旧服务器并更改`app.js`文件，使其指向我们的`offline_storage`文件夹来实现这一点：

```js
const server = http.createServer((req, res) => {
    return handler(req, res, {
        public : 'offline_storage'
    });
});
```

有了这个，我们可以通过运行`node app.js`重新运行我们的服务器。我们可能会看到我们的旧页面出现。如果是这种情况，我们可以转到开发者工具中的“应用程序”选项卡，并在“服务工作者”部分下点击“注销”选项。重新加载页面后，我们应该看到新的`index.html`页面出现。我们的处理程序目前不起作用，所以让我们在`ServiceWorker`中添加一些存根代码，以处理我们在`interactions.js`中添加的两种 fetch 情况。按照以下步骤进行：

1.  在 fetch 事件处理程序中添加以下支持：

```js
caches.match(event.request).then((response) => {
    if( event.request.url.includes('/request') ) {
        return handleRequest();
    }
})
// below in the global scope of the ServiceWorker
let counter = 0;
let name = 65;
const handleRequest = function() {
    const data = {
        id : counter,
        name : String.fromCharCode(name),
        phone : Math.round(Math.random() * 10000)
    }
    counter += 1;
    name += 1;
    return new Response(new Blob([JSON.stringify(data)], {type : 
     'application/json'}), {status : 200});
}
```

1.  通过确保它正确处理响应，确保它向我们的表中添加一行。重新加载页面并确保在单击请求按钮时添加了新行：

![](img/1636a18d-88bd-4817-8bf2-a68b8cb20cc5.png)

1.  现在我们已经确保该处理程序正在工作，让我们继续为我们的删除请求添加另一个处理程序。我们将在我们的`ServiceWorker`中模拟服务器上的数据库删除：

```js
caches.match(event.request).then((response) => {
    if( event.request.url.includes('/delete') ) {
        return handleDelete(event.request.url);
    }
})
// place in the global scope of the Service Worker
const handleDelete = function(url) {
    const id = url.split("/")[2];
    return new Response(new Blob([id], {type : 'text/plain'}), 
     {status : 200});
}
```

1.  有了这个，让我们继续测试一下，确保我们点击删除按钮时行被删除。如果所有这些都有效，我们将拥有一个可以在线或离线工作的功能应用程序。

现在，我们所需要做的就是为即将发出但由于我们目前处于离线状态而无法发出的请求添加支持。为此，我们将在一个数组中存储请求，并一旦在我们的`ServiceWorker`中检测到我们重新联机，我们将发送所有请求。我们还将添加一些支持，让我们的前端知道我们正在等待这么多请求，如果需要，我们可以取消它们。现在让我们添加这个：

在 Chrome 中，从离线切换到在线会触发我们的**在线**处理程序，但从在线切换到离线似乎不会触发事件。我们可以测试离线到在线系统的功能，但测试另一种情况可能会更加困难。请注意，这种限制可能存在于许多开发系统中，试图解决这个问题可能会非常困难。

1.  首先，将我们大部分的`caches.match`代码移动到一个独立的函数中，如下所示：

```js
caches.match(event.request).then((response) => {
    if( response ) {
        return response
    }
    return actualRequestHandler(event);
})
```

1.  编写独立的函数，如下所示：

```js
const actualRequestHandler = function(req) {
    if( req.request.url.includes('/request') ) {
        return handleRequest();
    }
    if( req.request.url.includes('/delete') ) {
        return handleDelete(req.request.url);
    }
    return fetch(req.request);
}
```

1.  我们将通过轮询处理请求，以查看我们是否重新联机。设置一个每 30 秒工作一次的轮询计时器，并将我们的`caches.match`处理程序更改如下：

```js
const pollTime = 30000;
self.addEventListener('fetch', (event) => {
    event.respondWith(
        caches.match(event.request).then((response) => {
            if( response ) {
                return response
            }
            if(!navigator.onLine ) {
                return new Promise((resolve, reject) => {
                    const interval = setInterval(() => {
                        if( navigator.onLine ) {
                            clearInterval(interval);
                            resolve(actualRequestHandler(event));
                        }
                    }, pollTime)
                })
            } else {
                return actualRequestHandler(event);
            }
        })
    )
});
```

我们刚刚做的是为一个 promise 设置了一个返回。如果我们看不到系统在线，我们将每 30 秒轮询一次，以查看我们是否重新联机。一旦我们重新联机，我们的 promise 将清除间隔，并在 resolve 处理程序中实际处理请求。我们可以设置一个在取消请求之前尝试多少次的系统。我们只需要在间隔之后添加一个拒绝处理程序。

最后，我们将添加一种方法来停止当前所有未处理的请求。为此，我们需要一种方法来跟踪我们是否有未处理的请求，并且一种在`ServiceWorker`中中止它们的方法。这将非常简单，因为我们可以很容易地在前端跟踪仍在等待的内容。我们可以通过以下方式添加这个功能：

1.  首先，我们将添加一个显示，显示前端有多少未处理的请求。我们将把这个显示放在我们的在线状态系统之后：

```js
// inside of our index.html
<p>Oustanding requests: <span id="outstanding">0</span></p>

//inside our interactions.js
const requestAmount = document.querySelector('#outstanding');
let numRequests = 0;
requestMaker.addEventListener('click', (ev) => {
    numRequests += 1;
    requestAmount.textContent = numRequests;
    fetch('/request').then((res) => res.json()).then((fin) => {
        // our previous fetch handler
        numRequests -= 1;
        requestAmount.textContent = numRequests;
    });
    // can be setup for delete requests also
});
```

1.  在我们的`index.html`文件中添加一个按钮，用于取消所有未处理的请求。同时，在我们的`interactions.js`文件中添加相应的 JavaScript 代码：

```js
//index.html
<button id="stop">Stop all Pending</button>

//interactions.js
const stopRequests = document.querySelector('#stop');
stopRequests.addEventListener('click', (ev) => {   
    fetch('/stop').then((res) => {
        numRequests = 0;
        requestAmount.textContent = numRequests;
    });
});
```

1.  为停止请求添加相应的处理程序到我们的`ServiceWorker`：

```js
caches.match(event.request).then((response) => {
    if( response ) {
        return response
    }
    if( event.request.url.includes('/stop') ) {
        controller.abort();
        return new Response(new Blob(["all done"], {type :
        'text/plain'}), {status : 200});
    }
    // our previous handler code
})
```

现在，我们将利用一个叫做`AbortController`的东西。这个系统允许我们向诸如 fetch 请求之类的东西发送信号，以便我们可以说我们想要停止等待的请求。虽然这个系统主要用于停止 fetch 请求，但实际上我们可以利用这个信号来停止任何异步请求。我们通过创建一个`AbortController`并从中获取信号来实现这一点。然后，在我们的 promise 中，我们监听信号上的中止事件并拒绝 promise。

1.  添加`AbortController`，如下所示：

```js
const controller = new AbortController();
const signal = controller.signal;
const pollTime = 30000;
self.addEventListener('fetch', (event) => {
    event.respondWith(
        caches.match(event.request).then((response) => {
            if( response ) {
                return response
            }
            if( event.request.url.includes('/stop') ) {
                controller.abort();
                return new Response(new Blob(["all done"], {type :
                'text/plain'}), {status : 200});
            }
            if(!navigator.onLine ) {
                return new Promise((resolve, reject) => {
                    const interval = setInterval(() => {
                        if( navigator.onLine ) {
                            clearInterval(interval);
                            resolve(actualRequestHandler(event));
                        }
                    }, pollTime)
                    signal.addEventListener('abort', () => {
                        reject('aborted');
                    })
                });
            } else {
                return actualRequestHandler(event);
            }
        })
    )
});
```

现在，如果我们进入我们的系统，在离线模式下准备一些请求，然后点击取消按钮，我们会看到所有的请求都被取消了！我们本可以把`AbortController`放在我们前端的`interactions.js`文件中的 fetch 请求上，但一旦我们恢复在线，所有的 promise 仍然会运行，所以我们想确保没有任何东西在运行。这就是为什么我们把它放在`ServiceWorker`中的原因。

通过这样做，我们不仅看到了我们可以通过缓存数据来处理请求，还看到了当我们处于不稳定的位置时，我们可以存储这些请求。除此之外，我们还看到了我们可以利用`AbortController`来停止等待的 promise 以及如何利用它们除了停止 fetch 请求之外的其他用途。

# 总结

在本章中，我们了解了服务工作者如何将我们的应用程序从始终在线转变为我们可以创建真正*始终工作*的应用程序的系统。通过保存状态、本地处理请求、本地丰富请求，甚至保存离线使用的请求，我们能够处理我们应用程序的完整状态。

现在我们已经从客户端和服务器端使用 JavaScript 创建了丰富的 Web 应用程序，我们将开始研究一些高级技术，这些技术可以帮助我们创建高性能的应用程序，这些应用程序以前只能通过本机应用程序代码实现。我们可以通过使用 C、C++或 Rust 来实现这一点。

然而，在我们讨论这个之前，一个经常被应用开发者忽视的应用开发的部分是部署过程。在下一章中，我们将介绍一种通过一个流行系统叫做 CircleCI 来建立持续集成和持续开发（CI/CD）的方法。
