# 第八章：树莓派图像流式传输

在本章中，我们将学习使用树莓派 3 和树莓派摄像头进行实时视频流。我们将从树莓派 3 实时流式传输视频到我们的网络浏览器，并可以在世界各地访问此视频。作为下一步，我们将向当前设置添加运动检测器，如果检测到运动，我们将开始流式传输视频。在本章中，我们将介绍以下主题：

+   理解 MJPEG

+   使用树莓派和树莓派摄像头进行设置

+   实时将摄像头图像流式传输到仪表板

+   捕捉运动中的视频

# MJPEG

引用维基百科，[`en.wikipedia.org/wiki/Motion_JPEG`](https://en.wikipedia.org/wiki/Motion_JPEG)。

在多媒体中，动态 JPEG（M-JPEG 或 MJPEG）是一种视频压缩格式，其中数字视频序列的每个视频帧或隔行场都单独压缩为 JPEG 图像。最初为多媒体 PC 应用程序开发，M-JPEG 现在被视频捕获设备（如数码相机、IP 摄像机和网络摄像头）以及非线性视频编辑系统所使用。它受 QuickTime Player、PlayStation 游戏机和 Safari、Google Chrome、Mozilla Firefox 和 Microsoft Edge 等网络浏览器的本地支持。

如前所述，我们将捕获一系列图像，每隔`100ms`并在一个主题上流式传输图像二进制数据到 API 引擎，我们将用最新的图像覆盖现有图像。

这个流媒体系统非常简单和老式。在流媒体过程中没有倒带或暂停。我们总是能看到最后一帧。

现在我们对我们要实现的目标有了很高的理解水平，让我们开始吧。

# 设置树莓派

使用树莓派 3 设置树莓派摄像头非常简单。您可以从任何知名在线供应商购买树莓派 3 摄像头([`www.raspberrypi.org/products/camera-module-v2/`](https://www.raspberrypi.org/products/camera-module-v2/))。然后您可以按照此视频进行设置：摄像头板设置：[`www.youtube.com/watch?v=GImeVqHQzsE`](https://www.youtube.com/watch?v=GImeVqHQzsE)。

我的摄像头设置如下：

![](img/00108.jpeg)

我使用了一个支架，将我的摄像头吊在上面。

# 设置摄像头

现在我们已经连接了摄像头并由树莓派 3 供电，我们将按照以下步骤设置摄像头：

1.  从树莓派内部，启动一个新的终端并运行：

```js
    sudo raspi-config
```

1.  这将启动树莓派配置屏幕。选择接口选项：

![](img/00109.jpeg)

1.  在下一个屏幕上，选择 P1 摄像头并启用它：

![](img/00110.jpeg)

1.  这将触发重新启动，完成重新启动并重新登录到树莓派。

一旦您的摄像头设置好了，我们将对其进行测试。

# 测试摄像头

现在摄像头已经设置并通电，让我们来测试一下。打开一个新的终端并在桌面上`cd`。然后运行以下命令：

```js
raspistill -o test.jpg
```

这将在当前工作目录`Desktop`中拍摄屏幕截图。屏幕看起来会像下面这样：

![](img/00111.jpeg)

正如您所看到的，`test.jpg`被创建在`Desktop`上，当我双击它时，显示的是我办公室玻璃墙的照片。

# 开发逻辑

现在我们能够测试摄像头，我们将把这个设置与我们的物联网平台集成。我们将不断地以`100ms`的间隔流式传输这些图像到我们的 API 引擎，然后通过网络套接字更新网络上的用户界面。

要开始，我们将复制`chapter4`并将其转储到名为`chapter8`的文件夹中。在`chapter8`文件夹中，打开`pi/index.js`并进行以下更新：

```js
var config = require('./config.js'); 
var mqtt = require('mqtt'); 
var GetMac = require('getmac'); 
var Raspistill = require('node-raspistill').Raspistill; 
var raspistill = new Raspistill({ 
    noFileSave: true, 
    encoding: 'jpg', 
    width: 640, 
    height: 480 
}); 

var crypto = require("crypto"); 
var fs = require('fs'); 

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
    GetMac.getMac(function(err, mac) { 
        if (err) throw err; 
        macAddress = mac; 
        client.publish('api-engine', mac); 
        startStreaming(); 
    }); 

}); 

client.on('message', function(topic, message) { 
    message = message.toString(); 
    if (topic === 'rpi') { 
        console.log('API Engine Response >> ', message); 
    } else { 
        console.log('Unknown topic', topic); 
    } 
}); 

function startStreaming() { 
    raspistill 
        .timelapse(100, 0, function(image) { // every 100ms ~~FOREVER~~ 
            var data2Send = { 
                data: { 
                    image: image, 
                    id: crypto.randomBytes(8).toString("hex") 
                }, 
                macAddress: macAddress 
            }; 

            client.publish('image', JSON.stringify(data2Send)); 
            console.log('[image]', 'published'); 
        }) 
        .then(function() { 
            console.log('Timelapse Ended') 
        }) 
        .catch(function(err) { 
            console.log('Error', err); 
        }); 
} 
```

正如我们从前面的代码中所看到的，我们正在等待 MQTT 连接完成，一旦连接完成，我们调用`startStreaming()`开始流式传输。在`startStreaming()`内部，我们调用`raspistill.timelapse()`传入`100ms`，作为每次点击之间的时间差，`0`表示捕获应该持续不断地进行。

一旦图像被捕获，我们就用一个随机 ID、图像缓冲区和设备的`macAddress`构造`data2Send`对象。在发布到图像主题之前，我们将`data2Send`对象转换为字符串。

现在，将这个文件移动到树莓派的`pi-client`文件夹中，位于桌面上。然后从树莓派的`pi-client`文件夹内运行：

```js
npm install && npm install node-raspistill --save  
```

这两个命令将安装`node-raspistill`和`package.json`文件内的其他节点模块。

有了这个，我们完成了树莓派和相机的设置。在下一节中，我们将更新 API 引擎以接受图像的实时流。

# 更新 API 引擎

现在我们完成了树莓派的设置，我们将更新 API 引擎以接受传入的数据。

我们要做的第一件事是按照以下方式更新`api-engine/server/mqtt/index.js`：

```js
var Data = require('../api/data/data.model'); 
var mqtt = require('mqtt'); 
var config = require('../config/environment'); 
var fs = require('fs'); 
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
    client.subscribe('image'); 
}); 

client.on('message', function(topic, message) { 
    // message is Buffer 
    // console.log('Topic >> ', topic); 
    // console.log('Message >> ', message.toString()); 
    if (topic === 'api-engine') { 
        var macAddress = message.toString(); 
        console.log('Mac Address >> ', macAddress); 
        client.publish('rpi', 'Got Mac Address: ' + macAddress); 
    } else if (topic === 'image') { 
        message = JSON.parse(message.toString()); 
        // convert string to buffer 
        var image = Buffer.from(message.data.image, 'utf8'); 
        var fname = 'stream_' + ((message.macAddress).replace(/:/g, '_')) + '.jpg'; 
        fs.writeFile(__dirname + '/stream/' + fname, image, { encoding: 'binary' }, function(err) { 
            if (err) { 
                console.log('[image]', 'save failed', err); 
            } else { 
                console.log('[image]', 'saved'); 
            } 
        }); 

        // as of now we are not going to 
        // store the image buffer in DB.  
        // Gridfs would be a good way 
        // instead of storing a stringified text 
        delete message.data.image; 
        message.data.fname = fname; 

        // create a new data record for the device 
        Data.create(message, function(err, data) { 
            if (err) return console.error(err); 
            // if the record has been saved successfully,  
            // websockets will trigger a message to the web-app 
            // console.log('Data Saved :', data); 
        }); 
    } else { 
        console.log('Unknown topic', topic); 
    } 
}); 
```

正如我们从前面的代码中所看到的，在 MQTT 的消息事件中，我们添加了一个名为`image`的新主题。在这个主题内，我们提取了图像缓冲区的字符串格式，并将其转换回图像二进制数据。然后使用`fs`模块，我们一遍又一遍地覆盖相同的图像。

我们同时将数据保存到 MongoDB，并触发一个 socket emit。

作为下一步，我们需要在`mqtt`文件夹内创建一个名为`stream`的文件夹。在这个文件夹内，放入一个图片，链接在这里：`http://www.iconarchive.com/show/small-n-flat-icons-by-paomedia/sign-ban-icon.html.` 如果相机没有可用的视频流，将显示这张图片。

所有的图像都将保存在`stream`文件夹内，对于相同的设备将更新相同的图像，正如前面提到的，不会有任何倒带或重播。

现在，图片被保存在`stream`文件夹内，我们需要暴露一个端点来将这张图片发送给请求的客户端。为此，打开`api-engine/server/routes.js`并将以下内容添加到`module.exports`函数中：

```js
app.get('/stream/:fname', function(req, res, next) { 
        var fname = req.params.fname; 
        var streamDir = __dirname + '/mqtt/stream/'; 
        var img = streamDir + fname; 
        console.log(img); 
        fs.exists(img, function(exists) { 
         if (exists) { 
                return res.sendFile(img); 
            } else { 
                // http://www.iconarchive.com/show/small-n-flat-icons-by-paomedia/sign-ban-icon.html 
                return res.sendFile(streamDir + '/no-image.png'); 
            } 
        }); 
    });  
```

这将负责将图像分发给客户端（Web、桌面和移动端）。

有了这个，我们就完成了 API 引擎的设置。

保存所有文件并启动代理、API 引擎和`pi-client`。如果一切顺利设置，我们应该能看到来自树莓派的数据被发布。

![](img/00112.jpeg)

以及在 API 引擎中出现的相同数据：

![](img/00113.jpeg)

此时，图像正在被捕获并通过 MQTT 发送到 API 引擎。下一步是实时查看这些图像。

# 更新 Web 应用程序

现在数据正在流向 API 引擎，我们将在 Web 应用程序上显示它。打开`web-app/src/app/device/device.component.html`并按照以下方式更新它：

```js
<div class="container"> 
    <br> 
    <div *ngIf="!device"> 
        <h3 class="text-center">Loading!</h3> 
    </div> 
    <div class="row" *ngIf="!lastRecord"> 
        <h3 class="text-center">No Data!</h3> 
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
                <div class="table-responsive" *ngIf="lastRecord"> 
                    <table class="table table-striped"> 
                        <tr> 
                            <td colspan="2" class="text-center"><img  [src]="lastRecord.data.fname"></td> 
                        </tr> 
                        <tr class="text-center" > 
                            <td>Received At</td> 
                            <td>{{lastRecord.createdAt | date: 'medium'}}</td> 
                        </tr> 
                    </table> 
                </div> 
            </div> 
        </div> 
    </div> 
</div> 
```

在这里，我们实时显示了我们创建的图像。接下来，按照以下方式更新`web-app/src/app/device/device.component.ts`：

```js
import { Component, OnInit, OnDestroy } from '@angular/core'; 
import { DevicesService } from '../services/devices.service'; 
import { Params, ActivatedRoute } from '@angular/router'; 
import { SocketService } from '../services/socket.service'; 
import { DataService } from '../services/data.service'; 
import { NotificationsService } from 'angular2-notifications'; 
import { Globals } from '../app.global'; 

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
               }); 
         }); 
   } 

   getData() { 
         this.dataService.get(this.device.macAddress).subscribe((response) => { 
               this.data = response.json(); 
               let d = this.data[0]; 
               d.data.fname = Globals.BASE_API_URL + 'stream/' + d.data.fname; 
               this.lastRecord = d; // descending order data 
               this.socketInit(); 
         }); 
   } 

   socketInit() { 
         this.subData = this.socketService.getData(this.device.macAddress).subscribe((data: any) => { 
               if (this.data.length <= 0) return; 
               this.data.splice(this.data.length - 1, 1); // remove the last record 
               data.data.fname = Globals.BASE_API_URL + 'stream/' + data.data.fname + '?t=' + (Math.random() * 100000); // cache busting 
               this.data.push(data); // add the new one 
               this.lastRecord = data; 
         }); 
   }
```

```js
   ngOnDestroy() { 
         this.subDevice.unsubscribe(); 
         this.subData ? this.subData.unsubscribe() : ''; 
   } 
} 
```

在这里，我们正在构建图像 URL 并将其指向 API 引擎。保存所有文件，并通过在`web-app`文件夹内运行以下命令来启动 Web 应用程序：

```js
npm start  
```

如果一切按预期工作，当导航到“查看设备”页面时，我们应该会看到视频流非常缓慢地显示。我正在监视放在椅子前面的杯子，如下所示：

![](img/00114.jpeg)

# 更新桌面应用程序

现在 Web 应用程序已经完成，我们将构建相同的应用程序并将其部署到我们的桌面应用程序内。

要开始，请返回到`web-app`文件夹的终端/提示符，并运行以下命令：

```js
ng build --env=prod  
```

这将在`web-app`文件夹内创建一个名为`dist`的新文件夹。`dist`文件夹的内容应该如下所示：

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

我们编写的所有代码最终都打包到了上述文件中。我们将获取`dist`文件夹中的所有文件（不包括`dist`文件夹），然后将其粘贴到`desktop-app/app`文件夹中。在进行上述更改后，`desktop-app`的最终结构将如下所示：

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

进行测试，运行以下命令：

```js
npm start 
```

然后当我们导航到 VIEW DEVICE 页面时，我们应该看到：

![](img/00115.jpeg)

这样我们就完成了桌面应用程序的开发。在下一节中，我们将更新移动应用程序。

# 更新移动应用程序

在上一节中，我们已经更新了桌面应用程序。在本节中，我们将更新移动应用程序模板以流式传输图像。

首先，我们将更新 view-device 模板。按照以下方式更新`mobile-app/src/pages/view-device/view-device.html`：

```js
<ion-header> 
    <ion-navbar> 
        <ion-title>Mobile App</ion-title> 
    </ion-navbar> 
</ion-header> 
<ion-content padding> 
    <div *ngIf="!lastRecord"> 
        <h3 class="text-center">Loading!</h3> 
    </div> 
    <div *ngIf="lastRecord"> 
        <ion-list> 
            <ion-item> 
                <img [src]="lastRecord.data.fname"> 
            </ion-item> 
            <ion-item> 
                <ion-label>Received At</ion-label> 
                <ion-label>{{lastRecord.createdAt | date: 'medium'}}</ion-label> 
            </ion-item> 
        </ion-list> 
    </div> 
</ion-content> 
```

在移动端显示图像流的逻辑与 Web/桌面端相同。接下来，按照以下方式更新`mobile-app/src/pages/view-device/view-device.ts`：

```js
import { Component } from '@angular/core'; 
import { IonicPage, NavController, NavParams } from 'ionic-angular'; 
import { Globals } from '../../app/app.globals'; 
import { DevicesService } from '../../services/device.service'; 
import { DataService } from '../../services/data.service'; 
import { ToastService } from '../../services/toast.service'; 
import { SocketService } from '../../services/socket.service'; 

@IonicPage() 
@Component({ 
   selector: 'page-view-device', 
   templateUrl: 'view-device.html', 
}) 
export class ViewDevicePage { 
   device: any; 
   data: Array<any>; 
   toggleState: boolean = false; 
   private subData: any; 
   lastRecord: any; 

   constructor(private navCtrl: NavController, 
         private navParams: NavParams, 
         private socketService: SocketService, 
         private deviceService: DevicesService, 
         private dataService: DataService, 
         private toastService: ToastService) { 
         this.device = navParams.get("device"); 
         console.log(this.device); 
   } 

   ionViewDidLoad() { 
         this.deviceService.getOne(this.device._id).subscribe((response) => { 
               this.device = response.json(); 
               this.getData(); 
         }); 
   } 

   getData() { 
         this.dataService.get(this.device.macAddress).subscribe((response) => { 
               this.data = response.json(); 
               let d = this.data[0]; 
               d.data.fname = Globals.BASE_API_URL + 'stream/' + d.data.fname; 
               this.lastRecord = d; // descending order data 
               this.socketInit(); 
         }); 
   } 

   socketInit() { 
         this.subData = this.socketService.getData(this.device.macAddress).subscribe((data: any) => { 
               if(this.data.length <= 0) return; 
               this.data.splice(this.data.length - 1, 1); // remove the last record 
               data.data.fname = Globals.BASE_API_URL + 'stream/' + data.data.fname + '?t=' + (Math.random() * 100000); 
               this.data.push(data); // add the new one 
               this.lastRecord = data; 
         }); 
   } 

   ionViewDidUnload() { 
         this.subData && this.subData.unsubscribe && this.subData.unsubscribe(); //unsubscribe if subData is defined 
   } 
} 
```

保存所有文件并通过以下方式运行移动应用程序：

```js
ionic serve  
```

或者使用以下代码：

```js
ionic cordova run android  
```

然后我们应该看到以下内容：

![](img/00116.jpeg)

这样我们就完成了在移动应用程序上显示摄像头数据。

# 基于运动的视频捕获

正如我们所看到的，流式传输有些不连贯，缓慢，并非实时，另一个可能的解决方案是在树莓派和摄像头上放置一个运动检测器。然后当检测到运动时，我们开始录制 10 秒的视频。然后将此视频作为附件通过电子邮件发送给用户。

现在，我们将开始更新我们现有的代码。

# 更新树莓派

首先，我们将更新我们的树莓派设置以适应 HC-SR501 PIR 传感器。您可以在此处找到 PIR 传感器：[`www.amazon.com/Motion-HC-SR501-Infrared-Arduino-Raspberry/dp/B00M1H7KBW/ref=sr_1_4_a_it`](https://www.amazon.com/Motion-HC-SR501-Infrared-Arduino-Raspberry/dp/B00M1H7KBW/ref=sr_1_4_a_it)。

我们将把 PIR 传感器连接到树莓派的 17 号引脚，将摄像头连接到摄像头插槽，就像我们之前看到的那样。

一旦连接如前所述，按照以下方式更新`pi/index.js`：

```js
var config = require('./config.js'); 
var mqtt = require('mqtt'); 
var GetMac = require('getmac'); 
var Raspistill = require('node-raspistill').Raspistill; 
var crypto = require("crypto"); 
var fs = require('fs'); 
var Gpio = require('onoff').Gpio; 
var exec = require('child_process').exec; 

var pir = new Gpio(17, 'in', 'both'); 
var raspistill = new Raspistill({ 
    noFileSave: true, 
    encoding: 'jpg', 
    width: 640, 
    height: 480 
}); 

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
    GetMac.getMac(function(err, mac) { 
        if (err) throw err; 
        macAddress = mac; 
        client.publish('api-engine', mac); 
        // startStreaming(); 
    }); 

}); 

client.on('message', function(topic, message) { 
    message = message.toString(); 
    if (topic === 'rpi') { 
        console.log('API Engine Response >> ', message); 
    } else { 
        console.log('Unknown topic', topic); 
    } 
}); 

function startStreaming() { 
    raspistill 
        .timelapse(100, 0, function(image) { // every 100ms ~~FOREVER~~ 
            var data2Send = { 
                data: { 
                    image: image, 
                    id: crypto.randomBytes(8).toString("hex") 
                }, 
                macAddress: macAddress 
            }; 

            client.publish('image', JSON.stringify(data2Send)); 
            console.log('[image]', 'published'); 
        }) 
        .then(function() { 
            console.log('Timelapse Ended') 
        }) 
        .catch(function(err) { 
            console.log('Error', err); 
        }); 
} 

var isRec = false; 

// keep watching for motion 
pir.watch(function(err, value) { 
    if (err) exit(); 
    if (value == 1 && !isRec) { 
        console.log('Intruder detected'); 
        console.log('capturing video.. '); 
        isRec = true; 
        var videoPath = __dirname + '/video.h264'; 
        var file = fs.createWriteStream(videoPath); 
        var video_path = './video/video' + Date.now() + '.h264'; 
        var cmd = 'raspivid -o ' + video_path + ' -t 5000'; 

        exec(cmd, function(error, stdout, stderr) { 
            // output is in stdout 
            console.log('Video Saved @ : ', video_path); 
            require('./mailer').sendEmail(video_path, true, function(err, info) { 
                setTimeout(function() { 
                    // isRec = false; 
                }, 3000); // don't allow recording for 3 sec after 
            }); 
        }); 
    } 
}); 

function exit() { 
    pir.unexport(); 
    process.exit(); 
} 
```

从上述代码中可以看出，我们已将 GPIO 17 标记为输入引脚，并将其分配给名为`pir`的变量。接下来，使用`pir.watch()`，我们不断查看运动检测器的值是否发生变化。如果运动检测器检测到某种变化，我们将检查值是否为`1`，这表示触发了运动。然后使用`raspivid`我们录制一个 5 秒的视频并通过电子邮件发送。

为了从树莓派 3 发送电子邮件所需的逻辑，创建一个名为`mailer.js`的新文件，放在`pi-client`文件夹的根目录，并按以下方式更新它：

```js
var fs = require('fs'); 
var nodemailer = require('nodemailer'); 

var transporter = nodemailer.createTransport({ 
    service: 'Gmail', 
    auth: { 
        user: 'arvind.ravulavaru@gmail.com', 
        pass: '**********' 
    } 
}); 

var timerId; 

module.exports.sendEmail = function(file, deleteAfterUpload, cb) { 
    if (timerId) return; 

    timerId = setTimeout(function() { 
        clearTimeout(timerId); 
        timerId = null; 
    }, 10000); 

    console.log('Sendig an Email..'); 

    var mailOptions = { 
        from: 'Pi Bot <pi.intruder.alert@gmail.com>', 
        to: 'user@email.com', 
        subject: '[Pi Bot] Intruder Detected', 
        html: 'Intruder Detected. Please check the video attached. <br/><br/> Intruder Detected At : ' + Date(), 
        attachments: [{ 
            path: file 
        }] 
    }; 

    transporter.sendMail(mailOptions, function(err, info) { 
        if (err) { 
            console.log(err); 
        } else { 
            console.log('Message sent: ' + info.response); 
            if (deleteAfterUpload) { 
                fs.unlink(path); 
            } 
        } 

        if (cb) { 
            cb(err, info); 
        } 
    }); 
} 
```

我们使用 nodemailer 发送电子邮件。根据需要更新凭据。

接下来，运行以下命令：

```js
npm install onoff -save  
```

在下一节中，我们将测试这个设置。

# 测试代码

现在我们已经完成设置，让我们来测试一下。给树莓派供电，如果尚未上传代码，则上传代码，并运行以下命令：

```js
npm start
```

代码运行后，触发一次运动。这将启动摄像头录制并保存 5 秒的视频。然后将此视频通过电子邮件发送给用户。以下是输出的列表：

![](img/00117.jpeg)

收到的电子邮件将如下所示：

![](img/00118.jpeg)

这是使用树莓派 3 进行监视的另一种方法。

# 总结

在本章中，我们已经看到了使用树莓派进行监视的两种方法。第一种方法是我们将图像流式传输到 API 引擎，然后在移动 Web 和桌面应用程序上使用 MJPEG 进行可视化。第二种方法是检测运动，然后开始录制视频。然后将此视频作为附件通过电子邮件发送。这两种方法也可以结合在一起，如果在第一种方法中检测到运动，则可以开始 MJPEG 流式传输。

在第九章中，*智能监控*，我们将把这个提升到下一个级别，我们将在我们的捕获图像上添加人脸识别，并使用 AWS Rekognition 平台进行人脸识别（而不是人脸检测）。
