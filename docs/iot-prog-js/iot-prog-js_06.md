# 第六章：从仪表板构建 Web 监视器和控制设备

在本章中，我们将讨论本书非常重要的一部分，即创建一个可以从仪表板控制不同类型设备的网页。在自动化的家庭中，有不同类型的设备可以被控制，例如：灯、门或窗户、洗衣机等等。

在本章中，我们将涵盖以下主题：

+   配置 MySQL 数据库服务器

+   安装 PhpMyAdmin 以管理数据库

+   带有 MySQL 的数据记录器

+   调光 LED

+   控制直流电机的速度

+   使用电路控制灯光

+   控制门锁

+   控制浇水的植物

+   从任何地方远程访问您的 Raspberry Pi Zero

+   控制灯光和测量电流消耗

+   从 Raspberry Pi Zero 控制和监视 Arduino、Wi-Fi 和以太网 shield、连接的设备和传感器

# 配置 MySQL 数据库服务器

在本节中，您将学习如何配置 MySQL 服务器，以创建数据库并将所有内容集成到您的仪表板中，以记录数据库中的数据。

## 安装 MySQL

我们的 Raspberry Pi Zero 正在配置为 Web 服务器。在本节中，我们将使用以下命令安装 MySQL 数据库服务器，以便我们可以接收来自客户端的连接，显示存储在数据库中的数据，并在 SQL 中使用查询：

```js
**sudo apt-get install mysql-server**

```

![安装 MySQL](img/B05170_06_01.jpg)

在过程中它会要求您输入`root`用户的密码：

![安装 MySQL](img/B05170_06_02.jpg)

安装完成后，连接到 MySQL 并键入以下命令：

```js
**mysql -u root -p**

```

![安装 MySQL](img/B05170_06_03.jpg)

键入以下命令：

```js
**show databases;**

```

![安装 MySQL](img/B05170_06_04.jpg)

在这里，我们可以看到现在安装在服务器上的系统数据库：

![安装 MySQL](img/B05170_06_05.jpg)

## 为 PHP 安装 MySQL 驱动程序

重要的是安装我们的驱动程序以使 PHP5 与 MySQL 数据库服务器通信，为此我们需要 MySQL 驱动程序以访问 MySQL 数据库，执行此命令以安装`PHP-MySQL`驱动程序。

```js
**sudo apt-get install php5 php5-mysql** 

```

## 测试 PHP 和 MySQL

在本节中，我们将使用以下命令创建一个简单的页面来测试 PHP 和 MySQL：

```js
**sudo nano /var/www/html/hellodb.php**

```

![测试 PHP 和 MySQL](img/B05170_06_06.jpg)

以下屏幕截图显示了包含访问数据库的代码、连接到服务器并从中获取数据的脚本：

![测试 PHP 和 MySQL](img/B05170_06_07.jpg)

要测试页面和 PHP 与 MySQL 之间的连接，请键入您的 Raspberry Pi 的 IP 地址：`http://192.168.1.105/hellodb.php`。页面应该类似于以下屏幕截图：

![测试 PHP 和 MySQL](img/B05170_06_08.jpg)

# 安装 PhpMyAdmin 以管理数据库

在本节中，我们将讨论如何配置您的 PhpMyAdmin 以从远程面板管理您的数据库。重要的是我们在 Apache 服务器中安装客户端和模块 PHP5，因此键入以下命令：

```js
**sudo apt-get install mysql-client php5-mysql** 

```

接下来，我们将使用以下命令安装`phpmyadmin`软件包：

```js
**sudo apt install phpmyadmin**

```

在下面的屏幕截图中，我们可以看到服务器的配置；在这种情况下，我们需要选择**apache2**：

![为管理数据库安装 PhpMyAdmin](img/B05170_06_09.jpg)

我们选择 apache2 服务器：

![为管理数据库安装 PhpMyAdmin](img/B05170_06_10.jpg)

之后我们可以选择数据库：

![为管理数据库安装 PhpMyAdmin](img/B05170_06_11.jpg)

我们选择**<No>**选项：

![为管理数据库安装 PhpMyAdmin](img/B05170_06_12.jpg)

## 配置 Apache 服务器

我们需要对文件`apache2.conf`进行配置。首先转到您的 Pi 上的终端：

```js
**sudo nano /etc/apache2/apache2.conf**

```

在下一个屏幕上，我们需要添加代码：

```js
**Include /etc/phpmyadmin/apche.conf**

```

![配置 Apache 服务器](img/B05170_06_13.jpg)

我们在文件底部包含以下行：

```js
**Include /etc/phpmyadmin/apche.conf**

```

![配置 Apache 服务器](img/B05170_06_14.jpg)

我们终于完成了安装我们的 Apache 服务器，现在我们已经准备好进行下一步了。

## 进入 phpMyAdmin 远程面板

在配置了服务器之后，我们将进入 phpMyAdmin 远程面板，我们需要打开我们喜欢的网络浏览器，并输入我们的树莓派的 IP 地址：`http://(树莓派地址)/phpmyadmin`，这将显示以下屏幕：

![进入 phpMyAdmin 远程面板](img/B05170_06_15.jpg)

## 显示 Arduinobd 数据库

以下截图显示了在服务器中创建的数据库：

![显示 Arduinobd 数据库](img/B05170_06_16.jpg)

以下截图显示了表**measurements**，列**id**，**temperature**和**humidity**：

![显示 Arduinobd 数据库](img/B05170_06_17.jpg)

## 从 Arduino 和以太网盾发送数据到 Web 服务器

我们使用 Arduino 和连接到网络的以太网盾，Arduino 将数据发送到树莓派 Zero 上发布的 Web 服务器。

您现在可以将代码复制到名为`arduino_xaamp_mysql.ino`的文件中，或者只需从本书的代码文件夹中获取完整的代码：

我们输入 Arduino UNO 的 IP 地址：

```js
IPAddress ip(192,168,1,50); 

```

我们配置了我们的树莓派 Zero 的 IP 地址：

```js
IPAddress server(192,168,1,108); 

```

我们需要连接到 Web 服务器：

```js
if (client.connect(server, 80)) 

```

这些行定义了从远程服务器发出的 HTTP 请求：

```js
client.println("GET /datalogger1.php?temp=" + temp + "&hum=" + hum + " HTTP/1.1"); 
      client.println("Host: 192.168.1.108"); 
      client.println("Connection: close"); 
      client.println(); 

```

其余的代码显示在以下行中：

```js
// Include libraries 
#include <SPI.h> 
#include <Ethernet.h> 
#include "DHT.h" 
// Enter a MAC address for your controller below. 
byte mac[] = { 0x90, 0xA2, 0xDA, 0x0E, 0xFE, 0x40 }; 
// DHT11 sensor pins 
#define DHTPIN 7  
#define DHTTYPE DHT11 
IPAddress ip(192,168,1,50); 
IPAddress server(192,168,1,108); 
EthernetClient client; 
DHT dht(DHTPIN, DHTTYPE); 
void setup() { 
  // Open serial communications 
  Serial.begin(9600); 
      Ethernet.begin(mac, ip); 
  Serial.print("IP address: "); 
  Serial.println(Ethernet.localIP()); 
  delay(1000); 
  Serial.println("Conectando..."); 

} 
void loop() 
{ 
  float h = dht.readHumidity(); 
  float t = dht.readTemperature(); 
  String temp = String((int) t); 
  String hum = String((int) h); 
    if (client.connect(server, 80)) { 
    if (client.connected()) { 
      Serial.println("conectado"); 

```

发出 HTTP 请求：

```js
      client.println("GET /datalogger1.php?temp=" + temp + "&hum=" + hum + " HTTP/1.1"); 
      client.println("Host: 192.168.1.108"); 
      client.println("Connection: close"); 
      client.println(); 
    }  
    else { 
      // If you didn't get a connection to the server 
      Serial.println("fallo la conexion"); 
    } 

```

这些行定义了客户端实例如何读取响应：

```js
    while (client.connected()) { 
      while (client.available()) { 
      char c = client.read(); 
      Serial.print(c); 
      } 
    } 

```

如果服务器断开连接，停止客户端：

```js
    if (!client.connected()) { 
      Serial.println(); 
      Serial.println("desconectado."); 
      client.stop(); 
    } 
  } 

```

每秒重复一次：

```js
  delay(5000); 
} 

```

在这里我们可以看到我们使用的硬件：

![从 Arduino 和以太网盾发送数据到 Web 服务器](img/B05170_06_18.jpg)

# 带有 MySQL 的数据记录器

在接下来的部分，我们将构建一个数据记录器，它将记录服务器中的温度和湿度数据，以便我们随时可以获取数据并在网页中显示。

## 编程脚本软件

在下面的代码中，我们有一个将与 Arduino 板通信的脚本，并且它已安装在服务器上。

您现在可以将代码复制到名为`datalogger1.php`的文件中，或者只需从本项目的文件夹中获取完整的代码：

```js
<?php 
if (isset($_GET["temp"]) && isset($_GET["hum"])) { 
$temperature = intval($_GET["temp"]); 
$humidity = intval($_GET["hum"]); 
$con=mysql_connect("localhost","root","ruben","arduinobd"); 
mysql_select_db('arduinobd',$con); 
      if(mysql_query("INSERT INTO measurements (temperature, humidity) VALUES ('$temperature', '$humidity');")){ 
        echo "Data were saved"; 
      } 
      else { 
      echo "Fail the recorded data"; 
      } 
mysql_close($con); 
} 
?> 

```

## 测试连接

安装了脚本文件后，我们需要在您的计算机上打开一个网络浏览器，并输入您的树莓派的 IP 地址 `Raspberry Pi/datalogger1.php?temp=70&hum=100`，链接看起来像 **(http://192.168.1.108/datalogger1.php?temp=70&hum=100)**：

![测试连接](img/B05170_06_19.jpg)

以下截图显示了保存在数据库中的数据的结果：

![测试连接](img/B05170_06_20.jpg)

以下截图显示了数据表格：

![测试连接](img/B05170_06_21.jpg)

# 从数据库查询数据

记录数据并进行一些查询以在网页中显示数据非常重要。

## 脚本软件

这里有我们用来在页面中显示数据的脚本：

您现在可以将代码复制到名为`query1.php`的文件中，或者只需从本项目的文件夹中获取完整的代码：

```js
<!DOCTYPE html> 
  <html> 
    <body> 
<h1>Clik on the buttons to get Data from  MySQL</h1> 
<form action="query1.php" method="get"> 
<input type="submit" value="Get all Data">  
</form> 
</br> 

<form action="query2.php" method="get"> 
<input type="submit"value="Humidity <= 15"> 
</form>  
</br> 

<form action="query3.php" method="get"> 
<input type="submit" value="Temperature <=25">  
</form> 
</br> 
<?php 

$con=mysql_connect("localhost","root","ruben","arduinobd"); 
mysql_select_db('arduinobd',$con); 
$result = mysql_query("SELECT * FROM measurements"); 
echo "<table border='1'> 
<tr> 
<th>Measurements</th> 
<th>Temperature (°C)</th> 
<th>Humidity (%)</th> 
</tr>"; 
while($row = mysql_fetch_array($result)) { 
  echo "<tr>"; 
  echo "<td>" . $row['id'] . "</td>"; 
  echo "<td>" . $row['temperature'] . "</td>"; 
  echo "<td>" . $row['humidity'] . "</td>"; 
  echo "</tr>"; 
} 
echo "</table>"; 
mysql_close($con); 
?> 
</body> 
</html> 

```

在以下截图中，我们有数据：

![脚本软件](img/B05170_06_22.jpg)

## 特定数据的脚本以显示

在接下来的几行中，我们可以看到我们可以进行一些 SQL 查询，以获取特定数值的信息，并从温度和湿度中获取数值：

```js
<?php 
$con=mysql_connect("localhost","root","ruben","arduinobd"); 
mysql_select_db('arduinobd',$con); 
$result = mysql_query("SELECT * FROM measurements where humidity <= 15 order by id"); 
echo "<table border='1'> 
<tr> 
<th>Measurements</th> 
<th>Temperature (°C)</th> 
<th>Humidity (%)</th> 
</tr>"; 
while($row = mysql_fetch_array($result)) { 
  echo "<tr>"; 
  echo "<td>" . $row['id'] . "</td>"; 
  echo "<td>" . $row['temperature'] . "</td>"; 
  echo "<td>" . $row['humidity'] . "</td>"; 
  echo "</tr>"; 
} 
echo "</table>"; 
mysql_close($con); 
?> 

```

## 查询记录温度

在本节中，我们将创建一个查询以获取温度测量值。我们将服务器引用称为`localhost`，在本例中是树莓派零设备，用户和数据库的名称：

```js
<?php 
$con=mysql_connect("localhost","root","ruben","arduinobd"); 
mysql_select_db('arduinobd',$con); 
$result = mysql_query("SELECT * FROM measurements where temperature <= 25 order by id"); 
echo "<table border='1'> 
<tr> 
<th>Measurements</th> 
<th>Temperature (°C)</th> 
<th>Humidity (%)</th> 
</tr>"; 
while($row = mysql_fetch_array($result)) { 
  echo "<tr>"; 
  echo "<td>" . $row['id'] . "</td>"; 
  echo "<td>" . $row['temperature'] . "</td>"; 
  echo "<td>" . $row['humidity'] . "</td>"; 
  echo "</tr>"; 
} 
echo "</table>"; 
mysql_close($con); 
?> 

```

查询结果显示在以下截图中：

![查询记录温度](img/B05170_06_23.jpg)

# 控制和调光 LED

在本节中，我们将讨论一个可以应用于家庭自动化的项目。我们将调暗直流 LED，这可以应用于房子里的灯。LED 将改变亮度，并且我们将 LED 连接到树莓派的**GPIO18**，并串联一个*330*欧姆的电阻。

## 软件要求

首先，我们需要安装`pigpio`包。在终端中，键入以下内容：

```js
**wget abyz.co.uk/rpi/pigpio/pigpio.zip**

```

然后解压包：

```js
**unzip pigpio.zip**

```

之后，使用以下内容导航到解压后的文件夹：

```js
**cd PIGPIO**

```

键入以下内容执行命令：

```js
**Make**

```

最后安装文件：

```js
**sudo make install**

```

## 测试 LED

在本节中，我们将使用**Node.js**脚本测试传感器：

```js
var Gpio = require('pigpio').Gpio; 

// Create led instance 
var led = new Gpio(18, {mode: Gpio.OUTPUT}); 
var dutyCycle = 0; 
// Go from 0 to maximum brightness 
setInterval(function () { 
  led.pwmWrite(dutyCycle); 
  dutyCycle += 5; 
  if (dutyCycle > 255) { 
    dutyCycle = 0; 
  } 
}, 20); 

```

我们现在可以测试这段代码，使用 Pi 上的终端导航到此项目的文件夹，并键入以下内容：

```js
**sudo npm install pigpio**

```

这将安装所需的`node.js`模块来控制 LED。然后，键入以下内容：

```js
**sudo node led_test.js**

```

这是最终结果：

![测试 LED](img/B05170_06_24.jpg)

## 从界面控制 LED

在本节中，我们将从网页控制 LED。为此，我们将使用 HTML 与用户进行界面交互，使用`node.js`。

让我们看一下以下代码中包含的 Node.js 文件：

```js
// Modules 
var Gpio = require('pigpio').Gpio; 
var express = require('express'); 
// Express app 
var app = express(); 

// Use public directory 
app.use(express.static('public')); 
// Create led instance 
var led = new Gpio(18, {mode: Gpio.OUTPUT}); 

// Routes 
app.get('/', function (req, res) { 

  res.sendfile(__dirname + '/public/interface.html'); 

}); 
app.get('/set', function (req, res) { 

  // Set LED 
  dutyCycle = req.query.dutyCycle; 
  led.pwmWrite(dutyCycle); 

  // Answer 
  answer = { 
    dutyCycle: dutyCycle 
  }; 
  res.json(answer); 

}); 

// Start server 
app.listen(3000, function () { 
  console.log('Raspberry Pi Zero LED control'); 
}); 

```

现在终于是时候测试我们的应用程序了！首先，从本书的存储库中获取所有代码，并像以前一样导航到项目的文件夹。然后，使用以下命令安装`express`：

```js
**sudo npm install express**

```

完成后，使用以下命令启动服务器：

```js
**sudo node led_control.js**

```

您现在可以测试项目，打开计算机上的网络浏览器，并输入链接- `http://（树莓派 PI）/set?dutyCycle=20`，我们可以看到 LED 随数值变化。

然后用`http://192.168.1.108:3000`打开您的网络浏览器，您应该在一个基本的网页上看到控制：

![从界面控制 LED](img/B05170_06_25.jpg)

# 控制直流电机的速度

在房子里通常会有窗户或车库门。我们需要自动化这些类型的设备，以便我们可以使用直流电机移动这些物体。在本节中，我们将看到如何将直流电机连接到树莓派。为此，我们将使用 L293D 电路来控制电机。

首先，我们将看到如何将电机连接到我们的树莓派 Zero 板上。在下图中，我们可以看到 LD293 芯片的引脚：

![控制直流电机的速度](img/B05170_06_26.jpg)

基本上，我们需要连接电路的组件，如下所示：

+   树莓派的**GPIO14**连接到引脚**1A**

+   树莓派的**GPIO15**连接到引脚**2A**

+   树莓派的**GPIO18**连接到引脚**1**，**2EN**

+   **DC**电机连接到引脚**1Y**和**2Y**

+   树莓派的**5V**连接到**VCC1**

+   树莓派的**GND**连接到**GND**

+   适配器调节器连接到**VCC2**和**GND**

以下图显示了结果：

![控制直流电机的速度](img/B05170_06_27.jpg)

我们现在将测试直流电机的速度从 0 到最高速度：

```js
// Modules 
var Gpio = require('pigpio').Gpio; 
// Create motor instance 
var motorSpeed = new Gpio(18, {mode: Gpio.OUTPUT}); 
var motorDirectionOne = new Gpio(14, {mode: Gpio.OUTPUT}); 
var motorDirectionTwo = new Gpio(15, {mode: Gpio.OUTPUT}) 

// Init motor direction 
motorDirectionOne.digitalWrite(0); 
motorDirectionTwo.digitalWrite(1); 
var dutyCycle = 0; 

// Go from 0 to maximum brightness 
setInterval(function () { 
  motorSpeed.pwmWrite(dutyCycle); 
  dutyCycle += 5; 
  if (dutyCycle > 255) { 
    dutyCycle = 0; 
  } 
}, 20); 

```

在这里，我们有这个应用程序的代码，可以使用网页界面来控制直流电机：

```js
// Modules 
var Gpio = require('pigpio').Gpio; 
var express = require('express'); 

// Express app 
var app = express(); 
// Use public directory 
app.use(express.static('public')); 

// Create led instance 
var motorSpeed = new Gpio(18, {mode: Gpio.OUTPUT}); 
var motorDirectionOne = new Gpio(14, {mode: Gpio.OUTPUT}); 
var motorDirectionTwo = new Gpio(15, {mode: Gpio.OUTPUT}); 

// Routes 
app.get('/', function (req, res) { 

  res.sendfile(__dirname + '/public/interface.html'); 

}); 

app.get('/set', function (req, res) { 
  // Set motor speed 
  speed = req.query.speed; 
  motorSpeed.pwmWrite(speed); 

  // Set motor direction 
  motorDirectionOne.digitalWrite(0); 
  motorDirectionTwo.digitalWrite(1); 

// Answer 
  answer = { 
    speed: speed 
  }; 
  res.json(answer); 

}); 

// Start server 
app.listen(3000, function () { 
  console.log('Raspberry Pi Zero Motor control started!'); 
}); 

```

我们在以下代码中看到用户界面：

```js
$( document ).ready(function() { 

  $( "#motor-speed" ).mouseup(function() { 

    // Get value 
    var speed = $('#motor-speed').val(); 

    // Set new value 
    $.get('/set?speed=' + speed); 

  }); 

}); 

<!DOCTYPE html> 
<html> 

<head> 
  <script src="https://code.jquery.com/jquery-2.2.4.min.js"></script> 
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css"> 
  <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"></script> 
  <script src="js/interface.js"></script> 
  <link rel="stylesheet" href="css/style.css"> 
  <meta name="viewport" content="width=device-width, initial-scale=1"> 
</head> 
<body> 

<div id="container"> 

  <h3>Motor Control</h3> 

  <div class='row'> 

    <div class='col-md-4'></div> 
    <div class='col-md-4 text-center'> 
     <input id="motor-speed" type="range" value="0" min="0" max="255" step="1"> 
    </div> 
    <div class='col-md-4'></div> 

  </div> 

</div> 

</body> 
</html> 

```

要测试应用程序，您需要在计算机上打开网络浏览器，链接为`http://192.168.1.108:3000`，然后您需要替换您的 Pi 的 IP 地址。这是我们的界面：

![控制直流电机的速度](img/B05170_06_28.jpg)

# 使用电气电路控制灯

在接下来的章节中，您将找到更多控制房屋其他设备的项目想法。

## 电器设备

在房子里，我们有电器设备，例如灯，洗衣机，加热器和其他我们只需要打开或关闭的设备。在本节中，我们将学习如何使用电气电路来控制连接到树莓派 Zero 的灯，以及如何使用光耦合器（如 MOC3011）和三角形。以下图显示了应用程序的电路：

![电器设备](img/B05170_06_29.jpg)

这里我们有连接到 Raspberry Pi Zero 的最终项目：

![电器设备](img/B05170_06_30.jpg)

这里有用于控制设备的 JavaScript 代码：

```js
// Modules 
var express = require('express'); 

// Express app 
var app = express(); 

// Pin 
var lampPin = 12; 

// Use public directory 
app.use(express.static('public')); 

// Routes 
app.get('/', function (req, res) { 

  res.sendfile(__dirname + '/public/interface.html'); 

}); 

app.get('/on', function (req, res) { 

  piREST.digitalWrite(lampPin, 1); 

  // Answer 
  answer = { 
    status: 1 
  }; 
  res.json(answer); 

}); 

app.get('/off', function (req, res) { 

  piREST.digitalWrite(lampPin, 0); 

  // Answer 
  answer = { 
    status: 0 
  }; 
  res.json(answer); 

}); 

// aREST 
var piREST = require('pi-arest')(app); 
piREST.set_id('34f5eQ'); 
piREST.set_name('my_rpi_zero'); 

// Start server 
app.listen(3000, function () { 
  console.log('Raspberry Pi Zero lamp control started!'); 
}); 

```

我们需要一个可以通过 HTML 语言的网页控制灯的界面：

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

<body> 

<div id="container"> 

  <h3>Lamp Control</h3> 

  <div class='row'> 

    <div class='col-md-4'></div> 
    <div class='col-md-2'> 
      <button id='on' class='btn btn-block btn-primary'>On</button> 
    </div> 
    <div class='col-md-2'> 
      <button id='off' class='btn btn-block btn-warning'>Off</button> 
    </div> 
    <div class='col-md-4'></div> 

  </div> 

</div> 

</body> 
</html> 

```

进入网络浏览器后，我们将看到以下界面：

![电器设备](img/B05170_06_31.jpg)

# 其他家用电器

在这一部分，我们将展示其他应用程序，您可以考虑创建和控制，然后在家里或不同的区域使用它们。

## 控制门锁

在这一部分，我们将看到其他可以从界面控制并连接到 Raspberry Pi 的家用电器。在家里，我们可以通过 Web 界面控制门锁：

![控制门锁](img/B05170_06_32.jpg)

## 控制浇水设备

我们可以控制的另一个家用电器是连接到 Raspberry Pi 的塑料水电磁阀-12V 的浇水设备：

![控制浇水设备](img/B05170_06_33.jpg)

通过这个项目，我们可以制作一个自动浇水系统，添加一个湿度传感器，并设置花园植物的浇水时间。

# 从任何地方远程访问您的 Raspberry Pi Zero

如果我们想要从网络外部访问我们的 Raspberry Pi，我们需要执行以下操作：

+   检查我们的调制解调器是否有公共 IP 地址

+   调查我们将在浏览器中使用的地址

+   在我们的浏览器中输入[`whatismyipaddress.com/`](http://whatismyipaddress.com/)![从任何地方远程访问您的 Raspberry Pi Zero](img/B05170_06_34.jpg)

ISP 提供的 IP 通常是动态 IP，会随时间变化。在我们的情况下，我们需要具有不时更改的静态地址。

## 如何访问我们的调制解调器进行配置

通过 IP 地址（网关）访问我们的调制解调器，并转到端口寻址部分。配置指向我们的 Web 服务器的端口*80*（输入我们帐户的 IP 地址），此 IP 地址是我们系统的 DHCP 服务器自动分配的 IP 地址。

这里有一些可以从调制解调器-路由器转发的端口：

![如何访问我们的调制解调器进行配置](img/B05170_06_35.jpg)

要获取网关 IP 地址，请输入`ipconfig`命令，您需要具有管理员权限。之后，在您的`router.1`的网络浏览器中输入`http://gatewayip_addres`：

![如何访问我们的调制解调器进行配置](img/B05170_06_36.jpg)

这是您在 Linksys 路由器上看到的示例，您的界面可能不同：

![如何访问我们的调制解调器进行配置](img/B05170_06_37.jpg)

要打开一个端口，我们需要配置我们的路由器以允许从外部进入，因此我们需要在我们的路由器中给予权限：

![如何访问我们的调制解调器进行配置](img/B05170_06_38.jpg)

这个屏幕截图显示了最终结果，如何打开端口号 3000，以及应用程序节点的名称：

![如何访问我们的调制解调器进行配置](img/B05170_06_39_Updated.jpg)

## 配置动态 DNS

我们需要配置域名服务，这样我们就可以通过输入我们域名的名称来访问我们的 Web 服务器（学习网页的 IP 地址非常困难）。这就是为什么创建了**域名服务器（DNS)**。请按照下一节创建域名。

您可能希望从家外访问您的物联网控制面板。在这种情况下，您的 Web 服务器将需要成为互联网上的主机。

这并不是一件简单的事情，因为它在您家中的路由器后面。您的 ISP 通常不会给您一个静态的公共 IP 地址，因为大多数用户只是访问网络，而不是提供网页服务。

因此，您的路由器的公共一侧会被分配一个可能会不时更改的 IP 地址。如果您浏览`<whatsmyip...>`，您将看到您当前的公共 IP 是什么。

明天可能会不同。要设置外部访问，您可以选择以下两种方法之一。如果您想模拟具有静态 IP，可以使用动态 DNS 等服务。如果您只是想“尝试”外部访问，可以在路由器上打开一个端口

动态 DNS 的好处：

+   一种解决方案是安装一个客户端，允许公共 IP 固定。客户端功能（安装在计算机上的软件）与网站[www.no-ip.org](http://www.no-ip.com)保持通信。

+   当我们的调制解调器的 IP 地址发生变化时，客户端会接受该 IP 变化。

+   这样我们的域名就可以始终指向我们的公共 IP 地址。安装的软件称为：No-IP DUC。

## 在 No-ip.org 创建一个账户

在下面的截图中，我们可以看到增强动态 DNS 设置：

![在 No-ip.org 创建一个账户](img/B05170_06_40.jpg)

# 控制灯光和测量电流消耗

现在，在本节中，我们将解释如何在灯开启或关闭时控制和监控您的电流消耗。通过 Web 页面使用您的 Arduino Wi-Fi shield，我们将监控此变量。当灯关闭时，它看起来如下：

![控制灯光和测量电流消耗](img/B05170_06_41.jpg)

当灯开启时，它看起来如下：

![控制灯光和测量电流消耗](img/B05170_06_42.jpg)

现在，您可以将代码复制到名为`Controlling_lights_Current_Consumption.ino`的文件中，或者只需从本书的文件夹中获取完整的代码。

定义监控和控制的变量和函数：

```js
#define NUMBER_VARIABLES 2 
#define NUMBER_FUNCTIONS 1 

```

导入库以使用：

```js
#include <Adafruit_CC3000.h> 
#include <SPI.h> 
#include <CC3000_MDNS.h> 
#include <aREST.h> 

```

配置继电器以激活：

```js
const int relay_pin = 8; 

```

计算电流的变量：

```js
float amplitude_current; 
float effective_value; 
float effective_voltage = 110; 
float effective_power; 
float zero_sensor; 

```

我们定义用于配置模块的引脚：

```js
#define ADAFRUIT_CC3000_IRQ   3 
#define ADAFRUIT_CC3000_VBAT  5 
#define ADAFRUIT_CC3000_CS    10 
Adafruit_CC3000 cc3000 = Adafruit_CC3000(ADAFRUIT_CC3000_CS,  
ADAFRUIT_CC3000_IRQ, ADAFRUIT_CC3000_VBAT); 

```

我们创建实例：

```js
aREST rest = aREST(); 

```

我们定义您网络的 SSID 和密码：

```js
#define WLAN_SSID       "xxxxxxxx" 
#define WLAN_PASS       "xxxxxxxx" 
#define WLAN_SECURITY   WLAN_SEC_WPA2 

```

我们配置服务器的端口：

```js
#define LISTEN_PORT 80 

```

服务器的实例：

```js
Adafruit_CC3000_Server restServer(LISTEN_PORT); 
MDNSResponder mdns; 

```

使用的变量：

```js
int power; 
int light; 

```

发布使用的变量：

```js
void setup(void) 
{   
  Serial.begin(115200); 
  rest.variable("light",&light); 
  rest.variable("power",&power); 

```

设置继电器引脚为输出：

```js
pinMode(relay_pin,OUTPUT); 

```

校准电流传感器：

```js
  zero_sensor = getSensorValue(A1); 

```

我们声明设备的 id 和名称：

```js
  rest.set_id("001"); 
  rest.set_name("control"); 

```

在这部分，我们检查设备是否已连接：

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

在这部分中，我们定义了通信请求：

```js
  if (!mdns.begin("arduino", cc3000)) { 
    while(1);  
  } 
  displayConnectionDetails(); 

```

让我们启动服务器：

```js
  restServer.begin(); 
  Serial.println(F("Listening for connections...")); 
} 

```

我们读取传感器：

```js
void loop() { 

  float sensor_reading = analogRead(A0); 
  light = (int)(sensor_reading/1024*100); 
  float sensor_value = getSensorValue(A1); 

```

我们进行电流计算并获取信号：

```js
  amplitude_current = (float)(sensor_value-zero_sensor)/1024*5/185*1000000; 
  effective_value = amplitude_current/1.414; 
  effective_power = abs(effective_value*effective_voltage/1000); 
  power = (int)effective_power; 
  mdns.update(); 

```

我们定义传入请求：

```js
Adafruit_CC3000_ClientRef client = restServer.available(); 
  rest.handle(client); 
 } 

```

我们显示 IP 地址配置：

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

电流传感器的功能，计算特定测量的平均值并返回电流计算：

```js
float getSensorValue(int pin) 
{ 
  int sensorValue; 
  float avgSensor = 0; 
  int nb_measurements = 100; 
  for (int i = 0; i < nb_measurements; i++) { 
    sensorValue = analogRead(pin); 
    avgSensor = avgSensor + float(sensorValue); 
  }      
  avgSensor = avgSensor/float(nb_measurements); 
  return avgSensor; 
} 

```

## 构建控制和监控界面

这里有用于显示控制灯光和监控传感器电流的界面的代码：

### 为 Node.js 安装 Jade

在这个项目中使用 Jade 界面很重要。为此，我们只需输入以下命令：

```js
**npm install arest express jade** 

```

如果需要，我们可以在系统需要更新时输入以下命令：

```js
**npm install pug**

```

## 用于控制和监控的界面

首先，我们定义页面的标题并添加 HTML 标签：

```js
doctype 
html 
  head 
    title Control and monitoring 

```

我们为 jQuery 和 Bootstrap 的功能定义链接：

```js
link(rel='stylesheet', href='/css/interface.css') 
    link(rel='stylesheet',  
      href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.0/css/bootstrap.min.css") 
    script(src="https://code.jquery.com/jquery-2.1.1.min.js") 
    script(src="/js/interface.js") 

```

我们在网页上显示控制按钮：

```js
  body 
    .container 
      h1 Controlling lights 
      .row.voffset 
        .col-md-6 
          button.btn.btn-block.btn-lg.btn-primary#1 On 
        .col-md-6 
          button.btn.btn-block.btn-lg.btn-danger#2 Off 
      .row 

```

显示功率和光照水平：

```js
        .col-md-4 
          h3#powerDisplay Power: 
        .col-md-4 
          h3#lightDisplay Light level:  
        .col-md-4 
          h3#status Offline 

```

现在我们将运行应用程序，如下截图所示。服务器在端口 3000 上打开，当它开始向板发送请求时，在 Web 浏览器中键入地址：`http://localhost:3000`。它显示了带有两个按钮的网页，设备已连接并在线：

![用于控制和监控的界面](img/B05170_06_43.jpg)

单击蓝色**开**按钮以激活板上的灯，几秒钟后我们可以看到功率增加：

![用于控制和监控的界面](img/B05170_06_44.jpg)

单击红色**关**按钮，几秒钟后功率下降直到*0 W*，这意味着一切都运行正常：

![用于控制和监控的界面](img/B05170_06_45.jpg)

# 在连接设备和传感器上控制和监控 Arduino、Wi-Fi 和以太网 shield

在前几节中，我们看到如何使用在 Windows 计算机上运行的`node.js`从网页控制和监视我们的 Arduino 板。在本节中，我们将使用我们神奇的树莓派 Zero，在其上安装了 Node.js，并在板上运行 JavaScript 应用程序。

我已经看到了使用树莓派 Zero 而不是使用个人计算机安装为 Web 服务器的潜力，通过这种经验制作这些项目，我想说应用程序在树莓派 Zero 上运行更有效。

我们将看到如何在单个仪表板中使用不同设备控制多个设备，例如以下设备：

+   Wi-Fi 盾

+   ESP8266 模块

+   以太网盾

## 构建控制和监视设备的代码，从单一界面进行监控

现在您可以将代码复制到名为`app.js`的文件中，或者只需从此项目的文件夹中获取完整的代码。

配置系统中连接的设备的输出：

```js
$.getq('queue', '/lamp_control/mode/8/o'); 
$.getq('queue', '/lamp_control2/mode/5/o'); 

```

启动控制功能：

```js
$(document).ready(function() { 

```

我们使用`aREST` API 进行`ON`的`GET`请求：

```js
// Function to control lamp Ethernet Shield 
  $("#1").click(function() { 
    $.getq('queue', '/lamp_control/digital/8/1'); 
  }); 

```

我们使用`ARESt` API 进行`OFF`的`GET`请求：

```js
  $("#2").click(function() { 
    $.getq('queue', '/lamp_control/digital/8/0'); 
  }); 

```

我们对连接的 ESP8266 设备进行相同的操作`ON`：

```js
//Function to control lamp ESP8266 
  $("#3").click(function() { 
    $.getq('queue', '/lamp_control2/digital/5/0'); 
  }); 

```

我们对连接的 ESP8266 设备进行相同的操作`OFF`：

```js
$("#4").click(function() { 
    $.getq('queue', '/lamp_control2/digital/5/1'); 
  }); 

```

从传感器温度和湿度获取数据：

```js
  function refresh_dht() { 
        $.getq('queue', '/sensor/temperature', function(data) { 
      $('#temperature').html("Temperature: " + data.temperature + " C"); 
    }); 

  $.getq('queue', '/sensor2/temperature2', function(data) { 
      $('#temperature2').html("Temperature: " + data.temperature2 + " C"); 
    }); 

  $.getq('queue', '/sensor/humidity', function(data) { 
      $('#humidity').html("Humidity: " + data.humidity + " %"); 
    }); 
         $.getq('queue', '/sensor2/humidity2', function(data) { 
      $('#humidity2').html("Humidity: " + data.humidity2 + " %"); 
}); 
  } 

```

此代码每 10000 秒刷新页面一次：

```js
refresh_dht(); 
setInterval(refresh_dht, 10000); 
}); 

```

## 添加要监视和控制的设备

我可以看到系统非常稳定；我们需要添加将从树莓派 Zero 监视的设备，使用以下 JavaScript 片段中的应用程序。

我们创建 express 模块和必要的库：

```js
var express = require('express'); 
var app = express(); 

```

我们定义将要打开的端口：

```js
var port = 3000; 

```

我们为 HTML 网页配置 Jade 引擎：

```js
app.set('view engine', 'jade'); 

```

我们创建公共目录以便访问：

```js
app.use(express.static(__dirname + '/public')); 

```

执行服务器指令的界面：

```js
app.get('/', function(req, res){ 
res.render('interface'); 
}); 

```

我们使用 rest 请求声明逮捕文件：

```js
var rest = require("arest")(app); 

```

此代码定义将被控制和监视的设备，我们可以添加想要的设备：

```js
rest.addDevice('http','192.168.1.108'); 
rest.addDevice('http','192.168.1.105'); 
rest.addDevice('http','192.168.1.107'); 
rest.addDevice('http','192.168.1.110'); 

```

我们在端口 3000 上设置服务器并监听 Web 浏览器客户端：

```js
app.listen(port); 
console.log("Listening on port " + port); 

```

如果一切都配置完美，我们通过输入以下命令来测试应用程序：

```js
**sudo npm install arest express jade**

```

这将安装 Jade 平台并识别来自树莓派 Zero 的`aREST` API。

如果需要更新某些内容，请输入以下命令：

```js
**sudo npm install pug**

```

要更新`arrest express`，请输入以下命令：

```js
**sudo npm install pi-arest express** 

```

安装此软件包非常重要，以包括逮捕 API：

```js
**sudo npm install arest --unsafe-perm**

```

要运行应用程序，请转到应用程序所在的文件夹，并输入以下命令：

```js
**node app.js**

```

在下面的屏幕截图中，我们看到服务器正在打开端口 3000：

![添加要监视和控制的设备](img/B05170_06_46.jpg)

最后的测试中，我们需要在您喜欢的网络浏览器中输入树莓派此刻的 IP 地址：`http://IP_Address_of_Raspberry_Pi_Zero/port`。

在下面的屏幕截图中，我们可以看到来自树莓派 Zero 的控制和监视数据仪表板，发布在单个网页上的不同设备上，这是一件有趣的事情，可以远程系统和控制面板：

![添加要监视和控制的设备](img/B05170_06_47.jpg)

最后，我们通过在单个数据仪表板中使用不同设备来展示控制和监视系统；我们得出结论，物联网可以在一个网页中拥有多个设备。

# 总结

在本章中，您学习了如何使用树莓派 Zero 与 Arduino 和之前章节中介绍的技术集成和构建监控仪表板。本章为您提供了基础知识和必要工具，可以帮助您创建自己的物联网系统，用于不同应用和领域，通过应用所有工具、Web 服务器、数据库服务器、连接的设备，并设置路由器来控制您的树莓派，从世界各地的任何地方。

在下一章中，您将为物联网构建非常好的设备；您将学习如何制作不同的迷你家庭自动化项目。
