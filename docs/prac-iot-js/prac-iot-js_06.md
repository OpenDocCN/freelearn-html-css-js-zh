# 第六章：智能可穿戴设备

在本章中，我们将介绍如何使用树莓派 3 创建一个简单的医疗保健应用程序。我们将构建一个带有 16x2 液晶显示屏的智能可穿戴设备，显示用户的位置，并在 Web/桌面/移动界面上显示加速度计的数值。这个产品的目标用户主要是年长者，主要用例是跌倒检测，我们将在第七章中进行讨论，*智能可穿戴设备和 IFTTT*。

在本章中，我们将讨论以下内容：

+   物联网和医疗保健

+   设置所需的硬件

+   整合加速度计并查看实时数据

# 物联网和医疗保健

想象一位成功接受心脏移植手术并在医院术后护理后被送回家的患者。对这位患者的关注程度将显著降低，因为家庭设施与医院相比将是最低的。这就是物联网以其实时能力介入的地方。

物联网和医疗保健是天作之合。风险和回报同样巨大。能够实时监测患者的健康状况，并获取他们的脉搏、体温和其他重要统计数据的信息，并对其进行诊断和处理是非常宝贵的。与此同时，如果连接中断两分钟，就会有人的生命受到威胁。

在我看来，要实现物联网在医疗保健领域的全部潜力，我们可能需要再等待 5-10 年，那时的连接将是绝对无缝的，数据包丢失将成为历史。

# 智能可穿戴设备

如前一节所述，我们将使用物联网在医疗保健领域做一些关键的事情。我们要构建的智能可穿戴设备的主要目的是识别跌倒。一旦识别到跌倒，我们就会通知云端。当我们周围有年长或患病的人因意外原因而倒下时，及时识别跌倒并采取行动有时可以挽救生命。

为了检测跌倒，我们将使用加速度计。引用维基百科的话：

"**加速度计**是一种测量真实加速度的设备。真实加速度是指物体在其瞬时静止参考系中的加速度（或速度变化率），并不同于坐标加速度，即在固定坐标系中的加速度。例如，静止在地球表面的加速度计将测量由于地球重力而产生的加速度，垂直向上（根据定义）为 g ≈ 9.81 m/s2。相比之下，自由下落的加速度计（以约 9.81 m/s2 的速率朝向地球中心下落）将测量为零。"

要了解更多关于加速度计及其工作原理的信息，请参阅*加速度计的工作原理*：[`www.youtube.com/watch?v=i2U49usFo10`](https://www.youtube.com/watch?v=i2U49usFo10)。

在本章中，我们将实现基本系统，收集 X、Y 和 Z 轴加速度原始值，并在 Web、桌面和移动应用程序上显示。在第七章中，*智能可穿戴设备和 IFTTT*，我们将使用这些数值来实现跌倒检测。

除了实时收集加速度计数值外，我们还将使用 16x2 液晶显示屏显示当前时间和用户的地理位置。如果需要，我们也可以在显示屏上添加其他文本。16x2 是一个简单的界面来显示内容。这可以通过诺基亚 5110 液晶屏（[`www.amazon.in/inch-Nokia-5110-KG075-KitsGuru/dp/B01CXNSJOA`](http://www.amazon.in/inch-Nokia-5110-KG075-KitsGuru/dp/B01CXNSJOA)）进行扩展，以获得具有图形的更高级显示。

在接下来的部分，我们将组装所需的硬件，然后更新树莓派代码。之后，我们将开始处理 API 引擎和 UI 模板。

# 设置智能可穿戴设备

关于硬件设置的第一件事是它又大又笨重。这只是一个 POC，甚至不是一个接近生产设置的远程。硬件设置将包括连接到树莓派 3 和 16X2 LCD 的加速度计。

加速度计 ADXL345 通过 I2C 协议提供 X、Y 和 Z 轴的加速度。

按照以下方式连接硬件：

![](img/00093.jpeg)

正如您在上面的原理图中所看到的，我们已经建立了以下连接：

+   树莓派和 LCD：

| **树莓派编号 - 引脚名称** | **16x2 LCD Pi 名称** |
| --- | --- |
| 6 - GND - 面包板导轨 1 | 1 - GND |
| 2 - 5V - 面包板导轨 2 | 2 - VCC |
| 1 k Ohm 电位计 | 3 - VEE |
| 32 - GPIO 12 | 4 - RS |
| 6 - GND - 面包板导轨 1 | 5 -R/W |
| 40 - GPIO 21 | 6 - EN |
| NC | 7 - DB0 |
| NC | 8 - DB1 |
| NC | 9 - DB2 |
| NC | 10 - DB3 |
| 29 - GPIO 5 | 11 - DB4 |
| 31 - GPIO 6 | 12 - DB5 |
| 11 - GPIO 17 | 13 - DB6 |
| 12 - GPIO 18 | 14 - DB7 |
| 2 - 5V - 面包板导轨 2 | 15 - LED+ |
| 6 - GND - 面包板导轨 1 | 16 - LED- |

+   树莓派和 ADXL345：

| **树莓派编号 - 引脚名称** | **ADXL345 引脚编号 - 引脚名称** |
| --- | --- |
| 1 - 3.3V | VCC |
| 6 - GND - 面包板导轨 1 | GND |
| 5 - GPIO3/SCL1 | SCL |
| 3 - GPIO2/SDA1 | SDA |
| 6 - GND - 面包板导轨 1 | SDO |

我们将添加所需的代码：

1.  首先创建一个名为`chapter6`的文件夹，然后将`chapter4`的内容复制到其中。我们将随着进展更新此代码

1.  现在，我们将开始使用`pi-client`。在树莓派上，打开`pi-client/index.js`并按照以下方式更新它：

```js
var config = require('./config.js'); 
var mqtt = require('mqtt'); 
var GetMac = require('getmac'); 
var request = require('request'); 
var ADXL345 = require('adxl345-sensor'); 
require('events').EventEmitter.prototype._maxListeners = 100; 

var adxl345 = new ADXL345(); // defaults to i2cBusNo 1, i2cAddress 0x53 

var Lcd = require('lcd'), 
    lcd = new Lcd({ 
        rs: 12, 
        e: 21, 
        data: [5, 6, 17, 18], 
        cols: 8, 
        rows: 2 
    }); 

var aclCtr = 0, 
    locCtr = 0; 

var x, prevX, y, prevY, z, prevZ; 
var locationG; // global location variable 

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
    client.subscribe('socket'); 
    GetMac.getMac(function(err, mac) { 
        if (err) throw err; 
        macAddress = mac; 
        displayLocation(); 
        initADXL345(); 
        client.publish('api-engine', mac); 
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

function initADXL345() { 
    adxl345.init().then(function() { 
            console.log('ADXL345 initialization succeeded'); 
            // init loop after ADXL345 has been setup 
            loop(); 
        }) 
        .catch(function(err) { 
            console.error('ADXL345 initialization failed: ', err); 
        }); 
} 

function loop() { 
    // infinite loop, with 1 seconds delay 
    setInterval(function() { 
        // wait till we get the location 
        // then start processing 
        if (!locationG) return; 

        if (aclCtr === 3) { // every 3 seconds 
            aclCtr = 0; 
            readSensorValues(function(acclVals) { 
                var x = acclVals.x; 
                var y = acclVals.y; 
                var z = acclVals.z; 

                var data2Send = { 
                    data: { 
                        acclVals: acclVals, 
                        location: locationG 
                    }, 
                    macAddress: macAddress 
                }; 

                // no duplicate data 
                if (x !== prevX || y !== prevY || z !== prevZ) { 
                    console.log('data2Send', data2Send); 
                    client.publish('accelerometer', JSON.stringify(data2Send)); 
                    console.log('Data Published'); 
                    prevX = x; 
                    prevY = y; 
                    prevZ = z; 
                } 
            }); 
        } 

        if (locCtr === 300) { // every 300 seconds 
            locCtr = 0; 
            displayLocation(); 
        } 

        aclCtr++; 
        locCtr++; 
    }, 1000); // every one second 
} 

function readSensorValues(CB) { 
    adxl345.getAcceleration(true) // true for g-force units, else false for m/s² 
        .then(function(acceleration) { 
            if (CB) CB(acceleration); 
        }) 
        .catch((err) => { 
            console.log('ADXL345 read error: ', err); 
        }); 
} 

function displayLocation() { 
    request('http://ipinfo.io', function(error, res, body) { 
        var info = JSON.parse(body); 
        // console.log(info); 
        locationG = info; 
        var text2Print = ''; 
        text2Print += 'City: ' + info.city; 
        text2Print += ' Region: ' + info.region; 
        text2Print += ' Country: ' + info.country + ' '; 
        lcd.setCursor(16, 0); // 1st row     
        lcd.autoscroll(); 
        printScroll(text2Print); 
    }); 
} 

// a function to print scroll 
function printScroll(str, pos) { 
    pos = pos || 0; 

    if (pos === str.length) { 
        pos = 0; 
    } 

    lcd.print(str[pos]); 
    //console.log('printing', str[pos]); 

    setTimeout(function() { 
        return printScroll(str, pos + 1); 
    }, 300); 
} 

// If ctrl+c is hit, free resources and exit. 
process.on('SIGINT', function() { 
    lcd.clear(); 
    lcd.close(); 
    process.exit(); 
}); 
```

从上述代码中可以看出，我们使用`displayLocation()`每小时显示一次位置，因为我们假设位置不会经常改变。我们使用[`ipinfo.io/`](http://ipinfo.io/)服务来获取用户的位置。

1.  最后，使用`readSensorValues()`我们每`3`秒获取一次`加速度计`的值，并将这些数据发布到名为`accelerometer`的主题中。

1.  现在，我们将安装所需的依赖项。从`pi-client`文件夹内部运行以下命令：

```js
npm install async getmac adxl345-sensor mqtt request --save
```

1.  保存所有文件并通过运行在服务器或我们的桌面机器上启动 mosca broker 来启动：

```js
mosca -c index.js -v | pino  
```

1.  接下来，在树莓派上运行代码：

```js
npm start  
```

这将启动`pi-client`并开始收集加速度计数据，并在 LCD 显示器上显示位置如下：

![](img/00094.jpeg)

我的设置如下所示：

![](img/00095.jpeg)

接下来，我们将与 API 引擎一起工作。

# 更新 API 引擎

现在我们已经让智能可穿戴设备运行并发送了三轴数据，我们现在将实现 API 引擎中接受该数据所需的逻辑，并将数据发送到 Web/桌面/移动应用程序中：

打开`api-engine/server/mqtt/index.js`并按照以下方式更新它：

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
    client.subscribe('accelerometer'); 
}); 

client.on('message', function(topic, message) { 
    // message is Buffer 
    // console.log('Topic >> ', topic); 
    // console.log('Message >> ', message.toString()); 
    if (topic === 'api-engine') { 
        var macAddress = message.toString(); 
        console.log('Mac Address >> ', macAddress); 
        client.publish('rpi', 'Got Mac Address: ' + macAddress); 
    } else if (topic === 'accelerometer') { 
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
```

在这里，我们订阅名为`accelerometer`的主题，并监听其变化。接下来，我们将按照以下方式更新`api-engine/server/api/data/data.controller.js`：

```js
'use strict'; 

var Data = require('./data.model'); 

/** 
 * Get Data for a device 
 */ 
exports.index = function(req, res) { 
    var macAddress = req.params.deviceId; 
    var limit = parseInt(req.params.limit) || 30; 

    Data 
        .find({ 
            macAddress: macAddress 
        }) 
        .sort({ 'createdAt': -1 }) 
        .limit(limit) 
        .exec(function(err, data) { 
            if (err) return res.status(500).send(err); 
            res.status(200).json(data); 
        }); 
}; 

/** 
 * Create a new data record 
 */ 
exports.create = function(req, res, next) { 
    var data = req.body || {}; 
    data.createdBy = req.user._id; 

    Data.create(data, function(err, _data) { 
        if (err) return res.status(500).send(err); 
        return res.json(_data); 
    }); 
}; 
```

上述代码用于将数据保存到数据库，并在从 Web、桌面和移动应用程序请求时从数据库中获取数据。

保存所有文件并运行 API 引擎：

```js
npm start
```

这将启动 API 引擎，如果需要，我们可以重新启动智能可穿戴设备，我们应该看到以下内容：

![](img/00096.jpeg)

在下一节中，我们将在 Web 应用程序中显示数据。

# 更新 Web 应用程序

现在我们已经完成了 API 引擎，我们将更新 Web 应用程序中的模板以显示三轴数据。打开`web-app/src/app/device/device.component.html`并按照以下方式更新它：

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
            <tr *ngIf="lastRecord">
              <td>X-Axis</td>
              <td>{{lastRecord.data.acclVals.x}} {{lastRecord.data.acclVals.units}}</td>
            </tr>
            <tr *ngIf="lastRecord">
              <td>Y-Axis</td>
              <td>{{lastRecord.data.acclVals.y}} {{lastRecord.data.acclVals.units}}</td>
            </tr>
            <tr *ngIf="lastRecord">
              <td>Z-Axis</td>
              <td>{{lastRecord.data.acclVals.z}} {{lastRecord.data.acclVals.units}}</td>
            </tr>
            <tr *ngIf="lastRecord">
              <td>Location</td>
              <td>{{lastRecord.data.location.city}}, {{lastRecord.data.location.region}}, {{lastRecord.data.location.country}}</td>
            </tr>
            <tr *ngIf="lastRecord">
              <td>Received At</td>
              <td>{{lastRecord.createdAt | date : 'medium'}}</td>
            </tr>
          </table>
          <hr>
          <div class="col-md-12" *ngIf="acclVals.length > 0">
            <canvas baseChart [datasets]="acclVals" [labels]="lineChartLabels" [options]="lineChartOptions" [legend]="lineChartLegend" [chartType]="lineChartType"></canvas>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
```

所需的逻辑将在`device.component.ts`中。打开`web-app/src/app/device/device.component.ts`并按照以下方式更新它：

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
          // steps: 10,
          // stepValue: 5,
          // max: 70
        }
      }],
      zAxes: [{
        display: true,
        ticks: {
          beginAtZero: true,
          // steps: 10,
          // stepValue: 5,
          // max: 70
        }
      }]
    },
    title: {
      display: true,
      text: 'X,Y,Z vs. Time'
    }
  };

  public lineChartLegend: boolean = true;
  public lineChartType: string = 'line';
  public acclVals: Array<any> = [];
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
      });
    });
  }

  getData() {
    this.dataService.get(this.device.macAddress).subscribe((response) => {
      this.data = response.json();
      this.lastRecord = this.data[0]; // descending order data
      this.toggleState = this.lastRecord.data.s;
      this.genChart();
      this.socketInit();
    });
  }

  socketInit() {
    this.subData = this.socketService.getData(this.device.macAddress).subscribe((data) => {
      if (this.data.length <= 0) return;
      this.data.splice(this.data.length - 1, 1); // remove the last record
      this.data.push(data); // add the new one
      this.lastRecord = data;
      this.toggleState = this.lastRecord.data.s;
      this.genChart();
    });
  }

  ngOnDestroy() {
    this.subDevice.unsubscribe();
    this.subData ? this.subData.unsubscribe() : '';
  }

  genChart() {
    let data = this.data;
    let _acclVals: Array<any> = [];
    let _lblArr: Array<any> = [];

    let xArr: Array<any> = [];
    let yArr: Array<any> = [];
    let zArr: Array<any> = [];

    for (var i = 0; i < data.length; i++) {
      let _d = data[i];
      xArr.push(_d.data.acclVals.x);
      yArr.push(_d.data.acclVals.y);
      zArr.push(_d.data.acclVals.z);
      _lblArr.push(this.formatDate(_d.createdAt));
    }

    // reverse data to show the latest on the right side
    xArr.reverse();
    yArr.reverse();
    zArr.reverse();
    _lblArr.reverse();

    _acclVals = [
      {
        data: xArr,
        label: 'X-Axis'
      },
      {
        data: yArr,
        label: 'Y-Axis'
      },
      {
        data: zArr,
        label: 'Z-Axis'
      }
    ]

    this.acclVals = _acclVals;

    this.lineChartLabels = _lblArr;
  }

  private formatDate(originalTime) {
    var d = new Date(originalTime);
    var datestring = d.getDate() + "-" + (d.getMonth() + 1) + "-" + d.getFullYear() + " " +
      d.getHours() + ":" + d.getMinutes();
    return datestring;
  }

}
```

保存所有文件并运行以下命令：

```js
npm start  
```

导航到`http://localhost:4200`并查看设备，我们应该看到以下内容：

![](img/00097.jpeg)

通过这样，我们已经完成了 Web 应用程序。

# 更新桌面应用程序

现在 Web 应用程序已经完成，我们将构建相同的应用程序并将其部署到我们的桌面应用程序中。

要开始，请返回到`web-app`文件夹的终端/提示符，并运行：

```js
ng build --env=prod
```

这将在`web-app`文件夹内创建一个名为`dist`的新文件夹。`dist`文件夹的内容应该类似于以下内容：

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

我们编写的所有代码最终都打包到了前面的文件中。我们将获取`dist`文件夹中的所有文件（而不是`dist`文件夹），然后将其粘贴到`desktop-app/app`文件夹中。在进行前述更改后，桌面应用程序的最终结构将如下所示：

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

要进行测试，请运行以下命令：

```js
npm start
```

然后当我们导航到 VIEW DEVICE 页面时，我们应该看到以下屏幕：

![](img/00098.jpeg)

通过这样，我们已经完成了桌面应用程序的开发。在下一节中，我们将更新移动应用程序。

# 更新移动应用程序模板

在上一节中，我们已经更新了桌面应用程序。在本节中，我们将更新移动应用程序模板以显示三轴数据。

首先，我们要更新 view-device 模板。按照以下步骤更新`mobile-app/src/pages/view-device/view-device.html`：

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
                <ion-label>Name</ion-label>
                <ion-label>{{device.name}}</ion-label>
            </ion-item>
            <ion-item>
                <ion-label>X-Axis</ion-label>
                <ion-label>{{lastRecord.data.acclVals.x}} {{lastRecord.data.acclVals.units}}</ion-label>
            </ion-item>
            <ion-item>
                <ion-label>Y-Axis</ion-label>
                <ion-label>{{lastRecord.data.acclVals.y}} {{lastRecord.data.acclVals.units}}</ion-label>
            </ion-item>
            <ion-item>
                <ion-label>Z-Axis</ion-label>
                <ion-label>{{lastRecord.data.acclVals.z}} {{lastRecord.data.acclVals.units}}</ion-label>
            </ion-item>
            <ion-item>
                <ion-label>Location</ion-label>
                <ion-label>{{lastRecord.data.location.city}}, {{lastRecord.data.location.region}}, {{lastRecord.data.location.country}}</ion-label>
            </ion-item>
            <ion-item>
                <ion-label>Received At</ion-label>
                <ion-label>{{lastRecord.createdAt | date: 'medium'}}</ion-label>
            </ion-item>
        </ion-list>
    </div>
</ion-content>
```

接下来，按照以下步骤更新`mobile-app/src/pages/view-device/view-device.ts`：

```js
import { Component } from '@angular/core'; 
import { IonicPage, NavController, NavParams } from 'ionic-angular'; 

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
               this.socketInit(); 
         }); 
   } 

   getData() { 
         this.dataService.get(this.device.macAddress).subscribe((response) => { 
               this.data = response.json(); 
               this.lastRecord = this.data[0]; // descending order data 
         }); 
   } 

   socketInit() { 
         this.subData = this.socketService.getData(this.device.macAddress).subscribe((data) => { 
               if (this.data.length <= 0) return; 
               this.data.splice(this.data.length - 1, 1); // remove the last record 
               this.data.push(data); // add the new one 
               this.lastRecord = data; 
         }); 
   } 

   ionViewDidUnload() { 
         this.subData && this.subData.unsubscribe && this.subData.unsubscribe(); //unsubscribe if subData is defined 
   } 
} 
```

保存所有文件，并通过`ionic serve`或`ionic cordova run android`来运行移动应用程序。

然后我们应该看到以下内容：

![](img/00099.gif)

通过这样，我们已经完成了在移动应用程序上显示智能可穿戴设备数据的工作。

# 摘要

在本章中，我们已经看到如何使用 Raspberry Pi 3 构建一个简单的智能可穿戴设备。我们设置了一个液晶显示屏和一个三轴加速度计，并在显示屏上显示了位置信息。我们实时将加速度计数据发布到云端，并在 Web、桌面和移动应用程序上显示出来。

在第七章，*智能可穿戴设备和 IFTTT*中，我们将通过在其上实施 IFTTT 规则，将智能可穿戴设备提升到一个新的水平。我们将执行诸如打电话或向急救联系人发送短信等操作，以便及时提供护理。
