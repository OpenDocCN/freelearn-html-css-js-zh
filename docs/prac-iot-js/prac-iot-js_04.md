# 第四章：智能农业

在本章中，我们将使用我们在第二章中构建的框架，*IoTFW.js - I*和第三章，*IoTFW.js - II*，并开始处理各种用例。我们将从农业领域开始，在本章中建立一个智能气象站。

任何农民的一个简单要求是能够了解他们农场附近和周围的环境因素。因此，我们将建立一个便携式气象站的原型。在本章中，我们将研究以下主题：

+   农业和物联网

+   设计智能气象站

+   为 Raspberry Pi 3 开发代码

+   在 API 引擎中更新 MQTT 代码

+   修改 Web 应用程序、桌面应用程序和移动应用程序的模板

# 农业和物联网

Beecham Research 的一份报告预测，到 2025 年，全球人口将达到 80 亿，到 2050 年将达到 96 亿，为了跟上步伐，到 2050 年，粮食产量必须增加 70%。这是报告：[`www.beechamresearch.com/files/BRL%20Smart%20Farming%20Executive%20Summary.pdf`](https://www.beechamresearch.com/files/BRL%20Smart%20Farming%20Executive%20Summary.pdf)

这意味着我们需要找到更快更有效的耕作方式。随着我们不断朝着 2050 年迈进，土地和资源将变得更加稀缺。这是因为，如果有资源的话，我们需要喂饱比以往任何时候都更多的人，除非有僵尸启示录发生，我们所有人都被其他僵尸吃掉！

这是技术提供解决方案的一个很好的机会。物联网几乎一直是关于智能家居、智能办公室和便利性，但它可以做得更多。这就是我们将在本章中涵盖的内容。我们将建立一个智能气象站，农民可以在他们的农场部署，以获取实时的温度、湿度、土壤湿度和雨水检测等指标。

其他传感器可以根据可用性和需求进行添加。

# 设计智能气象站

既然我们知道我们要构建什么以及为什么，让我们开始设计。我们要做的第一件事是确定所需的传感器。在这个智能气象站中，我们将使用以下传感器：

+   温度传感器

+   湿度传感器

+   土壤湿度传感器

+   雨水检测传感器

我选择了现成的传感器，以展示概念的证明。它们中的大多数对于测试和验证想法或作为爱好来说都很好，但不适合生产。

我们将把这些传感器连接到我们的 Raspberry Pi 3 上。我们将使用以下型号的传感器：

+   温度和湿度：[`www.amazon.com/Gowoops-Temperature-Humidity-Digital-Raspberry/dp/B01H3J3H82/ref=sr_1_5`](https://www.amazon.com/Gowoops-Temperature-Humidity-Digital-Raspberry/dp/B01H3J3H82/ref=sr_1_5)

+   土壤湿度传感器：[`www.amazon.com/Hygrometer-Humidity-Detection-Moisture-Arduino/dp/B01FDGUHBM/ref=sr_1_4`](https://www.amazon.com/Hygrometer-Humidity-Detection-Moisture-Arduino/dp/B01FDGUHBM/ref=sr_1_4)

+   雨水检测传感器：[`www.amazon.com/Uxcell-a13082300ux1431-Rainwater-Detection-3-3V-5V/dp/B00GN7O7JE`](https://www.amazon.com/Uxcell-a13082300ux1431-Rainwater-Detection-3-3V-5V/dp/B00GN7O7JE)

您也可以在其他地方购买这些传感器。

正如我们在[第三章](https://cdp.packtpub.com/b07286advancediotwithjavascripteas/wp-admin/post.php?post=266&action=edit#post_235)中看到的，温度和湿度传感器是数字传感器，我们将使用`node-dht-sensor`模块来读取温度和湿度值。土壤湿度传感器是模拟传感器，而 Raspberry Pi 3 没有任何模拟引脚。因此，我们将使用 Microchip 的 MCP3208 12 位 A/D IC，将传感器的模拟输出转换为数字，并通过 SPI 协议传输到 Raspberry Pi。

维基百科对 SPI 协议的定义如下：

**串行外围接口**（**SPI**）总线是一种用于短距离通信的同步串行通信接口规范，主要用于嵌入式系统。该接口是由 Motorola 在 20 世纪 80 年代后期开发的，并已成为事实上的标准。有关 SPI 的更多信息，请参阅：[`en.wikipedia.org/wiki/Serial_Peripheral_Interface_Bus`](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface_Bus)。

雨水检测传感器可以被读取为模拟和数字信号。我们将使用模拟输出来检测雨水的级别，而不仅仅是是否下雨。

回到 MCP3208，它是一个 16 引脚封装，可以同时读取八个模拟输入，并可以将它们转换并通过 SPI 协议馈送到树莓派。您可以在这里阅读更多关于 MCP3208 IC 的信息：[`http://ww1.microchip.com/downloads/en/DeviceDoc/21298c.pdf`](http://ww1.microchip.com/downloads/en/DeviceDoc/21298c.pdf)。您可以从这里购买：[`www.amazon.com/Adafruit-MCP3008-8-Channel-Interface-Raspberry/dp/B00NAY3RB2/ref=sr_1_1`](https://www.amazon.com/Adafruit-MCP3008-8-Channel-Interface-Raspberry/dp/B00NAY3RB2/ref=sr_1_1)。

我们将直接将温度和湿度传感器连接到树莓派 3，将湿度传感器和雨水传感器连接到 MCP3208，然后将 MCP3208 通过 SPI 连接到树莓派 3。

在代理上，我们不打算改变任何东西。在 API 引擎中，我们将向 MQTT 客户端添加一个名为`weather-status`的新主题，然后将来自树莓派 3 的数据发送到这个主题。

在 Web 应用程序中，我们将更新用于查看天气指标的模板。桌面应用程序和移动应用程序也是如此。

# 设置树莓派 3

让我们开始制作原理图。

设置树莓派 3 和传感器如下所示：

![](img/00054.jpeg)

这里是一个显示这些连接的表格：

# 树莓派和 MCP3208

参考以下表格：

| **树莓派引脚编号 - 引脚名称** | **MCP3208 引脚编号 - 引脚名称** |
| --- | --- |
| 1 - 3.3V | 16 - VDD |
| 1 - 3.3V | 15 - AREF |
| 6 - GND | 14 - AGND |
| 23 - GPIO11, SPI0_SCLK | 13 - CLK |
| 21 - GPIO09, SPI0_MISO | 12 - DOUT |
| 19 -GPIO10, SPI0_MOSI | 11 - DIN |
| 24 - GPIO08, CEO | 10 - CS |
| 6 - GND | 9 - DGND |

# 湿度传感器和 MCP3208

参考以下表格：

| **MCP3208 引脚编号 - 引脚名称** | **传感器名称** - **引脚编号** |
| --- | --- |
| 1 - A0 | 雨水传感器 - A0 |
| 2 - A1 | 湿度传感器 - A0 |

# 树莓派和 DHT11

参考以下表格：

| **树莓派编号 - 引脚名称** | **传感器名称** - **引脚编号** |
| --- | --- |
| 3 - GPIO2 | DHT11 - Data |

所有接地和所有 3.3V 都连接到一个公共点。

一旦我们按照之前显示的方式连接了传感器，我们将编写所需的代码来与传感器进行接口。

在我们继续之前，我们将把整个第二章，*IoTFW.js - I*，和第三章，*IoTFW.js - II*，的代码复制到另一个名为`chapter4`的文件夹中。

`chapter4`文件夹应该如下所示：

```js
.

├── api-engine

│ ├── package.json

│ └── server

├── broker

│ ├── certs

│ └── index.js

├── desktop-app

│ ├── app

│ ├── freeport.js

│ ├── index.css

│ ├── index.html

│ ├── index.js

│ ├── license

│ ├── package.json

│ ├── readme.md

│ └── server.js

├── mobile-app

│ ├── config.xml

│ ├── hooks

│ ├── ionic.config.json

│ ├── package.json

│ ├── platforms

│ ├── plugins

│ ├── resources

│ ├── src

│ ├── tsconfig.json

│ ├── tslint.json

│ └── www

└── web-app

├── README.md

├── e2e

├── karma.conf.js

├── package.json

├── protractor.conf.js

├── src

├── tsconfig.json

└── tslint.json
```

我们将返回到树莓派，并在`pi-client`文件夹中更新`index.js`文件。更新`pi-client/index.js`，如下所示：

```js
var config = require('./config.js');

var mqtt = require('mqtt');

var GetMac = require('getmac');

var async = require('async');

var rpiDhtSensor = require('rpi-dht-sensor');

var McpAdc = require('mcp-adc');

var adc = new McpAdc.Mcp3208();

var dht11 = new rpiDhtSensor.DHT11(2);

var temp = 0,

prevTemp = 0;

var humd = 0,

prevHumd = 0;

var macAddress;

var state = 0;

var moistureVal = 0,

prevMoistureVal = 0;

var rainVal = 0,

prevRainVal = 0;

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

// infinite loop, with 3 seconds delay

setInterval(function() {

readSensorValues(function(results) {

console.log('Temperature: ' + temp + 'C, ' + 'humidity: ' + humd + '%, ' + ' Rain level (%):' + rainVal + ', ' + 'moistureVal (%): ' + moistureVal);

// if the temperature and humidity values change

// then only publish the values

if (temp !== prevTemp || humd !== prevHumd || moistureVal !== prevMoistureVal || rainVal != prevRainVal) {

var data2Send = {

data: {

t: temp,

h: humd,

r: rainVal,

m: moistureVal

},

macAddress: macAddress

};

// console.log('Data Published');

client.publish('weather-status', JSON.stringify(data2Send));

// reset prev values to current

// for next loop

prevTemp = temp;

prevHumd = humd;

prevMoistureVal = moistureVal;

prevRainVal = rainVal;

}

});

}, 3000); // every three second

// `CB` expects {

// dht11Values: val,

// rainLevel: val,

// moistureLevel: val

// }

function readSensorValues(CB) {

async.parallel({

dht11Values: function(callback) {

var readout = dht11.read();

// update global variable

temp = readout.temperature.toFixed(2);

humd = readout.humidity.toFixed(2);

callback(null, { temp: temp, humidity: humd });

},

rainLevel: function(callback) {

// we are going to connect rain sensor

// on channel 0, hence 0 is the first arg below

adc.readRawValue(0, function(value) {

// update global variable

rainVal = value;

rainVal = (100 - parseFloat((rainVal / 4096) * 100)).toFixed(2);

callback(null, { rain: rainVal });

});

},

moistureLevel: function(callback) {

// we are going to connect moisture sensor

// on channel 1, hence 1 is the first arg below

adc.readRawValue(1, function(value) {

// update global variable

moistureVal = value;

moistureVal = (100 - parseFloat((moistureVal / 4096) * 100)).toFixed(2);

callback(null, { moisture: moistureVal });

});

}

}, function done(err, results) {

if (err) {

throw err;

}

// console.log(results);

if (CB) CB(results);

});

}
```

在前面的代码中，我们保留了 MQTT 设置。我们添加了`mcp-adc`（[`github.com/anha1/mcp-adc`](https://github.com/anha1/mcp-adc)）和`async`（[`github.com/caolan/async`](https://github.com/caolan/async)）模块。`mcp-adc`管理 MCP3208 暴露的 SPI 协议接口，我们使用`async`模块并行读取所有传感器的数据。

我们首先通过 MQTTS 与代理建立连接。然后，我们使用`setInterval()`设置了一个间隔为 3 秒的无限循环。在`setInterval()`的`callback`中，我们调用`readSensorValues()`来获取最新的传感器数值。

`readSensorValues()`使用`async.parallel()`并行读取三个传感器的数据，并更新我们定义的全局变量中的数据。一旦收集到所有传感器数据，我们将结果作为参数传递给`callback`函数。

一旦我们收到传感器数据，我们将检查温度、湿度、雨量和湿度数值之间是否有变化。如果没有变化，我们就放松；否则，我们将把这些数据发布到天气状态主题的代理上。

保存所有文件。现在，我们将从我们的桌面机器上启动 Mosca 代理：

```js
mosca -c index.js -v | pino
```

一旦您启动了 Mosca 服务器，请检查 Mosca 运行的服务器的 IP 地址。在树莓派的`config.js`文件中更新相同的 IP。否则，树莓派无法将数据发布到代理。

一旦 Mosca 成功启动并且我们已经验证了 IP，就在树莓派上运行这个：

```js
sudo node index.js
```

这将启动服务器，我们应该看到以下内容：

![](img/00055.jpeg)

当我启动树莓派时，雨传感器是干的，湿度传感器被放置在干燥的土壤中。最初，雨传感器的值为`1.86%`，湿度传感器的值为`4.57%`。

当我向植物/湿度传感器添加水时，百分比增加到`98.83%`；同样，当我在雨传感器上模拟降雨时，数值上升到`89.48%`。

这是我智能气象站的原型设置：

![](img/00056.jpeg)![](img/00057.jpeg)![](img/00058.jpeg)![](img/00059.jpeg)蓝色芯片是 DHT11，湿度传感器被放置在我的桌边植物中，雨传感器被放置在一个塑料托盘中，用于收集雨水。面包板上有 MCP3208 IC 和所需的连接。

很多电线！

通过这样，我们完成了树莓派 3 所需的代码。在下一节中，我们将设置 API 引擎所需的代码。

# 设置 API 引擎

在最后一节中，我们已经看到了如何设置组件和代码，以便使用树莓派 3 建立智能气象站。现在，我们将致力于管理从树莓派 3 接收的数据的 API 引擎。

打开`api-engine/server/mqtt/index.js`并更新，如下所示：

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
client.subscribe('weather-status'); 
}); 

client.on('message', function(topic, message) { 
    // message is Buffer 
    // console.log('Topic >> ', topic); 
    // console.log('Message >> ', message.toString()); 
if (topic === 'api-engine') { 
varmacAddress = message.toString(); 
console.log('Mac Address >> ', macAddress); 
client.publish('rpi', 'Got Mac Address: ' + macAddress); 
    } else if (topic === 'weather-status') { 
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

在这里，我们正在等待`weather-status`主题上的消息，当我们从树莓派接收到数据时，我们将其保存到我们的数据库，并将数据推送到 Web 应用程序、移动应用程序和桌面应用程序。

这些是我们需要做的所有更改，以吸收来自树莓派 3 的数据并传递给 Web、桌面和移动应用程序。

保存所有文件并运行以下代码：

```js
npm start  
```

这将启动 API 引擎并连接到 Mosca，以及树莓派：

![](img/00060.jpeg)

如果我们让 API 引擎运行一段时间，我们应该会看到以下内容：

![](img/00061.jpeg)

设备的数据在这里记录。

在下一节中，我们将更新 Web 应用程序，以便表示来自 API 引擎的数据。

# 设置 Web 应用程序

现在我们已经完成了 API 引擎，我们将开发所需的界面，以显示来自树莓派 3 的天气输出。

打开`web-app/src/app/device/device.component.html`并更新，如下所示：

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
                            <td>Temperature</td>
                            <td>{{lastRecord.data.t}}</td>
                        </tr>
                        <tr *ngIf="lastRecord">
                            <td>Humidity</td>
                            <td>{{lastRecord.data.h}} %</td>
                        </tr>
                        <tr *ngIf="lastRecord">
                            <td>Rain Level</td>
                            <td>{{lastRecord.data.r}} %</td>
                        </tr>
                        <tr *ngIf="lastRecord">
                            <td>Mositure Level</td>
                            <td>{{lastRecord.data.m}} %</td>
                        </tr>
                        <tr *ngIf="lastRecord">
                            <td>Received At</td>
                            <td>{{lastRecord.createdAt | date: 'medium'}}</td>
                        </tr>
                    </table>
                    <div class="col-md-6" *ngIf="tempHumdData.length > 0">
                        <canvas baseChart [datasets]="tempHumdData" [labels]="lineChartLabels" [options]="lineChartOptions" [legend]="lineChartLegend" [chartType]="lineChartType"></canvas>
                    </div>

                    <div class="col-md-6" *ngIf="rainMoisData.length > 0">
                        <canvas baseChart [datasets]="rainMoisData" [labels]="lineChartLabels" [options]="lineChartOptions" [legend]="lineChartLegend" [chartType]="lineChartType"></canvas>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

在上述代码中，我们在表中添加了四行，显示温度、湿度、雨量和湿度等级。我们还设置了画布来显示图表中的数值。

接下来是`DeviceComponent`的类定义，位于`web-app/src/app/device/device.component.ts`中。按照下面的示例更新`web-app/src/app/device/device.component.ts`：

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
   privatesubDevice: any; 
   privatesubData: any; 
   lastRecord: any; 

   // line chart config 
   publiclineChartOptions: any = { 
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
               }] 
         }, 
         title: { 
               display: true, 
               text: 'Sensor Data vs. Time' 
         } 
   }; 
   publiclineChartLegend: boolean = true; 
   publiclineChartType: string = 'line'; 
   publictempHumdData: Array<any> = []; 
   publicrainMoisData: Array<any> = []; 
   publiclineChartLabels: Array<any> = []; 

   constructor(private deviceService: DevicesService, 
         privatesocketService: SocketService, 
         privatedataService: DataService, 
         private route: ActivatedRoute, 
         privatenotificationsService: NotificationsService) { } 

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
               this.lastRecord = this.data[0]; // descending order data 
               this.genChart(); 
         }); 
   } 

   socketInit() { 
         this.subData = this.socketService.getData(this.device.macAddress).subscribe((data) => { 
               if (this.data.length<= 0) return; 
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
         let _thArr: Array<any> = []; 
         let _rmArr: Array<any> = []; 
         let _lblArr: Array<any> = []; 

         lettmpArr: Array<any> = []; 
         lethumArr: Array<any> = []; 
         letraiArr: Array<any> = []; 
         letmoiArr: Array<any> = []; 

         for (vari = 0; i<data.length; i++) { 
               let _d = data[i]; 
               tmpArr.push(_d.data.t); 
               humArr.push(_d.data.h); 
               raiArr.push(_d.data.r); 
               moiArr.push(_d.data.m); 
               _lblArr.push(this.formatDate(_d.createdAt)); 
         } 

         // reverse data to show the latest on the right side 
         tmpArr.reverse(); 
         humArr.reverse(); 
         raiArr.reverse(); 
         moiArr.reverse(); 
         _lblArr.reverse(); 

         _thArr = [ 
               { 
                     data: tmpArr, 
                     label: 'Temperature' 
               }, 
               { 
                     data: humArr, 
                     label: 'Humidity %' 
               } 
         ] 

         _rmArr = [ 
               { 
                     data: raiArr, 
                     label: 'Rain Levels' 
               }, 
               { 
                     data: moiArr, 
                     label: 'Moisture Levels' 
               } 
         ] 

         this.tempHumdData = _thArr; 
         this.rainMoisData = _rmArr; 

         this.lineChartLabels = _lblArr; 
   } 

   privateformatDate(originalTime) { 
         var d = new Date(originalTime); 
         vardatestring = d.getDate() + "-" + (d.getMonth() + 1) + "-" + d.getFullYear() + " " + 
               d.getHours() + ":" + d.getMinutes(); 
         returndatestring; 
   } 

} 
```

在上述代码中，我们使用了`ngOnInit`钩子，并发出请求以获取设备数据。使用`socketInit()`，连同数据，我们将为当前设备注册套接字数据事件。

在`getData()`中，我们从服务器获取数据，提取最新记录，并将其设置为`lastRecord`属性。最后，我们调用`genChart()`来绘制图表。

现在，我们已经完成了所需的更改。保存所有文件并运行以下命令：

```js
ng server
```

如果我们导航到`http://localhost:4200`，登录，并点击**DEVICE**，我们应该看到以下内容：

![](img/00062.jpeg)

每当数据发生变化时，我们应该看到 UI 会自动更新。

在下一节中，我们将构建相同的应用程序并在电子外壳中显示它。

# 设置桌面应用程序

在上一节中，我们为 Web 应用程序开发了模板和界面。在本节中，我们将构建相同的内容并将其放入桌面应用程序中。

要开始，请返回到`web-app`文件夹的终端/提示符，并运行以下命令：

```js
ng build --env=prod
```

这将在`web-app`文件夹内创建一个名为`dist`的新文件夹。`dist`文件夹的内容应包括：

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

我们编写的所有代码最终都打包到了上述文件中。我们将获取`dist`文件夹内的所有文件（而不是`dist`文件夹），然后将它们粘贴到`desktop-app/app`文件夹内。在进行上述更改后，桌面应用程序的最终结构将如下所示：

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

为了测试该过程，请运行以下命令：

```js
npm start
```

导航到**DEVICE**页面，我们应该看到以下内容：

![](img/00063.jpeg)

每当数据发生变化时，我们应该看到 UI 会自动更新。

通过这样，我们已经完成了桌面应用程序的开发。在下一节中，我们将更新移动应用程序。

# 设置移动应用程序

在上一节中，我们看到了如何为智能气象站构建和运行桌面应用程序。在本节中，我们将更新移动应用程序的模板，以显示气象站数据。

打开`mobile-app/src/pages/view-device/view-device.html`并更新它，如下所示：

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
                <ion-label>Temperature</ion-label>
                <ion-label>{{lastRecord.data.t}}</ion-label>
            </ion-item>
            <ion-item>
                <ion-label>Humidity</ion-label>
                <ion-label>{{lastRecord.data.h}} %</ion-label>
            </ion-item>
            <ion-item>
                <ion-label>Rain Level</ion-label>
                <ion-label>{{lastRecord.data.r}} %</ion-label>
            </ion-item>
            <ion-item>
                <ion-label>Moisture Level</ion-label>
                <ion-label>{{lastRecord.data.m}} %</ion-label>
            </ion-item>
            <ion-item>
                <ion-label>Received At</ion-label>
                <ion-label>{{lastRecord.createdAt | date: 'medium'}}</ion-label>
            </ion-item>
        </ion-list>
    </div>
</ion-content>
```

在上述代码中，我们在列表视图中创建了四个项目，用于显示温度、湿度、降雨量和湿度水平。`ViewDevicePage`类的所需逻辑将存在于`mobile-app/src/pages/view-device/view-device.ts`中。按照下面所示更新`mobile-app/src/pages/view-device/view-device.ts`：

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
   privatesubData: any; 
   lastRecord: any; 

   constructor(private navCtrl: NavController, 
         privatenavParams: NavParams, 
         privatesocketService: SocketService, 
         privatedeviceService: DevicesService, 
         privatedataService: DataService, 
         privatetoastService: ToastService) { 
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
               if(this.data.length<= 0) return; 
               this.data.splice(this.data.length - 1, 1); // remove the last record 
               this.data.push(data); // add the new one 
               this.lastRecord = data; 
         }); 
   } 

   ionViewDidUnload() { 
         this.subData&&this.subData.unsubscribe&&this.subData.unsubscribe(); //unsubscribe if subData is defined 
   } 
} 
```

在上述代码中，我们使用`getData()`从 API 引擎获取最新数据。然后，使用`socketInit()`，我们订阅了数据的最新更改。

检查 API 引擎运行的服务器的 IP 地址。在移动应用程序的`mobile-app/src/app/app.globals.ts`文件中更新相同的 IP。否则，移动应用程序无法与 API 引擎通信。

现在，保存所有文件并运行以下命令：

```js
ionic serve
```

或者，您也可以通过运行以下命令将其部署到您的设备上：

```js
ionic run android 
```

或

```js
 ionic run ios
```

应用程序启动后，当我们导航到**DEVICE**页面时，我们应该在屏幕上看到以下内容：

![](img/00064.gif)

从图像中可以看出，我们能够实时查看数据更新。

# 摘要

在本章中，我们利用了第二章和第三章中所学到的知识，构建了智能气象站的原型。我们首先确定了构建气象站所需的传感器。接下来，我们在树莓派 3 上设置了它们。我们编写了与传感器进行接口的所需代码。完成这些工作后，我们更新了 API 引擎，以从树莓派 3 上的新主题读取数据。一旦 API 引擎接收到数据，我们就将其保存在数据库中，然后通过 Web 套接字将其发送到 Web、桌面和移动应用程序。最后，我们更新了 Web、桌面和移动应用程序上的演示模板；然后，我们在 Web、桌面和移动应用程序上显示了来自树莓派的数据。

在第五章中，*智能农业和语音人工智能*，我们将使用亚马逊的 Alexa 和我们建立的智能气象站来进行语音人工智能的工作。
