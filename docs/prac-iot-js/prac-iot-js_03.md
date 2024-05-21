# 第三章：IoTFW.js - II

在上一章中，我们已经看到了树莓派、代理、API 引擎和 Web 应用程序之间的基本设置。在本章中，我们将继续处理框架的其余部分。我们还将构建一个涉及传感和执行的简单示例。我们将使用温湿度传感器读取温度和湿度，并使用 Web、桌面或移动应用程序打开/关闭连接到我们的树莓派的 LED。

在本章中，我们将涵盖以下主题：

+   更新 API 引擎

+   将 API 引擎与 Web 应用程序集成

+   使用 DHT11 和 LED 构建端到端示例

+   构建桌面应用程序

+   构建移动应用程序

# 更新 API 引擎

现在我们已经完成了 Web 应用程序的开发，我们将更新 API 引擎以添加设备的 API 和数据服务，以及 Web 套接字。

打开`api-engine/server/routes.js`；我们将在这里添加两个路由。更新`api-engine/server/routes.js`如下：

```js
'use strict'; 

var path = require('path'); 

module.exports = function(app) { 
  // Insert routes below 
  app.use('/api/v1/users', require('./api/user')); 
  app.use('/api/v1/devices', require('./api/device')); 
  app.use('/api/v1/data', require('./api/data')); 

  app.use('/auth', require('./auth')); 
}; 
```

现在，我们将为这些路由添加定义。在`api-engine/server/api`文件夹内，创建一个名为`device`的新文件夹。在`device`文件夹内，创建一个名为`index.js`的新文件。更新`api-engine/server/api/device/index.js`如下：

```js
'use strict'; 

var express = require('express'); 
var controller = require('./device.controller'); 
var config = require('../../config/environment'); 
var auth = require('../../auth/auth.service'); 

var router = express.Router(); 

router.get('/', auth.isAuthenticated(), controller.index); 
router.delete('/:id', auth.isAuthenticated(), controller.destroy); 
router.put('/:id', auth.isAuthenticated(), controller.update); 
router.get('/:id', auth.isAuthenticated(), controller.show); 
router.post('/', auth.isAuthenticated(), controller.create); 

module.exports = router; 
```

在这里，我们添加了五个路由，如下：

+   获取所有设备

+   删除设备

+   更新设备

+   获取一个设备

+   创建一个设备

接下来，在`api-engine/server/api/device/`文件夹内创建另一个文件，命名为`device.model.js`。这个文件将包含设备集合的 mongoose 模式。更新`api-engine/server/api/device/device.model.js`如下：

```js
'use strict'; 

var mongoose = require('mongoose'); 
var Schema = mongoose.Schema; 

var DeviceSchema = new Schema({ 
    name: String, 
    macAddress: String, 
    createdBy: { 
        type: String, 
        default: 'user' 
    }, 
    createdAt: { 
        type: Date 
    }, 
    updatedAt: { 
        type: Date 
    } 
}); 

DeviceSchema.pre('save', function(next) { 
    var now = new Date(); 
    this.updatedAt = now; 
    if (!this.createdAt) { 
        this.createdAt = now; 
    } 
    next(); 
}); 

module.exports = mongoose.model('Device', DeviceSchema); 
```

最后，控制器逻辑。在`api-engine/server/api/device`文件夹内创建一个名为`device.controller.js`的文件，并更新`api-engine/server/api/device/device.controller.js`如下：

```js
'use strict'; 

var Device = require('./device.model'); 

/** 
 * Get list of all devices for a user 
 */ 
exports.index = function(req, res) { 
    var currentUser = req.user._id; 
    // get only devices related to the current user 
    Device.find({ 
        createdBy: currentUser 
    }, function(err, devices) { 
        if (err) return res.status(500).send(err); 
        res.status(200).json(devices); 
    }); 
}; 

/** 
 * Add a new device 
 */ 
exports.create = function(req, res, next) { 
    var device = req.body; 
    // this device is created by the current user 
    device.createdBy = req.user._id; 
    Device.create(device, function(err, device) { 
        if (err) return res.status(500).send(err); 
        res.json(device); 
    }); 
}; 

/** 
 * Get a single device 
 */ 
exports.show = function(req, res, next) { 
    var deviceId = req.params.id; 
    // the current user should have created this device 
    Device.findOne({ 
        _id: deviceId, 
        createdBy: req.user._id 
    }, function(err, device) { 
        if (err) return res.status(500).send(err); 
        if (!device) return res.status(404).end(); 
        res.json(device); 
    }); 
}; 

/** 
 * Update a device 
 */ 
exports.update = function(req, res, next) { 
    var device = req.body; 
    device.createdBy = req.user._id; 

    Device.findOne({ 
        _id: deviceId, 
        createdBy: req.user._id 
    }, function(err, device) { 
        if (err) return res.status(500).send(err); 
        if (!device) return res.status(404).end(); 

        device.save(function(err, updatedDevice) { 
            if (err) return res.status(500).send(err); 
            return res.status(200).json(updatedDevice); 
        }); 
    }); 
}; 

/** 
 * Delete a device 
 */ 
exports.destroy = function(req, res) { 
    Device.findOne({ 
        _id: req.params.id, 
        createdBy: req.user._id 
    }, function(err, device) { 
        if (err) return res.status(500).send(err); 

        device.remove(function(err) { 
            if (err) return res.status(500).send(err); 
            return res.status(204).end(); 
        }); 
    }); 
}; 
```

在这里，我们已经定义了路由的逻辑。

设备 API 为我们管理设备。为了管理每个设备的数据，我们将使用这个集合。

现在，我们将定义数据 API。在`api-engine/server/api`文件夹内创建一个名为`data`的新文件夹。在`api-engine/server/api/data`文件夹内，创建一个名为`index.js`的新文件，并更新`api-engine/server/api/data/index.js`如下：

```js
'use strict'; 

var express = require('express'); 
var controller = require('./data.controller'); 
var auth = require('../../auth/auth.service'); 

var router = express.Router(); 

router.get('/:deviceId/:limit', auth.isAuthenticated(), controller.index); 
router.post('/', auth.isAuthenticated(), controller.create); 

module.exports = router; 
```

我们在这里定义了两个路由：一个用于基于设备 ID 查看数据，另一个用于创建数据。查看数据路由返回作为请求的一部分传递的数量限制的设备数据。如果您记得，在`web-app/src/app/services/data.service.ts`中，我们已经将`dataLimit`类变量定义为`30`。这是我们从 API 中一次获取的记录数。

接下来，对于 mongoose 模式，在`api-engine/server/api/data`文件夹内创建一个名为`data.model.js`的新文件，并更新`api-engine/server/api/data/data.model.js`如下：

```js
'use strict'; 

var mongoose = require('mongoose'); 
var Schema = mongoose.Schema; 

var DataSchema = new Schema({ 
    macAddress: { 
        type: String 
    }, 
    data: { 
        type: Schema.Types.Mixed 
    }, 
    createdBy: { 
        type: String, 
        default: 'raspberrypi3' 
    }, 
    createdAt: { 
        type: Date 
    }, 
    updatedAt: { 
        type: Date 
    } 
}); 

DataSchema.pre('save', function(next) { 
    var now = new Date(); 
    this.updatedAt = now; 
    if (!this.createdAt) { 
        this.createdAt = now; 
    } 
    next(); 
});
```

```js
DataSchema.post('save', function(doc) { 
    //console.log('Post Save Called', doc); 
    require('./data.socket.js').onSave(doc) 
}); 

module.exports = mongoose.model('Data', DataSchema); 
```

现在，数据 API 的控制器逻辑。在`api-engine/server/api/data`文件夹内创建一个名为`data.controller.js`的文件，并更新`api-engine/server/api/data/data.controller.js`如下：

```js
'use strict'; 

var Data = require('./data.model'); 

/** 
 * Get Data for a device 
 */ 
exports.index = function(req, res) { 
    var macAddress = req.params.deviceId; 
    var limit = parseInt(req.params.limit) || 30; 
    Data.find({ 
        macAddress: macAddress 
    }).limit(limit).exec(function(err, devices) { 
        if (err) return res.status(500).send(err); 
        res.status(200).json(devices); 
    }); 
}; 

/** 
 * Create a new data record 
 */ 
exports.create = function(req, res, next) { 
    var data = req.body; 
    data.createdBy = req.user._id; 
    Data.create(data, function(err, _data) { 
        if (err) return res.status(500).send(err); 
        res.json(_data); 
        if(data.topic === 'led'){ 
            require('../../mqtt/index.js').sendLEDData(data.data.l);// send led value 
        } 
    }); 
}; 
```

在这里，我们定义了两种方法：一种是为设备获取数据，另一种是为设备创建新的数据记录。

对于数据 API，我们也将实现套接字，因此当来自树莓派的新记录时，我们立即通知 Web 应用程序、桌面应用程序或移动应用程序，以便数据可以实时显示。

从前面的代码中可以看到，如果传入的主题是`LED`，我们将调用`sendLEDData()`，它会将数据发布到设备。

在`api-engine/server/api/data`文件夹内创建一个名为`data.socket.js`的文件，并更新`api-engine/server/api/data/data.socket.js`如下：

```js
/** 
 * Broadcast updates to client when the model changes 
 */ 

'use strict'; 

var data = require('./data.model'); 
var socket = undefined; 

exports.register = function(_socket) { 
   socket = _socket; 
} 

function onSave(doc) { 
    // send data to only the intended device 
    socket.emit('data:save:' + doc.macAddress, doc); 
} 

module.exports.onSave = onSave; 
```

这将负责在成功保存到数据库后发送新的数据记录。

接下来，我们需要将 socket 添加到 socket 配置中。打开`api-engine/server/config/socketio.js`并进行更新如下：

```js
'use strict'; 

var config = require('./environment'); 

// When the user disconnects.. perform this 
function onDisconnect(socket) {} 

// When the user connects.. perform this 
function onConnect(socket) { 
    // Insert sockets below 
    require('../api/data/data.socket').register(socket); 
} 
module.exports = function(socketio) { 
    socketio.use(require('socketio-jwt').authorize({ 
        secret: config.secrets.session, 
        handshake: true 
    })); 

    socketio.on('connection', function(socket) { 
        var socketId = socket.id; 
        var clientIp = socket.request.connection.remoteAddress; 

        socket.address = socket.handshake.address !== null ? 
            socket.handshake.address.address + ':' + socket.handshake.address.port : 
            process.env.DOMAIN; 

        socket.connectedAt = new Date(); 

        // Call onDisconnect. 
        socket.on('disconnect', function() { 
            onDisconnect(socket); 
            // console.info('[%s] DISCONNECTED', socket.address); 
        }); 

        // Call onConnect. 
        onConnect(socket); 
        console.info('[%s] Connected on %s', socketId, clientIp); 
    }); 
}; 
```

请注意，我们使用`socketio-jwt`来验证套接字连接，以查看它是否具有 JWT。如果没有提供有效的 JWT，我们不允许客户端连接。

通过这样，我们完成了对 API 引擎的所需更改。保存所有文件并通过运行以下命令启动 API 引擎：

```js
npm start  
```

这将启动 API 引擎。在下一节中，我们将测试 Web 应用程序和 API 引擎之间的集成，并继续从上一节开始的步骤。

# 集成 Web 应用程序和 API 引擎

启动代理商、API 引擎和 Web 应用程序。一旦它们都成功启动，导航到`http://localhost:4200/`。使用我们创建的凭据登录。一旦成功登录，我们应该看到以下屏幕：

![](img/00032.jpeg)

这是真的，因为我们的账户中没有任何设备。点击添加设备，我们应该看到如下内容：

![](img/00033.jpeg)

通过给设备命名来添加一个新设备。我给我的设备命名为`Pi 1`并添加了 mac 地址。我们将使用设备的 mac 地址作为识别设备的唯一方式。

点击创建，我们应该看到一个新设备被创建，它将重定向我们到主页并显示新创建的设备，可以在以下截图中看到：

![](img/00034.jpeg)

现在，当我们点击查看按钮时，我们应该看到以下页面：

![](img/00035.jpeg)

在本书的示例中，我们将不断更新此模板，并根据需要进行修改。目前，这是一个由`web-app/src/app/device/device.component.html`表示的虚拟模板。

如果我们打开开发者工具并查看网络选项卡 WS 部分，如下截图所示，我们应该能够看到一个带有 JWT 令牌的 Web 套接字请求被发送到我们的服务器：

![](img/00036.jpeg)

通过这样，我们完成了将树莓派与代理商、代理商与 API 引擎以及 API 引擎与 Web 应用程序连接起来。为了完成从设备到 Web 应用程序的整个数据往返，我们将在下一节实现一个简单的用例。

# 使用 DHT11 和 LED 测试端到端流程

在开始处理桌面和移动应用程序之前，我们将为树莓派到 Web 应用程序的端到端数据流实现一个流程。

我们将要处理的示例实现了执行器和传感器用例。我们将把 LED 连接到树莓派，并从 Web 应用程序中打开/关闭 LED，我们还将把 DHT11 温度传感器连接到树莓派，并在 Web 应用程序中实时查看其值。

我们将开始使用树莓派，在那里实现所需的逻辑；接下来，与 API 引擎一起工作，进行所需的更改，最后是 Web 应用程序来表示数据。

# 设置和更新树莓派

首先，我们将按照以下方式设置电路：

![](img/00037.jpeg)

现在，我们将进行以下连接：

| **源引脚** | **组件引脚** |
| --- | --- |
| 树莓派引脚 1 - 3.3V | 面包板+栏杆 |
| 树莓派引脚 6 - 地面 | 面包板-栏杆 |
| 树莓派引脚 3 - GPIO 2 | 温度传感器信号引脚 |
| 树莓派引脚 12 - GPIO 18 | LED 阳极引脚 |
| LED 阴极引脚 | 面包板-栏杆 |
| 温度传感器+引脚 | 面包板+栏杆 |
| 温度传感器-引脚 | 面包板-栏杆 |

我们在引脚 12/GPIO 18 和 LED 引脚的阳极之间使用了一个 220 欧姆的限流电阻。

一旦建立了这种连接，我们将编写所需的逻辑。在树莓派上，打开`pi-client/index.js`文件并更新如下：

```js
var config = require('./config.js'); 
var mqtt = require('mqtt'); 
var GetMac = require('getmac'); 
var rpiDhtSensor = require('rpi-dht-sensor'); 
var rpio = require('rpio'); 
var dht11 = new rpiDhtSensor.DHT11(2); 
var temp = 0, 
    prevTemp = 0; 
var humd = 0, 
    prevHumd = 0; 
var macAddress; 
var state = 0; 

// Set pin 12 as output pin and to low 
rpio.open(12, rpio.OUTPUT, rpio.LOW); 

var client = mqtt.connect({ 
    port: config.mqtt.port, 
    protocol: 'mqtts', 
    host: config.mqtt.host, 
    clientId: config.mqtt.clientId, 
    reconnectPeriod: 1000, 
    username: config.mqtt.clientId, 
    password: config.mqtt.clientId, 
    keepalive: 300, 
    rejectUnauthorized: false 
}); 

client.on('connect', function() { 
    client.subscribe('rpi'); 
    client.subscribe('led'); 
    GetMac.getMac(function(err, mac) { 
        if (err) throw err; 
        macAddress = mac; 
        client.publish('api-engine', mac); 
    }); 
}); 

client.on('message', function(topic, message) { 
    message = message.toString(); 
    if (topic === 'rpi') { 
        console.log('API Engine Response >> ', message); 
    } else if (topic === 'led') { 
        state = parseInt(message) 
        console.log('Turning LED', state ? 'On' : 'Off'); 
        // If we get a 1 we turn on the led, else off 
        rpio.write(12, state ? rpio.HIGH : rpio.LOW); 
    } else { 
        console.log('Unknown topic', topic); 
    } 
}); 

// infinite loop, with 3 seconds delay 
setInterval(function() { 
    getDHT11Values(); 
    console.log('Temperature: ' + temp + 'C, ' + 'humidity: ' + humd + '%'); 
    // if the temperature and humidity values change 
    // then only publish the values 
    if (temp !== prevTemp || humd !== prevHumd) { 
        var data2Send = { 
            data: { 
                t: temp, 
                h: humd, 
                l: state 
            }, 
            macAddress: macAddress 
        }; 
        console.log('Data Published'); 
        client.publish('dht11', JSON.stringify(data2Send)); 
        // reset prev values to current 
        // for next loop 
        prevTemp = temp; 
        prevHumd = humd; 
    } // else chill! 

}, 3000); // every three second 

function getDHT11Values() { 
    var readout = dht11.read(); 
    // update global variable 
    temp = readout.temperature.toFixed(2); 
    humd = readout.humidity.toFixed(2); 
} 
```

在上述代码中，我们添加了一些节点模块，如下所示：

+   `rpi-dht-sensor`: [`www.npmjs.com/package/rpi-dht-sensor`](https://www.npmjs.com/package/rpi-dht-sensor)；这个模块将帮助我们读取 DHT11 传感器的值

+   `rpio`: [`www.npmjs.com/package/rpio`](https://www.npmjs.com/package/rpio)；这个模块将帮助我们管理板上的 GPIO，我们将使用它来管理 LED

我们编写了一个`setInterval()`，它会每 3 秒运行一次。在`callback`中，我们调用`getDHT11Values()`来从传感器读取温度和湿度。如果温度和湿度值发生变化，我们就会将这些数据发布到代理。

还要注意`client.on('message')`；在这里，我们添加了另一个`if`条件，并监听`LED`主题。如果当前消息来自`LED`主题，我们知道我们将收到一个`1`或`0`，表示打开或关闭 LED。

最后，我们将安装这两个模块，运行：

```js
npm install rpi-dht-sensor -save
npm install rpio -save  
```

保存所有文件并运行`npm start`；这应该将树莓派连接到代理并订阅`LED`主题，如下所示：

![](img/00038.jpeg)

此外，如果我们从树莓派的控制台输出中看到，应该会看到以下内容：

![](img/00039.jpeg)

每当数据发生变化时，数据就会发布到代理。我们还没有实现对 API 引擎上的数据做出反应的逻辑，这将在下一节中完成。

# 更新 API 引擎

现在，我们将向在 API 引擎上运行的 MQTT 客户端添加所需的代码来处理来自设备的数据。更新`api-engine/server/mqtt/index.js`，如下所示：

```js
var Data = require('../api/data/data.model'); 
var mqtt = require('mqtt'); 
var config = require('../config/environment'); 

var client = mqtt.connect({ 
    port: config.mqtt.port, 
    protocol: 'mqtts', 
    host: config.mqtt.host, 
    clientId: config.mqtt.clientId, 
    reconnectPeriod: 1000, 
    username: config.mqtt.clientId, 
    password: config.mqtt.clientId, 
    keepalive: 300, 
    rejectUnauthorized: false 
}); 

client.on('connect', function() { 
    console.log('Connected to Mosca at ' + config.mqtt.host + ' on port ' + config.mqtt.port); 
    client.subscribe('api-engine'); 
    client.subscribe('dht11'); 
}); 

client.on('message', function(topic, message) { 
    // message is Buffer 
    // console.log('Topic >> ', topic); 
    // console.log('Message >> ', message.toString()); 
    if (topic === 'api-engine') { 
        var macAddress = message.toString(); 
        console.log('Mac Address >> ', macAddress); 
        client.publish('rpi', 'Got Mac Address: ' + macAddress); 
    } else if (topic === 'dht11') { 
        var data = JSON.parse(message.toString()); 
        // create a new data record for the device 
        Data.create(data, function(err, data) { 
            if (err) return console.error(err); 
            // if the record has been saved successfully,  
            // websockets will trigger a message to the web-app 
            console.log('Data Saved :', data.data); 
        }); 
    } else { 
        console.log('Unknown topic', topic); 
    } 
}); 

exports.sendLEDData = function(data) { 
    console.log('Sending Data', data); 
    client.publish('led', data); 
} 
```

在这里，我们订阅了一个名为`dht11`的主题，以监听树莓派发布的关于温度和湿度值的消息。我们还公开了另一个名为`sendLEDData`的方法，用于接受需要发送到设备的数据。

如果我们保存所有文件并重新启动 API 引擎，应该会看到以下内容：

![](img/00040.jpeg)

从上面的截图中，我们可以看到数据来自树莓派并保存到 MongoDB。要验证数据是否已保存，我们可以转到`mlab`数据库并查找名为`datas`的集合，应该如下所示：

![](img/00041.jpeg)

每当数据成功保存时，相同的副本也将发送到 Web 应用程序。在下一节中，我们将在 Web 仪表板上实时显示这些数据。

# 更新 Web 应用程序

在本节中，我们将开发在 Web 应用程序中实时显示数据所需的代码，以及提供一个界面，通过该界面我们可以打开/关闭 LED。

我们将首先添加一个切换开关组件。我们将使用`ngx-ui-switch` ([`github.com/webcat12345/ngx-ui-switch`](https://github.com/webcat12345/ngx-ui-switch))。

从`web-app-base`文件夹内运行以下命令：

```js
npm install ngx-ui-switch -save  
```

我们将使用`ng2-charts` [`valor-software.com/ng2-charts/`](https://valor-software.com/ng2-charts/)来绘制温度和湿度值的图表。我们也将通过运行以下命令安装这个模块：

```js
npm install ng2-charts --save
npm install chart.js --save  
```

这将安装切换开关和`ng2-charts`模块。接下来，我们需要将其添加到`@NgModule`中。打开`web-app/src/app/app.module.ts`并将以下命令添加到 imports 中：

```js
import { UiSwitchModule } from 'ngx-ui-switch'; 
import { ChartsModule } from 'ng2-charts'; 
```

然后，将`UiSwitchModule`和`ChartsModule`添加到 imports 数组中：

```js
// snipp snipp 
imports: [ 
    RouterModule.forRoot(appRoutes), 
    BrowserModule, 
    BrowserAnimationsModule, 
    FormsModule, 
    HttpModule, 
    LocalStorageModule.withConfig({ 
      prefix: 'web-app', 
      storageType: 'localStorage' 
    }), 
    SimpleNotificationsModule.forRoot(), 
    UiSwitchModule, 
    ChartsModule 
  ], 
// snipp snipp 
```

完成后，我们需要将`chart.js`导入到我们的应用程序中。打开`web-app/.angular-cli.json`并更新`scripts`部分，如下所示：

```js
// snipp snipp  
"scripts": [ 
        "../node_modules/chart.js/dist/Chart.js" 
      ], 
// snipp snipp  
```

保存所有文件并重新启动 Web 应用程序，如果它已经在运行。

现在，我们可以在设备组件中使用这个指令。

在我们当前的用例中，我们需要显示温度和湿度值，并提供一个切换开关来打开/关闭 LED。为此，我们在`web-app/src/app/device/device.component.html`中的模板将如下所示：

```js
<div class="container"> 
    <br> 
    <div *ngIf="!device"> 
        <h3 class="text-center">Loading!</h3> 
    </div> 
    <div class="row" *ngIf="lastRecord"> 
        <div class="col-md-12"> 
            <div class="panel panel-info"> 
                <div class="panel-heading"> 
                    <h3 class="panel-title"> 
                        {{device.name}} 
                    </h3> 
                    <span class="pull-right btn-click"> 
                        <i class="fa fa-chevron-circle-up"></i> 
                    </span> 
                </div> 
                <div class="clearfix"></div> 
                <div class="table-responsive"> 
                    <table class="table table-striped"> 
                        <tr> 
                            <td>Toggle LED</td> 
                            <td> 
                                <ui-switch [(ngModel)]="toggleState" (change)="toggleChange($event)"></ui-switch> 
                            </td> 
                        </tr> 
                        <tr *ngIf="lastRecord"> 
                            <td>Temperature</td> 
                            <td>{{lastRecord.data.t}}</td> 
                        </tr> 
                        <tr *ngIf="lastRecord"> 
                            <td>Humidity</td> 
                            <td>{{lastRecord.data.h}}</td> 
                        </tr> 
                        <tr *ngIf="lastRecord"> 
                            <td>Received At</td> 
                            <td>{{lastRecord.createdAt | date: 'medium'}}</td> 
                        </tr> 
                    </table> 
                    <div class="col-md-10 col-md-offset-1" *ngIf="lineChartData.length > 0"> 
                        <canvas baseChart [datasets]="lineChartData" [labels]="lineChartLabels" [options]="lineChartOptions" [legend]="lineChartLegend" [chartType]="lineChartType"></canvas> 
                    </div> 
                </div> 
            </div> 
        </div> 
    </div> 
</div> 
```

`DeviceComponent`类的所需代码：`web-app/src/app/device/device.component.ts`将如下所示：

```js
import { Component, OnInit, OnDestroy } from '@angular/core'; 
import { DevicesService } from '../services/devices.service'; 
import { Params, ActivatedRoute } from '@angular/router'; 
import { SocketService } from '../services/socket.service'; 
import { DataService } from '../services/data.service'; 
import { NotificationsService } from 'angular2-notifications'; 

@Component({ 
   selector: 'app-device', 
   templateUrl: './device.component.html', 
   styleUrls: ['./device.component.css'] 
}) 
export class DeviceComponent implements OnInit, OnDestroy { 
   device: any; 
   data: Array<any>; 
   toggleState: boolean = false; 
   private subDevice: any; 
   private subData: any; 
   lastRecord: any; 

   // line chart config 
   public lineChartOptions: any = { 
         responsive: true, 
         legend: { 
               position: 'bottom', 
         }, hover: { 
               mode: 'label' 
         }, scales: { 
               xAxes: [{ 
                     display: true, 
                     scaleLabel: { 
                           display: true, 
                           labelString: 'Time' 
                     } 
               }], 
               yAxes: [{ 
                     display: true, 
                     ticks: { 
                           beginAtZero: true, 
                           steps: 10, 
                           stepValue: 5, 
                           max: 70 
                     } 
               }] 
         }, 
         title: { 
               display: true, 
               text: 'Temperature & Humidity vs. Time' 
         } 
   }; 
   public lineChartLegend: boolean = true; 
   public lineChartType: string = 'line'; 
   public lineChartData: Array<any> = []; 
   public lineChartLabels: Array<any> = []; 

   constructor(private deviceService: DevicesService, 
         private socketService: SocketService, 
         private dataService: DataService, 
         private route: ActivatedRoute, 
         private notificationsService: NotificationsService) { } 

   ngOnInit() { 
         this.subDevice = this.route.params.subscribe((params) => { 
               this.deviceService.getOne(params['id']).subscribe((response) => { 
                     this.device = response.json(); 
                     this.getData(); 
                     this.socketInit(); 
               }); 
         }); 
   } 

   getData() { 
         this.dataService.get(this.device.macAddress).subscribe((response) => { 
               this.data = response.json(); 
               this.genChart(); 
               this.lastRecord = this.data[0]; // descending order data 
               if (this.lastRecord) { 
                     this.toggleState = this.lastRecord.data.l; 
               } 
         }); 
   } 

   toggleChange(state) { 
         let data = { 
               macAddress: this.device.macAddress, 
               data: { 
                     t: this.lastRecord.data.t, 
                     h: this.lastRecord.data.h, 
                     l: state ? 1 : 0 
               }, 
               topic: 'led' 
         } 

         this.dataService.create(data).subscribe((resp) => { 
               if (resp.json()._id) { 
                     this.notificationsService.success('Device Notified!'); 
               } 
         }, (err) => { 
               console.log(err); 
               this.notificationsService.error('Device Notification Failed. Check console for the error!'); 
         }) 
   } 

   socketInit() { 
         this.subData = this.socketService.getData(this.device.macAddress).subscribe((data) => { 
               if(this.data.length <= 0) return; 
               this.data.splice(this.data.length - 1, 1); // remove the last record 
               this.data.push(data); // add the new one 
               this.lastRecord = data; 
               this.genChart(); 
         }); 
   } 

   ngOnDestroy() { 
         this.subDevice.unsubscribe(); 
         this.subData ? this.subData.unsubscribe() : ''; 
   } 

   genChart() { 

         let data = this.data; 
         let _dtArr: Array<any> = []; 
         let _lblArr: Array<any> = []; 

         let tmpArr: Array<any> = []; 
         let humArr: Array<any> = []; 

         for (var i = 0; i < data.length; i++) { 
               let _d = data[i]; 
               tmpArr.push(_d.data.t); 
               humArr.push(_d.data.h); 
               _lblArr.push(this.formatDate(_d.createdAt)); 
         } 

         // reverse data to show the latest on the right side 
         tmpArr.reverse(); 
         humArr.reverse(); 
         _lblArr.reverse(); 

         _dtArr = [ 
               { 
                     data: tmpArr, 
                     label: 'Temperature' 
               }, 
               { 
                     data: humArr, 
                     label: 'Humidity %' 
               }, 
         ] 

         this.lineChartData = _dtArr; 
         this.lineChartLabels = _lblArr; 
   } 

   private formatDate(originalTime) { 
         var d = new Date(originalTime); 
         var datestring = d.getDate() + "-" + (d.getMonth() + 1) + "-" + d.getFullYear() + " " + 
               d.getHours() + ":" + d.getMinutes(); 
         return datestring;
```

```js
   } 
} 
```

需要注意的关键方法如下：

+   `getData()`: 此方法用于在页面加载时获取最近的 30 条记录。我们从 API 引擎中以降序发送数据；因此我们提取最后一条记录并将其保存为最后一条记录。如果需要，我们可以使用剩余的记录来绘制图表

+   `toggleChange()`: 当切换开关被点击时，将触发此方法。此方法将发送数据到 API 引擎以保存

+   `socketInit()`: 此方法一直监听设备上的数据保存事件。使用此方法，我们将`lastRecord`变量更新为设备上的最新数据

+   `genChart()`: 此方法获取数据集合，然后绘制图表。当新数据通过套接字到达时，我们会从数据数组中删除最后一条记录并推送新记录，始终保持 30 条记录的总数

有了这个，我们就完成了处理此设置所需的模板开发。

保存所有文件，启动代理程序、API 引擎和 Web 应用程序，然后登录应用程序，然后导航到设备页面。

如果一切设置正确，我们应该看到以下屏幕：

![](img/00042.jpeg)

现在，每当数据通过套接字传输时，图表会自动更新！

现在，为了测试 LED，切换 LED 按钮到开启状态，您应该看到我们在树莓派上设置的 LED 会亮起，同样，如果我们关闭它，LED 也会关闭。

# 构建桌面应用程序并实现端到端流程

现在我们已经完成了与 Web 应用程序的端到端流程，我们将扩展到桌面和移动应用程序。我们将首先构建相同 API 引擎的桌面客户端。因此，如果用户更喜欢使用桌面应用程序而不是 Web 或移动应用程序，他/她可以使用这个。

这个桌面应用程序将具有与 Web 应用程序相同的所有功能。

为了构建桌面应用程序，我们将使用 electron ([`electron.atom.io/`](https://electron.atom.io/)) 框架。使用名为`generator-electron` ([`github.com/sindresorhus/generator-electron`](https://github.com/sindresorhus/generator-electron)) 的 Yeoman ([`yeoman.io/`](http://yeoman.io/)) 生成器，我们将创建基本应用程序。然后，我们将构建我们的 Web 应用程序，并使用该构建的`dist`文件夹作为桌面应用程序的输入。一旦我们开始工作，所有这些将更加清晰。

要开始，请运行以下命令：

```js
npm install yo generator-electron -g  
```

这将安装 yeoman 生成器和 electron 生成器。接下来，在`chapter2`文件夹内，创建一个名为`desktop-app`的文件夹，然后，在新的命令提示符/终端中运行以下命令：

```js
yo electron
```

这个向导将询问一些问题，您可以相应地回答：

![](img/00043.jpeg)

这将安装所需的依赖项。安装完成后，我们应该看到以下文件夹结构：

```js
.

├── index.css

├── index.html

├── index.js

├── license

├── package.json

└── readme.md
```

有了根目录下的`node_modules`文件夹。

一切都始于`desktop-app/package.json`的启动脚本，它启动`desktop-app/index.js`。`desktop-app/index.js`创建一个新的浏览器窗口，并启动`desktop-app/index.html`页面。

要从`desktop-app`文件夹内快速测试驱动，请运行以下命令：

```js
npm start   
```

因此，我们应该看到以下屏幕：

![](img/00044.jpeg)

现在，我们将添加所需的代码。在`desktop-app`文件夹的根目录下，创建一个名为`freeport.js`的文件，并更新`desktop-app/freeport.js`如下：

```js
var net = require('net') 
module.exports = function(cb) { 
    var server = net.createServer(), 
        port = 0; 
    server.on('listening', function() { 
        port = server.address().port 
        server.close() 
    }); 
    server.on('close', function() { 
        cb(null, port) 
    }) 
    server.on('error', function(err) { 
        cb(err, null) 
    }) 
    server.listen(0, '127.0.0.1') 
} 
```

使用上述代码，我们将在最终用户的计算机上找到一个空闲端口，并在 electron 外壳中启动我们的 Web 应用程序。

接下来，在`desktop-app`文件夹的根目录下创建一个名为`app`的文件夹。我们将在这里倾倒文件。接下来，在`desktop-app`文件夹的根目录下，创建一个名为`server.js`的文件。更新`server.js`如下：

```js
var FreePort = require('./freeport.js'); 
var http = require('http'), 
    fs = require('fs'), 
    html = ''; 

module.exports = function(cb) { 
    FreePort(function(err, port) { 
        console.log(port); 
        http.createServer(function(request, response) { 
            if (request.url === '/') { 
                html = fs.readFileSync('./app/index.html'); 
            } else { 
                html = fs.readFileSync('./app' + request.url); 
            } 
            response.writeHeader(200, { "Content-Type": "text/html" }); 
            response.write(html); 
            response.end(); 
        }).listen(port); 
        cb(port); 
    }); 
} 
```

在这里，我们监听一个空闲端口并启动`index.html`。现在，我们需要做的就是更新`desktop-app/index.js`中的`createMainWindow()`如下：

```js
// snipp snipp 
function createMainWindow() { 
    const { width, height } = electron.screen.getPrimaryDisplay().workAreaSize; 
    const win = new electron.BrowserWindow({ width, height }) 
    const server = require("./server")(function(port) { 
        win.loadURL('http://localhost:' + port); 
        win.on('closed', onClosed); 
        console.log('Desktop app started on port :', port); 
    }); 

    return win; 
} 
// snipp snipp 
```

这就是我们需要的所有设置。

现在，返回到`web-app`文件夹的终端/提示符（是的`web-app`，而不是`desktop-app`），并运行以下命令：

```js
ng build --env=prod
```

这将在`web app`文件夹内创建一个名为`dist`的新文件夹。`dist`文件夹的内容应如下所示：

```js
.

├── favicon.ico

├── index.html

├── inline.bundle.js

├── inline.bundle.js.map

├── main.bundle.js

├── main.bundle.js.map

├── polyfills.bundle.js

├── polyfills.bundle.js.map

├── scripts.bundle.js

├── scripts.bundle.js.map

├── styles.bundle.js

├── styles.bundle.js.map

├── vendor.bundle.js

└── vendor.bundle.js.map
```

我们在 web 应用程序中编写的所有代码最终都打包到了前述文件中。我们将获取`dist`文件夹内的所有文件（而不是`dist`文件夹），然后将其粘贴到`desktop-app/app`文件夹中。在进行前述更改后，桌面应用程序的最终结构将如下所示：

```js
.

├── app

│ ├── favicon.ico

│ ├── index.html

│ ├── inline.bundle.js

│ ├── inline.bundle.js.map

│ ├── main.bundle.js

│ ├── main.bundle.js.map

│ ├── polyfills.bundle.js

│ ├── polyfills.bundle.js.map

│ ├── scripts.bundle.js

│ ├── scripts.bundle.js.map

│ ├── styles.bundle.js

│ ├── styles.bundle.js.map

│ ├── vendor.bundle.js

│ └── vendor.bundle.js.map

├── freeport.js

├── index.css

├── index.html

├── index.js

├── license

├── package.json

├── readme.md

└── server.js
```

从现在开始，我们只需将`web-app/dist`文件夹的内容粘贴到`desktop-app`的`app`文件夹中。

要进行测试，请运行以下命令：

```js
npm start 
```

这将带来登录屏幕，如下所示：

![](img/00045.jpeg)

如果您看到之前显示的弹出窗口，请允许。成功登录后，您应该能够看到您帐户中的所有设备，如下所示：

![](img/00046.jpeg)

最后，设备信息屏幕：

![](img/00047.jpeg)

现在我们可以打开/关闭 LED，它应该有相应的反应。

现在，我们已经完成了桌面应用程序。

在下一节中，我们将使用 Ionic 框架构建一个移动应用程序。

# 构建移动应用程序并实现端到端流程

在本节中，我们将使用 Ionic 框架（[`ionicframework.com/`](http://ionicframework.com/)）构建我们的移动伴侣应用程序。输出或示例与我们为 web 和桌面应用程序所做的相同。

开始时，我们将通过运行以下命令安装最新版本的`ionic`和`cordova`：

```js
npm install -g ionic cordova  
```

现在，我们需要移动应用程序基础。如果您还没有克隆该书的代码存储库，可以使用以下命令（在您的任何位置）进行克隆：

```js
git clone git@github.com:PacktPublishing/Practical-Internet-of-Things-with-JavaScript.git
```

或者您也可以从[`github.com/PacktPublishing/Practical-Internet-of-Things-with-JavaScript`](https://github.com/PacktPublishing/Practical-Internet-of-Things-with-JavaScript)下载 zip 文件。

一旦存储库被下载，`cd`进入`base`文件夹，并将`mobile-app-base`文件夹复制到`chapter2`文件夹中。

复制完成后，`cd`进入`mobile-app`文件夹并运行以下命令：

```js
npm install
```

然后

```js
ionic cordova platform add android 
```

或者

```js
 ionic cordova platform add ios 
```

这将负责安装所需的依赖项并添加 Android 或 iOS 平台。

如果我们查看`mobile-app`文件夹，应该会看到以下内容：

```js
.

├── README.md

├── config.xml

├── hooks

│ └── README.md

├── ionic.config.json

├── package.json

├── platforms

│ ├── android

│ └── platforms.json

├── plugins

│ ├── android.json

│ ├── cordova-plugin-console

│ ├── cordova-plugin-device

│ ├── cordova-plugin-splashscreen

│ ├── cordova-plugin-statusbar

│ ├── cordova-plugin-whitelist

│ ├── fetch.json

│ └── ionic-plugin-keyboard

├── resources

│ ├── android

│ ├── icon.png

│ ├── ios

│ └── splash.png

├── src

│ ├── app

│ ├── assets

│ ├── declarations.d.ts

│ ├── index.html

│ ├── manifest.json

│ ├── pages

│ ├── service-worker.js

│ ├── services

│ └── theme

├── tsconfig.json

├── tslint.json

└── www

├── assets

├── build

├── index.html

├── manifest.json

└── service-worker.js
```

在我们的`mobile-app`文件夹中，最重要的文件是`mobile-app/config.xml`。该文件包含了 cordova 需要将 HTML/CSS/JS 应用程序转换为混合移动应用程序所需的定义。

接下来，我们有`mobile-app/resources`、`mobile-app/plugins`和`mobile-app/platforms`文件夹，其中包含我们正在开发的应用程序的 cordova 封装代码。

最后，`mobile-app/src`文件夹，这个文件夹是我们所有源代码的所在地。移动端的设置与我们为 web 应用程序和桌面应用程序所做的设置类似。我们有一个服务文件夹，其中包括`mobile-app/src/services/auth.service.ts`用于身份验证，`mobile-app/src/services/device.service.ts`用于与设备 API 进行交互，`mobile-app/src/services/data.service.ts`用于从设备获取最新数据，`mobile-app/src/services/socket.service.ts`用于在我们的移动应用程序中设置 Web 套接字，最后，`mobile-app/src/services/toast.service.ts`用于显示适用于移动设备的通知。`mobile-app/src/services/toast.service.ts`类似于我们在 web 和桌面应用程序中使用的通知服务。

接下来，我们有所需的页面。移动应用程序只实现了登录页面。我们强制用户使用 Web 或桌面应用程序来创建新帐户。`mobile-app/src/pages/login/login.ts`包括身份验证逻辑。`mobile-app/src/pages/home/home.ts`包括用户注册的所有设备列表。`mobile-app/src/pages/add-device/add-device.ts`具有添加新设备所需的逻辑，`mobile-app/src/pages/view-device/view-device.ts`用于查看设备信息。

现在，在`mobile-app`文件夹中，运行以下命令：

```js
ionic serve  
```

这将在浏览器中启动应用程序。如果您想在实际应用程序上进行测试，可以运行以下命令：

```js
ionic cordova run android   
```

或者，您可以运行以下命令：

```js
ionic cordova run ios  
```

这将在设备上启动应用程序。在任何情况下，应用程序的行为都将相同。

应用程序启动后，我们将看到登录页面：

![](img/00048.jpeg)

一旦我们成功登录，我们应该看到如下的主页。我们可以使用标题栏中的+图标添加新设备：

![](img/00049.jpeg)

新创建的设备应该在我们的主屏幕上反映出来，如下所示：

![](img/00050.jpeg)

如果我们点击“查看设备”，我们应该看到设备信息，如下所示：

![](img/00051.jpeg)

当我们切换按钮开/关时，树莓派上的 LED 应该打开/关闭：

![](img/00052.jpeg)

同一设置的另一个视图如下所示：

![](img/00053.jpeg)

上述是使用 DHT11 传感器和 LED 设置的树莓派 3 的设置。

通过这样做，我们已经成功建立了一个端到端的架构，用于执行物联网示例。从现在开始，我们将与 Web 应用程序、移动应用程序、桌面应用程序、树莓派以及一些 API 引擎一起工作，用于我们接下来的示例。我们将进行最小的更改。我们将专注于用例，而不是一遍又一遍地构建设置。

# 故障排除

如果您没有看到预期的输出，请检查以下内容：

+   检查经纪人、API 引擎、Web 应用程序和树莓派应用程序是否正在运行

+   检查提供给树莓派的经纪人的 IP 地址

+   检查提供给移动应用程序的 API 引擎的 IP 地址

# 摘要

在第二章，*IoTFW.js - I*和在本章中，我们经历了设置整个框架以与物联网解决方案一起工作的整个过程。我们只使用 JavaScript 作为编程语言构建了整个框架。

我们从理解架构和数据流开始，从树莓派到最终用户设备，如 Web 应用程序、桌面应用程序或移动应用程序。然后我们开始使用 Mosca 设置经纪人，设置 MongoDB 后。接下来，我们设计并开发了 API 引擎，并完成了基本的树莓派设置。

我们在 Web 应用程序和桌面应用程序上工作，并将简单的 LED 和 DHT11 温湿度传感器与树莓派集成，并看到了从一端到另一端的简单流程。我们将温度和湿度实时传输到 Web 应用程序和桌面应用程序，并使用切换按钮打开了 LED。

最后，我们建立了一个移动应用程序，并实现/验证了 LED 和 DHT11 的设置。

在第四章，*智能农业*，使用当前设置作为基础，我们将构建智能农业解决方案。
