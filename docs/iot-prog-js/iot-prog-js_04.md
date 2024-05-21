# 第四章：控制连接的设备

在本章中，我们将看看如何使用我们的 Raspberry Pi Zero 和 Arduino UNO 从远程站点控制设备，使用以下模块在网络中进行通信：Wi-Fi shield 和 Ethernet shield。我们将在本章中涵盖以下主题：

+   使用 Node.js 创建一个简单的 Web 服务器

+   使用 Restful API 和 Node.js 从 Raspberry Pi Zero 控制继电器

+   在计算机上配置 Node.js 作为 Web 服务器

+   使用 Node.js 和 Arduino Wi-Fi 监控温度、湿度和光线

+   使用 Node.js 和 Arduino Ethernet 监控温度、湿度和光线

# 使用 Node.js 创建一个简单的 Web 服务器

拥有 Raspberry Pi 最重要的一个方面是我们有一个配置了服务和服务器的真正的计算机。在本节中，我们将解释如何安装 Node.js，这是一个强大的框架，我们将用它来运行我们将在本书中看到的大多数应用程序。幸运的是，我们在 Raspberry Pi 上安装 Node.js 非常简单。

在本章的文件夹中，打开名为`webserver.js`的文件。我们将在端口*8056*上创建一个服务器。要测试程序并查看结果，我们必须在 MS-DOS 界面上打开 Node.js 终端，并使用以下命令运行此文件：

```js
**node webserver.js**

```

添加以下行到`webserver.js`文件中声明 HTTP 请求命令：

```js
var http = require('http'); 

```

我们使用以下函数创建服务器：

```js
http.createServer(function (req, res) { 

```

我们定义将在 HTML 代码中显示的文件内容：

```js
res.writeHead(200, {'Content-Type': 'text/plain'}); 

```

我们从服务器发送响应：

```js
res.end('Hello  from Node.js'); 

```

重要的是要定义要打开的端口：

```js
}).listen(8056); 

```

显示服务器的消息：

```js
console.log('Server running at port 8056'); 

```

要测试此程序，请在本地计算机上打开浏览器，导航到以下链接：`http://192.168.1.105:8056`。如果您看到以下屏幕；您的 Node.js 服务器在您的计算机上运行正常；您需要更改计算机的 IP 地址：

![使用 Node.js 创建一个简单的 Web 服务器](img/B05170_04_01.jpg)

# 使用 Restful API 和 Node.js 从 Raspberry Pi Zero 控制继电器

在本节中，我们将向您展示如何控制连接到 Arduino UNO 板的继电器模块，以及用于从 Web 浏览器发送命令的继电器。让我们开始吧。

## JSON 结构

**JavaScript 对象表示法** **(JSON)** 是一种轻量级的数据交换格式。它易于人类阅读和编写。它易于机器解析和生成。它基于 JavaScript 编程语言的一个子集。

JSON 建立在两种结构上：

+   一组名称/值对。在各种语言中，这被实现为对象、记录、结构、字典、哈希表、键控列表或关联数组。

+   一系列值的有序列表。在大多数语言中，这被实现为数组、向量、列表或序列。

首先，我们需要知道如何应用我们用来描述这个结构的 JSON 格式，如下所示：

```js
{"data": "Pin D6 set to 1", "id": "1", "name": "Arduino", "connected": true}
```

这是我们需要遵循并使响应的格式：

+   **数据：**定义命令的编号，然后描述命令的定义

+   **名称：**跟随设备的名称

+   **已连接：**确认设备是否已连接

所有在`{ }`之间的数据定义了我们的 JSON 格式。

## 使用 aREST API 的命令

使用`aREST`命令，我们可以定义我们的 Arduino 和设备，然后从 Web 浏览器控制它们。以下是`aREST` API 的命令示例：

+   `设备的 IP 地址/模式/6/o`：这将配置数字引脚 6 为输出引脚

+   `设备的 IP 地址/digital/6/1`：配置输出 6 并使函数像数字写入一样。例如：`http://192.168.1.100/digital/6/1`；我们定义设备的 IP 地址和将被激活的引脚的编号。

## 在您的 Raspberry Pi Zero 上安装 Node.js

Node.js 是一个工具，它将允许我们使用 JavaScript 代码在设备上创建运行的服务器。最重要的是，我们将应用这个框架来使用这段代码构建一个 Web 服务器。

使用 Node.js 意味着我们配置了一个将打开端口并连接到 Web 服务器的设备的 Web 服务器。

使用以下命令，您将在树莓派 Zero 上安装 Node.js：

```js
**sudo apt-get install nodejs**

```

NPM 是 Node.js 的 JavaScript 运行时环境的默认包管理器。要配置和安装`aREST`模块，请在终端中输入以下命令：

```js
**sudo npm install arest**

```

Express 的理念是为 HTTP 服务器提供小巧、强大的工具，使其成为单页应用程序、网站、混合应用程序或公共 HTTP API 的绝佳解决方案。

我们还需要使用以下命令配置 express 模块：

```js
**sudo npm install express**

```

# 使用 aREST 命令从 Web 浏览器控制继电器

在接下来的部分中，我们将看到如何使用`Rest`命令从 Web 浏览器控制数字输出。让我们深入了解一下，以了解更多细节：

## 配置 Web 服务器

现在，您可以将代码复制到名为 outputcontrol.js 的文件中，或者只需从此项目的文件夹中获取完整的代码并使用 Node.js 执行它。在树莓派上打开终端并输入以下命令：

```js
**sudo node output control.js**

```

我们通过以下方式定义设备的 GPIO 来导入命令：

```js
var gpio = require('rpi-gpio'); 

```

现在我们将使用以下代码使用 Node.js 创建我们的 Web 服务器。

我们导入运行所需的 require 包。我们使用以下声明库：

```js
var express = require('express'); 
var app = express(); 

```

定义 body 解析器并打开端口，在本例中为*8099*：

```js
var Parser = require('body-parser'); 
var port = 8099; 

```

使用 body-parser：

```js
app.use(Parser.urlencoded({ extended: false })); 
app.use(Parser.json()); 

```

配置**GPIO 11**，我们将控制它：

```js
gpio.setup(11,gpio.DIR_OUT); 

```

我们定义将从 Web 浏览器调用的函数。

函数的名称是`ledon`；它激活**GPIO 11**并向屏幕发送消息`led1 is on`：

```js
function ledon() { 
    setTimeout(function() { 
        console.log('led1 is on'); 
        gpio.write(11, 1); 
      }, 2000); 
} 

```

函数的名称是`ledoff`；它关闭**GPIO 11**并向屏幕发送消息`led1 is off`：

```js
function ledoff() { 
    setTimeout(function() { 
        console.log('led1 is off'); 
        gpio.write(11, 0); 
   }, 2000); 
} 

```

我们定义了`GET`函数，这意味着当浏览器接收到名为`ledon`的函数时，它会向服务器发出请求；服务器以以下格式做出响应：`{status:"connected",led:"on"}`。

现在我们将声明用于来自客户端的传入请求的 app 函数：

```js
app.get('/ledon', function (req, res) { 
    ledon(); 
    var data ={status:"connected",led:"on"}; 
    res.json(data); 
}); 

```

我们定义`GET`函数。这意味着当浏览器接收到名为`/ledoff`的函数时，它会向服务器发出请求；服务器以以下格式做出响应：`{status:"connected",led:"off"}`。

```js
app.get('/ledoff', function (req, res) { 
    ledoff(); 
    var data ={status:"connected",led:"off"}; 
    res.json(data); 
}); 

```

现在我们打开 Web 服务器的端口：

```js
app.listen(port); 
console.log('Server was started on ' + port); 

```

如果一切正确，我们打开我们喜爱的浏览器并输入`http://您的 Raspberry_PI_zero 的 IP 地址:端口/命令`。

`在这种情况下，我们输入 192.168.1.105:8099/ledon`。

以下屏幕截图显示了 JSON 请求的响应：

![配置 Web 服务器](img/B05170_04_02.jpg)

接下来，我们将看到最终结果，如下图所示：

![配置 Web 服务器](img/B05170_04_03.jpg)

# 在计算机上配置 Node.js 作为 Web 服务器

Node.js 是一个开源的跨平台运行时环境，用于开发服务器端和网络应用程序。Node.js 应用程序是用 JavaScript 编写的，并可以在 OS X、Microsoft Windows 和 Linux 上的 Node.js 运行时环境中运行。

Node.js 还提供了丰富的各种 JavaScript 模块库，极大地简化了使用 Node.js 开发 Web 应用程序的过程。

在上一节中，我们在树莓派 Zero 上配置了 Node.js，现在在本节中，我们将使用 Windows 操作系统执行相同的操作，并配置我们的 Web 服务器 Node.js 在其上运行。

本节的主要目的是解释如何从在 Node.js 框架中运行的 Web 服务器控制我们的 Arduino 板。为此，安装它非常重要；我们的系统将在 Windows 计算机上运行。

在本节中，我们将解释如何在 Windows 上安装 Node.js。

## 下载 Node.js

首先，我们需要下载 Windows 64 位的 Node.js-它取决于您的操作系统版本，您只需要转到以下链接进行下载：[`nodejs.org/es/download/`](https://nodejs.org/es/download/)：

![下载 Node.js](img/B05170_04_04.jpg)

## 安装 Node.js

在下载了软件之后，按照以下步骤进行：

1.  点击**下一步**按钮：![安装 Node.js](img/B05170_04_05.jpg)

1.  点击**下一步**按钮：![安装 Node.js](img/B05170_04_06.jpg)

1.  选择安装位置：![安装 Node.js](img/B05170_04_07.jpg)

1.  选择默认配置：![安装 Node.js](img/B05170_04_08.jpg)

1.  完成配置后，点击**安装**：![安装 Node.js](img/B05170_04_09.jpg)

1.  安装完成后我们将看到以下内容：![安装 Node.js](img/B05170_04_10.jpg)

## 使用 Node.js 配置 web 服务器端口 8080

现在我们需要配置将要监听来自远程浏览器的打开连接的端口。打开本章文件夹中的文件，然后使用 Node.js 执行该文件。

现在你可以把代码复制到一个名为`server.js`的文件中，或者直接从这个项目的文件夹中获取完整的代码。

首先我们需要使用以下代码创建我们的服务器：

```js
var server = require('http'); 

```

创建一个名为`loadServer`的函数，其中包含响应浏览器的代码：

```js
function loadServer(requiere,response){ 
      console.log("Somebody is connected");     

```

如果这个函数响应数字 200，那么意味着连接已经建立，服务器工作正常：

```js
response.writeHead(200,{"Content-Type":"text/html"}); 
      response.write("<h1>The Server works perfect</h1>"); 
      response.end(); 
} 

```

创建并打开服务器端口：

```js
server.createServer(loadServer).listen(8080); 

```

在你的计算机上打开安装了 Node.js 服务器的终端，然后在 MS-DOS 界面中输入以下命令：

```js
**C:\users\PC>node server.js**

```

现在，为了测试服务器是否运行，我们将在网页浏览器中输入`localhost:端口号`；你应该在屏幕上看到类似以下截图的内容：

```js
http://localhost:8080  

```

![使用 Node.js 配置 web 服务器端口 8080](img/B05170_04_11.jpg)

# 使用 Node.js 和 Arduino Wi-Fi 监控温度、湿度和光线

在本章的这部分，我们将解释 Arduino 的 Wi-Fi shield 的代码：

![使用 Node.js 和 Arduino Wi-Fi 监控温度、湿度和光线](img/B05170_04_12.jpg)

我们定义变量的数量；在这种情况下，我们将监控三个变量（`温度`、`湿度`和`光线`）：

```js
#define NUMBER_VARIABLES 3 

```

这里我们必须包含传感器的库：

```js
#include "DHT.h" 

```

我们定义传感器的引脚：

```js
#define DHTPIN 7  
#define DHTTYPE DHT11 

```

我们定义传感器的实例：

```js
DHT dht(DHTPIN, DHTTYPE); 

```

我们导入模块的库：

```js
#include <Adafruit_CC3000.h> 
#include <SPI.h> 
#include <CC3000_MDNS.h> 
#include <aREST.h> 

```

我们定义连接模块的引脚：

```js
using a breakout board 
#define ADAFRUIT_CC3000_IRQ   3 
#define ADAFRUIT_CC3000_VBAT  5 
#define ADAFRUIT_CC3000_CS    10 

```

我们创建将要连接的模块的实例：

```js
Adafruit_CC3000 cc3000 = Adafruit_CC3000(ADAFRUIT_CC3000_CS,  
ADAFRUIT_CC3000_IRQ, ADAFRUIT_CC3000_VBAT); 

```

我们定义 aREST 实例：

```js
aREST rest = aREST(); 

```

然后我们定义 SSID 和密码，你需要进行更改：

```js
#define WLAN_SSID       "xxxxx" 
#define WLAN_PASS       "xxxxx" 
#define WLAN_SECURITY   WLAN_SEC_WPA2 

```

我们配置端口以监听传入的 TCP 连接：

```js
#define LISTEN_PORT           80 

```

我们定义模块的服务器实例：

```js
Adafruit_CC3000_Server restServer(LISTEN_PORT); 
// DNS responder instance 
MDNSResponder mdns; 

```

我们定义将要发布的变量：

```js
int temp; 
int hum; 
int light; 

```

这里有一个设置，定义了串行通信的配置：

```js
void setup(void) 
{   
  // Start Serial 
  Serial.begin(115200);  
  dht.begin(); 

```

我们开始将要发布的变量：

```js
  rest.variable("light",&light); 
  rest.variable("temp",&temp); 
  rest.variable("hum",&hum); 

```

我们定义设备的 ID 和名称：

```js
  rest.set_id("001"); 
  rest.set_name("monitor"); 

```

我们连接到网络：

```js
  if (!cc3000.begin()) 
  { 
    while(1); 
  } 
  if (!cc3000.connectToAP(WLAN_SSID, WLAN_PASS, WLAN_SECURITY)) { 
    while(1); 
  } 
  while (!cc3000.checkDHCP()) 
  { 
    delay(100); 
  } 

```

这里我们定义了连接设备的函数：

```js
  if (!mdns.begin("arduino", cc3000)) { 
    while(1);  
  } 

```

我们在串行接口中显示连接：

```js
  displayConnectionDetails(); 
  restServer.begin(); 
  Serial.println(F("Listening for connections...")); 
} 

```

在这部分，我们声明将要获取的变量：

```js
void loop() { 
  temp = (float)dht.readTemperature(); 
  hum = (float)dht.readHumidity(); 

```

然后我们测量光线水平：

```js
  float sensor_reading = analogRead(A0); 
  light = (int)(sensor_reading/1024*100); 

```

我们声明请求的函数：

```js
  mdns.update(); 

```

我们需要执行来自服务器的请求：

```js
Adafruit_CC3000_ClientRef client = restServer.available(); 
  rest.handle(client); 
} 

```

我们显示设备的网络配置：

```js
bool displayConnectionDetails(void) 
{ 
  uint32_t ipAddress, netmask, gateway, dhcpserv, dnsserv; 
  if(!cc3000.getIPAddress(&ipAddress, &netmask, &gateway, &dhcpserv, &dnsserv)) 
  { 
Serial.println(F("Unable to retrieve the IP Address!\r\n")); 
    return false; 
  } 
  else 
  { 
    Serial.print(F("\nIP Addr: ")); cc3000.printIPdotsRev(ipAddress); 
    Serial.print(F("\nNetmask: ")); cc3000.printIPdotsRev(netmask); 
    Serial.print(F("\nGateway: ")); cc3000.printIPdotsRev(gateway); 
    Serial.print(F("\nDHCPsrv: ")); cc3000.printIPdotsRev(dhcpserv); 
    Serial.print(F("\nDNSserv: ")); cc3000.printIPdotsRev(dnsserv); 
    Serial.println(); 
    return true; 
  } 
} 

```

在你的 Arduino 板上下载代码草图，然后转到串行监视器以查看从路由器获取的 IP 地址配置。之后，我们可以显示 Wi-Fi shield 的 IP 地址配置：

![使用 Node.js 和 Arduino Wi-Fi 监控温度、湿度和光线](img/B05170_04_13.jpg)

## 连接到 Wi-Fi 网络

现在我们可以看到你的 Arduino Wi-Fi shield 的 IP 地址，我们现在可以将我们的计算机连接到与 Arduino 板相同的网络。查看以下截图以获取更多细节：

![连接到 Wi-Fi 网络](img/B05170_01_23.jpg)

为了测试应用程序，我们需要转到以下路径，并在安装了 Node.js 服务器的计算机上运行以下命令，或者如下截图所示：

![连接到 Wi-Fi 网络](img/B05170_04_15.jpg)

在这个文件夹中，我们有 JavaScript 文件，输入命令 node app.js

在输入接口文件夹后，输入以下命令`node app.js`：

![连接到 Wi-Fi 网络](img/B05170_04_16.jpg)

现在您已经启动了 Web 服务器应用程序，切换到浏览器，在同一台机器上输入机器的 IP 地址以查看结果：

![连接到 Wi-Fi 网络](img/B05170_04_17.jpg)

服务器监听端口 300 后，它与 Wi-Fi 模块建立通信，向设备的 IP 地址发送请求：

![连接到 Wi-Fi 网络](img/B05170_04_18.jpg)

# 使用 Node.js 与 Arduino Ethernet 监控温度、湿度和光照

在上一节中，我们展示了如何使用*CC3000*模块通过 Wi-Fi 监视我们的 Arduino；现在我们将使用另一个重要模块：以太网盾。部分的硬件连接类似于以下图像：

![使用 Node.js 与 Arduino Ethernet 监控温度、湿度和光照](img/B05170_04_19.jpg)

## Arduino Ethernet 盾应用程序代码

现在您可以将代码复制到名为`Monitor_Ethernet.ino`的文件中，或者只需从此项目的文件夹中获取完整的代码；您需要使用 Arduino IDE。

以下是程序中包含的库：

```js
#include <SPI.h> 
#include <Ethernet.h> 
#include <aREST.h> 
#include <avr/wdt.h> 

```

包括 DHT11 传感器的库：

```js
#include "DHT.h" 

```

我们定义温度和湿度传感器的引脚：

```js
#define DHTPIN 7  
#define DHTTYPE DHT11 

```

我们有传感器的实例：

```js
DHT dht(DHTPIN, DHTTYPE); 

```

我们注册设备的 MAC 地址：

```js
byte mac[] = { 0x90, 0xA2, 0xDA, 0x0E, 0xFE, 0x40 }; 
IPAddress ip(192,168,1,153); 
EthernetServer server(80); 

```

我们现在创建`aREST` API 的实例：

```js
aREST rest = aREST(); 

```

我们发布将被监视的变量：

```js
int temp; 
int hum; 
int light; 

```

我们现在配置串行通信并启动传感器的实例：

```js
void setup(void) 
{   
  // Start Serial 
  Serial.begin(115200); 
  dht.begin(); 

```

我们开始发布变量：

```js
  rest.variable("light",&light); 
  rest.variable("temp",&temp); 
  rest.variable("hum",&hum); 

```

非常重要的是给出我们正在使用的设备的 ID 和名称：

```js
  rest.set_id("008"); 
  rest.set_name("Ethernet"); 

```

我们开始以太网连接：

```js
if (Ethernet.begin(mac) == 0) { 
    Serial.println("Failed to configure Ethernet using DHCP"); 
    Ethernet.begin(mac, ip); 
  } 

```

我们在串行监视器上显示 IP 地址：

```js
  server.begin(); 
  Serial.print("server is at "); 
  Serial.println(Ethernet.localIP()); 
  wdt_enable(WDTO_4S); 
} 

```

我们读取温度和湿度传感器：

```js
void loop() {   

  temp = (float)dht.readTemperature(); 
  hum = (float)dht.readHumidity(); 

```

我们测量传感器的光照水平：

```js
  float sensor_reading = analogRead(A0); 
  light = (sensor_reading/1024*100); 

```

我们监听将连接的传入客户端：

```js
  EthernetClient client = server.available(); 
  rest.handle(client); 
  wdt_reset(); 
} 

```

现在我们已经完成了配置，打开一个网络浏览器并输入您的 Arduino Ethernet 盾的 IP 地址：`http://192.168.1.153`。如果一切顺利，它将显示以下屏幕，并显示来自板的 JSON 响应：

![Arduino Ethernet 盾应用程序代码](img/B05170_04_20.jpg)

上述截图显示了 JSON 请求的结果。

## 在 Node.js 中配置设备

在本节中，我们将解释配置我们可以从网页控制的设备的代码。

在上一节中，我们安装了 express 包；如果遇到任何困难，只需打开终端并输入以下内容：

```js
**npm install express**

```

我们定义节点 express 并创建应用程序：

```js
var express = require('express'); 
var app = express(); 

```

然后我们定义要监听的端口：

```js
var port = 3000; 

```

我们定义 Jade 应用程序的实例，使用视图引擎：

```js
app.set('view engine', 'jade'); 

```

我们配置公共文件夹：

```js
app.use(express.static(__dirname + '/public')); 

```

我们现在定义要监视的设备：

```js
var rest = require("arest")(app); 
rest.addDevice('http','192.168.1.153'); 

```

我们提供应用程序：

```js
app.get('/', function(req, res){ 
res.render('interface'); 
}); 

```

我们启动服务器并在设备连接时发送消息：

```js
app.listen(port); 
console.log("Listening on port " + port); 

```

在 MS-DOS 中打开终端并在 Node.js 服务器中执行`app.js`

要测试应用程序，请打开您的网络浏览器并输入`http://localhost:3000`；如果出现以下屏幕，恭喜，您已正确配置了服务器：

![在 Node.js 中配置设备](img/B05170_04_21.jpg)

这里是我们在 Node.js 服务器中看到`app.js`执行的屏幕：

![在 Node.js 中配置设备](img/B05170_04_22.jpg)

# 摘要

在本章中，您学习了如何使用树莓派 Zero 中的网络通信模块来控制 Arduino 板，在中央界面仪表板中。我们已经看到了如何从中央界面控制和监视设备；您可以使用其他传感器，例如气压传感器。

在下一章中，您将进行更多有趣的项目，例如将网络摄像头配置和连接到您的 Arduino 板，从您的 Raspberry Pi Zero 上进行监控。
