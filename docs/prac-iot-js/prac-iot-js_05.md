# 第五章：智能农业和语音人工智能

在第四章中，*智能农业*，我们已经看到了物联网可以产生影响的主流领域之一；农业部门。在本章中，我们将把这一点提升到一个新的水平。使用亚马逊 Alexa 等语音人工智能引擎，我们将与我们建立的智能气象站交谈。

例如，一个农民可以问 Alexa *“Alexa，请问 smarty app 我的农场的湿度水平是多少”*，然后 Alexa 会回答 *“你的农场湿度水平是 20%。考虑现在浇水”*。然后，农民会说 *“Alexa，请问 smarty app 打开我的发动机”*，然后 Alexa 会打开它。很迷人，不是吗？

一般来说，基于语音人工智能的物联网在智能家居和智能办公的概念中更为常见。我想在智能农业中实现它。

在本章中，我们将致力于以下工作：

+   了解亚马逊 Alexa

+   构建一个由 IoT.js 控制的水泵

+   了解 AWS lambda

+   为亚马逊 Alexa 开发技能

+   测试气象站以及水泵

# 语音人工智能

曾经有一段时间，用智能手机打开/关闭某物是令人兴奋的。时代已经改变，语音人工智能的发展已经有了很大的进步。很多人用他们的声音做很多事情，从做笔记、建立购物清单到搜索互联网。我们不再用手做琐碎的活动。

“看吧，不用手！”

接下来呢？想到了就会发生吗？我很想活着看到那一天，因为我可以以思维的速度做事情。

如果你是语音人工智能的新手，你可以开始查找亚马逊 Alexa、Google Now/Google Assistant、Apple Siri 或 Windows Cortana，看看我在说什么。由于我们将在本章中使用亚马逊 Alexa，我们只会探索它。

亚马逊最近推出了两款名为亚马逊 Echo 和亚马逊 Echo Dot（最近也在印度上市），它们是由 Alexa，亚马逊的语音人工智能软件驱动的智能音箱。如果你想亲自体验 Alexa，而又不想购买 Echo 产品，可以在 Android 上下载 reverb 应用：[`play.google.com/store/apps/details?id=agency.rain.android.alexa&hl=en`](https://play.google.com/store/apps/details?id=agency.rain.android.alexa&hl=en) 或者在 iOS 上下载：[`itunes.apple.com/us/app/reverb-for-amazon-alexa/id1144695621?mt=8`](https://itunes.apple.com/us/app/reverb-for-amazon-alexa/id1144695621?mt=8)，然后启动该应用。

你应该看到一个带有麦克风图标的界面。按住麦克风，你应该在顶部看到“正在听...”的文字，就像下面的截图所示：

![](img/00065.jpeg)

现在说，*“Alexa，给我讲个笑话”*，然后被 Alexa 娱乐吧！

# 试驾

为了测试我们将要构建的东西，在 reverb 应用中按下麦克风图标，然后说 *“Alexa，请问 smarty app 天气报告”*，你应该听到保存在智能气象站数据库中的最新数据。然后你可以说 *“Alexa，请问 smarty app 打开发动机”*，或者 *“Alexa，请问 smarty app 关闭发动机”*；如果我的设备在线，它会关闭它。

除了智能气象站，我们还将建立一个智能插座，可以连接到农场中的发动机。然后使用 Alexa，我们将打开/关闭发动机。

现在，如果你有亚马逊 Echo 或 Echo Dot，你可以测试我们将要构建的技能。或者，你也可以使用 reverb 应用来做同样的事情。你也可以使用 [`reverb.ai/`](https://reverb.ai/) 或 [`echosim.io/`](https://echosim.io/) 来做同样的事情。

在你的 Alexa 技能发布之前，它只能在与你的亚马逊账户关联的设备上访问。如果你启用了测试版，那么你可以允许多人在他们的亚马逊账户关联的 Alexa 设备上访问这个技能。

如果你在探索演示时遇到问题，请查看这个视频录制：`/videos/chapter5/alexa_smarty_app_demo.mov`

那么，让我们开始吧！

# 构建智能插座

在本节中，我们将构建一个智能插座。设置将与第四章中的设置非常相似。创建一个名为`chapter5`的新文件夹，并将`chapter4`文件夹的内容复制到其中。`chapter4`文件夹中包含智能气象站的代码，现在，我们将添加智能插座所需的代码。

智能插座是一个可以通过互联网控制的简单电源插座。也就是说，打开插座和关闭插座。我们将使用**机械继电器**来实现这一点。

我们将从在树莓派上设置继电器开始。我将使用一个树莓派来演示智能气象站以及智能插座。您也可以使用两个树莓派来进行演示。

我们将向 API 引擎添加适当的 MQTT 客户端代码；接下来，更新 Web、桌面和移动应用程序，以添加一个切换开关来打开/关闭继电器。

我们将在`socket`上创建一个名为`socket`的新主题，我们将发送`1`或`0`来打开/关闭继电器，从而打开/关闭继电器另一端的负载。

请记住，我们正在探索可以使用物联网构建的各种解决方案，而不是构建最终产品本身。

# 使用树莓派设置继电器

目前，树莓派已连接智能气象站传感器。现在，我们将向设置添加一个继电器。

继电器是由电子信号驱动的电气开关。也就是说，用逻辑高`1`触发继电器会打开继电器，逻辑低`0`会关闭继电器。

一些继电器的工作方式相反，这取决于组件。要了解更多关于继电器类型和工作原理的信息，请参考[`www.phidgets.com/docs/Mechanical_Relay_Primer`](https://www.phidgets.com/docs/Mechanical_Relay_Primer)。

您可以从亚马逊购买一个简单的 5V 继电器：([`www.amazon.com/DAOKI%C2%AE-Arduino-Indicator-Channel-Official/dp/B00XT0OSUQ/ref=sr_1_3`](https://www.amazon.com/DAOKI%C2%AE-Arduino-Indicator-Channel-Official/dp/B00XT0OSUQ/ref=sr_1_3))。

继电器处理交流电流，在我们的示例中，我们不会将任何交流电源连接到继电器。我们将使用来自树莓派的 5V 直流电源来供电，并使用继电器上的 LED 指示灯来识别继电器是否已打开或关闭。如果您想将其连接到实际电源，请在这样做之前采取适当的预防措施。如果不注意，结果可能会令人震惊。

除了气象站，我们还将把继电器连接到树莓派 3\. 将继电器连接如下图所示。

树莓派与智能气象站的连接：

![](img/00066.jpeg)

树莓派与`继电器`（模块）的连接：

![](img/00067.jpeg)如果您购买了独立的继电器，您需要按照之前显示的电路进行设置。如果您购买了继电器模块，您需要在给继电器供电后，将引脚 18/GPIO24 连接到触发引脚。

为了重申之前的连接，请参见下表所示的表格：

+   树莓派和 MCP3208：

| **树莓派编号 - 引脚名称** | **MCP 3208 引脚编号 - 引脚名称** |
| --- | --- |
| 1 - 3.3V | 16 - VDD |
| 1 - 3.3V | 15 - AREF |
| 6 - GND | 14 - AGND |
| 23 - GPIO11, SPI0_SCLK | 13 - CLK |
| 21 - GPIO09, SPI0_MISO | 12 - DOUT |
| 19 - GPIO10, SPI0_MOSI | 11 - DIN |
| 24 - GPIO08, CEO | 10 - CS |
| 6 - GND | 9 - DGND |

+   湿度传感器和 MCP3208：

| **MCP 3208 引脚编号 - 引脚名称** | **传感器引脚** |
| --- | --- |
| 1 - A0 | 雨传感器 - A0 |
| 1 - A1 | 湿度传感器 - A0 |

+   树莓派和 DHT11：

| **树莓派编号 - 引脚名称** | **传感器引脚** |
| --- | --- |
| 3 - GPIO2 | DHT11 - 数据 |

+   树莓派和继电器：

| **树莓派编号 - 引脚名称** | **传感器引脚** |
| --- | --- |
| 12 - GPIO18 | 继电器 - 触发引脚 |

所有地线和所有 3.3V 引脚都连接到一个公共点。继电器所需的只是来自树莓派的 5V 电源，即引脚 2。

一旦我们按照之前所示连接了传感器，我们将编写所需的代码来与传感器进行接口。

前往`Raspberry Pi 3`内的`pi-client`文件夹，打开`pi-client/index.js`，并进行如下更新：

```js
var config = require('./config.js'); 
var mqtt = require('mqtt'); 
var GetMac = require('getmac'); 
var async = require('async'); 
var rpiDhtSensor = require('rpi-dht-sensor'); 
var McpAdc = require('mcp-adc'); 
var adc = new McpAdc.Mcp3208(); 
var rpio = require('rpio'); 

// Set pin 12 as output pin and to low 
rpio.open(12, rpio.OUTPUT, rpio.LOW); 

var dht11 = new rpiDhtSensor.DHT11(2); 
var temp = 0, 
    prevTemp = 0; 
var humd = 0, 
    prevHumd = 0; 
var macAddress; 
var state = 0; 

var mositureVal = 0, 
    prevMositureVal = 0; 
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
    client.subscribe('socket'); 
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
    } else if (topic === 'socket') { 
        state = parseInt(message) 
        console.log('Turning Relay', !state ? 'On' : 'Off'); 
        // Relays are almost always active low 
        //console.log(!state ? rpio.HIGH : rpio.LOW); 
        // If we get a 1 we turn on the relay, else off 
        rpio.write(12, !state ? rpio.HIGH : rpio.LOW); 
    } else { 
        console.log('Unknown topic', topic); 
    } 
}); 

// infinite loop, with 3 seconds delay 
setInterval(function() { 
    readSensorValues(function(results) { 
        console.log('Temperature: ' + temp + 'C, ' + 'humidity: ' + humd + '%, ' + ' Rain level (%):' + rainVal + ', ' + 'mositureVal (%): ' + mositureVal); 
        // if the temperature and humidity values change 
        // then only publish the values 
        if (temp !== prevTemp || humd !== prevHumd || mositureVal !== prevMositureVal || rainVal != prevRainVal) { 
            var data2Send = { 
                data: { 
                    t: temp, 
                    h: humd, 
                    r: rainVal, 
                    m: mositureVal, 
                    s: state 
                }, 
                macAddress: macAddress 
            }; 
            // console.log('Data Published'); 
            client.publish('weather-status', JSON.stringify(data2Send)); 
            // reset prev values to current 
            // for next loop 
            prevTemp = temp; 
            prevHumd = humd; 
            prevMositureVal = mositureVal; 
            prevRainVal = rainVal; 
        } 
    }); 
}, 3000); // every three second 

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
            // we are going to connect mositure sensor 
            // on channel 1, hence 1 is the first arg below 
            adc.readRawValue(1, function(value) { 
                // update global variable 
                mositureVal = value; 
                mositureVal = (100 - parseFloat((mositureVal / 4096) * 100)).toFixed(2); 
                callback(null, { moisture: mositureVal }); 
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

对于`Weather Station`代码，我们已经添加了`rpio`模块，并使用`rpio.open()`，我们已经将引脚 12 设置为输出引脚。我们还在名为 socket 的主题上进行监听。当我们从代理在此主题上收到响应时，我们根据数据将引脚 12 设置为高电平或低电平。

现在，我们将在树莓派`pi-client`文件夹内安装`rpio`模块，并运行以下命令：

```js
npm install rpio -save  
```

保存所有文件。现在，我们将从我们的桌面/机器上启动 Mosca 代理：

```js
mosca -c index.js -v | pino  
```

一旦您启动了 Mosca 服务器，请检查 Mosca 正在运行的服务器的 IP 地址。在树莓派`config.js`文件中更新相同的 IP，否则树莓派无法将数据发布到代理。

一旦 Mosca 成功启动并且我们已经验证了树莓派上的 IP 地址，请运行：

```js
sudo node index.js 
```

这将启动服务器并继续向代理发送天气信息。

在下一节中，我们将编写 API 引擎处理继电器所需的逻辑。

# 在 API 引擎中管理继电器

现在继电器已连接到树莓派，我们将编写逻辑，将打开/关闭命令发送到 socket 主题。打开`api-engine/server/mqtt/index.js`并进行如下更新：

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
        var macAddress = message.toString(); 
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

exports.sendSocketData = function(data) { 
    console.log('Sending Data', data); 
    client.publish('socket', JSON.stringify(data)); 
} 
```

我们添加了一个名为`sendSocketData`的方法并导出它。我们将在`api-engine/server/api/data/data.controller.jscreate`方法中调用此方法，如下所示：

```js
exports.create = function(req, res, next) { 
    var data = req.body; 
    data.createdBy = req.user._id; 
    Data.create(data, function(err, _data) { 
        if (err) return res.status(500).send(err); 
        if (data.topic === 'socket') { 
            require('../../mqtt/index.js').sendSocketData(_data.data.s); // send relay value 
        } 
        return res.json(_data); 
    }); 
}; 
```

保存所有文件并运行：

```js
npm start  
```

您应该在屏幕上看到以下内容：

![](img/00068.jpeg)

请注意，控制台中打印的数据字符串中的最后一个值; `s`，我们还发送继电器的状态以在 UI 中显示，如果继电器打开/关闭。

有了这个，我们就完成了开发 API 引擎所需的代码。在下一节中，我们将继续处理 Web 应用程序。

# 更新 Web 应用程序模板

在本节中，我们将更新 Web 应用程序模板，以便拥有一个切换按钮，与我们在第二章中所拥有的非常相似，*IoTFW.js - I*，以及第三章，*IoTFW.js - II*。使用切换按钮，我们将手动打开/关闭继电器。在后面的部分中，我们将对其进行自动化。

打开`web-app/src/app/device/device.component.html`并进行如下更新：

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
                            <td>Toggle Socket</td>
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

我们所做的只是添加了一个显示切换按钮的新行，通过使用它，我们可以打开/关闭插座。接下来，打开`web-app/src/app/device/device.component.ts`并进行如下更新，以管理切换按钮所需的逻辑：

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
               }] 
         }, 
         title: { 
               display: true, 
               text: 'Sensor Data vs. Time' 
         } 
   }; 
   public lineChartLegend: boolean = true; 
   public lineChartType: string = 'line'; 
   public tempHumdData: Array<any> = []; 
   public rainMoisData: Array<any> = []; 
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
               this.lastRecord = this.data[0]; // descending order data 
               this.toggleState = this.lastRecord.data.s; 
               this.genChart(); 
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

   toggleChange(state) { 
         let data = { 
               macAddress: this.device.macAddress, 
               data: { 
                     t: this.lastRecord.data.t, 
                     h: this.lastRecord.data.h, 
                     m: this.lastRecord.data.m, 
                     r: this.lastRecord.data.r, 
                     s: state ? 1 : 0 
               }, 
               topic: 'socket' 
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

   ngOnDestroy() { 
         this.subDevice.unsubscribe(); 
         this.subData ? this.subData.unsubscribe() : ''; 
   } 

   genChart() { 
         let data = this.data; 
         let _thArr: Array<any> = []; 
         let _rmArr: Array<any> = []; 
         let _lblArr: Array<any> = []; 

         let tmpArr: Array<any> = []; 
         let humArr: Array<any> = []; 
         let raiArr: Array<any> = []; 
         let moiArr: Array<any> = []; 

         for (var i = 0; i < data.length; i++) { 
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

   private formatDate(originalTime) { 
         var d = new Date(originalTime); 
         var datestring = d.getDate() + "-" + (d.getMonth() + 1) + "-" + d.getFullYear() + " " + 
               d.getHours() + ":" + d.getMinutes(); 
         return datestring; 
   } 

} 
```

我们在这里所做的一切就是管理切换按钮的状态。保存所有文件并运行以下命令：

```js
ng serve
```

导航到`http://localhost:4200`，然后导航到设备页面。现在，通过页面上的切换按钮，我们可以打开/关闭继电器，如下面的截图所示：

![](img/00069.jpeg)

如果一切设置正确，您应该看到继电器上的 LED 在继电器上打开/关闭，如下照片所示：

![](img/00070.jpeg)

电线！嘿！

有了这个，我们就完成了 Web 应用程序。在下一节中，我们将构建相同的 Web 应用程序并将其部署到我们的桌面应用程序中。

# 更新桌面应用程序

现在 Web 应用程序已完成，我们将构建相同的 Web 应用程序并将其部署到我们的桌面应用程序中。

要开始，请返回到`web-app`文件夹的终端/提示符，并运行：

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

我们编写的所有代码最终都打包到了前面的文件中。我们将获取`dist`文件夹中的所有文件（而不是`dist`文件夹），然后将其粘贴到`desktop-app/app`文件夹中。在之前的更改后，`desktop-app`的最终结构将如下所示：

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

然后，当我们导航到“查看设备”页面时，我们应该看到以下内容：

![](img/00071.jpeg)

使用切换按钮，我们应该能够打开/关闭继电器。

通过这样，我们已经完成了桌面应用的开发。在下一节中，我们将更新移动应用。

# 更新移动应用模板

在上一节中，我们已经更新了桌面应用。在本节中，我们将使用切换开关组件更新移动应用模板。因此，使用此切换开关，我们可以打开/关闭智能插座。

首先，我们要更新“查看设备”模板。更新`mobile-app/src/pages/view-device/view-device.html`，如下所示：

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
                <ion-label>Toggle LED</ion-label>
                <ion-toggle [(ngModel)]="toggleState" (click)="toggleChange($event)"></ion-toggle>
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

接下来，我们将添加所需的逻辑来管理切换按钮。更新`mobile-app/src/pages/view-device/view-device.ts`，如下所示：

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
               if (this.lastRecord) { 
                     this.toggleState = this.lastRecord.data.s; 
               } 
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

   toggleChange(state) { 
         let data = { 
               macAddress: this.device.macAddress, 
               data: { 
                     t: this.lastRecord.data.t, 
                     h: this.lastRecord.data.h, 
                     m: this.lastRecord.data.m, 
                     r: this.lastRecord.data.r, 
                     s: !state 
               }, 
               topic: 'socket' 
         } 

         console.log(data); 

         this.dataService.create(data).subscribe((resp) => { 
               if (resp.json()._id) { 
                     this.toastService.toggleToast('Device Notified!'); 
               } 
         }, (err) => { 
               console.log(err); 
               this.toastService.toggleToast('Device Notification Failed. Check console for the error!'); 
         }) 
   } 

   ionViewDidUnload() { 
         this.subData && this.subData.unsubscribe && this.subData.unsubscribe(); //unsubscribe if subData is defined 
   } 
} 
```

在这里，我们已经添加了所需的逻辑来管理切换按钮。保存所有文件并运行：

```js
ionic serve 
```

或者，您也可以将其部署到您的设备上，方法是运行：

```js
ionic run android  
```

或者：

```js
ionic run ios  
```

应用程序启动后，当我们导航到“查看设备”页面时，我们应该看到以下内容：

![](img/00072.jpeg)

我们应该能够使用移动应用上的切换按钮来控制插座。

通过这样，我们已经完成了智能电机的设置。

在下一节中，我们将为亚马逊 Alexa 构建一个新的技能。

# 开发 Alexa 技能

在上一节中，我们已经看到了如何构建智能插座并将其与我们现有的智能气象站集成。在本节中，我们将为与亚马逊 Alexa 接口的智能设备构建一个新的技能。

我们将创建一个名为 smarty app 的新技能，然后向其添加两个语音模型：

+   获取最新的天气状态

+   打开/关闭插座

如果您是 Alexa 及其技能开发的新手，我建议您在继续之前观看以下系列视频：开发 Alexa 技能：[`www.youtube.com/playlist?list=PL2KJmkHeYQTO6ci5KF08mvHYdAZu2jgkJ`](https://www.youtube.com/playlist?list=PL2KJmkHeYQTO6ci5KF08mvHYdAZu2jgkJ)

为了快速概述我们的技能创建，我们将按照以下步骤进行：

1.  登录到亚马逊开发者门户并创建和设置一个新技能

1.  训练语音模型

1.  在 AWS Lambda 服务中编写所需的业务逻辑

1.  部署和测试设置

那么，让我们开始吧。

# 创建技能

我们要做的第一件事是登录到[`developer.amazon.com`](https://developer.amazon.com)。一旦登录，点击页面顶部的 Alexa。您应该会登陆到一个页面，应该如下所示：

![](img/00073.jpeg)

点击“开始”>下面的 Alexa 技能套件，您应该被重定向到一个页面，您可以查看您现有的技能集或创建一个新的。点击右上角的金色按钮，名为“添加新技能”。

您应该被重定向到一个页面，如下所示：

![](img/00074.jpeg)

我已经提供了前面的信息。您可以根据需要进行配置。点击保存，然后点击左侧菜单上的“交互模型”，您应该被重定向到交互模型设置，如下所示：

![](img/00075.jpeg)

我们将使用技能构建器，这在撰写时仍处于测试阶段。技能构建器是一个简单的界面，用于训练我们的语音模型。

点击“启动技能构建器”按钮。

# 训练语音模型

一旦我们进入技能构建器，我们将开始训练模型。在我们的应用程序中，我们将有两个意图：

+   `WeatherStatusIntent`：获取所有四个传感器的值

+   `ControlMotorIntent`：打开/关闭电机

除此之外，你还可以根据自己的需求添加其他意图。你可以添加一个仅湿度传感器意图，以仅获取湿度传感器的值，或者添加一个仅雨传感器意图，以仅获取雨传感器的值。

现在，我们将继续设置这些意图并创建槽。

一旦你进入了技能构建器，你应该看到类似以下的东西：

![](img/00076.jpeg)

现在，在左侧的意图旁边使用 Add +，创建一个新的自定义意图并命名为`WeatherStatusIntent`，如下所示：

![](img/00077.jpeg)

现在，我们要训练语音模型。一旦意图被创建，点击左侧菜单上的意图名。现在，我们应该看到一个名为示例话语的部分。我们要提供用户如何调用我们服务的示例话语。

为了保持简单，我只添加了三个示例：

Alexa，问 smarty app：

+   天气报告

+   天气状况

+   字段条件

你可以在以下截图中看到这一点：

![](img/00078.jpeg)

接下来，我们将使用相同的过程创建另一个名为`ControlMotorIntent`的意图。点击左侧菜单上的 ControlMotorIntent，我们应该看到示例话语部分。

对于这个意图，我们要做一些不同的事情；我们要创建一些叫做**槽**的东西。我们要取用户可能说出的示例话语，并提取其中的一部分作为变量。

例如，如果用户说*Alexa，问 smarty app 打开电机*或*Alexa，问 smarty app 关闭电机*，除了打开或关闭之外，一切都是相同的，所以我们想将这些转换为变量，并分别处理每个指令。

如果槽被打开，我们就打开电机，如果槽被关闭，我们就关闭电机。

因此，一旦你输入了示例话语，比如打开电机，选择文本`打开`，如下截图所示：

![](img/00079.jpeg)

一旦你选择了文本，输入一个自定义意图槽名 motorAction 并点击*加号*图标。

对于这个意图，我们只会有一个话语。接下来，我们需要配置 motorAction 意图槽。

在页面的右侧，你应该看到新创建的意图槽。勾选 REQ 列下的复选框。这意味着这个值是意图调用所必需的。接下来，点击槽名下面的选择槽类型。

在这里，我们需要定义一个自定义意图槽类型。添加`motorActionIntentSlot`，如下所示：

![](img/00080.jpeg)

接下来，我们需要设置值。从左侧菜单中点击`motorActionIntentSlot`，然后添加两个值；turn on 和 turn off，如下所示：

![](img/00081.jpeg)

完成后，我们需要设置当用户没有说出我们定义的两个槽值时将会说的提示。点击 ControlMotorIntent 下的{motorAction}和对话模型下方，输入提示，比如`你想让我打开还是关闭电机？`，如下所示：

![](img/00082.jpeg)

通过这样，我们已经定义了我们的语音模型。

现在，我们需要要求 Alexa 技能引擎构建我们的语音模型，并将其添加到其技能引擎中。使用页面顶部的保存模型按钮保存模型，然后构建模型：

![](img/00083.jpeg)

通常构建过程只需要五分钟或更短的时间。

# ngrok API 引擎

在继续进行 lambda 服务之前，我们需要首先将 API 引擎暴露出来，以便通过公共 URL 可用，比如[`iotfwjs.com/api`](http://iotjs.com/api)，这样当用户询问 Alexa 技能服务问题或发布命令时，Alexa 技能服务可以通过 lambda 服务联系我们。

到目前为止，我们一直在使用基于本地 IP 的配置来与 API 引擎、代理、Web 应用程序或树莓派进行交互。但是，当我们希望 Alexa 技能服务找到我们时，这种方法就行不通了。

因此，我们将使用一个名为`ngrok`的服务（[`ngrok.com/`](https://ngrok.com/)）来临时托管我们的本地代码，并提供一个公共 URL，Amazon Alexa 服务可以通过 lambda 服务找到我们。

要设置`ngrok`，请按照以下步骤进行：

1.  从这里下载`ngrok`安装程序：[`ngrok.com/download`](https://ngrok.com/download)适用于运行 API 引擎的操作系统

1.  解压并复制`ngrok`下载的 ZIP 文件的内容到`api-engine`文件夹的根目录

1.  通过运行以下命令从`broker`文件夹的根目录启动 Mosca：

```js
mosca -c index.js -v | pino  
```

1.  通过运行以下命令从`api-engine`文件夹的根目录启动 API 引擎：

```js
npm start  
```

1.  现在开始使用`ngrok`进行隧道。从我们已经复制了`ngrok`可执行文件的`api-engine`文件夹的根目录运行：

```js
./ngrok http 9000  
```

运行`./ngrok http 9000`将在本地主机和`ngrok`服务器的公共实例之间启动一个新的隧道，我们应该看到以下内容：

![](img/00084.jpeg)

转发 URL 每次杀死和重新启动`ngrok`时都会更改。在前面的情况下，ngrok 的公共 URL：`http://add7231d.ngrok.io`映射到我的本地服务器：`http://localhost:9000`。这不是很容易吗？

要快速测试公共 URL，请打开`web-app/src/app/app.global.ts`并更新如下：

```js
export const Globals = Object.freeze({ 
   // BASE_API_URL: 'http://localhost:9000/', 
   BASE_API_URL: 'https://add7231d.ngrok.io/', 
   API_AUTH_TOKEN: 'AUTH_TOKEN', 
   AUTH_USER: 'AUTH_USER' 
}); 
```

现在，您可以从任何地方启动您的 web 应用程序，并且它将使用公共 URL 与 API 引擎进行通信。

在继续之前，请阅读`ngrok`的服务条款（[`ngrok.com/tos`](https://ngrok.com/tos)）和隐私政策（[`ngrok.com/privacy`](https://ngrok.com/privacy)）。

# 定义 lambda 函数

现在语音模型已经训练好，并且我们有一个可以访问 API 引擎的公共 URL，我们将编写所需的服务来响应用户的交互。

当用户说“Alexa，请问 smarty app 天气报告”时，Alexa 将向 AWS lambda 函数发出请求，lambda 函数将调用 API 引擎进行适当的活动。

引用 AWS：[`aws.amazon.com/lambda/details/`](https://aws.amazon.com/lambda/details/)

AWS Lambda 是一种无服务器计算服务，它根据事件运行您的代码，并自动管理底层计算资源。您可以使用 AWS Lambda 来扩展其他 AWS 服务的自定义逻辑，或者创建自己的后端服务，以在 AWS 规模、性能和安全性上运行。

要了解有关 AWS lambda 的更多信息，请参阅：[`aws.amazon.com/lambda/details/`](https://aws.amazon.com/lambda/details/)。

要开始，请转到 AWS 控制台：[`console.aws.amazon.com/`](https://console.aws.amazon.com/)并选择北弗吉尼亚地区。截至今天，只允许在北美和欧洲托管的 AWS lambda 服务与 Alexa 技能进行关联。

接下来，从顶部的服务菜单中，在计算部分下选择 Lambda。这将带我们到 lambda 服务的函数屏幕。单击创建 Lambda 函数，然后我们将被要求选择一个蓝图。选择空白函数。接下来，您将被要求选择一个触发器；选择 Alexa 技能集，如下：

![](img/00085.jpeg)

点击下一步。现在，我们需要配置函数。更新如下：

![](img/00086.jpeg)

对于 Lambda 函数代码，请输入以下代码：

```js
'use strict'; 

// Route the incoming request based on type (LaunchRequest, IntentRequest, 
// etc.) The JSON body of the request is provided in the event parameter. 
exports.handler = function(event, context) { 
    try { 
        console.log("event.session.application.applicationId=" + event.session.application.applicationId); 

        if (event.session.new) { 
            onSessionStarted({ requestId: event.request.requestId }, event.session); 
        } 

        if (event.request.type === "LaunchRequest") { 
            onLaunch(event.request, 
                event.session, 
                function callback(sessionAttributes, speechletResponse) { 
                    context.succeed(buildResponse(sessionAttributes, speechletResponse)); 
                }); 
        } else if (event.request.type === "IntentRequest") { 
            onIntent(event.request, 
                event.session, 
                function callback(sessionAttributes, speechletResponse) { 
                    context.succeed(buildResponse(sessionAttributes, speechletResponse)); 
                }); 
        } else if (event.request.type === "SessionEndedRequest") { 
            onSessionEnded(event.request, event.session); 
            context.succeed(); 
        } 
    } catch (e) { 
        context.fail("Exception: " + e); 
    } 
}; 

/** 
 * Called when the session starts. 
 */ 
function onSessionStarted(sessionStartedRequest, session) { 
    console.log("onSessionStarted requestId=" + sessionStartedRequest.requestId + ", sessionId=" + session.sessionId); 

    // add any session init logic here 
} 

/** 
 * Called when the user invokes the skill without specifying what they want. 
 */ 
function onLaunch(launchRequest, session, callback) { 
    console.log("onLaunch requestId=" + launchRequest.requestId + ", sessionId=" + session.sessionId); 

    var cardTitle = "Smarty App" 
    var speechOutput = "Hello, What would you like to know about your farm today?" 
    callback(session.attributes, 
        buildSpeechletResponse(cardTitle, speechOutput, "", true)); 
} 

/** 
 * Called when the user specifies an intent for this skill. 
 */ 
function onIntent(intentRequest, session, callback) { 
    console.log("onIntent requestId=" + intentRequest.requestId + ", sessionId=" + session.sessionId); 

    var intent = intentRequest.intent, 
        intentName = intentRequest.intent.name; 

    // dispatch custom intents to handlers here 
    if (intentName == 'WeatherStatusIntent') { 
        handleWSIRequest(intent, session, callback); 
    } else if (intentName == 'ControlMotorIntent') { 
        handleCMIRequest(intent, session, callback); 
    } else { 
        throw "Invalid intent"; 
    } 
} 

/** 
 * Called when the user ends the session. 
 * Is not called when the skill returns shouldEndSession=true. 
 */ 
function onSessionEnded(sessionEndedRequest, session) { 
    console.log("onSessionEnded requestId=" + sessionEndedRequest.requestId + ", sessionId=" + session.sessionId); 

    // Add any cleanup logic here 
} 

function handleWSIRequest(intent, session, callback) { 
    getData(function(speechOutput) { 
        callback(session.attributes, 
            buildSpeechletResponseWithoutCard(speechOutput, "", "true")); 
    }); 
} 

function handleCMIRequest(intent, session, callback) { 
    var speechOutput = 'Got '; 
    var status; 
    var motorAction = intent.slots.motorAction.value; 
    speechOutput += motorAction; 
    if (motorAction === 'turn on') { 
        status = 1; 
    } 

    if (motorAction === 'turn off') { 
        status = 0; 
    } 
    setData(status, function(speechOutput) { 
        callback(session.attributes, 
            buildSpeechletResponseWithoutCard(speechOutput, "", "true")); 
    }); 

} 

function getData(cb) { 
    var http = require('http'); 
    var chunk = ''; 
    var options = { 
        host: '31d664cf.ngrok.io', 
        port: 80, 
        path: '/api/v1/data/b8:27:eb:39:92:0d/30', 
        agent: false, 
        timeout: 10000, 
        method: 'GET', 
        headers: { 
            'AlexSkillRequest': true, 
            'authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI1OTFmZGI5ZGNlYjBiODM2YjIzMmI3MjMiLCJpYXQiOjE0OTcxNjE4MTUsImV4cCI6MTQ5NzI0ODIxNX0.ua-SXAqLb-XUEtbgY55TX_pKdD2Xj5OSM7b9Iox_Rd8' 
        } 
    }; 

    var req = http.request(options, function(res) { 
        res.on('data', function(_chunk) { 
            chunk += _chunk; 
        }); 

        res.on('end', function() { 
            var resp = chunk; 
            if (typeof chunk === 'string') { 
                resp = JSON.parse(chunk); 
            } 

            if (resp.length === 0) { 
                cb('Looks like we have not gathered any data yet! Please try again later!'); 
            } 

            var d = resp[0].data; 

            if (!d) { 
                cb('Looks like there is something wrong with the data we got! Please try again later!'); 
            } 

            var temp = d.t || 'invalid'; 
            var humd = d.h || 'invalid'; 
            var mois = d.m || 'invalid'; 
            var rain = d.r || 'invalid'; 

            cb('The temperature is ' + temp + ' degrees celsius, the humidity is ' + humd + ' percent, The moisture level is ' + mois + ' percent and the rain level is ' + rain + ' percent!'); 

        }); 

        res.on('error', function() { 
            console.log(arguments); 
            cb('Looks like something went wrong.'); 
        }); 
    }); 
    req.end(); 
} 

function setData(status, cb) { 
    var http = require('http'); 
    var chunk = ''; 
    var data = { 
        'status': status, 
        'macAddress': 'b8:27:eb:39:92:0d' 
    }; 

    data = JSON.stringify(data); 

    var options = { 
        host: '31d664cf.ngrok.io', 
        port: 80, 
        path: '/api/v1/data', 
        agent: false, 
        timeout: 10000, 
        method: 'POST', 
        headers: { 
            'AlexSkillRequest': true, 
            'Content-Type': 'application/json', 
            'Content-Length': Buffer.byteLength(data), 
            'authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI1OTFmZGI5ZGNlYjBiODM2YjIzMmI3MjMiLCJpYXQiOjE0OTcxNjE4MTUsImV4cCI6MTQ5NzI0ODIxNX0.ua-SXAqLb-XUEtbgY55TX_pKdD2Xj5OSM7b9Iox_Rd8' 
        } 
    }; 

    var req = http.request(options, function(res) { 
        res.on('data', function(_chunk) { 
            chunk += _chunk; 
        }); 

        res.on('end', function() { 
            var resp = chunk; 
            if (typeof chunk === 'string') { 
                resp = JSON.parse(chunk); 
            } 

            cb('Motor has been successfully ' + (status ? 'turned on' : 'turned off')); 

        }); 

        res.on('error', function() { 
            console.log(arguments); 
            cb('Looks like something went wrong.'); 
        }); 
    }); 

    // post the data 
    req.write(data); 
    req.end(); 
} 

// ------- Helper functions to build responses ------- 

function buildSpeechletResponse(title, output, repromptText, shouldEndSession) { 
    return { 
        outputSpeech: { 
            type: "PlainText", 
            text: output 
        }, 
        card: { 
            type: "Simple", 
            title: title, 
            content: output 
        }, 
        reprompt: { 
            outputSpeech: { 
                type: "PlainText", 
                text: repromptText 
            } 
        }, 
        shouldEndSession: shouldEndSession 
    }; 
} 

function buildSpeechletResponseWithoutCard(output, repromptText, shouldEndSession) { 
    return { 
        outputSpeech: { 
            type: "PlainText", 
            text: output 
        }, 
        reprompt: { 
            outputSpeech: { 
                type: "PlainText", 
                text: repromptText 
            } 
        }, 
        shouldEndSession: shouldEndSession 
    }; 
} 

function buildResponse(sessionAttributes, speechletResponse) { 
    return { 
        version: "1.0", 
        sessionAttributes: sessionAttributes, 
        response: speechletResponse 
    }; 
} 
```

代码中有很多内容。`exports.handler()`是我们需要为 lambda 设置的默认函数。在其中，我们定义了传入请求的类型。如果传入的是`IntentRequest`，我们调用`onIntent()`。在`onIntent()`中，我们获取`intentName`并调用适当的逻辑。

如果`intentName`是`WeatherStatusIntent`，我们调用`handleWSIRequest()`，否则如果 intentName 是`ControlMotorIntent`，我们调用`handleCMIRequest()`。

在`handleWSIRequest()`内，我们调用`getData()`，它将向我们的`ngrok` URL 发出 HTTP `GET`请求。一旦数据到达，我们构造一个响应并将其返回给技能服务。

而且，`handleCMIRequest()`也是一样，只是它首先获取`motorAction`槽值，然后调用`setData()`，这将调用或者打开/关闭电机。

一旦代码被复制，您应该在底部找到额外的配置。我们将保持处理程序不变。对于角色，点击创建自定义角色，并进行如下设置：

![](img/00087.jpeg)

然后点击允许。这将创建一个新的角色，该角色将在*现有角色*中填充，如下所示：

![](img/00088.jpeg)

完成后，点击下一步。验证摘要，然后点击页面底部的创建函数。

如果一切顺利，您应该看到以下屏幕：

![](img/00089.jpeg)

请注意右上角的 ARN。这是我们 lambda 函数的**Amazon 资源名称**（**ARN**）。我们需要将其作为输入提供给 Alexa Skills Kit。

# 部署和测试

现在我们拥有了所有的部件，我们将在我们创建的 Alexa 技能中配置 ARN。返回 Alexa 技能，点击配置，然后按照以下方式更新配置：

![](img/00090.jpeg)

点击下一步。如果一切设置正确，我们可以测试设置。

在测试页面的底部，我们应该看到一个名为`服务仿真器`的部分。您可以按照以下方式进行测试：

![](img/00091.jpeg)

以下截图显示了 lambda 从 Alexa 接收到的请求：

![](img/00092.jpeg)

通过这样，我们已经完成了将 Alexa 与我们的 IoT.js 框架集成。

# 总结

在本章中，我们探讨了如何将 Alexa 等语音 AI 服务与我们开发的 IoTFW.js 框架集成。我们继续使用第四章，*智能农业*中的相同示例，并通过设置可以打开/关闭电机的继电器开始了本章。接下来，我们了解了 Alexa 的工作原理。我们创建了一个新的自定义技能，然后设置了所需的语音模型。之后，我们在 AWS lambda 中编写了所需的业务逻辑，该逻辑将获取最新的天气状况并控制电机。

我们最终使用 reverb 应用程序测试了一切，并且也验证了一切。

在第六章，*智能可穿戴*中，我们将研究物联网和医疗保健。
