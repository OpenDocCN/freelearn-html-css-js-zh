# 第五章。将网络摄像头添加到监控安全系统

在之前的章节中，我们谈到了诸如连接到 Arduino 的传感器和从树莓派 Zero 监控，使用跨设备的网络，家庭安全项目的重要性以及家居自动化来监视现实世界中发生的事情。为此，我们为本章提出了一个建议。

在本章中，我们将配置我们的树莓派 Zero 来监视网络摄像头，并安装 TTL 串行摄像头以与 Arduino 板交互；我们将通过以下主题实现这一目标：

+   Arduino 和树莓派之间的交互

+   从树莓派 Zero 控制连接到 Arduino 的输出

+   将 TTL 串行摄像头连接到 Arduino 并将图片保存到 Micro SD

+   使用串行 TTL 摄像头检测运动

+   从树莓派控制快照

+   从网页控制您的摄像头

+   在网络中监控您的 USB 摄像头以确保安全

# Arduino 和树莓派之间的交互

在本章中，我们将看看树莓派如何作为终端计算机来编程，不仅可以将设备作为服务器并部署页面或应用程序，还可以拥有用于编程 Arduino 板的集成开发环境。为此，我们需要将树莓派连接到 Arduino，以便它们可以相互通信。

这里有一些树莓派具有的接口，所有这些都包括在设备中：I2C 协议，SPI 通信，USB 端口和串行**UART**端口。在这种情况下，我们将使用 USB 端口在 Arduino 和树莓派之间进行通信。

以下是配置 Arduino 和树莓派相互交互的步骤：

1.  为树莓派安装 Arduino IDE

1.  用 PuTTY 打开您的终端并检查树莓派的 IP 地址

1.  执行远程访问，并输入 IP 地址

1.  在图形界面中打开 Arduino IDE

## 在 Raspbian 中安装 Arduino IDE

输入以下命令在树莓派上安装 Arduino IDE：

```js
**sudo apt-get install arduino**

```

## 远程访问树莓派

在本节中，我们将查看访问远程桌面的屏幕，以执行安装在 Raspian 操作系统中的 Arduino IDE：一旦屏幕弹出，输入您的用户名和密码：

![远程访问树莓派](img/B05170_05_01.jpg)

## 在图形界面中执行 Arduino

现在我们有了主屏幕，我们转到**编程**菜单，如果我们看到进入 Arduino IDE 的图标，那么一切都已安装。点击**Arduino IDE**的图标：

![在图形界面中执行 Arduino](img/B05170_05_02.jpg)

# Raspian 中的 Arduino 界面

这里有 Arduino IDE 的界面，与我们在计算机上拥有的界面类似。从在树莓派上运行的 Arduino IDE 中，我们可以在两个板之间进行交互：

![Raspian 中的 Arduino 界面](img/B05170_05_03.jpg)

## 准备界面

我们需要验证我们选择了正确的板；在这种情况下，我们使用的是 Arduino UNO。在下一个窗口中选择板：

![准备界面](img/B05170_05_04.jpg)

## 选择串行端口

在我们选择要使用的板之后，我们需要验证并选择将与我们的 Arduino 通信的端口，该端口连接到树莓派的 USB 端口上；我们需要选择端口名称：`/dev/ttyACM0`：

![选择串行端口](img/B05170_05_05.jpg)

## 从图形界面下载草图

我们需要做的主要事情是从我们的树莓派 Zero 与 Arduino 进行通信，并将草图下载到 Arduino 板上，而不使用计算机，以便我们可以将树莓派用于其他目的。

以下截图显示了带有草图的界面：

![从图形界面下载草图](img/B05170_05_06.jpg)

我们应该在界面中下载草图。以下图片显示了连接的 Arduino-树莓派：太酷了！

![从图形界面下载草图](img/image_05_007.jpg)

# 从 Raspberry Pi Zero 控制连接到 Arduino 的输出

现在我们将看一个例子，使用 Python 从 Raspberry Pi 控制输出。

首先，我们需要将 sketch 下载到 Arduino 板上。为了测试我们的通信，我们将展示一个测试 Arduino 和 Raspberry Pi 之间链接的示例：

我们声明以下输出：

```js
int led_output = 13; 

```

我们从程序设置开始：

```js
void setup () { 

```

然后我们提到输出引脚：

```js
pinMode(led_output, OUTPUT); 

```

以 9600 开始串行通信：

```js
Serial.begin(9600); 
} 

```

声明程序的循环：

```js
void loop () {
```

这是我们检查串行端口是否可用的地方：

```js
if (Serial.available() > 0){ 

```

如果找到了某些内容，则读取内容并将内容保存在`c`变量中：

```js
char c = Serial.read(); 

```

如果读取了标记为高的字母`H`：

```js
      if (c == 'H'){ 

```

输出将打开连接到引脚**13**的 LED

```js
digitalWrite(led_output, HIGH); 

```

如果读取了标记为低的字母`L`：

```js
}  
else if (c == 'L'){ 

```

输出将关闭连接到引脚**13**的 LED：

```js
         digitalWrite(led_output, LOW); 
      }  
   } 
} 

```

# 从 Python 控制 Arduino 板

首先，我们需要安装串行库，因为这有助于通过 USB 端口通信与 Arduino 通信。键入以下命令以安装库：

```js
**sudo apt-get install python-serial**

```

以下代码控制 Arduino 来自 Raspberry Pi；现在您可以将代码复制到名为`ControlArduinoFromRasp.py`的文件中，或者只需从此项目的文件夹中获取完整的代码。

以下片段在 Python 中导入串行库：

```js
import serial 

```

我们定义串行通信：

```js
Arduino_UNO = serial.Serial('/dev/ttyACM0', 9600) 

```

打印一条消息以查看通信是否完成：

```js
print("Hello From Arduino!") 

```

在执行此操作时，用户可以输入命令：

```js
while True: 
      command = raw_input('Enter the command ') 
      Arduino_UNO.write(command) 

```

如果是`H`，它会打印消息；如果为假，则显示 LED 关闭：

```js
      if command == 'H': 
            print('LED ON') 
      elif command == 'L': 
            print('LED OFF') 

```

关闭连接：

```js
arduino_UNO.close() 

```

## 硬件连接

这是连接到 Arduino UNO 的 LED，可以使用 Python 从 Raspberry Pi 进行控制：

![硬件连接](img/B05170_05_08.jpg)

# 将 TTL 串行相机连接到 Arduino 并将照片保存到微型 SD 卡

在这里，我们有模式图，显示了微型 SD 卡与 TTL 串行相机的连接；我使用的是 Adafruit 的相机型号。以下链接包含您需要的所有信息，[`www.adafruit.com/product/397`](https://www.adafruit.com/product/397)。在下图中，我们有项目的连接：

![将 TTL 串行相机连接到 Arduino 并将照片保存到微型 SD 卡](img/image_05_009.jpg)

现在我们将解释如何拍照并将其保存到微型 SD 卡；主要想法是将相机连接到 Arduino，这样我们可以在家庭安全监控系统中实现这一点。

以下是用于测试 TTL 相机、拍照并将其保存在微型 SD 卡上的代码。请注意，代码太长，但我将解释执行先前操作所需的最重要和必要的代码。所有这些示例的代码都包含在书中，以获取更完整的信息。

在这里，我们从 TTL 相机导入文件，并与微型 SD 卡通信的文件：

```js
#include <Adafruit_VC0706.h> 
#include <SPI.h> 
#include <SD.h> 

```

我们定义软件库以通过串行通信：

```js
// comment out this line if using Arduino V23 or earlier 
#include <SoftwareSerial.h>        

```

将`chipSelect`定义为引脚 10：

```js
#define chipSelect 10 

```

代码将用于连接引脚：

```js
SoftwareSerial cameraconnection = SoftwareSerial(2, 3); 
Adafruit_VC0706 cam = Adafruit_VC0706(&cameraconnection); 

```

然后我们需要启动相机：

```js
  if (cam.begin()) { 
    Serial.println("Camera Found:"); 
  } else { 
    Serial.println("No camera found?"); 
    return; 
  } 

```

在这里我们定义图像大小：

```js
    cam.setImageSize(VC0706_640x480); 

```

这将显示图像大小：

```js
  uint8_t imgsize = cam.getImageSize(); 
  Serial.print("Image size: "); 

```

代码将拍照：

```js
  if (! cam.takePicture())  
    Serial.println("Failed to snap!"); 
  else  
    Serial.println("Picture taken!"); 

```

创建文件以保存所拍摄的图像：

```js
  char filename[13]; 

```

保存文件的代码：

```js
  strcpy(filename, "IMAGE00.JPG"); 
  for (int i = 0; i < 100; i++) { 
    filename[5] = '0' + i/10; 
    filename[6] = '0' + i%10; 

```

准备微型 SD 卡以保存文件：

```js
if (! SD.exists(filename)) { 
      break; 
    } 
  } 

```

打开拍摄的文件进行预览：

```js
  File imgFile = SD.open(filename, FILE_WRITE); 

```

显示所拍摄图像的大小：

```js
  uint16_t jpglen = cam.frameLength(); 
  Serial.print("Storing "); 
  Serial.print(jpglen, DEC); 
  Serial.print(" byte image."); 

```

从文件中读取数据：

```js
  byte wCount = 0; // For counting # of writes 
  while (jpglen > 0) { 

```

将文件写入内存：

```js
    uint8_t *buffer; 
    uint8_t bytesToRead = min(32, jpglen); 
    buffer = cam.readPicture(bytesToRead); 
    imgFile.write(buffer, bytesToRead); 

```

在屏幕上显示文件：

```js
    if(++wCount >= 64) { 
      Serial.print('.'); 
      wCount = 0; 
    } 

```

显示读取的字节数：

```js
Serial.print(bytesToRead, DEC);  
Serial.println(" bytes"); 
jpglen -= bytesToRead; 
  } 

```

关闭打开的文件：

```js
imgFile.close(); 

```

# 使用串行 TTL 相机检测运动

打开 TTL 相机的运动检测：

```js
  cam.setMotionDetect(true); 

```

验证运动是否激活：

```js
  Serial.print("Motion detection is "); 
  if (cam.getMotionDetect())  
    Serial.println("ON"); 
  else  
    Serial.println("OFF"); 
} 

```

当相机检测到运动时会发生什么：

```js
if (cam.motionDetected()) { 
   Serial.println("Motion!");    
   cam.setMotionDetect(false); 

```

如果检测到运动，则拍照或显示消息：

```js
  if (! cam.takePicture())  
    Serial.println("Failed to snap!"); 
  else  
    Serial.println("Picture taken!"); 

```

# 从 Raspberry Pi 控制快照

现在我们已经看到了如何在 Arduino 和 Raspberry Pi 之间进行通信，以控制板，我们可以将其应用于我们的安全系统项目。我们需要这样做以便从 Raspberry Pi 与控制我们的相机进行通信：

+   将 Arduino 和树莓派连接在一起

+   以 9,600 mbps 创建串行连接

+   调用将拍照并保存在微型 SD 卡中的函数

在树莓派上，我们需要做以下事情：

+   创建调用在 Arduino 中拍照的函数的脚本

+   使用 PuTTY 终端打开并执行脚本

以下部分是应下载到 Arduino 板中的草图：

首先我们开始串行通信：

```js
void setup () { 
    Serial.begin(9600);  
} 

```

这是将告诉摄像头拍照的函数：

```js
void loop () { 
   if (Serial.available() > 0) { 
      char c = Serial.read(); 
      if (c == 'T') {  

      takingpicture(): 

      }  
   } 
} 

```

## 拍照功能的代码

在这里我们讨论定义将提示摄像头拍照的函数的代码。

该函数包含将拍照的代码：

```js
void takingpicture(){ 

```

拍照：

```js
  if (!cam.takePicture())  
    Serial.println("Failed to snap!"); 
  else  
    Serial.println("Picture taken!"); 

```

在这里我们创建保存文件：

```js
  char filename[13]; 

```

在这里我们保存文件：

```js
  strcpy(filename, "IMAGE00.JPG"); 
  for (int i = 0; i < 100; i++) { 
    filename[5] = '0' + i/10; 
    filename[6] = '0' + i%10; 

```

准备微型 SD 卡保存文件：

```js
if (! SD.exists(filename)) { 
      break; 
    } 
  } 

```

打开文件进行预览：

```js
  File imgFile = SD.open(filename, FILE_WRITE); 

```

在保存之前获取文件的大小：

```js
  uint16_t jpglen = cam.frameLength(); 
  Serial.print("Storing "); 
  Serial.print(jpglen, DEC); 
  Serial.print(" byte image."); 

```

从保存的文件中读取数据：

```js
  byte wCount = 0; // For counting # of writes 
  while (jpglen > 0) { 

```

将文件写入内存：

```js
    uint8_t *buffer; 
    uint8_t bytesToRead = min(32, jpglen); 
    buffer = cam.readPicture(bytesToRead); 
    imgFile.write(buffer, bytesToRead); 

```

保存后显示文件：

```js
    if(++wCount >= 64) { 
      Serial.print('.'); 
      wCount = 0; 
    } 

```

显示读取的字节数：

```js
Serial.print(bytesToRead, DEC);  
Serial.println(" bytes"); 
jpglen -= bytesToRead; 
  } 

```

关闭已打开的文件：

```js
imgFile.close(); 
}
```

# 从网页上控制您的摄像头

在这一部分，我们将看看如何从 PHP 的网页上控制我们的摄像头，并在树莓派上运行一个 Web 服务器。我们需要以下内容来运行 PHP 文件和 Web 服务器：

+   在树莓派上运行 Apache 服务器

+   安装 PHP 软件

对于控制的网页，我们需要在以下路径创建我们的 PHP 文件：`/var/www/html`，例如我们需要编辑`index.php`文件，并复制以下行。

以下 HTML 文件包括 PHP：

```js
<!DOCTYPE html> 
<html> 
 <head> 
 <title>Control Camera</title> 
 </head> 
  <body> 

```

在这里我们定义了执行拍照动作的函数：

```js
<form  action="on.php">   
  <button type="submit">Taking the picture</button> 
  </form> 

```

在这里我们定义如果检测到运动要采取的动作：

```js
  <form action="off.php">   
  <button type="submit">Motion</button> 
  </form> 
</body> 
</html> 

```

## 从 PHP 调用 Python 脚本

在这一部分，我们需要从网页调用 Python 脚本并执行包含脚本的文件：

```js
<?php 
$prende= exec('sudo python on.py'); 
header('Location:index.php'); 
?> 

<?php 
$apaga = exec('sudo python motion.py'); 
header('Location:index.php'); 
?> 

```

## Python 脚本的代码

在服务器端，也就是树莓派上，我们有将从网页调用的 Python 脚本：

```js
import serial 
import time 
Arduino_1 = serial.Serial('/dev/ttyACM0',9600) 
Arduino_1.open() 
Command='H' 
if command:    
    Arduino_1.write(command) 
Arduino_1.close() 

import serial 
import time 
Arduino_1 = serial.Serial('/dev/ttyACM0',9600) 
Arduino_1.open() 
Command='L' 
if command:    
    Arduino_1.write(command) 
Arduino_1.close() 

```

如果一切都配置完美，以下页面将出现：在你喜欢的浏览器中，输入你的`PI/index.php`的 IP 地址：

![Python 脚本的代码](img/B05170_05_10.jpg)

# 在网络中监控您的 USB 摄像头安全

在这一部分，我们将创建一个项目，允许我们监控连接到 Arduino YUN 的 USB 摄像头，它具有 USB 端口并包括以太网和 Wi-Fi 通信。因此，它有许多优势。我们将致力于在树莓派和 Arduino YUN 之间建立一个网络，所以主要的想法是从树莓派的网页上监控摄像头。该页面将存储在树莓派上。

## 配置 Arduino YUN

我们将使用支持 UVC 协议的罗技摄像头：

![配置 Arduino YUN](img/B05170_05_11-1.jpg)

现在我们将解释安装我们的摄像头在 Arduino YUN 中的步骤：

+   将板连接到您的 Wi-Fi 路由器

+   验证 Arduino YUN 的 IP 地址

在我们输入 IP 地址后，将出现以下屏幕：

![配置 Arduino YUN](img/B05170_05_12-1.jpg)

我们现在将在命令提示符下发出一系列命令来完成设置：

更新软件包：

```js
**opkg update**

```

安装 UVC 协议：

```js
**opkg install kmod-video-uvc**

```

安装摄像头驱动程序：

```js
**opkg install fswebcam**

```

下载`Mjpgstreamer`：

```js
**wget http://www.custommobileapps.com.au/downloads/mjpgstreamer.Ipk**

```

安装`Mjpgstreamer`：

```js
**opkg install mjpg-streamer.ipk**

```

要手动启动摄像头，使用以下代码：

```js
**mjpg_streamer -i "input_uvc.so -d /dev/video0 -r 640x480 -f 25" -o**
**"output_http.so -p 8080 -w /www/webcam" &**

```

要自动启动摄像头，我们将使用以下代码：

安装`nano`程序：

```js
**opkg install nano**

```

输入以下文件：

```js
**nano /etc/config/mjpg-streamer**

```

使用以下参数配置摄像头：

```js
config mjpg-streamer core   
option enabled    "1"   
option device    "/dev/video0"   
option resolution  "640x480"   
option fps    "30"   
option www    "/www/webcam"   
option port    "8080" 

```

使用以下命令启动服务：

```js
**/etc/init.d/mjpg-streamer enable**
**/etc/init.d/mjpg-streamer stop**
**/etc/init.d/mjpg-streamer start**

```

## 从 MJPG-STREAMER 服务器监控

一旦你访问了 Arduino YUN 的服务器，就在你的网络浏览器中输入你的 Arduino YUN 的 IP 地址`http://Arduino.local:8080`。配置的结果如下截图所示：

![从 MJPG-STREAMER 服务器监控](img/B05170_05_13.jpg)

## 从树莓派监控 USB 摄像头

连接到 Arduino YUN 的摄像头，现在我们可以实时监控树莓派上发布的网页。

为网页提供一个标题：

```js
<html> 
<head> 
<title>Monitoring USB Camera</title> 

```

我们通过输入 Arduino YUN 的 IP 地址来调用摄像头图像：

```js
</head> 
<body> 
<center> 
<img src="http://192.168.1.107:8080/?action=stream"/> 
</center> 
</body> 
</html> 

```

通过在浏览器中输入树莓派的 IP 地址（`http://192.168.1.106/index.html`）访问网页：

![从树莓派监控 USB 摄像头](img/B05170_05_14.jpg)

在下一节中，我们将看如何配置连接的设备和将在网络中进行交互的硬件。

以下图片代表了我们用可以监控的设备创建的网络；例如，我们监控房子的每个房间，将所有设备连接到一个 Wi-Fi 网络，并从树莓派上监控它们：

![从树莓派监控 USB 摄像头](img/image_05_015.jpg)

# 摘要

在本章中，您已经学会了如何配置连接到网络的网络摄像头，并监控物联网安全系统。我们使用 Arduino 板连接安全摄像头，并将连接到网络的树莓派 Zero 用于监控系统。在下一章中，我们将集成我们的系统，将树莓派 Zero 与 Arduino 连接，构建一个完整的系统连接设备并进行监控。
