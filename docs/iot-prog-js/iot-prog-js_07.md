# 第七章。用物联网仪表盘构建间谍警察

在本章中，我们将看几个家庭项目。您可以将这些项目与我们在前几章中看到的其他工具结合使用。这样做将有助于提高您的知识，也让您自己开发。本章将涵盖以下主题：

+   检测噪音的间谍麦克风

+   调节交流灯调光器的电流

+   用 RFID 卡控制访问

+   检测烟雾

+   使用树莓派 Zero 构建报警系统

+   从远程仪表盘监控气候

# 检测噪音的间谍麦克风

在本节中，我们将看一个可以在房子里使用的项目，用于检测噪音或声音级别，以便我们可以检测到有人在房子前说话。这个项目包括一个带有麦克风的模块，类似于下面的图像：

![检测噪音的间谍麦克风](img/B05170_07_01.jpg)

## 软件代码

我们需要制作一个程序，可以读取模块发送到 Arduino 板的模拟信号：

```js
const int ledPin =  12;         // the number of the LED pin 
const int thresholdvalue = 400; //The threshold to turn the led on 

void setup() { 
 pinMode(ledPin, OUTPUT); 
 Serial.begin(9600); 
} 

void loop() { 
  int sensorValue = analogRead(A0);   //use A0 to read the electrical signal 
  Serial.print("Noise detected="); 
  Serial.println(sensorValue); 
  delay(100); 
  if(sensorValue > thresholdvalue) 
  digitalWrite(ledPin,HIGH);//if the value read from A0 is larger than 400,then light the LED 
  delay(200); 
  digitalWrite(ledPin,LOW); 
} 

```

然后我们下载草图，在下面的截图中我们有声音级别的结果：

![软件代码](img/B05170_07_02.jpg)

在下面的图像中，我们可以看到最终的电路连接到 Arduino 板：

![软件代码](img/B05170_07_03.jpg)

# 调节交流灯调光器的电流

在本节中，我们将看到如何调节交流灯。多年来，我一直想解释和分享这样的项目，现在终于可以了。这可以应用于调节家里的灯，以减少家庭用电量：接下来的章节将更详细地解释这个项目。

## 硬件要求

我们需要以下电子元件：

+   H 桥

+   24V 交流变压器

+   两个电阻 22k（1 瓦）

+   一个集成电路（4N25）

+   一个电阻 10k

+   一个 5k 的电位计

+   一个电阻 330 欧姆

+   一个电阻 180 欧姆

+   一个集成电路 MOC3011

+   一个 TRIAC 2N6073

在下面的电路图中，我们可以看到 Arduino 板上调光器的连接：

![硬件要求](img/B05170_07_04.jpg)

## 软件代码

现在您可以将代码复制到一个名为`Dimner.ino`的文件中，或者只需从此项目的文件夹中获取完整的代码：

```js
int load = 10;  
int intensity = 128; 

void setup() 
{ 
pinMode(loaf, OUTPUT); 
attachInterrupt(0, cross_zero_int, RISING); 
} 

void loop() 
{ 
intensity = map(analogRead(0),0,1023,10,128); 
} 

void cross_zero_int() 
{ 
int dimtime = (65 * intensity);  
delayMicroseconds(dimtime);  
digitalWrite(load, HIGH);  
delayMicroseconds(8);  
digitalWrite(load, LOW); 
} 

```

在下载了草图之后，我们可以看到最终的结果。通过电位器，我们可以调节灯的亮度。在我们的房子里，我们可以随时打开灯：也许我们可以根据环境光控制它们。

在下面的图像中，我们将看到灯的不同时刻，如果我们调节电位器的输入信号：

![软件代码](img/B05170_07_05.jpg)

在下面的图像中，我们可以看到调节灯的亮度的结果：

![软件代码](img/B05170_07_06.jpg)

在这里我们可以看到灯的调光器处于最大亮度：

![软件代码](img/B05170_07_07.jpg)

# 用 RFID 卡控制访问

在本节中，我们将看到如何通过门控制访问。在上一章中，我们看到了如何控制房子的锁和灯。这个项目可以作为上一个项目的补充，因为它可以让您控制门的打开，特定卧室的门，或其他房间的灯。

## 硬件要求

对于这个项目，我们需要以下设备：

+   读取标签卡

+   RFID RC522 模块

+   Arduino 板

下图显示了用于读取和控制访问的 RFID 标签：

![硬件要求](img/B05170_07_08.jpg)

下图显示了 Arduino 的 RFID 卡接口：

![硬件要求](img/RFID-RC522-pinout.jpg)

## 软件要求

我们需要安装`<MFRC522.h>`库，这个文件可以与并配置模块以读取标签卡进行通信。这个库可以从[`github.com/miguelbalboa/rfid`](https://github.com/miguelbalboa/rfid)下载。

## 软件代码

现在你可以将代码复制到名为`RFID.ino`的文件中，或者直接从该项目的文件夹中获取完整的代码：

```js
#include <MFRC522.h> 
#include <SPI.h> 
#define SAD 10 
#define RST 5 

MFRC522 nfc(SAD, RST); 

#define ledPinOpen  2 
#define ledPinClose 3 

void setup() { 
  pinMode(ledPinOpen,OUTPUT);    
  pinMode(ledPinClose,OUTPUT); 

  SPI.begin(); 
  Serial.begin(115200); 
  Serial.println("Looking for RC522"); 
  nfc.begin(); 
  byte version = nfc.getFirmwareVersion(); 

  if (! version) { 
    Serial.print("We don't find RC522"); 
    while(1); 
  } 
  Serial.print("Found RC522"); 
  Serial.print("Firmware version 0x"); 
  Serial.print(version, HEX); 
  Serial.println("."); 
} 

#define AUTHORIZED_COUNT 2 //number of cards Authorized 
byte Authorized[AUTHORIZED_COUNT][6] = {{0xC6, 0x95, 0x39, 0x31, 0x5B},{0x2E, 0x7, 0x9A, 0xE5, 0x56}}; 

void printSerial(byte *serial); 
boolean isSame(byte *key, byte *serial); 
boolean isAuthorized(byte *serial); 

void loop() { 
  byte status; 
  byte data[MAX_LEN]; 
  byte serial[5]; 
  boolean Open = false; 
  digitalWrite(ledPinOpen, Open); 
  digitalWrite(ledPinClose, !Open); 
  status = nfc.requestTag(MF1_REQIDL, data); 

  if (status == MI_OK) { 
    status = nfc.antiCollision(data); 
    memcpy(serial, data, 5); 

    if(isAuthorized(serial)) 
    {  
      Serial.println("Access Granted"); 
      Open = true; 
    } 
    else 
    {  
      printSerial(serial); 
      Serial.println("NO Access"); 
      Open = false; 
    } 

    nfc.haltTag(); 
    digitalWrite(ledPinOpen, Open); 
    digitalWrite(ledPinClose, !Open); 
    delay(2000); 

  } 
  delay(500); 
} 

boolean isSame(byte *key, byte *serial) 
{ 
    for (int i = 0; i < 4; i++) { 
      if (key[i] != serial[i]) 
      {  
        return false;  
      } 
    } 
    return true; 
} 

boolean isAuthorized(byte *serial) 
{ 
    for(int i = 0; i<AUTHORIZED_COUNT; i++) 
    { 
      if(isSame(serial, Authorized[i])) 
        return true; 
    } 
   return false; 
} 
void printSerial(byte *serial) 
{ 
    Serial.print("Serial:"); 
    for (int i = 0; i < 5; i++) { 
    Serial.print(serial[i], HEX); 
    Serial.print(" "); 
    } 
} 

```

当我们将标签卡放在连接到 Arduino 的 RFID 模块前时，如果以下代码，它将显示消息（访问已授权）。

在代码的这一部分，我们配置了授权卡的数量：

```js
**#define AUTHORIZED_COUNT 2**
**byte Authorized[AUTHORIZED_COUNT][6] = {{0xC6, 0x95, 0x39, 0x31, 0x5B},
      {0x2E, 0x7, 0x9A, 0xE5, 0x56}};**

```

![软件代码](img/B05170_07_10.jpg)

如果我们将卡片放在未注册的标签和卡上，它可以提供正确的访问：

![软件代码](img/B05170_07_11.jpg)

完整连接的最终结果如下图所示：

![软件代码](img/B05170_07_12.jpg)

# 检测烟雾

在这一部分，我们将测试一个可以检测烟雾的**MQ135**传感器。这也可以用于家庭检测气体泄漏。在这种情况下，我们将用它来检测烟雾。

在家庭自动化系统中，将所有传感器放置在家中以检测某些东西，我们测量真实世界：在这种情况下，我们使用了可以检测气体和烟雾的 MQ135 传感器，如下图所示：

![检测烟雾](img/B05170_07_13.jpg)

## 软件代码

在下面的代码中，我们解释了如何使用气体传感器编程和检测烟雾：

```js
const int sensorPin= 0; 
const int buzzerPin= 12; 
int smoke_level; 

void setup() { 
Serial.begin(115200);  
pinMode(sensorPin, INPUT); 
pinMode(buzzerPin, OUTPUT); 
} 

void loop() { 
smoke_level= analogRead(sensorPin); 
Serial.println(smoke_level); 

if(smoke_level > 200){  
digitalWrite(buzzerPin, HIGH); 
} 

else{ 
digitalWrite(buzzerPin, LOW); 
} 
} 

```

如果它没有检测到烟雾，它会产生以下数值，如下截图所示：

![软件代码](img/B05170_07_14.jpg)

如果检测到烟雾，测量值在*305*和*320*之间，可以在文件中看到如下截图：

![软件代码](img/B05170_07_15.jpg)

完整电路连接的最终结果如下图所示：

![软件代码](img/B05170_07_16.jpg)

# 使用树莓派 Zero 构建警报系统

在这一部分，我们将使用一个 PIR 传感器连接到树莓派 Zero 来构建一个简单的警报系统。这是一个重要的项目，因为它可以添加到家庭中，包括其他传感器，以便监控。

## 使用树莓派 Zero 的运动传感器

对于这个项目，我们需要树莓派 Zero、一个运动传感器 PIR 和一些电缆。实际上，这个项目的硬件配置将非常简单。首先，将运动传感器的**VCC**引脚连接到树莓派上的一个**3.3V**引脚。然后，将传感器的**GND**引脚连接到树莓派上的一个**GND**引脚。最后，将运动传感器的**OUT**引脚连接到树莓派上的**GPIO17**引脚。你可以参考前面的章节了解树莓派 Zero 板上的引脚映射。

以下图片显示了连接的最终电路：

![使用树莓派 Zero 的运动传感器](img/B05170_07_17.jpg)

## 软件代码

现在你可以将代码复制到名为`Project1`的文件夹中，或者直接从该项目的文件夹中获取完整的代码：

```js
// Modules 
var express = require('express'); 

// Express app 
var app = express(); 

// aREST 
var piREST = require('pi-arest')(app); 
piREST.set_id('34f5eQ'); 
piREST.set_name('motion_sensor'); 
piREST.set_mode('bcm'); 

// Start server 
app.listen(3000, function () { 
  console.log('Raspberry Pi Zero motion sensor started!'); 
}); 

```

## 警报模块

通常，家中会有模块，在检测到运动时会闪烁灯光并发出声音。当然，你也可以将其连接到真正的警报器而不是蜂鸣器，以便在检测到任何运动时发出响亮的声音。

要组装这个模块，首先将 LED 与 330 欧姆电阻串联放在面包板上，LED 的最长引脚与电阻接触。还要将蜂鸣器放在面包板上。然后，将电阻的另一端连接到树莓派上的**GPIO14**，LED 的另一端连接到树莓派上的一个**GND**引脚。对于蜂鸣器，将标有**+**的引脚连接到**GPIO15**，另一端连接到树莓派上的一个**GND**引脚。

## 软件代码

下面是编码细节：

```js
// Modules 
var express = require('express'); 

// Express app 
var app = express(); 

// aREST 
var piREST = require('pi-arest')(app); 
piREST.set_id('35f5fc'); 
piREST.set_name('alarm'); 
piREST.set_mode('bcm'); 

// Start server 
app.listen(3000, function () { 
  console.log('Raspberry Pi Zero alarm started!'); 
}); 

```

这是显示连接的最终电路：

![软件代码](img/B05170_07_18.jpg)

## 中央接口

首先，我们使用以下代码为应用程序创建一个中央接口：

```js
// Modules 
var express = require('express'); 
var app = express(); 
var request = require('request'); 

// Use public directory 
app.use(express.static('public')); 

// Pi addresses 
var motionSensorPi = "192.168.1.104:3000"; 
var alarmPi = "192.168.1.103:3000" 

// Pins 
var buzzerPin = 15; 
var ledPin = 14; 
var motionSensorPin = 17; 

// Routes 
app.get('/', function (req, res) { 
res.sendfile(__dirname + '/public/interface.html'); 
}); 

app.get('/alarm', function (req, res) { 
  res.json({alarm: alarm}); 
}); 

app.get('/off', function (req, res) { 

  // Set alarm off 
  alarm = false; 

  // Set LED & buzzer off 
  request("http://" + alarmPi + "/digital/" + ledPin + '/0'); 
  request("http://" + alarmPi + "/digital/" + buzzerPin + '/0'); 

  // Answer 
  res.json({message: "Alarm off"}); 

}); 

// Start server 
var server = app.listen(3000, function() { 
    console.log('Listening on port %d', server.address().port); 
}); 

// Motion sensor measurement loop 
setInterval(function() { 

  // Get data from motion sensor 
  request("http://" + motionSensorPi + "/digital/" + motionSensorPin, 
    function (error, response, body) { 

      if (!error && body.return_value == 1) { 

        // Activate alarm 
        alarm = true; 

        // Set LED on 
        request("http://" + alarmPi + "/digital/" + ledPin + '/1'); 

        // Set buzzer on 
        request("http://" + alarmPi + "/digital/" + buzzerPin + '/1'); 

      } 
  }); 

}, 2000);
```

## 图形界面

现在让我们看看界面文件，从 HTML 开始。我们首先导入项目所需的所有库和文件。

```js
<!DOCTYPE html> 
<html> 

<head> 
  <script src="https://code.jquery.com/jquery-2.2.4.min.js"></script> 
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css"> 
  <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"></script> 
  <script src="js/script.js"></script> 
  <link rel="stylesheet" href="css/style.css"> 
  <meta name="viewport" content="width=device-width, initial-scale=1"> 
</head> 

<script type="text/javascript"> 

/* Copyright (C) 2007 Richard Atterer, richardÂ©atterer.net 
   This program is free software; you can redistribute it and/or modify it 
   under the terms of the GNU General Public License, version 2\. See the file 
   COPYING for details. */ 

var imageNr = 0; // Serial number of current image 
var finished = new Array(); // References to img objects which have finished downloading 
var paused = false; 

</script> 
<div id="container"> 

  <h3>Security System</h3> 
  <div class='row voffset50'> 
  <div class='col-md-4'></div> 
  <div class='col-md-4 text-center'> 
      Alarm is OFF 
    </div> 
    <div class='col-md-4'></div> 

  </div> 

  <div class='row'> 

    <div class='col-md-4'></div> 
    <div class='col-md-4'> 
      <button id='off' class='btn btn-block btn-danger'>Deactivate Alarm</button> 
    </div> 
    <div class='col-md-4'></div> 

  </div> 

  </div> 

</body> 
</html> 

```

# 从远程仪表板监控气候

如今，大多数智能家居都连接到互联网，这使用户能够监控他们的家。在本节中，我们将学习如何远程监控您的气候。首先，我们只需向树莓派 Zero 添加一个传感器，并从云仪表板监测测量值。让我们看看它是如何工作的。

以下图片显示了最终的连接：

![从远程仪表板监控气候](img/B05170_07_19.jpg)

## 探索传感器测试

```js
var sensorLib = require('node-dht-sensor'); 
var sensor = { 
    initialize: function () { 
        return sensorLib.initialize(11, 4); 
    }, 
    read: function () { 
        var readout = sensorLib.read(); 
        console.log('Temperature: ' + readout.temperature.toFixed(2) + 'C, ' + 
            'humidity: ' + readout.humidity.toFixed(2) + '%'); 
        setTimeout(function () { 
            sensor.read(); 
        }, 2000); 
    } 
}; 

if (sensor.initialize()) { 
    sensor.read(); 
} else { 
    console.warn('Failed to initialize sensor'); 
} 

```

## 配置远程仪表板（Dweet.io）

我们需要访问[`freeboard.io`](http://freeboard.io)并创建一个账户：

![配置远程仪表板（Dweet.io）](img/B05170_07_20.jpg)

现在，我们创建一个新的仪表板来控制传感器：

![配置远程仪表板（Dweet.io）](img/B05170_07_21.jpg)

使用以下参数添加新的数据源：

![配置远程仪表板（Dweet.io）](img/B05170_07_22.jpg)

在仪表板内创建一个新的窗格，并为温度创建一个**表盘**小部件：

![配置远程仪表板（Dweet.io）](img/B05170_07_23.jpg)

然后我们会立即在界面上看到温度：

![配置远程仪表板（Dweet.io）](img/B05170_07_24.jpg)

我们也可以用湿度做同样的操作：

![配置远程仪表板（Dweet.io）](img/B05170_07_25.jpg)

我们应该看到最终结果：

![配置远程仪表板（Dweet.io）](img/B05170_07_26.jpg)

# 总结

在本章中，我们学习了如何构建和集成基于树莓派 Zero 和 Arduino 板的模块化安全系统。当然，有许多方法可以改进这个项目。例如，您可以简单地向项目添加更多模块，比如更多触发相同警报的运动传感器。即使您不在家庭 Wi-Fi 网络之外，也可以监控系统。

在下一章中，我们将学习如何从 Android 应用程序控制您的系统，以及如何从智能手机集成一个真实的系统，这太棒了！
