# 第七章：智能可穿戴和 IFTTT

在第六章 *智能可穿戴*中，我们看到了如何构建一个简单的可穿戴设备，显示用户的位置并读取加速计值。在本章中，我们将通过在设备上实现跌倒检测逻辑，然后在数据上添加**If This Then That**（**IFTTT**）规则，将该应用程序提升到下一个级别。我们将讨论以下主题：

+   什么是 IFTTT

+   IFTTT 和物联网

+   了解跌倒检测

+   基于加速计的跌倒检测

+   构建一个 IFTTT 规则引擎

# IFTTT 和物联网

这种反应模式可以轻松应用于某些情况。例如，如果病人摔倒，就叫救护车，或者如果温度低于 15 度，就关闭空调，等等。这些都是我们定义的简单规则，可以帮助我们自动化许多流程。

在物联网中，规则引擎是自动化大部分单调任务的关键。在本章中，我们将构建一个简单的硬编码规则引擎，将持续监视传入的数据。如果传入的数据与我们的任何规则匹配，它将执行一个响应。

我们正在构建的东西类似于[ifttt.com](https://ifttt.com/)（[`ifttt.com/discover`](https://ifttt.com/discover)）的概念，但非常特定于我们框架内存在的物联网设备。IFTTT（[`ifttt.com/discover`](https://ifttt.com/discover)）与我们在书中构建的内容无关。

# 跌倒检测

在第六章 *智能可穿戴*中，我们从加速计中收集了三个轴的值。现在，我们将利用这些数据来检测跌倒。

我建议观看视频*自由落体中的加速计*（[`www.youtube.com/watch?v=-om0eTXsgnY`](https://www.youtube.com/watch?v=-om0eTXsgnY)），它解释了加速计在静止和运动时的行为。

现在我们了解了跌倒检测的基本概念，让我们谈谈我们的具体用例。

跌倒检测中最大的挑战是区分跌倒和其他活动，比如跑步和跳跃。在本章中，我们将保持简单，处理非常基本的条件，即用户静止或持续运动时突然摔倒。

为了确定用户是否摔倒，我们使用信号幅度矢量或*SMV*。*SMV*是三个轴的值的均方根。也就是说：

![](img/00100.jpeg)

如果我们开始绘制用户在站立不动然后摔倒时的**SMV**随**时间**的图表，我们将得到以下图表：

![](img/00101.jpeg)

请注意图表末端的尖峰。这是用户实际摔倒的点。

现在，当我们从 ADXL345 收集加速计值时，我们将计算 SMV。通过使用我们构建的智能可穿戴进行多次迭代，我一直能够在 1 g SMV 值处稳定地检测到跌倒。对于小于 1 g SMV 的任何值，用户几乎总是被认为是静止的，而大于 1 g SMV 的任何值都被认为是跌倒。

请注意，我已经将加速计放置在 y 轴垂直于地面的位置。

一旦我们把设置放在一起，您就可以亲自看到 SMV 值随加速计位置的变化而变化。

请注意，如果您正在进行其他活动，比如跳跃或下蹲，可能会触发跌倒检测。您可以调整 1 g SMV 的阈值，以获得一致的跌倒检测。

你也可以参考*使用 3 轴数字加速度计检测人类跌倒*：([`www.analog.com/en/analog-dialogue/articles/detecting-falls-3-axis-digital-accelerometer.html`](http://www.analog.com/en/analog-dialogue/articles/detecting-falls-3-axis-digital-accelerometer.html))，或者*基于加速度计的身体传感器定位用于健康和医疗监测应用* ([`www.ncbi.nlm.nih.gov/pmc/articles/PMC3279922/`](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3279922/))，以及*开发用于检测日常活动中跌倒的算法，使用 2 个三轴加速度计* ([`waset.org/publications/2993/development-of-the-algorithm-for-detecting-falls-during-daily-activity-using-2-tri-axial-accelerometers`](http://waset.org/publications/2993/development-of-the-algorithm-for-detecting-falls-during-daily-activity-using-2-tri-axial-accelerometers))，以便更好地理解这个主题并提高系统的效率。

# 更新树莓派

现在我们知道需要做什么，我们将开始编写代码。

在继续之前，创建一个名为`chapter7`的文件夹，并在其中复制`chapter6`代码。

接下来，打开`pi/index.js`文件。我们将更新 ADXL345 初始化设置，然后开始处理数值。更新`pi/index.js`如下：

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

var prevX, prevY, prevZ, prevSMV, prevFALL; 
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
    adxl345.init() 
        .then(() => adxl345.setMeasurementRange(ADXL345.RANGE_2_G())) 
        .then(() => adxl345.setDataRate(ADXL345.DATARATE_100_HZ())) 
        .then(() => adxl345.setOffsetX(0)) // measure for your particular device 
        .then(() => adxl345.setOffsetY(0)) // measure for your particular device 
        .then(() => adxl345.setOffsetZ(0)) // measure for your particular device 
        .then(() => adxl345.getMeasurementRange()) 
        .then((range) => { 
            console.log('Measurement range:', ADXL345.stringifyMeasurementRange(range)); 
            return adxl345.getDataRate(); 
        }) 
        .then((rate) => { 
            console.log('Data rate: ', ADXL345.stringifyDataRate(rate)); 
            return adxl345.getOffsets(); 
        }) 
        .then((offsets) => { 
            console.log('Offsets: ', JSON.stringify(offsets, null, 2)); 
            console.log('ADXL345 initialization succeeded'); 
            loop(); 
        }) 
        .catch((err) => console.error('ADXL345 initialization failed:', err)); 
} 

function loop() { 
    // infinite loop, with 3 seconds delay 
    setInterval(function() { 
        // wait till we get the location 
        // then start processing 
        if (!locationG) return; 

        readSensorValues(function(acclVals) { 
            var x = acclVals.x; 
            var y = acclVals.y; 
            var z = acclVals.z; 
            var fall = 0; 
            var smv = Math.sqrt(x * x, y * y, z * z); 

            if (smv > 1) { 
                fall = 1; 
            } 

            acclVals.smv = smv; 
            acclVals.fall = fall; 

            var data2Send = { 
                data: { 
                    acclVals: acclVals, 
                    location: locationG 
                }, 
                macAddress: macAddress 
            }; 

            // no duplicate data 
            if (fall === 1 && (x !== prevX || y !== prevY || z !== prevZ || smv !== prevSMV || fall !== prevFALL)) { 
                console.log('Fall Detected >> ', acclVals); 
                client.publish('accelerometer', JSON.stringify(data2Send)); 
                console.log('Data Published'); 
                prevX = x; 
                prevY = y; 
                prevZ = z; 
            } 
        }); 

        if (locCtr === 600) { // every 5 mins 
            locCtr = 0; 
            displayLocation(); 
        } 

        aclCtr++; 
        locCtr++; 
    }, 500); // every one second 
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

注意`initADXL345()`。我们将测量范围定义为`2_G`，清除偏移量，然后调用无限循环函数。在这种情况下，我们将`setInterval()`每`500`毫秒运行一次，而不是每`1`秒。`readSensorValues()`每`500`毫秒调用一次，而不是每`3`秒。这是为了确保我们能够及时捕捉到跌倒。

在`readSensorValues()`中，一旦`x`、`y`和`z`值可用，我们就计算 SMV。然后，我们检查 SMV 值是否大于`1`：如果是，那么我们就检测到了跌倒。

除了`x`、`y`和`z`值之外，我们还发送 SMV 值以及跌倒值到 API 引擎。还要注意，在这个例子中，我们并不是在收集所有值后立即发送数据。我们只有在检测到跌倒时才发送数据。

保存所有文件。通过从`chapter7/broker`文件夹运行以下命令来启动代理：

```js
mosca -c index.js -v | pino  
```

接下来，通过从`chapter7/api-engine`文件夹运行以下命令来启动 API 引擎：

```js
npm start  
```

我们还没有将 IFTTT 逻辑添加到 API 引擎中，这将在下一节中完成。目前，为了验证我们的设置，让我们通过执行在树莓派上的`index.js`文件来开始：

```js
npm start  
```

如果一切顺利，加速度计应该成功初始化，并且数据应该开始传入。

如果我们模拟自由落体，我们应该看到我们的第一条数据发送到 API 引擎，并且它应该看起来像以下截图：

![](img/00102.jpeg)

正如你所看到的，模拟的自由落体给出了`2.048` g 的 SMV。

我的硬件设置如下所示：

![](img/00103.jpeg)

我将整个设置粘贴到了**聚苯乙烯**板上，这样我就可以舒适地测试跌倒检测逻辑。

在我确定自由落体的 SMV 时，我从设置中移除了 16 x 2 LCD。

在下一节中，我们将读取从设备接收到的数据，然后根据数据执行规则。

# 构建 IFTTT 规则引擎

现在我们正在将所需的数据发送到 API 引擎，我们将做两件事：

1.  在网页、桌面和移动应用程序上显示我们从智能可穿戴设备得到的数据

1.  在数据之上执行规则

我们将首先开始第二个目标。我们将构建一个规则引擎来根据我们收到的数据执行规则。

让我们从在`api-engine/server`文件夹的根目录下创建一个名为`ifttt`的文件夹开始。在`ifttt`文件夹中，创建一个名为`rules.json`的文件。更新`api-engine/server/ifttt/rules.json`如下：

```js
[{ 
    "device": "b8:27:eb:39:92:0d", 
    "rules": [ 
    { 
        "if": 
        { 
            "prop": "fall", 
            "cond": "eq", 
            "valu": 1 
        }, 
        "then": 
        { 
            "action": "EMAIL", 
            "to": "arvind.ravulavaru@gmail.com" 
        } 
    }] 
}] 
```

从前面的代码中可以看出，我们正在维护一个包含所有规则的 JSON 文件。在我们的情况下，每个设备只有一个规则，规则有两部分：`if`部分和`then`部分。`if`指的是需要针对传入数据进行检查的属性，检查条件以及需要进行检查的值。`then`部分指的是如果`if`匹配，则需要执行的操作。在前面的情况下，此操作涉及发送电子邮件。

接下来，我们将构建规则引擎本身。在`api-engine/server/ifttt`文件夹内创建一个名为`ifttt.js`的文件，并更新`api-engine/server/ifttt/ifttt.js`，如下所示：

```js
var Rules = require('./rules.json'); 

exports.processData = function(data) { 

    for (var i = 0; i < Rules.length; i++) { 
        if (Rules[i].device === data.macAddress) { 
            // the rule belows to the incoming device's data 
            for (var j = 0; j < Rules[i].rules.length; j++) { 
                // process one rule at a time 
                var rule = Rules[i].rules[j]; 
                var data = data.data.acclVals; 
                if (checkRuleAndData(rule, data)) { 
                    console.log('Rule Matched', 'Processing Then.'); 
                    if (rule.then.action === 'EMAIL') { 
                        console.log('Sending email to', rule.then.to); 
                        EMAIL(rule.then.to); 
                    } else { 
                        console.log('Unknown Then! Please re-check the rules'); 
                    } 
                } else { 
                    console.log('Rule Did Not Matched', rule, data); 
                } 
            } 
        } 
    } 
} 

/*   Rule process Helper  */ 
function checkRuleAndData(rule, data) { 
    var rule = rule.if; 
    if (rule.cond === 'lt') { 
        return rule.valu < data[rule['prop']]; 
    } else if (rule.cond === 'lte') { 
        return rule.valu <= data[rule['prop']]; 
    } else if (rule.cond === 'eq') { 
        return rule.valu === data[rule['prop']]; 
    } else if (rule.cond === 'gte') { 
        return rule.valu >= data[rule['prop']]; 
    } else if (rule.cond === 'gt') { 
        return rule.valu > data[rule['prop']]; 
    } else if (rule.cond === 'ne') { 
        return rule.valu !== data[rule['prop']]; 
    } else { 
        return false; 
    } 
} 

/*Then Helpers*/ 
function SMS() { 
    /// AN EXAMPLE TO SHOW OTHER THENs 
} 

function CALL() { 
    /// AN EXAMPLE TO SHOW OTHER THENs 
} 

function PUSHNOTIFICATION() { 
    /// AN EXAMPLE TO SHOW OTHER THENs 
} 

function EMAIL(to) { 
    /// AN EXAMPLE TO SHOW OTHER THENs 
    var email = require('emailjs'); 
    var server = email.server.connect({ 
        user: 'arvind.ravulavaru@gmail.com', 
        password: 'XXXXXXXXXX', 
        host: 'smtp.gmail.com', 
        ssl: true 
    }); 

    server.send({ 
        text: 'Fall has been detected. Please attend to the patient', 
        from: 'Patient Bot <arvind.ravulavaru@gmail.com>', 
        to: to, 
        subject: 'Fall Alert!!' 
    }, function(err, message) { 
        if (err) { 
            console.log('Message sending failed!', err); 
        } 
    }); 
} 
```

逻辑非常简单。当新的数据记录到达 API 引擎时，将调用`processData()`。然后，我们从`rules.json`文件中加载所有规则，并对它们进行迭代，以检查当前规则是否适用于传入设备。

如果是，则通过传递规则和传入数据来调用`checkRuleAndData()`，以检查当前数据集是否与预定义规则匹配。如果是，我们将检查动作，我们的情况是发送电子邮件。您可以在代码中更新相应的电子邮件凭据。

完成后，我们需要在`api-engine/server/mqtt/index.js client.on('message')`中使用`topic`等于`accelerometer`来调用`processData()`。

更新`client.on('message')`，如下所示：

```js
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
        console.log('data >> ', data); 
        // create a new data record for the device 
        Data.create(data, function(err, data) { 
            if (err) return console.error(err); 
            // if the record has been saved successfully,  
            // websockets will trigger a message to the web-app 
            // console.log('Data Saved :', data.data); 
            // Invoke IFTTT Rules Engine 
            RulesEngine.processData(data); 
        }); 
    } else { 
        console.log('Unknown topic', topic); 
    } 
}); 
```

就是这样。我们已经准备好了 IFTTT 引擎运行所需的所有部件。

保存所有文件并重新启动 API 引擎。现在，模拟一次跌倒，我们应该看到一封电子邮件，内容应该类似于这样：

![](img/00104.jpeg)

现在我们已经完成了 IFTTT 引擎，我们将更新界面以反映我们收集到的新数据。

# 更新 Web 应用程序

要更新 Web 应用程序，请打开`web-app/src/app/device/device.component.html`并进行如下更新：

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
              <td>Signal Magnitude Vector</td> 
              <td>{{lastRecord.data.acclVals.smv}}</td> 
            </tr> 
            <tr *ngIf="lastRecord"> 
              <td>Fall State</td> 
              <td>{{lastRecord.data.acclVals.fall ? 'Patient Down' : 'All is well!'}}</td> 
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

保存文件并运行：

```js
npm start
```

一旦我们导航到设备页面，我们应该看到以下内容：

![](img/00105.jpeg)

在下一节中，我们将更新桌面应用程序。

# 更新桌面应用程序

现在 Web 应用程序已经完成，我们将构建相同的内容并将其部署到我们的桌面应用程序中。

要开始，请返回到`web-app`文件夹的终端/提示符并运行：

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

我们编写的所有代码最终都被捆绑到了前述文件中。我们将获取`dist`文件夹内的所有文件（而不是`dist`文件夹），然后将它们粘贴到`desktop-app/app`文件夹内。这些更改后桌面应用程序的最终结构将如下所示：

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

进行测试，运行：

```js
npm start  
```

然后，当我们导航到 VIEW DEVICE 页面时，我们应该看到以下内容：

![](img/00106.jpeg)

现在桌面应用程序已经完成，我们将开始处理移动应用程序。

# 更新移动应用程序

为了在移动应用程序中反映新的模板，我们将更新`mobile-app/src/pages/view-device/view-device.html`，如下所示：

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
        <ion-label>Signal Magnitude Vector</ion-label> 
        <ion-label>{{lastRecord.data.acclVals.smv}}</ion-label> 
      </ion-item> 
      <ion-item> 
        <ion-label>Fall State</ion-label> 
        <ion-label>{{lastRecord.data.acclVals.fall ? 'Patient Down' : 'All is well!'}}</ion-label> 
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

保存所有文件并通过以下方式运行移动应用程序：

```js
ionic serve  
```

您也可以使用：

```js
ionic cordova run android 
```

我们应该看到以下内容：

![](img/00107.gif)

# 总结

在本章中，我们使用了跌倒检测和 IFTTT 的概念。使用我们在第六章中构建的智能可穿戴设备，我们添加了跌倒检测逻辑。然后，我们将相同的数据发送到 API 引擎，并在 API 引擎中构建了自己的 IFTTT 规则引擎。我们定义了一个规则，用于在检测到跌倒时发送电子邮件。

除此之外，我们还更新了 Web、桌面和移动应用程序，以反映我们收集到的新数据。

在第八章中，*树莓派图像流*，我们将使用树莓派进行视频监控。
