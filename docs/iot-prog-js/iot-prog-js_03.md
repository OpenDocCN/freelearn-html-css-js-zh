# 第三章 连接传感器-测量真实的事物

本书的目标是建立一个家庭安全系统，通过电子控制系统和传感器控制家用电器，并从仪表板监控它们。首先，我们需要考虑我们的传感器连接到一个可以读取信号并将其传输到网络的终端设备。

对于终端设备，我们将使用 Arduino 板从传感器中获取读数。我们可以看到树莓派没有模拟输入。因此，我们使用 Arduino 板来读取这些信号。

在上一章中，我们讨论了如何将设备连接到树莓派；在本节中，我们将看到如何将传感器与 Arduino 板进行接口，以了解如何从不同应用程序中读取真实信号进行实际测量。本章将涵盖以下主题：

+   使用流量传感器计算水的体积

+   使用传感器测量气体浓度

+   使用传感器测量酒精浓度

+   使用传感器检测火灾

+   为植物测量湿度

+   测量容器中的水位

+   测量温度、湿度和光，并在 LCD 中显示数据

+   使用 PIR 传感器检测运动

+   使用铁磁开关检测门是否打开

+   使用指纹传感器检测谁可以进入房屋

重要的是要考虑到我们需要将我们的系统与现实世界进行通信。由于我们正在建立一个家庭安全系统，我们需要学习如何连接和与一些必要的传感器进行交互，以在我们的系统中使用它们。

在下一节中，我们将介绍您在家居自动化和安全系统中需要读取的数据的传感器。

# 测量流量传感器以计算水的体积

我们需要对家中使用的水进行自动测量。为此项目，我们将使用传感器进行此读数，并使测量自动化。

要完成这个项目，我们需要以下材料：

流水传感器和 Arduino UNO 板：

![测量流量传感器以计算水的体积](img/image_03_001.jpg)

## 硬件连接

现在我们有了流量传感器的连接。我们可以看到它有三个引脚--红色引脚连接到+VCC 5 伏特，黑色引脚连接到 GND，黄色引脚连接到 Arduino 板的引脚 2，如下图所示：

![硬件连接](img/image_03_002.jpg)

## 读取传感器信号

中断用于计算通过水流的脉冲，如下所示：

```js
attachInterrupt(0, count_pulse, RISING); 

```

中断类型为`RISING`，计算从低状态到高状态的脉冲：

```js
**Function for counting pulses:** 

voidcount_pulse() 
{ 
pulse++; 
} 

```

# 使用 Arduino 读取和计数脉冲

在代码的这一部分中，我们解释了它如何使用中断来计算传感器的信号，我们已将其配置为`RISING`，因此它会计算从数字信号零到数字信号一的脉冲：

```js
int pin = 2; 
volatile unsigned int pulse; 
constintpulses_per_litre = 450; 

void setup() 
{ 
Serial.begin(9600); 

pinMode(pin, INPUT); 
attachInterrupt(0, count_pulse, RISING); 
} 

void loop() 
{ 
pulse=0; 
interrupts(); 
delay(1000); 
noInterrupts(); 

Serial.print("Pulses per second: "); 
Serial.println(pulse); 
} 

voidcount_pulse() 
{ 
pulse++; 
} 

```

打开 Arduino 串行监视器，并用嘴对水流传感器吹气。每个循环中每秒脉冲的数量将打印在 Arduino 串行监视器上，如下截图所示：

![使用 Arduino 读取和计数脉冲](img/B05170_03_03-1.jpg)

# 根据计数的脉冲计算水流速

在这部分中，我们测量脉冲并将其转换为水流，步骤如下：

1.  打开新的 Arduino IDE，并复制以下草图。

1.  验证并上传 Arduino 板上的草图。

```js
        int pin = 2; 
        volatile unsigned int pulse; 
        constintpulses_per_litre = 450; 

        void setup() 
        { 
          Serial.begin(9600); 

          pinMode(pin, INPUT); 
          attachInterrupt(0, count_pulse, RISING); 
        } 

```

1.  以下代码将计算从传感器读取的脉冲；我们将每秒计数的脉冲数除以脉冲数：

```js
      void loop() 
      { 
        pulse = 0; 
        interrupts(); 
        delay(1000); 
        noInterrupts(); 

        Serial.print("Pulses per second: "); 
        Serial.println(pulse); 

        Serial.print("Water flow rate: "); 
        Serial.print(pulse * 1000/pulses_per_litre); 
        Serial.println(" milliliters per second"); 
        delay(1000); 
      } 
      void count_pulse() 
      { 
        pulse++; 
      } 

```

1.  打开 Arduino 串行监视器，并用嘴吹气通过水流传感器。每个循环中脉冲的数量和每秒的水流速将打印在 Arduino 串行监视器上，如下截图所示：![基于计数的水流速率计算](img/B05170_03_04-1.jpg)

# 计算水流和水的体积：

现在你可以将代码复制到一个名为`Flow_sensor_measure_volume.ino`的文件中，或者直接从这个项目的文件夹中获取完整的代码。

在这部分中，我们从传感器计算流量和体积：

```js
int pin = 2; 
volatile unsigned int pulse; 
float volume = 0; 
floatflow_rate =0; 
constintpulses_per_litre = 450; 

```

我们设置中断：

```js
void setup() 
{ 
Serial.begin(9600); 
pinMode(pin, INPUT); 
attachInterrupt(0, count_pulse, RISING); 
} 

```

启动中断：

```js
void loop() 
{ 
pulse=0; 
interrupts(); 
delay(1000); 
noInterrupts(); 

```

然后我们显示传感器的流速：

```js
Serial.print("Pulses per second: "); 
Serial.println(pulse); 

flow_rate = pulse * 1000/pulses_per_litre; 

```

我们计算传感器的体积：

```js
Serial.print("Water flow rate: "); 
Serial.print(flow_rate); 
Serial.println(" milliliters per second"); 

volume = volume + flow_rate * 0.1; 

```

我们显示毫升的体积：

```js
Serial.print("Volume: "); 
Serial.print(volume); 
Serial.println(" milliliters"); 
} 

```

计算脉冲的函数如下：

```js
Void count_pulse() 
{ 
  pulse++; 
} 

```

结果可以在下面的截图中看到：

![计算水流和水的体积：](img/B05170_03_05-1.jpg)

## 在 LCD 上显示测量参数

您可以在新建的水表上添加 LCD 屏幕，以显示读数，而不是在 Arduino 串行监视器上显示它们。然后，在将草图上传到 Arduino 后，您可以将水表从计算机上断开连接。

首先，我们定义 LCD 库：

```js
#include <LiquidCrystal.h> 

```

然后我们定义程序中将使用的变量：

```js
int pin = 2; 
volatile unsigned int pulse; 
float volume = 0; 
floatflow_rate = 0; 
constintpulses_per_litre = 450; 

```

我们定义 LCD 引脚：

```js
// initialize the library with the numbers of the interface pins 
LiquidCrystallcd(12, 11, 6, 5, 4, 3); 

```

我们定义传感的中断：

```js
void setup() 
{ 
  Serial.begin(9600); 
  pinMode(pin, INPUT); 
  attachInterrupt(0, count_pulse, RISING); 

```

现在我们在 LCD 上显示消息：

```js
  // set up the LCD's number of columns and rows:  
  lcd.begin(16, 2); 
  // Print a message to the LCD. 
  lcd.print("Welcome..."); 
  delay(1000); 
} 

```

我们现在在主循环中定义中断：

```js
void loop() 
{ 
  pulse = 0; 

  interrupts(); 
  delay(1000); 
  noInterrupts(); 

```

我们在 LCD 上显示数值：

```js
  lcd.setCursor(0, 0); 
  lcd.print("Pulses/s: "); 
  lcd.print(pulse); 

  flow_rate = pulse*1000/pulses_per_litre; 

```

然后我们显示传感器的流速：

```js
  lcd.setCursor(0, 1); 
  lcd.print(flow_rate,2);//display only 2 decimal places 
  lcd.print(" ml"); 

```

我们现在显示体积的值：

```js
  volume = volume + flow_rate * 0.1; 
  lcd.setCursor(8, 1); 
  lcd.print(volume, 2);//display only 2 decimal places 
  lcd.println(" ml "); 
} 

```

然后我们定义计算脉冲的函数：

```js
void count_pulse() 
{ 
 pulse++; 
} 

```

水流的连接如下图所示：

![在 LCD 上显示测量参数](img/image_03_006.jpg)

以下图片显示了 LCD 上的测量结果：

![在 LCD 上显示测量参数](img/image_03_007.jpg)

您可以在 LCD 屏幕上看到一些信息，例如每秒脉冲、水流速和从时间开始的总水量。

# 测量气体浓度

在我们的系统中有一个检测气体的传感器是很重要的，这样我们就可以将其应用在家里，以便检测气体泄漏。现在我们将描述如何连接到 Arduino 板并读取气体浓度。

在这一部分，我们将使用一个气体传感器和甲烷 CH4。在这种情况下，我们将使用一个可以检测 200 到 10000ppm 浓度的 MQ-4 传感器。

该传感器在输出中具有模拟电阻，并可以连接到 ADC；它需要 5 伏的线圈激励。传感器的图像如下所示：

![测量气体浓度](img/B05170_03_08-1.jpg)

我们可以在[`www.sparkfun.com/products/9404`](https://www.sparkfun.com/products/9404)找到 MQ-4 传感器的信息。

![测量气体浓度](img/B05170_03_09-1.jpg)

## 传感器和 Arduino 板的连接

根据前面的图表，我们现在将在下面的图像中看到所做的连接：

![传感器和 Arduino 板的连接](img/B05170_03_10-1.jpg)

打开 Arduino IDE，并复制以下草图：

```js
void setup(){ 
  Serial.begin(9600); 
} 

void loop() 
{ 
  float vol; 
  int sensorValue = analogRead(A0); 
  vol=(float)sensorValue/1024*5.0; 
  Serial.println(vol,1); 
  Serial.print("Concentration of gas= "); 
  Serial.println(sensorValue); 
  delay(2000); 
} 

```

我们在屏幕上看到以下结果：

![传感器和 Arduino 板的连接](img/B05170_03_11-1.jpg)

# 用传感器测量酒精的浓度

在这一部分，我们将构建一个非常酷的项目：您自己的**酒精** **呼吸分析仪**。为此，我们将使用一个简单的 Arduino Uno 板以及一个乙醇气体传感器：

![用传感器测量酒精的浓度](img/B05170_03_12-1.jpg)

以下图表显示了传感器与 Arduino 的连接：

![用传感器测量酒精的浓度](img/B05170_03_13-1.jpg)

现在我们将为项目编写代码。在这里，我们将简单地介绍代码的最重要部分。

现在你可以将代码复制到名为`Sensor_alcohol.ino`的文件中，或者直接从该项目的文件夹中获取完整的代码：

```js
int readings=0; 
void setup(){ 
Serial.begin(9600); 
} 

void loop(){ 
lectura=analogRead(A1); 
Serial.print("Level of alcohol= "); 
Serial.println(readings); 
delay(1000); 
} 

```

当它没有检测到酒精时，我们可以看到 Arduino 读取的数值：

![使用传感器测量酒精含量](img/B05170_03_14-1.jpg)

如果检测到酒精，我们可以看到 Arduino 从模拟读取的数值，如下截图所示：

![使用传感器测量酒精含量](img/B05170_03_15-1.jpg)

# 使用传感器检测火灾

如果我们家中有火灾，及时检测是至关重要的；因此，在下一节中，我们将创建一个使用传感器检测火灾的项目。

在下图中，我们看到了火灾传感器模块：

![使用传感器检测火灾](img/B05170_03_16-1.jpg)

现在你可以将代码复制到名为`Sensor_fire.ino`的文件中，或者直接从该项目的文件夹中获取完整的代码。

我们在程序开始时为程序定义变量：

```js
int ledPin = 13;             
int inputPin= 2; 
int val = 0;                    

```

我们定义输出信号和串行通信：

```js
void setup() { 
pinMode(ledPin, OUTPUT);       
pinMode(inputPin, INPUT);      
Serial.begin(9600); 
} 

```

现在我们显示数字信号的值：

```js
void loop(){ 
val = digitalRead(inputPin); 
Serial.print("val : ");   
Serial.println(val); 
digitalWrite(ledPin, HIGH);  // turn LED ON 

```

然后我们进行比较：如果数值检测到高逻辑状态，它会关闭输出；如果读取相反的数值，它会打开数字信号；这意味着它已经检测到火灾：

```js
if (val == HIGH) {             
  Serial.print("NO Fire detected "); 
  digitalWrite(ledPin, LOW); // turn LED OFF 
} 
else{ 
  Serial.print("Fire DETECTED "); 
  digitalWrite(ledPin, HIGH);   
  } 
} 

```

当 Arduino 板检测到火灾时，它将在数字输入中读取`*1*`，这意味着没有检测到火灾：

![使用传感器检测火灾](img/B05170_03_17-1.jpg)

如果检测到火灾，数字输入从数字输入读取`*0*`逻辑：

![使用传感器检测火灾](img/B05170_03_18-1.jpg)

# 测量植物的湿度

![测量植物的湿度](img/B05170_03_19-1.jpg)

在本节中，我们将看到使用传感器测试植物和土壤中的湿度：

![测量植物的湿度](img/B05170_03_20-1.jpg)

我现在将介绍这段代码的主要部分。然后我们设置串行通信：

```js
int value;   

void setup() { 
Serial.begin(9600); 
}  

```

在主循环中，我们将从传感器读取模拟信号：

```js
void loop(){   
Serial.print("Humidity sensor value:"); 
Value = analogRead(0);   
Serial.print(value);   

```

我们比较传感器的数值，并在串行接口上显示结果：

```js
if (Value<= 300)   
Serial.println(" Very wet");   
if ((Value > 300) and (Value<= 700))   
Serial.println(" Wet, do not water");    
if (Value> 700)   
Serial.println(" Dry, you need to water");  
delay(1000);  
} 

```

这里，截图显示了读数的结果：

![测量植物的湿度](img/B05170_03_21-1.jpg)

以下截图显示植物不需要水；因为土壤中已经有足够的湿度：

![测量植物的湿度](img/B05170_03_22-1.jpg)

# 测量容器中的水位

有时，我们需要测量容器中的水位，或者如果你想看到水箱中的水位，测量它所含水量是必要的；因此，在本节中，我们将解释如何做到这一点。

传感器通常是打开的。当水超过限制时，接点打开，并向 Arduino 板发送信号。我们使用数字输入的引脚号`2`：

![测量容器中的水位](img/B05170_03_23-1.jpg)

在程序中声明变量和`const`：

```js
const int buttonPin = 2;     // the number of the input sensor pin 
const int ledPin =  13;      // the number of the LED pin 

```

我们还定义数字信号的状态：

```js
// variables will change: 
intbuttonState = 0;         // variable for reading the pushbutton status 

```

我们配置程序的信号、输入和输出：

```js
void setup() { 
  // initialize the LED pin as an output: 
pinMode(ledPin, OUTPUT); 
  // initialize the pushbutton pin as an input: 
pinMode(buttonPin, INPUT); 
Serial.begin(9600); 
} 

```

读取数字输入的状态：

```js
void loop() { 
  // read the state of the pushbutton value: 
buttonState = digitalRead(buttonPin); 

```

我们对传感器进行比较：

```js
if (buttonState == HIGH) { 
Serial.println(buttonState); 
Serial.println("The recipient is fulled"); 
digitalWrite(ledPin, HIGH); 
delay(1000); 
  } 

```

如果传感器检测到**低**水平，容器是空的：

```js
else { 
digitalWrite(ledPin, LOW); 
Serial.println(buttonState); 
Serial.println("The recipient is empty"); 
delay(1000); 
  } 
} 

```

以下截图显示了容器为空时的结果：

![测量容器中的水位](img/B05170_03_24-1.jpg)

水超过限制：

![测量容器中的水位](img/B05170_03_25-1.jpg)

# 测量温度、湿度和光线，并在 LCD 上显示数据

在本节中，我将教你如何在 LCD 屏幕上监测温度、湿度和光线检测。

## 硬件和软件要求

在这个项目中，你将使用 Arduino UNO 板；但你也可以使用 Arduino MEGA，它也可以完美地工作。

对于温度读数，我们需要一个 DHT11 传感器、一个 4.7k 电阻、一个光敏电阻（光传感器）和一个 10k 电阻。

还需要一个 16 x 2 的 LCD 屏幕，您可以在其中进行测试；我使用了一个 I2C 通信模块，用于与 Arduino 板接口的屏幕。我建议使用这种通信，因为只需要 Arduino 的两个引脚来发送数据：

![硬件和软件要求](img/B05170_03_26-1.jpg)

最后，它需要一个面包板和公对公和母对公的电缆进行连接。

以下是项目的组件清单：

+   Arduino UNO

+   温湿度传感器 DHT11

+   LCD 屏幕 16 x 2

+   LCD 的 I2C 模块

+   一个面包板

+   电缆

我们连接不同的组件：

![硬件和软件要求](img/B05170_03_27-1.jpg)

在这里，我们可以看到温湿度 DHT11 传感器的图像：

![硬件和软件要求](img/B05170_03_40.jpg)

然后将**DHT11 传感器（VCC）**的引脚号**1**连接到面包板上的红线，引脚**4**（GND）连接到蓝线。还要将传感器的引脚号**2**连接到 Arduino 板的引脚号**7**。最后，将 4.7k 欧姆的电阻连接到传感器的引脚号**1**和**2**之间。

在面包板上串联一个 10k 欧姆的电阻。然后将光敏电阻的另一端连接到面包板上的红线，电阻的另一端连接到蓝线（地线）。最后，将光敏电阻和电阻之间的公共引脚连接到 Arduino 模拟引脚**A0**。

现在让我们连接 LCD 屏幕。由于我们使用的是带有 I2C 接口的 LCD 屏幕，因此只需要连接两根信号线和两根电源线。将 I2C 模块的引脚**VDC**连接到面包板上的红线，**GND**引脚连接到面包板上的蓝线。然后将**SDA**引脚模块连接到 Arduino 引脚**A4**，**A5 SCL**引脚连接到 Arduino 的引脚：

![硬件和软件要求](img/B05170_03_29-1.jpg)

这是项目的完全组装图像，这样您就可以对整个项目有一个概念：

![硬件和软件要求](img/B05170_03_30-1.jpg)

## 测试传感器

现在硬件项目已经完全组装好，我们将测试不同的传感器。为此，我们将在 Arduino 中编写一个简单的草图。我们只会读取传感器数据，并将这些数据打印在串行端口上。

您现在可以将代码复制到名为`Testing_sensors_Temp_Hum.ino`的文件中，或者只需从此项目的文件夹中获取完整的代码。

首先我们定义库：

```js
#include "DHT.h" 
#define DHTPIN 7  
#define DHTTYPE DHT11 

```

我们定义传感器的类型：

```js
DHT dht(DHTPIN, DHTTYPE); 

```

然后我们配置串行通信：

```js
void setup() 
{ 
Serial.begin(9600); 
dht.begin(); 
} 

```

我们读取传感器数值：

```js
void loop() 
{ 
  float temp = dht.readTemperature(); 
  float hum = dht.readHumidity(); 
  float sensor = analogRead(0); 
  float light = sensor / 1024 * 100; 

```

我们在串行接口上显示数值：

```js
  Serial.print("Temperature: "); 
  Serial.print(temp); 
  Serial.println(" C"); 
  Serial.print("Humidity: "); 
  Serial.print(hum); 
  Serial.println("%"); 
  Serial.print("Light: "); 
  Serial.print(light); 
  Serial.println("%"); 
  delay(700); 
} 

```

将代码下载到 Arduino 板上，并打开串行监视器以显示发送的数据。重要的是要检查串行端口的传输速度，必须为 9600。您应该看到以下内容：

![测试传感器](img/B05170_03_31-1.jpg)

## 在 LCD 上显示数据

现在下一步是将我们的信息集成到 LCD 屏幕上显示。传感器读数部分将保持不变，只是在通信和在 LCD 上显示数据方面进行了详细说明。以下是这部分的完整代码，以及解释。

您现在可以将代码复制到名为`LCD_sensors_temp_hum.ino`的文件中，或者只需从此项目的文件夹中获取完整的代码。

我们为程序包括库：

```js
#include <Wire.h> 
#include <LiquidCrystal_I2C.h> 
#include "DHT.h" 
#define DHTPIN 7  
#define DHTTYPE DHT11 

```

我们为 LCD 定义 LCD 地址：

```js
LiquidCrystal_I2C lcd(0x3F,16,2); 
DHT dht(DHTPIN, DHTTYPE); 

```

我们启动 LCD 屏幕：

```js
void setup() 
{ 
lcd.init(); 
lcd.backlight(); 
lcd.setCursor(1,0); 
lcd.print("Hello !!!"); 
lcd.setCursor(1,1); 
lcd.print("Starting ..."); 

```

我们定义`dht`传感器的开始：

```js
dht.begin(); 
delay(2000); 
lcd.clear(); 
} 

```

我们读取传感器并将数值保存在变量中：

```js
void loop() 
{ 
  float temp = dht.readTemperature(); 
  float hum = dht.readHumidity(); 
  float sensor = analogRead(0); 
  float light = sensor / 1024 * 100; 

```

我们在 LCD 屏幕上显示数值：

```js
  lcd.setCursor(0,0); 
  lcd.print("Temp:"); 
  lcd.print(temp,1); 
  lcd.print((char)223); 
  lcd.print("C"); 
  lcd.setCursor(0,1); 
  lcd.print("Hum:"); 
  lcd.print(hum); 
  lcd.print("%"); 
  lcd.setCursor(11,1); 
  //lcd.print("L:"); 
  lcd.print(light); 
  lcd.print("%"); 
  delay(700); 
} 

```

下一步是在 Arduino 板上下载示例；稍等片刻，您将在 LCD 上看到显示读数。以下是项目运行的图像：

![在 LCD 上显示数据](img/B05170_03_32-1.jpg)

# 使用 PIR 传感器检测运动

我们将建立一个带有常见家庭自动化传感器的项目：运动传感器（PIR）。你是否注意到过那些安装在房屋某些房间顶角的小白色塑料模块，当有人走过时会变成红色的模块？这正是我们这个项目要做的事情。

运动传感器必须有三个引脚：两个用于电源供应，一个用于信号。你还应该使用 5V 电压级别以与 Arduino 卡兼容，Arduino 卡也是在 5V 下运行的。以下图片显示了一个简单的运动传感器：

![使用 PIR 传感器检测运动](img/B05170_03_33-1.jpg)

出于实际目的，我们将使用信号输入 8 来连接运动传感器，5 伏特的信号电压和地**GND**。

## PIR 传感器与 Arduino 接口

PIR 传感器检测体热（红外能量）。被动红外传感器是家庭安全系统中最常用的运动检测器。一旦传感器变热，它就可以检测周围区域的热量和运动，形成一个保护网格。如果移动物体阻挡了太多的网格区域，并且红外能量水平迅速变化，传感器就会被触发。

在这一点上，我们将测试 Arduino 和运动传感器之间的通信。

我们定义变量和串行通信，定义数字引脚 8，输入信号，读取信号状态，并显示传感器的状态信号：

```js
**int sensor = 8;**
**void setup() {**
**Serial.begin(9600);**
**pinMode(sensor,INPUT);**
**}**
**void loop(){**
**// Readind the sensor**
**int state = digitalRead(sensor);**
**Serial.print("Detecting sensor: ");**
**Serial.println(state);**
**delay(100);**
**}**

```

# 使用铁簧管检测门是否打开

已添加一个示例作为选项，以实现磁传感器来检测门或窗户何时打开或关闭。

使用铁簧管检测门是否打开

传感器在检测到磁场时输出`0`，当磁场远离时输出为`1`；因此你可以确定门是打开还是关闭。

Arduino 中的程序执行如下：

我们定义传感器的输入信号，并配置串行通信：

```js
void setup() { 
  pinMode(sensor, INPUT_PULLUP); 
  Serial.begin(9600); 
} 

```

我们读取传感器的状态：

```js
void loop() { 
state = digitalRead(sensor); 

```

它比较数字输入并在串行接口中显示门的状态：

```js
  if (state == LOW){ 
    Serial.println("Door Close"); 
  } 
  if (state == HIGH){ 
    Serial.println("Door Open"); 
  } 
} 

```

# 使用指纹传感器检测谁可以进入房屋

在本节中，我们将创建一个可以帮助我们建立完整安全系统的项目。在这个项目中，指纹访问将通过使用指纹传感器读取指纹来实现，如下图所示：

![使用指纹传感器检测谁可以进入房屋](img/B05170_03_35-1.jpg)

在这部分，我们将看到如何连接和配置我们的硬件，以便激活继电器。

## 硬件配置：

像往常一样，我们将使用 Arduino Uno 板作为项目的大脑。这个项目最重要的部分是指纹传感器。

首先，我们将看到如何组装这个项目的不同部分。让我们从连接电源开始。将 Arduino 板上的**5V**引脚连接到红色电源轨，将 Arduino 的**GND**连接到面包板上的蓝色电源轨。

现在，让我们连接指纹传感器。首先，通过将电缆连接到面包板上的相应颜色来连接电源。然后，将传感器的白色线连接到 Arduino 引脚 3，将绿色线连接到引脚 2。

之后，我们将连接继电器模块。将**VCC**引脚连接到红色电源轨，**GND**引脚连接到蓝色电源轨，将**EN**引脚连接到 Arduino 引脚 7：

![硬件配置：](img/B05170_03_36-1.jpg)

## 保存指纹：

以下示例用于直接从库`Adafruit_Fingerprint`注册 ID 的指纹。

首先，我们定义库：

```js
#include <Adafruit_Fingerprint.h> 
#include <SoftwareSerial.h> 

```

我们定义读取的 ID 和注册过程的功能：

```js
uint8_t id; 
uint8_tgetFingerprintEnroll(); 

```

我们定义与设备的串行通信：

```js
SoftwareSerialmySerial(2, 3); 
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial); 

```

我们声明传感器的实例：

```js
//Adafruit_Fingerprint finger = Adafruit_Fingerprint(&Serial1); 

```

我们设置并显示传感器是否正在配置：

```js
void setup()   
{ 
  while (!Serial); 
  delay(500); 

```

我们显示传感器确认：

```js
  Serial.begin(9600); 
  Serial.println("Adafruit Fingerprint sensor enrollment"); 
  // set the data rate for the sensor serial port 
  finger.begin(57600); 

```

我们识别传感器是否检测到：

```js
  if (finger.verifyPassword()) { 
  Serial.println("Found fingerprint sensor!"); 
  } else { 
    Serial.println("Did not find fingerprint sensor :("); 
    while (1); 
    } 
  } 
  uint8_treadnumber(void) { 
  uint8_tnum = 0; 
  booleanvalidnum = false;  
  while (1) { 
    while (! Serial.available()); 
      char c = Serial.read(); 
      if (isdigit(c)) { 
        num *= 10; 
        num += c - '0'; 
        validnum = true; 
        } else if (validnum) { 
          returnnum; 
        } 
      } 
    } 

```

我们显示注册 ID：

```js
void loop()                     // run over and over again 
{ 
Serial.println("Ready to enroll a fingerprint! Please Type in the ID # you want to save this finger as..."); 
id = readnumber(); 
Serial.print("Enrolling ID #"); 
Serial.println(id); 

while (!  getFingerprintEnroll() ); 
} 

```

注册的功能如下：

```js
uint8_tgetFingerprintEnroll() { 
int p = -1; 
Serial.print("Waiting for valid finger to enroll as #"); Serial.println(id); 
while (p != FINGERPRINT_OK) { 
    p = finger.getImage(); 
switch (p) { 
case FINGERPRINT_OK: 
Serial.println("Image taken"); 
break; 
case FINGERPRINT_NOFINGER: 
Serial.println("."); 
break; 
case FINGERPRINT_PACKETRECIEVEERR: 
Serial.println("Communication error"); 
break; 
case FINGERPRINT_IMAGEFAIL: 
Serial.println("Imaging error"); 
break; 
default: 
Serial.println("Unknown error"); 
break; 
    } 
  } 

```

如果传感器成功读取图像，您将看到以下内容：

```js
  p = finger.image2Tz(1); 
switch (p) { 
case FINGERPRINT_OK: 
Serial.println("Image converted"); 
break; 
case FINGERPRINT_IMAGEMESS: 
Serial.println("Image too messy"); 
return p; 
case FINGERPRINT_PACKETRECIEVEERR: 
Serial.println("Communication error"); 
return p; 
case FINGERPRINT_FEATUREFAIL: 
Serial.println("Could not find fingerprint features"); 
return p; 
case FINGERPRINT_INVALIDIMAGE: 

```

如果无法找到指纹特征，您将看到以下内容：Serial.println("无法找到指纹特征");

```js
return p; 
default: 
Serial.println("Unknown error"); 
return p; 
  } 

```

移除指纹传感器：

```js
Serial.println("Remove finger"); 
delay(2000); 
  p = 0; 
while (p != FINGERPRINT_NOFINGER) { 
p = finger.getImage(); 
  } 
Serial.print("ID "); Serial.println(id); 
p = -1; 
Serial.println("Place same finger again"); 
while (p != FINGERPRINT_OK) { 
    p = finger.getImage(); 
switch (p) { 
case FINGERPRINT_OK: 
Serial.println("Image taken"); 
break; 
case FINGERPRINT_NOFINGER: 
Serial.print("."); 
break; 
case FINGERPRINT_PACKETRECIEVEERR: 
Serial.println("Communication error"); 
break; 
case FINGERPRINT_IMAGEFAIL: 
Serial.println("Imaging error"); 
break; 
default: 
Serial.println("Unknown error"); 
break; 
    } 
  } 

```

指纹传感器的图像：

```js
  p = finger.image2Tz(2); 
switch (p) { 
case FINGERPRINT_OK: 
Serial.println("Image converted"); 
break; 
case FINGERPRINT_IMAGEMESS: 
Serial.println("Image too messy"); 
return p; 
case FINGERPRINT_PACKETRECIEVEERR: 
Serial.println("Communication error"); 
return p; 
case FINGERPRINT_FEATUREFAIL: 
Serial.println("Could not find fingerprint features"); 
return p; 
case FINGERPRINT_INVALIDIMAGE: 
Serial.println("Could not find fingerprint features"); 
return p; 
default: 
Serial.println("Unknown error"); 
return p; 
  } 

```

如果正确，您将看到以下内容：

```js
Serial.print("Creating model for #");  Serial.println(id); 

  p = finger.createModel(); 
if (p == FINGERPRINT_OK) { 
Serial.println("Prints matched!"); 
  } else if (p == FINGERPRINT_PACKETRECIEVEERR) { 
Serial.println("Communication error"); 
return p; 
  } else if (p == FINGERPRINT_ENROLLMISMATCH) { 
Serial.println("Fingerprints did not match"); 
return p; 
  } else { 
Serial.println("Unknown error"); 
return p; 
  }    

```

显示传感器的结果：

```js
Serial.print("ID "); Serial.println(id); 
  p = finger.storeModel(id); 
if (p == FINGERPRINT_OK) { 
Serial.println("Stored!"); 
  } else if (p == FINGERPRINT_PACKETRECIEVEERR) { 
Serial.println("Communication error"); 
return p; 
  } else if (p == FINGERPRINT_BADLOCATION) { 
Serial.println("Could not store in that location"); 
return p; 
  } else if (p == FINGERPRINT_FLASHERR) { 
Serial.println("Error writing to flash"); 
return p; 
  } else { 
Serial.println("Unknown error"); 
return p; 
}    
} 

```

## 测试传感器

打开串行监视器，然后输入在上一步保存的 ID 号码：

![测试传感器](img/B05170_03_37-1.jpg)

以下截图表明您应该再次将同一手指放在传感器上：

![测试传感器](img/B05170_03_38-1.jpg)

以下截图显示传感器响应表明数字指纹已成功保存：

![测试传感器](img/B05170_03_39-1.jpg)

# 摘要

在本章中，我们看到如何与连接到 Arduino 板的不同传感器进行交互，例如用于能源消耗的流量电流，检测家庭风险，实施气体传感器，实施流水传感器以测量水量，制作安全系统，并使用指纹传感器控制访问。所有这些传感器都可以集成到一个完整的系统中，用于监控和控制您在任何项目上工作的一切。

在下一章中，我们将看到如何集成所有内容，监控和控制一个完整的系统，并在仪表板上读取 Arduino 板和树莓派 Zero 作为中央接口的传感器和执行器。
